# 08 · Motor de emisión (playout)

## 8.1 Reloj maestro

- La fuente de verdad es el reloj del sistema (sincronizado por NTP). El playout nunca usa "el video terminó" como señal de tiempo, sino **la hora de inicio programada del siguiente elemento**.
- Bucle principal (cada 250 ms y en cada evento de mpv):

```
item, offset := plan.At(clock.Now())          // elemento y desplazamiento que tocan AHORA
if item != current:
    load(item, start = item.in_point + offset)
else if |player.pos - (item.in_point + offset)| > 750 ms:
    seek(exact)                                  // corrige deriva
schedule overlays (lower thirds, mosca, reloj) por tiempo
```

## 8.2 Incorporación tardía (join-in-progress)

Al arrancar o cambiar de canal: `plan.At(now)` devuelve p. ej. *Episodio 14, offset 12:03.4* → `mpv loadfile ... start=723.4` con `hr-seek=yes`. La guía y la app muestran el mismo progreso.

## 8.3 Transiciones sin cortes

- `mpv` con playlist de 2 elementos (actual + siguiente) y `prefetch-playlist=yes` → el siguiente archivo se abre antes de terminar el actual.
- Si el siguiente elemento empieza en `t`, se emite `playlist-next` en `t - latencia_media` (medida en tiempo de ejecución, típicamente 40-120 ms).
- Fundido a negro opcional de 200 ms en cortinillas (filtro `fade` en lavfi).

## 8.4 Overlays

| Overlay | Cuándo | Implementación |
|---|---|---|
| Mosca (logo) | Durante programas (no en promos) | Shell Wails (capa transparente) o `mpv overlay-add` |
| Reloj | Opcional, franjas matutinas | Shell |
| Lower third "A continuación" | `T_fin - 45 s`, 8 s de duración | Shell, animación deslizante |
| Insignia ESTRENO / REPETICIÓN / EN VIVO | Primeros 10 s del programa | Shell |
| Aviso de clasificación (+13) | Primeros 5 s | Shell |
| Cartón dinámico (ajuste fino) | Relleno residual | Shell renderiza; mpv muestra fondo de video en loop |

Los overlays se describen en `schedule_item.overlay_json` con tiempos relativos al elemento, de modo que el shell y el cliente Android TV los rendericen igual.

## 8.5 Fallos y recuperación

| Fallo | Reacción |
|---|---|
| Archivo inexistente o corrupto | Emitir **relleno de emergencia** (ident + cartón "Volvemos enseguida") hasta el siguiente elemento; marcar `media_file.status = broken`; notificar a la app. |
| mpv se cierra | Supervisor reinicia mpv (< 2 s) y retoma en `plan.At(now)`. |
| Corte de energía / reinicio | Al iniciar, el core carga el plan y entra en el punto exacto (RNF-05). |
| Plan vacío (no hay contenido) | Carta de ajuste con música y mensaje "Configura tu parrilla desde la app". |
| Reloj saltó (NTP corrigió > 5 s) | Re-sincronizar inmediatamente con seek. |

## 8.6 Fin de emisión

Si el canal tiene `signoff_start/end`: secuencia de cierre (ident nocturno → mensaje "Fin de las transmisiones, nos vemos a las 6:00" → carta de ajuste con tono de 1 kHz opcional o música lo-fi). A la hora de inicio: secuencia de apertura ("Buenos días, comienza la transmisión") e ident.

## 8.7 Multicanal

- Beta: 1 canal emitido; los demás se pueden configurar pero solo uno "al aire".
- Final: N canales lógicos; todos avanzan en el tiempo aunque nadie los mire (el plan es una función del tiempo, no requiere reproducir). Cambiar de canal = `plan(canal).At(now)`.
- Salida HLS (Final): `ffmpeg` transcodifica/reempaqueta en vivo el canal seleccionado a segmentos de 4 s para receptores Android TV; todos los receptores ven lo mismo a la misma hora.
