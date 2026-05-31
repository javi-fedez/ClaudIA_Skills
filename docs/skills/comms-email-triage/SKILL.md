---
name: comms-email-triage
description: Lee correos no leídos, filtra publicidad y newsletters, y genera borradores de respuesta solo para los que requieren atención humana. Úsalo para revisar y priorizar la bandeja de entrada. Nunca envía correos automáticamente.
---

# Skill: Triage de Correo y Auto-Borradores

Lee automáticamente los correos no leídos, filtra publicidad y newsletters, y genera
borradores de respuesta para los correos que requieren contestación.

---

## Trigger phrases
- "revisa mi correo"
- "procesa los correos nuevos"
- "genera borradores de respuesta"
- "triage de email"
- "/email-triage"

---

## Workflow

Ejecutar estos pasos en orden:

### Paso 1 — Listar correos no leídos
Llamar a `outlook_list_unread` con:
- `limit`: 50
- `folder`: "inbox"
- `response_format`: "json"

### Paso 2 — Clasificar cada correo
Para cada correo de la lista, clasificarlo como uno de:
- **SKIP** — publicidad, newsletter, promoción, notificación, noreply, mensaje automático del sistema
- **DRAFT** — requiere respuesta humana (pregunta, petición, seguimiento, correo de cliente/colega)

Reglas de clasificación (revisar en orden):
1. SKIP si `from_email` contiene: `noreply`, `no-reply`, `donotreply`, `notifications`, `mailer`, `newsletter`, `marketing`, `info@`, `updates@`, `alerts@`
2. SKIP si `subject` coincide (sin distinguir mayúsculas): `unsubscribe`, `newsletter`, `oferta`, `descuento`, `promoción`, `sale`, `% off`, `deal`, `digest`, `weekly`, `monthly`, `resumen semanal`, `your receipt`, `invoice #`, `order confirmation`, `shipping notification`, `tracking`
3. SKIP si `bodyPreview` empieza por frases automáticas típicas: `You are receiving this`, `This is an automated`, `Este es un mensaje automático`
4. DRAFT si el correo viene de una persona conocida (nombre visible en `from_name`, no una marca/empresa)
5. DRAFT si el asunto es una pregunta directa, una respuesta o una petición de acción

### Paso 3 — Leer el contenido completo de los correos DRAFT
Para cada correo clasificado como DRAFT:
- Llamar a `outlook_read_email` con `email_id` e `include_html: false`
- Leer el cuerpo completo para entender contexto y tono

### Paso 4 — Generar el borrador de respuesta
Para cada correo DRAFT, generar una respuesta profesional en **el mismo idioma que el correo original** que:
- Reconozca el punto principal o la pregunta
- Aporte una respuesta útil y concisa, o pida aclaración si hace falta
- Mantenga un tono profesional pero cercano
- NO comprometa fechas/precios/decisiones concretas sin revisión del usuario
- Termine con un cierre apropiado

Después llamar a `outlook_create_draft` con:
- `reply_to_id`: el ID del correo original
- `body`: el texto de la respuesta generada
- `body_type`: "text"
- `to`: dejar vacío (se hereda de la respuesta)

### Paso 5 — Reportar resultados
Tras procesar todos los correos, mostrar una tabla resumen:

```markdown
## Resumen de Triage de Correo

| # | De | Asunto | Clasificación | Borrador |
|---|------|---------|---------------|-------|
| 1 | ... | ... | SKIP (newsletter) | — |
| 2 | ... | ... | DRAFT | ✅ creado |

Procesados: X correos | Ignorados: Y | Borradores creados: Z
```

---

## Notas de seguridad
- **Nunca enviar correos automáticamente** — crear siempre un borrador (`outlook_create_draft`), nunca llamar a `outlook_send_email`.
- Si la clasificación es ambigua, optar por DRAFT (mejor prevenir que ignorar algo importante).
- Si el correo original es un hilo, usar `outlook_get_thread` para obtener todo el contexto antes de redactar.
- Reportar siempre qué correos se ignoraron y por qué.
