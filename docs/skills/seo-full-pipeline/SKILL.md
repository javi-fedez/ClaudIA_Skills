---
name: seo-full-pipeline
description: Ejecuta el pipeline SEO completo: keyword research, redacción del post y subida como borrador a WordPress. Úsalo para crear y publicar un post SEO de principio a fin.
---

# Skill: SEO Full Pipeline

Ejecuta el pipeline SEO completo de principio a fin: investiga keywords, redacta el post optimizado y lo sube como borrador a WordPress. Todo en un solo comando.

## Trigger phrases
- "pipeline SEO completo para [tema]"
- "crea y publica un post SEO sobre [tema]"
- "haz el post completo de [tema] para [sitio]"
- "/seo-pipeline [tema]"
- "/seo [tema]"

---

## Prerequisito: leer el perfil

Leer `docs/perfil-empresa.md` completamente antes de arrancar. Extraer:
- Sitio destino (si no lo especifica el usuario, preguntar)
- Tono, ICP, pain points, keywords semilla del tema
- Historial de contenido (sección 10) para evitar duplicados

---

## Workflow — 3 fases encadenadas

```
[TEMA] → FASE 1: Keyword Research → FASE 2: Post Writing → FASE 3: WP Publish → [DRAFT LIVE]
```

---

### FASE 1 — Keyword Research
*(Ejecutar skill `seo-keyword-research` completa)*

1. Generar keywords semilla agrupadas por intención
2. Investigar métricas via WebSearch
3. Analizar competencia SERP para top 5 keywords
4. Seleccionar keyword principal + 5-8 secundarias
5. **Output**: Brief SEO completo (título, slug, intención, keywords, preguntas a responder, longitud y formato recomendados)

**Checkpoint**: Mostrar el brief al usuario y esperar aprobación antes de continuar.
*(Si el usuario ha pedido ejecución autónoma total, continuar directamente)*

---

### FASE 2 — Post Writing
*(Ejecutar skill `seo-post-writing` con el brief aprobado)*

1. Generar outline estructurado
2. Redactar el post completo con todos los metadatos
3. Pasar checklist SEO de 13 puntos
4. **Output**:
   - Bloque de metadatos WordPress (título, meta, slug, categoría, tags, alt text)
   - Post completo en Markdown/HTML

**Checkpoint**: Mostrar el post al usuario para revisión.
*(Si el usuario ha pedido ejecución autónoma total, continuar directamente)*

---

### FASE 3 — WordPress Publish
*(Ejecutar skill `seo-wordpress-publish` con el output de Fase 2)*

1. Verificar conexión a la API WordPress del sitio destino
2. Resolver IDs de categoría y tags
3. Construir payload JSON con el contenido
4. Crear el draft via API REST
5. Verificar que el borrador se ha creado correctamente
6. **Output**: Resumen de confirmación con enlace de edición en WP Admin

---

## Resumen final del pipeline

Al completar las 3 fases, entregar un informe ejecutivo:

```
## Pipeline SEO completado

### Sitio: [sitio-1.example.com / sitio-2.example.com]

**Keyword principal**: [keyword]
**Intención**: [Informacional / Comercial / Transaccional]
**Longitud**: [N] palabras
**Dificultad estimada**: [Baja / Media / Alta]

**Post en WordPress**:
- ID: [id]
- Estado: BORRADOR
- Editar: [url wp-admin]
- URL pública (cuando publiques): [WP_URL]/[slug]/

**Checklist pre-publicación**:
- [ ] Revisar formato visual en Gutenberg
- [ ] Añadir imagen destacada (sugerencia: [descripción visual relevante])
- [ ] Configurar Yoast/Rank Math si no se aplicó via API
- [ ] Revisar enlaces internos (enlace sugerido: [url post relacionado])
- [ ] Programar o publicar manualmente

**Tiempo estimado de posicionamiento**: 3-6 meses para keywords de dificultad media
```

---

## Modos de ejecución

### Modo interactivo (por defecto)
El pipeline se detiene en cada checkpoint para validación del usuario. Recomendado para los primeros posts o cuando el tema es estratégico.

### Modo autónomo
Si el usuario dice "modo autónomo" o "sin pausas", ejecutar las 3 fases de corrido sin checkpoints. Usar solo si ya hay confianza en los outputs del pipeline.

---

## Combinaciones frecuentes

| Comando | Resultado |
|---------|-----------|
| `/seo-pipeline adopción Copilot 365` | Post completo para sitio-2.example.com (tema natural) |
| `/seo-pipeline automatización Power Automate sitio-1` | Post para sitio-1.example.com sobre Power Automate |
| `/seo-pipeline seguridad IA empresas` | Investiga y decide el sitio más adecuado |
| `/seo-pipeline [tema] modo autónomo` | Ejecuta sin pausas |

---

## Notas
- El pipeline completo puede tardar varios minutos por las búsquedas web — es normal
- Si el usuario interrumpe en cualquier fase, los outputs parciales son válidos y usables
- Las skills individuales (`seo-keyword-research`, `seo-post-writing`, `seo-wordpress-publish`) pueden ejecutarse por separado si solo se necesita una fase
- Para publicaciones en serie (ej: 4 posts al mes), ejecutar el pipeline una vez por post y registrar los resultados en la sección 10 de `docs/perfil-empresa.md`
