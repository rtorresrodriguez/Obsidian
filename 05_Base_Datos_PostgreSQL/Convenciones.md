# PostgreSQL: Convenciones y Buenas Prácticas

## Convenciones de nombres

| Elemento | Convención | Ejemplo |
|----------|-----------|---------|
| Tablas | `snake_case` plural | `user_profiles`, `order_items` |
| Columnas | `snake_case` singular | `first_name`, `created_at` |
| Clave primaria | `id` (integer o UUID) | `id SERIAL PRIMARY KEY` |
| Claves foráneas | `<tabla_singular>_id` | `user_id`, `order_id` |
| Índices | `idx_<tabla>_<columna>` | `idx_users_email` |
| Restricciones | `uq_<tabla>_<columna>` | `uq_users_email` |
| Enums | `snake_case` con tipo claro | `status` con valores `active`, `inactive` |

## Columnas estándar en toda tabla

Toda tabla debe tener estas columnas por defecto:

```sql
id         SERIAL PRIMARY KEY,           -- o UUID si necesitas global
created_at TIMESTAMP DEFAULT NOW(),
updated_at TIMESTAMP DEFAULT NOW(),
is_deleted BOOLEAN DEFAULT FALSE         -- soft delete, opcional
```

## Cómo documentar tablas

Usa este formato en `05_Base_Datos_PostgreSQL/Tablas/<nombre_tabla>.md`:

```markdown
# Tabla: users

## Propósito
Almacena usuarios registrados en el sistema.

## Columnas

| Columna      | Tipo         | Nullable | Default   | Descripción                  |
|--------------|--------------|----------|-----------|------------------------------|
| id           | SERIAL       | NO       | -         | Clave primaria               |
| email        | VARCHAR(255) | NO       | -         | Email único del usuario       |
| name         | VARCHAR(100) | NO       | -         | Nombre completo              |
| is_active    | BOOLEAN      | NO       | TRUE      | Estado del usuario            |
| created_at   | TIMESTAMP    | NO       | NOW()     | Fecha de creación            |
| updated_at   | TIMESTAMP    | YES      | -         | Última actualización         |

## Índices
- `idx_users_email` — en columna `email` (búsquedas frecuentes)

## Relaciones
- `orders.user_id` → `users.id` (un usuario tiene muchos órdenes)

## Notas
- El email es único y se usa para autenticación
- No borrar registros, usar `is_active = FALSE`
```

## Cómo documentar relaciones

```markdown
## Relaciones del sistema

| Tabla origen | FK columna  | Tabla destino | Tipo        |
|-------------|-------------|--------------|-------------|
| orders      | user_id     | users        | Muchos a uno |
| order_items | order_id    | orders       | Muchos a uno |
| order_items | product_id  | products     | Muchos a uno |
```

## Buenas prácticas de conexión

```python
# Siempre usa connection pooling
DATABASE_URL = "postgresql://user:password@host:5432/dbname"

# SQLAlchemy con pool configurado
engine = create_engine(
    DATABASE_URL,
    pool_size=5,           # Conexiones en el pool
    max_overflow=10,       # Conexiones adicionales bajo carga
    pool_pre_ping=True,    # Verifica conexiones antes de usarlas
    pool_recycle=3600,     # Recicla conexiones cada hora
)
```

```bash
# .env
DATABASE_URL=postgresql://app_user:CONTRASEÑA_SEGURA@localhost:5432/mi_base_de_datos
```

## Backups

### Backup manual

```bash
# Backup completo
pg_dump -U postgres -h localhost mi_base_de_datos > backup_$(date +%Y%m%d_%H%M%S).sql

# Backup comprimido
pg_dump -U postgres -h localhost mi_base_de_datos | gzip > backup_$(date +%Y%m%d).sql.gz

# Restaurar
psql -U postgres -h localhost mi_base_de_datos < backup.sql
```

### Backup automático con cron (en el servidor)

```bash
# Agregar a crontab: crontab -e
0 2 * * * pg_dump -U postgres mi_base_de_datos | gzip > /backups/db_$(date +\%Y\%m\%d).sql.gz
```

### Backup desde Docker

```bash
docker exec -t nombre_contenedor_postgres pg_dump -U postgres mi_base_de_datos > backup.sql
```

## Seguridad del puerto 5432

**El puerto 5432 NUNCA debe estar expuesto a internet.**

```bash
# Verificar que no está expuesto (desde el servidor)
sudo ufw status
# 5432 debe aparecer como DENY o no aparecer

# En AWS, verificar Security Groups:
# El puerto 5432 solo debe tener acceso desde la IP del servidor (localhost)
# o desde IPs específicas de tu VPC, NUNCA desde 0.0.0.0/0
```

```ini
# /etc/postgresql/14/main/postgresql.conf
# Escuchar solo en localhost (por defecto)
listen_addresses = 'localhost'

# Si necesitas acceso desde Docker, también en la red de Docker
listen_addresses = 'localhost,172.17.0.1'
```

```ini
# /etc/postgresql/14/main/pg_hba.conf
# Solo conexiones locales y desde contenedores Docker
local   all             all                                     trust
host    all             all             127.0.0.1/32            md5
host    all             all             172.16.0.0/12           md5
```

## Migraciones con Alembic

```bash
# Inicializar Alembic
alembic init alembic

# Crear migración
alembic revision --autogenerate -m "crear tabla users"

# Aplicar migraciones
alembic upgrade head

# Revertir última migración
alembic downgrade -1

# Ver estado
alembic current
alembic history
```

---

## Ver también

**Hub:** [[Backend]] — Hub central del backend (FastAPI + PostgreSQL + APIs)
- [[Tablas/README|Tablas documentadas del sistema]] — Índice de tablas con sus esquemas
- [[Plantilla_Tabla_PostgreSQL]] — Plantilla para documentar una tabla nueva
- [[Estructura_Proyecto]] — Cómo los modelos SQLAlchemy conectan con FastAPI
- [[Docker_Compose_Base]] — Configuración de PostgreSQL en Docker
- [[Puertos_y_Firewall]] — Por qué el puerto 5432 nunca debe estar expuesto
- [[Checklist_Seguridad]] — Seguridad de credenciales de base de datos
