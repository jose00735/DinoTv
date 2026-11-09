# 17 · Estructura de repositorios

Monorepo `DinoTv`:

```
DinoTv/
├── README.md
├── docs/
│   ├── specs/                 ← estos documentos
│   └── adr/                   ← decisiones de arquitectura adicionales
├── api/
│   └── openapi.yaml           ← contrato único (genera Go y Kotlin)
├── core/                      ← Go: dinotv-core
│   ├── cmd/dinotv/
│   ├── internal/{clock,library,scheduler,playout,promoforge,store,api,events}/
│   ├── migrations/
│   └── go.mod
├── shell/                     ← Go + Wails: interfaz TV / Media Center
│   ├── app.go
│   └── frontend/              ← HTML/CSS/TS (parte de screens/ como base)
├── android/                   ← Kotlin: app de control
│   ├── app/
│   ├── core/{design,network,data}/
│   └── feature/{pairing,onair,grid,wizard,library,events,continuity,remote,settings}/
├── androidtv/                 ← Kotlin: receptor (Final)
├── assets/
│   ├── packs/clasico/         ← idents, cortinillas, música CC0, fuentes OFL
│   └── shaders/crt.glsl
├── screens/                   ← maquetas HTML + CSS
├── manual/                    ← manual de uso (fuente HTML + PDF)
└── tools/                     ← scripts (generar videos de prueba, capturas)
```

## Convenciones
- Commits: Conventional Commits (`feat(scheduler): ...`).
- Ramas: `main` estable, `feat/*`, releases con etiquetas `v0.x.y`.
- Go: `gofmt`, errores envueltos con `%w`, sin estado global (salvo `main`).
- Kotlin: estilo oficial, `ktlint`, un ViewModel por pantalla, estado inmutable.
- Textos de UI en archivos de recursos (es/en) desde el día 1.

## Herramientas
| Herramienta | Uso |
|---|---|
| Go 1.23+, Wails v2 | Core y shell |
| mpv ≥ 0.37, ffmpeg ≥ 6 | Reproducción y promos |
| Android Studio, JDK 17, Gradle (Kotlin DSL + version catalog) | App Android |
| oapi-codegen, openapi-generator | Código desde el contrato |
| goreleaser, GitHub Actions | Releases y CI |
