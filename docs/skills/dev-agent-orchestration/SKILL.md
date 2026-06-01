---
name: dev-agent-orchestration
description: Lanza sub-agentes especializados en paralelo, delega tareas complejas y combina sus resultados. Úsalo para descomponer tareas grandes que conviene paralelizar.
---

# Skill: Orquestación de Agentes

Capacidad de ClaudIA para lanzar **sub-agentes especializados en paralelo**, delegar tareas complejas y combinar sus resultados — todo sin APIs externas.

---

## Trigger phrases
- "lanza agentes en paralelo para [tareas]"
- "investiga a la vez [tema A], [tema B] y [tema C]"
- "delega esto en sub-agentes"
- "descompón esta tarea y paralelízala"
- "/agent-orchestration"

---

## Herramienta disponible

| Herramienta | Función |
|-------------|---------|
| `Agent` | Lanza un sub-agente con su propio contexto, herramientas y tarea específica |

---

## Tipos de sub-agentes disponibles

| Tipo | Especialización |
|------|----------------|
| `general-purpose` | Investigación compleja, búsqueda en código, tareas multi-paso |
| `Explore` | Exploración rápida de directorios y código |
| `Plan` | Diseño de planes de implementación |

---

## Cuándo usar orquestación

Usar sub-agentes cuando la tarea es:
- **Paralelizable**: múltiples investigaciones independientes
- **Larga**: protege el contexto principal
- **Especializada**: cada parte requiere un enfoque diferente
- **Con mucho output**: evita saturar el contexto principal

---

## Patrones de uso para empresa de formación

### Investigación de mercado completa (paralela)

```
ClaudIA lanza 3 agentes en paralelo:
├── Agente 1: "Analiza los 5 principales competidores en España"
├── Agente 2: "Investiga tendencias de formación 2026"
└── Agente 3: "Identifica los cursos más demandados en LinkedIn Learning"

Resultado: Informe consolidado en < 3 minutos
```

### Preparación de propuesta multi-cliente

```
ClaudIA lanza 1 agente por cliente:
├── Agente A: "Prepara propuesta para Empresa A (sector retail)"
├── Agente B: "Prepara propuesta para Empresa B (sector industrial)"
└── Agente C: "Prepara propuesta para Empresa C (sector salud)"

Cada agente: lee el sector, adapta la propuesta, la guarda
```

### Análisis financiero mensual

```
ClaudIA lanza agentes especializados:
├── Agente 1: "Lee y analiza todas las facturas de marzo"
├── Agente 2: "Compara con el mes anterior y el objetivo"
└── Agente 3: "Genera el informe ejecutivo con recomendaciones"
```

---

## Ventajas vs ejecución secuencial

| | Secuencial | Paralelo (Orquestación) |
|--|-----------|------------------------|
| 3 investigaciones | ~9 min | ~3 min |
| 5 propuestas | ~15 min | ~3 min |
| Análisis completo | ~12 min | ~4 min |
| Contexto principal | Se satura | Protegido |

---

## Diseño de tarea para sub-agente

Un buen prompt para un sub-agente incluye:
1. **Objetivo claro y específico**
2. **Fuentes a consultar** (URLs, archivos, búsquedas)
3. **Formato de salida esperado**
4. **Dónde guardar el resultado**

**Ejemplo:**
```
Sub-agente: Investiga a fondo a Competitor X.
Fuentes: su web, LinkedIn, reviews en Google, artículos de prensa 2025-2026.
Formato: tabla con columnas [servicio, precio, duración, diferencial].
Guarda el resultado en /research/competitor-x.md
```

---

## Ejemplo de prompt para ClaudIA

```
Quiero preparar la estrategia comercial de Q2. Lanza agentes en paralelo para:
1. Investigar las 5 empresas de nuestra pipeline con más de 100 empleados
2. Analizar qué cursos vendimos más en Q1 y qué margen tuvieron
3. Buscar convocatorias abiertas de subvenciones para formación en nuestra región

Combina los resultados en un plan de acción de Q2 con prioridades claras.
```

---

## Notas de seguridad
- Cada sub-agente recibe solo el contexto que necesita — no compartir secretos ni datos sensibles innecesarios.
- Delegar **tareas de solo lectura/investigación** en paralelo; las acciones con efectos (enviar, publicar, borrar) deben pasar por el agente principal con confirmación del usuario.
- Definir siempre un objetivo y formato de salida claros para evitar resultados divergentes.
