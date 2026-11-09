# 12 · Wizard de contenido

Objetivo: dar de alta una serie o película **y dejarla programada** en menos de 2 minutos, y permitir el *append* de nuevos capítulos sin romper nada.

## 12.1 Puntos de entrada

1. Botón "+" en Biblioteca → **Nueva serie / Nueva película**.
2. Notificación o insignia "3 nuevos" en una serie → **Append de episodios**.
3. Bandeja de entrada (`/library/inbox`) con archivos detectados sin asignar.

## 12.2 Flujo "Nueva serie" (6 pasos)

| Paso | Nombre | Qué hace el usuario | Qué hace el sistema |
|---|---|---|---|
| 1 | **Origen** | Elige carpeta o archivos de la bandeja | Lista archivos, detecta patrones y duración |
| 2 | **Identidad** | Confirma título, póster, color de marca, clasificación | Sugiere título desde la carpeta (y metadatos online si hay) |
| 3 | **Tipo de serie** | Lineal / Episódica / Mixta (arcos) | Explica con un dibujo cada tipo; sugiere *lineal* si hay temporadas numeradas |
| 4 | **Orden y temporadas** | Revisa la lista ordenada; arrastra para corregir; marca especiales/OVAs como excluidos o como "especial" | Ordena por temporada/episodio; resalta huecos ("falta el ep. 7") y duplicados |
| 5 | **Horario** | Elige días, hora, **cuántas horas/episodios** por día, franja y fecha de inicio | Calcula cuánto durará la serie: "24 eps, 2/día L-V → termina el 18 dic" y qué pasa al terminar (on_finish) |
| 6 | **Promo y resumen** | Activa campaña "¡Nueva serie!" (días previos); revisa resumen | Muestra conflictos con la parrilla y propone soluciones; al confirmar: crea serie, regla, cursor y encarga promos |

### Paso 5 en detalle — dimensionar la serie
- Selector "Por episodios" (n por slot) o "Por tiempo" (slot de 30/60/90/120 min → el sistema calcula cuántos caben con cortes y relleno).
- Indicador visual de ocupación del slot: `[█████████░] 54:10 de 60:00 — 5:50 de continuidad`.
- Si un episodio dura más que el slot → propone ampliar el slot o la política `trim_credits`.

## 12.3 Flujo "Nueva película"

1. Archivo → 2. Identidad (título, año, clasificación, póster) → 3. **Cuándo**: *Estreno* (día/hora únicos), *Ciclo* (regla, p. ej. "Matiné del domingo") o *Solo biblioteca* → 4. Campaña de promos (días previos, insignia) → 5. Resumen con conflictos (la película dura 2:17 → ocupará el slot 20:00-22:30, recorta *Serie X* de las 22:00 → [Mover *Serie X* a 22:30] [Recortar] [Elegir otra hora]).

## 12.4 Flujo "Append de episodios"

```
Detectados 4 archivos nuevos en /Anime/Dino Rangers/
   S02E01  S02E02  S02E03  S02E04
   ¿Agregar al final de "Dino Rangers"?  (último actual: S01E24, abs #24)
```

1. **Confirmar coincidencia**: el sistema propone los archivos (por carpeta y patrón); el usuario puede desmarcar.
2. **Posición**: *Al final* (por defecto: abs 25-28) o *Insertar* después de un episodio (solo si aún no se han emitido los posteriores; si ya se emitieron, advierte).
3. **Efecto en la parrilla**:
   - Si la serie estaba en `retire` o terminada → "Reactivar con su horario anterior (L-V 15:00)" o elegir nuevo horario.
   - Si estaba en `loop` y ya en repetición → "Saltar a los nuevos al terminar la vuelta actual" o "Saltar ya a los nuevos".
4. **Promo**: "¡Nuevos episodios! Desde el lunes" (opcional).
5. **Confirmar** → re-materializa y muestra el diff de la parrilla.

Reglas: un *append* nunca renumera episodios existentes; el cursor no cambia salvo elección explícita.

## 12.5 Estado y API

El wizard es una sesión en servidor (`/wizard/sessions`) con borrador persistido: si la app se cierra, se puede retomar. `commit` es transaccional (todo o nada).

```json
{
  "id": "wz_8f2a",
  "mode": "new_series | new_movie | append",
  "step": 5,
  "draft": {
    "files": [101, 102, "..."],
    "series": { "title": "Dino Rangers", "kind": "linear", "on_finish": "next_season" },
    "schedule": { "days": ["mon","tue","wed","thu","fri"], "start": "15:00", "slot_minutes": 60, "episodes": 2 },
    "promo": { "campaign_days": 5 }
  },
  "analysis": { "missing": [7], "duplicates": [], "ends_on": "2026-12-18", "conflicts": [] }
}
```

## 12.6 Validaciones

| Validación | Mensaje |
|---|---|
| Episodios faltantes | "Falta el episodio 7. ¿Continuar igual?" |
| Duraciones dispares (>2×) | "El ep. 13 dura 46 min (¿especial doble?)" |
| Pistas de audio sin el idioma preferido | "3 episodios no tienen audio en español" |
| Conflicto con otra regla | "Se solapa con *Matiné* el sábado 15:00" |
| Clasificación vs. franja | "+13 en franja infantil (07:00-12:00)" |
