# Mapa del sistema

## Vista de alto nivel

```
                         INTERNET
                            │
                    ┌───────▼────────┐
                    │   DNS Público  │
                    │  (Cloudflare / │
                    │   Route53)     │
                    └───────┬────────┘
                            │ Registro A → IP Elástica
                            ▼
                    ┌───────────────┐
                    │   AWS EC2     │
                    │  (Ubuntu 22)  │
                    └───────┬───────┘
                            │
                    ┌───────▼───────┐
                    │    NGINX      │
                    │  :80 / :443   │
                    └───────┬───────┘
                            │ proxy_pass (red Docker interna)
              ┌─────────────┼──────────────┐
              │             │              │
    ┌─────────▼──┐  ┌───────▼────┐  ┌────▼──────┐
    │  FastAPI   │  │    n8n     │  │  Otros    │
    │  :8000     │  │  :5678     │  │  servicios│
    └─────────┬──┘  └────────────┘  └───────────┘
              │
    ┌─────────▼──────────────┐
    │      PostgreSQL        │
    │  :5432 (solo interno)  │
    └────────────────────────┘
```

## Componentes y responsabilidades

| Componente | Responsabilidad | Gestionado con |
|-----------|----------------|---------------|
| DNS | Traducir dominio → IP | Panel del proveedor DNS |
| AWS EC2 | Servidor físico/virtual | AWS Console |
| AWS Security Group | Firewall nivel instancia | AWS Console |
| UFW | Firewall nivel OS | CLI en servidor |
| Nginx | Reverse proxy, SSL, routing | Docker + config files |
| Let's Encrypt | Certificados SSL gratuitos | Certbot |
| FastAPI | Lógica de negocio, API REST | Docker |
| n8n | Orquestación y automatización | Docker |
| PostgreSQL | Persistencia de datos | Docker |
| Docker Compose | Orquestación de contenedores | CLI en servidor |

## Flujo de una request típica

```
1. Usuario → https://api.mi-dominio.com/api/items
2. DNS → resuelve a IP Elástica de AWS
3. AWS Security Group → permite tráfico en :443
4. Nginx → recibe en :443, termina SSL
5. Nginx → proxy_pass http://api:8000/api/items
6. FastAPI → Router → Service → Model
7. Model → SELECT FROM items (PostgreSQL)
8. PostgreSQL → devuelve datos
9. FastAPI → serializa con Pydantic schema
10. Nginx → devuelve respuesta HTTPS al usuario
```

## Puntos únicos de falla y mitigación

| Punto de falla | Impacto | Mitigación |
|---------------|---------|-----------|
| AWS EC2 cae | Todo el sistema | Elastic IP facilita reasignación; snapshot periódico |
| PostgreSQL cae | Sin datos | Restart automático (restart: unless-stopped en Docker) |
| Nginx cae | Sin acceso externo | Restart automático |
| Certificado SSL vence | HTTPS falla | Renovación automática con certbot.timer |
| Docker daemon cae | Todo el sistema | `systemctl enable docker` para inicio automático |

## Monitoreo básico

```bash
# Verificar que todos los servicios están en pie
docker compose ps

# Verificar que la API responde
curl https://api.mi-dominio.com/health

# Verificar uso de disco (crítico para logs y volúmenes)
df -h

# Verificar memoria
free -h

# Verificar logs recientes
docker compose logs --tail=50 --since=1h
```

---

## Ver también

- [[README]] — Estado de servicios y dominios activos
- [[Docker_Compose_Base]] — Configuración detallada de cada servicio
- [[Reverse_Proxy]] — Configuración completa de Nginx y SSL
- [[Puertos_y_Firewall]] — Mapa de puertos y configuración del firewall
- [[Errores_Comunes]] — Qué hacer cuando un componente del sistema falla
