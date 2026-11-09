# 16 · Calidad, pruebas y operación

## 16.1 Estrategia de pruebas

| Nivel | Go (core) | Kotlin (Android) |
|---|---|---|
| Unitarias | `scheduler` con reloj simulado y tablas de casos (*golden files* de parrillas) | ViewModels con Turbine |
| Propiedades | `testing/quick` / `rapid`: para cualquier conjunto de reglas, el plan no tiene huecos ni solapes | — |
| Integración | `playout` contra mpv real con videos sintéticos de `ffmpeg -f lavfi testsrc` (con reloj quemado en la imagen para medir desfase) | Repositorios contra servidor falso (Ktor MockEngine) |
| E2E | Docker compose: core + mpv headless (`--vo=null`) + verificación de `play_log` | Compose UI Test + Paparazzi (capturas de pantalla de referencia) |
| Resistencia | Emisión acelerada: reloj simulado ×60 para correr 7 días en ~3 h | — |
| Usabilidad | — | Pruebas con 3-5 personas del wizard (cronometradas) |

### Casos críticos del scheduler
- Cambio de horario de verano a mitad de un slot.
- Película más larga que su slot con cada `overflow_policy`.
- Cancelar un slot de serie lineal → el siguiente recibe el capítulo correcto.
- Append mientras la serie está en repetición.
- Estreno que pisa dos reglas parcialmente.
- Hueco residual de 1.3 s → cartón dinámico de duración exacta.

## 16.2 CI/CD

- GitHub Actions: `go vet`, `staticcheck`, `golangci-lint`, pruebas con `-race`; Android: `ktlint`, `detekt`, pruebas unitarias, build `assembleRelease`.
- Matrices: Linux amd64/arm64, Windows, macOS.
- Releases con `goreleaser` (binarios + paquetes `.deb`, instalador Windows) y APK firmada.
- Versionado semántico: `0.x` = Beta, `1.0` = Final.

## 16.3 Operación

| Tema | Diseño |
|---|---|
| Logs | `slog` JSON con rotación; nivel configurable desde la app |
| Métricas | Endpoint `/metrics` (Prometheus) opcional: desfase, fallos, uso de CPU |
| Respaldos | Copia diaria de la BD (`VACUUM INTO`) con retención de 14 días; exportación JSON |
| Actualizaciones | Comprobación manual desde la app; el core se actualiza durante el fin de emisión |
| Configuración | `dinotv.yaml` (carpetas, puerto, zona horaria) + todo lo demás en BD |
| Arranque kiosco | systemd unit (Linux) / tarea programada (Windows) |

## 16.4 Definición de terminado (DoD)
- Código con pruebas, revisado y con CI verde.
- Endpoint documentado en OpenAPI.
- Pantalla con captura de referencia actualizada.
- Manual de uso actualizado si cambia el comportamiento visible.
