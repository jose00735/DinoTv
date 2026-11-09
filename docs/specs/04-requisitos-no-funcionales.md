# 04 · Requisitos no funcionales

| ID | Categoría | Requisito | Medición |
|---|---|---|---|
| RNF-01 | Precisión | Cada programa empieza con desfase ≤ 1 s respecto a la hora programada. | Log de emisión vs. lista materializada, prueba de 7 días |
| RNF-02 | Precisión | Deriva acumulada 0: la emisión se re-sincroniza con el reloj del sistema en cada elemento. | Prueba de resistencia |
| RNF-03 | Reloj | Sincronización NTP recomendada; si no hay red se usa el RTC local. | Chequeo al iniciar |
| RNF-04 | Disponibilidad | Si un archivo falla, se emite relleno de emergencia en < 2 s (nunca pantalla negra > 2 s). | Prueba de fallo inyectado |
| RNF-05 | Recuperación | Tras un reinicio el canal vuelve al aire en < 10 s en el punto correcto. | Prueba de corte de energía |
| RNF-06 | Rendimiento | Materializar una semana de un canal en < 500 ms en Raspberry Pi 5. | Benchmark Go |
| RNF-07 | Rendimiento | Escaneo de 5 000 archivos en < 3 min (ffprobe concurrente). | Benchmark |
| RNF-08 | Recursos | Núcleo en reposo < 150 MB RAM; reproducción 1080p H.264/HEVC con decodificación por hardware. | Monitoreo |
| RNF-09 | Latencia app | Cambios de parrilla visibles en la TV en < 1 s (WebSocket). | Prueba E2E |
| RNF-10 | Seguridad | API solo en LAN por defecto; token de emparejamiento + TLS opcional; sin servicios en la nube obligatorios. | Revisión |
| RNF-11 | Privacidad | Ningún dato sale del dispositivo sin acción explícita (metadatos online opcionales). | Revisión |
| RNF-12 | Portabilidad | Núcleo en Linux (amd64/arm64), Windows 10+, macOS 13+. App Android 8.0+ (API 26). | CI multiplataforma |
| RNF-13 | Accesibilidad | Textos de TV ≥ 28 px a 1080p, contraste AA, navegable solo con flechas + OK + Atrás. | Checklist |
| RNF-14 | i18n | Español e inglés desde la Beta; formatos 12/24 h. | Revisión |
| RNF-15 | Mantenibilidad | Cobertura de pruebas ≥ 80 % en `scheduler` y `playout`. | CI |
| RNF-16 | Datos | Base de datos con respaldo automático diario y exportación JSON. | Prueba de restauración |
