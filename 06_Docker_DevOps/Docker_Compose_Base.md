# Docker y DevOps

## Qué debe ir en Docker

**Servicios de larga duración:**
- FastAPI (API backend)
- PostgreSQL
- n8n
- Nginx
- Redis (si se usa)
- Workers de Celery (si se usa)

**Regla:** Si el servicio debe correr continuamente y reiniciarse automáticamente, va en Docker.

## Qué no necesariamente va en Docker

- Scripts de migración de datos de una sola vez
- Herramientas CLI de desarrollo (puedes instalarlas en tu máquina local)
- Tests unitarios (corren en el CI o local sin contenedor)
- Certbot (puede correr en el host para renovar SSL)

---

## docker-compose.yml base

```yaml
version: "3.9"

services:
  # Backend FastAPI
  api:
    build: .
    container_name: mi_api
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "8000:8000"
    depends_on:
      postgres:
        condition: service_healthy
    volumes:
      - ./app:/app/app   # Solo en desarrollo; quitar en producción
    networks:
      - app_network

  # Base de datos
  postgres:
    image: postgres:15-alpine
    container_name: mi_postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app_network
    # IMPORTANTE: No exponer el puerto 5432 en producción
    # ports:
    #   - "5432:5432"   ← Solo para desarrollo local

  # n8n
  n8n:
    image: n8nio/n8n:latest
    container_name: mi_n8n
    restart: unless-stopped
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=${N8N_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_PASSWORD}
      - N8N_HOST=${N8N_HOST}
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://${N8N_HOST}/
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=${N8N_DB}
      - DB_POSTGRESDB_USER=${POSTGRES_USER}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
    ports:
      - "5678:5678"
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - app_network

  # Nginx (reverse proxy)
  nginx:
    image: nginx:alpine
    container_name: mi_nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - /etc/letsencrypt:/etc/letsencrypt:ro
      - /var/www/certbot:/var/www/certbot
    depends_on:
      - api
      - n8n
    networks:
      - app_network

volumes:
  postgres_data:
    driver: local
  n8n_data:
    driver: local

networks:
  app_network:
    driver: bridge
```

## .env para docker-compose

```bash
# .env (NO subir a git)
POSTGRES_USER=app_user
POSTGRES_PASSWORD=CONTRASEÑA_MUY_SEGURA_AQUI
POSTGRES_DB=mi_base_de_datos

N8N_USER=admin
N8N_PASSWORD=CONTRASEÑA_N8N_AQUI
N8N_HOST=n8n.tu-dominio.com
N8N_DB=n8n_db

DATABASE_URL=postgresql://app_user:CONTRASEÑA_MUY_SEGURA_AQUI@postgres:5432/mi_base_de_datos
SECRET_KEY=CLAVE_SECRETA_LARGA_ALEATORIA
ANTHROPIC_API_KEY=sk-ant-...
```

## Volúmenes persistentes

Los volúmenes son críticos. Son los únicos datos que sobreviven a `docker compose down`.

```bash
# Ver volúmenes existentes
docker volume ls

# Inspeccionar un volumen
docker volume inspect postgres_data

# NUNCA borrar un volumen de producción sin backup previo
# docker volume rm postgres_data  ← PELIGROSO

# Backup de volumen
docker run --rm -v postgres_data:/data -v $(pwd):/backup alpine \
  tar czf /backup/postgres_backup_$(date +%Y%m%d).tar.gz /data
```

## Comandos útiles

```bash
# Arrancar todos los servicios
docker compose up -d

# Arrancar solo un servicio
docker compose up -d api

# Ver estado
docker compose ps

# Ver logs en tiempo real
docker compose logs -f
docker compose logs -f api

# Reiniciar un servicio sin downtime
docker compose restart api

# Reconstruir imagen y reiniciar
docker compose up -d --build api

# Parar sin borrar datos
docker compose stop

# Parar y eliminar contenedores (datos en volúmenes se mantienen)
docker compose down

# Parar, eliminar contenedores Y volúmenes (PELIGROSO en producción)
docker compose down -v
```

## Cómo validar contenedores

```bash
# Estado de todos los contenedores
docker compose ps

# Ver uso de recursos
docker stats

# Inspeccionar un contenedor
docker inspect mi_api

# Entrar a un contenedor para debug
docker exec -it mi_api bash
docker exec -it mi_postgres psql -U app_user mi_base_de_datos

# Verificar que la API responde
curl http://localhost:8000/health
```

## Cómo revisar logs

```bash
# Logs del servicio (últimas 100 líneas)
docker compose logs --tail=100 api

# Logs en tiempo real
docker compose logs -f api

# Logs con timestamp
docker compose logs -t api

# Logs de múltiples servicios
docker compose logs -f api postgres

# Logs del sistema Docker
journalctl -u docker.service
```

## Dockerfile base para FastAPI

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Instalar dependencias primero (aprovecha cache de Docker)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar código
COPY . .

# Usuario no-root por seguridad
RUN useradd -m appuser
USER appuser

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## Ver también

**Hub:** [[DevOps]] — Hub central de DevOps (Docker, Nginx, despliegue, seguridad)
- [[Reverse_Proxy]] — Configuración de Nginx para exponer los servicios Docker
- [[Puertos_y_Firewall]] — Qué puertos deben estar cerrados en el docker-compose
- [[Convenciones]] — Configuración de PostgreSQL en producción
- [[Registro_Cambios_Docker]] — Registrar cambios al docker-compose de producción
- [[Checklist_Pre_Produccion]] — Verificaciones antes de hacer cambios en producción
- [[Errores_Comunes]] — Errores frecuentes de Docker y cómo resolverlos
