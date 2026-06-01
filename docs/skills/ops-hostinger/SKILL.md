---
name: ops-hostinger
description: Administra el VPS de Hostinger vía MCP hostinger (snapshots, servicios, despliegues). Úsalo para operaciones de servidor. Crea siempre un snapshot antes de cambios críticos.
---

# ops-hostinger — Administración VPS Hostinger

Gestiona el VPS `vps.example.com` mediante el MCP `hostinger`.
Usa las herramientas `mcp__hostinger__*` disponibles en esta sesión.

---

## ⚠️ REGLA OBLIGATORIA: Snapshot antes de cambios críticos

**ANTES de ejecutar cualquiera de estas acciones debes crear un snapshot:**
- Desplegar o actualizar software en el VPS
- Modificar `docker-compose.yml` o cualquier compose file
- Tocar la configuración de Traefik
- Cambiar registros DNS críticos
- Recrear o reinstalar la VM

**Cómo crear el snapshot:**

1. Llama a `VPS_getVirtualMachinesV1` para obtener el `virtualMachineId`
2. Llama a `VPS_createSnapshotV1` con:
   - `virtualMachineId`: el ID obtenido
   - descripción: `"Pre-[motivo] YYYY-MM-DD - $ARGUMENTS"` (usa la fecha actual)
3. Confirma al usuario el ID y nombre del snapshot creado
4. Solo entonces procede con el cambio solicitado

**VPS de referencia:** `vps.example.com`

---

## Casos de uso

### Crear snapshot manualmente
Sigue los 3 pasos de la sección anterior.
Argumento opcional: descripción del motivo (ej: `pre-deploy n8n update`).

### Listar snapshots existentes
Llama a `VPS_getSnapshotV1` para ver los snapshots disponibles y sus fechas.

### Listar VMs y estado
Llama a `VPS_getVirtualMachinesV1` y muestra: ID, hostname, estado, IP, plan.

### Ver métricas del VPS
Llama a `VPS_getMetricsV1` con el `virtualMachineId`. Muestra CPU, RAM y disco.

### Gestionar DNS
- Listar registros: `DNS_getDNSRecordsV1` con el dominio
- Crear/actualizar registros: `DNS_updateDNSRecordsV1`
- Siempre confirmar con el usuario antes de modificar DNS

### Gestionar firewall
- Ver reglas: `VPS_getFirewallListV1` / `VPS_getFirewallDetailsV1`
- Crear regla: `VPS_createFirewallRuleV1`
- Activar/desactivar: `VPS_activateFirewallV1` / `VPS_deactivateFirewallV1`

### Reiniciar VPS
⚠️ Crear snapshot primero.
Llama a `VPS_restartVirtualMachineV1` con el `virtualMachineId`.

### Restaurar snapshot
⚠️ Acción destructiva — confirmar con el usuario antes de ejecutar.
Llama a `VPS_restoreSnapshotV1` con el `virtualMachineId` y `snapshotId`.

---

## Servicios activos en el VPS

| Servicio | URL / Puerto | Compose |
|---|---|---|
| Traefik + n8n | `n8n.vps.example.com` | `/root/docker-compose.yml` |
| Nginx files | interno | `/root/docker-compose.files.yml` |
| Paperclip | `paperclip.vps.example.com` | `/opt/paperclip/docker-compose.prod.yml` |
| AFFiNE | puerto 56962 | dir `affine-d4wq` |
| OpenClaw | puerto 46961 | dir `openclaw-2fes` |

Traefik gestiona SSL automático con Let's Encrypt (`mytlschallenge`).
Nuevos servicios deben conectarse a la red `root_default` para obtener SSL.

---

## Flujo estándar pre-deploy

```
1. ops-hostinger → crear snapshot (motivo: "pre-deploy <servicio>")
2. Conectar al VPS via SSH o ejecutar cambios
3. Verificar que el servicio arranca correctamente
4. Si algo falla → ops-hostinger → restaurar snapshot
```
