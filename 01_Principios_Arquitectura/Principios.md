# Principios de Arquitectura

## Separación de responsabilidades

Cada capa del sistema tiene una única responsabilidad. Nunca mezcles lógica de negocio con acceso a datos, ni lógica de presentación con lógica de orquestación.

```
┌─────────────────────────────────────┐
│  Capa de presentación / Cliente     │  Solo muestra datos. No lógica.
├─────────────────────────────────────┤
│  Capa de API (FastAPI)              │  Valida, rutea, responde. No lógica de negocio compleja.
├─────────────────────────────────────┤
│  Capa de servicios (Services)       │  Aquí vive la lógica de negocio.
├─────────────────────────────────────┤
│  Capa de acceso a datos (Models)    │  Solo interacción con PostgreSQL.
├─────────────────────────────────────┤
│  Base de datos (PostgreSQL)         │  Solo almacena y consulta. Sin lógica de app.
└─────────────────────────────────────┘
```

### Regla práctica

- El router recibe la request y llama al servicio.
- El servicio contiene la lógica de negocio y llama al modelo.
- El modelo interactúa con la base de datos.
- **Nunca** llames a la base de datos directamente desde el router.
- **Nunca** pongas lógica de negocio dentro del router.

---

## Arquitectura en capas

### Backend (FastAPI)

```
app/
├── main.py          ← Inicialización de la app, middlewares, montaje de routers
├── routers/         ← Endpoints agrupados por dominio
├── services/        ← Lógica de negocio
├── models/          ← Modelos SQLAlchemy (ORM)
├── schemas/         ← Schemas Pydantic (validación de entrada/salida)
├── database.py      ← Conexión a PostgreSQL, session factory
└── config.py        ← Variables de entorno y configuración
```

### Base de datos (PostgreSQL)

- Solo almacena datos. No ejecuta lógica de negocio via stored procedures complejos.
- Las relaciones entre tablas son responsabilidad del ORM (SQLAlchemy), no de triggers.
- Los índices se definen explícitamente para las consultas más frecuentes.

### Orquestación (n8n)

- n8n orquesta flujos entre servicios externos, webhooks y la API interna.
- No reemplaza a FastAPI. Los flujos de n8n llaman endpoints de FastAPI, no la base de datos directamente.
- n8n es el pegamento entre servicios, no el lugar donde vive la lógica de negocio.

### IA (LLM / Agentes)

- El LLM es una capa de decisión, no de persistencia.
- El LLM nunca escribe directamente en la base de datos. Lo hace vía la API de FastAPI.
- La memoria del agente se gestiona en una capa separada (RAG, Engram, o sesión).

---

## Buenas prácticas para no mezclar servicios

| Mala práctica | Buena práctica |
|---------------|----------------|
| n8n conectado directamente a PostgreSQL para operaciones de negocio | n8n llama a FastAPI, FastAPI escribe en PostgreSQL |
| LLM ejecutando queries SQL | LLM llama a herramientas/tools que usan la API |
| Lógica de negocio en el router de FastAPI | Lógica en el service, router solo rutea |
| Credenciales en el código | Variables de entorno en `.env` |
| Un solo `main.py` con 500 líneas | Separado en routers, services, models |
| Docker para todo incluyendo scripts temporales | Docker para servicios de larga duración |

---

## Cuándo usar Docker

**Usa Docker cuando:**
- El servicio debe correr de forma persistente (API, base de datos, n8n, Nginx)
- Necesitas aislar dependencias entre proyectos
- Quieres reproducibilidad en desarrollo y producción
- El servicio tiene una configuración compleja de arranque

**No necesariamente necesitas Docker para:**
- Scripts de migración de una sola vez
- Herramientas de desarrollo local simples
- Procesamiento de datos ad-hoc
- Testing unitario

---

## Cuándo usar FastAPI

**Usa FastAPI cuando:**
- Necesitas exponer una API REST que otros servicios consuman
- Necesitas validación de datos de entrada con esquemas
- Necesitas documentación automática (Swagger / OpenAPI)
- Necesitas autenticación JWT o OAuth2
- Quieres rendimiento alto con Python (async)

**No uses FastAPI para:**
- Scripts de procesamiento de datos que no necesitan exponerse como API
- Flujos simples de automatización (para eso usa n8n)
- Servidores estáticos (para eso Nginx directamente)

---

## Cuándo usar n8n

**Usa n8n cuando:**
- Necesitas conectar múltiples servicios sin escribir código
- Tienes flujos de datos periódicos (crons, eventos)
- Necesitas manejar webhooks de servicios externos (Stripe, WhatsApp, Gmail)
- Quieres orquestar un agente IA paso a paso visualmente
- Necesitas transformar y enrutar datos entre sistemas

**No uses n8n para:**
- Lógica de negocio compleja (mejor en FastAPI)
- Almacenamiento de datos permanente (mejor PostgreSQL)
- Servir una API REST con autenticación compleja (mejor FastAPI)

---

## Cuándo usar PostgreSQL

**Usa PostgreSQL cuando:**
- Necesitas persistencia de datos relacionales
- Tienes datos estructurados con relaciones entre entidades
- Necesitas transacciones ACID
- Necesitas consultas complejas con JOINs
- Necesitas full-text search básico

**Considera alternativas cuando:**
- Datos no estructurados o semi-estructurados (considera MongoDB o JSON columns en PG)
- Caché temporal de alta velocidad (considera Redis)
- Embeddings vectoriales (considera pgvector extension en PostgreSQL, o Qdrant/Pinecone)

---

## Cuándo usar memoria/RAG

**Usa RAG cuando:**
- El agente IA necesita responder basándose en documentos específicos del negocio
- El contexto es demasiado largo para pasarlo en el prompt
- Necesitas que el agente tenga "conocimiento" de documentos que cambian frecuentemente

**Usa Engram u otra memoria conversacional cuando:**
- El agente necesita recordar conversaciones previas con un usuario
- Quieres persistencia de contexto entre sesiones

**Usa solo el contexto del prompt cuando:**
- La información es corta y no cambia
- Es una tarea de una sola sesión

---

## Ver también

**Hub:** [[Arquitectura]] — Hub central de arquitectura
- [[Cuando_Usar_Cada_Servicio]] — Árbol de decisión concreto para elegir tecnología
- [[Plantilla_Arquitectura]] — Cómo se implementan estos principios en una app real
- [[Checklist_Nueva_App]] — Verificar que se siguen los principios antes de empezar
- [[Estructura_Proyecto]] — Aplicación de la separación de capas en FastAPI
- [[INSTRUCCIONES_PARA_AGENTES]] — Reglas de comportamiento para agentes IA
