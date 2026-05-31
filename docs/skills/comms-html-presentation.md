# Skill: Presentaciones HTML (estilo PowerPoint)

Genera presentaciones interactivas en formato `.html` con CSS y JavaScript integrados.
Navegables con las teclas de cursor (← →), visualizables en cualquier navegador sin
dependencias externas. **Salida siempre en `docs/output/` relativo al project root — resolver a ruta absoluta antes de escribir.**

---

## Cuándo usar esta skill

- Presentaciones para clientes, formaciones o demos
- Decks de ventas o propuestas visuales
- Materiales de curso con diapositivas
- Informes ejecutivos con formato visual
- Onboarding de empleados o clientes

---

## Estructura del archivo HTML

El archivo tiene tres capas bien diferenciadas:

```
<head>
  ├── BLOQUE A — Colores personalizables (:root con --c1/c2/c3-dark/light)  ← EDITAR
  ├── BLOQUE B — Modos dark/light + variables semánticas                    ← NO TOCAR
  ├── BLOQUE C — CSS de layout: #presentation, #stage, .slide               ← NO TOCAR
  ├── BLOQUE D — CSS de tipos de slide (cover, list, kpi, etc.)              ← NO TOCAR
  └── BLOQUE E — CSS de controles de navegación                             ← NO TOCAR

<body>
  ├── #btn-fs           → botón fullscreen fijo arriba-derecha               ← NO TOCAR
  ├── #hint             → texto de ayuda que desaparece a los 4.5s           ← NO TOCAR
  ├── #presentation
  │   ├── #stage
  │   │   ├── #progress-bar                                                  ← NO TOCAR
  │   │   └── .slide × N                                                     ← EDITAR CONTENIDO
  │   └── #controls (prev · dots · counter · next · theme)                  ← NO TOCAR
  └── <script> — lógica de navegación completa                               ← NO TOCAR
```

**Regla principal:** solo se editan los valores de color en `:root` y el contenido de los `.slide`. Todo lo demás es infraestructura fija de navegación.

---

## BLOQUE A — Colores personalizables (único bloque editable en CSS)

```css
/* ═══════════════════════════════════════════════════
   COLORES PERSONALIZABLES — cambia solo estos valores
═══════════════════════════════════════════════════ */
:root {
  --c1-dark:  #6366f1;   /* acento 1 dark  — indigo   */
  --c2-dark:  #f59e0b;   /* acento 2 dark  — ámbar    */
  --c3-dark:  #10b981;   /* acento 3 dark  — esmeralda*/

  --c1-light: #4338ca;   /* acento 1 light             */
  --c2-light: #b45309;   /* acento 2 light             */
  --c3-light: #047857;   /* acento 3 light             */
}
```

**Uso de cada acento:**
- `--c1` → color primario: títulos, CTA, borde de progreso, dots activos, borde de h2
- `--c2` → color secundario: quote blocks, highlights, badges, cierre
- `--c3` → color terciario: listas, métricas positivas, éxito

**Ejemplos de paletas:**
| Paleta | c1-dark | c2-dark | c3-dark |
|--------|---------|---------|---------|
| Indigo/Ámbar/Verde (defecto) | `#6366f1` | `#f59e0b` | `#10b981` |
| Violeta/Naranja/Cyan | `#7c3aed` | `#f97316` | `#06b6d4` |
| Azul/Rosa/Lima | `#3b82f6` | `#ec4899` | `#84cc16` |
| Rojo/Índigo/Teal | `#ef4444` | `#6366f1` | `#14b8a6` |

Los valores `light` deben ser versiones más oscuras/saturadas de los `dark` para mantener contraste sobre fondo blanco. Regla práctica: bajar 15-20% de luminosidad respecto al dark.

---

## BLOQUE B — Modo dark/light + variables semánticas (NO TOCAR)

Este bloque es infraestructura. Cópialo exactamente:

```css
/* ═══════════════════════════════════════════════════
   MODO OSCURO (por defecto)
═══════════════════════════════════════════════════ */
:root, [data-theme="dark"] {
  --c1: var(--c1-dark);
  --c2: var(--c2-dark);
  --c3: var(--c3-dark);

  --bg:        #080808;
  --bg2:       #111111;
  --bg3:       #1c1c1c;
  --bg-glass:  rgba(255,255,255,0.04);
  --text:      #f0f0f0;
  --text2:     #888;
  --border:    rgba(255,255,255,0.08);
  --shadow:    0 0 0 1px rgba(255,255,255,0.06), 0 8px 32px rgba(0,0,0,0.6);
  --grid-color: rgba(255,255,255,0.03);
}

/* ═══════════════════════════════════════════════════
   MODO CLARO
═══════════════════════════════════════════════════ */
[data-theme="light"] {
  --c1: var(--c1-light);
  --c2: var(--c2-light);
  --c3: var(--c3-light);

  --bg:        #ffffff;
  --bg2:       #f4f4f5;
  --bg3:       #e4e4e7;
  --bg-glass:  rgba(0,0,0,0.03);
  --text:      #0a0a0a;
  --text2:     #71717a;
  --border:    rgba(0,0,0,0.09);
  --shadow:    0 0 0 1px rgba(0,0,0,0.07), 0 8px 32px rgba(0,0,0,0.1);
  --grid-color: rgba(0,0,0,0.04);
}
```

---

## BLOQUE C — Layout del stage (NO TOCAR)

```css
/* ═══════════════════════════════════════════════════
   RESET & BASE
═══════════════════════════════════════════════════ */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

html, body {
  width: 100%; height: 100%;
  background: var(--bg);
  font-family: 'Segoe UI', system-ui, -apple-system, 'Helvetica Neue', sans-serif;
  color: var(--text);
  overflow: hidden;
  transition: background 0.4s, color 0.4s;
}

/* ═══════════════════════════════════════════════════
   PRESENTACIÓN
═══════════════════════════════════════════════════ */
#presentation {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100vw;
  height: 100vh;
}

/* Stage 16:9 */
#stage {
  width:  min(100vw, calc(100vh * 16 / 9));
  height: min(100vh, calc(100vw * 9 / 16));
  position: relative;
  overflow: hidden;
  background: var(--bg);
  /* Grid de fondo */
  background-image:
    linear-gradient(var(--grid-color) 1px, transparent 1px),
    linear-gradient(90deg, var(--grid-color) 1px, transparent 1px);
  background-size: 48px 48px;
}

/* ═══════════════════════════════════════════════════
   SLIDES BASE
═══════════════════════════════════════════════════ */
.slide {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: flex-start;
  padding: clamp(2rem, 6%, 5rem);
  opacity: 0;
  pointer-events: none;
  z-index: 0;
  transform: translateX(80px) scale(0.98);
  transition: opacity 0.38s cubic-bezier(.4,0,.2,1),
              transform 0.38s cubic-bezier(.4,0,.2,1);
}
.slide.active {
  opacity: 1;
  pointer-events: auto;
  z-index: 2;
  transform: translateX(0) scale(1);
}
.slide.exit-left {
  opacity: 0;
  transform: translateX(-80px) scale(0.98);
  z-index: 1;
}

/* Número de slide decorativo (fondo) */
.slide::after {
  content: attr(data-n);
  position: absolute;
  bottom: -0.05em;
  right: 0.05em;
  font-size: clamp(6rem, 20vw, 18rem);
  font-weight: 900;
  line-height: 1;
  color: var(--c1);
  opacity: 0.04;
  pointer-events: none;
  z-index: 0;
  transition: color 0.4s;
}
```

---

## BLOQUE D — CSS de tipos de slide (NO TOCAR)

```css
/* ═══════════════════════════════════════════════════
   TIPO 1: PORTADA
═══════════════════════════════════════════════════ */
.slide-cover {
  align-items: center;
  text-align: center;
  justify-content: center;
}
/* Blob decorativo izquierda */
.slide-cover::before {
  content: '';
  position: absolute;
  width: 55%;
  aspect-ratio: 1;
  border-radius: 50%;
  background: radial-gradient(circle, color-mix(in srgb, var(--c1) 30%, transparent), transparent 70%);
  left: -20%;
  top: -20%;
  pointer-events: none;
  transition: background 0.4s;
}
.slide-cover .overline {
  font-size: clamp(0.6rem, 1.3vw, 0.85rem);
  letter-spacing: 0.25em;
  text-transform: uppercase;
  color: var(--c1);
  font-weight: 600;
  margin-bottom: 1.2em;
  position: relative; z-index: 1;
}
.slide-cover h1 {
  font-size: clamp(2rem, 5.5vw, 4.5rem);
  font-weight: 900;
  line-height: 1.05;
  letter-spacing: -0.02em;
  background: linear-gradient(135deg, var(--text) 40%, var(--c1));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  max-width: 14ch;
  margin-bottom: 0.5em;
  position: relative; z-index: 1;
  transition: background 0.4s;
}
.slide-cover .subtitle {
  font-size: clamp(0.85rem, 1.8vw, 1.2rem);
  color: var(--text2);
  max-width: 48ch;
  line-height: 1.6;
  margin-bottom: 2em;
  position: relative; z-index: 1;
}
.slide-cover .pills {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5em;
  justify-content: center;
  position: relative; z-index: 1;
}
.pill {
  display: inline-flex;
  align-items: center;
  gap: 0.35em;
  padding: 0.35em 0.9em;
  border-radius: 999px;
  font-size: clamp(0.65rem, 1.2vw, 0.82rem);
  font-weight: 500;
  border: 1px solid var(--border);
  background: var(--bg-glass);
  color: var(--text2);
  backdrop-filter: blur(8px);
  transition: border-color 0.3s;
}
.pill.c1 { border-color: color-mix(in srgb, var(--c1) 40%, transparent); color: var(--c1); }
.pill.c2 { border-color: color-mix(in srgb, var(--c2) 40%, transparent); color: var(--c2); }
.pill.c3 { border-color: color-mix(in srgb, var(--c3) 40%, transparent); color: var(--c3); }

/* ═══════════════════════════════════════════════════
   TIPO 2: CONTENIDO
═══════════════════════════════════════════════════ */
.slide-content h2, .slide-list h2, .slide-two-col h2,
.slide-kpi h2, .slide-table h2 {
  font-size: clamp(1.1rem, 2.8vw, 2rem);
  font-weight: 800;
  letter-spacing: -0.015em;
  margin-bottom: 0.9em;
  align-self: stretch;
  position: relative;
  padding-left: 0.8em;
  z-index: 1;
  line-height: 1.2;
}
.slide-content h2::before, .slide-list h2::before, .slide-two-col h2::before,
.slide-kpi h2::before, .slide-table h2::before {
  content: '';
  position: absolute;
  left: 0; top: 0.1em; bottom: 0.1em;
  width: 3px;
  border-radius: 3px;
  background: linear-gradient(to bottom, var(--c1), var(--c2));
}
.slide-content .body-text {
  font-size: clamp(0.85rem, 1.7vw, 1.15rem);
  line-height: 1.75;
  color: var(--text2);
  max-width: 68ch;
  position: relative; z-index: 1;
}
.slide-content .body-text strong { color: var(--text); font-weight: 600; }

.quote-block {
  margin-top: 1.5em;
  padding: 1em 1.4em;
  border-left: 3px solid var(--c2);
  background: color-mix(in srgb, var(--c2) 8%, transparent);
  border-radius: 0 0.6em 0.6em 0;
  font-size: clamp(0.8rem, 1.5vw, 1rem);
  font-style: italic;
  color: var(--c2);
  position: relative; z-index: 1;
  transition: border-color 0.4s, background 0.4s, color 0.4s;
}

/* ═══════════════════════════════════════════════════
   TIPO 3: LISTA NUMERADA
═══════════════════════════════════════════════════ */
.slide-list { align-items: flex-start; }
.list-items {
  list-style: none;
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 0.5em;
  position: relative; z-index: 1;
}
.list-items li {
  display: flex;
  align-items: center;
  gap: 1em;
  padding: 0.7em 1em;
  background: var(--bg-glass);
  border: 1px solid var(--border);
  border-radius: 0.6em;
  font-size: clamp(0.8rem, 1.5vw, 1rem);
  color: var(--text2);
  opacity: 0;
  transform: translateX(-24px);
  transition: opacity 0.32s ease, transform 0.32s ease,
              background 0.3s, border-color 0.3s;
}
.list-items li.visible { opacity: 1; transform: translateX(0); }
.list-items li:hover { background: var(--bg3); border-color: var(--c1); }
.list-num {
  font-size: clamp(1rem, 2.2vw, 1.6rem);
  font-weight: 900;
  min-width: 1.5ch;
  text-align: center;
  flex-shrink: 0;
  line-height: 1;
}
.list-items li:nth-child(3n+1) .list-num { color: var(--c1); }
.list-items li:nth-child(3n+2) .list-num { color: var(--c2); }
.list-items li:nth-child(3n+3) .list-num { color: var(--c3); }
.list-items li .item-text { flex: 1; }
.list-items li .item-text strong { color: var(--text); display: block; font-size: 1em; }
.list-items li .item-text span  { font-size: 0.85em; color: var(--text2); }

/* ═══════════════════════════════════════════════════
   TIPO 4: DOS COLUMNAS
═══════════════════════════════════════════════════ */
.slide-two-col { align-items: flex-start; }
.cols-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.2rem;
  width: 100%;
  position: relative; z-index: 1;
}
.col-card {
  background: var(--bg-glass);
  border: 1px solid var(--border);
  border-top: 3px solid var(--c1);
  border-radius: 0.8em;
  padding: 1.2em 1.4em;
  transition: border-color 0.3s, background 0.3s;
}
.col-card.c2 { border-top-color: var(--c2); }
.col-card.c3 { border-top-color: var(--c3); }
.col-card h3 {
  font-size: clamp(0.75rem, 1.4vw, 0.92rem);
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--c1);
  margin-bottom: 0.8em;
}
.col-card.c2 h3 { color: var(--c2); }
.col-card.c3 h3 { color: var(--c3); }
.col-card p, .col-card ul {
  font-size: clamp(0.75rem, 1.4vw, 0.9rem);
  color: var(--text2);
  line-height: 1.65;
}
.col-card ul { list-style: none; }
.col-card ul li { padding: 0.25em 0; border-bottom: 1px solid var(--border); }
.col-card ul li::before { content: "›  "; color: var(--c1); font-weight: 700; }
.col-card.c2 ul li::before { color: var(--c2); }
.col-card.c3 ul li::before { color: var(--c3); }
.col-card .big-num {
  font-size: clamp(2rem, 5vw, 3.5rem);
  font-weight: 900;
  color: var(--c1);
  line-height: 1;
  margin-bottom: 0.15em;
}
.col-card.c2 .big-num { color: var(--c2); }

/* ═══════════════════════════════════════════════════
   TIPO 5: KPIs
═══════════════════════════════════════════════════ */
.slide-kpi { align-items: flex-start; }
.kpi-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(160px, 22%), 1fr));
  gap: 1rem;
  width: 100%;
  position: relative; z-index: 1;
}
.kpi-card {
  padding: 1.2em 1em;
  border-radius: 0.8em;
  text-align: center;
  border: 1px solid var(--border);
  background: var(--bg-glass);
  position: relative;
  overflow: hidden;
  transition: transform 0.2s, background 0.3s;
}
.kpi-card::before {
  content: '';
  position: absolute;
  inset: 0;
  opacity: 0.07;
  transition: opacity 0.3s;
}
.kpi-card:hover { transform: translateY(-3px); }
.kpi-card:hover::before { opacity: 0.12; }
.kpi-card.c1::before { background: var(--c1); }
.kpi-card.c2::before { background: var(--c2); }
.kpi-card.c3::before { background: var(--c3); }
.kpi-card .kpi-num {
  font-size: clamp(2rem, 5vw, 3.8rem);
  font-weight: 900;
  line-height: 1;
  margin-bottom: 0.2em;
  position: relative;
}
.kpi-card.c1 .kpi-num { color: var(--c1); }
.kpi-card.c2 .kpi-num { color: var(--c2); }
.kpi-card.c3 .kpi-num { color: var(--c3); }
.kpi-card .kpi-label {
  font-size: clamp(0.65rem, 1.2vw, 0.82rem);
  color: var(--text2);
  text-transform: uppercase;
  letter-spacing: 0.08em;
  font-weight: 600;
  position: relative;
}
.kpi-card .kpi-delta {
  font-size: clamp(0.6rem, 1vw, 0.75rem);
  color: var(--c3);
  margin-top: 0.3em;
  position: relative;
}

/* ═══════════════════════════════════════════════════
   TIPO 6: TABLA
═══════════════════════════════════════════════════ */
.slide-table { align-items: flex-start; }
.data-table {
  width: 100%;
  border-collapse: collapse;
  font-size: clamp(0.72rem, 1.4vw, 0.9rem);
  position: relative; z-index: 1;
}
.data-table thead tr {
  background: color-mix(in srgb, var(--c1) 15%, transparent);
}
.data-table th {
  padding: 0.7em 1em;
  text-align: left;
  font-size: 0.8em;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--c1);
  border-bottom: 2px solid var(--c1);
}
.data-table td {
  padding: 0.65em 1em;
  color: var(--text2);
  border-bottom: 1px solid var(--border);
  transition: background 0.15s;
}
.data-table td strong { color: var(--text); }
.data-table tbody tr:nth-child(even) td { background: var(--bg-glass); }
.data-table tbody tr:hover td { background: color-mix(in srgb, var(--c1) 6%, transparent); }
.badge {
  display: inline-block;
  padding: 0.15em 0.6em;
  border-radius: 999px;
  font-size: 0.82em;
  font-weight: 600;
}
.badge.c1 { background: color-mix(in srgb, var(--c1) 15%, transparent); color: var(--c1); }
.badge.c2 { background: color-mix(in srgb, var(--c2) 15%, transparent); color: var(--c2); }
.badge.c3 { background: color-mix(in srgb, var(--c3) 15%, transparent); color: var(--c3); }

/* ═══════════════════════════════════════════════════
   TIPO 7: CIERRE / CTA
═══════════════════════════════════════════════════ */
.slide-closing {
  align-items: center;
  text-align: center;
}
.slide-closing::before {
  content: '';
  position: absolute;
  width: 70%;
  aspect-ratio: 1;
  border-radius: 50%;
  background: radial-gradient(circle,
    color-mix(in srgb, var(--c1) 20%, transparent),
    color-mix(in srgb, var(--c2) 10%, transparent) 50%,
    transparent 70%);
  pointer-events: none;
  transition: background 0.4s;
}
.slide-closing .closing-label {
  font-size: clamp(0.65rem, 1.2vw, 0.82rem);
  letter-spacing: 0.2em;
  text-transform: uppercase;
  color: var(--c2);
  font-weight: 600;
  margin-bottom: 0.8em;
  position: relative; z-index: 1;
}
.slide-closing .closing-title {
  font-size: clamp(2rem, 5vw, 4rem);
  font-weight: 900;
  line-height: 1.1;
  letter-spacing: -0.02em;
  max-width: 16ch;
  background: linear-gradient(135deg, var(--text) 30%, var(--c1) 70%, var(--c2));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 0.5em;
  position: relative; z-index: 1;
}
.slide-closing .closing-sub {
  font-size: clamp(0.82rem, 1.6vw, 1.05rem);
  color: var(--text2);
  max-width: 46ch;
  line-height: 1.65;
  margin-bottom: 2em;
  position: relative; z-index: 1;
}
.btn-cta {
  display: inline-flex;
  align-items: center;
  gap: 0.5em;
  background: var(--c1);
  color: #fff;
  border-radius: 999px;
  padding: 0.75em 2.2em;
  font-size: clamp(0.82rem, 1.5vw, 1rem);
  font-weight: 700;
  text-decoration: none;
  position: relative; z-index: 1;
  transition: opacity 0.2s, transform 0.2s, background 0.4s;
  box-shadow: 0 4px 24px color-mix(in srgb, var(--c1) 40%, transparent);
}
.btn-cta:hover { opacity: 0.88; transform: translateY(-2px); }
.closing-contact {
  margin-top: 2em;
  color: var(--text2);
  font-size: clamp(0.7rem, 1.2vw, 0.82rem);
  line-height: 2;
  position: relative; z-index: 1;
}
.closing-contact a { color: var(--c1); text-decoration: none; }
```

---

## BLOQUE E — CSS de controles de navegación (NO TOCAR)

```css
/* ═══════════════════════════════════════════════════
   BARRA DE CONTROL INFERIOR
═══════════════════════════════════════════════════ */
#controls {
  display: flex;
  align-items: center;
  gap: 0.8rem;
  padding: 0.5rem 1rem;
  margin-top: 0.4rem;
  width: min(100vw, calc(100vh * 16 / 9));
  justify-content: space-between;
}
.ctrl-btn {
  display: flex;
  align-items: center;
  gap: 0.35em;
  background: var(--bg-glass);
  border: 1px solid var(--border);
  color: var(--text2);
  border-radius: 0.5em;
  padding: 0.38em 0.9em;
  font-size: clamp(0.65rem, 1.2vw, 0.78rem);
  cursor: pointer;
  transition: background 0.15s, border-color 0.15s, color 0.15s;
  white-space: nowrap;
}
.ctrl-btn:hover:not(:disabled) {
  background: color-mix(in srgb, var(--c1) 12%, transparent);
  border-color: var(--c1);
  color: var(--c1);
}
.ctrl-btn:disabled { opacity: 0.25; cursor: not-allowed; }

/* Dots de progreso */
#dots {
  display: flex;
  align-items: center;
  gap: 0.35em;
  flex: 1;
  justify-content: center;
}
.dot {
  width: clamp(5px, 0.9vw, 8px);
  height: clamp(5px, 0.9vw, 8px);
  border-radius: 50%;
  background: var(--border);
  cursor: pointer;
  transition: background 0.25s, transform 0.25s;
  border: 1px solid transparent;
}
.dot.active {
  background: var(--c1);
  transform: scale(1.35);
}
.dot:hover:not(.active) {
  background: color-mix(in srgb, var(--c1) 40%, transparent);
}

/* Counter */
#counter {
  font-size: clamp(0.62rem, 1.1vw, 0.76rem);
  color: var(--text2);
  min-width: 4ch;
  text-align: center;
  font-variant-numeric: tabular-nums;
}

/* Toggle dark/light */
#btn-theme {
  display: flex;
  align-items: center;
  gap: 0.4em;
}
#btn-theme .icon { font-size: 1em; transition: transform 0.4s; }
[data-theme="light"] #btn-theme .icon { transform: rotate(180deg); }

/* ═══════════════════════════════════════════════════
   BARRA DE PROGRESO TOP
═══════════════════════════════════════════════════ */
#progress-bar {
  position: absolute;
  top: 0; left: 0;
  height: 3px;
  background: linear-gradient(90deg, var(--c1), var(--c2));
  transition: width 0.35s cubic-bezier(.4,0,.2,1), background 0.4s;
  z-index: 10;
}

/* ═══════════════════════════════════════════════════
   FULLSCREEN BUTTON (top-right)
═══════════════════════════════════════════════════ */
#btn-fs {
  position: fixed;
  top: 0.8rem;
  right: 0.8rem;
  background: var(--bg-glass);
  border: 1px solid var(--border);
  color: var(--text2);
  border-radius: 0.45em;
  padding: 0.3em 0.65em;
  font-size: clamp(0.62rem, 1vw, 0.75rem);
  cursor: pointer;
  z-index: 100;
  backdrop-filter: blur(8px);
  transition: background 0.15s, color 0.15s, border-color 0.15s;
}
#btn-fs:hover { color: var(--c1); border-color: var(--c1); }

/* ═══════════════════════════════════════════════════
   HINT
═══════════════════════════════════════════════════ */
#hint {
  position: fixed;
  bottom: 5rem;
  left: 50%;
  transform: translateX(-50%);
  font-size: clamp(0.6rem, 1vw, 0.72rem);
  color: var(--text2);
  pointer-events: none;
  transition: opacity 1.5s;
  white-space: nowrap;
  opacity: 0.6;
}
```

---

## Esqueleto HTML del body (NO TOCAR — solo el contenido interno de los .slide)

```html
<button id="btn-fs">⛶ FS</button>
<div id="hint">← → navegar &nbsp;·&nbsp; T modo &nbsp;·&nbsp; F pantalla completa</div>

<div id="presentation">
  <div id="stage">
    <div id="progress-bar"></div>

    <!-- ╔══════════════════════════════════════════╗
         ║  SLIDES — edita el contenido aquí        ║
         ╚══════════════════════════════════════════╝ -->

    <!-- 1 ─ PORTADA -->
    <div class="slide slide-cover active" data-n="01">
      <!-- ... contenido portada ... -->
    </div>

    <!-- 2..N ─ resto de slides -->
    <!-- data-n="02", "03", etc. -->

  </div><!-- /stage -->

  <!-- Controles — NO MODIFICAR -->
  <div id="controls">
    <button class="ctrl-btn" id="btn-prev">← Anterior</button>
    <div id="dots"></div>
    <span id="counter">1 / N</span>
    <button class="ctrl-btn" id="btn-next">Siguiente →</button>
    <button class="ctrl-btn" id="btn-theme"><span class="icon">☀︎</span> Modo</button>
  </div>
</div><!-- /presentation -->
```

**Notas:** el primer `.slide` lleva `class="slide slide-cover active"`. El resto solo `class="slide slide-TIPO"`. El atributo `data-n` es el número decorativo de fondo (formato `"01"`, `"02"`...). El contador `1 / N` en `#counter` se actualiza automáticamente por JS.

---

## Script de navegación completo (NO TOCAR — copiar íntegro)

```html
<script>
(() => {
  const slides    = Array.from(document.querySelectorAll('.slide'));
  const total     = slides.length;
  const btnPrev   = document.getElementById('btn-prev');
  const btnNext   = document.getElementById('btn-next');
  const btnTheme  = document.getElementById('btn-theme');
  const btnFs     = document.getElementById('btn-fs');
  const counter   = document.getElementById('counter');
  const dotsEl    = document.getElementById('dots');
  const bar       = document.getElementById('progress-bar');
  const hint      = document.getElementById('hint');
  const html      = document.documentElement;
  let current     = 0;
  let animating   = false;

  // ── Dots de progreso ──────────────────────────
  slides.forEach((_, i) => {
    const d = document.createElement('button');
    d.className = 'dot' + (i === 0 ? ' active' : '');
    d.setAttribute('aria-label', `Ir a slide ${i + 1}`);
    d.addEventListener('click', () => goTo(i));
    dotsEl.appendChild(d);
  });
  const dots = Array.from(dotsEl.querySelectorAll('.dot'));

  // ── Navegación ────────────────────────────────
  function goTo(idx) {
    if (animating || idx === current || idx < 0 || idx >= total) return;
    animating = true;

    const prev  = slides[current];
    const next  = slides[idx];
    const right = idx > current;

    // Position incoming
    next.style.transition = 'none';
    next.style.transform  = right ? 'translateX(80px) scale(0.98)' : 'translateX(-80px) scale(0.98)';
    next.style.opacity    = '0';
    next.classList.add('active');

    next.offsetHeight; // reflow

    prev.style.transition = '';
    prev.style.opacity    = '0';
    prev.style.transform  = right ? 'translateX(-80px) scale(0.98)' : 'translateX(80px) scale(0.98)';

    next.style.transition = '';
    next.style.opacity    = '1';
    next.style.transform  = 'translateX(0) scale(1)';

    setTimeout(() => {
      prev.classList.remove('active', 'exit-left');
      prev.style.transform = '';
      prev.style.opacity   = '';
      current   = idx;
      animating = false;
      updateUI();
      animateLists();
    }, 400);
  }

  function updateUI() {
    counter.textContent = `${current + 1} / ${total}`;
    btnPrev.disabled    = current === 0;
    btnNext.disabled    = current === total - 1;
    bar.style.width     = `${((current + 1) / total) * 100}%`;
    dots.forEach((d, i) => d.classList.toggle('active', i === current));
  }

  // ── Animación de listas ───────────────────────
  function animateLists() {
    const slide = slides[current];
    const items = slide.querySelectorAll('.list-items li');
    if (!items.length) return;
    items.forEach(li => li.classList.remove('visible'));
    items.forEach((li, i) =>
      setTimeout(() => li.classList.add('visible'), 100 + i * 110)
    );
  }

  btnPrev.addEventListener('click', () => goTo(current - 1));
  btnNext.addEventListener('click', () => goTo(current + 1));

  // ── Teclado ───────────────────────────────────
  document.addEventListener('keydown', e => {
    switch (e.key) {
      case 'ArrowRight': case 'ArrowDown': case ' ':
        e.preventDefault(); goTo(current + 1); break;
      case 'ArrowLeft':  case 'ArrowUp':
        e.preventDefault(); goTo(current - 1); break;
      case 'Home': e.preventDefault(); goTo(0);         break;
      case 'End':  e.preventDefault(); goTo(total - 1); break;
      case 'f': case 'F': toggleFs();    break;
      case 't': case 'T': toggleTheme(); break;
    }
  });

  // ── Swipe táctil ──────────────────────────────
  let tx = 0;
  document.addEventListener('touchstart', e => { tx = e.touches[0].clientX; }, { passive: true });
  document.addEventListener('touchend',   e => {
    const dx = e.changedTouches[0].clientX - tx;
    if (Math.abs(dx) > 50) goTo(current + (dx < 0 ? 1 : -1));
  }, { passive: true });

  // ── Toggle dark / light ───────────────────────
  function toggleTheme() {
    const isDark = html.dataset.theme === 'dark';
    html.dataset.theme = isDark ? 'light' : 'dark';
    try { localStorage.setItem('pres-theme', html.dataset.theme); } catch {}
  }
  btnTheme.addEventListener('click', toggleTheme);
  try {
    const saved = localStorage.getItem('pres-theme');
    if (saved) html.dataset.theme = saved;
  } catch {}

  // ── Fullscreen ────────────────────────────────
  function toggleFs() {
    if (!document.fullscreenElement) {
      document.documentElement.requestFullscreen().catch(() => {});
    } else {
      document.exitFullscreen().catch(() => {});
    }
  }
  btnFs.addEventListener('click', toggleFs);
  document.addEventListener('fullscreenchange', () => {
    btnFs.textContent = document.fullscreenElement ? '✕ Exit FS' : '⛶ FS';
  });

  // ── Init ──────────────────────────────────────
  setTimeout(() => { hint.style.opacity = '0'; }, 4500);
  updateUI();
  animateLists();
})();
</script>
```

---

## Tipos de slide disponibles y su HTML

### 1. `slide-cover` — Portada

```html
<div class="slide slide-cover active" data-n="01">
  <div class="overline">Categoría · Año</div>
  <h1>Título Principal</h1>
  <p class="subtitle">Descripción breve de la presentación en 1-2 líneas.</p>
  <div class="pills">
    <span class="pill c1">📍 Dato 1</span>
    <span class="pill c2">⏱ Dato 2</span>
    <span class="pill c3">👥 Dato 3</span>
    <span class="pill">🏷️ Dato 4</span>
  </div>
</div>
```

### 2. `slide-content` — Contenido con texto y quote

```html
<div class="slide slide-content" data-n="02">
  <h2>Título del slide</h2>
  <p class="body-text">
    Párrafo principal con <strong>texto destacado</strong> inline.
  </p>
  <div class="quote-block">
    "Cita o dato relevante que refuerza el mensaje."
  </div>
</div>
```

### 3. `slide-list` — Lista numerada con animación

```html
<div class="slide slide-list" data-n="03">
  <h2>Título del slide</h2>
  <ul class="list-items">
    <li>
      <span class="list-num">01</span>
      <div class="item-text">
        <strong>Título del ítem</strong>
        <span>Descripción corta del ítem</span>
      </div>
    </li>
    <!-- repetir para cada ítem -->
  </ul>
</div>
```

Los números rotan colores automáticamente: c1 → c2 → c3 → c1...

### 4. `slide-two-col` — Dos columnas / tarjetas

```html
<div class="slide slide-two-col" data-n="04">
  <h2>Título del slide</h2>
  <div class="cols-grid">
    <div class="col-card">          <!-- borde c1 -->
      <h3>Columna A</h3>
      <ul><li>ítem</li></ul>
    </div>
    <div class="col-card c2">       <!-- borde c2 -->
      <h3>Columna B</h3>
      <ul><li>ítem</li></ul>
    </div>
    <div class="col-card c3">       <!-- borde c3 -->
      <h3>Columna C</h3>
      <ul><li>ítem</li></ul>
    </div>
    <div class="col-card">
      <h3>Con número grande</h3>
      <div class="big-num">8.500€</div>
      <p>Descripción bajo el número.</p>
    </div>
  </div>
</div>
```

### 5. `slide-kpi` — Métricas / KPIs

```html
<div class="slide slide-kpi" data-n="05">
  <h2>Título del slide</h2>
  <div class="kpi-grid">
    <div class="kpi-card c1">
      <div class="kpi-num">+40%</div>
      <div class="kpi-label">Etiqueta métrica</div>
      <div class="kpi-delta">↑ Fuente / contexto</div>
    </div>
    <!-- repetir con c1, c2, c3 según necesidad -->
  </div>
</div>
```

### 6. `slide-table` — Tabla de datos

```html
<div class="slide slide-table" data-n="06">
  <h2>Título del slide</h2>
  <table class="data-table">
    <thead>
      <tr><th>#</th><th>Col A</th><th>Col B</th><th>Estado</th></tr>
    </thead>
    <tbody>
      <tr>
        <td><strong>01</strong></td>
        <td>Valor A</td>
        <td>Valor B</td>
        <td><span class="badge c3">Confirmado</span></td>
      </tr>
    </tbody>
  </table>
</div>
```

Badges disponibles: `.badge.c1` (primario), `.badge.c2` (alerta), `.badge.c3` (éxito).

### 7. `slide-closing` — Cierre / CTA

```html
<div class="slide slide-closing" data-n="07">
  <div class="closing-label">Siguiente paso</div>
  <div class="closing-title">Título de cierre impactante</div>
  <p class="closing-sub">
    Descripción breve del siguiente paso o propuesta de acción.
  </p>
  <a class="btn-cta" href="mailto:contacto@ejemplo.com">
    CTA principal →
  </a>
  <div class="closing-contact">
    📧 <a href="mailto:contacto@ejemplo.com">contacto@ejemplo.com</a>
    &nbsp;·&nbsp; 📞 +34 600 000 000
    &nbsp;·&nbsp; 🌐 ejemplo.com
  </div>
</div>
```

---

## Controles de navegación (referencia rápida)

| Tecla / Acción | Comportamiento |
|----------------|----------------|
| `← ↑` | Slide anterior |
| `→ ↓ Espacio` | Slide siguiente |
| `Home` | Primera diapositiva |
| `End` | Última diapositiva |
| `F` | Pantalla completa |
| `T` | Toggle dark/light |
| Click en dot | Navegar directamente al slide |
| Swipe táctil | Izquierda/derecha |

Barra inferior: `← Anterior · [●●○○○] · N/Total · Siguiente → · ☀︎ Modo`

---

## Patrones de uso (prompts de ejemplo)

### Propuesta comercial

```
Crea una presentación HTML de propuesta comercial para [Empresa] sobre
[tema]. 8 slides: portada, problema, propuesta de valor, programa
(4 módulos), metodología, KPIs esperados, inversión (2 cols), cierre CTA.
Colores de acento: violeta (#7c3aed), ámbar (#f59e0b), verde (#10b981).
Guárdala en docs/output/[nombre].html
```

### Presentación de formación / curso

```
Crea una presentación HTML para el módulo 1 del curso "[Curso]".
10 slides. Incluye: portada, objetivos, 5 conceptos (lista numerada),
ejercicio (2 cols), resumen, CTA.
Colores: azul (#3b82f6), naranja (#f97316), rosa (#ec4899).
Guárdala en docs/output/[nombre].html
```

### Informe ejecutivo / resultados

```
Crea una presentación HTML con el informe de resultados del Q1 2026.
Slides: portada, resumen ejecutivo, 4 KPIs, tabla de hitos,
dos columnas retos/logros, próximos pasos, cierre.
Datos: [datos concretos].
Colores: cian (#06b6d4), índigo (#6366f1), esmeralda (#10b981).
Guárdala en docs/output/[nombre].html
```

---

## Reglas de implementación (checklist)

1. **Archivo de salida:** `<project_root>/docs/output/<yyyyMMdd_nombre-kebab>.html` — resolver siempre a ruta absoluta
2. **Autocontenido:** todo CSS y JS inline en un único `.html`, sin CDN ni imports
3. **Solo editar:** BLOQUE A (colores) + contenido interno de `.slide`
4. **Orden de CSS en `<style>`:** A → B → C → D → E (exactamente en este orden)
5. **Primer slide:** debe incluir `active` en su clase: `class="slide slide-cover active"`
6. **`data-n`:** rellenar correlativamente `"01"`, `"02"`, ... en cada slide
7. **Script:** copiar el bloque JS completo sin modificaciones
8. **`#counter` inicial:** ajustar `1 / N` al número real de slides (el JS lo sobreescribe en `updateUI()`, pero es buena práctica tenerlo correcto en el HTML)
9. **Modo por defecto:** `<html lang="es" data-theme="dark">` — dark siempre por defecto
10. **No agregar JS adicional** que interfiera con la lógica de `goTo()` o `updateUI()`
