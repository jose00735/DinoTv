# 06 · Modelo de datos

## 6.1 Diagrama de entidades

```
channel 1───* block (franja) 1───* rule ───* rule_entry ──► series | movie | collection
   │                                  │
   │                                  ▼ (materialización)
   └──1───* schedule_item ◄───── override (reprogramación)
                 │
                 ▼
            media_file ◄─── episode *───1 season *───1 series
                 ▲                                    │
              movie                         series_cursor (por canal)

continuity_asset (promo, cortinilla, ident, anuncio, cartón) ─► media_file
promo_job ─► continuity_asset          event (estreno) ─► movie|episode
play_log (lo emitido realmente)
```

## 6.2 Esquema SQL (SQLite)

```sql
CREATE TABLE channel (
  id            INTEGER PRIMARY KEY,
  slug          TEXT UNIQUE NOT NULL,          -- "dino", "dino-kids"
  name          TEXT NOT NULL,
  number        INTEGER UNIQUE NOT NULL,       -- canal 1, 2, 3...
  color_hex     TEXT NOT NULL DEFAULT '#29B6F6',
  logo_path     TEXT,
  timezone      TEXT NOT NULL DEFAULT 'America/Bogota',
  signoff_start TEXT,                          -- '02:00' fin de emisión (NULL = 24 h)
  signoff_end   TEXT,                          -- '06:00'
  flexible_mode INTEGER NOT NULL DEFAULT 0
);

CREATE TABLE media_file (
  id           INTEGER PRIMARY KEY,
  path         TEXT UNIQUE NOT NULL,
  hash_quick   TEXT NOT NULL,                  -- tamaño + 64 KiB inicio/fin (detectar renombres)
  duration_ms  INTEGER NOT NULL,
  width        INTEGER, height INTEGER,
  video_codec  TEXT, audio_tracks_json TEXT, subtitle_tracks_json TEXT,
  intro_start_ms INTEGER, intro_end_ms INTEGER, credits_start_ms INTEGER,
  ad_markers_json TEXT,                        -- [ms, ms, ...] puntos de corte
  status       TEXT NOT NULL DEFAULT 'ok'      -- ok | missing | broken
);

CREATE TABLE series (
  id            INTEGER PRIMARY KEY,
  title         TEXT NOT NULL,
  kind          TEXT NOT NULL CHECK (kind IN ('linear','episodic','mixed')),
  folder_path   TEXT,                          -- para detectar nuevos episodios (append)
  poster_path   TEXT, brand_color TEXT,
  rating        TEXT,                          -- 'TP', '+7', '+13', '+18'
  default_audio_lang TEXT, default_sub_lang TEXT,
  on_finish     TEXT NOT NULL DEFAULT 'loop'   -- loop | next_season | retire | replace
                CHECK (on_finish IN ('loop','next_season','retire','replace')),
  replace_series_id INTEGER REFERENCES series(id),
  skip_intro_on_marathon INTEGER NOT NULL DEFAULT 0
);

CREATE TABLE season (
  id INTEGER PRIMARY KEY, series_id INTEGER NOT NULL REFERENCES series(id),
  number INTEGER NOT NULL, title TEXT, UNIQUE(series_id, number)
);

CREATE TABLE episode (
  id            INTEGER PRIMARY KEY,
  series_id     INTEGER NOT NULL REFERENCES series(id),
  season_id     INTEGER REFERENCES season(id),
  abs_number    INTEGER NOT NULL,              -- orden absoluto (clave del cursor)
  number        INTEGER NOT NULL,              -- número dentro de la temporada
  title         TEXT, synopsis TEXT,
  arc_id        INTEGER,                       -- series 'mixed': episodios del mismo arco van seguidos
  media_file_id INTEGER NOT NULL REFERENCES media_file(id),
  UNIQUE(series_id, abs_number)
);

CREATE TABLE movie (
  id INTEGER PRIMARY KEY, title TEXT NOT NULL, year INTEGER, synopsis TEXT,
  rating TEXT, poster_path TEXT,
  allowed_from TEXT, allowed_to TEXT,          -- ventana horaria permitida '21:00'-'05:00'
  media_file_id INTEGER NOT NULL REFERENCES media_file(id)
);

CREATE TABLE series_cursor (
  channel_id INTEGER NOT NULL REFERENCES channel(id),
  series_id  INTEGER NOT NULL REFERENCES series(id),
  next_abs_number INTEGER NOT NULL DEFAULT 1,  -- lineal
  shuffle_seed INTEGER, shuffle_pos INTEGER,   -- episódica: permutación determinista
  cycle INTEGER NOT NULL DEFAULT 1,            -- vuelta actual (repeticiones)
  PRIMARY KEY (channel_id, series_id)
);

CREATE TABLE block (                           -- franja
  id INTEGER PRIMARY KEY, channel_id INTEGER NOT NULL REFERENCES channel(id),
  name TEXT NOT NULL, color_hex TEXT,
  bumper_in_asset_id INTEGER, bumper_out_asset_id INTEGER,
  ident_asset_id INTEGER, theme TEXT           -- 'anime', 'matine', 'nocturno'...
);

CREATE TABLE rule (
  id          INTEGER PRIMARY KEY,
  channel_id  INTEGER NOT NULL REFERENCES channel(id),
  block_id    INTEGER REFERENCES block(id),
  days_mask   INTEGER NOT NULL,                -- bit0=Lunes ... bit6=Domingo
  start_time  TEXT NOT NULL,                   -- 'HH:MM'
  slot_minutes INTEGER NOT NULL,               -- duración del slot
  valid_from  DATE NOT NULL, valid_to DATE,    -- vigencia
  overflow_policy TEXT NOT NULL DEFAULT 'trim_credits'
              CHECK (overflow_policy IN ('trim_credits','push_next','skip_last','allow_overrun')),
  ad_breaks   INTEGER NOT NULL DEFAULT 1,      -- cortes por episodio
  priority    INTEGER NOT NULL DEFAULT 0
);

CREATE TABLE rule_entry (                      -- qué llena el slot (en orden)
  id INTEGER PRIMARY KEY, rule_id INTEGER NOT NULL REFERENCES rule(id),
  position INTEGER NOT NULL,
  series_id INTEGER REFERENCES series(id),
  movie_id INTEGER REFERENCES movie(id),
  episodes INTEGER NOT NULL DEFAULT 1,         -- episodios por slot (0 = los que quepan)
  CHECK ((series_id IS NULL) <> (movie_id IS NULL))
);

CREATE TABLE event (                           -- estrenos / especiales
  id INTEGER PRIMARY KEY, channel_id INTEGER NOT NULL,
  starts_at DATETIME NOT NULL, slot_minutes INTEGER NOT NULL,
  movie_id INTEGER, episode_id INTEGER, title TEXT,
  promo_campaign_days INTEGER NOT NULL DEFAULT 5,  -- días de promos previas
  badge TEXT DEFAULT 'ESTRENO'                 -- ESTRENO | ESPECIAL | MARATÓN | FINAL DE TEMPORADA
);

CREATE TABLE override (                        -- reprogramaciones
  id INTEGER PRIMARY KEY, channel_id INTEGER NOT NULL,
  target_start DATETIME NOT NULL,              -- slot afectado
  action TEXT NOT NULL CHECK (action IN ('cancel','replace','move','swap','insert')),
  payload_json TEXT NOT NULL,
  reason TEXT, announce INTEGER NOT NULL DEFAULT 1,
  created_at DATETIME NOT NULL, created_by TEXT
);

CREATE TABLE continuity_asset (
  id INTEGER PRIMARY KEY, channel_id INTEGER,
  kind TEXT NOT NULL CHECK (kind IN ('ident','bumper_in','bumper_out','promo','ad','card','filler','signoff')),
  media_file_id INTEGER REFERENCES media_file(id),
  target_series_id INTEGER, target_event_id INTEGER, -- de qué trata la promo
  valid_until DATETIME, weight INTEGER NOT NULL DEFAULT 1,
  max_per_hour INTEGER, generated INTEGER NOT NULL DEFAULT 0
);

CREATE TABLE schedule_item (                   -- PARRILLA MATERIALIZADA
  id INTEGER PRIMARY KEY,
  channel_id INTEGER NOT NULL,
  starts_at_ms INTEGER NOT NULL,               -- epoch ms UTC
  duration_ms INTEGER NOT NULL,
  kind TEXT NOT NULL,                          -- episode | movie | promo | bumper | ad | ident | card | filler | signoff
  media_file_id INTEGER, in_point_ms INTEGER NOT NULL DEFAULT 0, out_point_ms INTEGER,
  episode_id INTEGER, movie_id INTEGER, asset_id INTEGER,
  rule_id INTEGER, override_id INTEGER, event_id INTEGER,
  slot_start_ms INTEGER NOT NULL,              -- slot al que pertenece
  overlay_json TEXT,                           -- lower thirds, badges, mosca
  generation INTEGER NOT NULL,                 -- versión de materialización
  UNIQUE(channel_id, starts_at_ms, generation)
);
CREATE INDEX idx_sched_time ON schedule_item(channel_id, starts_at_ms);

CREATE TABLE play_log (
  id INTEGER PRIMARY KEY, channel_id INTEGER, schedule_item_id INTEGER,
  started_at_ms INTEGER, ended_at_ms INTEGER, drift_ms INTEGER, result TEXT
);
```

## 6.3 Invariantes

1. Para un canal, los `schedule_item` de la generación activa **no se solapan** y **no dejan huecos** (`starts_at[i+1] = starts_at[i] + duration[i]`).
2. Cada slot de regla empieza exactamente en `start_time` (zona horaria del canal), salvo `allow_overrun` o `push_next`.
3. Para una serie lineal, los episodios emitidos en un canal forman una secuencia estrictamente creciente de `abs_number` dentro de un ciclo.
4. El cursor solo se **confirma** cuando un episodio se emite ≥ 80 % (o pasa su hora en `play_log`); materializar el futuro usa un cursor *proyectado*.
5. Un *append* nunca modifica `abs_number` existentes; los nuevos episodios reciben `max(abs_number)+1…`.

## 6.4 Exportación

`GET /api/v1/export` devuelve un JSON con todas las tablas menos `schedule_item` y `play_log` (regenerables), para respaldos y para mover la configuración a otro equipo.
