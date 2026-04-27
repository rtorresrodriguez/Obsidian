# Puertos y Firewall

## Mapa de puertos del sistema

| Puerto | Servicio | Acceso desde internet | Acceso interno | Estado |
|--------|---------|----------------------|---------------|--------|
| 22 | SSH | Solo tu IP (whitelist) | SÍ | Abierto con restricción |
| 80 | Nginx HTTP | SÍ (para redirect a 443) | SÍ | Abierto |
| 443 | Nginx HTTPS | SÍ | SÍ | Abierto |
| 5432 | PostgreSQL | **NUNCA** | Solo servicios Docker | Cerrado |
| 5678 | n8n | **NO** (via Nginx) | Sí (via Nginx) | Cerrado externamente |
| 8000 | FastAPI | **NO** (via Nginx) | Sí (via Nginx) | Cerrado externamente |

## Regla de oro

> Solo los puertos 22, 80 y 443 deben estar accesibles desde internet.
> Todo lo demás se accede via Nginx (reverse proxy) o via la red interna de Docker.

## Configuración AWS Security Group

### Reglas de entrada (Inbound rules) recomendadas

| Tipo | Protocolo | Rango de puertos | Origen |
|------|----------|-----------------|-------|
| SSH | TCP | 22 | Tu IP / 0.0.0.0/0 si no tienes IP fija |
| HTTP | TCP | 80 | 0.0.0.0/0, ::/0 |
| HTTPS | TCP | 443 | 0.0.0.0/0, ::/0 |

**No agregar:** 5432, 5678, 8000, ni ningún otro puerto de aplicación.

## Verificar puertos en el servidor

```bash
# Ver puertos escuchando y en qué interfaz
sudo ss -tlnp

# Ejemplo de output seguro:
# LISTEN  0  128  0.0.0.0:80    → Nginx (OK, acceso público)
# LISTEN  0  128  0.0.0.0:443   → Nginx (OK, acceso público)
# LISTEN  0  128  0.0.0.0:22    → SSH (OK, acceso público pero protegido)
# LISTEN  0  128  127.0.0.1:5432 → PostgreSQL (OK, solo localhost)

# Ejemplo problemático:
# LISTEN  0  128  0.0.0.0:5432  → PostgreSQL expuesto! (CERRAR)
# LISTEN  0  128  0.0.0.0:8000  → FastAPI expuesto! (CERRAR)
```

## UFW (Firewall en Ubuntu)

```bash
# Ver estado
sudo ufw status verbose

# Configuración básica
sudo ufw default deny incoming    # Denegar todo por defecto
sudo ufw default allow outgoing   # Permitir salida

# Permitir SSH (antes de activar UFW para no perder acceso)
sudo ufw allow 22/tcp

# Permitir HTTP y HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Activar
sudo ufw enable

# Verificar
sudo ufw status numbered
```

## PostgreSQL: asegurar que solo escucha en localhost

```bash
# Verificar configuración actual
sudo grep "listen_addresses" /etc/postgresql/*/main/postgresql.conf

# Debe mostrar:
listen_addresses = 'localhost'

# Si muestra '*' o la IP pública, cambiarlo:
sudo nano /etc/postgresql/*/main/postgresql.conf
# Cambiar: listen_addresses = 'localhost'

sudo systemctl restart postgresql
```

## En Docker: no exponer puertos innecesarios

```yaml
# docker-compose.yml

# CORRECTO: PostgreSQL sin puerto expuesto al host
postgres:
  image: postgres:15
  # NO hay "ports:" aquí - solo accesible dentro de la red Docker

# CORRECTO: FastAPI sin puerto expuesto al exterior
api:
  image: mi_api
  # ports: - "8000:8000"  ← COMENTADO, Nginx hace el proxy internamente
  networks:
    - app_network

# CORRECTO: Solo Nginx expone puertos
nginx:
  image: nginx:alpine
  ports:
    - "80:80"
    - "443:443"
  networks:
    - app_network
```

---

## Ver también

**Hub:** [[DevOps]] — Hub central de DevOps, seguridad e infraestructura
- [[Checklist_Seguridad]] — Checklist completo de seguridad antes de hacer push
- [[Docker_Compose_Base]] — Cómo configurar los puertos en docker-compose correctamente
- [[Reverse_Proxy]] — Por qué Nginx es el único servicio que debe exponer puertos
- [[Errores_Comunes]] — Errores de SSH y acceso al servidor
