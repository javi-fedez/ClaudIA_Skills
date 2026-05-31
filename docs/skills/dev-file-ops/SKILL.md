---
name: dev-file-ops
description: Lee, escribe, edita y busca archivos en el sistema local. Úsalo para cualquier manipulación de archivos.
---

# Skill: Operaciones de Archivos

Capacidad nativa de ClaudIA para leer, escribir, editar y buscar archivos en el sistema local.

---

## Herramientas disponibles

| Herramienta | Función | Cuándo usarla |
|-------------|---------|---------------|
| `Read` | Leer contenido de un archivo | Analizar documentos, facturas, contratos |
| `Write` | Crear un archivo nuevo | Generar propuestas, informes, plantillas |
| `Edit` | Modificar parte de un archivo | Actualizar datos, corregir contenido |
| `Glob` | Buscar archivos por patrón | Encontrar todos los .pdf, .xlsx, etc. |
| `Grep` | Buscar texto dentro de archivos | Localizar cláusulas, nombres, importes |

---

## Patrones de uso

### Generar propuesta comercial

```
Tarea: "Crea una propuesta para el cliente Empresa X"

Pasos:
1. Read(plantilla de propuesta existente)
2. Write(nueva propuesta personalizada con datos del cliente)
```

### Analizar contrato o documento

```
Tarea: "Revisa este contrato y dime los puntos clave"

Pasos:
1. Read(contrato.pdf o contrato.md)
2. Extraer: partes, objeto, precio, duración, penalizaciones
3. Resumen ejecutivo
```

### Buscar documentos de un cliente

```
Tarea: "Encuentra todos los documentos de Empresa X"

Pasos:
1. Glob("**/*EmpresaX*") o Glob("clientes/empresa-x/**")
2. Read de los más relevantes
3. Resumen de la relación comercial
```

### Actualizar base de datos de clientes

```
Tarea: "Actualiza el CRM con los datos del nuevo cliente"

Pasos:
1. Read(clientes/database.csv o .json)
2. Edit para añadir nueva fila/entrada
```

### Buscar cláusula en múltiples contratos

```
Tarea: "¿Cuál es el periodo de preaviso en todos mis contratos?"

Pasos:
1. Glob("contratos/**/*.md")
2. Grep("preaviso|notice period") en todos
3. Tabla comparativa
```

---

## Estructura recomendada de carpetas para una empresa de formación

```
negocio/
├── clientes/
│   ├── empresa-a/
│   │   ├── contrato.md
│   │   ├── propuesta-2026.md
│   │   └── facturas/
├── cursos/
│   ├── liderazgo/
│   │   ├── temario.md
│   │   └── materiales/
├── marketing/
│   ├── emails/
│   └── redes-sociales/
├── finanzas/
│   ├── facturas/
│   └── presupuestos/
└── plantillas/
    ├── propuesta.md
    ├── contrato.md
    └── factura.md
```

---

## Ejemplo de prompt para ClaudIA

```
Tengo una carpeta en /negocio/propuestas/. Lee todas las propuestas del
último trimestre, identifica cuál fue el precio medio ofertado y qué
servicios se incluyeron con más frecuencia. Dame un resumen en tabla.
```
