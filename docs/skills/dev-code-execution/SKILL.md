---
name: dev-code-execution
description: Ejecuta comandos de shell, scripts Python y automatiza tareas del sistema operativo. Úsalo cuando haya que ejecutar código o automatizar el sistema.
---

# Skill: Ejecución de Código y Automatización

Capacidad nativa de ClaudIA para ejecutar comandos de shell, scripts Python, y automatizar tareas del sistema operativo.

---

## Herramienta disponible

| Herramienta | Función |
|-------------|---------|
| `Bash` | Ejecutar cualquier comando de terminal: Python, shell scripts, herramientas CLI |

---

## Casos de uso para una empresa de formación

### Procesar datos de alumnos (Excel/CSV)

```bash
# ClaudIA ejecuta Python para analizar un Excel
python3 - <<'EOF'
import csv
with open('alumnos.csv') as f:
    alumnos = list(csv.DictReader(f))
aprobados = [a for a in alumnos if float(a['nota']) >= 5]
print(f"Total: {len(alumnos)}, Aprobados: {len(aprobados)}")
EOF
```

### Generar PDF desde Markdown

```bash
# Convertir propuesta.md a propuesta.pdf
pandoc propuesta.md -o propuesta.pdf --pdf-engine=wkhtmltopdf
# O con markdown-pdf si está instalado
```

### Calcular facturación mensual

```bash
python3 - <<'EOF'
import csv, datetime
mes_actual = datetime.date.today().strftime('%Y-%m')
total = 0
with open('facturas.csv') as f:
    for row in csv.DictReader(f):
        if row['fecha'].startswith(mes_actual):
            total += float(row['importe'])
print(f"Facturación {mes_actual}: {total:.2f} €")
EOF
```

### Enviar emails por plantilla (con configuración previa)

```bash
python3 enviar_email.py --plantilla confirmacion_matricula.txt --lista alumnos_nuevos.csv
```

### Comprimir y archivar documentos antiguos

```bash
tar -czf archivo_2025.tar.gz facturas/2025/ contratos/2025/
mv archivo_2025.tar.gz historico/
```

### Generar estadísticas de cursos

```bash
python3 - <<'EOF'
# Ejemplo: tasa de finalización por curso
import json
with open('stats.json') as f:
    data = json.load(f)
for curso in data['cursos']:
    tasa = curso['completados'] / curso['matriculados'] * 100
    print(f"{curso['nombre']}: {tasa:.1f}% finalización")
EOF
```

---

## Automatizaciones útiles configurables

| Tarea | Frecuencia | Script |
|-------|-----------|--------|
| Backup de documentos | Diario | `backup.sh` |
| Informe de ventas | Semanal | `informe_semanal.py` |
| Recordatorio a alumnos sin pago | Semanal | `pagos_pendientes.py` |
| Limpieza de archivos temporales | Mensual | `limpieza.sh` |
| Exportar lista de alumnos activos | Bajo demanda | `export_alumnos.py` |

---

## Limitaciones

- Los comandos ejecutan en el entorno local del usuario
- No usar para operaciones destructivas sin confirmar primero (`rm -rf`, drops de DB, etc.)
- Los scripts con credenciales deben leer de `.env`, nunca hardcoded

---

## Ejemplo de prompt para ClaudIA

```
Tengo un CSV con los alumnos de este trimestre en /datos/alumnos_q1.csv
con columnas: nombre, email, curso, nota_final. Calcula la tasa de
aprobados por curso y genera un informe en markdown con gráfico de
texto (barras ASCII). Guárdalo en /informes/q1_resultados.md
```
