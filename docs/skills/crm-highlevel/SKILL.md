---
name: crm-highlevel
description: Gestiona el CRM de GoHighLevel (Lead Connector) vía el MCP lead-connector: contactos, oportunidades y pipelines. Úsalo para operar el CRM. Requiere GHL_API_TOKEN en .env.
---

# crm-highlevel — CRM GoHighLevel / Lead Connector

Gestiona el CRM de GoHighLevel (Lead Connector) mediante el MCP `lead-connector`.
Usa las herramientas `mcp__lead-connector__*` disponibles en esta sesión.

---

## Autenticación

- **Tipo:** Private Integration Token (PIT) — sub-cuenta única
- **Variable:** `GHL_API_TOKEN` en `.env`
- **Obtener token:** GoHighLevel → Settings → Private Integrations → Create New Integration
  - Scopes recomendados: Contacts, Conversations, Opportunities, Calendars, Locations

---

## Casos de uso

### Buscar contactos

Busca por nombre, email, teléfono o tags.

```
Búsqueda: "busca el contacto con email javi@ejemplo.com"
Búsqueda: "encuentra todos los contactos con el tag 'cliente-potencial'"
```

Herramientas: `searchContacts` / `getContact`

### Crear o actualizar un contacto

Crea un contacto nuevo o actualiza datos existentes (nombre, email, teléfono, empresa, tags, notas).

```
"Crea un contacto: nombre Pedro García, email pedro@empresa.com, teléfono +34600000000"
"Añade el tag 'reunión-pendiente' al contacto con ID xxx"
```

Herramientas: `createContact` / `updateContact` / `addContactTag`

### Enviar mensaje (SMS / email)

Envía un mensaje a un contacto a través de una conversación existente o nueva.

```
"Envía un SMS a Pedro García: 'Hola Pedro, te llamo mañana a las 10h'"
"Envía un email al contacto xxx con asunto 'Propuesta comercial'"
```

Herramientas: `sendMessage` / `createConversation`

### Ver conversaciones e inbox

Lista o lee el hilo de conversaciones de un contacto.

```
"Muéstrame las últimas conversaciones sin responder"
"Lee el hilo de SMS con el contacto xxx"
```

Herramientas: `listConversations` / `getConversation` / `getMessages`

### Ver y mover oportunidades (pipeline)

Consulta el pipeline de ventas, filtra por etapa y mueve deals.

```
"Lista todas las oportunidades en la etapa 'Propuesta enviada'"
"Mueve la oportunidad xxx a la etapa 'Cerrado ganado'"
"Muéstrame las últimas 10 oportunidades abiertas"
```

Herramientas: `searchOpportunities` / `getOpportunity` / `updateOpportunity`

### Ver pipelines disponibles

Consulta qué pipelines y etapas existen en la cuenta.

```
"Lista todos los pipelines y sus etapas"
```

Herramientas: `getPipelines`

### Gestionar tareas de un contacto

Crea, lista o completa tareas asociadas a un contacto.

```
"Crea una tarea para el contacto xxx: 'Llamar el lunes'"
"Lista las tareas pendientes del contacto xxx"
```

Herramientas: `createTask` / `getContactTasks` / `updateTask`

### Ver citas del calendario

Consulta citas próximas o crea nuevas.

```
"Muéstrame las citas de esta semana"
"Crea una cita con Pedro García el 20 de marzo a las 11h"
```

Herramientas: `getAppointments` / `createAppointment`

---

## Flujo típico de seguimiento de lead

```
1. searchContacts → localizar el contacto
2. getConversation → ver historial de comunicación
3. searchOpportunities → ver estado en el pipeline
4. sendMessage → contactar si procede
5. updateOpportunity → actualizar etapa del deal
6. createTask → crear recordatorio de seguimiento
```

---

## Notas

- El token PIT ya está vinculado a una sub-cuenta (Location); no es necesario especificar `GHL_LOCATION_ID` en la mayoría de llamadas.
- Las herramientas usan la API v2 de GoHighLevel (`services.leadconnectorhq.com`).
- Si una herramienta falla con 401, verificar que el token PIT sigue activo y tiene los scopes necesarios.

---

## Problemas conocidos y soluciones (verificados en producción)

### `locationId` es OBLIGATORIO aunque el token sea PIT

Aunque la documentación indica que el PIT ya está vinculado a una sub-cuenta, **todas las herramientas MCP que crean o listan recursos requieren pasar `locationId` explícitamente** en el cuerpo de la petición.

Sin él, la API devuelve:
```
403 — "The token does not have access to this location."
```

**Solución:** Añadir `GHL_LOCATION_ID` a `.env` y pasarlo en cada llamada MCP:
```
GHL_LOCATION_ID=xxx  # Settings → Business Info → Location ID, o la URL al entrar en la sub-cuenta
```

### `search-locations` devuelve 401 con token PIT

El endpoint `/locations/search` requiere un token de agencia, no un PIT de sub-cuenta. Devuelve:
```
401 — "No Authorization header found for authentication!"
```
No usar este endpoint con PIT para descubrir el locationId. En su lugar, obtenerlo manualmente desde la interfaz web de GHL: **Settings → Business Info** o la URL de la sub-cuenta.

### `search-locations` y `get-pipelines` sin locationId

Ambos fallan con PIT si no se pasa `locationId`. El locationId debe estar disponible en `.env` antes de usar cualquier herramienta MCP de GHL.

### Cómo obtener el locationId de una sub-cuenta

1. Entrar en GoHighLevel → seleccionar la sub-cuenta
2. La URL contiene el locationId: `https://app.gohighlevel.com/location/**{locationId}**/...`
3. O en **Settings → Business Info → Location ID**
4. Añadir a `.env`: `GHL_LOCATION_ID=<el valor>`
