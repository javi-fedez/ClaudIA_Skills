# Skill — Supabase MCP

Habilidad para gestionar **todos los proyectos Supabase** del usuario (Postgres, Auth, Storage, Edge Functions, Realtime) desde Claude Code mediante el servidor MCP oficial `@supabase/mcp-server-supabase`.

**Invocación:** `/supabase` (o cualquier instrucción en lenguaje natural sobre Supabase).

---

## Configuración

- Servidor MCP: `supabase` — definido en `.mcp.json` → `mcp-servers/supabase/start.js`
- Token: `SUPABASE_ACCESS_TOKEN` en `.env` (Personal Access Token de https://supabase.com/dashboard/account/tokens)
- Modo: **read-write** (puede aplicar migraciones, ejecutar DDL/DML, deploy de edge functions)
- Scope: **cuenta completa** — un único PAT cubre todas las organizaciones del usuario
- Prefijo de herramientas: `mcp__supabase__*`

### Organizaciones y proyectos visibles

| Organización | Proyectos |
|--------------|-----------|
| **marca-a** | adoptify (`hcmakhonzyzyqekdwtes`) · adopty.app (`zackhelfbxqicccspbrm`) |
| **Marca B** | Villasan y Puertas (`wcjuniulgotzxdhnjuaw`) · marca-b_General (`rmmcuqyniwtlnkcyalls`) · Marca B OS (`xvyxjlyefbovgtcitbtq`) |
| **EMPRESA B SL** | Gestionada vía Vercel (solo lectura) |

> **Importante:** dado que un único PAT ve TODOS los proyectos, **siempre confirma el `project_ref` con el usuario** antes de ejecutar cambios destructivos (migraciones, DROP, edge functions). No asumas el proyecto activo.

---

## Flujo de trabajo recomendado

### 1. Identificar el proyecto correcto

Antes de cualquier operación, listar y elegir explícitamente:

```
mcp__supabase__list_projects
```

Si el usuario dice "en adoptify" → busca por nombre o pídele confirmar el `id` (project ref).

### 2. Explorar antes de modificar

```
mcp__supabase__list_tables    project_id=<ref>
mcp__supabase__list_extensions project_id=<ref>
mcp__supabase__list_migrations project_id=<ref>
mcp__supabase__get_advisors    project_id=<ref> type=security  (o performance)
```

### 3. Consultas de solo lectura → `execute_sql`

```
mcp__supabase__execute_sql project_id=<ref> query="SELECT ..."
```

### 4. Cambios de esquema → `apply_migration`

Para DDL (CREATE TABLE, ALTER TABLE, etc.) usa **siempre** `apply_migration`, no `execute_sql`. Las migraciones quedan registradas y son reversibles.

```
mcp__supabase__apply_migration project_id=<ref> name=add_users_table query="CREATE TABLE ..."
```

### 5. Edge Functions

```
mcp__supabase__list_edge_functions project_id=<ref>
mcp__supabase__deploy_edge_function project_id=<ref> name=<fn> files=[...]
```

### 6. Database branching (entornos efímeros para preview)

```
mcp__supabase__create_branch  project_id=<ref> name=feature-x
mcp__supabase__list_branches  project_id=<ref>
mcp__supabase__merge_branch   branch_id=<id>
mcp__supabase__delete_branch  branch_id=<id>
```

### 7. Generar tipos TypeScript del esquema

```
mcp__supabase__generate_typescript_types project_id=<ref>
```

### 8. Conectar la app cliente

```
mcp__supabase__get_project_url project_id=<ref>
mcp__supabase__get_anon_key    project_id=<ref>
```

---

## Reglas de oro

1. **Confirma el `project_ref`** antes de cualquier escritura. Un único PAT = blast radius alto.
2. **DDL → `apply_migration`**, nunca `execute_sql`. Mantiene historial reversible.
3. **Antes de migraciones, ejecuta `get_advisors` (security + performance)** para detectar problemas previos.
4. **Antes de borrar/alterar datos**, muestra al usuario el SQL a ejecutar y espera aprobación.
5. **Pega siempre `project_id`** en cada llamada — no hay "proyecto activo" implícito.
6. **No expongas el `service_role` key** ni el PAT en respuestas, logs o documentos publicados.
7. Para tareas largas (poblar datos, backfill), prefiere edge functions o scripts en lugar de muchas llamadas SQL secuenciales.

---

## Patrones comunes

### "Inspecciona el esquema de adopty.app"

```
1. list_projects                                    → encuentra ref de adopty.app
2. list_tables          project_id=zackhelfbxqicccspbrm
3. (opcional) execute_sql project_id=... query="\\d <tabla>"
```

### "Crea una tabla en marca-b_General"

```
1. Confirma con el usuario: marca-b_General → ref=rmmcuqyniwtlnkcyalls
2. list_tables (para ver qué hay)
3. Muestra el SQL al usuario y espera OK
4. apply_migration project_id=rmmcuqyniwtlnkcyalls name=create_xxx query="CREATE TABLE ..."
5. list_migrations para confirmar
```

### "Despliega una edge function en Marca B OS"

```
1. project_id=xvyxjlyefbovgtcitbtq
2. list_edge_functions (ver si ya existe)
3. deploy_edge_function project_id=... name=... files=[{name:"index.ts", content:"..."}]
4. Muestra al usuario el endpoint resultante
```

### "Audita la seguridad de un proyecto"

```
1. get_advisors project_id=<ref> type=security
2. get_advisors project_id=<ref> type=performance
3. Resume hallazgos por severidad y propón correcciones
```

---

## Troubleshooting

| Error | Causa probable | Solución |
|-------|----------------|----------|
| `SUPABASE_ACCESS_TOKEN no definido` | Falta en `.env` | Añadirlo y reiniciar Claude Code / bot |
| `401 Unauthorized` | PAT revocado o expirado | Generar uno nuevo en Account → Access Tokens |
| `403 Forbidden` en un proyecto | El usuario no es miembro de esa org | Ser invitado o usar otro PAT |
| `project_id` no encontrado | Ref incorrecto | `list_projects` para obtener el ref exacto |
| MCP no aparece | `.mcp.json` no recargado | Salir y volver a entrar en Claude Code; `pm2 restart claudia-bot --update-env` para el bot |

---

## Referencia rápida de herramientas

```
list_organizations           # orgs visibles al PAT
get_organization             # detalle de una org
list_projects                # todos los proyectos
get_project                  # detalle (status, region, etc.)
create_project               # nuevo proyecto
pause_project / restore_project
list_tables                  # tablas de un proyecto
list_extensions              # extensiones Postgres habilitadas
list_migrations              # historial de migraciones
apply_migration              # DDL como migración versionada
execute_sql                  # SELECT / DML puntual
get_logs                     # logs de Postgres / Edge / Auth
get_advisors                 # security · performance
list_edge_functions
get_edge_function
deploy_edge_function
create_branch / list_branches / merge_branch / delete_branch / rebase_branch / reset_branch
generate_typescript_types
get_project_url / get_anon_key
search_docs                  # buscar en docs.supabase.com
```

Documentación oficial del paquete: https://github.com/supabase-community/supabase-mcp
