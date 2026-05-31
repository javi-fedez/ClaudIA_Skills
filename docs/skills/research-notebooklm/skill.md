# Skill: NotebookLM — Consultas, Generación de Contenido y Automatización

Skill completo para conectarse a Google NotebookLM de forma programática usando
la librería **notebooklm-py**. Permite gestionar notebooks, añadir fuentes,
hacer preguntas con citas, y generar audios, vídeos, presentaciones, informes,
quizzes, mapas mentales e infografías directamente desde ClaudIA.

> **Basado en:** https://github.com/teng-lin/notebooklm-py
> **Estado:** Beta activo (APIs internas de Google, puede cambiar sin aviso)
> **Última verificación:** 2026-03-15 — patrones probados y corregidos en producción

---

## Trigger phrases
- "consulta mi notebooklm"
- "crea un notebook sobre [tema]"
- "añade esta URL a notebooklm"
- "genera un audio/podcast de [notebook]"
- "crea una presentación sobre [tema]"
- "genera un quiz de [notebook]"
- "haz un informe / briefing de [notebook]"
- "crea un mapa mental de [notebook]"
- "pregunta a notebooklm sobre [tema]"
- "/notebooklm"

---

## ⚠️ ERRORES COMUNES Y CORRECCIONES CRÍTICAS

Estos errores se han detectado en producción. Leer antes de escribir código.

| ❌ MAL (NO funciona) | ✅ BIEN (verificado) |
|----------------------|----------------------|
| `from notebooklm import NotebookLM` | `from notebooklm import NotebookLMClient` |
| `NotebookLMClient.from_storage()` | Auth manual: `extract_cookies_from_storage` + `fetch_tokens` + `AuthTokens` |
| `from notebooklm import AuthTokens` desde `types` | `from notebooklm import AuthTokens` (desde raíz) |
| `from notebooklm.auth import from_storage` | No existe. Usar `load_auth_from_storage(path)` solo si tienes Path, pero devuelve dict, no AuthTokens |
| `client.notebooks()` | `await client.notebooks.list()` |
| `status.id` en GenerationStatus | `status.task_id` |
| `status.is_completed` | `status.is_complete` |
| `client.artifacts.generate_infographic(nb_id)` sin args extra | Falla silenciosamente. Pasar siempre `language`, `orientation`, `detail_level`, `style` |
| `download_infographic(nb_id, artifact_id=x, output_path=p)` | `download_infographic(nb_id, output_path_str, artifact_id=x)` — `output_path` es el 2º arg posicional |
| `ref.number` en ChatReference | No existe. Iterar `result.references` directamente |
| Usar dotenv con `from dotenv import load_dotenv; load_dotenv()` | Funciona en Python, pero **la CLI no carga .env** — usar siempre el cliente Python |
| `async with await NotebookLMClient(tokens) as client` | `async with NotebookLMClient(tokens) as client` — sin `await` |

---

## Instalación y Setup inicial

```bash
# Instalación básica (API Python + CLI)
uv pip install notebooklm-py

# Con soporte de login por navegador (recomendado para primera auth)
uv pip install "notebooklm-py[browser]"
playwright install chromium
playwright install-deps chromium  # Solo Linux
```

---

## Patrón base de autenticación (VERIFICADO — usar siempre este)

> ClaudIA usa `NOTEBOOKLM_AUTH_JSON` en `.env`. El cliente NO puede leerla
> directamente. Hay que hacer la auth manual con las funciones de bajo nivel.
> Además, los métodos de descarga necesitan `~/.notebooklm/storage_state.json`
> en disco — guardarlo siempre al inicio.

```python
import asyncio, json, os, pathlib
from notebooklm import NotebookLMClient, AuthTokens
from notebooklm.auth import extract_cookies_from_storage, fetch_tokens

def load_env(path='.env'):
    """Carga .env manualmente sin dependencia de python-dotenv."""
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
            if key and not os.environ.get(key):
                os.environ[key] = val

def get_auth_json() -> dict:
    """Obtiene el JSON de auth y lo guarda en disco (requerido para descargas)."""
    load_env()
    raw = os.environ.get('NOTEBOOKLM_AUTH_JSON', '')
    if not raw:
        raise ValueError("NOTEBOOKLM_AUTH_JSON no encontrada en .env")
    data = json.loads(raw)
    # Guardar en disco — requerido por download_infographic y otros métodos
    storage_path = pathlib.Path.home() / '.notebooklm' / 'storage_state.json'
    storage_path.parent.mkdir(parents=True, exist_ok=True)
    storage_path.write_text(json.dumps(data))
    return data

async def get_client_tokens():
    """Devuelve AuthTokens listos para usar con NotebookLMClient."""
    auth_json = get_auth_json()
    cookies = extract_cookies_from_storage(auth_json)
    csrf, session_id = await fetch_tokens(cookies)
    return AuthTokens(cookies=cookies, csrf_token=csrf, session_id=session_id)

# Uso:
async def main():
    tokens = await get_client_tokens()
    async with NotebookLMClient(tokens) as client:   # ← SIN await antes del with
        # operaciones aquí
        pass

asyncio.run(main())
```

---

## Herramientas disponibles (APIs del cliente Python)

| API | Objeto | Acciones principales |
|-----|--------|----------------------|
| Notebooks | `client.notebooks` | list, create, delete, get_description, rename, share |
| Sources | `client.sources` | list, add_url, add_youtube, add_file, add_text, delete |
| Chat | `client.chat` | ask, get_history, save_as_note |
| Artifacts | `client.artifacts` | generate_*, wait_for_completion, download_*, list, list_infographics |
| Research | `client.research` | start, poll, import_sources |
| Notes | `client.notes` | create, list, update, delete |

---

## Workflows por caso de uso

### 1. Listar notebooks

```python
async def main():
    tokens = await get_client_tokens()
    async with NotebookLMClient(tokens) as client:
        notebooks = await client.notebooks.list()   # ← await obligatorio
        for nb in notebooks:
            print(f"{nb.id} | {nb.title}")

asyncio.run(main())
```

### 2. Descripción / resumen de un notebook

```python
desc = await client.notebooks.get_description(notebook_id)
print(desc.summary)
for topic in desc.suggested_topics:
    print(f"  - {topic.question}")
```

### 3. Chat — Preguntas con citas

```python
result = await client.chat.ask(notebook_id, "¿Cuáles son los puntos clave?")
print(result.answer)          # respuesta con citas [1], [2]...
# result.references → lista de ChatReference; NO tienen atributo .number
for ref in result.references:
    print(ref)
```

### 4. Fuentes

```python
# Listar
sources = await client.sources.list(notebook_id)
for s in sources:
    print(f"{s.id} | {s.title}")

# Añadir
await client.sources.add_url(notebook_id, "https://docs.anthropic.com")
await client.sources.add_youtube(notebook_id, "https://youtube.com/watch?v=...")
await client.sources.add_text(notebook_id, "Contenido...", "Título fuente")
await client.sources.add_file(notebook_id, "/ruta/local/doc.pdf")
```

---

## Generación de Artefactos — Patrón completo

> **GenerationStatus** es el objeto retornado por todos los `generate_*`.
> Tiene `.task_id` (no `.id`), `.status`, `.is_complete` (no `.is_completed`).

### Flujo genérico (aplica a audio, vídeo, informe, slides, quiz, etc.)

```python
# 1. Lanzar generación → devuelve GenerationStatus
status = await client.artifacts.generate_TIPO(notebook_id, ...)
task_id = status.task_id    # ← usar .task_id, NO .id

# 2. Polling hasta completar
while not status.is_complete and not status.is_failed:
    await asyncio.sleep(5)
    status = await client.artifacts.wait_for_completion(notebook_id, task_id)

if status.is_failed:
    print(f"Error: {status.error}")
    return

# 3. Descargar
```

### Infografía (VERIFICADO)

```python
from notebooklm import (
    NotebookLMClient, AuthTokens,
    InfographicOrientation, InfographicDetail, InfographicStyle
)

async def generar_infografia(notebook_id: str, output_path: str):
    tokens = await get_client_tokens()
    async with NotebookLMClient(tokens) as client:
        # Generar — pasar siempre language, orientation, detail_level, style
        status = await client.artifacts.generate_infographic(
            notebook_id,
            language='es',                              # o 'en'
            orientation=InfographicOrientation.PORTRAIT, # PORTRAIT | LANDSCAPE | SQUARE
            detail_level=InfographicDetail.DETAILED,     # CONCISE | STANDARD | DETAILED
            style=InfographicStyle(1),                   # 1-11 estilos disponibles
            instructions="Tema o foco específico de la infografía"
        )
        task_id = status.task_id

        # Polling
        while not status.is_complete and not status.is_failed:
            await asyncio.sleep(5)
            status = await client.artifacts.wait_for_completion(notebook_id, task_id)

        if status.is_failed:
            raise RuntimeError(f"Infografía fallida: {status.error}")

        # Descargar — output_path es el 2º arg posicional, artifact_id es keyword
        result_path = await client.artifacts.download_infographic(
            notebook_id,
            output_path,            # ← 2º arg posicional (string de ruta)
            artifact_id=task_id     # ← keyword arg
        )
        return result_path

# Uso:
import asyncio
from claudia.config import OUTPUT_DIR
OUTPUT_DIR.mkdir(parents=True, exist_ok=True)
asyncio.run(generar_infografia(
    "TU_NOTEBOOK_ID",
    str(OUTPUT_DIR / "20260315_mi-infografia.png")
))
```

### Audio / Podcast

```python
from notebooklm import AudioFormat

status = await client.artifacts.generate_audio(
    notebook_id,
    format=AudioFormat.DEEP_DIVE   # DEEP_DIVE | BRIEF | CRITIQUE | DEBATE
)
task_id = status.task_id

while not status.is_complete and not status.is_failed:
    await asyncio.sleep(5)
    status = await client.artifacts.wait_for_completion(notebook_id, task_id)

mp3 = await client.artifacts.download_audio(notebook_id, task_id, format="mp3")
from claudia.config import OUTPUT_DIR
(OUTPUT_DIR / "podcast.mp3").write_bytes(mp3)
```

### Informe / Briefing

```python
from notebooklm import ReportFormat

# Formatos: BRIEFING_DOC | STUDY_GUIDE | BLOG_POST | FAQ
status = await client.artifacts.generate_report(
    notebook_id,
    format=ReportFormat.BRIEFING_DOC,
    append="Instrucciones adicionales opcionales"
)
task_id = status.task_id

while not status.is_complete and not status.is_failed:
    await asyncio.sleep(5)
    status = await client.artifacts.wait_for_completion(notebook_id, task_id)

pdf = await client.artifacts.download_report(notebook_id, task_id, format="pdf")
from claudia.config import OUTPUT_DIR
(OUTPUT_DIR / "informe.pdf").write_bytes(pdf)
```

### Presentación (Slides)

```python
status = await client.artifacts.generate_slide_deck(notebook_id, style="professional")
task_id = status.task_id

while not status.is_complete and not status.is_failed:
    await asyncio.sleep(5)
    status = await client.artifacts.wait_for_completion(notebook_id, task_id)

pptx = await client.artifacts.download_slide_deck_pptx(notebook_id, task_id)
from claudia.config import OUTPUT_DIR
(OUTPUT_DIR / "presentacion.pptx").write_bytes(pptx)
```

### Quiz

```python
from notebooklm import QuizDifficulty

status = await client.artifacts.generate_quiz(
    notebook_id,
    difficulty=QuizDifficulty.HARD   # EASY | MEDIUM | HARD
)
task_id = status.task_id

while not status.is_complete and not status.is_failed:
    await asyncio.sleep(5)
    status = await client.artifacts.wait_for_completion(notebook_id, task_id)

quiz_data = await client.artifacts.download_quiz_json(notebook_id, task_id)
```

### Mapa Mental

```python
status = await client.artifacts.generate_mind_map(notebook_id)
task_id = status.task_id

while not status.is_complete and not status.is_failed:
    await asyncio.sleep(5)
    status = await client.artifacts.wait_for_completion(notebook_id, task_id)

mindmap_data = await client.artifacts.download_mind_map_json(notebook_id, task_id)
```

---

## Listar artefactos existentes de un notebook

```python
from notebooklm import ArtifactType

# Todos los artefactos
artifacts = await client.artifacts.list(notebook_id)

# Solo infografías
infographics = await client.artifacts.list_infographics(notebook_id)

for a in artifacts:
    print(f"{a.id} | {a.kind} | status={a.status} | {a.title}")
```

---

## Workflow completo: Pregunta + Infografía

```python
import asyncio, json, os, pathlib
from datetime import datetime
from notebooklm import (
    NotebookLMClient, AuthTokens,
    InfographicOrientation, InfographicDetail, InfographicStyle
)
from notebooklm.auth import extract_cookies_from_storage, fetch_tokens

def load_env(path='.env'):
    if not pathlib.Path(path).exists(): return
    with open(path) as f:
        for line in f:
            line = line.strip()
            if not line or line.startswith('#'): continue
            if '=' not in line: continue
            eq = line.index('=')
            key, val = line[:eq].strip(), line[eq+1:].strip().split(' #')[0].strip().strip('"\'')
            if key and not os.environ.get(key): os.environ[key] = val

async def main(notebook_id: str, pregunta: str, tema_infografia: str):
    load_env()
    auth_json = json.loads(os.environ['NOTEBOOKLM_AUTH_JSON'])

    # Guardar en disco (requerido para descargas)
    sp = pathlib.Path.home() / '.notebooklm' / 'storage_state.json'
    sp.parent.mkdir(parents=True, exist_ok=True)
    sp.write_text(json.dumps(auth_json))

    cookies = extract_cookies_from_storage(auth_json)
    csrf, session_id = await fetch_tokens(cookies)
    tokens = AuthTokens(cookies=cookies, csrf_token=csrf, session_id=session_id)

    async with NotebookLMClient(tokens) as client:
        # 1. Pregunta
        result = await client.chat.ask(notebook_id, pregunta)
        print("=== RESPUESTA ===")
        print(result.answer)

        # 2. Infografía
        status = await client.artifacts.generate_infographic(
            notebook_id,
            language='es',
            orientation=InfographicOrientation.PORTRAIT,
            detail_level=InfographicDetail.DETAILED,
            style=InfographicStyle(1),
            instructions=tema_infografia
        )
        task_id = status.task_id
        print(f"Generando infografía (task: {task_id})...")

        while not status.is_complete and not status.is_failed:
            await asyncio.sleep(5)
            status = await client.artifacts.wait_for_completion(notebook_id, task_id)

        if status.is_failed:
            print(f"Error: {status.error}")
            return

        from claudia.config import OUTPUT_DIR
        OUTPUT_DIR.mkdir(parents=True, exist_ok=True)
        fecha = datetime.now().strftime("%Y%m%d")
        slug = tema_infografia.lower().replace(" ", "-")[:40]
        out = OUTPUT_DIR / f"{fecha}_{slug}.png"

        await client.artifacts.download_infographic(notebook_id, str(out), artifact_id=task_id)
        print(f"Infografía guardada: {out}")

asyncio.run(main(
    notebook_id="TU_NOTEBOOK_ID",
    pregunta="¿Cuáles son los factores clave de adopción de IA en empresas?",
    tema_infografia="Adopción IA Generativa en Empresas"
))
```

---

## Convención de archivos de salida

Usar siempre `OUTPUT_DIR` de `claudia.config` — se resuelve automáticamente al project root sin depender del CWD:

```python
from claudia.config import OUTPUT_DIR
OUTPUT_DIR.mkdir(parents=True, exist_ok=True)
out = OUTPUT_DIR / "20260315_infografia-adopcion-ia.png"
```

Formato del nombre: `yyyyMMdd_{titulo}.ext`:
```
<OUTPUT_DIR>/20260315_infografia-adopcion-ia.png
<OUTPUT_DIR>/20260315_podcast-adopcion-ia.mp3
<OUTPUT_DIR>/20260315_informe-adopcion-ia.pdf
<OUTPUT_DIR>/20260315_presentacion-adopcion-ia.pptx
```

---

## Referencia rápida de formatos

| Tipo | Parámetros clave | Descarga | Notas |
|------|-----------------|----------|-------|
| Infografía | `language`, `orientation`, `detail_level`, `style`, `instructions` | PNG via `download_infographic(nb_id, path_str, artifact_id=task_id)` | output_path es 2º arg posicional |
| Audio | `format=AudioFormat.DEEP_DIVE` | MP3 via `download_audio(nb_id, task_id, format="mp3")` | |
| Informe | `format=ReportFormat.BRIEFING_DOC` | PDF via `download_report(nb_id, task_id, format="pdf")` | |
| Slides | `style="professional"` | PPTX via `download_slide_deck_pptx(nb_id, task_id)` | |
| Quiz | `difficulty=QuizDifficulty.HARD` | JSON via `download_quiz_json(nb_id, task_id)` | |
| Mapa mental | — | JSON via `download_mind_map_json(nb_id, task_id)` | |

---

## Depuración y problemas comunes

| Problema | Causa | Solución |
|----------|-------|----------|
| `cannot import name 'NotebookLM'` | Clase incorrecta | Usar `NotebookLMClient` |
| `'GenerationStatus' has no attribute 'id'` | Atributo incorrecto | Usar `status.task_id` |
| `'is_completed' not found` | Atributo incorrecto | Usar `status.is_complete` |
| `FileNotFoundError: ~/.notebooklm/storage_state.json` | Falta archivo en disco | Llamar `get_auth_json()` al inicio — guarda el JSON en disco |
| `TypeError: got multiple values for output_path` | Arg posicional + keyword | `download_infographic(nb_id, path, artifact_id=x)` — path es posicional |
| `'dict' object has no attribute 'cookie_header'` | `load_auth_from_storage` devuelve dict | Usar el patrón `extract_cookies_from_storage` + `fetch_tokens` + `AuthTokens` |
| `'coroutine' has no len()` | Falta `await` | Todos los métodos del cliente son `async` |
| `RuntimeError: Client not initialized` | Falta `async with` | Usar siempre `async with NotebookLMClient(tokens) as client` |
| `Authentication expired` | Token caducado (semanas) | Actualizar `NOTEBOOKLM_AUTH_JSON` en `.env` |
| Infografía falla silenciosamente | Sin `language`/`style` | Pasar siempre `language='es'`, `orientation`, `detail_level`, `style` |

---

## Notas importantes

- **API no oficial** — Google puede cambiar las APIs internas sin previo aviso.
- **Async** — Todo el cliente Python es `async/await`. Usar siempre dentro de `asyncio.run()`.
- **storage_state.json en disco** — Los métodos de descarga usan `load_httpx_cookies()` que lee `~/.notebooklm/storage_state.json`. Guardarlo siempre al inicio con `get_auth_json()`.
- **Rate limiting** — No lanzar múltiples generaciones en paralelo; Google puede rechazarlas.
- **Tokens** — Duran días o semanas. Si falla auth, renovar `NOTEBOOKLM_AUTH_JSON` en `.env`.
- **Credenciales** — `~/.notebooklm/` y `.env` nunca se commitean al repo.
