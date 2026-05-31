# Archive Skills — Directorio de referencia

Este directorio contiene **37 skills reutilizables** archivados en un solo lugar para reducir la carga de contexto cuando trabajas en la carpeta raíz de ClaudIA.

---

## 📋 Índice de skills

Ver `SKILLS-INDEX.md` para descripción completa de todos los skills:

```bash
cat SKILLS-INDEX.md
```

Resumen rápido por dominio:
- **`dev-*`** (7) — Ingeniería: agentes, code, git, scripts
- **`research-*`** (4) — Investigación: web, data, NotebookLM, YouTube
- **`comms-*`** (6) — Comunicación: email, content, presentations, Marca A
- **`ops-*`** (4) — Operaciones: CEO menu, calendar, publish, Hostinger
- **`crm-*`** (1) — CRM: GoHighLevel
- **`holded`** (1) — ERP: Facturas y contabilidad
- **`content-pipeline`** (1) — End-to-end: research → publish → email
- **`infra-*`** (2) — Infraestructura: Telegram bot, hooks & logging
- **`supabase`** (1) — Backend: Postgres, migraciones, edge functions
- **`hyperframes-brands`** (1) — Vídeo: inyección de marca
- **`seo-*`** (4) — SEO: keywords, posts, WordPress, pipeline
- **`tool-generar-pdf`** (1) — Utilidad: HTML → PDF
- **Global skills** (14) — HyperFrames (12) + SEO (1) + video-use (1)

---

## ⚡ Cómo usar estos skills

### Opción 1: Copiar a `~/.claude/skills/` (instalación global)

Para usar los skills en cualquier proyecto Claude Code:

```bash
# Una sola vez — copia todos los skills
cp archive-skills/*.md ~/.claude/skills/

# Luego en cualquier proyecto, invoca:
/dev-code-execution
/comms-email-triage
/seo-full-pipeline
# etc.
```

### Opción 2: Usar desde ClaudIA_Agent (local)

Si estás dentro de `/workspace/ClaudIA_Agent/`:

```bash
# Los skills se encuentran en archive-skills/
# Invoca dentro de Claude Code:
/dev-code-execution

# O refiérete explícitamente:
cat archive-skills/dev-code-execution.md
```

### Opción 3: Incluir en tus CLAUDE.md (custom)

Si tienes otro proyecto con su propio `CLAUDE.md`:

```markdown
# CLAUDE.md — Mi Proyecto

Para skills reutilizables, referencia:
- `/workspace/ClaudIA_Agent/archive-skills/` — 33 skills de ClaudIA

Copía la que necesites:
```bash
cp /workspace/ClaudIA_Agent/archive-skills/dev-code-execution.md ~/mi-proyecto/.claude/skills/
```
```

---

## 📦 Estructura de cada skill

Cada archivo `.md` tiene:

```markdown
# Skill: Nombre corto

Descripción breve.

---

## Herramientas disponibles

| Herramienta | Función |

---

## Cuándo usar

...

---

## Ejemplo

```
```

Todos siguen el mismo patrón para consistencia.

---

## 🔧 Mantenimiento

### Actualizar un skill

Si encuentras un bug o mejora:

```bash
# Edita en ClaudIA_Agent
vi archive-skills/dev-code-execution.md

# Si copiastes a ~/.claude/skills/, también copia la versión actualizada
cp archive-skills/dev-code-execution.md ~/.claude/skills/
```

### Crear un skill nuevo

```bash
# En archive-skills/, crea:
cat > archive-skills/my-new-skill.md << 'EOF'
# Skill: Mi nueva habilidad

Descripción breve.

---

## Herramientas disponibles

| Herramienta | Función |
|-------------|---------|
| Tool A | Descripción |

---

## Cuándo usar

...
EOF
```

Luego actualiza `SKILLS-INDEX.md` con una entrada nueva.

---

## 📊 Reducción de contexto lograda

**Antes:** 7,000+ líneas de skills/.md cargadas automáticamente como contexto
**Ahora:** 37 archivos archivados, solo se cargan cuando se necesitan

**Beneficio:** Reduce errores de "token budget exhausted" cuando trabajas en ClaudIA_Agent

---

## 🔗 Referencias

- **CLAUDE.md** — Punto de entrada (lee esto primero)
- **SYSTEM.md** — Arquitectura, stack, convenciones
- **OPERATIONS.md** — MCPs, infra, workflows, runbooks
- **SKILLS-INDEX.md** — Catálogo completo (en este directorio)

---

**Última actualización:** 2026-05-12
