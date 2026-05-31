# Skill: Catálogo de scripts del proyecto

Inventario de los scripts Python presentes en `/workspace/ClaudIA_Agent/scripts/`. **No es una skill ejecutable** — sirve como referencia rápida en `/docs` para saber qué hay disponible y cómo invocarlo.

## Trigger phrases
- "qué scripts tengo en el proyecto"
- "listar scripts"
- "/dev-scripts-catalog"

---

## Scripts del proyecto

### `content_pipeline.py` — Pipeline autónomo de contenido

Pipeline end-to-end: research → NotebookLM/HTML → publicar → email → git. Devuelve JSON estructurado con la URL pública.

```bash
python /workspace/ClaudIA_Agent/scripts/content_pipeline.py \
  --topic "..." --format presentation --content-file /tmp/research.md \
  --lang en [--no-notebooklm] [--no-git]
```

Output: `{"ok": true, "url": "https://...", "local": "...", "fallback": false}`

### `log-user-prompt.py` — Hook UserPromptSubmit

Captura cada mensaje del usuario en `docs/logs/interactions.jsonl`. Configurado en `~/.claude/settings.json`.

### `log-session-stop.py` — Hook Stop

Registra el cierre de cada turno/sesión en `docs/logs/interactions.jsonl`. Configurado en `~/.claude/settings.json`.

### `cleanup-interactions-log.py` — Cron diario

Limpia entradas antiguas del log (retención configurable). Programado en cron a las 03:00 AM.

```bash
crontab -l | grep cleanup
```

---

## Scripts de generación NotebookLM (uso puntual)

Scripts ad-hoc que demuestran patrones de uso de `notebooklm-py`. Útiles como plantillas para generar nuevos artefactos.

| Script | Genera | Notebook destino |
|--------|--------|------------------|
| `marca-b_video.py` | Vídeo Overview (EXPLAINER, ES) | IA ALMIA – MARCA B |
| `gen_video_marca-b.py` | Vídeo del Framework Marca B | IA ALMIA – MARCA B (`VIDEO_ID…`) |
| `marca-b_infografia.py` | Infografía visual del Framework | Marca B Development Memoria |
| `notebooklm_marca-b_visual.py` | Infografía visual + guardado en `docs/output/` | Marca B Development Memoria |
| `notebooklm_spec_ai.py` | Audio Deep Dive (Spec AI-Driven Development) | Notebook ad-hoc |

Todos siguen el patrón documentado en `skills/research-notebooklm.md` (auth con `storage_state.json` + polling manual con `poll_status` cada 10-15 s).

---

## Convenciones

- Scripts utilitarios (hooks, cron) tienen nombres `kebab-case.py`.
- Scripts de generación de contenido siguen `<topic>_<formato>.py`.
- Todos importan `OUTPUT_DIR` de `claudia.config` cuando escriben en `docs/output/`.
- Scripts ad-hoc deben moverse a `claudia/tools/` si se reutilizan más de 2 veces.

---

## Cómo añadir un script nuevo

1. Crearlo en `scripts/` con shebang `#!/usr/bin/env python3` y docstring inicial breve.
2. Si genera artefactos: usar `OUTPUT_DIR` y nombrar con la convención `yyyyMMdd_{titulo-kebab}.ext`.
3. Si es hook o cron: documentar el evento/cadencia en este fichero y en `infra-hooks-and-logging.md`.
4. Si es reutilizable: pensar si debería ser una skill (`skills/*.md`) en lugar de un script.
