---
name: comms-marca-a-video
description: Edita videos para marca-a.example.com (reels, tutoriales, minilecciones del programa "21 días adoptando IA / Copilot 365") usando video-use + ffmpeg. Workflow completo de transcripción → análisis → cortes → render → títulos animados con el branding de Marca A (paleta blanco/amarillo/rojo, Montserrat Black, 3 tamaños fijos, lateral derecho, sin sombra). Invocar SIEMPRE que Javi quiera editar un video para marca-a, generar títulos animados estilo Marca A, aplicar la identidad visual al video, o cuando mencione "video marca-a", "reel marca-a", "subtítulos laterales", "títulos para video", "edición de reel".
---

# Marca A Video Editor

Workflow completo para producir videos del programa **21 días adoptando IA** (Copilot 365) con identidad visual consistente, ejecutado en el VPS Linux.

## Cuándo usar

Javi tiene material crudo (talking head, screen recording, demos) y quiere convertirlo en pieza pulida con cortes precisos y títulos animados que sigan el branding de Marca A.

Tipos de pieza:
- **Reels** (Instagram/TikTok/YouTube Shorts) — verticales o cuadrados, 30-90s
- **Tutoriales** — horizontales, 2-10 min, demos de Copilot
- **Minilecciones** — del programa 21 días, conectan con la web

## Stack y rutas (Linux VPS)

```
Proyecto/output:   /workspace/ClaudIA_Agent/docs/output/<yyyyMMdd>_<slug>-video/
Material crudo:    <output>/raw-footage/    (donde el usuario suba el .mov/.mp4)
video-use:         /root/.claude/skills/video-use/   (helpers de Python)
ElevenLabs key:    /root/.claude/skills/video-use/.env  (ELEVENLABS_API_KEY ya configurada)
ffmpeg:            /usr/bin/ffmpeg  (system-wide, soporta libass)
Brand spec:        /workspace/ClaudIA_Agent/brands/marca-a/design.md
Fonts (.woff2):    /workspace/ClaudIA_Agent/brands/marca-a/fonts/        (web/HyperFrames)
Fonts (.ttf):      /workspace/ClaudIA_Agent/brands/marca-a/fonts/ttf/    (Montserrat-Black/Bold/Medium/Regular para libass)
```

> **Nota Linux**: en este entorno **no** hace falta `PYTHONIOENCODING=utf-8` (era un workaround Windows). Tampoco hay que tunear el PATH de ffmpeg — está instalado vía `apt`.

> **Por qué .ttf y no .woff2**: libass (motor de subtítulos de ffmpeg) sólo carga `.ttf`/`.otf`. El brand mantiene `.woff2` para web/HyperFrames; para vídeo usamos los `.ttf` estáticos en `fonts/ttf/`.

## Workflow (8 pasos)

### 1. Preparar la carpeta del proyecto

```bash
SLUG="reel-21-dias"   # ajustar al contenido
DATE=$(date +%Y%m%d)
PROJ=/workspace/ClaudIA_Agent/docs/output/${DATE}_${SLUG}-video
mkdir -p "$PROJ/raw-footage/edit"
```

El usuario sube su `.mov`/`.mp4` a `$PROJ/raw-footage/`. Sin renombrar — los nombres tipo `REEL IMPRO 022A3608_CHROMA.mov` son la convención de Javi.

### 2. Transcribir con ElevenLabs Scribe

```bash
cd "$PROJ"
python /root/.claude/skills/video-use/helpers/transcribe.py "raw-footage/VIDEO.mov"
```

Costo: ~$0.40 / 10 min de audio. Output: `raw-footage/edit/transcripts/VIDEO.json` con timestamps word-level.

### 3. Pack transcripts

```bash
cd "$PROJ/raw-footage"
python /root/.claude/skills/video-use/helpers/pack_transcripts.py --edit-dir edit
```

Genera `edit/takes_packed.md` con frases agrupadas por silencios ≥0.5s.

### 4. Analizar — qué cortar

Lee tanto `takes_packed.md` (macro) como el JSON word-level (micro). Identifica:

| Patrón | Ejemplo | Acción |
|---|---|---|
| Falsos inicios | `"una..."` + silencio largo | Cortar |
| Muletillas | `"eh,"`, `"ah,"`, `"umm"` | Cortar |
| Repeticiones | `"y, y adoptar"` | Conservar uno |
| Frases incompletas | `"sin--"`, `"eh, la empresa..."` | Cortar todo el fragmento |
| Doble palabra cercana | `"realmente... realmente"` | Conservar uno |
| Silencios > 0.5s | gaps entre palabras | Cortar |
| Auto-correcciones | `"inmetido, involucrado, iba a decir, inmerso"` | Mantener solo "inmerso" |

**Sílabas en bordes**: si el `end` cae a mitad de palabra, extiende o recorta hasta una pausa limpia.

Comando útil para volcar palabras con silencios marcados:

```bash
python3 -c "
import json, pathlib
data = json.loads(pathlib.Path('raw-footage/edit/transcripts/VIDEO.json').read_text(encoding='utf-8'))
prev = 0
for w in data['words']:
    gap = w['start'] - prev
    g = f'  [SILENCIO {gap:.2f}s]' if gap > 0.4 else ''
    print(f\"{w['start']:07.3f}-{w['end']:07.3f}  {w['type']:10s}  {w['text']}{g}\")
    prev = w['end']
"
```

### 5. Construir EDL JSON

`raw-footage/edit/edl.json`:

```json
{
  "sources": {"REEL": "../VIDEO.mov"},
  "grade": "eq=brightness=0.06:contrast=1.02:saturation=1.04",
  "ranges": [
    {"source": "REEL", "start": 1.260, "end": 13.600, "note": "primera frase limpia"},
    {"source": "REEL", "start": 16.040, "end": 42.600, "note": "cortado: 'inmetido, involucrado, iba a decir,'"}
  ]
}
```

Decisiones de grade:
- Talking head con luz buena: `"eq=brightness=0.06:contrast=1.02:saturation=1.04"`
- Talking head subexpuesto: `"eq=brightness=0.10:contrast=1.05:saturation=1.05"`
- Material ya bien: `""`
- **Nunca** `warm_cinematic` para talking head (aplasta negros + grano).

### 6. Render base con video-use

```bash
cd "$PROJ/raw-footage/edit"
python /root/.claude/skills/video-use/helpers/render.py edl.json -o draft.mp4 --no-subtitles
```

Aplica grade, concat lossless, normaliza loudness a -14 LUFS / -1 dBTP (audio social-ready).

### 7. (Opcional) Speed change con dos pasos

`.mov` profesionales tienen timecode offset (`start: 0.083s`) que descompasa video/audio si se acelera naïve. Receta correcta:

```bash
# A. Normalizar timecode
ffmpeg -y -i draft.mp4 \
  -c:v libx264 -preset fast -crf 18 -c:a aac -b:a 192k \
  -fps_mode cfr -r 24 -avoid_negative_ts make_zero \
  draft_clean.mp4

# B. Aplicar speed reseteando PTS de ambos streams
SPEED=1.15
ffmpeg -y -i draft_clean.mp4 \
  -filter_complex "[0:v]setpts=PTS/${SPEED},setpts=PTS-STARTPTS[v];[0:a]atempo=${SPEED},asetpts=PTS-STARTPTS[a]" \
  -map "[v]" -map "[a]" \
  -c:v libx264 -preset fast -crf 18 -c:a aac -b:a 192k \
  draft_speed.mp4
```

> Si **no** vas a hacer speed change, igualmente conviene generar `draft_clean.mp4` (paso A) para el burn-in posterior — evita rarezas de re-encode.

### 8. Títulos animados (lo importante)

#### 8.1 Mapear timestamps source → output

```python
import json, pathlib
data = json.loads(pathlib.Path('transcripts/VIDEO.json').read_text(encoding='utf-8'))
edl = json.loads(pathlib.Path('edl.json').read_text(encoding='utf-8'))
words = [w for w in data['words'] if w['type'] == 'word']

def src_to_out(src_t):
    offset = 0.0
    for r in edl['ranges']:
        s, e = float(r['start']), float(r['end'])
        if s <= src_t <= e:
            return offset + (src_t - s)
        offset += (e - s)
    return None

for kw in ['encima', 'inmerso', '2026', 'sobrevivir']:
    for w in words:
        if w['text'].lower().rstrip(',.;:?!') == kw:
            out_t = src_to_out(w['start'])
            print(f"{kw}: src={w['start']:.2f} → out={out_t:.2f}")
            break
```

#### 8.2 Generar `lateral_titles.ass`

Copia el template `gen_titles_template.py` que vive junto a esta skill (ver `skills/assets/aua_video/gen_titles_template.py`) a `raw-footage/edit/gen_titles.py` y rellena los `card()`. Reglas innegociables: ver `## Reglas de estilo` abajo.

```bash
cp /workspace/ClaudIA_Agent/skills/assets/aua_video/gen_titles_template.py raw-footage/edit/gen_titles.py
# editar gen_titles.py con las cards reales
python3 raw-footage/edit/gen_titles.py
```

Si los `assert` saltan, el script aborta — ajusta antes de continuar.

#### 8.3 Burn-in con libass

```bash
ffmpeg -y -i draft_clean.mp4 \
  -vf "ass=lateral_titles.ass:fontsdir=/workspace/ClaudIA_Agent/brands/marca-a/fonts/ttf" \
  -c:v libx264 -preset fast -crf 17 -c:a copy \
  final.mp4
```

`-c:a copy` para no re-encodear audio (mantiene loudnorm intacto).

Verifica que ffmpeg cargó la fuente correcta en stderr:
```
[Parsed_ass_0] fontselect: (Montserrat Black, 800, 0) -> Montserrat-Black ✓
```
Si aparece fallback a Arial-BoldMT → revisa que `fontsdir` apunta al directorio con los `.ttf`.

## Reglas de estilo Marca A (innegociables)

Las completas en `/workspace/ClaudIA_Agent/brands/marca-a/design.md` y en el bloque "Las 10 reglas de títulos" del workflow. Resumen:

1. **Solo Montserrat Black** escalado por size — una sola fuente = composición coherente
2. **3 tamaños fijos**: N=52 / G=84 / X=94 (1.0× / 1.6× / 1.8×)
3. **Cards multi-línea** mezclando los 3 tamaños — máx 4 líneas, máx 2 palabras/línea
4. **Conceptos completos** en una sola card, no fragmentar entre cards
5. **Lateral derecho exclusivo** (`\an9`, x=1200 en 1280×720)
6. **Cards estrictamente secuenciales** — cero solapamiento temporal (buffer 100ms)
7. **Pre-roll 350ms** — texto en pantalla CUANDO se dice la palabra clave
8. **Pop scale + fade snappy** — 65→108→100% en 220ms, fade 60/100ms
9. **Paleta vídeo: blanco / amarillo `#FFD400` / rojo `#FF3B30`** — nunca rojo+amarillo en mismo card
10. **Flat total**: sin sombra, sin outline, sin fondo, sin glow

> **Decisión brand para vídeo**: paleta es **blanco + amarillo + rojo** (no azul). El amarillo da más contraste sobre fondos variables de talking head y mejor asociación con "destacado/CTA". Regla heredada del brand: "Solo dos accents y NUNCA juntos en la misma frase".

Mapping concepto → color:

| Tipo de mensaje | Color | Ejemplo |
|---|---|---|
| Información neutra | Blanco | "qué hace / la / COMPETENCIA" |
| Hito / proceso / CTA | Amarillo | "estamos en / 2026" |
| Warning / corrección | Rojo | "puestos / AMENAZADOS" |
| Llamado a actuar | Amarillo | "↓ COMÉNTAME / abajo" |
| Consecuencia negativa | Rojo | "no va / a / SOBREVIVIR" |

## Voice / copy

Tono **profesor cercano** del programa "21 días adoptando IA":
- Lowercase como base, MAYÚSCULAS solo para impacto
- "Tú" empresarial, "vosotros" plural
- Frases activas, verbo primero
- Vocabulario adopción: "uso real", "tasa de activación", "champions", "pilots", "ahorro semanal", "change management"
- Anglicismos OK para producto: "Copilot", "prompt", "agent", "KPI"
- **Evitar**: "transformación digital", "imparable", "el cambio es ahora", "descubre el secreto", "revolucionario", clichés pitch

## Pitfalls (Linux)

| Síntoma | Causa | Fix |
|---|---|---|
| `fontselect → Arial-BoldMT` en stderr | `fontsdir` no carga `.ttf` | Apunta a `/workspace/ClaudIA_Agent/brands/marca-a/fonts/ttf` (no a `fonts/` que tiene .woff2) |
| Speed change desync vídeo/audio | Timecode offset MOV | Receta dos pasos del paso 7 (no `setpts/atempo` directo) |
| Sílaba cortada en borde | `segment_end` parte palabra | Mover end a `w['end']+0.05` o pausa anterior |
| `--build-subtitles` no genera SRT | `--no-subtitles` lo cancela | Quitar `--no-subtitles` o generar SRT manual desde JSON+EDL |
| Líneas se pisan dentro de un card | LINE_GAP pequeño | `LINE_GAP >= 10` o `size * 1.15` |
| Cards solapan en tiempo | `card_N.end > card_N+1.start` | Validar con assert: `prev_end <= card.start` |
| Vídeo sale muy oscuro | Grade `warm_cinematic` | Usar `eq=brightness=0.06:contrast=1.02:saturation=1.04` |

## Outputs esperados

```
docs/output/<yyyyMMdd>_<slug>-video/
└── raw-footage/
    ├── VIDEO.mov                              # original
    └── edit/
        ├── transcripts/VIDEO.json             # palabras + timestamps
        ├── takes_packed.md                    # legible
        ├── edl.json                           # decisiones de corte
        ├── draft.mp4                          # base + audio normalizado
        ├── draft_clean.mp4                    # normalizado para overlay
        ├── gen_titles.py                      # script generador
        ├── lateral_titles.ass                 # títulos brand-compliant
        └── final.mp4                          # pieza final
```

Antes de declarar listo: revisar `final.mp4` con Javi, aplicar feedback siguiendo las reglas (no improvisar fuera del brand). Versionar con `_v1`, `_v2`, etc. para comparar iteraciones.
