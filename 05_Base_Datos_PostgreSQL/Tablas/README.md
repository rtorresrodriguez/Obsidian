# Tablas de PostgreSQL

Documenta cada tabla del sistema en un archivo separado usando la plantilla de `99_Plantillas/Plantilla_Tabla_PostgreSQL.md`.

## Tablas del sistema

| Tabla | Base de datos | Propósito | Archivo |
|-------|--------------|----------|--------|
| (agregar cuando se creen) | | | |

## Convención de nombres de archivos

Usa el nombre exacto de la tabla como nombre de archivo:

```
Tablas/
├── users.md
├── conversations.md
├── messages.md
├── documents.md
└── knowledge_chunks.md
```

---

## Ver también

**Hub:** [[Backend]] — Hub central del backend (FastAPI + PostgreSQL + APIs)
- [[Convenciones]] — Convenciones de nombres, tipos, relaciones, backups, Alembic
- [[Plantilla_Tabla_PostgreSQL]] — Plantilla para documentar una tabla nueva
- [[Estructura_Proyecto]] — Cómo los modelos SQLAlchemy mapean estas tablas
