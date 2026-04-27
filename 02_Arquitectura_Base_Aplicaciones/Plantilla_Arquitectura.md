# Plantilla de Arquitectura Base: FastAPI + PostgreSQL + n8n + LLM

Esta es la arquitectura por defecto para cualquier aplicación nueva en este sistema.

## Diagrama de flujo

```
┌──────────────────────────────────────────────────────────┐
│                    CAPA DE ENTRADA                        │
│  Cliente Web / App / Webhook externo / n8n trigger        │
└──────────────────────┬───────────────────────────────────┘
                       │ HTTPS
                       ▼
┌──────────────────────────────────────────────────────────┐
│                      NGINX                                │
│  Reverse proxy | SSL termination | Rate limiting          │
│  Puerto 80 → redirect 443                                 │
│  Puerto 443 → proxy_pass → FastAPI:8000                   │
└──────────────────────┬───────────────────────────────────┘
                       │ HTTP interno
                       ▼
┌──────────────────────────────────────────────────────────┐
│                   FASTAPI (puerto 8000)                   │
│                                                          │
│  main.py → routers → services → models                   │
│                                                          │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   │
│  │   Router    │→  │   Service   │→  │    Model    │   │
│  │  (HTTP)     │   │  (lógica)   │   │   (datos)   │   │
│  └─────────────┘   └─────────────┘   └──────┬──────┘   │
│                           │                  │           │
│                    ┌──────▼──────┐           │           │
│                    │ Anthropic   │           │           │
│                    │   API       │           │           │
│                    └─────────────┘           │           │
└──────────────────────────────────────────────┼──────────┘
                                               │
                       ┌───────────────────────▼──────────┐
                       │         POSTGRESQL (5432)          │
                       │  Tablas, relaciones, índices       │
                       └───────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│                    N8N (puerto 5678)                      │
│  Orquestación | Webhooks | Crons | Integraciones         │
│  → Llama a FastAPI via HTTP                               │
│  → No conecta directamente a PostgreSQL                   │
└──────────────────────────────────────────────────────────┘
```

## Flujo recomendado de datos

### Flujo de lectura (GET)

```
Cliente → Nginx → FastAPI Router → Service → Model → PostgreSQL
                                                        ↓
Cliente ← Nginx ← FastAPI Router ← Service ← Schema (Pydantic response)
```

### Flujo de escritura (POST/PUT)

```
Cliente → Nginx → FastAPI Router
                      ↓
              Schema Pydantic (validación)
                      ↓
              Service (lógica de negocio)
                      ↓
              Model (ORM SQLAlchemy)
                      ↓
              PostgreSQL (commit)
                      ↓
              Response Schema → Cliente
```

### Flujo con LLM

```
Cliente → FastAPI → Service
                      ↓
              Anthropic API (Claude)
                      ↓
              Procesar respuesta
                      ↓
              Guardar en PostgreSQL (si aplica)
                      ↓
              Responder al cliente
```

### Flujo con n8n como orquestador

```
Trigger (cron / webhook externo)
    ↓
n8n Flow
    ↓
HTTP Request node → POST /api/endpoint (FastAPI)
    ↓
FastAPI procesa → PostgreSQL
    ↓
Response → n8n
    ↓
n8n → notificación / siguiente paso
```

## Estructura de carpetas del proyecto

```
nombre_proyecto/
├── app/
│   ├── __init__.py
│   ├── main.py                 ← Inicialización, middlewares, routers
│   ├── config.py               ← Pydantic Settings, vars de entorno
│   ├── database.py             ← Engine SQLAlchemy, SessionLocal, Base
│   ├── dependencies.py         ← Dependencias comunes (get_db, get_current_user)
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── users.py
│   │   ├── items.py
│   │   └── ai.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── user_service.py
│   │   ├── item_service.py
│   │   └── ai_service.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── item.py
│   └── schemas/
│       ├── __init__.py
│       ├── user.py
│       └── item.py
├── tests/
│   ├── __init__.py
│   ├── test_users.py
│   └── test_items.py
├── alembic/                    ← Migraciones de base de datos
│   ├── versions/
│   └── env.py
├── .env                        ← Variables de entorno (NO subir a git)
├── .env.example                ← Plantilla de variables (SÍ subir a git)
├── .gitignore
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
└── README.md
```

---

## Ver también

**Hub:** [[Arquitectura]] — Hub central de arquitectura
- [[Principios]] — Los principios de diseño que esta plantilla implementa
- [[Cuando_Usar_Cada_Servicio]] — Cómo decidir qué componentes incluir
- [[Estructura_Proyecto]] — Detalle de la estructura interna de FastAPI
- [[Docker_Compose_Base]] — docker-compose.yml completo con todos los servicios
- [[Checklist_Nueva_App]] — Verificaciones antes de crear la app
- [[Checklist_Pre_Produccion]] — Verificaciones antes de desplegar
