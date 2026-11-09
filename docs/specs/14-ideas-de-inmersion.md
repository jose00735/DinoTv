# 14 · Ideas de inmersión

Funciones propuestas para que DinoTV se sienta como un canal de verdad. Cada una indica etapa sugerida.

## 14.1 Imagen y sonido de época
| Idea | Descripción | Etapa |
|---|---|---|
| **Filtro CRT** | Shader GLSL en mpv: líneas de barrido, curvatura, viñeta, *bloom*; intensidad por franja (más fuerte de madrugada). | F |
| **Modo 4:3** | Series antiguas en 4:3 con bandas laterales temáticas del canal en lugar de negro. | B |
| **Zapping con estática** | Al cambiar de canal: 300 ms de nieve + sonido de estática + número del canal en verde estilo OSD antiguo. | F |
| **Calidad VHS** | Filtro opcional de ruido de cinta y temblor para "Clásicos de los 80". | F |
| **Normalización de volumen** | `loudnorm` a -23 LUFS para que comerciales y series suenen parejo (como exige la TV real). | B |

## 14.2 Continuidad y personalidad del canal
| Idea | Descripción | Etapa |
|---|---|---|
| **Mascota del canal** | "Dino", un dinosaurio que aparece en idents, cortinillas y cartones, con animaciones por temporada. | B |
| **Locutor de continuidad** | Voz TTS local que anuncia lo siguiente ("Y ahora… ¡Dino Rangers!"). | F |
| **Franjas temáticas** | "Despertar Anime" (7:00), "Matiné" (sábado 15:00), "Noche de terror" (viernes 23:00) con su propio paquete visual. | B |
| **Maratones** | Arcos completos o temporadas en un día festivo, con contador "Episodio 5 de 12". | F |
| **"Lo mejor de la semana"** | Domingo: repetición automática de los episodios más vistos (según `play_log` con TV encendida). | F |
| **Semanas especiales** | Halloween / Navidad / aniversario del canal: paquete de identidad y parrilla plantilla que se activan solas. | F |
| **Fin e inicio de emisión** | Carta de ajuste, himno o cortinilla de buenas noches; apertura con "Buenos días". | B |
| **Teletexto** | Página tipo teletexto (botón TXT) con guía, trivia, "cartas de los espectadores" (notas que escribes en la app). | F |
| **Ticker** | Barra inferior desplazable con avisos ("Este sábado: ESTRENO") y clima/hora si hay internet. | F |
| **Clasificación por edades** | Aviso "+13" al inicio y franja de protección infantil que impide programar +13 antes de las 20:00. | B |

## 14.3 Interacción
| Idea | Descripción | Etapa |
|---|---|---|
| **"Tú decides"** | Votación desde el móvil (de la familia) entre 2-3 opciones para el slot del viernes; el ganador se anuncia en pantalla. | F |
| **Grabar (VCR)** | Botón REC en la guía: "grabar" un slot → queda disponible en "Mis grabaciones" con las promos y cortes de ese día, como una cinta. | F |
| **Recordatorios** | Desde la guía: "Avisarme" → notificación en móvil y aviso en TV 1 min antes. | F |
| **Perfiles y control parental** | Canal infantil bloqueado en horario; PIN para canales +18. | F |
| **Segunda pantalla** | El móvil muestra trivia/sinopsis sincronizada con lo que se emite (como el mando con pantalla de la consola). | F |
| **Temporizador de apagado** | "Apagar al terminar este programa" (vía CEC). | F |
| **Encendido programado** | La TV (CEC) se enciende para el estreno del sábado 20:00. | F |

## 14.4 Realismo de parrilla
| Idea | Descripción | Etapa |
|---|---|---|
| **Repeticiones nocturnas** | Los episodios de la tarde se repiten de madrugada automáticamente. | B |
| **"Anteriormente en…"** | Para series lineales, clip de 20 s con escenas del episodio anterior antes del capítulo (generado por PromoForge). | F |
| **Créditos comprimidos** | Al final, créditos en pantalla dividida con la promo del siguiente programa (como hacían los canales de cable). | F |
| **Programa de relleno "Dino Cortos"** | Bloque de cortos animados de 1-5 min para huecos de 5-15 min. | B |
| **Estadísticas del canal** | "Rating" casero: cuánto tiempo estuvo encendida la TV en cada programa. | F |
| **Generador de parrilla automática** | "Rellena mi semana": propone una parrilla equilibrada según tus series y gustos. | F |
