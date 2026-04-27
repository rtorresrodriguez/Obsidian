# Mapa Mental del Sistema

> Navegación rápida: [[README]] · [[Arquitectura]] · [[Backend]] · [[DevOps]] · [[IA]] · [[INSTRUCCIONES_PARA_AGENTES]]

---

## Vista completa del sistema

```
╔══════════════════════════════════════════════════════════════════════╗
║                         INTERNET / USUARIOS                          ║
╚══════════════════════════════════════╤═══════════════════════════════╝
                                       │ HTTPS
                          ┌────────────▼────────────┐
                          │         NGINX            │
                          │  SSL · Reverse Proxy     │
                          │  Let's Encrypt           │
                          └──┬──────────────────┬───┘
                             │                  │
               ┌─────────────▼──┐    ┌──────────▼──────────┐
               │   FASTAPI      │    │      n8n             │
               │   :8000        │    │  :5678               │
               │   API REST     │    │  Workflows           │
               │   Auth/JWT     │    │  Webhooks            │
               │   Validación   │    │  Crons               │
               └───────┬────────┘    └──────────┬───────────┘
                       │                        │
                       │◄───────────────────────┘
                       │    n8n llama a FastAPI
                       │
          ┌────────────▼──────────────┐
          │       POSTGRESQL          │
          │       :5432               │
          │  Datos relacionales       │
          │  pgvector (RAG/embeddings)│
          └───────────────────────────┘

   ┌────────────────────────────────────────────────┐
   │           CAPA DE INTELIGENCIA                  │
   │                                                │
   │  Claude (Anthropic API)                        │
   │  claude-sonnet-4-6 · Haiku · Opus              │
   │                                                │
   │  Memoria:  PostgreSQL · RAG · Engram           │
   └────────────────────────────────────────────────┘

   ┌────────────────────────────────────────────────┐
   │           INFRAESTRUCTURA                       │
   │                                                │
   │  AWS EC2  →  Docker Compose  →  Volúmenes      │
   │  IP Elástica  →  Security Groups  →  UFW       │
   └────────────────────────────────────────────────┘
```

---

## Flujo de información: Casos de uso principales

### Caso 1 — Usuario habla con un agente IA

```
Usuario escribe mensaje
    ↓
HTTPS → Nginx → FastAPI /api/ai/chat
    ↓
FastAPI carga historial de conversación (PostgreSQL)
    ↓
FastAPI llama a Anthropic API (Claude) con contexto
    ↓
Claude genera respuesta usando Tools si necesita datos
    ↓
FastAPI guarda respuesta en PostgreSQL (memoria)
    ↓
Respuesta viaja: FastAPI → Nginx → Usuario
```
> Notas relacionadas: [[Arquitectura_Agente]] · [[Estructura_Proyecto]] · [[Modelos_y_Costos]]

---

### Caso 2 — Automatización por evento externo (webhook)

```
Evento externo (Gmail, Stripe, WhatsApp...)
    ↓
n8n recibe webhook en https://n8n.dominio.com/webhook/...
    ↓
n8n procesa el evento (transforma datos, filtra condiciones)
    ↓
n8n → POST https://api.dominio.com/api/endpoint (FastAPI)
    ↓
FastAPI valida con Pydantic → llama a Service → escribe en PostgreSQL
    ↓ (si involucra IA)
FastAPI → Anthropic API → respuesta procesada
    ↓
n8n recibe resultado → envía notificación (Slack, email, etc.)
```
> Notas relacionadas: [[n8n_Como_Orquestador]] · [[Endpoints_Documentados]] · [[Flujos_Documentados]]

---

### Caso 3 — Sistema RAG (base de conocimiento interna)

```
Documento nuevo ingresado al sistema
    ↓
Pipeline de ingesta: chunk_text() → embedding local (sentence-transformers)
    ↓
PostgreSQL + pgvector almacena chunks con vectores
    ↓
                    ↑ (cuando llega una consulta)
Usuario hace pregunta
    ↓
FastAPI genera embedding de la pregunta
    ↓
pgvector: búsqueda semántica → top-N chunks relevantes
    ↓
Claude: pregunta + contexto → respuesta con fuentes citadas
```
> Notas relacionadas: [[Memoria_y_RAG]] · [[Sistema_Memoria_Empresarial]] · [[Convenciones]]

---

### Caso 4 — Despliegue de una aplicación nueva

```
Código desarrollado localmente
    ↓
git push → servidor AWS (SSH o CI)
    ↓
docker compose build <servicio>
    ↓
docker compose up -d <servicio>
    ↓
Nginx: nginx -s reload (si cambió la config)
    ↓
Verificar: curl https://api.dominio.com/health
    ↓
Documentar cambio en Obsidian
```
> Notas relacionadas: [[Checklist_Pre_Produccion]] · [[Docker_Compose_Base]] · [[Reverse_Proxy]]

---

## Stack de tecnologías: referencias cruzadas

| Tecnología | Hub | Nota principal | Notas relacionadas |
|-----------|-----|---------------|-------------------|
| FastAPI | [[Backend]] | [[Estructura_Proyecto]] | [[Autenticacion_JWT]] · [[Endpoints_Documentados]] |
| PostgreSQL | [[Backend]] | [[Convenciones]] | [[Tablas/README\|Tablas]] · [[Plantilla_Tabla_PostgreSQL]] |
| Docker | [[DevOps]] | [[Docker_Compose_Base]] | [[Registro_Cambios_Docker]] |
| Nginx | [[DevOps]] | [[Reverse_Proxy]] | [[Dominios_Activos]] |
| n8n | [[DevOps]] | [[n8n_Como_Orquestador]] | [[Flujos_Documentados]] |
| Claude LLM | [[IA]] | [[Modelos_y_Costos]] | [[Arquitectura_Agente]] |
| RAG/Memoria | [[IA]] | [[Memoria_y_RAG]] | [[Obsidian_Como_Fuente_IA]] |
| Seguridad | [[DevOps]] | [[Checklist_Seguridad]] | [[Puertos_y_Firewall]] |
| Arquitectura | [[Arquitectura]] | [[Principios]] | [[Cuando_Usar_Cada_Servicio]] |

---

## Flujo de decisión al iniciar un proyecto nuevo

```
¿Tengo claro el objetivo?
    → NO → Detente. Define en 1 oración primero.
    → SÍ ↓

¿Revisé los principios de arquitectura?
    → NO → Lee [[Principios]] y [[Cuando_Usar_Cada_Servicio]]
    → SÍ ↓

¿Completé el checklist de nueva app?
    → NO → [[Checklist_Nueva_App]]
    → SÍ ↓

¿Definí el stack exacto?
    → FastAPI + PostgreSQL + Docker: [[Plantilla_Arquitectura]]
    → Con agente IA: también [[Arquitectura_Agente]] y [[Modelos_y_Costos]]
    → Con n8n: también [[n8n_Como_Orquestador]]
    ↓

Crear estructura → [[Estructura_Proyecto]]
Configurar Docker → [[Docker_Compose_Base]]
Configurar seguridad → [[Checklist_Seguridad]]
Documentar en → [[Plantilla_Proyecto]] → carpeta 14_Proyectos/
```

---

## Proyectos activos del sistema

| Proyecto | Descripción | Stack específico |
|---------|-------------|----------------|
| [[Agente_IA_FastAPI_n8n_PostgreSQL]] | Chatbot con memoria conversacional | FastAPI + Claude + PostgreSQL + n8n |
| [[Automatizacion_Correos_IA]] | Clasificación automática de correos | n8n + Claude Haiku + FastAPI |
| [[Sistema_Memoria_Empresarial]] | RAG sobre documentos internos | FastAPI + pgvector + sentence-transformers + Claude |

---

## Hubs de navegación

```
[[Arquitectura]]  ← Principios, plantillas, checklists
[[Backend]]       ← FastAPI, PostgreSQL, APIs, Auth
[[DevOps]]        ← Docker, Nginx, Seguridad, Despliegue
[[IA]]            ← Agentes, LLMs, Memoria, RAG
```
