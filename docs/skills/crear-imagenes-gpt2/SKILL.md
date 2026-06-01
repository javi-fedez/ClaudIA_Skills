---
name: crear-imagenes-gpt2
description: Genera imágenes con la API de OpenAI (gpt-image) para presentaciones, banners, ilustraciones web, diagramas, miniaturas e iconografía. Úsalo para cualquier necesidad visual generada por IA. La clave se lee de OPENAI_API_KEY en .env.
---

# Skill: Crear imágenes con GPT Image (OpenAI)

Genera imágenes con la API de OpenAI (`gpt-image-2`) para cualquier necesidad visual:
arte conceptual, recursos para presentaciones y documentos, banners e ilustraciones
para webs, diagramas explicativos de una idea, miniaturas, iconografía, etc.

Abierto a cualquier caso de uso visual. La API key ya está cargada en
`/workspace/ClaudIA_Agent/.env` como `OPENAI_API_KEY`.

---

## Cuándo usar esta skill

- "Genera una imagen artística de…"
- "Hazme un visual para esta diapositiva / documento"
- "Necesito un banner / hero / icono / miniatura para…"
- "Dibuja un diagrama que explique…"
- "Inserta una imagen ilustrativa en este post / informe"
- Cualquier punto de un flujo más amplio (HTML, slides, vídeo, post de blog, email, doc) donde haga falta un recurso visual generado.

---

## Modelo y parámetros

| Parámetro | Valores | Notas |
|-----------|---------|-------|
| `model` | `gpt-image-2` | Único modelo a usar |
| `size` | `1024x1024` · `1536x1024` (landscape) · `1024x1536` (portrait) · `auto` | Elegir según el destino |
| `quality` | `low` · `medium` · `high` · `auto` | `high` por defecto para entregables; `low/medium` para iteraciones rápidas |
| `background` | `transparent` · `opaque` · `auto` | `transparent` solo con formatos `png`/`webp` (iconos, recortes) |
| `output_format` | `png` · `jpeg` · `webp` | `png` para arte/diagramas; `jpeg` para fotorrealismo o reducir peso |
| `n` | 1–10 | Número de variantes |

La respuesta llega siempre como **base64** en `data[i].b64_json` — hay que decodificarla a binario antes de guardarla.

---

## Output: dónde van las imágenes

Dos modos según contexto:

### Modo A — entregable final (`docs/output/`)
Cuando la imagen es el producto pedido o quedará referenciada en un documento que se publica.

```
/workspace/ClaudIA_Agent/docs/output/yyyyMMdd_{titulo-en-kebab-case}.png
# Ej: 20260503_diagrama-flujo-rag.png
```

Si además se publica con `/ops-publish-content`, copiar a `docs/public/`.

### Modo B — recurso intermedio (temporal)
Cuando la imagen se va a embeber en otro entregable (HTML, slide, vídeo, post). Guardar
en `docs/output/_tmp/` para no contaminar la lista de outputs finales:

```
/workspace/ClaudIA_Agent/docs/output/_tmp/<slug>.png
```

Y referenciarla desde el archivo principal con ruta relativa o data URI según haga falta.

---

## Uso vía `curl` (rápido, una imagen)

```bash
set -a; source /workspace/ClaudIA_Agent/.env; set +a

OUT="/workspace/ClaudIA_Agent/docs/output/$(date -u +%Y%m%d)_diagrama-rag.png"

curl -s https://api.openai.com/v1/images/generations \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "Diagrama explicativo limpio de una arquitectura RAG: usuario → embeddings → vector DB → LLM → respuesta. Estilo flat, líneas finas, fondo blanco, etiquetas en español.",
    "size": "1536x1024",
    "quality": "high",
    "n": 1
  }' \
| python3 -c "import sys,json,base64,os; d=json.load(sys.stdin); open(os.environ['OUT'],'wb').write(base64.b64decode(d['data'][0]['b64_json'])); print(os.environ['OUT'])" \
  OUT="$OUT"
```

---

## Uso vía Python (recomendado para varias imágenes / control fino)

```python
import os, base64, datetime, pathlib
from openai import OpenAI

# Cargar OPENAI_API_KEY desde .env si no está en el entorno
from dotenv import load_dotenv
load_dotenv("/workspace/ClaudIA_Agent/.env")

client = OpenAI()

prompt = (
    "Ilustración editorial minimalista que representa la idea de 'agente IA "
    "autónomo orquestando herramientas': figura central abstracta conectada por "
    "líneas finas a iconos de email, calendario, base de datos y editor de código. "
    "Paleta dark navy + lime green. Estilo flat, alto contraste, fondo sólido."
)

result = client.images.generate(
    model="gpt-image-2",
    prompt=prompt,
    size="1536x1024",
    quality="high",
    n=1,
    # background="transparent",   # solo png/webp
    # output_format="png",
)

today = datetime.datetime.utcnow().strftime("%Y%m%d")
out = pathlib.Path(f"/workspace/ClaudIA_Agent/docs/output/{today}_agente-orquestador.png")
out.parent.mkdir(parents=True, exist_ok=True)
out.write_bytes(base64.b64decode(result.data[0].b64_json))
print(out)
```

Para uso intermedio cambiar el destino a `docs/output/_tmp/` (Modo B).

---

## Edición y variaciones (opcional)

`gpt-image-2` también soporta:

- **Edición/inpainting**: `client.images.edit(image=open(path,"rb"), mask=open(mask,"rb"), prompt="…")` — requiere imagen base y opcionalmente máscara PNG con alfa.
- **Composición**: pasar varias imágenes como referencia en `image=[f1, f2, …]` y describir cómo combinarlas.

Útil para: cambiar fondo, añadir elementos a un diagrama existente, crear variaciones de un mockup, mantener consistencia entre láminas de una presentación.

---

## Buenas prácticas de prompt

- **Describe estilo + composición + paleta + propósito.** Cuanto más concreto, mejor.
- **Idioma del texto dentro de la imagen**: indícalo explícitamente ("etiquetas en español", "labels in English"). `gpt-image-2` renderiza texto razonablemente bien si se pide.
- **Negativos**: usa frases como "sin texto extra", "sin marca de agua", "fondo sólido sin gradientes" en lugar de listas de prohibiciones largas.
- **Para diagramas**: pide explícitamente "diagrama flat, líneas finas, fondo blanco, sin sombras realistas" — evita que derive a fotorrealismo.
- **Para marca**: si es Marca B / Marca A / Marca C, leer primero `brands/<slug>/design.md` y trasladar paleta + tipografía + tono al prompt.

---

## Integración con otros flujos

| Flujo principal | Cómo encaja esta skill |
|------|------|
| `comms-html-presentation` / slides | Genera visuales de portada o de slide específico → guardar en `_tmp/` y embeber con `<img src="…">` o data URI |
| `comms-content-creation` (post, propuesta) | Imagen de cabecera + ilustraciones intermedias en `docs/output/` con el mismo `yyyyMMdd_titulo` que el documento |
| `seo-post-writing` / `seo-wordpress-publish` | Featured image y dentro del post; subir junto al post a WordPress |
| `dev-remotion-video` / `hyperframes` | Backgrounds, overlays, frames estáticos para componer con vídeo |
| `ops-publish-content` | Tras generar la imagen final, publicarla y devolver URL |
| `comms-outlook-email` | Generar imagen → publicar → enviar URL en el cuerpo (MCP no soporta adjuntos) |

---

## Errores y notas

- `400 invalid_request_error` con mensaje sobre `background` → estás pidiendo `transparent` con `output_format=jpeg`. Cambia a `png` o `webp`.
- `429` → respeta el rate limit; reintenta con backoff (5–15 s) o reduce `quality` a `medium`.
- El campo en la respuesta es `b64_json`, **no** `url`. No buscar URLs en la respuesta.
- Coste: `quality: high` es notablemente más caro que `medium`. Iterar prompt en `low`/`medium` y subir a `high` para la versión final.
- Política: no generar imágenes de personas reales identificables, contenido sexual, gore, marcas registradas concretas, ni cualquier cosa que infrinja la policy de OpenAI — la API la rechazará igualmente.
