# ClaudIA — Catálogo de Skills

Skills instalables para ClaudIA (formato Claude Code / Agent Skills). Cada skill vive
en su propia carpeta con un fichero `SKILL.md` que incluye frontmatter YAML
(`name` + `description`) y la documentación de uso.

**Instalación:** copia la carpeta del skill a tu directorio de skills
(`~/.claude/skills/<nombre>/` global, o `.claude/skills/<nombre>/` del proyecto).
Claude lo carga automáticamente y lo invoca cuando la tarea coincide con su `description`.

> Las credenciales y valores reales se cargan desde `.env` (ver `.env.example` en la raíz).
> Ningún skill contiene secretos ni datos reales — los dominios, marcas y tokens están anonimizados.

---

## 💬 Comunicación (`comms-`)

| Skill | Carpeta | Resumen |
|-------|---------|---------|
| `comms-content-creation` | `comms-content-creation/` | Redacta y estructura documentos profesionales (propuestas, emails, posts, materiales de curso)  |
| `comms-email-triage` | `comms-email-triage/` | Lee correos no leídos, filtra publicidad y newsletters, y genera borradores de respuesta solo  |
| `comms-html-presentation` | `comms-html-presentation/` | Genera presentaciones interactivas en formato .html con CSS y JavaScript integrados. |
| `comms-marca-a-document` | `comms-marca-a-document/` | Crea documentos web largos tipo briefing/whitepaper/dossier scrolleable con identidad de marca  |
| `comms-marca-a-slides` | `comms-marca-a-slides/` | Crea presentaciones web one-page con navegación tipo diapositiva (← → · Espacio · F pant |
| `comms-marca-a-video` | `comms-marca-a-video/` | Edita videos para marca-a.example.com (reels, tutoriales, minilecciones del programa "21 días  |
| `comms-outlook-email` | `comms-outlook-email/` | Consulta, busca, lee, responde y envía correos vía el servidor MCP de Outlook 365 (Microsoft  |

## 🛠️ Desarrollo y sistema (`dev-`)

| Skill | Carpeta | Resumen |
|-------|---------|---------|
| `dev-agent-orchestration` | `dev-agent-orchestration/` | Lanza sub-agentes especializados en paralelo, delega tareas complejas y combina sus resultados. |
| `dev-code-execution` | `dev-code-execution/` | Ejecuta comandos de shell, scripts Python y automatiza tareas del sistema operativo. |
| `dev-file-ops` | `dev-file-ops/` | Lee, escribe, edita y busca archivos en el sistema local. |
| `dev-git-operations` | `dev-git-operations/` | Gestiona el historial de documentos, cambios y trazabilidad con git. |
| `dev-remotion-video` | `dev-remotion-video/` | Diseña, escribe y renderiza vídeos programáticos con Remotion (React/TypeScript), con búsqu |
| `dev-scripts-catalog` | `dev-scripts-catalog/` | Inventario de referencia de los scripts Python del proyecto y cómo invocarlos. |
| `dev-subagents-catalog` | `dev-subagents-catalog/` | Inventario de referencia de los sub-agentes disponibles y cuándo delegarles trabajo. |

## ⚙️ Operaciones e infraestructura (`ops-`, `infra-`)

| Skill | Carpeta | Resumen |
|-------|---------|---------|
| `infra-hooks-and-logging` | `infra-hooks-and-logging/` | Documentación del sistema de hooks de Claude Code que registra toda interacción y el cierre d |
| `infra-telegram-bot` | `infra-telegram-bot/` | Documentación de referencia del bot de Telegram que conecta con Claude Code en el VPS. |
| `ops-calendar` | `ops-calendar/` | Consulta, crea, modifica y elimina eventos del calendario (Outlook/M365). |
| `ops-ceo-menu` | `ops-ceo-menu/` | Panel de 20 tareas autónomas listas para el CEO de una empresa pequeña de formación (comerci |
| `ops-hostinger` | `ops-hostinger/` | Administra el VPS de Hostinger vía MCP hostinger (snapshots, servicios, despliegues). |
| `ops-publish-content` | `ops-publish-content/` | Publica archivos generados (HTML, PDF, imágenes, docs) en el servidor nginx y devuelve una URL |

## 🔎 Investigación (`research-`)

| Skill | Carpeta | Resumen |
|-------|---------|---------|
| `research-data-analysis` | `research-data-analysis/` | Lee, procesa e interpreta datos de negocio (CSV/JSON) usando Python nativo. |
| `research-notebooklm` | `research-notebooklm/` | Conecta con Google NotebookLM de forma programática para crear y consultar notebooks y generar |
| `research-web-research` | `research-web-research/` | Busca en internet y extrae información de páginas web sin APIs de terceros. |

## 📈 SEO y contenido web (`seo-`)

| Skill | Carpeta | Resumen |
|-------|---------|---------|
| `seo-agentic-audit` | `seo-agentic-audit/` | Realiza auditorías SEO deterministas y LLM-first sobre webs, posts y repos GitHub, con sub-ski |
| `seo-full-pipeline` | `seo-full-pipeline/` | Ejecuta el pipeline SEO completo: keyword research, redacción del post y subida como borrador  |
| `seo-keyword-research` | `seo-keyword-research/` | Investiga y prioriza palabras clave para un tema y sitio destino. |
| `seo-post-writing` | `seo-post-writing/` | Redacta un post de blog optimizado para SEO a partir de un brief, con metadatos. |
| `seo-wordpress-publish` | `seo-wordpress-publish/` | Publica un post como borrador en WordPress vía REST API con todos los metadatos SEO. |

## 🔌 Integraciones de negocio (CRM, ERP, backend)

| Skill | Carpeta | Resumen |
|-------|---------|---------|
| `crm-highlevel` | `crm-highlevel/` | Gestiona el CRM de GoHighLevel (Lead Connector) vía el MCP lead-connector: contactos, oportuni |
| `holded` | `holded/` | Gestiona contactos, documentos, facturación y CRM en Holded (ERP) vía MCP y API REST directa, |
| `supabase` | `supabase/` | Gestiona todos los proyectos Supabase del usuario (Postgres, Auth, Storage, Edge Functions, Rea |

## 🎬 Producción de contenido y utilidades

| Skill | Carpeta | Resumen |
|-------|---------|---------|
| `content-pipeline` | `content-pipeline/` | Pipeline autónomo de producción de contenido de extremo a extremo: investiga un tema, genera  |
| `crear-imagenes-gpt2` | `crear-imagenes-gpt2/` | Genera imágenes con la API de OpenAI (gpt-image) para presentaciones, banners, ilustraciones w |
| `hyperframes-brands` | `hyperframes-brands/` | Apply per-brand design specs (Marca B, Marca A, Marca C) to HyperFrames video compositions. |
| `tool-generar-pdf` | `tool-generar-pdf/` | Genera un PDF de alta fidelidad a partir de una URL pública o un archivo HTML local usando Pla |
| `youtube-research` | `youtube-research/` | Investiga una temática localizando vídeos de YouTube relevantes, analiza su contenido y los d |

## 📚 Índices y catálogos de referencia

| Skill | Carpeta | Resumen |
|-------|---------|---------|
| `skills-index` | `skills-index/` | Catálogo índice de todos los skills locales reutilizables, agrupados por dominio. |

---

**Total: 37 skills** organizados por dominio.

## Convenciones

- **Acciones irreversibles o salientes** (enviar email, borrar, publicar, migrar BD, cambios en VPS)
  requieren confirmación explícita del usuario o crean siempre un borrador / snapshot previo.
- Cada skill es **autónomo**: su `SKILL.md` documenta cuándo usarlo, el flujo y las notas de seguridad.
- Los skills marcados como *catálogo/índice/referencia* (`skills-index`, `dev-scripts-catalog`,
  `dev-subagents-catalog`, `infra-*`) documentan componentes pero **no son ejecutables**.
- Las credenciales de las integraciones (Holded, Supabase, GoHighLevel, OpenAI, Telegram, Hostinger)
  se leen siempre de `.env`, nunca hardcodeadas.
