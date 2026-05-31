---
name: seo-wordpress-publish
description: Publica un post como borrador en WordPress vía REST API con todos los metadatos SEO. Úsalo para subir el draft; nunca publica directamente.
---

# Skill: SEO WordPress Publish

Publica un post como borrador en WordPress via REST API, con todos los metadatos SEO correctamente configurados. Nunca publica directamente — siempre crea un draft para revisión manual.

## Trigger phrases
- "publica el post en WordPress"
- "sube el borrador a WordPress"
- "crea el draft en WP de [sitio]"
- "/seo-wordpress-publish"

---

## Prerequisito: leer el perfil y credenciales

1. Leer `docs/perfil-empresa.md` sección 14 para obtener las URLs de la API
2. Las credenciales van en `.env` — NUNCA en este archivo:
   ```
   WP_SITE1_URL=https://tu-sitio-1.example.com
   WP_SITE1_USER=tu_usuario
   WP_SITE1_PASSWORD=tu_application_password   # WP Application Password
   WP_SITE2_URL=https://tu-sitio-2.example.com
   WP_SITE2_USER=tu_usuario
   WP_SITE2_PASSWORD=tu_application_password
   ```
   Los Application Passwords se generan en: WP Admin → Usuarios → Tu perfil → Application Passwords

---

## Workflow

### Paso 1 — Confirmar sitio destino

Preguntar al usuario si no está claro:
- ¿Publicar en **sitio 1** (`$WP_SITE1_URL`) o **sitio 2** (`$WP_SITE2_URL`)?

Cargar la configuración correspondiente del `.env`.

### Paso 2 — Verificar credenciales y conexión

```bash
# Test de conexión a la API
curl -u "$WP_USER:$WP_PASSWORD" \
  "$WP_URL/wp-json/wp/v2/users/me"
```

Si falla: verificar credenciales en `.env` y que los Application Passwords estén habilitados en WordPress.

### Paso 3 — Resolver IDs de categoría y tags

Las categorías y tags de WordPress requieren IDs numéricos, no nombres.

```bash
# Listar categorías existentes
curl -u "$WP_USER:$WP_PASSWORD" \
  "$WP_URL/wp-json/wp/v2/categories?per_page=50"

# Listar tags existentes
curl -u "$WP_USER:$WP_PASSWORD" \
  "$WP_URL/wp-json/wp/v2/tags?per_page=100"
```

Si la categoría o tag no existe, crear primero:
```bash
# Crear categoría nueva
curl -X POST -u "$WP_USER:$WP_PASSWORD" \
  "$WP_URL/wp-json/wp/v2/categories" \
  -H "Content-Type: application/json" \
  -d '{"name": "Nombre Categoría", "slug": "nombre-categoria"}'

# Crear tag nuevo
curl -X POST -u "$WP_USER:$WP_PASSWORD" \
  "$WP_URL/wp-json/wp/v2/tags" \
  -H "Content-Type: application/json" \
  -d '{"name": "Nombre Tag", "slug": "nombre-tag"}'
```

### Paso 4 — Preparar el payload del post

Construir el JSON con todos los campos del post:

```json
{
  "title": "[TÍTULO SEO del post]",
  "content": "[CONTENIDO en HTML — convertir el Markdown del post]",
  "excerpt": "[META DESCRIPTION — usado como extracto]",
  "slug": "[slug-del-post]",
  "status": "draft",
  "categories": [ID_categoria],
  "tags": [ID_tag1, ID_tag2, ID_tag3],
  "meta": {
    "_yoast_wpseo_title": "[TÍTULO SEO — si usa Yoast]",
    "_yoast_wpseo_metadesc": "[META DESCRIPTION — si usa Yoast]",
    "_rank_math_title": "[TÍTULO SEO — si usa Rank Math]",
    "_rank_math_description": "[META DESCRIPTION — si usa Rank Math]",
    "_rank_math_focus_keyword": "[KEYWORD PRINCIPAL — si usa Rank Math]"
  }
}
```

**Conversión Markdown → HTML**: los bloques Markdown deben convertirse a HTML limpio compatible con el editor de bloques de WordPress (Gutenberg). Reglas:
- `# Título` → `<h1>Título</h1>` (solo para el contenido, no el título del post)
- `## Sección` → `<h2>Sección</h2>`
- `### Subsección` → `<h3>Subsección</h3>`
- `**negrita**` → `<strong>negrita</strong>`
- Párrafos → `<p>párrafo</p>`
- Listas → `<ul><li>item</li></ul>` o `<ol><li>item</li></ol>`

### Paso 5 — Crear el draft en WordPress

```bash
curl -X POST -u "$WP_USER:$WP_PASSWORD" \
  "$WP_URL/wp-json/wp/v2/posts" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "...",
    "content": "...",
    "excerpt": "...",
    "slug": "...",
    "status": "draft",
    "categories": [...],
    "tags": [...]
  }'
```

Guardar el `id` del post devuelto en la respuesta.

### Paso 6 — Verificar el borrador creado

```bash
# Recuperar el post recién creado para verificar
curl -u "$WP_USER:$WP_PASSWORD" \
  "$WP_URL/wp-json/wp/v2/posts/[POST_ID]?status=draft"
```

### Paso 7 — Reportar resultado

Entregar al usuario un resumen de confirmación:

```
## Draft creado en WordPress

Sitio:      [sitio 1 / sitio 2]
Post ID:    [ID]
Título:     [Título]
Slug:       /[slug]/
Estado:     BORRADOR (draft)
Categoría:  [Nombre categoría]
Tags:       [tag1, tag2, tag3]
Editar en:  [WP_URL]/wp-admin/post.php?post=[ID]&action=edit

Pendiente de revisión manual:
- [ ] Revisar formato visual en el editor
- [ ] Añadir imagen destacada
- [ ] Verificar/ajustar metadatos SEO en Yoast/Rank Math
- [ ] Revisar enlaces internos y externos
- [ ] Publicar cuando esté listo
```

---

## Manejo de errores

| Error | Causa probable | Solución |
|-------|---------------|----------|
| 401 Unauthorized | Credenciales incorrectas | Regenerar Application Password en WP Admin |
| 403 Forbidden | Usuario sin permisos de editor | Asignar rol Editor o Administrator |
| 404 Not Found | URL de API incorrecta | Verificar que la REST API está activa en WP |
| 400 Bad Request | JSON malformado o campo inválido | Revisar el payload, especialmente los IDs de categorías |

---

## Notas de seguridad
- **NUNCA** usar user:password en texto plano en código — solo via variables de entorno
- **NUNCA** usar `"status": "publish"` — siempre `"draft"`
- Los Application Passwords de WordPress son la forma segura (Basic Auth sobre HTTPS)
- Rotar las credenciales si se sospecha que han sido expuestas
