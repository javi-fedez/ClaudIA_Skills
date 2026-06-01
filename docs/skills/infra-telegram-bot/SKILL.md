---
name: infra-telegram-bot
description: Documentación de referencia del bot de Telegram que conecta con Claude Code en el VPS. NO es una skill ejecutable. Úsalo para consultar el estado o la configuración del bot.
---

# Skill: Telegram Bot — Infraestructura VPS

Documentación de referencia del bot de Telegram que conecta con Claude Code en el VPS. **No es una skill ejecutable** — sirve para que el componente aparezca en el dashboard `/docs`.

## Trigger phrases
- "estado del bot Telegram"
- "reiniciar bot"
- "logs del bot"
- "configuración del bot"

---

## Qué es

Bot Telegram que actúa como **puente entre Telegram y Claude Code**. Permite usar ClaudIA desde cualquier lugar mediante texto o notas de voz. Vive **fuera del repo** en `/opt/telegram-claude-bot/` (infraestructura del VPS) y corre como servicio PM2 bajo usuario `root`.

## Características

- **Texto** → Claude Code con contexto de conversación (8 turnos · 30 min TTL)
- **Voz** → transcripción local con `whisper.cpp` → Claude Code
- **Un único usuario autorizado** (`TELEGRAM_USER_ID`)
- **Modo sin prompts:** `--permission-mode acceptEdits` (compatible con root)
- **Heredeo de entorno** → `HOME=/root`, `USER=root` (sin `uid`/`gid`)
- **Gestión PM2** (no systemd)

## Comandos PM2

```bash
pm2 status                              # ver estado
pm2 restart claudia-bot --update-env    # reiniciar con env actualizado
pm2 logs claudia-bot --lines 50         # ver logs recientes
pm2 logs claudia-bot --lines 5 --nostream  # últimas 5 líneas sin tail
pm2 monit                               # monitorización en tiempo real
```

## Variables de entorno (PM2 / `ecosystem.config.js`)

| Variable | Descripción |
|---|---|
| `TELEGRAM_BOT_TOKEN` | Token de @BotFather |
| `TELEGRAM_USER_ID` | Tu Telegram user ID (único usuario permitido) |
| `CLAUDE_CODE_OAUTH_TOKEN` | Token OAuth de Claude Code |
| `CLAUDE_SKIP_PERMISSIONS` | `'true'` para `--permission-mode acceptEdits` |

## Notas críticas de configuración

- **`HOME: '/root'` es obligatorio** — con cualquier otro HOME, Claude Code no encuentra `~/.claude/`, arranca sin MCPs, sin memoria y sin permisos pre-autorizados.
- **No usar `uid`/`gid`** en el `spawn` — Claude debe correr como el mismo usuario del proceso padre (root).
- **`bypassPermissions` está bloqueado en root** por Claude Code — usar `acceptEdits` (suficiente para uso en bot personal: lee, escribe y ejecuta sin prompts).
- **`cwd: '/workspace/ClaudIA_Agent'`** para que Claude cargue el `.mcp.json` y `CLAUDE.md` del proyecto.

## Arquitectura del bot (`bot.js`)

```js
// ✅ CORRECTO — Claude hereda entorno root con config completa
const child = spawn('claude', claudeArgs, {
  cwd: '/workspace/ClaudIA_Agent',
  env: { ...process.env, HOME: '/root', USER: 'root' }
  // SIN uid/gid — hereda root del proceso padre
});

if (SKIP_PERMISSIONS) claudeArgs.push('--permission-mode', 'acceptEdits');
```

## Whisper.cpp para voz

- Modelo local (no usa API externa).
- Transcribe el audio enviado por Telegram → texto → se inyecta en Claude Code igual que un mensaje de texto.
- No requiere claves de API ni costes recurrentes.

## Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| Bot responde sin acceder a MCPs | `HOME` distinto de `/root` | Revisar `ecosystem.config.js` |
| Prompts de permiso bloqueando comandos | Falta `acceptEdits` o `uid` añadido | Eliminar `uid`/`gid`, añadir `--permission-mode acceptEdits` |
| Bot no arranca tras reiniciar VPS | PM2 no persistido | `pm2 save && pm2 startup` |
| Voz no se transcribe | `whisper.cpp` mal compilado | Recompilar con `make` en directorio whisper.cpp |
