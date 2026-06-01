---
name: skills-index
description: Catálogo índice de todos los skills locales reutilizables, agrupados por dominio. Documento de referencia para descubrir qué skills existen y cómo invocarlos.
---

# Skills Index — ClaudIA Agent

Catálogo de **37 skills locales reutilizables** archivados en `archive-skills/`. Se invocan con `/nombre-skill` en Claude Code.

**Organización:** Agrupadas por dominio (dev, research, ops, comms, etc.)

---

## 🏗️ Ingeniería (`dev-*`)

### dev-agent-orchestration
Lanza sub-agentes especializados en paralelo para tareas complejas. Combina resultados sin APIs externas.
- Uso: Investigaciones paralelas, protección de contexto
- Sub-agentes: `general-purpose`, `Explore`, `Plan`

### dev-code-execution
Ejecuta scripts Python, comandos shell y código arbitrario de forma segura.
- Uso: Automatización, procesamiento de datos, testing
- Entorno: `.venv/` aislado, sin permisos elevados

### dev-file-ops
Lee, escribe, busca y edita archivos con seguridad.
- Uso: Manipulación de archivos, búsquedas recursivas
- Herramientas: `Read`, `Write`, `Edit`, `grep`, `find`

### dev-git-operations
Versionado, historial, recuperación de cambios, branches.
- Uso: Control de versión, manejo de conflictos
- Operaciones: commit, push, rebase, stash

### dev-remotion-video
Diseña, escribe y renderiza vídeos programáticos con Remotion.
- Uso: Vídeos procedurales, animaciones sincronizadas
- Stack: Remotion API + MCP Remotion Documentation
- Alternativa moderna: `/hyperframes` (recomendado)

### dev-scripts-catalog
Catálogo de scripts Python en `scripts/` (content_pipeline, hooks, NotebookLM, browser-use).
- Uso: Automatización end-to-end
- Ubicación: `scripts/`, `scripts/adopty/`

### dev-subagents-catalog
Referencia de sub-agentes nativos (Explore, general-purpose, Plan, code-simplifier).
- Uso: Delegar tareas especializadas
- Invocación: `Agent(subagent_type="explore")`

### tool-generar-pdf
Convierte HTML → PDF con Playwright, preservando dark mode y sangrado a borde.
- Uso: Reportes, documentos, presentaciones
- Features: Saltos por `<section>`, sin dependencias externas

---

## 🔬 Investigación (`research-*`)

### research-web-research
Búsqueda web y extracción de contenido desde URLs.
- Uso: Investigación de tendencias, análisis competitivo
- Herramientas: `WebSearch`, `WebFetch`

### research-data-analysis
Análisis de CSV/JSON, generación de informes ejecutivos y gráficos.
- Uso: Análisis de datos, visualizaciones
- Formato: Tablas, gráficos ASCII, markdown

### research-notebooklm
Gestión de notebooks en Google NotebookLM: crear, gestionar audio, slides, quiz, infografías.
- Uso: Resúmenes, audios, presentaciones
- Librería: `notebooklm-py`

### youtube-research
Investigación temática en YouTube con keywords adaptativas → notebook NotebookLM automático.
- Uso: Análisis de tendencias en video
- Output: Notebook + audio + slides

---

## 💬 Comunicación (`comms-*`)

### comms-email-triage
Triaje automático de bandeja de entrada con borradores de respuesta inteligentes.
- Uso: Procesamiento de emails en masa
- MCP: Outlook

### comms-outlook-email
Consultar, buscar, leer, contestar y enviar correos vía MCP Outlook.
- Uso: Automatización de email
- Herramientas: `outlook_list_unread`, `outlook_send_email`, etc.

### comms-content-creation
Redacción de propuestas, emails, posts y materiales de curso.
- Uso: Generación de contenido reutilizable
- Formatos: Markdown, HTML, YAML frontmatter

### comms-html-presentation
Presentaciones HTML/CSS/JS navegables (estilo PowerPoint) sin dependencias externas.
- Uso: Decks, pitches, propuestas
- Features: Transitions, speaker notes, notas de voz

### comms-marca-a-slides
Presentaciones one-page con marca Marca A (hand-drawn whiteboard, navegación tipo deck).
- Uso: Contenido Marca A
- Brand: Montserrat, paleta blanco/amarillo/rojo

### comms-marca-a-document
Documentos web largos (briefings, dossiers, whitepapers) con marca Marca A.
- Uso: Contenido extenso Marca A
- Features: TOC automático, estilos marca

### comms-marca-a-video
Edición de reels/tutoriales del programa "21 días adoptando IA" con video-use + ffmpeg.
- Uso: Vídeos educativos Marca A
- Stack: ffmpeg, libass, Montserrat Black
- Paleta: Blanco/amarillo/rojo

---

## 🏢 Operaciones de negocio (`ops-*`)

### ops-ceo-menu
20 tareas autónomas para CEO: ventas, marketing, OKRs, finanzas, estrategia.
- Uso: Automatización de ejecutivas
- Ejemplos: Reportes semanales, análisis de pipeline

### ops-calendar
Consultar, crear, modificar y eliminar eventos vía MCP Calendar.
- Uso: Gestión de calendario
- MCP: Microsoft Calendar (M365)

### ops-publish-content
Publicar archivos en el servidor nginx del VPS y obtener URL pública.
- Uso: Publicación rápida de assets
- Destino: `docs/public/` → URL pública vía Hostinger

### ops-hostinger
Administración del VPS Hostinger: snapshots, firewall, DNS, métricas, dominios.
- Uso: Operaciones de infra
- MCP: Hostinger VPS + DNS Manager

---

## 💼 CRM

### crm-highlevel
Gestionar contactos, oportunidades, conversaciones y automatizaciones en GoHighLevel.
- Uso: Operaciones CRM
- MCP: Lead Connector (GoHighLevel)

---

## 🧾 ERP

### holded
Facturas, presupuestos, albaranes, contactos, productos y tesorería en dos tenants Holded.
- Uso: Operaciones contables
- Tenants: autonomo + empresa
- MCP: Holded (x2)

---

## 📦 Pipeline de contenido

### content-pipeline
Pipeline end-to-end: research → NotebookLM/HTML → publicar → email → git.
- Uso: Automatización de producción de contenido
- Stack: research-*, comms-*, ops-publish-content

---

## 🔧 Infraestructura del VPS (`infra-*`)

### infra-telegram-bot
Bot de Telegram + whisper.cpp, gestión PM2, variables y troubleshooting.
- Uso: Operaciones del bot
- Ubicación: `/opt/telegram-claude-bot/`

### infra-hooks-and-logging
Hooks `UserPromptSubmit` y `Stop`, interaction logging, cron de retención.
- Uso: Monitoreo y auditoría
- Logs: `docs/logs/interactions.jsonl`

---

## 🗄️ Backend

### supabase
Gestionar todos los proyectos Supabase (marca-a + Marca B): esquema, migraciones, edge functions, branches y RLS.
- Uso: Operaciones de backend
- MCP: Supabase oficial (`@supabase/mcp-server-supabase`)
- Cobertura: Múltiples orgs con un único PAT

---

## 🎬 Vídeo por marca

### hyperframes-brands
Loader de brand specs (Marca B, Marca A, Marca C).
- Función: Copia `brands/<slug>/design.md` + `fonts/` + `assets/` a raíz del proyecto
- Uso pre-invocar: `/hyperframes` para inyectar marca
- Brands: marca-b, marca-a, marca-c

---

## 🔍 SEO

### seo-keyword-research
Investigación de palabras clave y análisis competitivo.
- Uso: Estrategia de contenido SEO
- Output: CSV con keywords, volumen, dificultad

### seo-post-writing
Redacción de posts SEO optimizados (E-E-A-T, schema markup, structure).
- Uso: Contenido web SEO
- Features: Headings, meta descriptions, internal links

### seo-wordpress-publish
Publicación directa en WordPress vía API REST con validación previa.
- Uso: Publicación automática
- Validación: Checks previos, rollback si falla

### seo-full-pipeline
Pipeline completo: keyword → redacción → publicación → monitoreo.
- Uso: Automatización end-to-end SEO
- Integra: seo-keyword-research + seo-post-writing + seo-wordpress-publish

### seo-agentic-audit
Documentación de la skill global `/seo` (audit, page, technical, schema, sitemap, geo, aeo, links, plan, github).
- Uso: Referencia de auditoria SEO
- Comando global: `/seo` (instalado en `~/.claude/skills/`)

---

## 🌐 Skills Globales (instaladas en `~/.claude/skills/`)

### HyperFrames (12 skills)

**hyperframes**
- Autoría de composiciones HTML con timing, GSAP, escenas, palettes
- Features: Animation timelines, scene management, responsive design
- Alternativa a Remotion (recomendado para HTML-first)

**hyperframes-cli**
- Comandos: `init`, `preview`, `render`, `lint`, `validate`, `inspect`, `tts`, `transcribe`
- Stack: Node 22+, Puppeteer, ffmpeg 6.1+

**hyperframes-registry**
- Catálogo de bloques reutilizables
- Instalación: `hyperframes add <block>`

**gsap** / **animejs** / **lottie** / **three** / **waapi** / **css-animations**
- Adaptadores de motores de animación para composiciones HyperFrames
- Cada uno cubre un stack: GSAP timeline, Anime.js, Lottie JSON, Three.js/WebGL, Web Animations API, CSS keyframes

**tailwind**
- Utilidades Tailwind CSS v4.2 en composiciones HyperFrames
- Runtime browser o compilado a CSS

**remotion-to-hyperframes** / **website-to-hyperframes**
- Migradores: Remotion → HyperFrames, website HTML → HyperFrames composition

### Agentic SEO (global — `~/.claude/skills/seo`)

**seo**
- Auditoría completa con comandos: `audit`, `page`, `technical`, `schema`, `sitemap`, `images`, `geo`, `aeo`, `links`, `plan`, `github`
- Generación automática: `FULL-AUDIT-REPORT.md`, `ACTION-PLAN.md`

### Video Editing (global)

**video-use**
- Edición conversacional: transcribe, cut, color grade, overlay animations, burn subtítulos
- Soporta: talking heads, montajes, tutoriales, viajes, entrevistas

---

## 📥 Cómo instalar/usar

### Skills locales (`archive-skills/`)

```bash
# Opción 1: Copiar a ~/.claude/skills/ para uso global
cp archive-skills/*.md ~/.claude/skills/

# Opción 2: Invocar en Claude Code (local)
/dev-code-execution       # Debe estar en archive-skills/
```

### Skills globales (HyperFrames, SEO, video-use)

Ya instaladas en `~/.claude/skills/`. Se invocan directamente:

```bash
/hyperframes              # HyperFrames composition authoring
/seo audit https://...   # SEO auditoría
/video-use               # Video editing
```

---

## 📊 Estadísticas

| Métrica | Valor |
|---------|-------|
| **Skills locales** | 37 (en archive-skills/) |
| **Skills globales** | 12 HyperFrames + 1 SEO + 1 video = **14** |
| **Dominios** | dev (7) + tool (1) + research (4) + comms (6) + ops (4) + crm (1) + holded (1) + pipeline (1) + infra (2) + supabase (1) + brands (1) + seo (5) + marca-a (3, dentro de comms) = **37** |
| **Líneas totales** | ~7,000 líneas de markdown |
| **MCPs integrados** | 9 (Outlook, Calendar, Holded x2, n8n, Remotion, Lead Connector, Hostinger, Supabase) |

---

## 🔗 Referencias rápidas

- **System:** `/workspace/ClaudIA_Agent/SYSTEM.md` (arquitectura, stack, convenciones)
- **Operations:** `/workspace/ClaudIA_Agent/OPERATIONS.md` (MCPs, infra, workflows, runbooks)
- **Main:** `/workspace/ClaudIA_Agent/CLAUDE.md` (punto de entrada)
- **Brands:** `/workspace/ClaudIA_Agent/brands/` (marca-b, marca-a, marca-c)
- **Scripts:** `/workspace/ClaudIA_Agent/scripts/` (automatización, hooks, logging)
- **Demos:** `/workspace/ClaudIA_Agent/docs/output/` (vídeos, reportes generados)

---

**Última actualización:** 2026-05-15 · **Total de skills documentados:** 37 locales + 14 globales = 51
