# Skill: Gestión de Agenda (MCP Calendar)

Skill completo para consultar, crear, modificar y eliminar eventos del calendario
Microsoft 365 a través del servidor MCP de Calendar, conectado a Microsoft Graph API.

---

## Trigger phrases
- "consulta mi agenda"
- "¿qué tengo esta semana?"
- "dime las citas de los próximos días"
- "crea una reunión con [persona]"
- "añade un evento el [fecha]"
- "mueve la reunión de [asunto]"
- "cancela el evento de [asunto]"
- "¿quién ha aceptado la reunión?"
- "/calendar"

---

## Herramientas disponibles (MCP Calendar)

El servidor MCP expone **6 herramientas** vía `mcp-servers/calendar/`:

| Herramienta | Acción | Modifica datos |
|-------------|--------|----------------|
| `calendar_list_calendars` | Lista todos los calendarios disponibles | No |
| `calendar_list_events` | Lista eventos en un rango de fechas | No |
| `calendar_get_event` | Obtiene detalles completos de un evento por ID | No |
| `calendar_create_event` | Crea un nuevo evento | Sí |
| `calendar_update_event` | Modifica un evento existente | Sí |
| `calendar_delete_event` | Elimina permanentemente un evento | Sí ⚠️ |

> **Regla de oro:** Antes de crear, modificar o eliminar un evento, mostrar al usuario
> lo que se va a hacer y esperar confirmación explícita.
> `calendar_delete_event` es irreversible — siempre confirmar.

---

## Calendarios del usuario (hola@sitio-1.example.com)

| Nombre | ID | Editable |
|--------|----|----------|
| Calendario *(principal)* | `AQMkAGFlMDdk...KQSwAAAA==` | ✅ |
| Días festivos de España | `AQMkAGFlMDdk...KQTAAAAA==` | Solo lectura |
| Cumpleaños | `AQMkAGFlMDdk...KQTwAAAA==` | Solo lectura |

> Omitir `calendar_id` para operar sobre el calendario principal.
> Usar el ID completo solo cuando se necesite acceder a un calendario secundario.

---

## Workflows por caso de uso

### 1. Listar todos mis calendarios

```
calendar_list_calendars()
```

Devuelve: ID, nombre, color, si es el predeterminado, permisos de edición.

---

### 2. Consultar agenda de los próximos días / semana

```
calendar_list_events(
  start_date: "2026-03-14",
  end_date:   "2026-03-21",
  limit: 50,
  response_format: "markdown"
)
```

Devuelve: asunto, fecha/hora inicio y fin, ubicación, organizador, estado RSVP.

**Variantes frecuentes:**
```
// Esta semana
start_date: "2026-03-09", end_date: "2026-03-15"

// Este mes
start_date: "2026-03-01", end_date: "2026-03-31"

// Próximos 7 días desde hoy
start_date: "<hoy>", end_date: "<hoy + 7 días>"

// Un calendario específico (p.ej. festivos)
calendar_id: "AQMkAGFlMDdk...KQTAAAAA=="
```

---

### 3. Ver detalles de un evento

Primero obtener el `id` del evento desde `calendar_list_events`, luego:

```
calendar_get_event(
  event_id: "AAMkAGFlMDdk..."
)
```

Devuelve: todos los asistentes con su estado RSVP (aceptado/pendiente/rechazado),
descripción completa, enlace de Teams, recurrencia.

---

### 4. Crear un evento nuevo

```
calendar_create_event(
  subject:          "Reunión propuesta formación IA",
  start_datetime:   "2026-03-20T10:00:00",
  end_datetime:     "2026-03-20T11:00:00",
  timezone:         "Europe/Madrid",
  location:         "Google Meet / Presencial Madrid",
  body:             "Revisión de propuesta de formación con el cliente.",
  body_type:        "text",
  attendees:        ["cliente@empresa.com"],
  is_online_meeting: true        // añade enlace Teams automáticamente
)
```

**Campos requeridos:** `subject`, `start_datetime`, `end_datetime`
**Zona horaria por defecto:** `Europe/Madrid`

---

### 5. Modificar un evento existente

Solo incluir los campos que cambian — el resto no se toca:

```
// Cambiar hora
calendar_update_event(
  event_id:       "AAMkAGFlMDdk...",
  start_datetime: "2026-03-20T11:00:00",
  end_datetime:   "2026-03-20T12:00:00",
  timezone:       "Europe/Madrid"
)

// Cambiar título y descripción
calendar_update_event(
  event_id: "AAMkAGFlMDdk...",
  subject:  "Reunión de seguimiento Biogen",
  body:     "Sesión de seguimiento post-taller."
)

// Añadir asistente (pasar lista completa — reemplaza la existente)
calendar_update_event(
  event_id:  "AAMkAGFlMDdk...",
  attendees: ["ya.estaba@empresa.com", "nuevo@empresa.com"]
)
```

---

### 6. Eliminar un evento

> ⚠️ Irreversible. Si eres el organizador, se elimina del calendario de todos los asistentes.

```
calendar_delete_event(
  event_id: "AAMkAGFlMDdk..."
)
```

Siempre confirmar con el usuario antes de ejecutar esta herramienta.

---

## Workflow completo: Resumen diario de agenda

Para obtener el día/semana de un vistazo:

### Paso 1 — Listar eventos del período
```
calendar_list_events(
  start_date: "<hoy>",
  end_date:   "<fin de semana>",
  response_format: "json"
)
```

### Paso 2 — Enriquecer con festivos
Consultar también el calendario "Días festivos de España":
```
calendar_list_events(
  start_date:  "<hoy>",
  end_date:    "<fin de período>",
  calendar_id: "<id festivos>",
  response_format: "json"
)
```

### Paso 3 — Presentar en tabla combinada
```markdown
## Agenda — semana del 14 al 21 mar 2026

| Día | Hora | Evento | Lugar |
|-----|------|--------|-------|
| Lun 16 | 10:00–11:00 | Reunión Biogen | Teams |
| Mié 18 | 15:00–16:00 | Clase Vibe Coding II | The Valley |
| Jue 19 | 🗓️ Todo el día | Día de San José (festivo) | — |
```

---

## Workflow completo: Crear evento desde un correo

Cuando el usuario quiere agendar una reunión a partir de un email:

1. Leer el correo con `outlook_read_email` para extraer: participantes, asunto, fecha propuesta.
2. Confirmar con el usuario los datos del evento.
3. Crear con `calendar_create_event` incluyendo los asistentes del correo.
4. Opcionalmente, crear borrador de confirmación con `outlook_create_draft`.

---

## Llamada directa a Graph API (avanzada)

Para casos no cubiertos por las herramientas MCP (p.ej. buscar en todos los calendarios):

```js
import { createGraphClient } from './mcp-servers/calendar/dist/graphClient.js';

const client = await createGraphClient();

// Vista unificada de todos los calendarios en un rango
const res = await client.get(
  `/me/calendarView` +
  `?startDateTime=2026-03-14T00:00:00Z` +
  `&endDateTime=2026-04-14T00:00:00Z` +
  `&$top=50` +
  `&$orderby=start/dateTime` +
  `&$select=subject,start,end,location,isAllDay,organizer,bodyPreview`
);
console.log(res.data.value);

// Eventos de un calendario específico (festivos, cumpleaños)
const festivos = await client.get(
  `/me/calendars/${encodeURIComponent(calendarId)}/calendarView` +
  `?startDateTime=...&endDateTime=...&$top=20&$select=subject,start,end,isAllDay`
);
```

> Usar Graph API directamente cuando se necesite combinar múltiples calendarios
> en una sola consulta o aplicar filtros no soportados por las herramientas MCP.

---

## Autenticación — Setup inicial

Comparte el token OAuth2 con el servidor MCP de Outlook (misma app Azure).

```bash
cd mcp-servers/calendar
npm run auth
```

Token almacenado en `mcp-servers/calendar/token-cache.json`.

### Variables de entorno requeridas (en `.env`)
```
AZURE_CLIENT_ID=<id de la App Registration en Azure>
AZURE_TENANT_ID=<id del tenant de M365>
AZURE_CLIENT_SECRET=<secreto de la app>
```

### Permisos de Microsoft Graph necesarios (delegados)

| Permiso | Para qué |
|---------|----------|
| `Calendars.Read` | Leer eventos y calendarios |
| `Calendars.ReadWrite` | Crear, modificar y eliminar eventos |
| `offline_access` | Renovar tokens automáticamente |

---

## Notas importantes

- **Zona horaria:** Usar siempre `Europe/Madrid` salvo indicación contraria.
- **Formato de fechas:** ISO 8601 — `2026-03-20T10:00:00` (sin Z para hora local).
- **`calendar_delete_event` es irreversible** — siempre pedir confirmación.
- **`calendar_update_event` reemplaza la lista de asistentes completa** — pasar siempre todos los emails.
- **Agenda principal vacía ≠ sin eventos** — recordar consultar también calendarios secundarios (festivos, cumpleaños).
- **IDs de evento** empiezan por `AAMk` (~150 chars). Obtener siempre de `calendar_list_events`.
