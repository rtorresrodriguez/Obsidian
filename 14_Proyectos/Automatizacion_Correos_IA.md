# Proyecto: Automatización de correos con IA

## Descripción

Sistema que lee correos entrantes, los clasifica con IA (Claude), extrae información clave, y ejecuta acciones automáticas según la clasificación (responder, crear tarea, notificar por Slack, etc.). Orquestado completamente por n8n con IA via FastAPI.

## Stack técnico

| Componente | Uso |
|-----------|-----|
| n8n | Orquestador principal: leer correos, ejecutar acciones |
| FastAPI | Endpoint de clasificación y procesamiento con IA |
| Claude | Clasificar y extraer información del correo |
| PostgreSQL | Guardar clasificaciones y acciones ejecutadas |
| Gmail API | Lectura de correos (via n8n nativo) |

## Flujo

```
1. n8n Gmail Trigger (cada 5 min)
   ↓
2. Filtrar: ¿Ya fue procesado? (check en PostgreSQL)
   ↓ NO procesado
3. POST /api/email/classify
   Body: { subject, body, from, date }
   ↓
4. FastAPI → Claude: clasificar correo
   Respuesta: { category, priority, summary, action, extracted_data }
   ↓
5. n8n Switch según category:
   → "soporte"    → Crear ticket en sistema de soporte
   → "factura"    → Guardar en tabla facturas + notificar contador
   → "lead"       → Crear contacto en CRM + notificar ventas
   → "spam"       → Marcar como leído, ignorar
   → "urgente"    → Slack alert + respuesta automática
   ↓
6. POST /api/email/log (guardar acción ejecutada)
```

## Endpoint principal

### POST /api/email/classify

**Request:**
```json
{
  "message_id": "gmail_message_id_123",
  "from": "cliente@empresa.com",
  "subject": "Factura vencida - urgente",
  "body": "Buen día, les escribo porque...",
  "date": "2024-01-15T10:30:00Z"
}
```

**Response:**
```json
{
  "category": "factura",
  "priority": "alta",
  "summary": "Cliente reclama factura N°123 vencida hace 30 días",
  "action": "notificar_contador",
  "extracted_data": {
    "invoice_number": "123",
    "days_overdue": 30,
    "client_email": "cliente@empresa.com"
  }
}
```

## Prompt de clasificación (Claude)

```python
CLASSIFICATION_PROMPT = """
Analiza el siguiente correo electrónico y clasifícalo.

Correo:
De: {from_email}
Asunto: {subject}
Cuerpo: {body}

Responde ÚNICAMENTE en formato JSON con esta estructura:
{{
  "category": "soporte|factura|lead|spam|urgente|otro",
  "priority": "alta|media|baja",
  "summary": "resumen en una oración",
  "action": "acción recomendada",
  "extracted_data": {{}}
}}

Reglas de clasificación:
- "urgente": menciona palabras como urgente, inmediato, crítico, o tiene señales de problema grave
- "factura": relacionado con pagos, facturas, cobros
- "lead": potencial cliente nuevo interesado en los servicios
- "soporte": cliente existente con problema técnico
- "spam": publicidad no solicitada, correos automatizados irrelevantes
"""
```

## Estado del proyecto

- [ ] Diseño del flujo en n8n
- [ ] Endpoint de clasificación en FastAPI
- [ ] Tabla de log en PostgreSQL
- [ ] Conexión Gmail en n8n
- [ ] Acciones por categoría
- [ ] Pruebas con correos reales
- [ ] Manejo de errores y reintentos

---

## Ver también

- [[n8n_Como_Orquestador]] — Patrones de n8n usados en este proyecto
- [[Arquitectura_Agente]] — Cómo Claude clasifica los correos como agente
- [[Modelos_y_Costos]] — Por qué usar Haiku para clasificación masiva de correos
- [[Estructura_Proyecto]] — Estructura del endpoint de clasificación en FastAPI
- [[Plantilla_Proyecto]] — Plantilla base usada para crear este documento
