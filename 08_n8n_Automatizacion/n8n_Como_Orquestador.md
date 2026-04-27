# n8n como Orquestador

## ¿Para qué sirve n8n?

n8n es una herramienta de automatización de flujos de trabajo (workflow automation) de código abierto. En este sistema, actúa como el pegamento entre servicios.

**Casos de uso principales:**
- Conectar webhooks externos (Stripe, WhatsApp, Gmail, Telegram) con la API interna
- Ejecutar tareas programadas (crons) sin escribir código
- Orquestar flujos de un agente IA paso a paso
- Enviar notificaciones, correos, mensajes de Slack
- Transformar y enrutar datos entre sistemas
- Crear pipelines de datos sin código

**n8n NO es un reemplazo de FastAPI.** Es el orquestador externo; FastAPI es el núcleo de la lógica de negocio.

---

## Cómo conectar n8n con FastAPI

### Desde n8n hacia FastAPI

Usa el nodo **HTTP Request** en n8n:

```
Node: HTTP Request
Method: POST
URL: http://api:8000/api/endpoint   ← URL interna Docker (sin Nginx)
     o
URL: https://api.tu-dominio.com/api/endpoint  ← URL pública (con Nginx)

Headers:
  Content-Type: application/json
  Authorization: Bearer {{ $env.API_KEY }}  ← Usar credenciales de n8n

Body (JSON):
{
  "data": "{{ $json.data }}",
  "user_id": "{{ $json.user_id }}"
}
```

### Autenticación entre n8n y FastAPI

```python
# En FastAPI: verificar API key
from fastapi import Security, HTTPException
from fastapi.security import APIKeyHeader

api_key_header = APIKeyHeader(name="Authorization")

async def verify_api_key(api_key: str = Security(api_key_header)):
    if api_key != f"Bearer {settings.internal_api_key}":
        raise HTTPException(status_code=403, detail="API key inválida")
    return api_key
```

```bash
# En .env
INTERNAL_API_KEY=clave_secreta_larga_para_n8n
```

---

## Cómo conectar n8n con PostgreSQL

n8n puede conectarse directamente a PostgreSQL usando el nodo **Postgres**. Sin embargo, **en este sistema preferimos que n8n llame a FastAPI** en lugar de conectar directamente a la base de datos. Esto garantiza:
- Validación de datos (Pydantic)
- Lógica de negocio centralizada
- Control de acceso

**Excepción:** Puedes usar el nodo Postgres directamente si el flujo es de solo lectura para reportes y n8n es el consumidor final.

```
Nodo PostgreSQL en n8n:
Host: postgres       ← nombre del servicio Docker
Port: 5432
Database: mi_base_de_datos
User: {{ $env.POSTGRES_USER }}
Password: {{ $env.POSTGRES_PASSWORD }}
```

---

## Cómo usar webhooks en n8n

### Webhook que recibe datos externos

```
1. Crear nodo "Webhook" en n8n
2. Método: POST
3. URL generada: https://n8n.tu-dominio.com/webhook/mi-webhook
4. Copiar URL → configurar en el servicio externo (Stripe, WhatsApp, etc.)
```

### Validar el webhook

```python
# Si el servicio externo firma los webhooks (ej. Stripe)
# n8n tiene nodo "Stripe Trigger" que ya maneja la validación

# Para webhooks sin firma, puedes validar con un secret en el header
Headers del webhook en n8n:
  x-webhook-secret: {{ $env.WEBHOOK_SECRET }}
```

---

## Ejemplos de flujos para agentes IA

### Flujo 1: Chatbot simple con Claude

```
Trigger: Webhook (POST /webhook/chat)
    ↓
Nodo: HTTP Request → POST https://api.tu-dominio.com/api/ai/chat
  Body: { "message": "{{ $json.message }}", "session_id": "{{ $json.session_id }}" }
    ↓
Nodo: Respond to Webhook
  Body: { "response": "{{ $json.response }}" }
```

### Flujo 2: Procesamiento de correos con IA

```
Trigger: Email Trigger (Gmail / IMAP) - cada 5 minutos
    ↓
Nodo: IF → ¿El correo es de un remitente importante?
    ↓ SÍ
Nodo: HTTP Request → POST /api/ai/classify-email
  Body: { "subject": "...", "body": "..." }
    ↓
Nodo: Switch → según clasificación
  → "urgente"   → Slack notification
  → "tarea"     → POST /api/tasks (crear tarea)
  → "ignorar"   → End
```

### Flujo 3: Agente con memoria y contexto

```
Trigger: Webhook
    ↓
Nodo: HTTP Request → GET /api/memory/{{ $json.user_id }}
  (obtener memoria del usuario)
    ↓
Nodo: HTTP Request → POST /api/ai/agent
  Body: {
    "message": "{{ $json.message }}",
    "memory": "{{ $json.memory }}",
    "user_id": "{{ $json.user_id }}"
  }
    ↓
Nodo: HTTP Request → POST /api/memory/{{ $json.user_id }}
  (guardar nueva memoria)
    ↓
Nodo: Respond to Webhook
```

### Flujo 4: Pipeline de datos nocturno

```
Trigger: Schedule (cron: 0 2 * * *)  ← 2 AM todos los días
    ↓
Nodo: HTTP Request → GET /api/reports/pending
    ↓
Nodo: Loop sobre cada reporte
    ↓
  Nodo: HTTP Request → POST /api/ai/generate-report
  Body: { "data": "{{ $json.report_data }}" }
    ↓
  Nodo: Gmail → Enviar reporte por correo
    ↓
  Nodo: HTTP Request → PATCH /api/reports/{{ $json.id }}/status
  Body: { "status": "sent" }
```

---

## Variables de entorno en n8n

n8n puede acceder a variables de entorno con `{{ $env.VARIABLE_NAME }}`. Define las variables en el `docker-compose.yml`:

```yaml
n8n:
  environment:
    - N8N_BASIC_AUTH_ACTIVE=true
    - API_BASE_URL=http://api:8000
    - INTERNAL_API_KEY=${INTERNAL_API_KEY}
    - WEBHOOK_SECRET=${WEBHOOK_SECRET}
```

## Buenas prácticas n8n

- Usa credenciales de n8n (menú Credenciales) en lugar de poner secrets directamente en los nodos
- Siempre activa autenticación básica (`N8N_BASIC_AUTH_ACTIVE=true`)
- Usa el nodo "Error Trigger" para manejar errores y recibir notificaciones
- Nombra los flujos claramente: `[DOMINIO] Descripción del flujo`
- Documenta el propósito del flujo en la descripción del workflow
- No uses n8n para lógica de negocio compleja; ponla en FastAPI

---

## Ver también

**Hubs:** [[DevOps]] — Infraestructura donde corre n8n · [[IA]] — n8n como orquestador de agentes
- [[Flujos_Documentados]] — Registro de todos los flujos activos en n8n
- [[Estructura_Proyecto]] — Endpoints de FastAPI que n8n puede llamar
- [[Arquitectura_Agente]] — Cómo n8n puede orquestar agentes IA
- [[Cuando_Usar_Cada_Servicio]] — Cuándo usar n8n vs escribir código en FastAPI
- [[Docker_Compose_Base]] — Configuración de n8n en docker-compose
- [[Reverse_Proxy]] — Cómo exponer n8n via Nginx con autenticación
- [[Endpoints_Documentados]] — Los endpoints de FastAPI disponibles para los flujos
