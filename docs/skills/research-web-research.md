# Skill: Web Research

Capacidad nativa de ClaudIA para buscar en internet y extraer información de páginas web **sin necesidad de APIs de terceros**.

---

## Herramientas disponibles

| Herramienta | Función | Cuándo usarla |
|-------------|---------|---------------|
| `WebSearch` | Búsqueda en Google/web | Encontrar páginas, noticias, tendencias |
| `WebFetch` | Extraer contenido de una URL | Leer artículos, documentación, precios |

---

## Patrones de uso

### Búsqueda básica de tendencias

```
Tarea: "Investiga las tendencias en formación corporativa en España 2026"

Pasos:
1. WebSearch("tendencias formación corporativa España 2026")
2. WebFetch(URL de los 3 primeros resultados relevantes)
3. Sintetizar en informe estructurado
```

### Análisis de competidores

```
Tarea: "Analiza los precios y oferta de [competidor]"

Pasos:
1. WebSearch("[empresa] cursos precios formación")
2. WebFetch(web del competidor)
3. WebFetch(páginas de precios y catálogo)
4. Comparativa en tabla
```

### Investigación de tema de curso

```
Tarea: "¿Qué debe incluir un curso de liderazgo para mandos medios?"

Pasos:
1. WebSearch("mejores cursos liderazgo mandos medios contenidos 2026")
2. WebFetch(3-5 páginas de referencia)
3. Extraer temarios y estructurarlos
```

### Seguimiento de noticias del sector

```
Tarea: "Novedades en formación bonificada FUNDAE esta semana"

Pasos:
1. WebSearch("FUNDAE novedades 2026 formación bonificada")
2. WebSearch("cambios formación continua empresa España 2026")
3. Resumen ejecutivo con fuentes
```

---

## Limitaciones

- Solo disponible en región US (WebSearch)
- WebFetch no puede acceder a páginas que requieren login
- Para PDFs de más de 10 páginas usar parámetro `pages`

---

## Ejemplo de prompt para ClaudIA

```
Haz una investigación de mercado sobre cursos de inteligencia artificial
para empresas en España. Quiero saber: precios habituales, temarios más
demandados, y los 5 principales competidores. Formatea el resultado en
un informe ejecutivo de máximo 2 páginas.
```
