---
name: tool-generar-pdf
description: Genera un PDF de alta fidelidad a partir de una URL pública o un archivo HTML local usando Playwright + Chromium headless. Conserva el dark mode original, fuerza saltos de página por sección, mantiene fondo a sangre con padding interior uniforme en cada página. Invocar cuando el usuario pida "convertir a PDF", "exportar como PDF", "imprimir esta web a PDF", "PDF de un informe HTML", o cualquier flujo que necesite renderizar HTML→PDF preservando el diseño original (especialmente con fondos oscuros, hero sections o documentos largos paginables).
---

# tool-generar-pdf

Renderiza HTML → PDF con Playwright preservando el diseño original (incluido dark mode), partiendo cada `<section>` en su propia página.

## Cuándo usar

- "Exporta esta web a PDF" / "Conviértemelo en PDF"
- "PDF del informe / propuesta / presentación HTML"
- Documentos con fondo oscuro donde Chrome/imprimir-a-PDF estándar destroza la estética
- Informes con secciones (`<main> > <section>`) que se quieren paginar individualmente

## Entorno

```
Script:           /workspace/ClaudIA_Agent/skills/assets/pdf_gen/html_to_pdf.py
Venv estable:     /root/.local/venvs/pdf-gen/        (uv venv + playwright)
Chromium binary:  /root/.cache/ms-playwright/chromium-1208/chrome-linux64/chrome
Python intérprete:/root/.local/venvs/pdf-gen/bin/python
```

> No hace falta `playwright install` ni descargar browsers — el script apunta directamente al `chromium-1208` ya cacheado en disco. Si una versión futura de Playwright pide otro build, se actualiza la constante `CHROMIUM_PATH` del script.

## Uso básico

```bash
PY=/root/.local/venvs/pdf-gen/bin/python
SCRIPT=/workspace/ClaudIA_Agent/skills/assets/pdf_gen/html_to_pdf.py
DATE=$(date +%Y%m%d)

$PY $SCRIPT \
  /workspace/ClaudIA_Agent/docs/public/informe.html \
  /workspace/ClaudIA_Agent/docs/output/${DATE}_informe.pdf

# y según la regla de auto-publish, copiar a /docs/public
cp /workspace/ClaudIA_Agent/docs/output/${DATE}_informe.pdf /workspace/ClaudIA_Agent/docs/public/
```

Acepta URL (`https://...`), `file://...` o ruta absoluta — si es ruta, la convierte a `file://` automáticamente.

## Flags disponibles

| Flag | Default | Descripción |
|---|---|---|
| `--bg` | `#181818` | Color de fondo a fijar en `<html>` (sangrado en cada página) |
| `--fg` | `#ffffff` | Color de texto base que sobrescribe el `@media print` original |
| `--margin` | `12mm` | Padding uniforme en cada página (top/bottom/left/right) |
| `--media` | `print` | `print` (recomendado) o `screen`. `print` es necesario para que `break-before:page` funcione |
| `--no-section-breaks` | off | Desactiva el salto de página por `<section>` |
| `--wait-ms` | `1500` | Espera extra tras `networkidle` para fuentes/animaciones |

Ejemplo con tema claro y sin saltos forzados:

```bash
$PY $SCRIPT input.html out.pdf --bg "#ffffff" --fg "#111111" --no-section-breaks
```

## Auth: webs detrás de login

El dashboard `dashboard.marca-c.example.com/preview/` redirige a `/login` (`HTTP 302`). Para esos casos:

1. **Renderiza desde el archivo local** equivalente que vive en `/workspace/ClaudIA_Agent/docs/public/`
2. O añade autenticación al script (`page.context.add_cookies(...)`) — no implementado por defecto

## Decisiones técnicas (por qué este script funciona)

Los cuatro problemas clásicos de HTML→PDF con dark themes y por qué los resolvemos así:

### 1. El `@media print` del HTML invierte el fondo a blanco

Muchos documentos (incluido `informe_politica_ia_generativa.html`) tienen reglas tipo:

```css
@media print {
  body { background:#fff; color:#000 }
  h1,h2,h3,h4,p { color:#000 !important }
}
```

Esto en PDF deja **texto blanco sobre fondo blanco** = ilegible.

**Fix**: el script inyecta su propio bloque `@media print` con `!important` que sobrescribe esos colores tras `add_style_tag`. Como Playwright añade el `<style>` al final del `<head>`, gana en cascada.

### 2. El fondo oscuro deja franja blanca en los márgenes

Si pones `body { background: #181818 }` y `@page` tiene márgenes, las zonas de margen quedan blancas porque el bg de `body` solo cubre el área de contenido.

**Fix**: ponemos el background en `<html>`, no en body. Chromium con `print_background=true` pinta el background de `html` **a toda la página, incluida la zona de margen** (sangrado completo).

### 3. `padding` en `body` solo se ve en la primera página

`body { padding: 12mm }` solo aplica al inicio del flujo del body — tras un salto de página, el contenido pega contra el borde superior.

**Fix**: usamos `page.pdf(margin="12mm")` que es equivalente al `@page { margin: 12mm }` y se respeta **en cada página**. El body queda con `padding: 0`.

### 4. Las secciones se cortan a mitad o se mezclan en una página

`break-before: page` con `emulate_media("screen")` no se respeta de forma fiable en Chromium aunque la propiedad computada lo indique.

**Fix**: emulamos `print` (no `screen`) y compensamos los efectos visuales del `@media print` original (ver punto 1). En modo print, Chromium honra `break-before: page` consistentemente.

Reglas adicionales añadidas:
- `h2, h3 { break-after: avoid }` → un heading nunca queda colgado al final de página
- `table, figure, pre, blockquote { break-inside: avoid }` → estos bloques no se parten

## Verificación

`file` reporta el conteo de páginas de PDFs **mal** (lee `/Count` del primer objeto). Para confirmar páginas reales:

```bash
$PY -c "from pypdf import PdfReader; print(len(PdfReader('out.pdf').pages))"
```

`pypdf` está instalado en el venv `pdf-gen`.

## Output

Por convención, los PDFs van a:

```
docs/output/<yyyyMMdd>_<slug>.pdf
```

Y según la regla de auto-publish (memoria `feedback_auto_publish`), se copian inmediatamente a `docs/public/` para que el dashboard sirva la URL.
