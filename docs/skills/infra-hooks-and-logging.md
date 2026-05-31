# Skill: Hooks & Interaction Logging

Documentación del sistema de hooks de Claude Code que registra **toda interacción** del usuario y el cierre de cada sesión a `docs/logs/interactions.jsonl`. Configurado globalmente en `~/.claude/settings.json` y aplicado a todas las instancias (CLI, Telegram bot, dashboard chat).

## Trigger phrases
- "ver logs de interacciones"
- "limpiar logs antiguos"
- "estado del cron de logs"
- "configurar hook"

---

## Hooks activos

| Hook | Script | Cuándo se ejecuta |
|---|---|---|
| `UserPromptSubmit` | `scripts/log-user-prompt.py` | Cada vez que el usuario envía un mensaje |
| `Stop` | `scripts/log-session-stop.py` | Al finalizar cada turno/sesión |

Configurados en `~/.claude/settings.json`:

```json
"hooks": {
  "UserPromptSubmit": [{
    "matcher": "",
    "hooks": [{ "type": "command", "command": "python3 /workspace/ClaudIA_Agent/scripts/log-user-prompt.py" }]
  }],
  "Stop": [{
    "matcher": "",
    "hooks": [{ "type": "command", "command": "python3 /workspace/ClaudIA_Agent/scripts/log-session-stop.py" }]
  }]
}
```

## Regla CLAUDE.md complementaria

Además de los hooks automáticos, `CLAUDE.md` obliga a Claude a registrar **un resumen de cada respuesta** al final de cada turno con esta estructura JSONL:

```json
{"ts": "<ISO UTC>", "session_id": "<id>", "role": "assistant", "summary": "<resumen 1-2 frases>", "tools_used": ["Read","Edit"], "files_modified": ["path1"]}
```

Esto es **mandatory** y silencioso — el usuario nunca tiene que pedirlo.

## Formato del log (`docs/logs/interactions.jsonl`)

Cada línea es un evento independiente:

```jsonl
{"ts": "2026-04-29T09:12:37Z", "session_id": "abc123", "role": "user", "content": "..."}
{"ts": "2026-04-29T09:12:40Z", "session_id": "abc123", "role": "assistant", "summary": "...", "tools_used": ["Read","Edit"], "files_modified": [...]}
{"ts": "2026-04-29T09:12:41Z", "session_id": "abc123", "role": "session_end", "stop_reason": "end_turn"}
```

## Cron de limpieza diaria

Tarea cron que rota el log eliminando entradas con más antigüedad que la retención configurada.

| Componente | Detalle |
|---|---|
| Script | `scripts/cleanup-interactions-log.py` |
| Cadencia | Diaria a las 03:00 AM |
| Retención | Configurable (CLAUDE.md indica 180 días) |
| Acción | Reescribe el JSONL filtrando entradas antiguas |

## Comandos útiles

```bash
# Tail del log en vivo
tail -f /workspace/ClaudIA_Agent/docs/logs/interactions.jsonl

# Contar interacciones por sesión
jq -r .session_id /workspace/ClaudIA_Agent/docs/logs/interactions.jsonl | sort | uniq -c

# Ver últimos resúmenes de Claude
jq -r 'select(.role=="assistant") | "\(.ts) - \(.summary)"' \
  /workspace/ClaudIA_Agent/docs/logs/interactions.jsonl | tail -20

# Forzar limpieza manual
python3 /workspace/ClaudIA_Agent/scripts/cleanup-interactions-log.py

# Ver el cron registrado
crontab -l | grep cleanup
```

## Dashboard

El tab "Logs" de `dashboard.marca-c.example.com` lee este mismo fichero y muestra los eventos de forma navegable.

## Troubleshooting

| Síntoma | Causa | Acción |
|---|---|---|
| Hook no se dispara | settings.json mal formado | `claude doctor` o validar JSON |
| Log crece sin límite | Cron no se ejecuta | Verificar `crontab -l` y permisos del script |
| Resúmenes faltan | Claude olvida la regla CLAUDE.md | Recordarle expresamente; revisar carga del CLAUDE.md |
| Permisos denegados | Hook corre como otro user | Asegurar `HOME=/root` en el spawn |
