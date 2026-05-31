# Skill: Gestión de Correo Outlook (MCP)

Skill completo para consultar, buscar, leer, contestar y enviar correos electrónicos
a través del servidor MCP de Outlook 365, conectado a Microsoft Graph API.

---

## Trigger phrases
- "consulta mis correos"
- "lista los correos no leídos"
- "busca correos de [remitente]"
- "lee el correo de [asunto]"
- "contesta el correo de [persona]"
- "redacta un borrador para [destinatario]"
- "envía un correo a [destinatario]"
- "dame las tareas pendientes del correo"
- "triage de correo"
- "/outlook-email"

---

## Herramientas disponibles (MCP Outlook)

El servidor MCP expone **6 herramientas** vía `mcp-servers/outlook/`:

| Herramienta | Acción | Modifica datos |
|-------------|--------|----------------|
| `outlook_list_unread` | Lista correos NO leídos de una carpeta | No |
| `outlook_search_emails` | Busca correos por texto, remitente, asunto o fechas | No |
| `outlook_read_email` | Lee el cuerpo completo de un correo por ID | No |
| `outlook_get_thread` | Obtiene todos los mensajes de un hilo | No |
| `outlook_create_draft` | Crea un borrador (NO lo envía) | Sí |
| `outlook_send_email` | Envía un correo nuevo o un borrador existente | Sí ⚠️ |

> **Regla de oro:** Nunca llamar a `outlook_send_email` sin confirmación explícita del usuario.
> Siempre crear primero un borrador con `outlook_create_draft` y mostrar el contenido para revisión.

---

## Workflows por caso de uso

### 1. Listar correos recientes (no leídos)

```
outlook_list_unread(
  limit: 20,           // máximo 50
  folder: "inbox",     // inbox | sentitems | drafts | deleteditems | junkemail
  response_format: "json"
)
```

**Cuándo usarlo:** El usuario quiere ver qué correos nuevos tiene.

**Limitación:** Solo devuelve correos NO leídos. Para ver todos (leídos y no leídos)
usar la llamada directa a Graph API (ver sección avanzada al final).

---

### 2. Buscar correos por criterio

```
// Por texto libre (busca en asunto, cuerpo y remitente)
outlook_search_emails(query: "propuesta formación", limit: 10)

// Por remitente
outlook_search_emails(from: "cliente@empresa.com", limit: 20)

// Por asunto
outlook_search_emails(subject: "factura", limit: 10)

// Por rango de fechas
outlook_search_emails(after: "2026-03-01", before: "2026-03-14", limit: 30)

// No leídos con adjuntos
outlook_search_emails(unread_only: true, has_attachments: true, limit: 20)
```

> ⚠️ `query` es incompatible con `from`, `subject` y filtros de fecha (limitación de Graph API).
> Usar uno u otro, nunca ambos en la misma llamada.

---

### 3. Leer el contenido completo de un correo

Primero obtener el `id` del correo (desde list o search), luego:

```
outlook_read_email(
  email_id: "AAMkAGI2...",   // ID del correo
  include_html: false,        // true si necesitas ver el formato HTML
  response_format: "markdown"
)
```

Devuelve: remitente, destinatarios, CC, fecha, cuerpo completo, adjuntos, importancia.

---

### 4. Leer un hilo completo (conversación)

Obtener el `conversationId` desde `outlook_read_email` o `outlook_search_emails`, luego:

```
outlook_get_thread(
  conversation_id: "AAQkAGI2...",
  limit: 20,
  response_format: "markdown"
)
```

Devuelve todos los mensajes del hilo ordenados cronológicamente (más antiguo primero).
Ideal para: resumir una conversación, extraer compromisos, entender contexto antes de responder.

---

### 5. Contestar un correo (crear borrador de respuesta)

```
outlook_create_draft(
  reply_to_id: "AAMkAGI2...",   // ID del correo original
  body: "Hola Juan,\n\nGracias por tu mensaje...",
  body_type: "text"             // text | html
)
```

Devuelve el `draft_id` y un enlace para abrir en Outlook.
**No envía el correo.** Siempre mostrar el borrador al usuario antes de enviar.

---

### 6. Crear un correo nuevo (borrador)

```
outlook_create_draft(
  to: ["destinatario@empresa.com"],
  cc: ["copia@empresa.com"],         // opcional
  bcc: ["copiaOculta@empresa.com"],  // opcional
  subject: "Propuesta de formación en IA",
  body: "Estimado cliente,\n\n...",
  body_type: "text"
)
```

---

### 7. Enviar un borrador existente

Solo después de confirmación explícita del usuario:

```
outlook_send_email(
  draft_id: "AAMkAGI2..."   // ID obtenido al crear el borrador
)
```

---

### 8. Enviar correo directo (sin borrador previo)

Solo si el usuario indica explícitamente que quiere enviarlo directamente:

```
outlook_send_email(
  to: ["destinatario@empresa.com"],
  subject: "Confirmación de reunión",
  body: "Confirmo la reunión para el jueves a las 10h.",
  body_type: "text",
  save_to_sent: true
)
```

---

## Workflow completo: Tareas pendientes del correo

Este es el flujo para obtener tareas pendientes a partir del correo (como en el ejemplo del 14/03/2026):

### Paso 1 — Obtener correos recientes (leídos + no leídos)

Usar llamada directa a Graph API desde Node.js, ya que `outlook_list_unread` solo devuelve no leídos:

```js
// Ejecutar en Node.js con el GraphClient del MCP server
import { createGraphClient } from './mcp-servers/outlook/dist/graphClient.js';

const client = await createGraphClient();
const res = await client.get(
  '/me/mailFolders/inbox/messages' +
  '?$top=25' +
  '&$orderby=receivedDateTime DESC' +
  '&$select=subject,from,receivedDateTime,isRead,importance,bodyPreview'
);
console.log(JSON.stringify(res.data.value, null, 2));
```

### Paso 2 — Filtrar correos relevantes

Clasificar cada correo como:
- **ACCIÓN** — requiere respuesta o seguimiento (preguntas, propuestas, clientes)
- **SKIP** — newsletter, notificación automática, confirmación de sistema

Criterios de SKIP (revisar en orden):
1. `from` contiene: `noreply`, `no-reply`, `donotreply`, `notifications`, `customerservice@`, `no-reply@sharepointonline`
2. `subject` contiene: `newsletter`, `oferta`, `promoción`, `digest`, `tu presentación`, `grabación`, `QR de encuesta`
3. `bodyPreview` empieza por frases automáticas: `Este es un mensaje automático`, `You are receiving this`

### Paso 3 — Extraer la tarea pendiente de cada correo ACCIÓN

Para cada correo relevante, determinar:
- **¿Qué acción requiere?** (responder, enviar factura, preparar material, hacer seguimiento)
- **¿Qué información falta?** (precio, fecha, datos de facturación)
- **¿Tiene urgencia?** (mención de plazo, correo sin leer, seguimiento sin respuesta)

### Paso 4 — Presentar las 9 más importantes

Ordenar por prioridad (importancia + urgencia + valor económico) y presentar en tabla:

```markdown
| # | Asunto | Acción | Remitente |
|---|--------|--------|-----------|
| 1 | **[Asunto más urgente]** | [Acción concreta] | [email] |
...
```

---

## Workflow completo: Triage y respuesta automática

### Paso 1 — Listar no leídos
```
outlook_list_unread(limit: 50, folder: "inbox", response_format: "json")
```

### Paso 2 — Clasificar (SKIP / DRAFT)
Ver reglas en sección anterior.

### Paso 3 — Leer correos a responder
```
outlook_read_email(email_id: "<id>", include_html: false)
```
Si es un hilo con contexto importante:
```
outlook_get_thread(conversation_id: "<conversationId>")
```

### Paso 4 — Generar borrador de respuesta
Redactar en el mismo idioma que el correo original. Tono profesional y cercano.
No comprometer fechas, precios ni decisiones sin revisión del usuario.

```
outlook_create_draft(
  reply_to_id: "<id>",
  body: "<respuesta generada>",
  body_type: "text"
)
```

### Paso 5 — Mostrar resumen
```
## Triage de Correo — [fecha]

| # | Remitente | Asunto | Resultado |
|---|-----------|--------|-----------|
| 1 | ...       | ...    | SKIP (newsletter) |
| 2 | ...       | ...    | ✅ Borrador creado |

Procesados: X | Ignorados: Y | Borradores: Z
```

---

## Autenticación — Setup inicial

El servidor MCP usa **OAuth2 Device Code Flow** con Microsoft Graph.
Solo es necesario autenticarse una vez; el token se almacena en caché.

```bash
# Desde el directorio del servidor MCP
cd mcp-servers/outlook
npm run auth
```

Sigue el enlace que aparece en pantalla, introduce el código en `microsoft.com/devicelogin`
y autoriza la app con tu cuenta Microsoft 365.

El token se guarda en `mcp-servers/outlook/token-cache.json`.
Si el token expira, el servidor devuelve el error:
`"Authentication expired. Please run 'npm run auth'"`.

### Variables de entorno requeridas (en `.env`)

```
AZURE_CLIENT_ID=<id de la App Registration en Azure>
AZURE_TENANT_ID=<id del tenant de M365>
AZURE_CLIENT_SECRET=<secreto de la app>
```

### Permisos de Microsoft Graph necesarios (delegados)

| Permiso | Para qué |
|---------|----------|
| `Mail.Read` | Leer correos |
| `Mail.ReadWrite` | Crear borradores |
| `Mail.Send` | Enviar correos |
| `offline_access` | Renovar tokens automáticamente |

---

## Notas importantes

- **Nunca enviar sin confirmación.** Siempre crear borrador primero.
- **Los IDs de correo son largos** (~150 chars, empiezan por `AAMk`). Copiar siempre del resultado de list/search.
- **`conversationId` ≠ `email_id`.** Usar el correcto según la herramienta.
- **`query` vs filtros específicos:** Son mutuamente excluyentes en `outlook_search_emails`.
- **Para correos leídos + no leídos:** Usar Graph API directamente (ver sección avanzada).
- **Carpetas disponibles:** `inbox`, `sentitems`, `drafts`, `deleteditems`, `junkemail`.
- **Idioma:** Responder siempre en el mismo idioma que el correo original.
