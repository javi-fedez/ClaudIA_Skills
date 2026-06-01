---
name: seo-post-writing
description: Redacta un post de blog optimizado para SEO a partir de un brief, con metadatos. Úsalo para escribir el contenido del artículo.
---

# Skill: SEO Post Writing

Redacta un post de blog completamente optimizado para SEO a partir de un brief, usando la voz y estilo de la marca.

## Trigger phrases
- "escribe el post sobre [tema]"
- "redacta un artículo SEO sobre [tema]"
- "crea el contenido para [keyword]"
- "/seo-post-writing [tema o brief]"

---

## Prerequisito: leer el perfil

Leer `docs/perfil-empresa.md` y extraer antes de escribir:
- Tono y voz (sección 9) — frases cortas, tuteo, experiencia real
- ICP (sección 7) — para quién escribes
- Pain points (sección 8) — qué problemas resuelve la marca
- Sitio destino — sitio-1.example.com o sitio-2.example.com
- CTA habituales (sección 9)

---

## Input esperado

El post puede arrancarse de dos formas:
- **Con brief completo** — output de la skill `seo-keyword-research`
- **Con tema libre** — el usuario da un tema; en ese caso ejecutar primero el Paso 0

---

## Workflow

### Paso 0 — (Solo si no hay brief) Definir parámetros mínimos
Preguntar al usuario:
1. ¿Para qué sitio? (sitio-1.example.com o sitio-2.example.com)
2. ¿Keyword principal?
3. ¿Intención del lector? (aprender, comparar, contratar)
4. ¿Longitud aproximada? (800 / 1.200 / 2.000+ palabras)

### Paso 1 — Estructurar el post (outline)
Antes de redactar, generar y mostrar el outline para validación:

```
TÍTULO SEO (H1): [incluye keyword principal, max 60 chars]
META DESCRIPTION: [150-160 chars, incluye keyword, CTA]
SLUG: /[slug]/

H2: [Sección 1]
  H3: [Subsección opcional]
H2: [Sección 2]
  H3: [Subsección opcional]
H2: [Sección 3]
H2: [Conclusión / CTA final]

Palabras objetivo: X
Keyword density objetivo: 1-2%
```

Esperar confirmación del usuario antes de continuar, salvo que haya pedido ejecución automática.

### Paso 2 — Redactar el post completo

#### Estructura obligatoria del post

**BLOQUE META (para WordPress)**
```
Título SEO: [H1 — max 60 chars, keyword al inicio]
Meta description: [150-160 chars — keyword + beneficio + CTA]
Slug: /[keyword-principal-en-minusculas]/
Categoría: [categoría WordPress correspondiente]
Tags: [tag1, tag2, tag3, tag4, tag5]
Imagen destacada alt text: [descripción con keyword]
```

**INTRODUCCIÓN (150-200 palabras)**
- Gancho en la primera línea — pregunta, dato impactante o situación reconocible
- Identificar el problema del lector (pain point del ICP de la marca)
- Prometer qué va a aprender/conseguir al leer el artículo
- Incluir keyword principal de forma natural en los primeros 100 palabras

**CUERPO (según longitud objetivo)**
Reglas de redacción:
- Párrafos de 2-4 líneas máximo (legibilidad móvil)
- Un H2 cada 300-400 palabras aproximadamente
- Incluir keyword principal 1 vez por cada 500 palabras (aprox.)
- Keywords secundarias distribuidas naturalmente en H2/H3 y cuerpo
- Usar listas con viñetas o numeradas donde aporte claridad
- Incluir al menos 1 ejemplo real o caso práctico (puede ser ficticio pero verosímil, basado en la experiencia de la marca formando empresas)
- Datos y cifras siempre que refuercen el argumento (usar los de `perfil-empresa.md` sección 12 si aplican)
- Negritas para destacar conceptos clave (no abusar — máximo 1-2 por párrafo)

**CONCLUSIÓN + CTA (100-150 palabras)**
- Resumir el aprendizaje principal en 2-3 frases
- CTA claro y directo usando uno de los habituales de la marca:
  - Para sitio-1.example.com: suscripción al blog / newsletter
  - Para sitio-2.example.com: "Solicita tu diagnóstico inicial gratuito" / "Escríbeme a hola@sitio-2.example.com"

#### Reglas de estilo obligatorias
- **Tuteo siempre** — nunca "usted"
- **Voz activa** — "Copilot te ahorra tiempo", no "el tiempo es ahorrado por Copilot"
- **Sin jerga sin explicar** — si usas un término técnico, explícalo en la misma frase
- **Sin promesas vacías** — fundamentar cada afirmación con lógica o dato
- **Tono**: como un colega experto que te explica algo en una cafetería, no como un consultor en PowerPoint

### Paso 3 — Checklist SEO pre-entrega

Verificar antes de entregar el post:

- [ ] Keyword principal en H1 (título)
- [ ] Keyword principal en meta description
- [ ] Keyword principal en los primeros 100 palabras
- [ ] Keyword principal en al menos un H2
- [ ] Keywords secundarias distribuidas en el cuerpo
- [ ] Meta description entre 150-160 caracteres
- [ ] Título entre 50-60 caracteres
- [ ] Slug solo con keyword principal, en minúsculas, con guiones
- [ ] Al menos un enlace interno sugerido (a otro post o página del mismo sitio)
- [ ] Al menos un enlace externo sugerido (fuente de autoridad)
- [ ] Alt text de imagen destacada incluye keyword
- [ ] CTA al final del post
- [ ] Longitud dentro del objetivo (±10%)

### Paso 4 — Entregar el post formateado

Entregar en dos bloques:

**BLOQUE 1 — Metadatos WordPress**
```
TÍTULO:
META DESCRIPTION:
SLUG:
CATEGORÍA:
TAGS:
ALT TEXT IMAGEN:
ENLACE INTERNO SUGERIDO:
ENLACE EXTERNO SUGERIDO:
```

**BLOQUE 2 — Contenido del post (en Markdown)**
El post completo listo para copiar/pegar o pasar a la skill `seo-wordpress-publish`.

---

## Notas
- Si el usuario pide un post largo (+2.000 palabras), ofrecer dividirlo en una serie de 2-3 posts interconectados
- No inventar estadísticas — usar datos reales de `perfil-empresa.md` o buscarlos con WebSearch
- El ángulo diferencial de la marca: siempre anclar el contenido en la experiencia real formando empresas
- Para sitio-2.example.com: el tono puede ser ligeramente más corporativo pero sin perder cercanía
