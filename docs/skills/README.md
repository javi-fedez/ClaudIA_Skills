# ClaudIA — Catálogo de Skills

Skills instalables para ClaudIA (formato Claude Code / Agent Skills). Cada skill vive
en su propia carpeta con un fichero `SKILL.md` que incluye frontmatter YAML
(`name` + `description`) y la documentación de uso.

**Instalación:** copia la carpeta del skill a tu directorio de skills
(`~/.claude/skills/<nombre>/` o `.claude/skills/<nombre>/` del proyecto). Claude lo
cargará automáticamente y lo invocará cuando la tarea coincida con su `description`.

> Las credenciales y valores reales se cargan desde `.env` (ver `.env.example`).
> Ningún skill contiene secretos.

---

## 💬 Comunicación (`comms-`)

| Skill | Carpeta | Resumen |
|-------|---------|---------|
| Content Creation | `comms-content-creation/` | Redacta y estructura documentos profesionales (propuestas, emails, posts, materiales). |
| Email Triage | `comms-email-triage/` | Lee correos no leídos, filtra ruido y genera un resumen accionable de la bandeja. |
| HTML Presentation | `comms-html-presentation/` | Genera presentaciones interactivas `.html` con CSS y JavaScript integrados. |
| Outlook Email | `comms-outlook-email/` | Consulta, busca, lee, responde y envía correos vía Outlook 365 (Graph). No envía sin confirmar. |

## 🛠️ Desarrollo y sistema (`dev-`)

| Skill | Carpeta | Resumen |
|-------|---------|---------|
| Agent Orchestration | `dev-agent-orchestration/` | Lanza sub-agentes en paralelo, delega tareas complejas y combina resultados. |
| Code Execution | `dev-code-execution/` | Ejecuta shell, scripts Python y automatiza tareas del sistema operativo. |
| File Operations | `dev-file-ops/` | Lee, escribe, edita y busca archivos en el sistema local. |
| Git Operations | `dev-git-operations/` | Versiona documentos, gestiona historial y trazabilidad con git. |

## ⚙️ Operaciones y negocio (`ops-`)

| Skill | Carpeta | Resumen |
|-------|---------|---------|
| Calendar | `ops-calendar/` | Consulta, crea, modifica y elimina eventos del calendario. Confirma antes de borrar. |
| CEO Menu | `ops-ceo-menu/` | Panel de 20 tareas autónomas de gestión para el CEO (comercial, marketing, finanzas…). |
| Publish Content | `ops-publish-content/` | Publica archivos generados en el servidor nginx y devuelve una URL pública. |

## 🔎 Investigación (`research-`)

| Skill | Carpeta | Resumen |
|-------|---------|---------|
| Data Analysis | `research-data-analysis/` | Lee, procesa e interpreta datos de negocio (CSV/JSON) con Python. |
| NotebookLM | `research-notebooklm/` | Conecta con Google NotebookLM de forma programática para crear y consultar notebooks. |
| Web Research | `research-web-research/` | Busca en internet y extrae contenido de páginas web sin APIs de terceros. |

## 📈 SEO y contenido web (`seo-`)

| Skill | Carpeta | Resumen |
|-------|---------|---------|
| Full Pipeline | `seo-full-pipeline/` | Pipeline SEO completo: keyword research + redacción + borrador en WordPress. |
| Keyword Research | `seo-keyword-research/` | Investiga y prioriza palabras clave para un tema y sitio destino. |
| Post Writing | `seo-post-writing/` | Redacta un post optimizado para SEO a partir de un brief, con metadatos. |
| WordPress Publish | `seo-wordpress-publish/` | Publica un post como borrador en WordPress vía REST API. Nunca publica directo. |

---

**Total: 18 skills** organizados en 5 familias.

## Convenciones

- **Acciones irreversibles o salientes** (enviar email, borrar evento, publicar) requieren
  confirmación explícita del usuario o crean siempre un borrador.
- Cada skill es **autónomo**: su `SKILL.md` documenta cuándo usarlo, el flujo paso a paso
  y las notas de seguridad.
- Los namespaces (`comms-`, `dev-`, `ops-`, `research-`, `seo-`) agrupan skills por dominio
  para facilitar el routing en un sistema multiagente.
