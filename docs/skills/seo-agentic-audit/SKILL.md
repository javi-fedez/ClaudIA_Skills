---
name: seo-agentic-audit
description: Realiza auditorías SEO deterministas y LLM-first sobre webs, posts y repos GitHub, con sub-skills, agentes y scripts especializados. Skill global instalada en ~/.claude/skills/seo; este doc es referencia. Invócala con /seo.
---

# Skill: SEO Agentic Audit (skill global `/seo`)

Skill agéntica avanzada instalada globalmente en `~/.claude/skills/seo`. Realiza auditorías SEO deterministas y LLM-first sobre webs, posts y repositorios GitHub. Incluye **16 sub-skills**, **10 agentes especializados** y **33 scripts** de análisis.

> **Importante:** esta skill **NO** está en el directorio del proyecto (`/workspace/ClaudIA_Agent/skills/`); vive en `~/.claude/skills/seo/`. Este `.md` actúa solo como documentación de referencia para que aparezca en el dashboard. Para invocarla escribe `/seo` en Claude Code.

## Trigger phrases
- "haz una auditoría SEO de [url]"
- "analiza el SEO de [url]"
- "revisa schema/sitemap/CWV de [url]"
- "plan estratégico SEO para [url]"
- "/seo audit [url]"

---

## Comandos disponibles

| Comando | Descripción |
|---------|-------------|
| `/seo audit <url>` | Auditoría completa del sitio · scoring ponderado por 7 categorías |
| `/seo page <url>` | Análisis profundo de una única página |
| `/seo technical <url>` | Checks técnicos: robots.txt, redirects, seguridad, Core Web Vitals |
| `/seo content <url>` | Calidad de contenido, legibilidad (Flesch-Kincaid) y E-E-A-T |
| `/seo schema <url>` | Detección, validación y generación de schema JSON-LD |
| `/seo sitemap <url>` | Análisis y validación del sitemap XML |
| `/seo images <url>` | Optimización de imágenes y alt text |
| `/seo geo <url>` | AI Search Optimization — `llms.txt`, AI crawlers en `robots.txt` |
| `/seo links <url>` | Perfil de backlinks y salud de enlaces externos |
| `/seo aeo <url>` | Answer Engine Optimization (Featured Snippets, People Also Ask) |
| `/seo plan <url>` | Plan estratégico SEO con detección automática de industria |
| `/seo github <repo>` | Auditoría SEO de repositorios GitHub |

---

## Entregables automáticos

Cada auditoría completa genera dos ficheros en el directorio de trabajo:

- **`FULL-AUDIT-REPORT.md`** — informe completo con hallazgos verificados, evidencia y nivel de confianza
- **`ACTION-PLAN.md`** — plan priorizado (impacto × esfuerzo) con quick wins y mejoras a medio plazo

---

## Scripts incluidos

Los 33 scripts se ejecutan automáticamente según el comando elegido:

| Script | Propósito |
|--------|-----------|
| `robots_checker.py` | Validación de `robots.txt` |
| `llms_txt_checker.py` | Verificación de `llms.txt` (AI crawlers) |
| `pagespeed.py` | Core Web Vitals · LCP/CLS/INP |
| `security_headers.py` | HTTPS, HSTS, CSP, X-Frame-Options |
| `broken_links.py` | Enlaces rotos internos y externos |
| `redirect_checker.py` | Cadenas de redirects y bucles |
| `readability.py` | Flesch-Kincaid, longitud frase/párrafo |
| `social_meta.py` | Open Graph, Twitter Cards |
| `internal_links.py` | Anchor text y profundidad de clicks |
| `article_seo.py` | Title, meta, headings, keyword density |
| `generate_report.py` | Composición del informe final |
| `finding_verifier.py` | Verificación de hallazgos antes de reportar |
| + scripts GitHub SEO | README, topics, social preview, releases |

---

## Cuándo usar cada comando

| Caso de uso | Comando recomendado |
|---|---|
| Auditar un sitio existente desde cero | `/seo audit https://midominio.com` |
| Optimizar una página concreta | `/seo page https://midominio.com/url` |
| Solo migración técnica / pre-launch | `/seo technical https://midominio.com` |
| Validar JSON-LD de schema | `/seo schema https://midominio.com/url` |
| Preparar para AI search (ChatGPT, Perplexity) | `/seo geo https://midominio.com` |
| Diseñar estrategia de contenidos | `/seo plan https://midominio.com` |
| Optimizar repo open-source | `/seo github user/repo` |

---

## Diferencia con las skills `seo-*` del proyecto

Las otras skills SEO de este proyecto (`seo-keyword-research`, `seo-post-writing`, `seo-wordpress-publish`, `seo-full-pipeline`) están orientadas a **crear y publicar contenido nuevo**. La skill global `/seo` está orientada a **auditar y optimizar activos existentes**.

| Skill | Objetivo | Ámbito |
|---|---|---|
| `/seo` (global) | Auditar y optimizar | Webs, posts, repos ya publicados |
| `seo-keyword-research` | Investigar | Keywords antes de escribir |
| `seo-post-writing` | Crear | Redactar post optimizado |
| `seo-wordpress-publish` | Publicar | Subir a WordPress como draft |
| `seo-full-pipeline` | Crear + publicar | Pipeline end-to-end |

---

## Ubicación física

Skill global: `~/.claude/skills/seo/` (incluye sub-skills, agents y scripts)
Documentación de referencia: `skills/seo-agentic-audit.md` (este fichero)
