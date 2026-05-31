---
name: ops-ceo-menu
description: Panel de 20 tareas autónomas listas para el CEO de una empresa pequeña de formación (comercial, marketing, operaciones, finanzas, estrategia). Úsalo como menú de tareas de gestión de negocio.
---

# ClaudIA — Menú CEO: Empresa de Formación

Panel de tareas autónomas para el CEO de una empresa de formación pequeña.
Copia y pega el prompt correspondiente directamente en ClaudIA.

---

## COMERCIAL Y VENTAS

### 1. Crear propuesta comercial completa
```
Crea una propuesta de formación completa para [NOMBRE EMPRESA],
empresa del sector [SECTOR] con [N] empleados. Necesidad detectada:
[NECESIDAD]. Presupuesto aproximado: [X] €. Formato en persona [online/presencial/mixto].
Incluye: portada, resumen ejecutivo, diagnóstico, propuesta de contenidos,
metodología, equipo, inversión desglosada y siguiente paso.
Guarda el resultado en /clientes/[nombre-empresa]/propuesta-[fecha].md
```

### 2. Seguimiento de pipeline semanal
```
Revisa el archivo /comercial/pipeline.csv (o .json). Para cada
oportunidad en estado "en negociación" o "propuesta enviada",
redacta un email de seguimiento personalizado. Agrúpalos por
urgencia (fecha de cierre esperada). Guarda los emails en
/comercial/seguimientos/semana-[fecha]/
```

### 3. Análisis de propuestas perdidas
```
Lee todas las propuestas en /clientes/**/propuesta*.md donde el
estado sea "perdida". Identifica patrones: precio, sector, tamaño
de empresa, motivo de rechazo si está anotado. Dame 3 conclusiones
accionables para mejorar la tasa de conversión.
```

### 4. Dossier comercial actualizado
```
Genera un dossier comercial de nuestra empresa de formación en PDF-ready
markdown. Incluye: quiénes somos, propuesta de valor, catálogo de cursos
(lee /cursos/ para extraerlos), metodología, clientes destacados, y CTA.
Tono: profesional y diferenciador. Guarda en /marketing/dossier-[año].md
```

---

## MARKETING Y COMUNICACIÓN

### 5. Calendario editorial mensual
```
Crea el calendario editorial de [MES] [AÑO] para nuestra empresa de formación.
Canales: LinkedIn (3 posts/semana), Newsletter (1/semana), Blog (2/mes).
Tema del mes: [TEMA]. Para cada pieza: título, gancho, CTA, mejor día/hora.
Guarda en /marketing/calendarios/[mes-año].md
```

### 6. Newsletter mensual lista para enviar
```
Redacta la newsletter de [MES]. Estructura: saludo personalizado,
titular llamativo, 1 artículo principal sobre [TEMA], 2 noticias breves
del sector formación, curso destacado del mes, CTA principal.
Tono cercano y experto. Máximo 400 palabras. Guarda en /marketing/newsletters/[mes].md
```

### 7. Pack de posts LinkedIn del mes
```
Genera 12 posts de LinkedIn para todo el mes de [MES] sobre el tema
[TEMA MENSUAL]. Mix de tipos: reflexión (3), consejo práctico (4),
caso real/historia (2), pregunta a la audiencia (2), promocional suave (1).
Para cada uno: texto completo + 3-5 hashtags + mejor horario de publicación.
Guarda en /marketing/linkedin/[mes-año].md
```

### 8. Investigación de mercado y competidores
```
Haz una investigación de mercado completa del sector formación corporativa
en [REGIÓN/PAÍS]. Lanza agentes en paralelo para:
1. Analizar los 5 principales competidores (precios, oferta, posicionamiento)
2. Identificar tendencias y cursos más demandados en 2026
3. Detectar nichos o necesidades no cubiertas

Consolida en un informe de 2 páginas con oportunidades concretas.
Guarda en /research/mercado-[fecha].md
```

---

## OPERACIONES Y GESTIÓN

### 9. Informe ejecutivo mensual
```
Genera el informe ejecutivo de [MES]. Lee los datos disponibles en
/finanzas/, /clientes/ y /cursos/. Incluye:
- Facturación vs objetivo
- Nuevos clientes y clientes activos
- Cursos impartidos y alumnos formados
- Pipeline activo (valor total)
- Top 3 logros del mes
- Top 3 prioridades para el mes siguiente
Formato tabla + texto. Guarda en /informes/[mes-año]-ejecutivo.md
```

### 10. Planificación de objetivos trimestrales (OKRs)
```
Ayúdame a definir los OKRs de Q[N] [AÑO] para nuestra empresa de formación.
Contexto: [BREVE DESCRIPCIÓN SITUACIÓN ACTUAL]. Áreas: Comercial, Marketing,
Operaciones, Producto (cursos), Finanzas. Para cada OKR: objetivo ambicioso
+ 3 key results medibles + iniciativas clave. Guarda en /estrategia/okr-q[n]-[año].md
```

### 11. Onboarding de nuevo cliente
```
Crea el pack de bienvenida y checklist de onboarding para el cliente
[NOMBRE EMPRESA] que acaba de contratar [SERVICIO/CURSO]. Incluye:
email de bienvenida, checklist interno de tareas (agenda kick-off, compartir
materiales, accesos, etc.), y guía del alumno/responsable. Guarda en
/clientes/[nombre]/onboarding.md
```

### 12. Revisión de contratos activos
```
Lee todos los contratos en /clientes/**/contrato*.md. Para cada uno extrae:
fecha de inicio, fecha de fin, importe, estado de pago, y cláusula de renovación.
Identifica: contratos que vencen en los próximos 60 días, contratos sin confirmar
pago, y oportunidades de upsell. Genera alerta en /operaciones/alertas-[fecha].md
```

---

## PRODUCTO Y FORMACIÓN

### 13. Diseñar nuevo curso desde cero
```
Diseña un curso completo sobre [TEMA] para el público objetivo [PERFIL].
Formato: [presencial/online/blended], [N] horas en [N] sesiones.
Entrega: temario completo por módulos (con objetivos, contenidos, actividades
y evaluación), guía del formador, presentación estructura (outline), y
ficha comercial del curso (para web/propuestas). Guarda en /cursos/[nombre-curso]/
```

### 14. Actualizar material existente con IA/tendencias
```
Lee el temario en /cursos/[nombre-curso]/temario.md. Investiga en web las
últimas tendencias, herramientas o cambios normativos relevantes para este
curso en 2026. Propón qué módulos están desactualizados, qué contenido nuevo
añadir, y qué eliminar. Genera el temario v2 en /cursos/[nombre-curso]/temario-v2.md
```

### 15. Evaluación y encuesta de satisfacción
```
Crea un cuestionario de evaluación para el curso [NOMBRE CURSO].
Incluye: evaluación de conocimientos (10 preguntas tipo test), encuesta
de satisfacción NPS (5 preguntas escala + 2 abiertas), y autoevaluación
de competencias (antes/después). Formato markdown exportable a formulario.
Guarda en /cursos/[nombre-curso]/evaluacion.md
```

---

## FINANZAS Y ADMINISTRACIÓN

### 16. Análisis de facturación y rentabilidad
```
Lee los datos en /finanzas/facturas-[año].csv. Genera un análisis de:
1. Facturación mensual y acumulada vs objetivo anual
2. Rentabilidad por tipo de curso/servicio
3. Clientes por volumen (Pareto: quién genera el 80% de ingresos)
4. Facturas pendientes de cobro y días de deuda media
5. Proyección de cierre de año

Guarda el informe en /finanzas/analisis-[fecha].md
```

### 17. Presupuesto de proyecto/curso
```
Calcula el presupuesto para impartir el curso [NOMBRE CURSO] a [N] alumnos
de la empresa [CLIENTE]. Considera:
- Coste del formador: [X] €/hora
- Duración: [N] horas
- Materiales: [X] €/alumno
- Desplazamiento: [X] €
- Plataforma/tecnología: [X] €
- Margen objetivo: [X]%
Genera presupuesto con precio de venta recomendado y punto de equilibrio.
```

### 18. Informe para gestoría/asesoría
```
Prepara el resumen mensual de [MES] para la gestoría. Lee /finanzas/facturas-[mes].csv
y /finanzas/gastos-[mes].csv. Genera: listado de ingresos con datos fiscales,
listado de gastos deducibles clasificados, y nota de cualquier operación especial.
Formato: tabla limpia en markdown. Guarda en /finanzas/gestoría/[mes-año].md
```

---

## ESTRATEGIA Y CRECIMIENTO

### 19. Plan de captación de nuevos clientes
```
Crea un plan de captación de clientes para Q[N] [AÑO]. Objetivo: [N] nuevos
clientes. Investiga en web empresas del sector [SECTOR] en [REGIÓN] con más
de [N] empleados. Para los 20 más prometedores: nombre, sector, tamaño estimado,
cargo del decisor de compra típico, y approach recomendado.
Guarda la lista en /comercial/prospecting-q[n].md
```

### 20. Análisis DAFO actualizado
```
Genera un análisis DAFO actualizado de nuestra empresa de formación para [AÑO].
Investiga el mercado actual, la situación del sector y las tendencias en IA/formación.
Lee nuestro /marketing/dossier*.md para conocer nuestra propuesta de valor actual.
Sé específico y accionable en cada cuadrante. Incluye 3 estrategias derivadas del DAFO.
Guarda en /estrategia/dafo-[año].md
```

---

## USO RÁPIDO

Para tareas urgentes sin configuración:

```
ClaudIA, necesito [TAREA] para [CLIENTE/SITUACIÓN] antes de [HORA/DÍA].
Prioridad máxima.
```

**Ejemplos directos:**
- `"Prepara un email de disculpa para Empresa X por el retraso en la entrega de materiales"`
- `"Busca las 10 empresas más grandes del sector logístico en Madrid"`
- `"Calcula cuánto debo cobrar por un curso de 20 horas con margen del 40%"`
- `"Resume este contrato en 5 puntos clave" [adjuntar archivo]`
- `"¿Cuánto hemos facturado este trimestre?" [si hay datos en /finanzas/]`
