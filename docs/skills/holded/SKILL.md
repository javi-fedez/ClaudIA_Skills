---
name: holded
description: Gestiona contactos, documentos, facturación y CRM en Holded (ERP) vía MCP y API REST directa, con soporte multi-tenant. Úsalo para operar Holded. Las API keys van en .env.
---

# holded — ERP & Facturación Holded (MCP + API directa)

Gestiona contactos, documentos y CRM en Holded mediante las herramientas MCP `mcp__holded-autonomo__*` / `mcp__holded-empresa__*` y la API REST directa.

---

## Tenants disponibles

| Tenant | Prefijo MCP | Cuenta | Variable `.env` |
|--------|-------------|--------|-----------------|
| Autónomo | `mcp__holded-autonomo__*` | hola@marca-c.example.com | `TENANT_1_API_KEY` |
| Empresa | `mcp__holded-empresa__*` | admin@empresa-b.example.com | `TENANT_2_API_KEY` |

---

## Autenticación API directa

Todas las llamadas directas usan el header `key` (sin `Bearer`):

```bash
curl -s "https://api.holded.com/api/..." \
  -H "key: $TENANT_2_API_KEY" \
  -H "Content-Type: application/json"
```

- **No** usar `Authorization: Bearer ...` — Holded usa su propio header `key`.
- La misma clave funciona para todos los módulos (invoicing, crm, contacts…).

---

## MCP vs API directa — cuándo usar cada una

| Operación | MCP | API directa |
|-----------|-----|-------------|
| Crear/buscar contactos | ✅ Funciona bien | También válida |
| Crear documentos (`estimate`, `invoice`…) | ⚠️ Bug en campo `date` | ✅ Recomendada |
| Oportunidades CRM (`/crm/v1/leads`) | ❌ No disponible en MCP | ✅ Única opción |
| Listar/consultar documentos | ✅ Funciona bien | También válida |
| Enviar/pagar documentos | ✅ Funciona bien | También válida |

---

## Módulo Contactos

### Crear contacto via MCP (funciona correctamente)

```
mcp__holded-empresa__create_contact
  name:      "Nombre Apellidos" o "Nombre Empresa"
  type:      "client" | "supplier" | "lead" | "debtor" | "creditor"
  email:     "email@ejemplo.com"
  phone:     "+34600000000"
  note:      "LinkedIn, origen, contexto..."
  vatnumber: "B12345678"   # CIF/NIF — solo si es empresa
```

Respuesta: `{"status": 1, "info": "Created", "id": "<contactId>"}` — guardar el `id`.

**Reglas clave:**
- Persona + empresa = **dos contactos separados** (Holded no tiene relación nativa persona↔empresa en contactos, solo en CRM via `person`).
- `type: "lead"` para contactos comerciales nuevos; `type: "client"` una vez que haya contrato.
- El campo `note` acepta texto libre (LinkedIn, fuente, etc.).

### Crear contacto via API directa

```bash
curl -s -X POST "https://api.holded.com/api/contacts/v1/contacts" \
  -H "key: $TENANT_2_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "Nombre", "type": "lead", "email": "...", "notes": "..."}'
```

---

## Módulo Facturación — Documentos

### Tipos de documento (`docType`)

| docType | Descripción |
|---------|-------------|
| `estimate` | Presupuesto / Oferta comercial |
| `invoice` | Factura de venta |
| `salesreceipt` | Ticket de venta |
| `creditnote` | Factura rectificativa |
| `receiptnote` | Albarán de venta |
| `salesorder` | Pedido de venta |
| `waybill` | Nota de entrega |
| `proform` | Proforma |
| `purchase` | Factura de compra |
| `purchaserefund` | Abono de compra |
| `purchaseorder` | Pedido de compra |

### Crear documento via API directa (recomendado)

```bash
curl -s -X POST "https://api.holded.com/api/invoicing/v1/documents/{docType}" \
  -H "key: $TENANT_2_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contactId": "<id del contacto>",
    "date":      1775952000,
    "currency":  "EUR",
    "items": [
      {"name": "Descripción del servicio", "units": 1, "subtotal": 12000}
    ],
    "notes": "Notas internas"
  }'
```

Respuesta exitosa:
```json
{"status": 1, "id": "...", "invoiceNum": "PAIA260029", "contactId": "..."}
```

- `date`: Unix timestamp en segundos (ej. `1775952000` = 2026-04-12). **Obligatorio.**
- `items[].subtotal`: importe sin IVA. El campo `tax` es opcional.
- `invoiceNum`: asignado automáticamente. Formato empresa: `PAIA{AA}{NNNN}` (Presupuesto → PA, Empresa → I, Año, Nº secuencial).

### Bug MCP: `create_document` rechaza el campo `date`

**Síntoma:**
```
Validation error: date: Invalid input: expected string, received number
```

**Causa:** El schema MCP declara `date` como `type: number` pero el validador interno lo rechaza como número. Sin `date`, la API devuelve `400 — "Wrong date"` (campo obligatorio).

**Solución:** Usar siempre la API directa con curl para crear documentos.

### Actualizar etapa de pipeline de un documento (MCP)

```
mcp__holded-empresa__update_document_pipeline
  docType:    "estimate" (u otro tipo)
  documentId: "<id del documento>"
  pipelineId: "<id del pipeline>"
  stageId:    "<id de la etapa>"
```

---

## Módulo CRM — Oportunidades en el Funnel

### Endpoint correcto: `/api/crm/v1/leads`

> **Importante:** El módulo CRM de Holded **no está disponible en el MCP**. Solo funciona via API directa.

Rutas que **no funcionan** (devuelven HTML 404):
- `/api/crm/v1/funnel`
- `/api/crm/v1/funnel/{id}`
- `/api/crm/v1/deals`

Ruta que **sí funciona:**
- `/api/crm/v1/leads` — lista y crea oportunidades

### Obtener leads existentes (y descubrir funnelId + stageIds)

```bash
curl -s "https://api.holded.com/api/crm/v1/leads" -H "key: $TENANT_2_API_KEY"
```

Cada lead contiene `funnelId` y `stageId`. Son la única forma de conocer los IDs de funnel y etapas — no hay endpoint para listarlos directamente.

**IDs del funnel activo (Empresa B — verificado 2026-04-12):**
- `funnelId`: `FUNNEL_ID`
- Etapa inicial: `STAGE_ID_INICIAL`
- Etapa avanzada: `STAGE_ID_AVANZADA`

### Crear oportunidad en el funnel

```bash
curl -s -X POST "https://api.holded.com/api/crm/v1/leads" \
  -H "key: $TENANT_2_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "funnelId":  "FUNNEL_ID",
    "stageId":   "STAGE_ID_INICIAL",
    "contactId": "<id empresa>",
    "person":    "<id persona de contacto>",
    "name":      "Nombre descriptivo de la oportunidad",
    "value":     12000
  }'
```

Campos:
- `funnelId` + `stageId`: obtener de `/crm/v1/leads` si no se conocen.
- `contactId`: id de la empresa (tipo `client` o `lead` en contactos).
- `person`: id de la persona de contacto — **campo separado del `contactId`**, permite vincular empresa + persona en la misma oportunidad.
- `name`: título visible en el kanban del CRM.
- `value`: importe estimado en EUR (número, sin símbolo).

Respuesta: `{"status": 1, "info": "Created", "id": "<leadId>"}`

---

## Flujo completo: nuevo lead comercial

```
1. MCP create_contact (empresa, type: "client")   → guardar id empresa
2. MCP create_contact (persona, type: "lead")     → guardar id persona
3. API POST /crm/v1/leads                         → crear oportunidad en funnel
     contactId: id empresa · person: id persona · value: importe
4. API POST /invoicing/v1/documents/estimate      → crear presupuesto vinculado
     contactId: id persona · items: [...] · date: unix timestamp
5. (Opcional) MCP send_document                   → enviar presupuesto por email
6. (Opcional) MCP pay_document                    → registrar cobro al cerrar
```

---

## Referencias

- API base invoicing: `https://api.holded.com/api/invoicing/v1/`
- API base CRM: `https://api.holded.com/api/crm/v1/`
- API base contactos: `https://api.holded.com/api/contacts/v1/`
- Auth: header `key: {API_KEY}` (sin "Bearer")
- Docs oficiales: https://developers.holded.com/reference/
