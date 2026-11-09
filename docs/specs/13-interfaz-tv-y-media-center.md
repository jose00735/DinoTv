# 13 · Interfaz de TV y Media Center

## 13.1 Lenguaje visual "estilo Wii U"

Inspiración: menús de consolas de sobremesa de 2012 — sensación limpia, luminosa y amigable. Se replica el **estilo**, no logotipos ni recursos de ninguna marca.

| Elemento | Especificación |
|---|---|
| Fondo | Gradiente blanco → gris perla (`#F4F7FA` → `#E3E9EF`) con rejilla fina de líneas o burbujas difusas animadas lentamente |
| Mosaicos | Esquinas 24 px, borde blanco 4 px, sombra `0 6px 18px rgba(0,40,80,.15)`; al enfocar: escala 1.06 con rebote y halo celeste |
| Barras | Barra superior translúcida (reloj, canal, notificaciones); barra inferior con "píldoras" de acciones y botones de control (Ⓐ OK, Ⓑ Atrás, Ⓧ Info) |
| Color | Celeste `#29B6F6` primario, turquesa `#00C2A8`, ámbar `#FFB300`, texto `#3A4A5A` |
| Tipografía | Redondeada (M PLUS Rounded 1c / Nunito), 28-64 px a 1080p |
| Sonido | Clic suave al mover foco, "burbuja" al confirmar, música ambiental tipo *bossa* en menús (desactivable) |
| Movimiento | Transiciones 220-320 ms con *ease-out-back*; nada brusco |

## 13.2 Pantallas de TV

| Pantalla | Descripción | Maqueta |
|---|---|---|
| **Menú principal (Plaza)** | Mosaicos de canales con vista previa en vivo del programa actual y progreso; mosaicos secundarios: Guía, Estrenos, Biblioteca, Ajustes; tarjeta "Emparejar móvil" con QR | `screens/tv-menu.html` |
| **Guía (EPG)** | Rejilla canales × tiempo con línea roja de "ahora", panel de detalle con póster y sinopsis; insignias ESTRENO/REPETICIÓN | `screens/tv-guia.html` |
| **En vivo + OSD** | Video a pantalla completa con mosca, lower third "A continuación" y, al pulsar Info, panel inferior con progreso | `screens/tv-en-vivo.html` |
| **Promo / Próximamente** | Ejemplo de promo de estreno renderizada | `screens/tv-promo.html` |
| **Fin de emisión** | Carta de ajuste con reloj y hora de regreso | `screens/tv-carta-ajuste.html` |
| Emparejamiento | QR grande + código de 6 dígitos | (en menú) |

### Navegación con control
- Flechas: mover foco · OK: abrir · Atrás: volver al canal en vivo · **Guía**: EPG · **Info**: OSD · CH+/CH-: zapping.
- Con el canal en vivo, cualquier tecla de flecha muestra el mini-OSD; Menú abre la Plaza **sin detener la emisión** (el video queda en el mosaico del canal: "la TV sigue en marcha").
- Compatibles: teclado, control de TV vía HDMI-CEC (Final), gamepads (SDL), app móvil.

## 13.3 Media Center (PC)

Misma aplicación `dinotv-shell` en modo **Media Center**:
- Ventana con panel lateral de administración (escritorio): Parrilla en vista amplia (semana completa visible), Biblioteca, Wizard, Continuidad, Registro de emisión, Ajustes.
- Vista "monitor de continuidad" (como sala de control): emisión en vivo, siguiente elemento, cola de relleno, desfase en ms, alertas.
- Edición con ratón: arrastrar slots, redimensionar para cambiar duración, clic derecho → Reprogramar.
- Maqueta: `screens/pc-media-center.html`.

## 13.4 Paridad entre interfaces

| Función | TV | Android | PC Media Center |
|---|---|---|---|
| Ver canal en vivo | ✔ | Vista previa (Final) | ✔ |
| Guía | ✔ | ✔ | ✔ |
| Editar parrilla | — (solo ver) | ✔ | ✔ |
| Wizard | — | ✔ | ✔ |
| Control remoto | — | ✔ | ✔ (teclado) |
| Monitor de continuidad | — | Básico | ✔ completo |

## 13.5 Rendimiento
- 60 fps en navegación a 1080p en N100 / RPi 5; animaciones CSS por *transform/opacity* únicamente.
- Las vistas previas en vivo de los mosaicos usan miniaturas actualizadas cada 10 s (no video) en hardware modesto; video real en PC.
