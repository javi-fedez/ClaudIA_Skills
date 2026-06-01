---
name: hyperframes-brands
description: Apply per-brand design specs (Marca B, Marca A, Marca C) to HyperFrames video compositions. Use when the user asks for a video, animation, or HTML composition for any of these three brands. This skill copies the brand's `design.md` (and optional `fonts/`, `assets/`) into the project root before the agent starts authoring, so the global hyperframes skill picks up the brand spec automatically. Output language: Spanish, with English only for technical/product names that have higher divulgative weight (LLM, agent, prompt, Copilot, etc.).
---

# Hyperframes — Brand Profiles (ClaudIA)

Centraliza los **brand specs** (`design.md`) de las tres marcas que ClaudIA produce vídeo para. Funciona como un loader: cuando el usuario pida un vídeo HyperFrames para Marca B, Marca A o Marca C, esta skill le dice al agente cómo inyectar el spec correcto en el proyecto.

> Esta skill **no sustituye** ni `hyperframes` ni `house-style.md` ni las paletas globales. Solo aporta el `design.md` de marca, que tiene prioridad sobre house-style según el SKILL.md oficial de hyperframes.

## Marcas registradas

| Slug | Marca | Sector | Fuente | Path |
|---|---|---|---|---|
| `marca-b` | Marca B | Consultoría B2B de IA enterprise | https://marca-b.example.com | `/workspace/ClaudIA_Agent/brands/marca-b/` |
| `marca-a` | Marca A | Programa de adopción de Microsoft Copilot 365 | https://es.marca-a.example.com | `/workspace/ClaudIA_Agent/brands/marca-a/` |
| `marca-c` | Marca C | Marca personal de Javier Fernández — divulgación IA generativa | https://marca-c.example.com | `/workspace/ClaudIA_Agent/brands/marca-c/` |

## Flujo de trabajo

Cuando el usuario diga *"haz un vídeo para Marca B sobre X"*, *"composición HyperFrames para Marca A"*, o equivalente:

### 1. Detecta la marca

Identifica el slug por nombre (`marca-b`, `marca-a`, `marca-c`) o por contexto inequívoco. Si hay ambigüedad, pregunta antes de iniciar el proyecto.

### 2. Crea el proyecto HyperFrames

Antes de invocar `npx hyperframes init` o de generar HTML manualmente, crea/elige el directorio del proyecto. Convención sugerida (alineada con el resto de outputs de ClaudIA):

```
/workspace/ClaudIA_Agent/docs/output/<yyyyMMdd>_<slug-marca>_<titulo-kebab>/
```

### 3. Copia el brand spec antes de empezar

```bash
BRAND=marca-b            # o marca-a / marca-c
PROJECT=/workspace/ClaudIA_Agent/docs/output/20260502_marca-b_demo

mkdir -p "$PROJECT"
cp /workspace/ClaudIA_Agent/brands/$BRAND/design.md "$PROJECT/design.md"

# Si la marca tiene fonts custom (.woff2):
if [ -d /workspace/ClaudIA_Agent/brands/$BRAND/fonts ] && [ -n "$(ls -A /workspace/ClaudIA_Agent/brands/$BRAND/fonts 2>/dev/null)" ]; then
  cp -r /workspace/ClaudIA_Agent/brands/$BRAND/fonts "$PROJECT/fonts"
fi

# Si la marca tiene assets (logo, imágenes):
if [ -d /workspace/ClaudIA_Agent/brands/$BRAND/assets ] && [ -n "$(ls -A /workspace/ClaudIA_Agent/brands/$BRAND/assets 2>/dev/null)" ]; then
  cp -r /workspace/ClaudIA_Agent/brands/$BRAND/assets/* "$PROJECT/"
fi
```

### 4. Procede con el flujo normal de hyperframes

A partir de ahí, la skill global `hyperframes` detectará el `design.md` automáticamente (lo dice su `SKILL.md` línea 27) y lo usará como fuente de verdad. Sigue su workflow estándar: discovery (si aplica) → prompt expansion → plan → HTML → preview → render.

### 5. Verificación de marca al terminar

Antes de servir el preview/render, verifica que la composición respeta el `design.md` (esto ya está pautado por hyperframes — pasos 1–6 de "Brand verification" en su `SKILL.md`):

1. Cada hex usado aparece en el `design.md` de la marca.
2. Las fuentes (`Cardo`, `Inter`, `Jost`, `Montserrat`, `League Spartan`) coinciden — sin sustituciones.
3. El radio de esquinas y la elevación coinciden con `rounded` y `elevation` del frontmatter.
4. Ningún ítem de la sección "Don't" del `design.md` está presente.

## Fonts

| Marca | Heading | Body | ¿Built-in en hyperframes? |
|---|---|---|---|
| Marca B | Jost | Jost | Comprobar — si no, descargar `.woff2` a `brands/marca-b/fonts/` |
| Marca A | Montserrat | League Spartan | Montserrat sí (Google Fonts mainstream); League Spartan probablemente no — descargar |
| Marca C | Cardo | Inter | Inter sí; Cardo probablemente no — descargar |

> Si la fuente no es built-in, hyperframes obliga a aportar `.woff2` (`SKILL.md` líneas 29 y 288). Si no hay archivos, **avisa al usuario antes de escribir HTML** ofreciendo el fallback más cercano.

Para descargar Google Fonts en `.woff2` a un directorio local:

```bash
# Ejemplo: League Spartan
mkdir -p /workspace/ClaudIA_Agent/brands/marca-a/fonts
cd /workspace/ClaudIA_Agent/brands/marca-a/fonts
# Descarga manual desde fonts.google.com → Download family → extraer .woff2
# o usar google-webfonts-helper (https://gwfh.mranftl.com/fonts) para .woff2 directos
```

## Mantenimiento

- **Actualizar un brand spec:** edita `/workspace/ClaudIA_Agent/brands/<slug>/design.md`. El cambio aplica al siguiente proyecto que copie el spec, sin afectar a vídeos ya renderizados.
- **Añadir una marca nueva:** crea `/workspace/ClaudIA_Agent/brands/<nuevo-slug>/` con su `design.md` siguiendo el mismo formato (YAML frontmatter + secciones Overview / Colors / Typography / Layout / Elevation / Components / Voice / Do's and Don'ts) y añade una fila a la tabla de "Marcas registradas" arriba.
- **Cambiar globalmente la dirección creativa para todos los vídeos** (independiente de marca): hay que tocar el `house-style.md` global en `~/.agents/skills/hyperframes/house-style.md`. **No es lo habitual** — solo si tienes una regla universal (p. ej. "todos los vídeos en español por defecto").

## Anti-patrones

- ❌ **No** mezcles dos marcas en un mismo `design.md` — una composición = una marca.
- ❌ **No** edites el `house-style.md` global para meter colores de marca. Eso rompe la separación brand-vs-creative-direction de hyperframes.
- ❌ **No** añadas paletas de marca a `~/.agents/skills/hyperframes/palettes/` — ese directorio es taxonómico por mood (bold, corporate, dark, etc.), no por marca, y se sobrescribe al reinstalar.
- ❌ **No** inventes hex que no estén en el `design.md`. El framework lo verifica explícitamente al final (SKILL.md línea 352).

## Referencias

- Skill global: `~/.agents/skills/hyperframes/SKILL.md` (mecanismo `design.md`, líneas 27–37 y 350–358).
- House-style global: `~/.agents/skills/hyperframes/house-style.md` (defaults creativos, no de marca).
- CLI: `~/.agents/skills/hyperframes-cli/` (comandos `init`, `preview`, `render`, `lint`, `validate`).
