# Skill: Análisis de Datos

Capacidad de ClaudIA para leer, procesar e interpretar datos de negocio usando Python nativo — sin necesidad de BI externo.

---

## Herramientas combinadas

| Herramienta | Rol |
|-------------|-----|
| `Read` / `Glob` | Localizar los archivos de datos |
| `Bash` (Python) | Procesar con pandas, csv, json |
| `Write` | Guardar informes con resultados |

---

## Tipos de análisis disponibles

### Análisis de ventas y facturación

```python
# ClaudIA ejecuta esto automáticamente
import csv
from collections import defaultdict

ventas_por_curso = defaultdict(float)
with open('facturas.csv') as f:
    for row in csv.DictReader(f):
        ventas_por_curso[row['curso']] += float(row['importe'])

for curso, total in sorted(ventas_por_curso.items(),
                           key=lambda x: -x[1]):
    print(f"{curso}: {total:,.0f} €")
```

**Output:** Ranking de cursos por facturación.

---

### Análisis de pipeline comercial

```python
import json

with open('pipeline.json') as f:
    oportunidades = json.load(f)

por_etapa = defaultdict(list)
for op in oportunidades:
    por_etapa[op['etapa']].append(op['valor'])

for etapa, valores in por_etapa.items():
    print(f"{etapa}: {len(valores)} ops | {sum(valores):,.0f} € potencial")
```

---

### Tasa de conversión de propuestas

```python
enviadas = [p for p in propuestas if p['estado'] == 'enviada']
ganadas  = [p for p in propuestas if p['estado'] == 'ganada']
perdidas = [p for p in propuestas if p['estado'] == 'perdida']

tasa = len(ganadas) / len(enviadas) * 100
ticket_medio = sum(p['valor'] for p in ganadas) / len(ganadas)

print(f"Tasa de cierre: {tasa:.1f}%")
print(f"Ticket medio: {ticket_medio:,.0f} €")
```

---

### Análisis de satisfacción de alumnos

```python
# Procesar encuestas de valoración
import statistics

notas = [float(r['valoracion']) for r in respuestas]
print(f"Media: {statistics.mean(notas):.1f}")
print(f"NPS positivos (>=9): {sum(1 for n in notas if n >= 9)}")
comentarios_negativos = [r['comentario'] for r in respuestas
                         if float(r['valoracion']) < 6]
```

---

### Visualización en texto (sin librerías externas)

```python
def barra_ascii(valor, maximo, ancho=30):
    proporcion = valor / maximo
    llenas = int(proporcion * ancho)
    return '█' * llenas + '░' * (ancho - llenas)

for curso, ventas in ranking:
    barra = barra_ascii(ventas, max_ventas)
    print(f"{curso:<25} {barra} {ventas:>8,.0f} €")
```

---

## Formatos de datos que ClaudIA puede procesar

| Formato | Notas |
|---------|-------|
| `.csv` | Nativo con módulo `csv` de Python |
| `.json` | Nativo con módulo `json` |
| `.xlsx` | Con `openpyxl` si está instalado |
| `.txt` | Lectura directa con `Read` |
| `.md` | Lectura y análisis semántico directo |

---

## Dashboards en Markdown

ClaudIA puede generar informes como este:

```markdown
## Informe Mensual — Marzo 2026

| Métrica | Este mes | Mes anterior | Δ |
|---------|----------|-------------|---|
| Facturación | 18.450 € | 15.200 € | +21% |
| Nuevos clientes | 3 | 2 | +50% |
| Alumnos formados | 47 | 38 | +24% |
| Propuestas enviadas | 8 | 6 | +33% |
| Tasa de cierre | 62% | 58% | +4pp |
```

---

## Ejemplo de prompt para ClaudIA

```
Tengo mis datos de ventas en /finanzas/ventas_2026.csv con columnas:
fecha, cliente, curso, importe, estado (pagado/pendiente/cancelado).

Genera un informe ejecutivo de Q1 2026 que incluya:
1. Facturación total y por mes
2. Top 5 cursos más vendidos
3. Top 5 clientes por volumen
4. Tasa de impagados
5. Comparativa con Q1 2025 si existe el archivo

Guarda el informe en /informes/q1_2026_ejecutivo.md
```
