# Checklist Pre-Producción

**Usar antes de cualquier despliegue o cambio en producción.**

## Preparación

- [ ] El cambio está documentado (sé exactamente qué voy a hacer)
- [ ] El código está en el repositorio git (no solo en local)
- [ ] Hice backup de la base de datos si el cambio incluye migraciones
- [ ] Tengo un plan de rollback documentado
- [ ] Tengo otra sesión SSH abierta como respaldo

## Seguridad

- [ ] No hay credenciales hardcodeadas en el código nuevo
- [ ] El `.env` de producción tiene las variables correctas
- [ ] Los nuevos puertos (si los hay) están correctamente restringidos
- [ ] El `.gitignore` sigue protegiendo archivos sensibles

## Antes de ejecutar

```bash
# En el servidor, verificar estado actual
docker compose ps
docker compose logs --tail=20
df -h  # Espacio en disco
free -h  # Memoria disponible
```

## Durante el despliegue

```bash
# Pull del código nuevo
git pull origin main

# Reconstruir solo el servicio que cambió
docker compose build api
docker compose up -d api

# Verificar que arrancó correctamente
docker compose logs -f api  # Ctrl+C cuando veas que arrancó
docker compose ps
```

## Validación post-despliegue

- [ ] `docker compose ps` → todos los servicios en "Up" / "healthy"
- [ ] `curl https://api.tu-dominio.com/health` → `{"status": "ok"}`
- [ ] Los logs no muestran errores: `docker compose logs --tail=20 api`
- [ ] Si hubo migración de base de datos: verificar que las tablas existen
- [ ] Si cambió Nginx: `curl -I https://tu-dominio.com` muestra 200
- [ ] Si cambió algo visible al usuario: probar manualmente el flujo principal

## Si algo falla

```bash
# Ver el error
docker compose logs api

# Rollback: volver al commit anterior
git log --oneline -5  # Ver commits
git checkout COMMIT_HASH_ANTERIOR -- app/  # Revertir solo los archivos de app

# Reconstruir con el código anterior
docker compose build api
docker compose up -d api

# Si la base de datos tiene datos incompatibles:
# Restaurar backup (DOCUMENTAR el backup antes de hacer cambios)
```

## Documentar el despliegue

- [ ] Agregar entrada en [[Registro_Cambios_Docker]] si aplica
- [ ] Actualizar estado del proyecto en `14_Proyectos/`
- [ ] Documentar cualquier error encontrado en [[Errores_Comunes]] usando [[Plantilla_Error]]

---

## Ver también

**Hubs:** [[Arquitectura]] — Principios que guían el despliegue · [[DevOps]] — Infraestructura y herramientas
- [[Checklist_Nueva_App]] — Checklist para antes de crear la app (no solo antes de desplegar)
- [[Checklist_Seguridad]] — Revisión de seguridad antes del push
- [[Docker_Compose_Base]] — Comandos de Docker para validar el despliegue
- [[Errores_Comunes]] — Si algo falla durante el despliegue
- [[Registro_Cambios_Docker]] — Dónde documentar el cambio después del despliegue
