# Dominios activos

## Estado de dominios

| Dominio | IP apunta a | Servicio backend | SSL | Vencimiento SSL | Notas |
|---------|------------|-----------------|-----|----------------|-------|
| (completar) | IP AWS | FastAPI:8000 | Activo | (completar) | - |
| (completar) | IP AWS | n8n:5678 | Activo | (completar) | - |

## Cómo agregar un dominio nuevo

### Paso 1: Configurar DNS

En tu proveedor de DNS, agrega un registro tipo A:
```
Tipo: A
Nombre: subdominio (o @ para el dominio raíz)
Valor: IP_ELASTICA_DE_AWS
TTL: 3600
```

### Paso 2: Crear configuración Nginx

Crea `/etc/nginx/conf.d/nuevo-dominio.conf` siguiendo la plantilla de `07_Nginx_SSL_Dominios/Reverse_Proxy.md`.

Primero sin SSL (para que Certbot pueda validar):

```nginx
server {
    listen 80;
    server_name nuevo-dominio.com;
    
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }
    
    location / {
        proxy_pass http://servicio:puerto;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Paso 3: Obtener certificado SSL

```bash
# Verificar primero que el DNS ya resuelve
dig nuevo-dominio.com A

# Obtener certificado
sudo certbot --nginx -d nuevo-dominio.com

# Si Nginx está en Docker:
sudo certbot certonly --standalone -d nuevo-dominio.com
# (Nginx debe estar detenido para usar standalone)
```

### Paso 4: Actualizar Nginx con SSL

Reemplazar la configuración del paso 2 con la plantilla completa de SSL de `07_Nginx_SSL_Dominios/Reverse_Proxy.md`.

```bash
# Recargar Nginx
docker exec mi_nginx nginx -s reload
# o
sudo systemctl reload nginx
```

### Paso 5: Verificar y documentar

```bash
# Verificar que HTTPS funciona
curl -I https://nuevo-dominio.com

# Ver cuándo vence el certificado
sudo certbot certificates
```

Luego actualizar la tabla de dominios al inicio de este archivo.

---

## Ver también

**Hub:** [[DevOps]] — Hub central de DevOps
- [[Reverse_Proxy]] — Configuración completa de Nginx y Certbot para cada dominio
- [[Puertos_y_Firewall]] — AWS Security Group: puertos 80 y 443 deben estar abiertos
- [[Mapa_del_Sistema]] — Mapa de toda la infraestructura incluyendo dominios y AWS
- [[Errores_Comunes]] — DNS no resuelve, Certbot falla: soluciones paso a paso
