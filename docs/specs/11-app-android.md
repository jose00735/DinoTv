# 11 · App Android (Kotlin)

## 11.1 Stack

| Área | Tecnología |
|---|---|
| Lenguaje | Kotlin 2.x |
| UI | Jetpack Compose + Material 3 con tema propio "DinoU" |
| Arquitectura | MVVM + UDF (StateFlow), módulos por *feature* |
| DI | Hilt |
| Red | Ktor Client (OkHttp) + kotlinx.serialization; cliente generado de OpenAPI |
| Tiempo real | Ktor WebSockets |
| Descubrimiento | `NsdManager` (mDNS) |
| Escaneo QR | CameraX + ML Kit Barcode |
| Persistencia local | Room (caché de parrilla offline), DataStore (token, preferencias) |
| Trabajo en segundo plano | WorkManager (notificaciones de estrenos) |
| Imágenes | Coil 3 |
| Pruebas | JUnit5, Turbine, Compose UI Test, Paparazzi (capturas) |
| Mín./Objetivo | API 26 / API 35 |

## 11.2 Módulos

```
app/                     navegación, tema, inicio
core/design/             componentes DinoU (Tile, PillButton, BubbleBackground, sonidos)
core/network/            cliente API, WS, auth
core/data/               repositorios, Room
feature/pairing/         descubrimiento + QR
feature/onair/           "Al aire"
feature/grid/            editor de parrilla
feature/wizard/          alta de contenido / append
feature/library/         biblioteca
feature/events/          estrenos y especiales
feature/continuity/      promos, anuncios, paquetes
feature/remote/          control remoto
feature/settings/        canales, fin de emisión, perfiles
```

## 11.3 Navegación

Barra inferior estilo "menú de consola" con 5 destinos: **Al aire · Parrilla · Biblioteca · Estrenos · Más**. Botón flotante redondo (celeste) contextual: en Parrilla abre "Nueva regla"; en Biblioteca abre el wizard.

## 11.4 Pantallas

| Pantalla | Contenido | Maqueta |
|---|---|---|
| Emparejamiento | Lista de servidores encontrados + "Escanear QR" | — |
| **Al aire** | Tarjeta grande del programa actual con barra de progreso en vivo, "A continuación", línea de tiempo de las próximas 6 h, accesos rápidos (Reprogramar siguiente, Control remoto) | `screens/movil-al-aire.html` |
| **Parrilla semanal** | Rejilla días × horas, slots de color de franja, arrastrar para mover, mantener pulsado para menú (Reprogramar, Duplicar, Cancelar), pellizco para zoom | `screens/movil-parrilla.html` |
| Detalle de slot | Lista al segundo: episodios, cortes, promos, relleno; editar política de desborde | — |
| **Reprogramar** | Hoja inferior: Sustituir / Mover / Intercambiar / Cancelar + "Anunciar en pantalla" | `screens/movil-reprogramar.html` |
| Nueva regla | Días (chips), hora, duración (30/60/90/120/libre), contenido, episodios por slot, cortes, vigencia | — |
| Biblioteca | Mosaicos de series con insignias (LINEAL / EPISÓDICA, "3 nuevos"), películas | — |
| **Wizard** | Ver 12 | `screens/movil-wizard.html` |
| Estrenos | Calendario de eventos, crear estreno con campaña de promos | — |
| Continuidad | Promos generadas (previsualizar), anuncios propios, paquete de identidad por franja | — |
| Control remoto | Cruceta grande, CH+/CH-, Guía, Info, Subtítulos, volumen | — |

## 11.5 Lenguaje visual (tema DinoU)

- Fondo blanco/gris perla con patrón sutil de líneas o burbujas difusas.
- Mosaicos con radio 20-28 dp, borde blanco de 3 dp, sombra suave.
- Color primario celeste `#29B6F6`, secundario `#00C2A8`, acento `#FFB300`.
- Tipografía redondeada (M PLUS Rounded 1c / Nunito).
- Micro-animaciones elásticas (`spring(dampingRatio = 0.6f)`) y sonidos cortos de "clic" y "burbuja" (desactivables).

## 11.6 Comportamiento offline

La app guarda la última parrilla en Room; si pierde conexión muestra "Sin conexión con DinoTV" y permite **preparar cambios** que se envían al reconectar (con resolución de conflictos por `generation`).

## 11.7 Notificaciones (Final)

- "🎬 *Película X* empieza en 10 minutos en DinoTV".
- "Quedan 3 capítulos de *Serie Y*. ¿Agregar la siguiente temporada?".
- "Se detectaron 4 episodios nuevos de *Serie Z*" → abre el wizard en modo *append*.
