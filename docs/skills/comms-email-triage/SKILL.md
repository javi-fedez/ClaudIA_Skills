---
name: comms-email-triage
description: Lee correos no leídos, filtra publicidad y newsletters, y genera un resumen accionable. Úsalo para revisar y priorizar la bandeja de entrada.
---

# Skill: Email Triage & Auto-Draft

Automatically reads unread emails, filters out ads and newsletters, and generates
reply drafts for emails that require a response.

## Trigger phrases
- "revisa mi correo"
- "procesa los correos nuevos"
- "genera borradores de respuesta"
- "triage de email"
- "/email-triage"

## Workflow

Execute these steps in order:

### Step 1 — List unread emails
Call `outlook_list_unread` with:
- `limit`: 50
- `folder`: "inbox"
- `response_format`: "json"

### Step 2 — Classify each email
For each email in the list, classify it as one of:
- **SKIP** — advertising, newsletter, promotional, notification, noreply, automated system message
- **DRAFT** — requires a human reply (question, request, follow-up, client/colleague email)

Classification rules (check in order):
1. SKIP if `from_email` contains: `noreply`, `no-reply`, `donotreply`, `notifications`, `mailer`, `newsletter`, `marketing`, `info@`, `updates@`, `alerts@`
2. SKIP if `subject` matches (case-insensitive): `unsubscribe`, `newsletter`, `oferta`, `descuento`, `promoción`, `sale`, `% off`, `deal`, `digest`, `weekly`, `monthly`, `resumen semanal`, `your receipt`, `invoice #`, `order confirmation`, `shipping notification`, `tracking`
3. SKIP if `bodyPreview` starts with typical automated phrases: `You are receiving this`, `This is an automated`, `Este es un mensaje automático`
4. DRAFT if the email comes from a known person (name visible in `from_name` and not a brand/company)
5. DRAFT if the subject is a direct question, reply, or action request

### Step 3 — Read full content of DRAFT emails
For each email classified as DRAFT:
- Call `outlook_read_email` with `email_id` and `include_html: false`
- Read the full body to understand context and tone

### Step 4 — Generate draft reply
For each DRAFT email, generate a professional reply in the **same language as the original email** that:
- Acknowledges the main point or question
- Provides a helpful, concise response or asks for clarification if needed
- Maintains a professional but warm tone
- Does NOT commit to specific dates/prices/decisions without user review
- Ends with an appropriate closing

Then call `outlook_create_draft` with:
- `reply_to_id`: the original email ID
- `body`: the generated reply text
- `body_type`: "text"
- `to`: leave empty (inherited from reply)

### Step 5 — Report results
After processing all emails, output a summary table:

```
## Email Triage Summary

| # | From | Subject | Classification | Draft |
|---|------|---------|---------------|-------|
| 1 | ... | ... | SKIP (newsletter) | — |
| 2 | ... | ... | DRAFT | ✅ created |

Processed: X emails | Skipped: Y | Drafts created: Z
```

## Notes
- Never send emails automatically — always create DRAFT, never call `outlook_send_email`
- If classification is ambiguous, default to DRAFT (better safe than skip)
- If the original email is a thread, use `outlook_get_thread` to get full context before drafting
- Always report which emails were skipped and why
