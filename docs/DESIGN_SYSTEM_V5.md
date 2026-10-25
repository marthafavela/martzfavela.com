# Design system — Landing V5

**Proyecto:** MartzFavela.com
**Fase:** MF-003 — auditoría de solo lectura
**Fuente de verdad visual:** `reference/landing-v5/`
**Estado:** documentación; no se modificó WordPress ni la página de laboratorio 153.

## 1. Alcance y fuentes examinadas

Las tres páginas son variantes completas y paralelas, no una única plantilla de idioma: `index.html` (ES), `en.html` (EN) y `fr.html` (FR). Cada una carga `styles.css` y `app.js`, contiene la misma estructura semántica y los mismos elementos decorativos, tarjetas y diálogos, con texto, metadatos, atributos accesibles y enlaces localizados.

| Fuente | Tamaño observado | Papel |
|---|---:|---|
| [`index.html`](../reference/landing-v5/index.html) | 37,637 bytes | Página española y fallback principal. |
| [`en.html`](../reference/landing-v5/en.html) | 36,872 bytes | Página inglesa independiente. |
| [`fr.html`](../reference/landing-v5/fr.html) | 38,741 bytes | Página francesa independiente. |
| [`styles.css`](../reference/landing-v5/styles.css) | 39,308 bytes | Tokens, layout, estilos de componentes, breakpoints y animaciones. |
| [`app.js`](../reference/landing-v5/app.js) | 40,630 bytes | Navegación, animación ligada al scroll, casos accesibles e idioma. |
| [`translations.json`](../reference/landing-v5/translations.json) | 34,117 bytes | 167 claves, cada una con `es`, `en` y `fr`. |

Activos locales inspeccionados:

| Activo | Dimensiones / formato | Uso observado y consideraciones |
|---|---|---|
| [`logo-color.svg`](../reference/landing-v5/assets/logo-color.svg) | SVG, `viewBox` 1366 × 434.67 | Logo para fondos claros; contiene sus propios colores. |
| [`logo-white.svg`](../reference/landing-v5/assets/logo-white.svg) | SVG, `viewBox` 280 × 70 | Logo blanco para fondos oscuros. |
| [`isologo-ornament.svg`](../reference/landing-v5/assets/isologo-ornament.svg) | SVG, `viewBox` 100 × 70 | Isotipo ornamental repetido en varios puntos de la composición. |
| [`isologo.png`](../reference/landing-v5/assets/isologo.png) | PNG RGBA, 64 × 64 | Está en el paquete de activos; no aparece como `<img>` en las tres páginas examinadas. |
| [`martha-favela.jpg`](../reference/landing-v5/assets/martha-favela.jpg) | JPEG, 1086 × 1448 | Retrato vertical, con texto alternativo localizado. |

Las páginas también incluyen 11 SVG en línea cada una (entre ellos `hero-wave` y `contact-wave`). Los SVG complejos no son sustituibles por un widget HTML monolítico: deben convertirse en activos independientes o conservarse como gráficos controlados y acotados.

## 2. Paleta y variables CSS

El diseño fuente no usa la paleta de prueba anterior (`#15111E` / violeta). La landing V5 emplea los siguientes valores; prevalecen los originales frente a cualquier propuesta de rediseño.

| Variable | Valor original | Función visual |
|---|---|---|
| `--ink` | `#100921` | Tinta oscura y texto sobre superficies claras. |
| `--deep` | `#110720` | Fondo oscuro profundo. |
| `--purple` | `#8b34fa` | Violeta brillante / energía. |
| `--violet` | `#6320e8` | Violeta de interacción y superficies. |
| `--brand` | `#6c30e9` | Color principal de marca y del isologo. |
| `--brand-magenta` / `--magenta` | `#f43bbb` | Acento magenta. |
| `--cyan` | `#06d9ff` | Acento cian, alternado con violeta. |
| `--white` | `#fff` | Texto sobre fondos oscuros y superficies. |
| `--muted` | `#6f687c` | Texto secundario. |
| `--stroke` | `rgba(255,255,255,.14)` | Contornos translúcidos. |
| `--ambient` | `108,48,233` | Canales RGB interpolados por JavaScript para la iluminación de escena. |
| `--scene-p` | `.5` | Progreso normalizado de la escena ligada al scroll. |

También hay colores literales en gradientes y componentes (por ejemplo `#431084`, `#16092c`, `#10061e`, `#ddd4ed`, `#a455ff` y `#00d6ff`). No se deben reemplazar por una paleta aproximada: catalogar y mapear esos valores antes de consolidar tokens.

## 3. Tipografía y jerarquía

| Uso | Familia / tratamiento | Valores observados |
|---|---|---|
| Texto general | `DM Sans`, `sans-serif` | Cuerpo alrededor de 17 px en Hero (15–16 px en ciertos breakpoints); peso base 400. |
| Titulares display | `Space Grotesk`, `sans-serif` | H1 Hero en mayúsculas, peso 700, `clamp(49px, 7vw, 102px)`; H2 alrededor de `clamp(42px, 5.5vw, 73px)`. |
| Énfasis / navegación | Familias de marca anteriores | Pesos 600, 700 y 800; navegación alrededor de 12 px y peso 700. |
| Fallbacks | Sistema, Arial y `inherit` | Hay reglas de fallback específicas, especialmente en móvil; no asumir que son las fuentes de marca. |

**Riesgo de carga:** los archivos inspeccionados enlazan `styles.css` y `app.js`, pero no muestran `@font-face` ni una hoja externa de fuentes. Antes de comparar fidelidad hay que confirmar cómo se cargan DM Sans y Space Grotesk en el navegador objetivo; si no están disponibles, el fallback alterará el ancho de titulares, saltos de línea y altura.

## 4. Layout, espaciado y responsive

- Contenedor base: `width: min(1210px, calc(100% - 68px))`, centrado.
- Navegación flotante: ancho máximo 1140 px, a 20 px del borde superior, alto base 68 px, fondo blanco, radio 18 px y sombra amplia.
- Hero: `min-height: 780px`, contenido centrado, `padding: 185px 0 170px`, `text-align: center`; H1 con ancho máximo 1110 px. El Hero no es simplemente un bloque oscuro de 85 vh.
- Párrafo Hero: ancho máximo 550 px, 17 px y `line-height: 1.55`.
- Secciones generales: padding vertical de 112 px; la escena de transformación cambia dinámicamente a 150–170 vh cuando se activa el modo de scroll.
- Hero CTAs: fila centrada, separación 12 px y wrap; los enlaces llevan a WhatsApp.
- Grid de servicios: 7 columnas en una variante base del CSS, 4, 3, 2 y finalmente 1 según la cascada y el viewport. Verificar el orden efectivo de reglas y media queries, no inferir el layout de un único selector aislado.
- Grid de método: 4 columnas en escritorio y 2 en móvil.
- Experiencia: 3 columnas, separadores verticales; en móvil pasa a una columna y separadores horizontales.
- Cursos: mazo de tarjetas superpuestas con perspectiva; en pantallas pequeñas se transforma en lista.

Breakpoints/condiciones encontrados en `styles.css`:

| Condición | Uso a considerar |
|---|---|
| `max-width: 1100px` y `max-width: 1100px` con `min-width: 901px` | Ajustes de navegación y elementos laterales. |
| `max-width: 1050px` | Cambios de grid y tamaño de H1. |
| `max-width: 900px` | Navegación móvil, botón de menú y menú alternativo. |
| `max-width: 760px` | Cambios principales a layout móvil, espaciados y grids. |
| `max-width: 410px`, `380px` y `370px` | Correcciones para móviles estrechos, botones y mazo de cursos. |
| `prefers-reduced-motion: reduce` / `no-preference` | Desactivación o habilitación de movimiento. |

Las reglas aparecen en una hoja extensa con overrides sucesivos. Antes de migrar, la referencia visual debe verificarse en los tamaños reales, sobre todo entre 901–1100 px y por debajo de 410 px. Elementor MCP permite variantes por breakpoint usando su sintaxis de breakpoint, pero no se debe asumir que cada umbral personalizado del CSS fuente coincide con los breakpoints configurados en el kit.

## 5. Componentes visuales y tratamiento

- **Superficies claras y oscuras:** navegación clara y flotante; Hero con fondo violeta profundo y secciones de contenido claras/oscuras alternadas.
- **Hero:** gradiente radial violeta, viñeta oscura, SVG de onda brillante en el borde inferior, eyebrow, titular en mayúsculas, párrafo y dos CTA.
- **Transformación:** escena aislada con gradientes radial/lineal, nodos de estrategia/tecnología/marketing/IA, conexiones SVG, marca de agua y color ambiental dinámico.
- **Servicios:** tarjetas translúcidas, borde fino, radios de 10 px, brillo interior y blur (`backdrop-filter`), iconografía/acento cian y violeta, elevación en hover.
- **Cursos:** tarjetas verticales superpuestas, perspectiva 3D, gradientes y sombras; variante en lista en móvil.
- **Experiencia:** retrato vertical y tres bloques de contenido con líneas divisorias.
- **Decoración:** pseudo-elementos, `drop-shadow`, gradientes radiales, `mix-blend-mode`, ondas SVG y ornamentos.
- **Movimiento CSS:** diez keyframes, incluidos `float`, `bob`, `dash`, `waveDraw`, `waveRibbon`, `waveSway`, `orbitSpin`, `prismDrift`, `ribbonThread` y `gentleShine`.
- **Glass / sombras:** `backdrop-filter` de 12–24 px, algunos hasta 18 px con saturación; sombras múltiples con capas interiores, resplandor y profundidad. Si el blur no es soportado, el color de fondo debe conservar legibilidad.
- **Tarjetas y botones:** hover con transición, elevación y cambio de borde/fondo; respetar contraste y foco visible, además del hover.

## 6. Navegación, interacción y accesibilidad

- Header con logo, cuatro enlaces a secciones (`servicios`, `metodo`, `cursos`, `sobre-mi`), selector ES/EN/FR y CTA de WhatsApp.
- En escritorio se muestra la navegación en línea. Desde 900 px aparece el botón hamburguesa con `aria-expanded` y `aria-controls`; hay un menú `<details>` de fallback para navegación sin JavaScript.
- En móvil también existe un acceso flotante a WhatsApp; footer con navegación, marca y derechos.
- `skip-link` al contenido principal, landmarks (`header`, `main`, `footer`), enlaces con etiquetas accesibles, `aria-current` en el idioma seleccionado y atributos `aria-label`.
- Siete tarjetas `<a>` de servicio enlazan siete secciones `role="dialog"`/`.case-modal`. Sin JavaScript, los hash links siguen permitiendo llegar a los casos.
- El realce de modal con JavaScript bloquea el fondo mediante `inert`, fija `aria-modal`, mueve el foco al cierre, lo devuelve al trigger, cierra con Escape o clic en backdrop y mantiene el foco dentro con Tab/Shift+Tab.
- No hay formularios en las páginas fuente; las conversiones llevan directamente a WhatsApp. LinkedIn abre en pestaña nueva con `rel="noopener noreferrer"`.
- Evitar convertir enlaces de tarjeta en botones sin conservar navegación por teclado, `href`, hash y degradación sin JavaScript.

## 7. Animaciones e interacciones JavaScript

`app.js` incluye comportamientos funcionales, no solo decoración:

1. Barra de progreso de lectura calculada con scroll y actualizada mediante `requestAnimationFrame`.
2. Parallax ligero del contenido Hero solo en escritorio, con límite de scroll.
3. Resplandor que sigue el puntero en dispositivos con puntero fino; se oculta si no aplica.
4. Apertura/cierre del menú, cierre al navegar, clic fuera y Escape, sincronizando `aria-expanded`.
5. Escena de transformación: progreso de scroll, interpolación entre violeta/magenta/cian, actualización de variables CSS y entrada de elementos.
6. Aparición de elementos con `IntersectionObserver` (threshold 0.12), Web Animations API y retrasos escalonados; se reduce el movimiento si `prefers-reduced-motion` está activo.
7. Inclinación 3D de tarjetas de servicio en dispositivos hover/puntero fino, restaurada al salir.
8. Cierre del menú offline (`<details>`) al elegir un enlace y retorno del foco en Escape.
9. Modal accesible descrito arriba.
10. Mejora multilingüe del HTML: traduce nodos de texto y atributos, actualiza `lang`, `title`, `description`, Open Graph, `aria-current` y el texto de WhatsApp por idioma/caso.
11. Actualización del año en footer.

No se encontró `MutationObserver`; el script presupone que el HTML existe al inicializarse. Elementor puede renderizar o volver a renderizar contenido después, lo que hace insuficiente una inicialización única en `DOMContentLoaded`.

## 8. Sistema multilingüe y SEO actual

- Las páginas ES, EN y FR tienen `lang` propio, títulos/descripciones y Open Graph localizados.
- Las URLs fuente son `index.html`, `en.html` y `fr.html`; cada una contiene navegación a las tres variantes.
- `translations.json` tiene 167 claves completas en `es`, `en` y `fr`; el JS aplica traducciones in situ y localiza los enlaces de WhatsApp.
- El contenido no depende exclusivamente del JS: los HTML de cada idioma ya contienen texto localizado.
- No se encontraron declaraciones `rel=canonical` ni `hreflang` en los heads inspeccionados; deben definirse en WordPress/SEO.
- Para staging/propuesta, mantener URLs independientes por idioma (por ejemplo `/`, `/en/`, `/fr/`), enlaces de idioma entre traducciones, títulos/descripciones traducidos, canonical por idioma y hreflang recíproco. La elección entre Polylang y WPML queda pendiente de evaluación; esta auditoría no instala ni configura plugins.

## 9. Activos de WordPress staging y diferencias

La consulta de lectura MCP de la biblioteca devolvió cuatro activos: tres SVG y un WebP. No enumeró el retrato `martha-favela.jpg` ni `isologo-ornament.svg` de la fuente. Los logos existentes parecen candidatos de marca, pero se debe verificar visualmente su equivalencia antes de reutilizarlos; no afirmar que son idénticos sólo por sus dimensiones o título.

El servidor MCP listado ofrece lectura de biblioteca (`elementor-list-assets`) pero no se identificó una herramienta de carga de archivos locales. La carga de los activos ausentes requiere un procedimiento aprobado fuera de la fase MF-003. No se cargó ni cambió ningún medio.

## 10. Límite entre sistema fuente y kit Elementor

La lectura del kit staging devolvió **cero clases globales** y ocho variables globales: `Brand_Primary #7427F5`, `Cyan_Crecimiento #15DDF4`, `Brand_Accent #A855F7`, `Backgroud_Midnight #080625`, `Background_Violet #180843`, `Text_primary #211B3A`, `Text_onDark #D8D5E8`, `Neutral_White #FFFFFF`. Difieren de varios colores de V5. No deben actualizarse ni suponerse intercambiables: usar estilos locales aislados y, en una fase de implementación aprobada, decidir qué estilos por elemento se expresan con Atomic y qué detalles de fidelidad se conservan en el CSS de página.

## 11. Referencias

- Fuente completa: [`reference/landing-v5/`](../reference/landing-v5/)
- Arquitectura del proyecto: [`ARCHITECTURE.md`](./ARCHITECTURE.md)
- MCP de Elementor: matriz y límites en [`ELEMENTOR_MIGRATION_MATRIX.md`](./ELEMENTOR_MIGRATION_MATRIX.md)
