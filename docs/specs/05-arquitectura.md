# 05 · Arquitectura

## 5.1 Vista general

```
┌───────────────────────── Red local (LAN) ─────────────────────────┐
│                                                                   │
│  ┌──────────────┐   REST/WS (JSON)   ┌──────────────────────────┐ │
│  │ App Android  │◄──────────────────►│      dinotv-core (Go)    │ │
│  │ Kotlin       │   mDNS discovery   │  ┌────────┐ ┌─────────┐  │ │
│  │ Compose      │                    │  │Library │ │Scheduler│  │ │
│  └──────────────┘                    │  └───┬────┘ └────┬────┘  │ │
│                                      │      │  SQLite   │       │ │
│  ┌──────────────┐   REST/WS          │  ┌───▼───────────▼────┐  │ │
│  │ Android TV   │◄──────────────────►│  │  Playout Engine    │  │ │
│  │ (Final)      │   HLS opcional     │  └───┬───────────┬────┘  │ │
│  └──────────────┘                    │      │ JSON IPC  │ ffmpeg │ │
│                                      │  ┌───▼───┐  ┌────▼─────┐ │ │
│                                      │  │  mpv  │  │PromoForge│ │ │
│                                      │  └───┬───┘  └──────────┘ │ │
│                                      │  ┌───▼──────────────────┐│ │
│                                      │  │ dinotv-shell (Wails) ││ │
│                                      │  │ UI TV / Media Center ││ │
│                                      │  └──────────────────────┘│ │
│                                      └─────────────┬────────────┘ │
│                                                    │ HDMI         │
│                                                ┌───▼───┐          │
│                                                │  TV   │          │
│                                                └───────┘          │
└───────────────────────────────────────────────────────────────────┘
```

## 5.2 Componentes

| Componente | Lenguaje | Responsabilidad |
|---|---|---|
| `dinotv-core` | Go 1.23+ | Daemon principal. Expone API, contiene biblioteca, scheduler, playout y PromoForge. |
| `library` | Go | Escaneo de carpetas (fsnotify), `ffprobe`, parseo de nombres, miniaturas. |
| `scheduler` | Go | Reglas → parrilla materializada; relleno; reprogramaciones; cursores. **Puro y determinista** (sin E/S) para poder probarlo. |
| `playout` | Go | Reloj maestro, cola de reproducción, control de `mpv` vía JSON IPC, overlays, recuperación de fallos. |
| `promoforge` | Go + ffmpeg | Generación offline de promos, cortinillas y cartones a partir de plantillas. |
| `api` | Go (`net/http` + `chi`, `nhooyr/websocket`) | REST, WebSocket, autenticación por token, mDNS (`hashicorp/mdns`). |
| `dinotv-shell` | Go + Wails v2 (HTML/CSS/TS) | Interfaz de TV y Media Center. Ventana transparente sobre `mpv` para menús, guía y OSD. |
| `dinotv-android` | Kotlin, Jetpack Compose | App de control (panel de parrilla, wizard, control remoto). |
| `dinotv-androidtv` | Kotlin, Compose for TV + Media3/ExoPlayer | Cliente receptor (Final): reproduce el canal desde el core vía HLS. |

## 5.3 Modos de despliegue

1. **Caja TV (recomendado)**: mini-PC (N100 o Raspberry Pi 5) conectado al TV por HDMI. Ejecuta `dinotv-core` + `dinotv-shell` en modo kiosco al arrancar.
2. **PC Media Center**: Windows/Linux/macOS; el shell en ventana o pantalla completa con panel de administración.
3. **Servidor + receptores (Final)**: el core en un NAS/PC; los televisores con Android TV reciben el canal como HLS "en vivo" (todos ven lo mismo a la misma hora, como TV real).

## 5.4 Flujo principal

1. El usuario agrega carpetas → `library` indexa → la app muestra contenido nuevo.
2. El usuario crea reglas en la app → `api` → `scheduler` valida y **materializa** la semana → guarda `schedule_items`.
3. `playout` lee los items de la ventana [ahora-1 h, ahora+6 h], calcula el elemento y el offset actual, y ordena a `mpv` reproducir con `start=offset`.
4. Al terminar cada elemento, `playout` re-sincroniza con el reloj (ver 08).
5. Cada cambio publica eventos por WebSocket → la app y el shell se actualizan.

## 5.5 Decisiones de arquitectura (ADR)

### ADR-001 · mpv como motor de reproducción
- **Decisión**: usar `mpv` controlado por JSON IPC (socket Unix / named pipe).
- **Motivo**: decodificación por hardware en todas las plataformas, seek preciso (`hr-seek`), soporte de shaders GLSL (filtro CRT), subtítulos ASS impecables (anime), *gapless* con `prefetch-playlist`.
- **Alternativas**: GStreamer (más complejo), libVLC (bindings Go inestables), HTML5 video (códecs limitados).

### ADR-002 · Wails para la interfaz de TV/PC
- **Decisión**: Wails v2 (Go + WebView) con frontend HTML/CSS/TypeScript.
- **Motivo**: un único lenguaje de backend (Go), la estética tipo Wii U se logra fácil con CSS, se reutilizan las maquetas de `screens/`.
- **Integración con video**: ventana `mpv` a pantalla completa debajo; ventana Wails transparente sin bordes encima para menús y overlays. En Raspberry Pi se usa `mpv --vo=gpu --gpu-context=drm` y overlays nativos de mpv (OSD ASS) como respaldo.

### ADR-003 · SQLite como almacenamiento
- **Decisión**: SQLite (modo WAL) con `modernc.org/sqlite` (sin CGO) y migraciones con `goose`.
- **Motivo**: cero administración, respaldo = copiar un archivo, rendimiento sobrado.

### ADR-004 · Parrilla materializada
- **Decisión**: las reglas se expanden a filas concretas con hora al segundo (no se calculan al vuelo).
- **Motivo**: la TV y la app ven exactamente lo mismo, permite previsualizar, editar puntualmente y auditar lo emitido.
- **Coste**: re-materializar al cambiar reglas (rápido: < 500 ms por semana).

### ADR-005 · Kotlin + Jetpack Compose para Android
- **Decisión**: Compose + Material 3 con tema propio, Ktor Client, kotlinx.serialization, Hilt, Room (caché offline), DataStore.
- **Motivo**: UI declarativa ideal para la parrilla con drag & drop y animaciones elásticas.

### ADR-006 · Comunicación
- **Decisión**: REST para comandos y consultas; WebSocket para eventos (`now_playing`, `schedule_changed`, `library_updated`).
- **Contrato**: OpenAPI 3.1 en `api/openapi.yaml`; se genera el cliente Kotlin con `openapi-generator` y los handlers Go con `oapi-codegen`.

## 5.6 Paquetes Go (core)

```
dinotv-core/
  cmd/dinotv/            main, flags, arranque
  internal/clock/        reloj inyectable (real / simulado para pruebas)
  internal/library/      scanner, probe, parser de nombres, watcher
  internal/scheduler/    rules, materialize, filler, overrides, cursor
  internal/playout/      engine, mpvipc, overlays, failover
  internal/promoforge/   templates, ffmpeg graph builder, cache
  internal/store/        sqlite, migraciones, repositorios
  internal/api/          http, ws, auth, mdns
  internal/events/       bus de eventos interno (pub/sub)
  pkg/model/             tipos compartidos
```
