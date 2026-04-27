# Proyecto: Agente IA con FastAPI + n8n + PostgreSQL

## Descripción

Sistema de agente IA conversacional con memoria persistente. El usuario envía mensajes via API, el agente usa Claude para razonar, guarda el historial en PostgreSQL, y n8n orquesta flujos automáticos (notificaciones, reportes).

## Stack técnico

| Componente | Tecnología | Versión |
|-----------|-----------|--------|
| Backend | FastAPI | 0.110+ |
| LLM | Anthropic Claude | claude-sonnet-4-6 |
| Base de datos | PostgreSQL | 15 |
| Orquestación | n8n | Latest |
| Reverse proxy | Nginx | Alpine |
| Contenedores | Docker + Compose | v2 |
| Infraestructura | AWS EC2 | - |

## Arquitectura

```
Usuario → POST /api/ai/chat
              ↓
         FastAPI (Router)
              ↓
         AIService
           ├── Obtener historial de PostgreSQL
           ├── Construir contexto
           ├── Llamar Anthropic API (Claude)
           └── Guardar respuesta en PostgreSQL
              ↓
         Response al usuario

n8n (paralelo):
  - Trigger diario → GET /api/ai/reports/daily → Email
  - Webhook externo → POST /api/ai/process-event → Agente
```

## Estructura de carpetas

```
agente_ia/
├── app/
│   ├── main.py
│   ├── config.py
│   ├── database.py
│   ├── routers/
│   │   └── ai.py
│   ├── services/
│   │   └── ai_service.py
│   ├── models/
│   │   ├── conversation.py
│   │   └── message.py
│   └── schemas/
│       └── ai.py
├── .env
├── .env.example
├── requirements.txt
├── Dockerfile
└── docker-compose.yml
```

## Esquema de base de datos

### Tabla: conversations

| Columna | Tipo | Descripción |
|---------|------|-------------|
| id | UUID | Clave primaria |
| user_id | VARCHAR | Identificador del usuario |
| title | VARCHAR | Título de la conversación |
| created_at | TIMESTAMP | Fecha de creación |
| updated_at | TIMESTAMP | Última actualización |

### Tabla: messages

| Columna | Tipo | Descripción |
|---------|------|-------------|
| id | SERIAL | Clave primaria |
| conversation_id | UUID | FK → conversations |
| role | VARCHAR | 'user' o 'assistant' |
| content | TEXT | Contenido del mensaje |
| tokens_used | INTEGER | Tokens consumidos |
| created_at | TIMESTAMP | Fecha |

## Variables de entorno

```bash
# .env.example
DATABASE_URL=postgresql://user:PASSWORD@postgres:5432/agente_ia_db
ANTHROPIC_API_KEY=sk-ant-OBTENER_EN_console.anthropic.com
SECRET_KEY=GENERAR_ALEATORIAMENTE
MAX_CONVERSATION_TOKENS=100000
DEFAULT_MODEL=claude-sonnet-4-6
```

## Estado del proyecto

- [ ] Definición de arquitectura
- [ ] Setup inicial (carpetas, Docker, .env)
- [ ] Modelos de base de datos
- [ ] Endpoint de chat básico
- [ ] Memoria conversacional (historial)
- [ ] Integración con n8n
- [ ] Despliegue en AWS
- [ ] SSL con Let's Encrypt
- [ ] Documentación de endpoints

## Decisiones de arquitectura

**¿Por qué UUID para conversation_id?**
Las conversaciones pueden crearse desde múltiples instancias del servicio; UUID evita colisiones sin coordinar con la base de datos.

**¿Por qué guardar tokens_used?**
Para monitorear costos de la API de Anthropic y implementar límites por usuario en el futuro.

**¿Por qué n8n para los reportes y no un cron en Python?**
n8n permite cambiar la lógica de los flujos sin redesplegar el backend. Los no-técnicos pueden ajustar cuándo y cómo se envían los reportes.

---

## Ver también

- [[Arquitectura_Agente]] — Patrones de agentes IA usados en este proyecto
- [[Modelos_y_Costos]] — Decisión del modelo Claude para este proyecto
- [[Estructura_Proyecto]] — Estructura FastAPI base de este proyecto
- [[n8n_Como_Orquestador]] — Cómo n8n orquesta los flujos de este agente
- [[Memoria_y_RAG]] — Implementación de la memoria conversacional
- [[Plantilla_Proyecto]] — Plantilla base usada para crear este documento
