# Skill: Catálogo de sub-agentes

Inventario de los sub-agentes disponibles en este entorno. **No es una skill ejecutable** — sirve como referencia rápida para decidir cuándo delegar trabajo a un sub-agente.

## Trigger phrases
- "qué agentes tengo disponibles"
- "delegar a un agente"
- "/dev-subagents-catalog"

---

## Sub-agentes nativos de Claude Code

Disponibles vía la herramienta `Agent` con `subagent_type=...`:

| Agente | Cuándo usarlo |
|---|---|
| `Explore` | Búsqueda rápida y solo-lectura para localizar código por patrón, símbolo o concepto. Especificar `quick`, `medium` o `very thorough`. **Usar para:** "¿dónde está X?", "¿qué referencia Y?". **No usar para:** revisión de código, auditorías o análisis abierto. |
| `general-purpose` | Tareas multi-paso, búsquedas complejas o investigación abierta. Tiene acceso a todas las herramientas. **Usar para:** investigaciones web, refactors guiados, ejecutar pipelines. |
| `Plan` | Diseñar plan de implementación antes de tocar código. Devuelve pasos, ficheros críticos y trade-offs. **Usar para:** features nuevas, migraciones, refactors arquitectónicos. |
| `statusline-setup` | Configurar la status line de Claude Code. **Usar solo para eso.** |

---

## Plugins instalados con agente propio

| Plugin | Agente | Función |
|--------|--------|---------|
| `code-simplifier` (oficial) | code-simplifier | Revisa código modificado buscando reuso, calidad y eficiencia, y aplica fixes. Invocable con `/simplify`. |

Ubicación: `~/.claude/plugins/marketplaces/claude-plugins-official/plugins/`

---

## Cómo invocar un sub-agente

```python
# Pseudocódigo del tool-call
Agent({
  description: "Buscar referencias a getCwd",
  subagent_type: "Explore",
  prompt: "Localiza todas las llamadas a getCwd() en /workspace/foo/. Quick search."
})
```

**Reglas:**
- Sub-agentes empiezan **sin contexto previo** — el prompt debe ser autocontenido.
- Para tareas independientes, **invocar varios en paralelo** en un solo mensaje.
- Pedir reportes cortos (`"reporta en menos de 200 palabras"`) si solo necesitas el resumen.
- Verificar siempre lo que dice el sub-agente — su informe describe lo que **intentó hacer**, no necesariamente lo que **hizo**.

---

## Cuándo NO usar un sub-agente

- Si ya sabes el archivo/símbolo: usa `Read`, `Grep` o `Glob` directamente.
- Para tareas de 1-2 pasos triviales: ejecutarlas directamente es más rápido y barato.
- Si necesitas mantener contexto entre pasos en la misma conversación.

---

## Nota sobre la colección "53 agentes" (msitarzewski/agency-agents)

`README.md` y `CLAUDE.md` mencionan 53 agentes especializados de [msitarzewski/agency-agents](https://github.com/msitarzewski/agency-agents) (Engineering, Design, Product, etc.). **Esa colección no está instalada actualmente** en `~/.claude/agents/`. Si se quiere reactivar:

```bash
git clone https://github.com/msitarzewski/agency-agents ~/.claude/agents
# Reiniciar Claude Code para que detecte los .md
```

Tras instalarlos, este fichero debería actualizarse con la lista detallada por categoría.
