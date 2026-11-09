# 01 · Visión y alcance

## 1. El problema

Las plataformas de streaming eliminaron algo que hacía especial a la televisión: **la cita**. Ya no existe "el anime de las 4 de la tarde", el "estreno del viernes por la noche" ni la emoción de ver una promo y contar los días. Tener toda la biblioteca disponible genera parálisis de elección y consumo en atracones.

## 2. La propuesta

DinoTV convierte una biblioteca local de series y películas en **uno o más canales de TV que emiten 24/7 según una parrilla**:

- Al encender el televisor, el canal **ya está en emisión**: si el episodio empezó a las 15:00 y enciendes a las 15:12, lo ves desde el minuto 12.
- Entre programas hay **cortes, cortinillas ("ya volvemos"), promos de lo que viene** y anuncios de estrenos, al estilo de los bloques juveniles de canales de cable de principios de los 2000.
- Desde el **móvil** se edita la parrilla semanal, se reprograma, se agregan series nuevas con un *wizard* y se anuncian películas para un día y hora concretos.

## 3. Objetivos

| ID | Objetivo | Métrica de éxito |
|---|---|---|
| OBJ-1 | Emitir a hora exacta | Desfase ≤ 1 s respecto a la parrilla durante 7 días continuos |
| OBJ-2 | Gestionar la parrilla sin PC | 100 % de las operaciones de programación posibles desde Android |
| OBJ-3 | Respetar la continuidad narrativa | Series lineales nunca saltan ni repiten capítulos sin orden explícito |
| OBJ-4 | Sensación de TV real | Toda transición entre programas tiene continuidad (promo, cortinilla o corte) |
| OBJ-5 | Agregar contenido en < 2 min | Wizard completo para una temporada de 24 episodios en ≤ 2 minutos |

## 4. Público

- **Persona A — "El nostálgico"**: 25-40 años, creció con TV de cable, tiene una colección de anime/caricaturas y quiere revivir la sensación.
- **Persona B — "La familia"**: padres que quieren horarios fijos de TV para sus hijos en lugar del autoplay infinito.
- **Persona C — "El curador"**: disfruta armando la programación como si fuera director de un canal.

## 5. Principios de producto

1. **La TV no espera.** El tiempo manda; el contenido se adapta al reloj, no al revés (con un *modo flexible* opcional).
2. **Cero huecos negros.** Ningún segundo sin imagen: todo hueco se rellena con continuidad.
3. **El móvil es el control maestro.** La TV solo muestra; las decisiones se toman en el teléfono.
4. **Estética amable.** Interfaz luminosa, redondeada, con sonidos suaves y animaciones elásticas.
5. **Local primero.** Funciona sin internet; los datos son del usuario.

## 6. Dentro del alcance

- Núcleo de emisión y programación (Go) para Linux/Windows/macOS y mini-PC (Raspberry Pi 5 / N100).
- App Android de control (Kotlin + Jetpack Compose).
- Interfaz de TV y Media Center para PC con estética tipo Wii U.
- Generación automática de promos y relleno de continuidad.
- Cliente Android TV (etapa Final).

## 7. Fuera del alcance

- Descarga o distribución de contenido con derechos de autor: DinoTV solo reproduce **archivos que el usuario ya posee**.
- Streaming público a internet / multiusuario remoto masivo.
- Edición de video avanzada (más allá de recortes para promos).
- App iOS (posible en el futuro vía Kotlin Multiplatform).
