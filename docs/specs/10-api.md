# 10 · API

Base: `http://<host>:7770/api/v1` · Formato JSON · Contrato completo en `api/openapi.yaml`.

## 10.1 Descubrimiento y emparejamiento

1. El core anuncia `_dinotv._tcp.local` por mDNS (TXT: `version`, `name`, `id`).
2. La TV muestra un **QR** con `dinotv://pair?host=192.168.1.20:7770&code=483-921`.
3. La app hace `POST /pair {code, device_name}` → recibe `token` (JWT firmado con clave local, sin caducidad, revocable).
4. Todas las peticiones llevan `Authorization: Bearer <token>`. Roles: `admin`, `viewer`.

## 10.2 Recursos REST

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/status` | Versión, hora del servidor, desfase NTP, canales al aire |
| GET | `/channels` · POST · PATCH `/channels/{id}` | CRUD de canales |
| GET | `/channels/{id}/now` | Elemento actual, offset, siguiente |
| GET | `/channels/{id}/schedule?from&to&granularity=slot\|item` | Parrilla materializada |
| GET/POST/PATCH/DELETE | `/channels/{id}/rules` | Reglas recurrentes |
| POST | `/channels/{id}/rules:validate` | Valida sin guardar, devuelve warnings y vista previa |
| GET/POST/DELETE | `/channels/{id}/overrides` | Reprogramaciones |
| POST | `/channels/{id}/overrides:undo` | Deshacer último cambio |
| GET/POST/PATCH | `/channels/{id}/events` | Estrenos y especiales |
| GET/POST/PATCH | `/blocks` | Franjas |
| GET | `/library/series` · `/library/series/{id}` | Series con temporadas/episodios |
| PATCH | `/library/series/{id}` | Tipo, política de fin, idiomas |
| PUT | `/library/series/{id}/cursor?channel=` | Mover cursor manualmente |
| GET | `/library/movies` · PATCH `/library/movies/{id}` | Películas |
| POST | `/library/scan` | Re-escanear carpetas |
| GET | `/library/inbox` | Archivos nuevos sin asignar (para el wizard) |
| POST | `/wizard/sessions` | Inicia sesión de wizard (ver 12) |
| PATCH | `/wizard/sessions/{id}` | Actualiza paso |
| POST | `/wizard/sessions/{id}:commit` | Confirma el alta/append |
| GET/POST | `/continuity/assets` | Promos, anuncios, cortinillas |
| POST | `/continuity/promos:generate` | Forzar generación de promo |
| GET/POST | `/packs` | Paquetes de identidad |
| POST | `/remote` | Comandos de control remoto `{cmd: "channel_up" \| "guide" \| "info" \| "volume", value}` |
| GET | `/export` · POST `/import` | Respaldo de configuración |

## 10.3 Ejemplos

**Crear regla** — "Lun a Vie 15:00, slot de 60 min, 2 episodios de *Dino Rangers*":

```http
POST /api/v1/channels/1/rules
{
  "block_id": 3,
  "days": ["mon","tue","wed","thu","fri"],
  "start_time": "15:00",
  "slot_minutes": 60,
  "entries": [{ "series_id": 12, "episodes": 2 }],
  "ad_breaks": 1,
  "overflow_policy": "trim_credits",
  "valid_from": "2026-11-16"
}
→ 201 { "id": 44, "warnings": [], "preview": [ {"start":"2026-11-16T15:00:00-05:00","items":[...]} ] }
```

**Reprogramar** — sustituir el slot del viernes 20:00 por una película:

```http
POST /api/v1/channels/1/overrides
{ "target_start": "2026-11-20T20:00:00-05:00", "action": "replace",
  "payload": { "movie_id": 7 }, "reason": "Noche de cine", "announce": true }
```

## 10.4 WebSocket `/ws`

Mensajes del servidor: `{ "type": "...", "channel_id": 1, "data": {...}, "ts": 1731182400000 }`

| type | data |
|---|---|
| `now_playing` | item, offset_ms, next_item |
| `schedule_changed` | generation, changed_slots[] |
| `library_updated` | added[], removed[], inbox_count |
| `promo_ready` | asset_id, target |
| `alert` | level, message ("Archivo dañado: …") |
| `remote` | eco de comandos (para sincronizar UI) |

Cliente → servidor: `subscribe {channels: [1,2]}`, `ping`.

## 10.5 Errores

Formato RFC 9457 (*problem+json*): `{ "type": "/errors/slot-conflict", "title": "El slot se solapa", "detail": "...", "conflicts": [...] }`.
