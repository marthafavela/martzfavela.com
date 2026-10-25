# Plan de implementación Frontend — Landing V5

**Proyecto:** MartzFavela.com
**Fase:** MF-003 — análisis y documentación
**Entorno previsto:** staging `staging.martzfavela.com`
**Página de laboratorio conocida:** ID 153; no se modificó durante esta auditoría.
**Producción:** fuera de alcance.

## 1. Objetivo y principios

Migrar la landing V5 sin rediseñarla, de HTML/CSS/JS estático a páginas WordPress/Elementor Atomic V4 donde la estructura y el contenido principal sean editables en el editor visual, mientras los efectos específicos continúan siendo pequeños, locales, accesibles y mantenidos en Git.

- Tratar `reference/landing-v5/` como fuente de verdad del diseño y los textos.
- Evitar copiar la hoja completa de forma indiscriminada o alterar el kit, estilos predeterminados, header/footer global o plantillas.
- Construir widgets V4 para contenido, layout y controles. Conservar CSS/JS sólo donde sea necesario para reproducir la fuente o cubrir funciones que el schema nativo no resuelve.
- Mantener los tres idiomas, links de conversión y fallback funcional sin JavaScript.
- Hacer toda validación funcional en staging; no publicar ni cambiar URLs durante MF-003.

## 2. Arquitectura propuesta bajo control de versiones

El repositorio actualmente contiene la documentación y los archivos de referencia; todavía no contiene un tema/plugin frontend, manifest de dependencias ni pipeline de despliegue. No asumir que un archivo PHP o enqueue ya existe.

Cuando se apruebe la implementación, elegir con el equipo el destino dentro del tema activo propio. Mantener por separado:

```text
theme (destino por confirmar)
└── assets/
    ├── css/landing-v5.css
    ├── js/landing-v5.js
    └── images/                 # sólo activos aprobados/versionables
```

El árbol exacto se definirá después de confirmar el nombre/ubicación del tema existente. El código fuente permanece versionado en GitHub; un despliegue aprobado publica únicamente assets de landing y el código de enqueue requerido, nunca credenciales ni datos de clientes. No se propone instalar plugins ni crear PHP en esta fase.

### Aislamiento de estilos

- Prefijar las clases y variables de implementación con una raíz propia (por ejemplo `.mf-v5` y `--mfv5-*`); no escribir reglas desnudas `body`, `h1`, `button`, `a`, `:root`, `section` o `p` en el ámbito global.
- Encolar `landing-v5.css` sólo cuando se renderice la página autorizada (por condición de página/ID verificado en WordPress). Un selector `body.page-id-{ID} .mf-v5` sirve como segunda barrera; sustituir el ID sólo después de verificar el documento de destino.
- Aplicar los estilos sencillos de cada heading, párrafo, grid y CTA al elemento Atomic correspondiente. Mantener en la hoja local únicamente gradientes complejos, glass, pseudo-elementos, ondas, arte SVG y estados/transformaciones especiales.
- No añadir ni actualizar variables, clases o estilos predeterminados del kit global. La paleta V5 difiere de las ocho variables globales vistas en staging.
- En la primera fase de MF-004 no buscar simular exactamente cada sombra/blur en un CSS global. Usar captura visual y registrar desviaciones explícitas por sección.

### Aislamiento e inicialización de JavaScript

- Encolar `landing-v5.js` sólo para la misma página; no adjuntar listeners globales a toda página WordPress.
- Inicializar en frontend cuando el DOM del fragmento esté listo y enlazar con el ciclo de Elementor (hook `elementor/frontend/init` y hook `elementor/frontend/element_ready/...` apropiado una vez confirmado el tipo de wrapper utilizado).
- Implementar `init(root)` idempotente, limitado a `.mf-v5`; marcar nodos ya inicializados o usar un registro `WeakSet`. Repetir init no debe agregar listeners duplicados ni reaplicar animaciones.
- Si la navegación del editor o widgets vuelve a renderizar el fragmento, reconectar sólo el subtree nuevo. Usar `MutationObserver` como último mecanismo acotado, filtrando nodos propios y evitando observar/escribir el mismo atributo en bucle; desconectarlo cuando el root se retire.
- Mantener funciones puras para traducir textos/crear URLs. No sustituir texto traducido de usuario o widgets fuera del root, ni cambiar idioma sólo por un efecto colateral del editor.
- Respetar `prefers-reduced-motion` en CSS y JS. Para movimiento simple probar primero interacciones nativas Atomic; mantener JS para la interpolación continua de la escena, tilt preciso o focus trap si una función equivalente accesible no existe.
- Separar helpers por función (nav, escena, tarjetas, diálogos, idioma) sin convertir todo `app.js` en un único inicializador opaco. Cada comportamiento debe tener una prueba manual repetible.

## 3. Multidioma y SEO

**Dirección recomendada:** un documento WordPress por idioma con traducciones relacionadas y permalinks independientes: español `/`, inglés `/en/`, francés `/fr/` (las rutas finales se decidirán en una fase aparte; no se modifican aquí).

- Evaluar Polylang y WPML —uno, no ambos— según compatibilidad con Elementor V4, edición de traducciones, licencia/hosting, SEO, sitemap, redirects y flujo editorial. No instalar ni configurar ninguno ahora.
- Crear cada traducción a partir del contenido fuente en su idioma, no depender de que un script cambie una misma URL para representar tres contenidos SEO.
- Configurar traducción revisada de título, meta description, Open Graph, slug, texto alternativo de medios y navegación; `lang` del documento corresponde al idioma servido.
- Establecer canonical autorreferente por idioma y enlaces hreflang recíprocos para `es`, `en` y `fr` (más `x-default` sólo si se acuerda una ruta neutral). No canonicalizar las tres a una sola URL.
- Mantener selector de idioma enlazado al equivalente traducido, con fallback claro si una traducción falta. No almacenar estado que haga que Google o lectores sin JS reciban otro idioma.
- Mantener `translations.json` como fuente de cotejo/QA durante la migración. Reconciliar las 167 claves con el contenido publicado y avisar de claves huérfanas o nuevos textos; no generar traducciones automáticas en producción.
- Verificar idioma en HTML, metadatos, `aria-current`, CTA de WhatsApp con texto codificado y enlaces de cada idioma.

La fuente ya ofrece HTML localizado independiente y traducción JS progresiva. La arquitectura propuesta conserva el contenido localizado como base, trasladando a cada página los metadatos de idioma y usando JS sólo para mejorar la interacción, no para ocultar páginas o cambiar SEO.

## 4. Riesgos técnicos y mitigaciones

| Riesgo | Consecuencia | Mitigación |
|---|---|---|
| Hojas originales contienen estilos globales y overrides sucesivos | Colisión con Hello Elementor, kit Atomic o widgets de otras páginas. | Scope local, prefijos, diff/QA visual por página; no añadir estilos globales. |
| Breakpoints originales no equivalen necesariamente a los del kit Atomic | Reflow, salto de línea o navegación móvil a distinto ancho. | Comparar matriz de viewports; traducir reglas a breakpoints disponibles y conservar umbrales especiales sólo en CSS local aprobado. |
| Fuentes nombradas pero sin carga identificada en los archivos | Sustitución de fuentes y pérdida de fidelidad/espacio. | Confirmar disponibilidad en staging; registrar licencia y fuente de carga antes de adoptar fuentes remotas. |
| Efectos de scroll dependen del DOM estático | Reveal/parallax duplicado o ausente en render asíncrono de Elementor. | Hooks Elementor, init idempotente, teardown y pruebas del editor/frontend. |
| Escena y tarjetas dependen de APIs de animación/puntero | Alto costo de replicación y diferencias en móvil/reduced motion. | Mantener scripts pequeños, fallback estático y compara fotogramas a scroll/viewport definidos. |
| Dialogs accesibles son JavaScript propio | Riesgo de bloquear el fondo o perder foco al cerrar. | Mantener semántica/atributos, trap/restore, Escape, backdrop, modo no-JS y pruebas de teclado/lector. |
| Activos fuente faltantes de la biblioteca staging | Retrato/isotipo no se renderizan al usar `e-image`/`e-svg`. | Confirmar equivalencia de logos; solicitar carga aprobada del retrato y SVGs ausentes antes de componer. |
| Material original mezcla SVG inline, pseudo-elementos y activos externos | Pérdida de ornamentos al construir sólo con widgets de texto. | Inventariar y versionar cada dibujo; usar IDs reales de media tras carga; nunca inventar IDs ni usar un widget HTML como página completa. |
| 167 claves pueden desviarse de los textos editados en Elementor | Traducciones truncadas o WhatsApp con idioma incorrecto. | QA de cobertura texto por texto y validación automatizada del JSON/fuentes antes del release. |
| Assets MCP de staging no demuestran equivalencia con archivos fuente | Reutilizar un recurso visualmente distinto o alt incorrecto. | Comparar archivo y render; validar alt; usar biblioteca de staging sólo tras confirmar. |
| Slugs/permalinks de página no gestionados por esta tarea | URL final no coincide con ruta objetivo SEO o con selector de carga. | Acordar ruta y traducciones por separado; no cambiar URLs durante auditoría/MF-004 sin permiso explícito. |
| Interacciones/código dependientes de Elementor en modo editor | Interferencia con iframe, editor o render server-side. | Pruebas separadas en editor y frontend; excluir únicamente efectos incompatibles del canvas sin deshabilitarlos en frontend. |

## 5. Backlog técnico y criterios de aceptación

Priorizar secuencialmente; cada tarea requiere revisión antes de pasar a la siguiente cuando cambie el HTML visible.

| ID | Tarea | Dependencia | Criterios de aceptación |
|---|---|---|---|
| MF-004 | Construir Hero fuente en página de laboratorio autorizada | Confirmar la página/árbol destino, estado draft y acceso a activos requeridos; revisar `DESIGN_SYSTEM_V5.md`. | H1, eyebrow, descripción, CTAs y orden coinciden con la variante ES aprobada; estructura V4 editable; CSS local aislado; no crea páginas, no publica ni toca kit/plantillas. Capturas desktop/tablet/móvil guardadas para revisión. |
| MF-005 | Cargar/verificar activos fuente necesarios | Aprobación de medios y método de carga permitido. | Cada logo/retrato/SVG comparado con la fuente; medios consultables por ID en staging; dimensiones y alt localizados verificados; se mantiene fuente en Git. |
| MF-006 | Auditar y migrar estilos del Hero | MF-004 y MF-005. | Gradiente, composición, proporción, tipografía y CTA se comparan en viewport igual; reglas aplicadas sólo a root de página; no hay CSS global; diferencias aceptadas documentadas. |
| MF-007 | Convertir red de Filosofía y arte de Transformación | Contratos de SVG y test responsive del Hero. | Red, nodos, conectores y transición visual preservan contenido y degradan de forma legible sin JS; la escena pasa resize, scroll y reduced-motion. |
| MF-008 | Migrar Servicios + siete casos | Estrategia modal aprobada y copy/URLs revisados. | Siete tarjetas y siete diálogos completos; abrir/cerrar por click/Escape/backdrop; focus trap, `inert` y restore focus; fallback hash; sin pérdida de CTAs. |
| MF-009 | Migrar Método y Cursos | Sistema de grids/typography aprobado. | Cuatro pasos y tres cursos conservan orden, copy, diseño y links; desktop y móvil contrastados; las tarjetas son editables de forma individual. |
| MF-010 | Migrar Sobre mí y Contacto | Verificación de retrato y CTA. | Retrato con alt correcto, tres columnas adaptables, enlace LinkedIn seguro y todos los CTAs WhatsApp con texto localizado; no se añade formulario inexistente en V5. |
| MF-011 | Rehacer navegación, footer y traducciones | Estructura página y decisión multilingüe/SEO. | Menú funcional con teclado, móvil, click fuera, Escape y ARIA; ES/EN/FR linkean a su traducción; metadatos y hreflang correctos; fallback disponible. |
| MF-012 | QA visual, funcional, accesibilidad y regresión staging | Secciones MF-004–MF-011. | Screenshots a 1440, 1024, 900, 760, 410, 390 y 360 px; comparación con V5 fuente; no scroll horizontal accidental; prueba de keyboard/reader/reduced-motion; todos los enlaces de prueba validados; homepage existente no varía. |
| MF-013 | Preparar release bajo revisión | QA aprobada y permiso de publicación/despliegue aparte. | Diff de código revisado; assets/cambios limitados a landing; no credenciales; checklist de backup/rollback y aprobación de release; publicación sólo con autorización explícita. |

### Criterios globales de aceptación

- Texto, idioma, jerarquía y comportamiento se contrastan con los tres archivos HTML, no con una maqueta inventada.
- Estructura de sección/texto/tarjetas/botones es V4 y editable por widget donde el schema lo permita; `get-page-structure` confirma el árbol después de cada escritura autorizada.
- CSS/JS están en control de versiones, cargan sólo en la página aprobada y no cambian el aspecto de otras páginas.
- No hay widgets HTML monolíticos, plugins añadidos, archivos de producción ni credenciales en el repositorio.
- Reduced-motion, teclado, enlaces, `alt`, `aria`, idioma, responsive y CTA se prueban además de la imagen visual.
- Ningún draft se publica; los cambios realizados mediante MCP se confirman en lectura y el estado se verifica de nuevo.

## 6. Plan MF-004 — Hero original V5

MF-004 debe implementar el Hero existente en la página de laboratorio autorizada con su texto original, no el hero genérico de pruebas anterior.

**Conflicto a resolver antes de escribir:** la lectura MCP de MF-003 encontró que la página 153 ya contiene un árbol Atomic V4 con un root `Hero Section` (ID observado `544a554a`) y sus hijos. No se debe añadir otro Hero por defecto ni reemplazar/eliminar ese árbol sin autorización explícita. En MF-004, después de leer de nuevo la página, se debe acordar si se reutiliza/actualiza esa estructura o se sustituye sólo el Hero de prueba; nunca alterar elementos fuera del alcance autorizado.

1. **Preflight:** leer por MCP el post objetivo, estado, URL de staging y árbol. Guardar el tree/ID preexistente como referencia; no sobrescribir elementos ni usar IDs inventados. Confirmar permiso de edición y que el documento siga en borrador.
2. **Captura de referencia:** registrar la vista V5 ES a 1440 × 900, 1024 × 768, 760 × 900, 390 × 844 y 360 × 800. Anotar salto de línea H1, altura/espacio, nav y posición visible de CTAs.
3. **Assets:** cotejar logos de staging con los SVG fuente; confirmar cómo se servirán `hero-wave` y el retrato/isotipo. Sin IDs de biblioteca confirmados, no referenciar medios por ID inventado.
4. **Estructura V4:** contenedor root semántico `section`, contenido centrado, eyebrow, H1, párrafo, contenedor de CTA con enlaces/button Atomic y la onda como activo SVG si está disponible. Una configuración por elemento con IDs únicos; no HTML monolítico.
5. **Estilos locales:** preservar altura mínima original (780 px como referencia fuente), radios/espaciados y sizing tipográfico; construir gradiente y viñeta sólo en el root. No cambiar clases/variables/default styles del kit. Respetar anchos y breakpoints V5 con overrides locales después de mapear los breakpoints MCP.
6. **Comportamiento:** conservar hash/WhatsApp real después de verificar los destinos; implementar progresión/hover sólo con aprobación y comparar con la fuente. Mantener CTA accesible sin JS; no dejar URL de prueba como conversión final.
7. **Guardar/verificar:** construir por `elementor-build-composition` en modo que preserve todo contenido ajeno al área autorizada; confirmar el resultado y los IDs devueltos. Releer estructura/estado con `elementor-get-page-structure` y `elementor-list-posts`, generar preview temporal sólo para revisión y comparar capturas.
8. **Cierre:** registrar exactitud alcanzada, ajustes manuales, breakpoints sin equivalente, activos pendientes y estado borrador. No publicar ni pasar a homepage sin una autorización posterior.

### Aceptación de MF-004

- El Hero corresponde al copy y composición V5 en el idioma acordado y se edita como Atomic V4 en el panel de navegación de Elementor.
- Los tres viewports (desktop, tablet, móvil) no presentan clipping ni overflow horizontal; título y CTA permanecen legibles y alcanzables.
- La paleta, onda, forma del fondo y jerarquía se comparan con el original; cada desviación por limitación de widget/activo se enumera explícitamente.
- La página conserva su ID/estado draft; una segunda lectura confirma los elementos y sus IDs reales; ningún otro documento o estilo global se tocó.
- Captura/preview se revisa con usuario/editor autenticado cuando corresponda; la URL firmada/temporal no se trata como URL pública estable.

## 7. Resultado MCP de esta fase

La auditoría usó sólo llamadas de lectura del servidor `martz-favela-staging-elementor`:

- `elementor-list-widget-schemas` y `elementor-get-widget-schema` para capacidades/esquemas.
- `elementor-get-page-structure` para leer el árbol de la página 153.
- `elementor-list-assets` para leer medios disponibles en staging.
- `elementor-list-components` para confirmar que no hay componentes existentes.
- `elementor-list-resources` y `elementor-read-resource` para recursos de clases/variables, interacciones y guías.

**No se ejecutó ninguna herramienta de escritura de Elementor/WordPress.** La página 153, homepage ID 7, estilos, plantillas, estado de publicación y biblioteca de medios no se modificaron durante MF-003.
