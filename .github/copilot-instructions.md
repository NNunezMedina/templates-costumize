# Copilot Instructions — Custom Shopify Sections (`c-*`)

Este proyecto es un **theme de Shopify (Dawn-based)** con secciones custom prefijadas con `c-`. Estas instrucciones documentan las convenciones extraídas de las secciones existentes para que Copilot genere código consistente.

---

## 1. Estructura de Archivos

### Convención de nombres

- **Secciones custom**: `sections/c-{nombre}.liquid` (ej. `c-blocks.liquid`, `c-hero-marquee.liquid`)
- **Schema name**: Prefijo `"C "` + nombre descriptivo en Title Case (ej. `"C Blocks"`, `"C Hero Marquee"`)
- **Snippets de iconos**: `snippets/c-icon-{nombre}.liquid` (ej. `c-icon-check`, `c-icon-star`, `c-icon-chevron`)

### Estructura de un archivo `.liquid` de sección

Cada sección sigue este orden obligatorio:

```
1. <style> ... </style>         ← CSS inline con scoping por section.id
2. {% liquid %} (opcional)      ← Variables Liquid si son necesarias
3. <section> ... </section>     ← HTML del componente
4. <script> ... </script>       ← JS inline (opcional, solo si hay interactividad)
5. {% schema %} ... {% endschema %} ← Configuración JSON del schema
```

---

## 2. CSS — Convenciones de Estilo

### Scoping obligatorio

Todo el CSS va dentro de un `<style>` tag al inicio del archivo. **Nunca** se usan archivos CSS externos para secciones `c-*`. Todas las reglas usan `#{{ section.id }}` como scope:

```css
#{{ section.id }} {
  font-size: 16px;
  padding-top: {{ section.settings.padding_top }}em;
  padding-bottom: {{ section.settings.padding_bottom }}em;
  background-color: {{ section.settings.bg_color }};
}
```

### Reset base

Siempre incluir `font-size: 16px` en el contenedor raíz `#{{ section.id }}`. Esto permite usar `em` como unidad relativa consistente.

> ⚠️ **Nota**: Este `font-size: 16px` hardcodeado ignora intencionalmente el setting `--font-body-scale` de Dawn (configurable en el customizer). Es un tradeoff aceptado para que todos los valores `em` de las secciones `c-*` sean predecibles. No cambiar a `font-size: inherit` o similar.

### Colores de texto — patrón estándar

Siempre aplicar colores de texto a estos selectores:

```css
#{{ section.id }} .c-section-top-subheading,
#{{ section.id }} div:not(.c-button),
#{{ section.id }} h2,
#{{ section.id }} h3 {
  color: {{ section.settings.text_color }};
}
```

> ⚠️ **Importante**: Usar `div:not(.c-button)` — **nunca** `div` a secas. El selector `div` sobreescribe el color de los botones y cualquier elemento con color propio. Si hay otros elementos que no deben heredar el color (badges, chips, etc.), agregarlos al `:not()`. Ejemplo: `div:not(.c-button, .mi-badge)`.

Si la sección usa `h1` (como heros), agregarlo al grupo.

### Accent color (opcional)

Cuando la sección distingue un color de acento:

```css
#{{ section.id }} h2 {
  color: {{ section.settings.accent_color }};
}
```

### Prefijo de clases CSS

Todas las clases CSS custom usan un prefijo corto derivado del nombre de la sección:

| Sección | Prefijo |
|---------|---------|
| `c-blocks` | `c-b-` |
| `c-hero-marquee` | `c-hm-` |
| `c-cards-grid` | `c-cg-` |
| `c-image-cta` | `c-ic-` |
| `c-faqs` | `c-f-` |
| `c-tabs-content` | `c-tc-` |
| `c-testimonials` | `c-t18-` |
| `c-content-text` | `c-ct-` |
| `c-logos` | `c-l-` |
| `c-reviews-grid` | `f-rg-` |
| `c-hero-grid` | `c-hg-` |
| `c-rich-text` | `c-rt-` |
| `c-strip-badge` | `c-sb-` |
| `c-competitors-comparison` | `c-cc-` |

**Regla**: Tomar las iniciales de las palabras significativas del nombre (sin el `c-` prefix del nombre de archivo, aunque se conserva el `c-` en la clase). Ejemplo: `c-video-grid` → `c-vg-`.

### Clases utilitarias globales

Estas clases se usan en todas las secciones y **no** se redefinen por sección:

| Clase | Uso |
|-------|-----|
| `c-section` | Clase del `<section>` wrapper |
| `c-section-top` | Contenedor del header de la sección (subheading + heading + paragraph) |
| `c-section-top-subheading` | Clase del subheading (`<p>`) |
| `c-t-80` | Wrapper de párrafos con ancho limitado |
| `c-button` | Clase base de botones |
| `c-b1`, `c-b2`, `c-b3` | Estilos de botón (se aplican como `class="c-button c-b1"`) |
| `c-buttons-div` | Contenedor flex de botones |
| `c-active` | Clase para estados activos (tabs, toggles) |
| `page-width` | Contenedor con ancho máximo del theme |
| `page-width2` | Contenedor más estrecho |

### Grid y Layout

- Usar CSS Grid para layouts de columnas
- La propiedad `grid-template-columns` se genera con Liquid loops o settings:

```css
/* Con loop para columnas dinámicas */
grid-template-columns:{% for i in (1..section.settings.columns) %} 1fr{% endfor %};

/* Con setting directo */
grid-template-columns: {{ section.settings.grid_template_columns }};
```

- Gaps en `em` (ej. `gap: 1em;`, `gap: 3em;`)

### Responsive — Breakpoints

Usar los breakpoints del theme settings (variables globales):

| Variable | Uso típico |
|----------|-----------|
| `{{ settings.c_breakpoint1 }}` | Desktop grande |
| `{{ settings.c_breakpoint2 }}` | Tablet landscape |
| `{{ settings.c_breakpoint3 }}` | Tablet / Mobile (el más usado) |
| `{{ settings.c_breakpoint4 }}` | Mobile pequeño |

Formato:
```css
@media (max-width: {{ settings.c_breakpoint3 }}px) {
  /* mobile styles */
}
```

En secciones simples también se usa `768px` como breakpoint directo cuando no se necesita la variable del theme.

### Estilo de media queries

- Siempre usar `max-width` para mobile-first overrides
- El breakpoint más común es `c_breakpoint3` para el cambio a 1 columna
- Pattern para grid responsive:

```css
@media (max-width: {{ settings.c_breakpoint3 }}px) {
  #{{ section.id }} .c-XX-grid {
    grid-template-columns: 1fr;
  }
}
```

### Unidades

- **Padding/spacing**: `em` (basado en el `font-size: 16px` del root)
- **Tamaños de icono/imagen**: `em`
- **Breakpoints**: `px`
- **Nunca** usar `rem` en los valores de CSS inline (aunque los settings dicen "rem" como label, el valor real se usa como `em`)

### Animaciones CSS

Para marquees/scroll infinito, usar `@keyframes` con ID único por sección:

```css
@keyframes {{ section.id }}-scroll {
  from { transform: translateX(0); }
  to { transform: translateX(calc(-100% - var(--gap))); }
}
```

---

## 3. HTML — Estructura del Markup

### Wrapper principal

```liquid
<section id="{{ section.id }}" class="c-section">
  <div class="{{ section.settings.section_container }}">
    <!-- contenido -->
  </div>
</section>
```

O con `page-width` fijo:
```liquid
<section id="{{ section.id }}" class="c-section">
  <div class="page-width">
    <!-- contenido -->
  </div>
</section>
```

### Section Top (header de sección)

Patrón obligatorio cuando la sección tiene heading:

```liquid
<div class="c-section-top">
  {%- if section.settings.section_top_subheading != empty -%}
    <p class="c-section-top-subheading">{{ section.settings.section_top_subheading }}</p>
  {%- endif -%}
  {%- if section.settings.section_top_heading != empty -%}
    <h2>{{ section.settings.section_top_heading }}</h2>
  {%- endif -%}
  {%- if section.settings.section_top_paragraph != empty -%}
    <div class="c-t-80">{{ section.settings.section_top_paragraph }}</div>
  {%- endif -%}
</div>
```

- Usar `{%- -%}` (whitespace trim) en los condicionales
- El heading es `<h2>` (excepto en heros donde es `<h1>`)
- El párrafo se envuelve en `<div class="c-t-80">`

### Blocks loop

```liquid
{% for block in section.blocks %}
  {% case block.type %}
    {% when 'Item' %}
      <div class="c-XX-item" {{ block.shopify_attributes }}>
        <!-- contenido del block -->
      </div>
  {% endcase %}
{% endfor %}
```

- Siempre incluir `{{ block.shopify_attributes }}` en el wrapper del block
- Usar `{% case block.type %}` incluso cuando solo hay un tipo de block

### Imágenes

```liquid
{% if block.settings.block_image %}
  {{ block.settings.block_image | image_url: width: 750 | image_tag: loading: 'lazy', height: 'auto' }}
{% endif %}
```

- Usar `image_url` + `image_tag` (nunca `img_url`)
- `loading: 'lazy'` por defecto, `loading: 'eager'` solo para el hero principal (primera imagen visible en viewport al cargar la página)
- Los marquees usan `loading: 'lazy'` — aunque sean animados, tienen muchas imágenes (duplicadas) y la mayoría no están above-the-fold. Usar `eager` en marquees daña el LCP.
- Anchos típicos: `width: 500` (iconos), `width: 750` (cards), `width: 1000` (medianas), `width: 1250` (grandes), `width: 1500` (full-width)

### Botones

```liquid
{%- if section.settings.button_text != empty -%}
  <div class="c-buttons-div">
    <a class="c-button {{ section.settings.button_style }}" href="{{ section.settings.button_url }}">
      {{ section.settings.button_text }}
    </a>
  </div>
{%- endif -%}
```

- El estilo del botón viene de un setting (`c-b1`, `c-b2`, `c-b3`)
- Los botones se envuelven en `c-buttons-div`

---

## 4. JavaScript — Convenciones

### JS inline con scoping

El JS se escribe dentro de un `<script>` tag después del HTML, scopeado por `section.id`:

```javascript
document.querySelectorAll("#{{ section.id }} .c-f-card").forEach(item => {
  item.addEventListener("click", () => {
    // toggle logic
  });
});
```

### Patrones comunes

**Accordion/FAQ toggle:**
```javascript
// Cachear queries fuera del handler — nunca hacer querySelectorAll dentro del click
const faqs{{ section.id | replace: '-', '_' }} = document.querySelectorAll("#{{ section.id }} .c-f-card");
faqs{{ section.id | replace: '-', '_' }}.forEach(faq => {
  faq.addEventListener("click", () => {
    // Cerrar todos
    faqs{{ section.id | replace: '-', '_' }}.forEach(faq2 => faq2.classList.remove("c-f-active"));
    // Abrir el clickeado
    faq.classList.add("c-f-active");
  });
});
```

**Tabs:**
```javascript
// Cachear queries fuera del handler — nunca hacer querySelectorAll dentro del click
const tcCards{{ section.id | replace: '-', '_' }} = document.querySelectorAll("#{{ section.id }} .c-tc-card");
const tcTriggers{{ section.id | replace: '-', '_' }} = document.querySelectorAll("#{{ section.id }} .c-tc-trigger");
tcTriggers{{ section.id | replace: '-', '_' }}.forEach((tab, index) => {
  tab.addEventListener("click", () => {
    // Remove active de todos
    tcCards{{ section.id | replace: '-', '_' }}.forEach(c => c.classList.remove("c-active"));
    tcTriggers{{ section.id | replace: '-', '_' }}.forEach(t => t.classList.remove("c-active"));
    // Activar el correspondiente
    tcCards{{ section.id | replace: '-', '_' }}[index].classList.add("c-active");
    tcTriggers{{ section.id | replace: '-', '_' }}[index].classList.add("c-active");
  });
});
```

> ⚠️ **Performance**: Siempre cachear los resultados de `querySelectorAll` en variables **fuera** del event handler. Llamar `querySelectorAll` dentro de un handler recorre el DOM en cada click, lo que escala mal cuando hay muchos elementos o múltiples secciones en la página.

### GSAP (opcional)

GSAP + ScrollTrigger está activo en el proyecto y cargado globalmente vía `layout/theme.liquid`. Se usa sin comentar en secciones como `c-tabs-content`, `c-blocks`, `c-stop-start`, `c-compare-models-v2`, `c-content-blocks` y `c-collection-circles`.

```javascript
let myTl{{ section.id | replace: '-', '_' }} = gsap.timeline({
  scrollTrigger: {
    trigger: "#{{ section.id }} .c-XX-element",
    toggleActions: "play none none none",
    start: "0% 90%",
    end: "100% 70%"
  }
})
.fromTo("#{{ section.id }} .c-XX-element > *",
  { y: "100%", opacity: 0 },
  { y: "0%", opacity: 1, stagger: 0.3 }
);
```

- Variable name: `myTl{{ section.id | replace: '-', '_' }}` para evitar conflictos entre secciones
- GSAP ya está disponible globalmente — no hace falta cargarlo dentro de la sección
- **Siempre incluir un CSS fallback** para los elementos animados por GSAP. Si GSAP no carga (CDN caído, ad-blocker, JS deshabilitado), los elementos que inician en `opacity: 0` quedan permanentemente invisibles:

```css
/* Fallback: los elementos son visibles por CSS por defecto */
#{{ section.id }} .c-XX-element > * {
  opacity: 1;
}
```
GSAP sobreescribirá este valor al inicializar. Si no carga, el contenido sigue siendo visible.

---

## 5. Schema — Convenciones del JSON

### Estructura del schema

```json
{
  "name": "C Nombre Sección",
  "blocks": [ ... ],
  "settings": [ ... ],
  "presets": [
    {
      "name": "C Nombre Sección"
    }
  ]
}
```

### Orden de settings (obligatorio)

1. **Settings específicos de la sección** (contenido, layout, etc.)
2. **`"header": "Button"`** → button_text, button_url, button_style
3. **`"header": "Section Colors"`** → accent_color (opcional), bg_color, text_color
4. **`"header": "Section Top"`** → section_top_subheading, section_top_heading, section_top_paragraph, section_top_alignment, text_alignment

   > ⚠️ **Inconsistencia en el proyecto**: El ID del setting de alineación de texto tiene dos variantes según la sección — `text_alignment` (usado en `c-blocks`, `c-hero-marquee`, `c-faqs`) y `section_top_text_alignment` (usado en `c-image-cta`, `c-cards-grid`, `c-tabs-content`). Para secciones nuevas usar **`text_alignment`** como estándar y asegurarse de que el CSS referencia el mismo ID: `text-align: {{ section.settings.text_alignment }}`.
5. **`"header": "Section Padding"`** → padding_top, padding_bottom

### Settings recurrentes — Copiar exactamente

**Button group:**
```json
{
  "type": "header",
  "content": "Button"
},
{
  "type": "text",
  "id": "button_text",
  "label": "Button text",
  "default": "Buy Now"
},
{
  "type": "url",
  "id": "button_url",
  "label": "Button Link"
},
{
  "type": "text",
  "id": "button_style",
  "label": "Button style",
  "default": "c-b1"
}
```

**Section Colors group:**
```json
{
  "type": "header",
  "content": "Section Colors"
},
{
  "type": "color",
  "id": "accent_color",
  "label": "Accent color",
  "default": "#5d3be4"
},
{
  "type": "color",
  "id": "bg_color",
  "label": "Background color",
  "default": "#ffffff"
},
{
  "type": "color",
  "id": "text_color",
  "label": "Text color",
  "default": "#000000"
}
```

**Section Top group:**
```json
{
  "type": "header",
  "content": "Section Top"
},
{
  "type": "text",
  "id": "section_top_subheading",
  "label": "Subheading",
  "default": "subheading"
},
{
  "type": "text",
  "id": "section_top_heading",
  "label": "Heading",
  "default": "Section Heading"
},
{
  "type": "richtext",
  "id": "section_top_paragraph",
  "label": "Paragraph",
  "default": "<p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Suspendisse varius enim in eros elementum tristique. Duis cursus, mi quis viverra ornare.</p>"
},
{
  "type": "select",
  "id": "section_top_alignment",
  "label": "Section Top Alignment",
  "default": "center",
  "options": [
    { "value": "flex-start", "label": "Left" },
    { "value": "center", "label": "Center" },
    { "value": "flex-end", "label": "Right" }
  ]
},
{
  "type": "select",
  "id": "text_alignment",
  "label": "Text alignment",
  "default": "center",
  "options": [
    { "value": "left", "label": "Left" },
    { "value": "center", "label": "Center" },
    { "value": "right", "label": "Right" }
  ]
}
```

> ⚠️ Algunas secciones existentes usan `section_top_text_alignment` en lugar de `text_alignment`. Para nuevas secciones usar siempre `text_alignment`. Verificar que el CSS use el mismo ID: `text-align: {{ section.settings.text_alignment }}`.

```json
```

**Section Padding group:**
```json
{
  "type": "header",
  "content": "Section Padding"
},
{
  "type": "range",
  "id": "padding_top",
  "label": "Padding top",
  "min": 0,
  "max": 15,
  "step": 1,
  "default": 4,
  "unit": "rem"
},
{
  "type": "range",
  "id": "padding_bottom",
  "label": "Padding bottom",
  "min": 0,
  "max": 15,
  "step": 1,
  "default": 4,
  "unit": "rem"
}
```

**Section container (cuando aplica):**
```json
{
  "type": "select",
  "id": "section_container",
  "label": "Section container",
  "default": "page-width",
  "options": [
    { "value": "page-width", "label": "Normal" },
    { "value": "", "label": "None" },
    { "value": "page-width2", "label": "Smaller" }
  ]
}
```

**Columns (cuando aplica):**
```json
{
  "type": "range",
  "id": "columns",
  "label": "Columns",
  "min": 1,
  "max": 6,
  "step": 1,
  "default": 3
},
{
  "type": "range",
  "id": "mobile_columns",
  "label": "Mobile Columns",
  "min": 1,
  "max": 6,
  "step": 1,
  "default": 1
}
```

### Blocks

- **Tipo por defecto**: `"Item"` (type y name iguales, en PascalCase)
- **Limit**: Generalmente `50`
- **IDs de block settings** usan prefijo `block_`: `block_heading`, `block_paragraph`, `block_image`, `block_icon`
- Tipos de block especializados: `"FAQ"`, `"Button"`, `"Image"`, `"Logo"`, `"Review"`, `"Filter"`

### Presets

Siempre incluir al menos un preset con el mismo nombre que la sección:

```json
"presets": [
  {
    "name": "C Nombre Sección"
  }
]
```

Para secciones con blocks, incluir blocks default en el preset:

```json
"presets": [
  {
    "name": "C Cards Grid",
    "blocks": [
      { "type": "Item" },
      { "type": "Item" },
      { "type": "Item" }
    ]
  }
]
```

---

## 6. Marquee / Scroll Infinito

Patrón estándar para marquees (logos, imágenes):

```liquid
<div class="c-XX-wrapper">
  {% for i in (1..2) %}
    <div class="c-XX-marquee-content" {% if forloop.index == 2 %}aria-hidden="true"{% endif %}>
      {% for block in section.blocks %}
        <!-- contenido -->
      {% endfor %}
    </div>
  {% endfor %}
</div>
```

- Duplicar el contenido (loop `1..2`) para scroll continuo
- El segundo grupo lleva `aria-hidden="true"`
- CSS animation con `--gap` variable
- Incluir `will-change: transform` en el elemento que se anima para que el browser lo promueva a su propio compositor layer y evite jank, especialmente en mobile:

```css
#{{ section.id }} .c-XX-marquee-content {
  will-change: transform;
}
```
- Settings típicos: `marquee_speed`, `direction`, `image_gap`, `toggle_pause_on_hover`

---

## 7. Snippets de Iconos

Los iconos SVG inline se renderizan como snippets:

```liquid
{% render 'c-icon-check' %}
{% render 'c-icon-star' %}
{% render 'c-icon-chevron' %}
{% render 'icon-plus' %}       {# del theme base Dawn #}
{% render 'icon-arrow' %}      {# del theme base Dawn #}
```

Los snippets `c-icon-*` son custom del proyecto. Los snippets `icon-*` (sin `c-`) son del theme Dawn base.

---

## 8. Defaults y Valores Comunes

| Setting | Default |
|---------|---------|
| `bg_color` | `#ffffff` o `#fff` |
| `text_color` | `#000000` o `#000` |
| `accent_color` | `#5d3be4` |
| `padding_top` | `4` |
| `padding_bottom` | `4` |
| `button_style` | `c-b1` |
| `section_top_alignment` | `center` |
| `text_alignment` | `center` |
| `columns` | `3` |
| `mobile_columns` | `1` |

---

## 9. Checklist para Nueva Sección

Al crear una nueva sección `c-*`:

- [ ] Nombre del archivo: `sections/c-{nombre}.liquid`
- [ ] `font-size: 16px` en `#{{ section.id }}`
- [ ] Scoping CSS con `#{{ section.id }}`
- [ ] Prefijo de clases CSS único (iniciales)
- [ ] Colores de texto aplicados a `div:not(.c-button), h2, h3, .c-section-top-subheading` (nunca `div` a secas)
- [ ] `<section id="{{ section.id }}" class="c-section">`
- [ ] Section Top con el patrón estándar (si aplica)
- [ ] `{{ block.shopify_attributes }}` en cada block wrapper
- [ ] Imágenes con `image_url` + `image_tag` + `loading: 'lazy'`
- [ ] Media query con `{{ settings.c_breakpoint3 }}` para mobile
- [ ] Schema con orden correcto: Settings específicos → Button → Colors → Section Top → Padding
- [ ] Preset con nombre `"C Nombre"`
- [ ] Block settings con prefijo `block_`
