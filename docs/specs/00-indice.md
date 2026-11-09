# DinoTV — Planos del proyecto

> **DinoTV** es un canal de televisión personal que emula la experiencia de la TV abierta de los 90/2000: la programación corre a horas exactas, con cortes, cortinillas, promos de "lo que viene" y películas de estreno en días señalados. La parrilla se gestiona desde una app Android (Kotlin) y la emisión la realiza un núcleo en Go conectado al televisor o PC.

| Campo | Valor |
|---|---|
| Nombre en clave | DinoTV |
| Lenguajes | Go (núcleo, emisión, media center) · Kotlin (app Android, cliente Android TV) |
| Estética | Interfaz inspirada en consolas de sobremesa de 2012 (estilo "Wii U": blanco, celeste, mosaicos redondeados, sonidos suaves) |
| Etapas | **Beta** (MVP jugable) → **Final** (experiencia completa) |
| Versión del documento | 1.0 — 2026-11-09 |

## Índice

| # | Documento | Contenido |
|---|---|---|
| 01 | [Visión y alcance](01-vision-y-alcance.md) | Problema, objetivos, público, principios de producto |
| 02 | [Glosario](02-glosario.md) | Vocabulario de televisión y del sistema |
| 03 | [Requisitos funcionales](03-requisitos-funcionales.md) | Historias de usuario y requisitos numerados |
| 04 | [Requisitos no funcionales](04-requisitos-no-funcionales.md) | Precisión horaria, rendimiento, fiabilidad |
| 05 | [Arquitectura](05-arquitectura.md) | Componentes, despliegue, decisiones (ADR) |
| 06 | [Modelo de datos](06-modelo-de-datos.md) | Entidades, esquema SQL, invariantes |
| 07 | [Motor de programación](07-motor-de-programacion.md) | Reglas, materialización, relleno al segundo, reprogramaciones |
| 08 | [Motor de emisión (playout)](08-motor-de-emision.md) | Reproducción en vivo, incorporación tardía, transiciones |
| 09 | [Promos y continuidad](09-promos-y-continuidad.md) | Generador de promos estilo "bloque juvenil", cortinillas, cortes |
| 10 | [API](10-api.md) | REST + WebSocket, autenticación, eventos |
| 11 | [App Android](11-app-android.md) | Pantallas, navegación, arquitectura Kotlin |
| 12 | [Wizard de contenido](12-wizard-de-contenido.md) | Alta de series/películas y *append* de capítulos |
| 13 | [Interfaz TV y Media Center](13-interfaz-tv-y-media-center.md) | Lenguaje visual, pantallas, navegación con control |
| 14 | [Ideas de inmersión](14-ideas-de-inmersion.md) | Funciones extra para que se sienta como TV real |
| 15 | [Roadmap: Beta y Final](15-roadmap-beta-y-final.md) | Hitos, alcance por etapa, criterios de aceptación |
| 16 | [Calidad, pruebas y operación](16-calidad-pruebas-y-operacion.md) | Estrategia de pruebas, CI, respaldos, observabilidad |
| 17 | [Estructura de repositorios](17-estructura-de-repositorios.md) | Árbol de carpetas, convenciones, herramientas |

## Entregables relacionados

- `screens/` — maquetas HTML + CSS de las pantallas de TV, móvil y PC (abrir `screens/index.html`).
- `manual/DinoTV-Manual-de-Uso.pdf` — manual de usuario con previsualizaciones.
- `dist/DinoTV-planos.zip` — todo lo anterior empaquetado.
