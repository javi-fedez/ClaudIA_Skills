# Skill: SEO Keyword Research

Investiga y prioriza palabras clave para un tema dado, adaptadas al perfil de la marca y sus dos sitios web (sitio-1.example.com y sitio-2.example.com).

## Trigger phrases
- "investiga keywords para [tema]"
- "keyword research sobre [tema]"
- "qué palabras clave usar para [tema]"
- "/seo-keyword-research [tema]"

---

## Prerequisito: leer el perfil

Antes de cualquier búsqueda, leer `docs/perfil-empresa.md` y extraer:
- Temas pillar relevantes para el tema dado
- ICP (público objetivo) para orientar la intención de búsqueda
- Sitio destino: ¿sitio-1.example.com (PYMEs genérico) o sitio-2.example.com (Copilot/adopción IA)?

---

## Workflow

### Paso 1 — Entender el tema
Si el usuario no especifica el sitio destino, preguntar:
- ¿Es para **sitio-1.example.com** (blog general de IA para PYMEs) o **sitio-2.example.com** (Copilot 365 y adopción IA)?

### Paso 2 — Generar keywords semilla
A partir del tema, generar una lista inicial de 20-30 keywords semilla agrupadas por:

**A) Intención informacional** (el usuario quiere aprender)
- Preguntas: "cómo...", "qué es...", "para qué sirve..."
- Guías: "guía completa de...", "tutorial de..."

**B) Intención comercial/investigación** (el usuario compara opciones)
- Comparativas: "[herramienta] vs [herramienta]"
- Mejores: "mejores herramientas de...", "alternativas a..."

**C) Intención transaccional** (el usuario quiere contratar/comprar)
- Servicios: "consultoría de...", "formación en...", "empresa de..."
- Localización: "... para empresas", "... en España", "... para PYMEs"

**D) Long tail** (3-5 palabras, menor competencia)
- Combinaciones específicas del ICP de Javi

### Paso 3 — Investigar métricas (búsqueda web)
Para las 10-15 keywords más prometedoras, usar WebSearch para:
- Verificar volumen de búsqueda aproximado (resultados en Google, herramientas públicas)
- Identificar dificultad estimada (analizar quién rankea: grandes corporaciones vs blogs)
- Detectar la intención real (¿qué tipo de contenido aparece en top 10?)
- Encontrar keywords de cola larga adicionales en los resultados

Buscar también:
- `site:sitio-1.example.com [tema]` — evitar duplicar contenido existente
- `site:sitio-2.example.com [tema]` — idem

### Paso 4 — Analizar competencia SERP
Para las top 5 keywords seleccionadas, analizar los primeros 3 resultados de Google:
- ¿Qué tipo de contenido es? (guía, listicle, landing, vídeo)
- ¿Qué longitud aproximada?
- ¿Qué preguntas responden?
- ¿Qué keywords secundarias usan?

### Paso 5 — Seleccionar y priorizar
Seleccionar **1 keyword principal** + **5-8 keywords secundarias** con este criterio:
- Keyword principal: volumen medio-alto, dificultad media-baja, intención alineada con el sitio
- Secundarias: long tail, preguntas relacionadas, sinónimos semánticos

### Paso 6 — Generar el Brief SEO
Entregar un brief estructurado listo para la skill `seo-post-writing`:

```
## Brief SEO — [Tema]

**Sitio destino**: sitio-1.example.com / sitio-2.example.com
**URL slug sugerida**: /[slug-en-minusculas-con-guiones]/
**Keyword principal**: [keyword]
**Intención de búsqueda**: Informacional / Comercial / Transaccional

### Keywords secundarias
1. [keyword secundaria 1]
2. [keyword secundaria 2]
3. [keyword secundaria 3]
4. [keyword secundaria 4]
5. [keyword secundaria 5]

### Preguntas a responder en el post
- ¿[Pregunta 1]?
- ¿[Pregunta 2]?
- ¿[Pregunta 3]?

### Referencia de competencia
- Resultado 1: [URL] — [tipo de contenido, aprox. N palabras]
- Resultado 2: [URL] — [tipo de contenido, aprox. N palabras]

### Longitud recomendada: [X] palabras
### Formato recomendado: [Guía / Listicle / Tutorial / Comparativa]

### Notas adicionales para el redactor
[Observaciones sobre tono, datos a incluir, ángulo diferencial de Javi]
```

---

## Notas
- Priorizar keywords con intención comercial para sitio-2.example.com (más cerca de la conversión)
- Priorizar keywords informacionales para sitio-1.example.com (captación de audiencia y newsletter)
- Nunca proponer keywords ya cubiertas en posts existentes (verificar en el perfil sección 10 si está actualizada)
- El ángulo diferencial de Javi siempre es: experiencia real formando empresas, resultados medibles, seguridad Microsoft vs IA gratuita
