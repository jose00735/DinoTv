# 15 · Roadmap: etapa Beta y etapa Final

## 15.1 Resumen

| Etapa | Objetivo | Duración estimada (1 dev) | Resultado visible |
|---|---|---|---|
| **Beta** | Un canal que emite 24/7 a hora exacta, gestionado desde Android | ~12 semanas | Encender la TV y "ver televisión" con tus series; editar la parrilla desde el móvil |
| **Final** | Experiencia completa e inmersiva, multicanal y multi-pantalla | ~16 semanas adicionales | Promos con clips, CRT, zapping, Android TV, Media Center completo, votaciones |

## 15.2 Etapa Beta — hitos

| Hito | Semanas | Entregables | Criterio de aceptación |
|---|---|---|---|
| **B0 · Cimientos** | 1 | Monorepo, CI, esquema SQLite, OpenAPI inicial, reloj inyectable | CI verde en Linux/Windows; migraciones aplican |
| **B1 · Biblioteca** | 2 | Scanner + ffprobe + parser de nombres + watcher | 500 archivos de prueba clasificados con ≥ 95 % de acierto |
| **B2 · Scheduler** | 3 | Reglas, materialización, cursores, relleno al segundo, overrides básicos (cancel/replace/move) | Suite de pruebas con reloj simulado: 0 huecos, 0 solapes, inicios exactos en 4 semanas simuladas |
| **B3 · Playout** | 2 | Control de mpv, join-in-progress, transiciones, fallos, fin de emisión | 72 h de emisión real con desfase máx. ≤ 1 s; recuperación tras matar mpv < 2 s |
| **B4 · Shell TV** | 2 | Menú principal, guía, OSD, mosca, lower third "A continuación", emparejamiento QR | Navegable solo con flechas/OK/Atrás a 60 fps |
| **B5 · App Android** | 3 (paralelo a B3-B4) | Emparejamiento, Al aire, Parrilla (drag & drop), Reprogramar, Wizard (nueva serie, película, append), Control remoto | Usuario nuevo programa una serie de 24 eps en ≤ 2 min (prueba con 3 personas) |
| **B6 · Continuidad básica** | 1 | Idents/cortinillas incluidos, promos de plantilla simple (póster + texto + música), anuncios del usuario, cartón de cambio de programación | Toda transición tiene continuidad; ninguna promo anuncia algo cancelado |
| **B7 · Estabilización** | 1 | Respaldos, exportar/importar, i18n ES/EN, instalador | Prueba de 7 días (OBJ-1) superada; manual actualizado |

### Alcance Beta (resumen)
✔ 1 canal al aire (varios configurables) · ✔ series lineales/episódicas/mixtas · ✔ películas y estrenos · ✔ reprogramaciones · ✔ relleno exacto · ✔ cortes comerciales · ✔ promos de plantilla · ✔ app Android completa de gestión · ✔ interfaz TV estilo Wii U · ✔ fin de emisión · ✔ modo 4:3 y normalización de volumen

✘ Fuera de Beta: clips automáticos en promos, voz en off, CRT, zapping multicanal, Android TV, Media Center admin, votaciones, grabaciones.

## 15.3 Etapa Final — hitos

| Hito | Semanas | Entregables |
|---|---|---|
| **F1 · PromoForge** | 3 | Clips por detección de escena/audio, plantillas YAML, "Anteriormente en…", créditos comprimidos, paquetes de identidad `.dinopack` |
| **F2 · Multicanal** | 2 | N canales simultáneos, zapping con estática, guía multicanal |
| **F3 · Media Center PC** | 3 | Panel de administración de escritorio, monitor de continuidad, edición con ratón |
| **F4 · Android TV** | 3 | Cliente Kotlin (Compose for TV + Media3), salida HLS del core, overlays sincronizados |
| **F5 · Inmersión** | 3 | Filtro CRT/VHS, locutor TTS, teletexto, ticker, semanas especiales, maratones |
| **F6 · Interacción** | 2 | "Tú decides", grabaciones VCR, recordatorios, perfiles/control parental, CEC |
| **F7 · Release 1.0** | 1 | Pulido, accesibilidad, documentación, instaladores firmados |

## 15.4 Criterios de "Beta lista"
1. Se puede instalar en un mini-PC y queda en modo kiosco al arrancar.
2. La app Android se empareja por QR en < 30 s.
3. Una semana completa de parrilla se crea solo desde el móvil.
4. 7 días de emisión continua sin intervención con desfase ≤ 1 s y 0 pantallas negras > 2 s.
5. *Append* de episodios mantiene el orden y el cursor.

## 15.5 Criterios de "Final lista (1.0)"
1. Todos los RF con prioridad M y S implementados.
2. Promos con clips generadas automáticamente para cada serie programada.
3. Dos receptores (TV + Android TV) muestran el mismo canal sincronizados (± 2 s).
4. Media Center permite toda la administración sin móvil.
5. Pruebas de usabilidad: SUS ≥ 80.

## 15.6 Riesgos

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Ventana transparente sobre mpv inestable en algunos SO | Alto | Respaldo con OSD nativo de mpv (ASS) para overlays |
| Decodificación por HW en RPi para HEVC 10-bit | Medio | Recomendar N100; transcodificación nocturna opcional |
| Nombres de archivo caóticos | Medio | Wizard con corrección manual por arrastre |
| Deriva de reloj sin internet | Bajo | RTC + aviso en app si desfase NTP > 2 s |
| Alcance excesivo de la Final | Alto | Hitos independientes y priorizados; F5/F6 recortables |
