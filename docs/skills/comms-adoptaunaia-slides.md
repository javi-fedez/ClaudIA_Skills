---
name: comms-marca-a-slides
description: Crea presentaciones web one-page con navegación tipo diapositiva (← → · Espacio · F pantalla completa) usando la identidad de marca Marca A (negro + whiteboard hand-drawn + rojo correctivo + azul de flujo, Montserrat + League Spartan). Plantilla autocontenida sin dependencias. Reutilizable para Marca B o Marca C intercambiando design tokens. Use this when the user asks for a new slide presentation, a deck, a webinar storyboard, a one-page presentation, or anything that follows the Marca A visual language.
---

# Skill — Marca A Slides

Plantilla completa para construir presentaciones web one-page en formato diapositiva. Pensada para ser **literal**: copias el archivo, reemplazas placeholders, y tienes una presentación funcional con la identidad de marca aplicada.

**Invocación:** `/comms-marca-a-slides "tema de la presentación"` o cualquier petición de "presentación", "deck" o "storyboard" para Marca A.

---

## Qué entrega

Un único archivo `index.html` con:

- **Navegación tipo deck:** `←` `→` `Espacio` `Enter` para avanzar, `Home` `End` para extremos, `F` pantalla completa, `R` repetir animación, `?` ayuda, swipe en móvil, click en mitades izquierda/derecha de la pantalla.
- **Top nav** con marca + contador `01 / 19 · Etiqueta de slide actual` + versión.
- **Bottom dock** con barra de progreso clickable + botones Anterior/Siguiente + atajos visibles.
- **Help overlay** (`?`) con todos los atajos.
- **Animaciones** que reinician al entrar en cada slide (no al hacer scroll).
- **Deep linking:** `?slide=N` o `#id-de-slide` saltan a una diapositiva concreta.
- **Branding Marca A** aplicado: fondo negro, Montserrat + League Spartan, rojo `#FF3B30` correctivo, azul `#2E7DFF` de flujo, SVG hand-drawn (subrayados torcidos, círculos imperfectos, flechas a marcador).

Ejemplos de salida en producción:
- `docs/output/20260503_marca-a_storyboard-webinar/` — storyboard ejecutado de 19 slides (hero + 17 fichas + outro).
- `docs/output/20260503_marca-a_briefing-graficos-webinar/` — briefing scrollable (no tipo deck) con la misma marca.

---

## Estructura del proyecto

```
docs/output/<yyyyMMdd>_marca-a_<titulo-kebab>/
├── index.html        ← copiado desde skills/templates/marca-a-slides/index.html
├── design.md         ← copia de brands/marca-a/design.md (referencia)
├── fonts/            ← .woff2 de Montserrat + League Spartan
└── assets/           ← logos, imágenes (opcional)
```

Tras crearlo, copiar también a `docs/public/` para obtener URL pública (auto-publish memory).

---

## Workflow paso a paso

### 1. Scaffold del proyecto

```bash
DATE=$(date +%Y%m%d)
SLUG=mi-presentacion         # kebab-case del título
PROJ=/workspace/ClaudIA_Agent/docs/output/${DATE}_marca-a_${SLUG}

mkdir -p "$PROJ"
cp /workspace/ClaudIA_Agent/skills/templates/marca-a-slides/index.html "$PROJ/index.html"
cp /workspace/ClaudIA_Agent/brands/marca-a/design.md "$PROJ/design.md"
cp -r /workspace/ClaudIA_Agent/brands/marca-a/fonts "$PROJ/fonts"
# Opcional: cp -r /workspace/ClaudIA_Agent/brands/marca-a/assets "$PROJ/assets"
```

### 2. Reemplazar placeholders en `index.html`

| Placeholder | Significa |
|---|---|
| `{{TITULO}}` | Título de la página (aparece en `<title>` y SEO). |
| `{{DESCRIPCION}}` | Meta description. |
| `{{MARCA}}` | Texto de marca en top-nav (ej. `Marca A`). |
| `{{VERSION}}` | Etiqueta de versión / fecha (ej. `v1.0 · 03 may 2026`). |
| `{{KICKER}}` | Eyebrow del hero (uppercase, tracking amplio). |
| `{{HERO_PRE}}` | Texto del hero antes del concepto rodeado en rojo. |
| `{{HERO_KEY}}` | Concepto rodeado por el círculo SVG rojo. |
| `{{HERO_TAIL}}` | Texto entre el concepto y el número final. |
| `{{HERO_NUM}}` | Número/cifra destacada en rojo al final del hero. |
| `{{SUBTITULO}}` | Subtítulo del hero (Montserrat 600, 30px). |
| `{{META_1}}`, `{{META_2}}`, `{{META_3}}` | 3 pills outlined (default · azul · rojo). |
| `{{URL}}` | URL de la marca en el outro. |

### 3. Construir las slides

La plantilla incluye 8 tipos de slide listos. **Duplica el `<section>` que necesites, cambia el contenido, ajusta el `id`, y listo**. El JS detecta automáticamente todos los `.hero`, `.stage` y `.outro` dentro de `.deck`.

#### Tipos de slide disponibles

| Tipo | Cuándo usarlo | Estructura clave |
|------|---------------|------------------|
| **`hero`** | Portada · 1 sola por presentación | `<section class="hero">` con `<h1>` y meta-pills |
| **`kinetic-text`** | Frases secuenciales con énfasis final | `.kt-stack > .kt-line` (varias) + `.kt-final` |
| **`stat-card`** | Cifra héroe + sub + bonus | `.stat-card > .stat-num` + `.stat-sub` + `.stat-bonus` |
| **`split-compare`** | Antes vs después (con tachado opcional) | `.split-compare > .side.left + .vdiv + .side.right` |
| **`sketchy-concept`** | Idea abstracta visualizada con SVG hand-drawn | `<section class="stage sketchy">` + `<svg>` con paths `.draw` |
| **`index-3pts`** | Índice del vídeo / 3 puntos clave | `.index-3 > .item.×3` |
| **`marker`** | Misma estructura que `index-3pts` pero con `.active`/`.dim` para marcar la sección actual | `.index-3` con uno `.active` y otros `.dim` |
| **`process-blocks`** | 3 fases de un método + nombre del método con animación letra a letra | `.process-blocks > .block.×3` + `.method-name > .letter` |
| **`outro`** | Cierre · 1 sola por presentación | `<section class="outro">` |

### 4. Animaciones (clases utilitarias)

Aplica estas clases a cualquier elemento dentro de un `.hero`/`.stage`/`.outro`:

| Clase | Efecto |
|-------|--------|
| `.anim-fade` | Fade-in (.6s) |
| `.anim-up` | Fade + translateY 24px → 0 (.7s) |
| `.anim-down` | Fade + translateY -24px → 0 |
| `.anim-right` | Fade + translateX -40px → 0 (entra desde izquierda) |
| `.anim-left` | Fade + translateX 40px → 0 (entra desde derecha) |
| `.anim-scale` | Fade + scale 0.85 → 1, easing back-out |
| `.draw` | SVG path se dibuja con `stroke-dashoffset` (1.2s) |
| `.draw-fast` | Idem 0.6s |

Cada clase admite `style="--d: 0.4s"` para retrasar (delay). Las animaciones **reinician** al entrar en una slide — no acumulan delays entre slides.

> **Importante:** los SVG paths con `.draw`/`.draw-fast` necesitan que `stroke-dasharray: 2200` sea suficiente para tu path. Si tu path es más largo, sustituye en CSS o aumenta. La plantilla ya incluye un valor que cubre la mayoría de casos.

### 5. Hooks JS personalizados (para animaciones complejas)

Si una slide necesita lógica JS (contador, generación dinámica de elementos, etc.), regístrala en el objeto `customHooks` al final del `<script>`:

```js
function runStatCardCounter() {
  var el = document.getElementById('stat-counter');
  if (!el) return;
  // ... lógica de contador
}
var customHooks = {
  'stat-card': runStatCardCounter,    // nombre del id de la <section>
};
```

El hook se llama cada vez que el usuario entra en esa slide (incluyendo `R` para repetir).

### 6. Personalizar la marca (override Marca B / Marca C / otra)

La plantilla está parametrizada con CSS variables en `:root`. Para cambiar a otra marca:

```css
/* Marca B: dark navy + lime + gradients */
:root {
  --bg: #0B1A2E;
  --bg-alt: #102843;
  --fg: #FFFFFF;
  --soft: #9DB1C9;
  --rule: rgba(200,250,58,0.12);
  --red: #C8FA3A;          /* tu accent principal aquí */
  --blue: #5BE0A1;         /* tu accent secundario aquí */
  --font-head: 'Jost', sans-serif;
  --font-body: 'Jost', sans-serif;
}
```

```css
/* Marca C: B&W puro editorial */
:root {
  --bg: #FFFFFF;
  --bg-alt: #F7F7F7;
  --fg: #000000;
  --soft: #5A5A5A;
  --rule: #D4D4D4;
  --red: #000000;          /* sin color, énfasis con peso */
  --blue: #000000;
  --font-head: 'Cardo', serif;
  --font-body: 'Inter', sans-serif;
}
```

También sustituye los `@font-face` por las fuentes correspondientes (copia desde `brands/<marca>/fonts/`). Para Marca C, considera además quitar el círculo SVG hand-drawn del hero y simplificar elementos sketchy — la marca rechaza ese lenguaje (B&W estricto).

### 7. Publicar

```bash
SRC=/workspace/ClaudIA_Agent/docs/output/${DATE}_marca-a_${SLUG}
DST=/workspace/ClaudIA_Agent/docs/public/${DATE}_marca-a_${SLUG}
rm -rf "$DST"; cp -r "$SRC" "$DST"

URL="https://files.example.com/TU_TOKEN_DE_ACCESO/${DATE}_marca-a_${SLUG}/index.html"
echo "$URL"
```

---

## Atajos de teclado (lo que el usuario final puede pulsar)

| Tecla | Acción |
|-------|--------|
| `→` `↓` `Espacio` `Enter` `PageDown` | Siguiente |
| `←` `↑` `Backspace` `PageUp` | Anterior |
| `Home` / `End` | Primera / última slide |
| `F` | Pantalla completa (toggle) |
| `R` | Repetir animación de la slide actual |
| `?` | Mostrar/ocultar ayuda |
| `Esc` | Cerrar ayuda |

Soporte adicional:
- **Click izquierdo (30 % izq) / derecho (30 % der)** — anterior / siguiente
- **Swipe** táctil
- **Pips** clickables en la barra de progreso

---

## Reglas de oro

1. **Una idea por slide.** Si una slide tiene más de un titular y dos elementos visuales, divídela.
2. **Color como semántica, no decoración.** Rojo `#FF3B30` solo para énfasis correctivo (subrayar/tachar/marcar lo importante). Azul `#2E7DFF` solo para flujo (flechas, conectores, fases). Si dudas, blanco.
3. **No mezcles más de dos accents en un mismo bloque de texto.**
4. **SVG hand-drawn para todo lo «manual»** — círculos imperfectos, subrayados torcidos, tachados curvados. Nunca shapes CSS perfectas para anotaciones tipo marcador.
5. **Stagger pequeños** (0.3–0.5s entre elementos) en lugar de macro-animaciones largas.
6. **Tipografía grande** — Montserrat 700/800 a 80–168px en titulares héroe; League Spartan 400/500 a 18–28px en body.
7. **No abuses de `anim-scale`** — solo en frases finales o elementos clave.
8. **Verifica ratio 16:9** — la plantilla escala bien hasta 1480px de ancho. Si presentas en otro aspect ratio, ajusta `padding` y tamaños.

---

## Anti-patrones

- ❌ Fondos de cualquier color que no sea negro `#000000` (salvo override de marca documentado).
- ❌ Gradientes de cualquier tipo en versión Marca A.
- ❌ Cards con `border-radius` grandes — la marca prefiere rectos.
- ❌ Sombras / glows / neon.
- ❌ Stock photos, fotos de equipo, ilustraciones cartoon, iconos 3D Memphis.
- ❌ Mezclar Cardo o cualquier otra serif en una presentación Marca A (si quieres serif, usa la marca Marca C en su lugar).
- ❌ Tono pitch-de-curso. La voz es coaching técnico/profesor cercano.

---

## Referencia de archivos

- **Plantilla:** `/workspace/ClaudIA_Agent/skills/templates/marca-a-slides/index.html`
- **Brand spec:** `/workspace/ClaudIA_Agent/brands/marca-a/design.md`
- **Fuentes:** `/workspace/ClaudIA_Agent/brands/marca-a/fonts/*.woff2`
- **Logo:** `/workspace/ClaudIA_Agent/brands/marca-a/assets/logo.png`
- **Ejemplo en producción:** `/workspace/ClaudIA_Agent/docs/output/20260503_marca-a_storyboard-webinar/index.html`

---

## Cuándo NO usar esta skill

- Si el usuario pide un **vídeo** (mp4) → usar `hyperframes-brands` + skill global `hyperframes`. Este deck es web-only.
- Si el usuario pide una **infografía estática** (1 sola imagen, sin navegación) → usar `comms-html-presentation` o un PNG.
- Si el usuario pide una presentación **PowerPoint/Keynote/Google Slides** real → este formato no exporta a esos. Es web HTML.
- Si la marca destino es **Marca C** y se requiere fidelidad estricta → mejor crear una skill `comms-marca-c-editorial` separada (B&W editorial denso es un lenguaje muy distinto al whiteboard sketchy).
