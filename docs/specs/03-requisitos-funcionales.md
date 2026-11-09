# 03 · Requisitos funcionales

Prioridad: **M** = Must (Beta), **S** = Should (Final), **C** = Could (posterior). Etapa: **B** = Beta, **F** = Final.

## 3.1 Biblioteca

| ID | Requisito | Prio | Etapa |
|---|---|---|---|
| RF-BIB-01 | Escanear carpetas configuradas y detectar archivos de video (mkv, mp4, avi, webm, ts). | M | B |
| RF-BIB-02 | Obtener duración exacta, pistas de audio y subtítulos con `ffprobe`. | M | B |
| RF-BIB-03 | Agrupar episodios por serie/temporada a partir de patrones de nombre (`S01E02`, `1x02`, `- 02`, `Ep 02`). | M | B |
| RF-BIB-04 | Detectar archivos nuevos en carpetas de series existentes y proponer *append*. | M | B |
| RF-BIB-05 | Metadatos manuales: título, sinopsis, póster, clasificación, color de marca. | M | B |
| RF-BIB-06 | Metadatos opcionales desde proveedores públicos (TMDB/AniList) si hay internet. | S | F |
| RF-BIB-07 | Marcar puntos de intro/créditos (manual o detección de audio) para cortes y promos. | S | F |

## 3.2 Series y películas

| ID | Requisito | Prio | Etapa |
|---|---|---|---|
| RF-SER-01 | Tipos de serie: **lineal** (orden estricto), **episódica** (aleatoria sin repetir), **mixta** (arcos lineales dentro de una serie episódica). | M | B |
| RF-SER-02 | Cursor por serie y por canal; editable manualmente ("volver al capítulo 12"). | M | B |
| RF-SER-03 | Política al terminar: repetir desde el inicio, pasar a la siguiente temporada, retirar de parrilla, sustituir por otra serie. | M | B |
| RF-SER-04 | Películas: duración, clasificación, ventana de emisión permitida (p. ej. solo después de las 21:00). | M | B |
| RF-SER-05 | Arcos/especiales: emitir un rango de episodios como maratón. | S | F |

## 3.3 Parrilla

| ID | Requisito | Prio | Etapa |
|---|---|---|---|
| RF-PAR-01 | Vista semanal (7 días × 24 h) por canal con zoom de 15 min. | M | B |
| RF-PAR-02 | Crear reglas recurrentes: días, hora inicio, duración del slot (30/60/90/120 min o libre), serie, episodios por slot. | M | B |
| RF-PAR-03 | Definir **cuántas horas** ocupa una serie en el día/semana (p. ej. "2 h los sábados"). | M | B |
| RF-PAR-04 | Franjas con nombre, color y cortinillas propias. | M | B |
| RF-PAR-05 | Copiar día/semana, plantillas de semana ("Semana normal", "Semana de vacaciones"). | S | B |
| RF-PAR-06 | Validar conflictos (solapes, slots sin contenido, películas más largas que el slot). | M | B |
| RF-PAR-07 | Vista previa de la lista materializada al segundo de un slot. | M | B |
| RF-PAR-08 | Programar con hasta 8 semanas de antelación. | S | F |

## 3.4 Reprogramaciones

| ID | Requisito | Prio | Etapa |
|---|---|---|---|
| RF-REP-01 | Mover, intercambiar, cancelar o sustituir un slot concreto sin alterar la regla. | M | B |
| RF-REP-02 | Reprogramación "en caliente" del programa siguiente mientras se emite el actual. | M | B |
| RF-REP-03 | Al cancelar una serie lineal, el cursor **no avanza** (no se pierden capítulos). | M | B |
| RF-REP-04 | Aviso automático en pantalla "Cambio de programación" (cartón + lower third). | S | B |
| RF-REP-05 | Historial de cambios y deshacer. | S | F |
| RF-REP-06 | Interrupción por "Especial": insertar un evento y desplazar o recortar lo siguiente según política. | S | F |

## 3.5 Continuidad y promos

| ID | Requisito | Prio | Etapa |
|---|---|---|---|
| RF-CON-01 | Relleno automático entre el fin del contenido y la siguiente hora exacta. | M | B |
| RF-CON-02 | Cortinillas de entrada/salida de corte configurables por franja. | M | B |
| RF-CON-03 | Cortes comerciales dentro de episodios (cada N minutos o en marcadores), con duración máx. | M | B |
| RF-CON-04 | Promos automáticas "A continuación", "Hoy más tarde", "Esta semana", "Estreno". | M | B (plantillas simples) / F (clips) |
| RF-CON-05 | Generador de promos con clips extraídos del episodio, título, día/hora y música. | S | F |
| RF-CON-06 | Anuncios del usuario (comerciales retro propios) con pesos y límites de frecuencia. | S | B |
| RF-CON-07 | Lower third "A continuación" X segundos antes del final del programa. | M | B |
| RF-CON-08 | Mosca del canal y reloj opcional en pantalla. | M | B |

## 3.6 Emisión

| ID | Requisito | Prio | Etapa |
|---|---|---|---|
| RF-EMI-01 | Reproducción continua 24/7 según la lista materializada. | M | B |
| RF-EMI-02 | Incorporación tardía al encender o cambiar de canal. | M | B |
| RF-EMI-03 | Cambio de canal (zapping) con efecto de estática/transición. | S | F |
| RF-EMI-04 | Selección de audio/subtítulos por serie (idioma predeterminado). | M | B |
| RF-EMI-05 | Fin de emisión nocturno con carta de ajuste y horario de reinicio. | S | B |
| RF-EMI-06 | Modo flexible (pausa permitida, desplaza la parrilla del día). | C | F |

## 3.7 App Android

| ID | Requisito | Prio | Etapa |
|---|---|---|---|
| RF-AND-01 | Descubrir el servidor en LAN (mDNS) y emparejar mediante código QR mostrado en la TV. | M | B |
| RF-AND-02 | Pantalla "Al aire": qué se emite, progreso, lo siguiente. | M | B |
| RF-AND-03 | Editor de parrilla con arrastrar y soltar. | M | B |
| RF-AND-04 | Wizard de alta de series/películas y *append*. | M | B |
| RF-AND-05 | Control remoto (canal, volumen, guía, subtítulos). | M | B |
| RF-AND-06 | Notificaciones: "Tu estreno empieza en 10 min", "Quedan 3 capítulos de X". | S | F |
| RF-AND-07 | Varios perfiles con permisos (administrador / solo ver). | S | F |

## 3.8 Interfaz de TV / Media Center

| ID | Requisito | Prio | Etapa |
|---|---|---|---|
| RF-TV-01 | Menú principal con mosaicos de canales y vista previa en vivo. | M | B |
| RF-TV-02 | Guía de programación (EPG) navegable con control/teclado. | M | B |
| RF-TV-03 | OSD de información (botón Info): título, episodio, sinopsis, progreso. | M | B |
| RF-TV-04 | Media Center en PC: misma interfaz + panel de administración completo. | S | F |
| RF-TV-05 | Soporte HDMI-CEC (control del televisor). | C | F |
