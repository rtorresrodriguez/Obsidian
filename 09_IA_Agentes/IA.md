# Hub: IA y Agentes

> Este archivo es el hub central de inteligencia artificial. Desde aquí navega a agentes, LLMs, memoria y RAG.
> Hubs del sistema: [[Arquitectura]] · [[Backend]] · [[DevOps]] · [[MAPA_MENTAL]]

---

## ¿Cómo se usa la IA en este sistema?

La IA no es un componente aislado. Es una capa de decisión que se integra con el resto del stack:

```
ENTRADA
   │
   ▼
FastAPI Service
   │
   ├── Llama a Anthropic API (Claude)
   │       │
   │       ├── Genera texto / razona
   │       └── Decide qué Tool usar
   │               │
   │        Tools disponibles:
   │           ├── GET /api/data          ← Lee de PostgreSQL via FastAPI
   │           ├── POST /api/action       ← Escribe via FastAPI
   │           ├── search_knowledge_base  ← RAG en pgvector
   │           └── send_notification      ← Email, Slack, etc.
   │
   └── Guarda resultado en PostgreSQL
           ↑
           │  (memoria para próxima sesión)
```

**Principio fundamental:** El LLM decide, las Tools ejecutan. El LLM nunca accede a la base de datos directamente.

---

## Modelos disponibles

| Modelo | Cuándo usarlo | Costo relativo |
|--------|--------------|---------------|
| claude-haiku-4-5-20251001 | Clasificación, extracción, tareas simples y de alto volumen | Bajo |
| claude-sonnet-4-6 | **Default** — balance calidad/costo para la mayoría de casos | Medio |
| claude-opus-4-7 | Razonamiento complejo, decisiones críticas, multiagente | Alto |

> Detalle y configuración de código: [[Modelos_y_Costos]]

---

## Tipos de memoria disponibles

| Tipo | Duración | Cuándo usar | Implementación |
|------|---------|------------|---------------|
| Contexto del prompt | Solo esta sesión | Instrucciones y datos cortos | Lista de mensajes |
| PostgreSQL | Permanente | Historial de conversaciones | Tabla `messages` |
| RAG (pgvector) | Permanente | Documentos extensos, búsqueda semántica | [[Memoria_y_RAG]] |
| Engram | Permanente | Preferencias y contexto de usuario entre sesiones | [[Memoria_y_RAG]] |
| Este Obsidian | Permanente | Arquitectura y reglas del sistema | [[Obsidian_Como_Fuente_IA]] |

---

## Patrones de agente

| Patrón | Cuándo usar | Nota |
|--------|------------|------|
| Agente simple (loop) | Una tarea con herramientas | [[Arquitectura_Agente]] |
| Agente con memoria | Chatbot con historial | [[Arquitectura_Agente]] + [[Memoria_y_RAG]] |
| Agente + n8n | Flujo orquestado visualmente | [[n8n_Como_Orquestador]] |
| Multiagente | Tareas complejas paralelizables | [[Arquitectura_Agente]] (sección multiagente) |
| RAG sobre documentos | Base de conocimiento interna | [[Memoria_y_RAG]] |

---

## Sub-notas de IA

### Agentes y LLMs
- [[Arquitectura_Agente]] — Agente simple, multiagente, tools, patrones de implementación
- [[Modelos_y_Costos]] — Claude Haiku / Sonnet / Opus, cuándo usar cada uno, monitoreo de costos

### Memoria y conocimiento
- [[Memoria_y_RAG]] — Diferencias Obsidian / RAG / Engram, implementación de pgvector
- [[Obsidian_Como_Fuente_IA]] — Cómo usar este Obsidian como fuente de contexto para agentes

### Proyectos de IA activos
- [[Agente_IA_FastAPI_n8n_PostgreSQL]] — Agente conversacional con memoria persistente
- [[Automatizacion_Correos_IA]] — Clasificación de correos con Claude Haiku
- [[Sistema_Memoria_Empresarial]] — RAG interno con pgvector

### Prompts y uso con Claude Code
- [[Prompts_Para_Claude_Code]] — Prompts listos para crear agentes, revisar arquitectura, etc.
- [[COMO_USAR_ESTE_OBSIDIAN]] — Cómo dar contexto de Obsidian a Claude Code antes de programar

---

## n8n como orquestador de agentes

n8n puede envolver un agente IA completo en un flujo visual:

```
Trigger (webhook / cron)
    ↓
Nodo: Obtener contexto  →  GET /api/memory/{user_id}
    ↓
Nodo: Llamar al agente →  POST /api/ai/chat
    Body: { message, memory, user_id }
    ↓
Nodo: Guardar memoria  →  POST /api/memory/{user_id}
    ↓
Nodo: Enviar respuesta → Slack / Email / Response webhook
```

> Flujos completos: [[n8n_Como_Orquestador]] · [[Flujos_Documentados]]

---

## Relación con otros hubs

```
IA
    ├── usa         →  [[Backend]]      (FastAPI expone los endpoints que las Tools llaman)
    ├── orquestada por → [[n8n_Como_Orquestador]]
    ├── alojada en  →  [[DevOps]]       (misma infra Docker en AWS)
    └── sigue       →  [[Arquitectura]] (LLM decide, Tools ejecutan, DB persiste)
```

---

## Antes de implementar cualquier feature de IA

- [ ] Elegí el modelo correcto — ver [[Modelos_y_Costos]]
- [ ] Definí las Tools que el agente necesita (no más de las necesarias)
- [ ] Cada Tool llama a un endpoint de FastAPI, no a la DB directamente
- [ ] Definí cómo se maneja la memoria (corta/larga duración)
- [ ] El system prompt tiene reglas claras y restricciones explícitas
- [ ] Implementé logging de tokens usados para monitorear costos
