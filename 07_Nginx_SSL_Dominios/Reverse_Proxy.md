# Nginx, SSL y Dominios

## Cómo funciona el flujo completo

```
Usuario escribe: https://mi-api.com/api/items

1. DNS lookup: mi-api.com → IP Elástica de AWS
2. AWS Security Group: permite tráfico en puerto 443 desde internet
3. Nginx (puerto 443): recibe la conexión HTTPS
4. Nginx: termina SSL (descifra con el certificado de Let's Encrypt)
5. Nginx: hace proxy_pass a http://api:8000 (red interna Docker)
6. FastAPI: procesa la request
7. FastAPI → Nginx → Usuario: respuesta cifrada HTTPS
```

```
Dominio DNS (Registro A)
    ↓
IP Elástica de AWS EC2
    ↓
AWS Security Group (80, 443 abiertos)
    ↓
Nginx (contenedor o instalado en host)
    ├── SSL termination (Let's Encrypt)
    ├── Redirect 80 → 443
    └── proxy_pass → servicios internos
         ├── api:8000      (FastAPI)
         └── n8n:5678      (n8n)
```

## Configuración de DNS (Registro tipo A)

En tu proveedor de DNS (Cloudflare, Route53, GoDaddy, etc.):

```
Tipo    Nombre              Valor               TTL
A       api.tu-dominio.com  <IP Elástica AWS>   3600
A       n8n.tu-dominio.com  <IP Elástica AWS>   3600
A       @                   <IP Elástica AWS>   3600  (dominio raíz)
```

**Nota:** Si usas Cloudflare, desactiva el proxy (nube naranja → gris) hasta que el SSL esté funcionando. Luego puedes volver a activarlo.

---

## Plantilla de reverse proxy Nginx

### Para FastAPI

```nginx
# /etc/nginx/conf.d/api.conf

server {
    listen 80;
    server_name api.tu-dominio.com;

    # Redirect HTTP → HTTPS
    location / {
        return 301 https://$host$request_uri;
    }

    # Para Certbot
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }
}

server {
    listen 443 ssl;
    server_name api.tu-dominio.com;

    ssl_certificate /etc/letsencrypt/live/api.tu-dominio.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.tu-dominio.com/privkey.pem;

    # Seguridad SSL
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;

    # Headers de seguridad
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";

    # Proxy a FastAPI
    location / {
        proxy_pass http://api:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeouts para operaciones largas (LLM puede tardar)
        proxy_read_timeout 120s;
        proxy_connect_timeout 10s;
        proxy_send_timeout 120s;
    }
}
```

### Para n8n

```nginx
# /etc/nginx/conf.d/n8n.conf

server {
    listen 80;
    server_name n8n.tu-dominio.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name n8n.tu-dominio.com;

    ssl_certificate /etc/letsencrypt/live/n8n.tu-dominio.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/n8n.tu-dominio.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;

    location / {
        proxy_pass http://n8n:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # n8n necesita WebSockets
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_read_timeout 300s;
    }
}
```

---

## Certbot / Let's Encrypt

### Instalar Certbot (en el host, no en Docker)

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx

# Obtener certificado
sudo certbot --nginx -d api.tu-dominio.com

# Obtener múltiples dominios en un comando
sudo certbot --nginx -d api.tu-dominio.com -d n8n.tu-dominio.com

# Renovación manual
sudo certbot renew

# Renovación automática (ya viene con el paquete)
sudo systemctl status certbot.timer
```

### Certbot con Nginx en Docker

```bash
# Obtener certificado antes de levantar Nginx con SSL
# Primero levanta Nginx solo con HTTP (sin SSL), consigue el cert, luego agrega SSL

# Método standalone (para cuando Nginx no está corriendo)
sudo certbot certonly --standalone -d api.tu-dominio.com

# Renovar
sudo certbot renew --quiet
```

### Cron de renovación automática

```bash
# En el servidor (crontab -e)
0 3 * * * certbot renew --quiet && nginx -s reload
```

---

## Checklist para validar SSL

- [ ] El registro DNS tipo A apunta a la IP correcta
- [ ] `ping api.tu-dominio.com` devuelve la IP de AWS
- [ ] El puerto 80 y 443 están abiertos en el Security Group de AWS
- [ ] Nginx está corriendo: `docker compose ps nginx` o `systemctl status nginx`
- [ ] El certificado existe: `ls /etc/letsencrypt/live/api.tu-dominio.com/`
- [ ] HTTPS responde: `curl -I https://api.tu-dominio.com/health`
- [ ] El certificado es válido: `curl -vI https://api.tu-dominio.com 2>&1 | grep "SSL certificate"`
- [ ] HTTP redirige a HTTPS: `curl -I http://api.tu-dominio.com` debe devolver 301

### Verificar SSL desde la línea de comandos

```bash
# Ver detalles del certificado
echo | openssl s_client -connect api.tu-dominio.com:443 2>/dev/null | openssl x509 -noout -dates

# Ver cuándo expira
certbot certificates

# Test de configuración Nginx
nginx -t
# o
docker exec mi_nginx nginx -t
```

---

## Comandos Nginx útiles

```bash
# Recargar configuración sin downtime
docker exec mi_nginx nginx -s reload

# Verificar configuración
docker exec mi_nginx nginx -t

# Ver logs de acceso
docker logs mi_nginx

# Ver errores de Nginx
docker exec mi_nginx tail -f /var/log/nginx/error.log
```

---

## Ver también

**Hub:** [[DevOps]] — Hub central de DevOps
- [[Dominios_Activos]] — Registro de todos los dominios configurados
- [[Docker_Compose_Base]] — Cómo Nginx se integra en el stack Docker
- [[Puertos_y_Firewall]] — Qué puertos deben estar abiertos en AWS para que Nginx funcione
- [[Errores_Comunes]] — Errores frecuentes de Nginx y Certbot con soluciones
