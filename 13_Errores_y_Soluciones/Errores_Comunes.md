# Errores Comunes y Soluciones

Usa la plantilla de `99_Plantillas/Plantilla_Error.md` para documentar nuevos errores.

---

## DNS no resuelve

**Síntoma:** `curl: (6) Could not resolve host: mi-api.com` o el dominio no carga en el navegador.

**Causas comunes y soluciones:**

```bash
# 1. Verificar que el DNS se propagó
dig mi-api.com A
nslookup mi-api.com
# Debe mostrar la IP de AWS

# 2. Si no se propagó, esperar (puede tardar hasta 48h)
# Si es urgente, prueba con la IP directa:
curl http://IP_ELASTICA_AWS/health

# 3. Verificar que el registro A apunta a la IP correcta
# Entrar al panel de DNS (Cloudflare, Route53, etc.)
# Tipo A → nombre → IP Elástica de AWS

# 4. Verificar que la IP Elástica sigue asignada a la instancia
# En AWS Console → EC2 → Elastic IPs
```

**Estado cuando está resuelto:** `dig mi-api.com A` muestra la IP de AWS y `curl https://mi-api.com/health` devuelve `{"status": "ok"}`.

---

## Nginx falla al arrancar

**Síntoma:** `docker compose up` falla con error de Nginx, o Nginx no responde.

```bash
# 1. Ver el error específico
docker logs mi_nginx
docker exec mi_nginx nginx -t

# Error más común: sintaxis en el archivo de configuración
# Error: "nginx: [emerg] unexpected "}" in /etc/nginx/conf.d/api.conf:25"
# → Revisar el archivo de configuración, hay un error de sintaxis

# 2. Si el certificado SSL no existe todavía
# Nginx no puede arrancar si el certificado está en el config pero no existe
# Solución: primero arranca Nginx sin SSL, genera el cert, luego agrega SSL

# Nginx sin SSL (temporal para obtener cert):
server {
    listen 80;
    server_name api.tu-dominio.com;
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }
    location / {
        return 200 "ok";
    }
}

# 3. Puerto 80 o 443 ya está en uso
sudo lsof -i :80
sudo lsof -i :443
# Si otro proceso usa el puerto, detenerlo

# 4. Recargar Nginx sin reiniciar
docker exec mi_nginx nginx -s reload
```

---

## Docker no levanta (contenedor no arranca)

**Síntoma:** `docker compose ps` muestra el contenedor como "Exit" o "Restarting".

```bash
# 1. Ver logs del contenedor
docker compose logs api
docker logs mi_api --tail=50

# Error común 1: variable de entorno faltante
# Error: "KeyError: 'DATABASE_URL'"
# Solución: revisar que .env tiene todas las variables y está en el mismo dir que docker-compose.yml

# Error común 2: puerto ya en uso
# Error: "address already in use"
sudo lsof -i :8000
# Matar el proceso que usa el puerto o cambiar el puerto

# Error común 3: imagen no construida
docker compose build api
docker compose up -d api

# Error común 4: error en el código de la aplicación
# Ver el traceback en los logs
docker compose logs api

# 2. Entrar al contenedor para debug
docker run -it --entrypoint bash mi_api_image

# 3. Verificar que los servicios que necesita están arriba
docker compose ps postgres  # ¿PostgreSQL está healthy?
```

---

## PostgreSQL no conecta

**Síntoma:** `sqlalchemy.exc.OperationalError: could not connect to server`

```bash
# 1. Verificar que PostgreSQL está corriendo
docker compose ps postgres
# Debe mostrar "healthy"

# 2. Verificar credenciales en .env
cat .env | grep POSTGRES
# DATABASE_URL debe tener usuario, contraseña, host, puerto y nombre de DB correctos

# 3. Verificar que el host en DATABASE_URL es correcto
# Si FastAPI está en Docker y PostgreSQL también:
DATABASE_URL=postgresql://user:pass@postgres:5432/db  # ← nombre del servicio Docker
# Si PostgreSQL está en el host (fuera de Docker):
DATABASE_URL=postgresql://user:pass@host.docker.internal:5432/db  # Mac/Windows
DATABASE_URL=postgresql://user:pass@172.17.0.1:5432/db            # Linux

# 4. Probar conexión desde dentro del contenedor FastAPI
docker exec -it mi_api bash
python -c "import psycopg2; conn = psycopg2.connect('postgresql://user:pass@postgres:5432/db'); print('Conectado')"

# 5. Conectar a PostgreSQL directamente
docker exec -it mi_postgres psql -U app_user -d mi_base_de_datos
```

---

## Certbot falla al generar certificado

**Síntoma:** `certbot: error: The requested nginx plugin does not appear to be installed` o `Challenge failed`.

```bash
# Error 1: El dominio no resuelve aún
# Verificar primero que el DNS apunta a la IP del servidor
dig api.tu-dominio.com A

# Error 2: El puerto 80 no está accesible desde internet
# Verificar Security Group de AWS: puerto 80 abierto a 0.0.0.0/0

# Error 3: Nginx no está sirviendo /.well-known/acme-challenge/
# Asegurarse de que Nginx tiene esta configuración antes de correr certbot:
location /.well-known/acme-challenge/ {
    root /var/www/certbot;
}

# Solución alternativa: método standalone (para cuando Nginx no está corriendo)
sudo certbot certonly --standalone -d api.tu-dominio.com

# Error 4: Rate limit de Let's Encrypt (máximo 5 certificados por dominio por semana)
# Usar --staging para pruebas:
sudo certbot --nginx --staging -d api.tu-dominio.com
```

---

## Git push rechazado

**Síntoma:** `! [rejected] main -> main (fetch first)` o `Updates were rejected`.

```bash
# Caso 1: El remote tiene cambios que no tienes local
git pull origin main
# Resolver conflictos si los hay
git push origin main

# Caso 2: Force push bloqueado (¡NO usar force push en main!)
# Investigar por qué el historial divergió
git log --oneline -10
git log origin/main --oneline -10

# Caso 3: Branch protegida
# Crear un PR en lugar de push directo
git checkout -b fix/mi-cambio
git push origin fix/mi-cambio
# Luego crear PR en GitHub/GitLab
```

---

## SSH permission denied

**Síntoma:** `ssh: Permission denied (publickey)` o `Warning: Unprotected private key file!`

```bash
# Error 1: Permisos incorrectos de la clave
chmod 400 ~/.ssh/mi_clave.pem
# Luego intentar de nuevo

# Error 2: Clave incorrecta
ssh -i ~/.ssh/mi_clave.pem -v ubuntu@IP_SERVIDOR
# El flag -v muestra el detalle del error de autenticación

# Error 3: Usuario incorrecto
# AWS Ubuntu → usuario: ubuntu
# AWS Amazon Linux → usuario: ec2-user
# AWS Debian → usuario: admin
ssh -i ~/.ssh/mi_clave.pem ubuntu@IP_SERVIDOR

# Error 4: IP incorrecta o instancia detenida
# Verificar en AWS Console que la instancia está "running"
# Verificar que la IP Elástica sigue asignada

# Error 5: Firewall bloqueando puerto 22
# Verificar Security Group de AWS: puerto 22 abierto a tu IP
```

---

## Ver también

**Hub:** [[DevOps]] — Hub central de DevOps (donde viven la mayoría de estos errores)
- [[Plantilla_Error]] — Plantilla para documentar un error nuevo que encuentres
- [[Reverse_Proxy]] — Configuración correcta de Nginx para evitar errores de proxy
- [[Docker_Compose_Base]] — Configuración correcta de Docker para evitar errores de contenedor
- [[Puertos_y_Firewall]] — Configuración correcta de puertos para evitar problemas de acceso
- [[Convenciones]] — Configuración correcta de PostgreSQL
