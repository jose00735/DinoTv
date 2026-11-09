# 07 · Motor de programación (scheduler)

El scheduler es una librería Go **pura**: recibe reglas, contenido, cursores y reloj; devuelve una lista de `schedule_item`. No hace E/S, lo que permite pruebas deterministas con reloj simulado.

## 7.1 Pipeline de materialización

```
Reglas vigentes ─┐
Eventos ─────────┼─► 1. Expandir slots ─► 2. Aplicar eventos ─► 3. Aplicar overrides
Overrides ───────┘                                                    │
                                                                      ▼
            6. Validar ◄─ 5. Rellenar continuidad ◄─ 4. Asignar contenido (cursores)
                │
                ▼
         schedule_item (generation N+1)  → swap atómico → evento `schedule_changed`
```

1. **Expandir slots**: para cada día del rango y cada regla cuyo `days_mask` incluya el día → slot `[start, start+slot_minutes)` en la zona horaria del canal (manejo de cambios de horario de verano incluido).
2. **Aplicar eventos**: un estreno reemplaza los slots que solapa. Los slots parcialmente pisados se recortan según la política de la regla.
3. **Aplicar overrides** en orden cronológico de creación (`cancel`, `replace`, `move`, `swap`, `insert`).
4. **Asignar contenido** a cada slot en orden temporal usando cursores proyectados (7.3).
5. **Rellenar** el tiempo sobrante con continuidad (7.4).
6. **Validar** invariantes (06 §6.3). Si falla, se mantiene la generación anterior y se reporta el error a la app.

Los huecos sin regla se rellenan con la **programación de relleno del canal** (una regla "comodín" con series episódicas marcadas como *relleno*), o con carta de ajuste si está en horario de fin de emisión.

## 7.2 Tipos de serie

| Tipo | Selección del siguiente episodio |
|---|---|
| `linear` | `cursor.next_abs_number`; avanza 1 por episodio emitido. |
| `episodic` | Permutación determinista (`shuffle_seed`, Fisher-Yates) de todos los episodios; se recorre sin repetir hasta agotarla; luego nueva semilla y `cycle+1`. Evita que el último de un ciclo sea el primero del siguiente. |
| `mixed` | Como `episodic`, pero los episodios con el mismo `arc_id` forman una unidad que se emite en orden y de forma contigua (aunque cruce días). |

Al terminar una serie lineal se aplica `on_finish`:
- `loop`: vuelve al episodio 1 (`cycle+1`) y el slot muestra la insignia **"REPETICIÓN"**.
- `next_season`: igual que *loop* pero solo si hay temporada siguiente; si no, *retire*.
- `retire`: los slots futuros de esa serie pasan al contenido de relleno y la app avisa "Se acabó *X*: elige reemplazo".
- `replace`: los slots pasan a `replace_series_id` y se genera automáticamente una promo "¡Nueva serie! Desde el lunes".

## 7.3 Cursores proyectados

Materializar 2 semanas por adelantado requiere saber qué episodio caerá el martes siguiente. Se usa un **cursor proyectado**: copia en memoria del cursor real que avanza conforme se asignan slots. El cursor real (en BD) solo avanza al confirmarse la emisión en `play_log`.

Consecuencia: si se cancela el slot del lunes, al re-materializar el martes recibe el episodio que iba a ir el lunes → **no se pierden capítulos** (RF-REP-03).

## 7.4 Algoritmo de relleno al segundo

Para cada slot con contenido de duración total `C` y duración de slot `S`:

```
gap = S - C - Σ(cortes dentro del contenido)
si gap >= 0:
    rellenar(gap)
si gap < 0:   # el contenido no cabe
    según overflow_policy:
      trim_credits  → recortar créditos finales (credits_start_ms) de los episodios hasta |gap|
      skip_last     → quitar el último episodio del slot (no avanza el cursor) y recalcular
      push_next     → el slot termina tarde y empuja el siguiente (solo si el siguiente lo permite)
      allow_overrun → se come el inicio del siguiente slot (películas largas)
```

`rellenar(gap)` construye una secuencia con prioridad y presupuesto de tiempo, eligiendo assets por **mochila acotada** (knapsack 0/1 sobre duraciones en ms, con pesos de prioridad) para ajustar lo más cerca posible:

| Orden | Pieza | Regla |
|---|---|---|
| 1 | Cortinilla de salida del programa | Siempre si existe |
| 2 | Promo "A continuación" del siguiente slot | Siempre si gap ≥ 10 s |
| 3 | Promo de evento/estreno próximo | Si hay campaña activa |
| 4 | Promos "Esta semana" / "Hoy más tarde" | Rotación por peso, máx. 1 de la misma serie |
| 5 | Anuncios del usuario | Respeta `max_per_hour` |
| 6 | ID del canal | Justo antes del siguiente programa |
| 7 | **Ajuste fino** | El residuo (< duración mínima de asset) se cubre con un *cartón dinámico* generado en tiempo real (reloj del canal + "A continuación" + música), de duración exacta al milisegundo. |

Así cada programa comienza **exactamente** en su hora.

### Cortes comerciales internos
Si `ad_breaks = n > 0`, el episodio se divide en `n+1` partes usando `ad_markers_json` (o, si no hay, en puntos equidistantes ajustados al silencio más cercano detectado con `ffmpeg silencedetect`). Cada corte: cortinilla de salida → 1-3 piezas → cortinilla de entrada ("Regresamos con…"). El tiempo de los cortes se descuenta del `gap`.

## 7.5 Reprogramaciones (overrides)

| Acción | Payload | Efecto |
|---|---|---|
| `cancel` | `{}` | El slot queda vacío → se llena con relleno o con lo que diga `fill_with`. |
| `replace` | `{series_id\|movie_id, episodes}` | Cambia el contenido del slot. |
| `move` | `{to_start}` | Mueve el slot a otra hora (el origen se cancela). |
| `swap` | `{other_start}` | Intercambia dos slots. |
| `insert` | `{start, minutes, content}` | Inserta un especial; lo afectado se recorta o empuja (`push_policy`). |

Reprogramar el slot **en emisión** solo se permite para el futuro (desde el siguiente elemento). Si `announce = 1` se agrega al relleno previo un cartón "**Cambio de programación**: hoy a las 20:00 *Película X* en lugar de *Serie Y*".

## 7.6 Eventos y campañas de promos

Al crear un evento (estreno) con `promo_campaign_days = d`:
1. Se encarga a PromoForge una promo con insignia (`ESTRENO`), título, día y hora ("Este viernes 8 PM").
2. Durante los `d` días previos, la promo tiene prioridad 3 en el relleno, creciendo en peso a medida que se acerca la fecha ("Mañana", "Hoy 8 PM", "En minutos").
3. 1 h antes, el lower third cambia a "**Hoy a las 8 PM: estreno**".

## 7.7 Re-materialización

- Ventana: desde *ahora + elemento actual* hasta `+14 días` (configurable a 8 semanas).
- Disparadores: cambio de reglas/overrides/eventos, *append* de episodios, archivo faltante, medianoche (extender ventana).
- Lo que ya se está emitiendo **nunca** cambia; la nueva generación arranca en el siguiente límite de elemento.
- Diff entre generaciones para notificar a la app solo los slots modificados.

## 7.8 Interfaz Go (borrador)

```go
type Clock interface{ Now() time.Time }

type Materializer interface {
    Materialize(ctx context.Context, ch model.Channel, from, to time.Time, in Inputs) (Plan, error)
}

type Inputs struct {
    Rules     []model.Rule
    Events    []model.Event
    Overrides []model.Override
    Cursors   map[model.SeriesID]model.Cursor
    Library   LibraryView     // episodios, películas, assets con duraciones
    Filler    FillerPolicy
}

type Plan struct {
    Items      []model.ScheduleItem
    Warnings   []Warning       // "Película excede slot por 4 min", "Serie X sin episodios"
    Generation int64
}
```
