---
name: content-pipeline
description: Pipeline autónomo de producción de contenido de extremo a extremo: investiga un tema, genera el artefacto (NotebookLM con fallback HTML), lo publica, envía el enlace por email, registra la ejecución y hace commit. Úsalo para producir y entregar contenido en un solo flujo.
---

# Skill: Content Pipeline — Research → Generate → Publish → Deliver

Pipeline autónomo de producción de contenido de extremo a extremo. Toma un tema y formato,
investiga, genera el artefacto vía NotebookLM (con fallback HTML), publica en el servidor de
archivos, envía el enlace por email, registra la ejecución y hace commit a git.

---

## Trigger phrases
- "crea una presentación sobre [tema]"
- "genera un informe de [tema] y envíamelo"
- "pipeline de contenido: [tema]"
- "investiga [tema] y genera [presentación/infografía/informe]"
- "/content-pipeline [tema]"
- "/pipeline [tema]"

---

## Parámetros

| Parámetro | Valores | Defecto | Descripción |
|-----------|---------|---------|-------------|
| `topic` | texto libre | — | Tema del contenido (**obligatorio**) |
| `format` | `presentation`, `infographic`, `report` | `presentation` | Tipo de artefacto a generar |
| `recipient` | email address | — | Destinatario del email (opcional) |
| `lang` | `es`, `en` | `es` | Idioma del contenido |
| `nb_id` | UUID NotebookLM | — | Notebook existente (opcional; si no se da, se crea uno nuevo) |

**Ejemplo de invocación:**
```
/content-pipeline "AI Agents for Small Business Automation in 2026" format=presentation recipient=contacto@marca-c.example.com lang=en
```

---

## Pipeline — 7 pasos con fallbacks

### PASO 1 — Investigación web

Buscar fuentes sobre el tema y extraer contenido clave.

```
# 5-8 búsquedas con variantes del tema
WebSearch("AI agents small business automation 2026")
WebSearch("autonomous AI workflows SMB use cases")
WebSearch("AI agent platforms small business tools 2026")
# ... adaptar según el tema

# Leer las 3-5 URLs más relevantes
WebFetch(url, prompt="Extract key facts, stats, trends, and insights about the topic")
```

Guardar el resultado consolidado en un archivo temporal:
```bash
cat > /tmp/pipeline_research_$(date +%s).md << 'RESEARCH'
# Research: {topic}

{consolidated research content in structured Markdown}
RESEARCH
RESEARCH_FILE=/tmp/pipeline_research_<timestamp>.md
```

El contenido debe incluir:
- Contexto y definición del tema
- Datos y estadísticas relevantes (con fuentes)
- Tendencias y proyecciones
- Casos de uso / ejemplos concretos
- Puntos clave para el artefacto final

---

### PASO 2 — Generación del artefacto

Ejecutar el script de pipeline con el contenido investigado:

```bash
cd /workspace/ClaudIA_Agent
source .venv/bin/activate 2>/dev/null || true

python scripts/content_pipeline.py \
  --topic "{topic}" \
  --format "{format}" \
  --content-file "$RESEARCH_FILE" \
  --lang "{lang}" \
  [--nb-id "{nb_id}"]
```

El script devuelve una línea JSON a stdout:
```json
{"ok": true, "url": "https://files.example.com/TU_TOKEN_DE_ACCESO/...", "local": "/workspace/.../docs/output/...", "fallback": false}
```

**Si el script falla:**
```bash
# Intentar con fallback HTML explícito
python scripts/content_pipeline.py \
  --topic "{topic}" \
  --format "{format}" \
  --content-file "$RESEARCH_FILE" \
  --lang "{lang}" \
  --no-notebooklm
```

---

### PASO 3 — Verificar publicación

El script escribe el artefacto directamente en `docs/public/`, que el contenedor nginx
(`root-files-1`) sirve en read-only bajo `/usr/share/nginx/html/TU_TOKEN_DE_ACCESO/`.
No hay paso de copia ni reinicio de nginx necesario.

Verificar que el archivo es accesible:

```bash
curl -sk -o /dev/null -w "%{http_code}" "{url}"
# Esperado: 200
```

Si devuelve 404: confirmar que el archivo existe en `docs/public/` y que el nombre en la URL coincide exactamente (tildes/caracteres en el slug pueden diferir).

---

### PASO 4 — Enviar email (si se especificó recipient)

**Primero crear borrador:**
```
mcp__outlook__outlook_create_draft(
  to: ["{recipient}"],
  subject: "{format_label}: {topic}",
  body: "Hola,\n\nAquí tienes {format_label} sobre '{topic}':\n\n{url}\n\nGenerado automáticamente por ClaudIA.\n\nSaludos"
)
```

Mostrar el borrador y confirmar con el usuario antes de enviar, salvo que la invocación sea completamente autónoma.

**Si el MCP de Outlook no está conectado:**
```
→ Verificar que Claude Code fue iniciado desde /workspace/ClaudIA_Agent/
→ Reiniciar Claude Code desde ese directorio y repetir el envío
→ Si sigue fallando: incluir URL en la respuesta al usuario para envío manual
```

---

### PASO 5 — Registrar en log de interacciones

Añadir entrada a `/workspace/ClaudIA_Agent/docs/logs/interactions.jsonl`:

```json
{"ts": "<ISO UTC>", "session_id": "<session>", "role": "assistant", "summary": "Content pipeline: generado {format} sobre '{topic}'. URL: {url}. Fallback: {true|false}.", "tools_used": ["WebSearch", "WebFetch", "Bash"], "files_modified": ["{local_path}"]}
```

---

### PASO 6 — Actualizar log del pipeline

El script escribe en `docs/logs/pipeline-runs.jsonl`. Verificar que el registro se guardó:

```bash
tail -1 /workspace/ClaudIA_Agent/docs/logs/pipeline-runs.jsonl
```

---

### PASO 7 — Git commit y push

```bash
cd /workspace/ClaudIA_Agent
git add docs/public/ docs/logs/
git commit -m "feat: content pipeline — {format} sobre {topic[:50]}"
git push origin main
```

---

## Fallback completo por paso

| Paso | Fallo posible | Fallback |
|------|---------------|----------|
| Investigación | URL no accesible | Continuar con otras fuentes; si todas fallan, generar contenido desde conocimiento general |
| NotebookLM auth | `NOTEBOOKLM_AUTH_JSON` no existe o expiró | El script genera HTML automáticamente (flag `--no-notebooklm`) |
| NotebookLM timeout | Generación supera 15 min | El script detecta timeout y genera HTML |
| Publicación | `docs/public/` no existe | El script lo crea con `mkdir -p` |
| Email MCP | Outlook no conectado | Informar URL al usuario; no bloquear el pipeline |
| Git push | Conflicto o red | `git pull --rebase && git push`; si falla, omitir push y avisar |

---

## Formatos de salida por tipo

| Format | NotebookLM | Fallback HTML | Extensión |
|--------|------------|---------------|-----------|
| `presentation` | Slide deck (PDF) | Presentación HTML navegable (← →) | `.pdf` / `.html` |
| `infographic` | Infografía (PNG) | Infografía HTML cards visuales | `.png` / `.html` |
| `report` | Briefing (PDF) | Informe HTML de lectura | `.pdf` / `.html` |

---

## URL pública resultante

```
{PUBLIC_FILES_URL}/{yyyyMMdd}_{slug}.{ext}
```

`PUBLIC_FILES_URL` se lee de `.env`. El contenedor nginx (`root-files-1`) sirve `docs/public/`
en read-only — no se requiere ninguna acción adicional para publicar.

Arquitectura de publicación:
```
docs/public/{file}  →  nginx bind mount  →  https://files.example.com/TU_TOKEN_DE_ACCESO/{file}
```

---

## Ejemplo completo — línea de comandos

```bash
# Desde Claude Code (skill invocation)
/content-pipeline "AI Agents for Small Business Automation in 2026" format=presentation recipient=contacto@marca-c.example.com lang=en

# Directo por terminal
cd /workspace/ClaudIA_Agent && source .venv/bin/activate
python scripts/content_pipeline.py \
  --topic "AI Agents for Small Business Automation in 2026" \
  --format presentation \
  --lang en \
  --no-git  # omitir commit si se hace manualmente
```

---

## Notas

- El script `scripts/content_pipeline.py` es el motor del pipeline; esta skill es el orquestador.
- Los artefactos se escriben directamente en `docs/public/` — el nginx los sirve sin pasos adicionales.
- `PUBLIC_FILES_URL` en `.env` controla la URL base — no hay valores hardcodeados en el script.
- NotebookLM requiere `NOTEBOOKLM_AUTH_JSON` en `.env`. Si no está o expira → HTML fallback automático.
- El log del pipeline se guarda en `docs/logs/pipeline-runs.jsonl` (junto con interactions.jsonl).
- La URL pública incluye un token de seguridad en el path — no publicar en repositorios públicos.
