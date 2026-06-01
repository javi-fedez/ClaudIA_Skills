---
name: youtube-research
description: Investiga una temática localizando vídeos de YouTube relevantes, analiza su contenido y los documenta en un notebook nuevo de NotebookLM, con estrategia de keywords autónoma. Úsalo para research basada en vídeo.
---

# Skill: YouTube Research → NotebookLM

Investiga una temática a fondo localizando vídeos de YouTube relevantes, analiza su contenido y los documenta en un cuaderno de NotebookLM nuevo. La estrategia de keywords es **completamente autónoma**: el agente decide cuántas palabras clave usar, si empezar amplio o específico, y si iterar y ampliar según los resultados obtenidos.

---

## Trigger phrases
- "investiga [tema] en YouTube"
- "busca vídeos de YouTube sobre [tema] y crea un notebook"
- "youtube research [tema]"
- "analiza el contenido de YouTube sobre [tema]"
- "/youtube-research [tema]"
- "/yt-research [tema]"

---

## Objetivo del skill

1. **Investigar** el tema con keywords adaptativas (pocas → más según necesidad)
2. **Descubrir** vídeos de YouTube relevantes y de calidad
3. **Analizar** el contenido de cada vídeo (título, descripción, canal, transcripción si disponible)
4. **Crear** un cuaderno NotebookLM con todos los vídeos como fuentes
5. **Documentar** las referencias en un `.txt` registrado también en el cuaderno

---

## FASE 1 — Estrategia de keywords (autónoma)

### Principio de adaptación progresiva

Antes de lanzar ninguna búsqueda, razona sobre el tema:

```
Criterios para decidir la estrategia inicial:
- Tema muy específico/técnico → comenzar con 1-2 keywords exactas
- Tema amplio/exploratorio  → comenzar con 2-3 términos paraguas
- Tema en español           → buscar en ES y también en EN (más volumen YT)
- Tema con jerga/siglas     → incluir variante expandida (ej: "IA" + "inteligencia artificial")
```

### Flujo adaptativo (obligatorio seguirlo)

> **Mínimo absoluto: 10 vídeos relevantes antes de pasar a FASE 2.**
> Si no se alcanza ese umbral, continuar iterando con nuevas keywords.

```
ITERACIÓN 1: Buscar con keyword(s) semilla (mín. 1, máx. 3)
  → Evaluar resultados: ¿cuántos vídeos relevantes y únicos?

  SI < 10 vídeos relevantes:
    → OBLIGATORIO ampliar: añadir sinónimos, términos relacionados, variante EN
    → Lanzar ITERACIÓN 2 con 2-4 keywords nuevas
    → Repetir hasta alcanzar el mínimo de 10

  SI 10-20 vídeos relevantes:
    → Suficiente. Pasar a FASE 2.
    → (Opcional) Una búsqueda adicional de refinamiento si el tema lo merece

  SI > 20 vídeos pero muchos irrelevantes:
    → Refinar: añadir modificadores ("tutorial", "explicación", "2024", "en español")
    → Filtrar por relevancia real hasta quedarse con los 10-20 mejores

MÁXIMO 5 iteraciones de búsqueda. Objetivo: 10-20 vídeos de alta calidad.
NUNCA pasar a FASE 2 con menos de 10 vídeos. Si tras 5 iteraciones no se llega
a 10, informar al usuario y continuar con los que haya (mínimo real: todos los
disponibles sobre el tema, aunque sean menos de 10).
```

### Búsquedas de YouTube recomendadas

Usar `WebSearch` con estas variantes para descubrir vídeos:

```
# Búsqueda directa en YouTube
site:youtube.com "[keyword principal]"
site:youtube.com "[keyword]" tutorial
site:youtube.com "[keyword]" explicación
site:youtube.com "[keyword]" curso completo

# Búsqueda de listas curadas
"mejores vídeos de youtube sobre [tema]"
"[keyword] youtube playlist"
"[keyword] youtube channel"

# Búsqueda en inglés si el tema es técnico/global
site:youtube.com "[english keyword]"
```

---

## FASE 2 — Análisis de vídeos

Para cada URL de YouTube descubierta:

### 2a. Fetch de la página del vídeo

```
WebFetch(url_youtube)
Extraer:
  - Título completo
  - Canal (nombre y URL)
  - Fecha de publicación (si visible)
  - Descripción completa
  - Duración (si aparece en metadata)
  - Número de vistas (si visible)
  - Tags/palabras clave de la descripción
```

### 2b. Evaluación de relevancia

Tras el fetch, puntuar cada vídeo (1-5):

| Criterio | Señales positivas |
|----------|------------------|
| Relevancia temática | Keywords del tema en título/descripción |
| Profundidad | Duración > 5 min, descripción detallada, canales especializados |
| Calidad de fuente | Canal con autoridad en el nicho, vistas > 1K |
| Actualidad | Publicado en los últimos 2-3 años (salvo que el tema sea atemporal) |

**Descartar** vídeos con puntuación ≤ 2. Mantener los de puntuación ≥ 3.

### 2c. Búsqueda de transcripción (si disponible)

```
# Intentar obtener transcripción automática o subtítulos
WebSearch("[título del vídeo] transcripción OR subtítulos OR transcript")
WebFetch(URL si encuentra transcript en terceros como youtubetranscript.com o similar)
```

Si no hay transcripción disponible, usar la descripción + título como resumen de contenido.

---

## FASE 3 — Creación del cuaderno NotebookLM

> **Este paso usa el skill `research-notebooklm` (`skills/research-notebooklm.md`).**
> Seguir sus patrones de auth, client y gestión de fuentes al pie de la letra.
> Ver ese skill para referencia de errores comunes, polling y descarga de artefactos.

### 3a. Preparar el documento de referencias

Antes de abrir el cliente NotebookLM, construir el archivo `referencias-youtube.txt`:

```
Ruta: /workspace/ClaudIA_Agent/docs/output/[FECHA]_[SLUG]_referencias-youtube.txt

Contenido:
  REFERENCIAS — INVESTIGACIÓN YOUTUBE: [TEMA]
  Generado: [ISO timestamp]
  Total vídeos: [N]
  ════════════════════════════════════════════════════════════
  [1] [Título del vídeo]
      Canal   : [Nombre del canal]
      URL     : https://youtube.com/watch?v=...
      Fecha   : [Fecha publicación]
      Resumen : [Resumen extraído del análisis en FASE 2]

  [2] ...
  ════════════════════════════════════════════════════════════
  Fuentes indexadas en este cuaderno NotebookLM como vídeos de YouTube.
  Investigación realizada por ClaudIA.
```

Guardar el archivo con `Write` antes de ejecutar el script.

### 3b. Script Python (ejecutar con `mcp__ide__executeCode`)

Usar el patrón de auth del skill `research-notebooklm` (funciones `load_env`,
`get_auth_json`, `get_client_tokens`). El flujo completo es:

```python
import asyncio, json, os, pathlib
from datetime import datetime
from notebooklm import NotebookLMClient, AuthTokens
from notebooklm.auth import extract_cookies_from_storage, fetch_tokens

# ── Auth (patrón canónico de skills/research-notebooklm.md) ──────────────────

def load_env(path='/workspace/ClaudIA_Agent/.env'):
    if not pathlib.Path(path).exists():
        return
    with open(path) as f:
        for line in f:
            line = line.strip()
            if not line or line.startswith('#'): continue
            if '=' not in line: continue
            eq = line.index('=')
            key = line[:eq].strip()
            val = line[eq+1:].strip().split(' #')[0].strip().strip('"\'')
            if key and not os.environ.get(key): os.environ[key] = val

def get_auth_json() -> dict:
    load_env()
    raw = os.environ.get('NOTEBOOKLM_AUTH_JSON', '')
    if not raw:
        raise ValueError("NOTEBOOKLM_AUTH_JSON no encontrada en .env")
    data = json.loads(raw)
    sp = pathlib.Path.home() / '.notebooklm' / 'storage_state.json'
    sp.parent.mkdir(parents=True, exist_ok=True)
    sp.write_text(json.dumps(data))
    return data

async def get_client_tokens():
    auth_json = get_auth_json()
    cookies = extract_cookies_from_storage(auth_json)
    csrf, session_id = await fetch_tokens(cookies)
    return AuthTokens(cookies=cookies, csrf_token=csrf, session_id=session_id)

# ── Datos de la investigación (rellenar con resultados de FASE 2) ─────────────

TEMA  = "NOMBRE DEL TEMA"
FECHA = datetime.now().strftime("%Y%m%d")
SLUG  = TEMA.lower().replace(" ", "-")[:40]

# Lista de vídeos seleccionados en FASE 2
# {"url": "https://youtube.com/watch?v=XXX", "title": "...", "channel": "...",
#  "date": "...", "summary": "..."}
VIDEOS = []

REFS_PATH = f"/workspace/ClaudIA_Agent/docs/output/{FECHA}_{SLUG}_referencias-youtube.txt"

# ── Main ──────────────────────────────────────────────────────────────────────

async def main():
    tokens = await get_client_tokens()
    async with NotebookLMClient(tokens) as client:   # ← SIN await antes del with

        # 1. Crear cuaderno
        notebook_title = f"YouTube Research: {TEMA} ({FECHA})"
        notebook = await client.notebooks.create(notebook_title)
        notebook_id = notebook.id
        print(f"Cuaderno creado: {notebook_title}  ID={notebook_id}")

        # 2. Añadir vídeos como fuentes YouTube
        added, failed = [], []
        for v in VIDEOS:
            try:
                await client.sources.add_url(notebook_id, v['url'])
                added.append(v)
                print(f"  OK: {v.get('title', v['url'])}")
                await asyncio.sleep(2)   # pausa anti-rate-limit
            except Exception as e:
                failed.append(v)
                print(f"  FAIL: {v['url']} — {e}")

        # 3. Leer el referencias.txt ya guardado en disco (paso 3a)
        refs_content = pathlib.Path(REFS_PATH).read_text(encoding='utf-8')

        # 4. Añadir referencias como fuente de texto en el cuaderno
        await client.sources.add_text(
            notebook_id,
            refs_content,
            f"Referencias YouTube — {TEMA} ({FECHA})"
        )
        print(f"Referencias añadidas como fuente. TXT: {REFS_PATH}")

        print(f"\nFuentes totales: {len(added)} vídeos + 1 doc referencias")
        if failed:
            print(f"Fallidos: {[v['url'] for v in failed]}")
        return notebook_id

asyncio.run(main())
```

---

## FASE 4 — Informe de la investigación

Tras crear el cuaderno, generar un resumen ejecutivo para el usuario:

```markdown
## Investigación YouTube: [TEMA]

### Estrategia de keywords utilizada
- Iteración 1: [keywords usadas] → [N vídeos encontrados]
- Iteración 2 (si hubo): [keywords añadidas] → [N vídeos adicionales]
- Total buscado: [N queries] | Total analizado: [N vídeos] | Seleccionados: [N]

### Cuaderno NotebookLM creado
- Título: [título del cuaderno]
- ID: [notebook_id]
- Fuentes: [N vídeos] + documento de referencias

### Vídeos indexados
| # | Título | Canal | Fecha | Relevancia |
|---|--------|-------|-------|------------|
| 1 | ...    | ...   | ...   | ⭐⭐⭐⭐⭐ |
...

### Archivo de referencias
`docs/output/[fecha]_[slug]_referencias-youtube.txt`

### Próximos pasos sugeridos
- Preguntar al cuaderno: "¿Cuáles son los puntos clave sobre [tema]?"
- Generar un audio/podcast con el skill notebooklm
- Generar un informe BRIEFING_DOC del cuaderno
```

---

## Guía de decisión para keywords

```
TEMA DADO → ¿Es técnico y específico?
              Sí → 1-2 keywords exactas (ej: "RAG retrieval augmented generation")
              No → ¿Es amplio y multifacético?
                    Sí → 2-3 términos paraguas + refinar en it.2
                    No → 1-2 keywords + sinónimo (ej: "formación online" + "cursos e-learning")

IDIOMA → ¿El tema tiene más contenido en inglés?
           (IA, programación, tech, finanzas, ciencia)
           Sí → Buscar también en EN (duplica el universo de vídeos)
           No → Priorizar ES, reforzar con EN si < 8 vídeos relevantes

RESULTADOS ESCASOS → Añadir: "explicado", "tutorial", "guía completa", "[año actual]"
RESULTADOS IRRELEVANTES → Añadir: entrecomillado, "-" exclusiones, filtros de canal
```

---

## Errores comunes y soluciones

| Error | Causa | Solución |
|-------|-------|---------|
| `add_youtube` falla con URL | URL acortada o con parámetros extra | Usar siempre formato `https://youtube.com/watch?v=VIDEO_ID` |
| Pocos resultados en WebSearch | Keyword demasiado específica | Ampliar a sinónimos o búsqueda en inglés |
| Vídeos duplicados | Misma URL con parámetros distintos | Normalizar URLs antes de añadir |
| Rate limit NotebookLM | Demasiadas fuentes seguidas | `asyncio.sleep(2)` entre cada `add_youtube` |
| `NOTEBOOKLM_AUTH_JSON` no encontrada | `.env` no cargado | Usar `load_env('/workspace/ClaudIA_Agent/.env')` explícito |

---

## Ejemplo de invocación

```
/youtube-research inteligencia artificial en educación
```

Flujo esperado:
1. Evaluar tema → amplio, técnico/pedagógico → buscar en ES + EN
2. It.1: `site:youtube.com "IA en educación"` → 6 vídeos
3. It.2: `site:youtube.com "artificial intelligence education"` → 9 vídeos más
4. Total: 15 vídeos analizados → 12 superan el umbral de relevancia
5. Crear cuaderno NotebookLM con 12 vídeos + referencias.txt
6. Presentar informe al usuario con próximos pasos

---

## Notas importantes

- **Autonomous keyword logic**: No preguntar al usuario qué keywords usar. Razonar y decidir de forma autónoma; informar al usuario de las decisiones tomadas en el resumen final.
- **Calidad > cantidad**: Mejor 8 vídeos muy relevantes que 20 mediocres.
- **Transcripciones**: Si una herramienta como `youtubetranscript.com` o similar devuelve el texto, incluirlo como fuente adicional en NotebookLM (`add_text`) para enriquecer el análisis.
- **Anti rate-limit**: Pausar 2 segundos entre cada `add_youtube`. No lanzar más de 20 fuentes por cuaderno en una sola sesión.
- **Referencias TXT**: Es obligatorio crearlo y añadirlo al cuaderno. Es la memoria permanente de la investigación.
