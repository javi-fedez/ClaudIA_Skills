---
name: dev-git-operations
description: Gestiona el historial de documentos, cambios y trazabilidad con git. Úsalo para versionar documentación y recuperar versiones anteriores.
---

# Skill: Operaciones Git y Control de Versiones

Capacidad de ClaudIA para gestionar el historial de documentos, colaborar en cambios y mantener trazabilidad de toda la documentación de la empresa.

---

## Herramienta disponible

| Herramienta | Función |
|-------------|---------|
| `Bash` (git) | Commits, ramas, historial, diffs |

---

## Casos de uso en una empresa de formación

### Versionado de materiales de curso

```bash
# Cada vez que se actualiza un temario o material
git add cursos/liderazgo/temario-v2.md
git commit -m "Actualiza módulo 3 de liderazgo con ejercicios de IA"
```

**Beneficio:** Puedes ver qué cambió en el temario de hace 6 meses y por qué.

### Historial de propuestas comerciales

```bash
# Ver qué versión de propuesta se envió a un cliente
git log --all -- clientes/empresa-x/propuesta.md
git show abc1234:clientes/empresa-x/propuesta.md
```

### Ramas por proyecto o cliente

```bash
# Trabajar en una propuesta sin afectar la principal
git checkout -b propuesta/empresa-x-Q2-2026
# ... editar, revisar, validar ...
git checkout main
git merge propuesta/empresa-x-Q2-2026
```

### Ver qué cambió esta semana

```bash
git log --since="1 week ago" --oneline
git diff HEAD~7 -- docs/
```

### Recuperar un documento borrado por error

```bash
git log --all --full-history -- clientes/empresa-y/contrato.md
git checkout abc1234 -- clientes/empresa-y/contrato.md
```

---

## Flujo recomendado para gestión documental

```
main (producción/enviado)
  └── propuesta/cliente-x        ← en preparación
  └── curso/actualizacion-q2     ← en revisión
  └── marketing/campana-junio    ← en borrador
```

**Regla de oro:** Solo llega a `main` lo que está aprobado y enviado/publicado.

---

## Convenciones de commits para empresa de formación

| Tipo | Prefijo | Ejemplo |
|------|---------|---------|
| Nueva propuesta | `propuesta:` | `propuesta: Empresa X curso liderazgo` |
| Actualización curso | `curso:` | `curso: Añade módulo IA a comunicación efectiva` |
| Nuevo cliente | `cliente:` | `cliente: Alta Empresa Y con contrato firmado` |
| Marketing | `marketing:` | `marketing: Newsletter junio sobre formación IA` |
| Finanzas | `finanzas:` | `finanzas: Factura 2026-034 Empresa Z` |

---

## Ejemplo de prompt para ClaudIA

```
Revisa todos los cambios en la carpeta /cursos/ del último mes.
Dime qué materiales se han actualizado, cuáles no se tocan desde
hace más de 6 meses (posible contenido obsoleto) y genera un
informe de "estado de los materiales" en /informes/materiales_estado.md
```
