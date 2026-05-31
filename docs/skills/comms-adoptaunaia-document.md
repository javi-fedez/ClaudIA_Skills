---
name: comms-marca-a-document
description: Crea documentos web largos tipo briefing/whitepaper/dossier scrolleable con identidad de marca Marca A (fondo #181818 + whiteboard hand-drawn + rojo correctivo + azul de flujo, Montserrat + League Spartan). Plantilla autocontenida sin dependencias, con topnav anclado, hero, secciones numeradas, cards, tablas, fichas con quote/grid/why, checklist y footer. Use this when the user asks for a briefing, dossier, whitepaper, internal document, technical brief, content brief, production document, long-form web report, or anything multi-section scrolleable that follows the Marca A visual language. NOT for slide decks (use comms-marca-a-slides).
---

# Skill — Marca A Document

Plantilla para construir **documentos web largos** (briefings, dossiers, whitepapers, manuales internos, technical briefs) con la identidad de Marca A. Pensada para ser **literal**: copias el directorio plantilla, reemplazas placeholders, sustituyes el contenido y publicas.

**Invocación:** `/comms-marca-a-document "tema del documento"` o cualquier petición de "briefing", "dossier", "whitepaper", "documento técnico" para Marca A.

> **No es slides.** Para presentaciones tipo deck con navegación ←→ usa `/comms-marca-a-slides`. Para vídeo HTML→MP4 usa `/hyperframes-brands` con slug `marca-a`.

---

## Qué entrega

Un único `index.html` autocontenido con:

- **Top nav** sticky con marca · enlaces de anclaje a secciones · versión/fecha
- **Hero** con eyebrow, título grande Montserrat 800/96px con SVG circle hand-drawn rojo, subtítulo, meta-pills (rotadas levemente), flecha SVG decorativa
- **Secciones numeradas** (`01`, `02`, ...) con eyebrow + h2 con underline SVG hand-drawn + lede
- **Cards** en grid 3-col con badge de prioridad (alta/media/baja → rojo/azul/soft)
- **Tablas de datos** con thead bordeado en blanco y filas con `--rule`
- **Fichas** (production cards): corner-mark numérico, type-tag, h3 con énfasis rojo/azul, meta-row de tags rotados, quote-block con barra roja, grid 2-col de fields, panel "Why" en azul
- **Checklist** con SVG check hand-drawn
- **Footer** con copyright + uso

Tokens CSS aplicados:
```css
--bg: #181818;       --bg-alt: #181818;
--fg: #FFFFFF;       --soft: #A0A0A0;
--muted: #666666;    --rule: #323232;
--red: #FF3B30;      --blue: #2E7DFF;
```

Fonts embebidas: **Montserrat** 400-800 + **League Spartan** 400-700 (woff2 locales).

Ejemplo de salida en producción: `docs/output/20260503_marca-a_briefing-graficos-webinar/`.

---

## Workflow

### 1. Crear el directorio del documento

Convención: `docs/output/<yyyyMMdd>_marca-a_<titulo-kebab>/`

```bash
SLUG=briefing-q3-2026
DATE=$(date +%Y%m%d)
PROJECT=/workspace/ClaudIA_Agent/docs/output/${DATE}_marca-a_${SLUG}
mkdir -p "$PROJECT"
```

### 2. Copiar la plantilla y los assets de marca

```bash
TEMPLATE=/workspace/ClaudIA_Agent/skills/templates/marca-a-document
BRAND=/workspace/ClaudIA_Agent/brands/marca-a

cp "$TEMPLATE/index.html" "$PROJECT/index.html"
cp -r "$BRAND/fonts" "$PROJECT/fonts"
cp -r "$BRAND/assets" "$PROJECT/assets"
cp "$BRAND/design.md" "$PROJECT/design.md"
```

### 3. Reemplazar placeholders del hero

| Placeholder | Reemplazar con |
|---|---|
| `{{TITLE}}` | Título completo del documento (también va al `<title>` y al footer) |
| `{{DESCRIPTION}}` | Meta description SEO |
| `{{EYEBROW}}` | Línea pre-título en mayúsculas pequeñas (ej: "Briefing interno · Q3 2026") |
| `{{TITLE_PRE}}` | Texto antes del término destacado (ej: "Cómo aplicar") |
| `{{TITLE_HIGHLIGHT}}` | Término que llevará el círculo SVG rojo (ej: "Microsoft Copilot 365") |
| `{{TITLE_POST}}` | Texto entre highlight y número (ej: "en") |
| `{{TITLE_NUMBER}}` | Cifra o palabra de cierre rojo (ej: "21 días") |
| `{{SUBTITLE}}` | Subtítulo Montserrat 600/32px |
| `{{DATE}}` | Fecha legible (ej: "3 de mayo de 2026") |

### 4. Sustituir las secciones de ejemplo por contenido real

La plantilla incluye 4 secciones de ejemplo:
- **Sección 1** — `card-grid` con 3 cards (red/blue/soft)
- **Sección 2** — `data-table` con thead + tbody
- **Fichas** — 1 ficha plantilla con todos los componentes (corner-mark, type-tag, h3, meta-row, quote-block, ficha-grid de 2 cols + 1 full, why)
- **Checklist** — items con SVG check hand-drawn

Borra/duplica/reordena según el documento. Cada sección debe tener un `id` único para los anchor links del topnav.

### 5. Verificación de marca

Antes de publicar:
- ✅ Solo se usan los hex de los tokens (`#181818`, `#FFFFFF`, `#A0A0A0`, `#666666`, `#323232`, `#FF3B30`, `#2E7DFF`)
- ✅ Fonts: solo Montserrat (heading) + League Spartan (body) — sin sustitutos
- ✅ Cero gradientes, cero sombras, cero glows
- ✅ Rojo y azul nunca juntos en la misma frase
- ✅ Subrayados/círculos/flechas son SVG hand-drawn (paths irregulares, `stroke-linecap: round`), NO shapes CSS perfectas
- ✅ Cards con `border-radius: 0` y border de 1px en `--rule`
- ✅ Idioma: español por defecto. Anglicismos solo para nombres propios técnicos (Copilot, prompt, agent, KPI...)

### 6. Publicar

```bash
# Auto-publicación: cualquier output va a docs/public/ vía la regla de feedback
cp -r "$PROJECT" /workspace/ClaudIA_Agent/docs/public/
```

URL pública: `https://files.example.com/<token>/<dirname>/index.html` (o usar `/ops-publish-content`).

---

## Componentes reutilizables

### Hero accent (círculo SVG rojo alrededor de un término)

```html
<span class="accent">
  Término destacado
  <svg class="hero-circle" viewBox="0 0 1200 200" preserveAspectRatio="none">
    <path class="ink-red" stroke-width="6" d="M40 100 C 30 30, 200 5, 600 8 S 1170 30, 1160 110 C 1170 180, 700 198, 400 192 S 30 180, 40 100 Z"/>
  </svg>
</span>
```

### Subrayado SVG hand-drawn dentro de un h2

```html
<h2>
  Texto con
  <span class="underline">parte subrayada
    <svg class="underline-svg" viewBox="0 0 460 22" preserveAspectRatio="none">
      <path class="ink-red" stroke-width="6" d="M5 14 C 80 4, 180 22, 280 10 S 450 16, 455 12"/>
    </svg>
  </span>
  más texto.
</h2>
```

### Quote block (cita destacada con barra roja)

```html
<div class="quote-block" data-label="PIE DE GUIÓN">
  <q>Texto literal entre comillas francesas rojas.</q>
</div>
```

El `data-label` es la etiqueta en mayúsculas que aparece arriba (`"PIE DE GUIÓN"`, `"CITA"`, `"NOTA"`...).

### Ficha completa (card de producción)

```html
<article class="ficha">
  <div class="corner-mark">G<span class="em-red">01</span></div>
  <div class="header-row"><span class="type-tag">Tipo</span></div>
  <h3>Título con <span class="em-red">énfasis</span>.</h3>
  <div class="meta-row">
    <span class="meta-tag blue">timing</span>
    <span class="meta-tag red">prio alta</span>
    <span class="meta-tag soft">duration</span>
  </div>
  <div class="quote-block" data-label="PIE DE GUIÓN"><q>...</q></div>
  <div class="ficha-grid">
    <div class="field"><div class="lbl">Etiqueta</div><div class="val">...</div></div>
    <div class="field"><div class="lbl">Etiqueta</div><div class="val">...</div></div>
    <div class="field full"><div class="lbl">Span completo</div><div class="val">...</div></div>
  </div>
  <div class="why">
    <div class="lbl">Por qué</div>
    <div class="val">Razonamiento.</div>
  </div>
</article>
```

---

## Anti-patrones

- ❌ Usar `#000000` para el fondo. La marca actualizó a `#181818` (gris muy oscuro casi negro).
- ❌ Diferenciar `bg` de `bg-alt` con cambio de color — ambos son `#181818`. La separación se hace con border `--rule`.
- ❌ Cards con `border-radius` grande. Esquinas casi rectas (0–4px máximo).
- ❌ Sombras, gradientes, glows. La marca es **flat** absoluto.
- ❌ Combinar rojo y azul en la misma frase o card.
- ❌ Sustituir Montserrat o League Spartan por fallback web. Si la fuente no carga, el documento se rompe a propósito.
- ❌ Stock photos, ilustraciones cartoon, emojis decorativos.
- ❌ Tono comercial ("descubre", "transforma"). El registro es coaching técnico.

---

## Diferencias con `comms-marca-a-slides`

| | Document (esta skill) | Slides |
|---|---|---|
| Formato | Long-form scrolleable | One-page con navegación tipo deck |
| Navegación | Anchor links + scroll | `←` `→` `Espacio` `F` |
| Caso de uso | Briefings, dossiers, whitepapers | Webinars, presentaciones, talks |
| Nº de "pantallas" | 1 página larga | N slides con animaciones |
| Densidad | Alta (mucho texto, datos, fichas) | Baja (1 idea por slide) |

---

## Referencias

- Plantilla: `/workspace/ClaudIA_Agent/skills/templates/marca-a-document/index.html`
- Brand spec: `/workspace/ClaudIA_Agent/brands/marca-a/design.md`
- Ejemplo en producción: `/workspace/ClaudIA_Agent/docs/output/20260503_marca-a_briefing-graficos-webinar/`
- Skill hermana (slides): `skills/comms-marca-a-slides.md`
- Skill loader de marca: `skills/hyperframes-brands.md`
