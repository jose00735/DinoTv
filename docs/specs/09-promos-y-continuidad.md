# 09 · Promos y continuidad

La continuidad es lo que separa a DinoTV de una lista de reproducción. Inspiración: los **bloques juveniles de cable de principios de los 2000**: promos rápidas con cortes al ritmo de la música, tipografía gruesa, rótulos de "día y hora" y cortinillas que te mantenían pegado al canal. DinoTV recrea ese *estilo* con identidad propia (sin usar marcas ajenas).

## 9.1 Tipos de pieza

| Pieza | Duración típica | Ejemplo |
|---|---|---|
| **Ident** | 3-8 s | Logo de DinoTV animado (dinosaurio que asoma y ruge) |
| **Cortinilla de salida** | 3-5 s | "¡Ya volvemos!" con el personaje de la serie |
| **Cortinilla de entrada** | 3-5 s | "Regresamos con… *Serie X*" |
| **Promo "A continuación"** | 8-15 s | Clip del próximo episodio + "Sigue: *Serie Y*" |
| **Promo "Hoy más tarde"** | 15-20 s | Montaje + "Hoy 18:00" |
| **Promo semanal** | 20-30 s | "Esta semana en DinoTV" con 4-5 series y sus días |
| **Promo de estreno** | 20-30 s | Película con insignia ESTRENO, cuenta regresiva de días |
| **Cartón** | variable | Pantalla estática animada: guía del bloque, "Cambio de programación" |
| **Anuncio del usuario** | 15-60 s | Comerciales retro que el usuario agregue |
| **Mini corto / interstitial** | 30-90 s | Cortos animados, "Sabías que…", trivia del canal |

## 9.2 PromoForge (generador)

Servicio Go que construye promos con `ffmpeg` *filter_complex* a partir de **plantillas declarativas** (YAML) y las guarda como `continuity_asset` (generated = 1) en caché. Se regeneran solo cuando cambia su información (p. ej. el día de emisión).

### Selección de clips
1. Se excluyen intro y créditos (`intro_*`, `credits_start_ms`).
2. Se calculan candidatos con **detección de escenas** (`select='gt(scene,0.35)'`) y **energía de audio** (picos de volumen = acción).
3. Se eligen 3-6 clips de 1.5-3 s repartidos por el episodio, evitando el último 20 % (sin *spoilers* del final). Para series lineales, opción "sin spoilers estricto": usar solo el primer 40 %.

### Plantilla de ejemplo

```yaml
id: promo_hoy_mas_tarde_v1
duration: 15s
resolution: 1920x1080
music: assets/music/energetic_loop_01.ogg   # librería libre de derechos incluida
layers:
  - type: clips
    source: "{{episode}}"
    count: 4
    each: 2.2s
    transition: whip_pan          # zoom/desenfoque rápido
  - type: shape
    kind: slanted_bar
    color: "{{series.brand_color}}"
    from: 9s
    animate: slide_in_left
  - type: text
    text: "{{series.title | upper}}"
    font: assets/fonts/DinoRounded-Black.ttf
    size: 96
    from: 9.3s
    animate: pop
  - type: text
    text: "{{slot.relative_day}} {{slot.time}}"  # "HOY 18:00", "VIERNES 8 PM"
    size: 72
    from: 10s
  - type: logo
    corner: bottom_right
    from: 0s
outro:
  ident: short
```

### Rótulos de tiempo relativos
| Distancia al slot | Texto |
|---|---|
| < 5 min | "A CONTINUACIÓN" |
| mismo día | "HOY 18:00" |
| mañana | "MAÑANA 18:00" |
| < 7 días | "ESTE VIERNES 8 PM" |
| recurrente | "DE LUNES A VIERNES 15:00" |
| estreno | "ESTRENO · SÁBADO 20:00" |

### Voz en off (Final, opcional)
TTS local (Piper) con voz de locutor de continuidad: "Hoy, a las seis… ¡no te lo pierdas!". Se mezcla con *ducking* de la música.

## 9.3 Paquetes de identidad ("skins" de continuidad)

Cada canal/franja usa un **paquete** con colores, tipografías, música, idents y cortinillas:
- *Clásico* (azul celeste, burbujas — por defecto, coherente con la UI).
- *Acción juvenil* (negro + verde neón, cortes rápidos, glitch).
- *Matiné* (rojo telón, tipografía de cine).
- *Nocturno* (morado, estrellas, música calmada).
- *Temporada*: Halloween, Navidad (se activan por fecha).

Formato: carpeta con `pack.yaml` + assets; se pueden importar/exportar como `.dinopack` (zip).

## 9.4 Reglas de rotación

- No repetir la misma promo dos veces en 30 min.
- Máx. 1 promo de la misma serie por bloque de relleno.
- Peso de una promo de estreno: `w = base * (1 + 3 / max(1, días_restantes))`.
- Las promos nunca anuncian algo que ya se emitió o que fue cancelado (se invalidan al re-materializar).

## 9.5 Material incluido de serie

Para que la Beta funcione sin trabajo previo, se incluyen: 3 idents, 4 cortinillas, música libre de derechos (CC0) para 6 plantillas, carta de ajuste, cartones de fin/inicio de emisión y fuentes con licencia OFL.
