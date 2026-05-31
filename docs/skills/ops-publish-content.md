# Skill: Publicar contenido en el servidor de archivos públicos

Publica archivos generados (HTML, PDF, imágenes, docs) en el servidor nginx del VPS
y devuelve una URL pública accesible desde cualquier navegador.

---

## Infraestructura del servidor de archivos

| Elemento | Valor |
|----------|-------|
| Dominio | `files.example.com` |
| Protocolo | HTTPS (certificado Let's Encrypt via Traefik) |
| Contenedor | `files-server` (nginx:alpine) |
| Directorio VPS | `/workspace/ClaudIA_Agent/docs/public/` |
| Path URL protegido | `/REDACTED_FILES_TOKEN/` |
| Acceso sin token | HTTP 403 — bloqueado |
| Config nginx | `/root/nginx-files.conf` |
| Compose file | `/root/docker-compose.files.yml` |

**Patrón de URL resultante:**
```
https://files.example.com/REDACTED_FILES_TOKEN/<nombre-del-archivo>
```

---

## Cuándo usar esta skill

- Compartir una presentación HTML generada
- Publicar un informe o documento para que el cliente lo abra en el navegador
- Enviar por email un enlace a un archivo en lugar del adjunto (el MCP Outlook no soporta adjuntos)
- Dar acceso temporal a cualquier archivo generado en `docs/output/`

---

## Proceso paso a paso

### Paso 1 — Verificar que el archivo existe

El archivo debe estar en `docs/output/` (ruta absoluta: `/workspace/ClaudIA_Agent/docs/output/`).

```bash
ls /workspace/ClaudIA_Agent/docs/output/<nombre-del-archivo>
```

Si no existe, generarlo primero con la skill correspondiente antes de continuar.

### Paso 2 — Copiar al directorio público

```bash
cp /workspace/ClaudIA_Agent/docs/output/<nombre-del-archivo> \
   /workspace/ClaudIA_Agent/docs/public/
```

El contenedor Docker monta `/workspace/ClaudIA_Agent/docs/public/` en
`/usr/share/nginx/html/REDACTED_FILES_TOKEN/`, por lo que el archivo
queda disponible de forma inmediata sin reiniciar ningún servicio.

### Paso 3 — Verificar accesibilidad

```bash
curl -sk -o /dev/null -w "%{http_code}" \
  "https://files.example.com/REDACTED_FILES_TOKEN/<nombre-del-archivo>"
```

Respuesta esperada: `200`. Si devuelve otro código:

| Código | Causa | Solución |
|--------|-------|----------|
| `403` | Path incorrecto o acceso a `/` | Verificar que la URL incluye el token completo |
| `404` | Archivo no copiado o nombre incorrecto | Verificar `ls /workspace/ClaudIA_Agent/docs/public/` |
| `000` | Docker caído | `docker ps` y comprobar `files-server` |

### Paso 4 — Devolver la URL al usuario

Formato de respuesta:

```
https://files.example.com/REDACTED_FILES_TOKEN/<nombre-del-archivo>
```

---

## Flujo completo de ejemplo

```bash
# 1. El archivo ya existe en docs/output/
ls /workspace/ClaudIA_Agent/docs/output/20260316_framework-consultoria-cliente-ejemplo.html

# 2. Copiar al directorio público
cp /workspace/ClaudIA_Agent/docs/output/20260316_framework-consultoria-cliente-ejemplo.html \
   /workspace/ClaudIA_Agent/docs/public/

# 3. Verificar HTTP 200
curl -sk -o /dev/null -w "%{http_code}" \
  "https://files.example.com/REDACTED_FILES_TOKEN/20260316_framework-consultoria-cliente-ejemplo.html"

# 4. URL resultante:
# https://files.example.com/REDACTED_FILES_TOKEN/20260316_framework-consultoria-cliente-ejemplo.html
```

---

## Listar archivos publicados actualmente

```bash
ls -lh /workspace/ClaudIA_Agent/docs/public/
```

O acceder al índice web (autoindex activado):

```
https://files.example.com/REDACTED_FILES_TOKEN/
```

---

## Eliminar un archivo publicado

```bash
rm /workspace/ClaudIA_Agent/docs/public/<nombre-del-archivo>
```

El cambio es inmediato. No requiere reiniciar el contenedor.

---

## Publicar + enviar por email en un solo flujo

Cuando el usuario pide "envíame el archivo por correo", el flujo correcto es:

1. Publicar el archivo según este skill
2. Enviar el email con la URL usando el MCP Outlook (`mcp__outlook__outlook_send_email`)

Ejemplo de cuerpo de email:
```
Aquí tienes el archivo listo para abrir en el navegador:

https://files.example.com/REDACTED_FILES_TOKEN/<nombre-del-archivo>
```

**Por qué este flujo:** el MCP de Outlook no soporta adjuntos. Publicar primero
y enviar el enlace es siempre más útil que intentar incluir el archivo en el cuerpo.

---

## Notas de seguridad

- El path `/REDACTED_FILES_TOKEN/` actúa como token de acceso — no publicar esta URL en repositorios públicos ni documentos compartidos abiertamente.
- Cualquier archivo copiado a `docs/public/` es accesible para quien tenga la URL.
- No existe autenticación adicional — el token en el path es la única protección.
- Si se necesita eliminar el acceso a un archivo, borrarlo con `rm`.
