# Hub: DevOps

> Este archivo es el hub central de DevOps. Desde aquí navega a Docker, Nginx, SSL, seguridad y despliegue.
> Hubs del sistema: [[Arquitectura]] · [[Backend]] · [[IA]] · [[MAPA_MENTAL]]

---

## ¿Qué cubre DevOps en este sistema?

```
INTERNET
   │ 80/443
   ▼
NGINX ──────────────────────────── Termina SSL
   │ proxy_pass (red Docker interna)
   ├──────────────────────────────► FastAPI :8000
   └──────────────────────────────► n8n :5678

DOCKER COMPOSE ─────────────────── Orquesta todo
   ├── api (FastAPI)
   ├── postgres
   ├── n8n
   └── nginx

AWS EC2 ────────────────────────── Infraestructura
   ├── Security Group (Firewall nivel instancia)
   └── IP Elástica (IP pública estable)

UFW ────────────────────────────── Firewall nivel OS
   └── Solo 22, 80, 443 abiertos

CERTBOT ────────────────────────── SSL automático
   └── Let's Encrypt + renovación automática
```

---

## Reglas de oro de DevOps

| Regla | Nota |
|-------|------|
| Solo Nginx expone puertos 80/443 | [[Puertos_y_Firewall]] |
| PostgreSQL nunca expuesto (:5432 cerrado) | [[Puertos_y_Firewall]] |
| Volúmenes Docker = datos de producción, nunca borrar | [[Docker_Compose_Base]] |
| Todo cambio en docker-compose se registra | [[Registro_Cambios_Docker]] |
| Todo dominio nuevo se documenta | [[Dominios_Activos]] |
| Checklist antes de cualquier cambio en producción | [[Checklist_Pre_Produccion]] |

---

## Sub-notas de DevOps

### Docker
- [[Docker_Compose_Base]] — docker-compose.yml completo con todos los servicios, Dockerfile, comandos esenciales
- [[Registro_Cambios_Docker]] — Historial de cambios en Docker en producción

### Nginx y dominios
- [[Reverse_Proxy]] — Configuración completa de Nginx, Certbot, Let's Encrypt, checklist SSL
- [[Dominios_Activos]] — Registro de dominios activos y cómo agregar uno nuevo

### Seguridad
- [[Checklist_Seguridad]] — Variables de entorno, .gitignore, qué nunca subir a git
- [[Puertos_y_Firewall]] — Mapa de puertos, UFW, AWS Security Group

### Despliegue
- [[Checklist_Pre_Produccion]] — Verificaciones antes de cualquier cambio en producción
- [[Mapa_del_Sistema]] — Diagrama completo de infraestructura AWS

### Errores conocidos
- [[Errores_Comunes]] — Docker no levanta, Nginx falla, Certbot falla, SSH bloqueado

---

## Flujo de despliegue estándar

```
1. Desarrollar y testear localmente
2. git push al repositorio
3. SSH al servidor AWS
4. git pull origin main
5. docker compose build <servicio>   ← solo el servicio que cambió
6. docker compose up -d <servicio>
7. Verificar: docker compose ps
8. Verificar: curl https://api.dominio.com/health
9. Revisar logs: docker compose logs --tail=20 <servicio>
10. Documentar en [[Registro_Cambios_Docker]] si aplica
```

> Checklist completo: [[Checklist_Pre_Produccion]]

---

## Comandos de emergencia (quick reference)

```bash
# Ver estado de todos los servicios
docker compose ps

# Ver logs en tiempo real de un servicio
docker compose logs -f api

# Reiniciar un servicio sin bajar los demás
docker compose restart api

# Recargar Nginx sin downtime
docker exec mi_nginx nginx -s reload

# Verificar que PostgreSQL acepta conexiones
docker exec mi_postgres pg_isready -U app_user

# Ver uso de disco (volúmenes de datos)
df -h && docker system df
```

---

## Relación con otros hubs

```
DevOps
    ├── expone     →  [[Backend]]      (Nginx hace proxy a FastAPI y n8n)
    ├── aloja      →  [[IA]]           (agentes corren en la misma infra)
    └── sigue      →  [[Arquitectura]] (principios de seguridad y separación)
```

---

## Alertas de seguridad frecuentes

| Situación de riesgo | Dónde verificar |
|--------------------|----------------|
| Puerto 5432 expuesto | [[Puertos_y_Firewall]] |
| Credencial en código | [[Checklist_Seguridad]] |
| Volumen sin backup | [[Convenciones]] (sección backups) |
| SSL vencido o por vencer | [[Reverse_Proxy]] (sección Certbot) |
| Dominio mal configurado | [[Dominios_Activos]] |
