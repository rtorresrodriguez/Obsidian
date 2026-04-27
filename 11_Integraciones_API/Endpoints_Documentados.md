# Documentación de Endpoints

## Cómo documentar un endpoint

Usa este formato estándar para cada endpoint que crees. Agrega entradas a este archivo o crea un archivo separado por servicio en esta carpeta.

---

## Plantilla de documentación de endpoint

```markdown
### [MÉTODO] /api/ruta/endpoint

**Descripción:** Qué hace este endpoint en una línea.
**Autenticación:** Bearer Token / API Key / Ninguna
**Tags:** [dominio, funcionalidad]

#### Request

Headers:
```
Authorization: Bearer <token>
Content-Type: application/json
```

Body (JSON):
```json
{
  "campo1": "string",
  "campo2": 123,
  "campo3": true
}
```

| Campo    | Tipo    | Requerido | Descripción              |
|----------|---------|-----------|--------------------------|
| campo1   | string  | SÍ        | Descripción del campo    |
| campo2   | integer | NO        | Valor por defecto: 0     |
| campo3   | boolean | NO        | Por defecto: true        |

#### Responses

**200 OK:**
```json
{
  "id": 1,
  "campo1": "valor",
  "created_at": "2024-01-01T00:00:00Z"
}
```

**400 Bad Request:**
```json
{
  "detail": "campo1 es requerido"
}
```

**401 Unauthorized:**
```json
{
  "detail": "Token inválido o expirado"
}
```

**500 Internal Server Error:**
```json
{
  "detail": "Error interno del servidor"
}
```

#### Ejemplo de uso (curl)

```bash
curl -X POST https://api.tu-dominio.com/api/ruta/endpoint \
  -H "Authorization: Bearer TU_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"campo1": "valor", "campo2": 123}'
```
```

---

## Endpoints del sistema (completar con los reales)

### GET /health

**Descripción:** Verificar que el servidor está en línea.
**Autenticación:** Ninguna

**Response 200:**
```json
{"status": "ok"}
```

---

### POST /api/ai/chat

**Descripción:** Enviar un mensaje al agente IA y recibir respuesta.
**Autenticación:** Bearer Token

**Body:**
```json
{
  "message": "¿Cuántos usuarios se registraron hoy?",
  "session_id": "user_123_session_abc"
}
```

**Response 200:**
```json
{
  "response": "Hoy se registraron 15 usuarios nuevos.",
  "session_id": "user_123_session_abc",
  "tokens_used": 245
}
```

---

## Cómo documentar payloads JSON

### Tipos de datos en JSON

| Tipo Python | Tipo JSON | Ejemplo |
|------------|----------|---------|
| str | string | `"nombre"` |
| int | integer | `42` |
| float | number | `3.14` |
| bool | boolean | `true` / `false` |
| list | array | `[1, 2, 3]` |
| dict | object | `{"key": "value"}` |
| None | null | `null` |
| datetime | string ISO 8601 | `"2024-01-15T10:30:00Z"` |

### Validación de payloads (Pydantic)

```python
from pydantic import BaseModel, EmailStr, Field
from datetime import datetime

class CreateUserRequest(BaseModel):
    name: str = Field(..., min_length=2, max_length=100, description="Nombre completo")
    email: EmailStr = Field(..., description="Email válido")
    age: int = Field(..., ge=0, le=150, description="Edad en años")
    tags: list[str] = Field(default=[], description="Etiquetas opcionales")

class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    created_at: datetime

    class Config:
        from_attributes = True
```

---

## Cómo documentar errores de API

### Códigos de error estándar del sistema

| Código HTTP | Significado | Cuándo ocurre |
|------------|------------|--------------|
| 200 | OK | Operación exitosa |
| 201 | Created | Recurso creado exitosamente |
| 400 | Bad Request | Datos de entrada inválidos |
| 401 | Unauthorized | Token ausente o inválido |
| 403 | Forbidden | Sin permisos para esta acción |
| 404 | Not Found | El recurso no existe |
| 409 | Conflict | El recurso ya existe (ej: email duplicado) |
| 422 | Unprocessable Entity | Error de validación Pydantic |
| 500 | Internal Server Error | Error no manejado en el servidor |
| 503 | Service Unavailable | Servicio externo no disponible |

### Formato estándar de errores en FastAPI

```python
# Siempre devolver errores en este formato:
{
  "detail": "Mensaje descriptivo del error"
}

# Para errores de validación Pydantic (automático en FastAPI):
{
  "detail": [
    {
      "loc": ["body", "email"],
      "msg": "value is not a valid email address",
      "type": "value_error.email"
    }
  ]
}
```

---

## Plantilla de integración con API externa

```python
# services/external_api_service.py
import httpx
from app.config import settings

class ExternalAPIService:
    BASE_URL = "https://api.servicio-externo.com"
    
    @classmethod
    async def get_data(cls, resource_id: str) -> dict:
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"{cls.BASE_URL}/resource/{resource_id}",
                headers={"Authorization": f"Bearer {settings.external_api_key}"},
                timeout=30.0
            )
            response.raise_for_status()
            return response.json()
    
    @classmethod
    async def create_resource(cls, data: dict) -> dict:
        async with httpx.AsyncClient() as client:
            response = await client.post(
                f"{cls.BASE_URL}/resource",
                json=data,
                headers={
                    "Authorization": f"Bearer {settings.external_api_key}",
                    "Content-Type": "application/json"
                },
                timeout=30.0
            )
            response.raise_for_status()
            return response.json()
```

---

## Ver también

**Hub:** [[Backend]] — Hub central del backend (FastAPI + PostgreSQL + APIs)
- [[Plantilla_Endpoint]] — Plantilla en blanco para documentar un endpoint nuevo
- [[Estructura_Proyecto]] — Cómo se implementan estos endpoints en FastAPI
- [[Autenticacion_JWT]] — Cómo funciona la autenticación en los endpoints protegidos
- [[Integraciones_Externas]] — APIs externas que el sistema consume (vs. las que expone)
- [[n8n_Como_Orquestador]] — Estos endpoints son los que n8n llama desde sus flujos
