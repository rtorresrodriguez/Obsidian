# Tabla: [nombre_tabla]

> Base de datos: [nombre_base_de_datos]
> Fecha de creación: YYYY-MM-DD
> Última modificación: YYYY-MM-DD

## Propósito

Para qué sirve esta tabla. Qué datos almacena y quién la usa.

## Columnas

| Columna | Tipo | Nullable | Default | Descripción |
|---------|------|----------|---------|-------------|
| id | SERIAL | NO | - | Clave primaria |
| created_at | TIMESTAMP | NO | NOW() | Fecha de creación |
| updated_at | TIMESTAMP | YES | - | Última actualización |

## SQL de creación

```sql
CREATE TABLE nombre_tabla (
    id SERIAL PRIMARY KEY,
    -- columnas aquí
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP
);
```

## Índices

```sql
-- Índices para mejorar performance de consultas frecuentes
CREATE INDEX idx_nombre_tabla_columna ON nombre_tabla(columna);
```

## Restricciones

```sql
-- Uniqueness, foreign keys, checks
ALTER TABLE nombre_tabla ADD CONSTRAINT uq_nombre_tabla_columna UNIQUE (columna);
```

## Relaciones

| Esta tabla | Columna FK | Tabla relacionada | Tipo de relación |
|-----------|-----------|------------------|-----------------|
| nombre_tabla | user_id | users | Muchos a uno |

## Consultas frecuentes

```sql
-- Obtener todos los registros activos
SELECT * FROM nombre_tabla WHERE is_active = TRUE;

-- Buscar por columna específica
SELECT * FROM nombre_tabla WHERE columna = 'valor';
```

## Notas

- Reglas de negocio relevantes para esta tabla
- Cuándo se insertan registros, cuándo se actualizan
- Si se usa soft delete (is_deleted) o hard delete
- Volumen esperado de datos

---

*Plantilla — guardar el documento completado en `05_Base_Datos_PostgreSQL/Tablas/`*
**Ver también:** [[Convenciones]] · [[Backend]] · [[Tablas/README|Tablas documentadas]]
