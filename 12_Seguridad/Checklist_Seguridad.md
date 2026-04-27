# Seguridad

## Variables de entorno y archivos .env

### Reglas absolutas

- **Nunca** pongas credenciales, API keys o contraseñas directamente en el código
- **Nunca** subas `.env` a Git
- **Siempre** mantén `.env.example` actualizado con todas las variables (sin valores reales)
- **Siempre** usa valores distintos para desarrollo y producción

### Estructura correcta de .env

```bash
# .env (NO subir a Git — agregar a .gitignore)

# Base de datos
POSTGRES_USER=app_user
POSTGRES_PASSWORD=CONTRASEÑA_MUY_SEGURA_ALEATORIA
POSTGRES_DB=mi_base_de_datos
DATABASE_URL=postgresql://app_user:CONTRASEÑA@localhost:5432/mi_base_de_datos

# Aplicación
SECRET_KEY=CLAVE_SECRETA_DE_AL_MENOS_32_CARACTERES_ALEATORIA
ENVIRONMENT=production

# APIs externas
ANTHROPIC_API_KEY=sk-ant-...

# n8n
N8N_USER=admin
N8N_PASSWORD=CONTRASEÑA_N8N
INTERNAL_API_KEY=CLAVE_INTERNA_PARA_N8N

# Email (si aplica)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=tu@email.com
SMTP_PASSWORD=APP_PASSWORD_DE_GMAIL
```

```bash
# .env.example (SÍ subir a Git)

POSTGRES_USER=app_user
POSTGRES_PASSWORD=CAMBIAR_POR_CONTRASEÑA_SEGURA
POSTGRES_DB=mi_base_de_datos
DATABASE_URL=postgresql://app_user:PASSWORD@localhost:5432/mi_base_de_datos

SECRET_KEY=GENERAR_CON_python_-c_"import_secrets;print(secrets.token_hex(32))"
ENVIRONMENT=development

ANTHROPIC_API_KEY=sk-ant-OBTENER_EN_console.anthropic.com

N8N_USER=admin
N8N_PASSWORD=CAMBIAR_POR_CONTRASEÑA_SEGURA
INTERNAL_API_KEY=GENERAR_ALEATORIAMENTE
```

### Generar valores seguros

```bash
# Generar SECRET_KEY segura
python -c "import secrets; print(secrets.token_hex(32))"

# Generar contraseña segura
python -c "import secrets, string; print(''.join(secrets.choice(string.ascii_letters + string.digits) for _ in range(32)))"

# Usando openssl
openssl rand -hex 32
```

---

## Qué nunca subir a GitHub

Agregar al `.gitignore`:

```gitignore
# Variables de entorno
.env
.env.local
.env.production
*.env

# Claves SSH
*.pem
*.key
id_rsa
id_ed25519

# Certificados SSL (los gestiona certbot)
*.crt
*.cert
fullchain.pem
privkey.pem

# Bases de datos locales
*.sqlite
*.db
*.sql       # Si contienen datos reales

# Logs
*.log
logs/

# Caché y compilados
__pycache__/
*.pyc
.pytest_cache/
.mypy_cache/
node_modules/
.next/
dist/
build/

# IDEs
.vscode/settings.json   # si contiene rutas sensibles
.idea/

# Backups
*.bak
*backup*
```

---

## Puertos y Firewall

### Qué puertos deben estar abiertos en AWS Security Group

| Puerto | Protocolo | Abierto a | Servicio |
|--------|----------|----------|---------|
| 22 | TCP | Solo tu IP | SSH |
| 80 | TCP | 0.0.0.0/0 (internet) | HTTP (redirect a HTTPS) |
| 443 | TCP | 0.0.0.0/0 (internet) | HTTPS |
| 5432 | TCP | CERRADO o solo VPC | PostgreSQL |
| 5678 | TCP | CERRADO (via Nginx) | n8n |
| 8000 | TCP | CERRADO (via Nginx) | FastAPI |

**Regla:** Solo los puertos 80, 443 y 22 deben estar accesibles desde internet.

### Verificar puertos en el servidor

```bash
# Ver puertos escuchando
sudo ss -tlnp

# Ver reglas de firewall (UFW)
sudo ufw status verbose

# Configurar UFW
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw deny 5432/tcp   # PostgreSQL - CERRAR
sudo ufw deny 8000/tcp   # FastAPI interno - CERRAR
sudo ufw enable
```

---

## SSH

```bash
# Conectar al servidor
ssh -i ~/.ssh/mi_clave.pem ubuntu@IP_DEL_SERVIDOR

# Copiar clave al servidor (para agregar colaboradores)
ssh-copy-id -i ~/.ssh/clave_colaborador.pub ubuntu@IP_DEL_SERVIDOR

# Configurar ~/.ssh/config para conexión más fácil
Host mi-servidor
  HostName IP_DEL_SERVIDOR
  User ubuntu
  IdentityFile ~/.ssh/mi_clave.pem
  
# Luego conectar con:
ssh mi-servidor
```

### Hardening SSH

```ini
# /etc/ssh/sshd_config (en el servidor)
PasswordAuthentication no     # Solo claves SSH, no contraseñas
PermitRootLogin no            # No permitir login como root
MaxAuthTries 3                # Máximo 3 intentos
```

---

## Backups de seguridad

### Qué hacer backup y con qué frecuencia

| Datos | Frecuencia | Método |
|-------|-----------|--------|
| PostgreSQL (base de datos) | Diario | pg_dump + cron |
| Archivos de n8n (.n8n/) | Semanal | Backup del volumen Docker |
| Configuración Nginx | Al cambiar | Guardar en git privado |
| .env de producción | Al cambiar | Guardar en gestor de secretos (no en git) |

---

## Checklist de seguridad antes de hacer push a git

- [ ] Revisar `git diff` o `git status` — no hay archivos `.env` en el staging area
- [ ] No hay API keys, passwords ni tokens en el código nuevo
- [ ] El `.gitignore` incluye `.env` y archivos sensibles
- [ ] Los logs no contienen información sensible
- [ ] Los endpoints nuevos tienen autenticación si la necesitan
- [ ] Los inputs de usuario son validados con Pydantic antes de procesar
- [ ] No hay queries SQL construidas con f-strings (riesgo de SQL injection)
- [ ] No hay `print()` con datos de usuarios o credenciales
- [ ] Las respuestas de error no devuelven stack traces en producción

```python
# En producción, no devolver detalles del error
from fastapi import Request
from fastapi.responses import JSONResponse

@app.exception_handler(Exception)
async def generic_exception_handler(request: Request, exc: Exception):
    if settings.environment == "production":
        return JSONResponse(status_code=500, content={"detail": "Error interno"})
    raise exc  # En desarrollo, dejar que FastAPI muestre el error completo
```

---

## Ver también

**Hub:** [[DevOps]] — Hub central de DevOps, seguridad e infraestructura
- [[Puertos_y_Firewall]] — Qué puertos están expuestos y cómo cerrarlos
- [[Docker_Compose_Base]] — Cómo configurar Docker sin exponer puertos innecesarios
- [[Checklist_Pre_Produccion]] — Checklist completo antes de desplegar en producción
- [[INSTRUCCIONES_PARA_AGENTES]] — Reglas de seguridad que los agentes IA deben seguir
