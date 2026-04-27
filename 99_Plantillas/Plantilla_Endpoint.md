# Endpoint: [MÉTODO] /api/ruta

> Servicio: [Nombre del proyecto]
> Fecha: YYYY-MM-DD
> Autor: [Nombre]

## Descripción

Qué hace este endpoint en una oración.

## Autenticación

- [ ] Sin autenticación (público)
- [ ] Bearer Token (JWT)
- [ ] API Key (header)
- [ ] Basic Auth

## Request

**Headers:**
```
Content-Type: application/json
Authorization: Bearer <token>
```

**Path parameters:**
| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| id | integer | ID del recurso |

**Query parameters:**
| Parámetro | Tipo | Requerido | Default | Descripción |
|-----------|------|-----------|---------|-------------|
| page | integer | NO | 1 | Número de página |
| limit | integer | NO | 20 | Resultados por página |

**Body (JSON):**
```json
{
  "campo1": "string",
  "campo2": 0
}
```

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| campo1 | string | SÍ | Descripción |
| campo2 | integer | NO | Descripción |

## Response

**200 OK:**
```json
{
  "id": 1,
  "campo1": "valor"
}
```

**400 Bad Request:**
```json
{"detail": "Descripción del error de validación"}
```

**404 Not Found:**
```json
{"detail": "Recurso no encontrado"}
```

## Ejemplo de uso

```bash
curl -X POST https://api.tu-dominio.com/api/ruta \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"campo1": "valor"}'
```

```python
import httpx

response = httpx.post(
    "https://api.tu-dominio.com/api/ruta",
    headers={"Authorization": "Bearer TOKEN"},
    json={"campo1": "valor"}
)
data = response.json()
```

## Notas

- Consideraciones especiales del endpoint
- Límites de rate
- Efectos secundarios (si escribe en otras tablas, envía emails, etc.)

---

*Plantilla — guardar el documento completado en `11_Integraciones_API/`*
**Ver también:** [[Endpoints_Documentados]] · [[Backend]] · [[Autenticacion_JWT]]
