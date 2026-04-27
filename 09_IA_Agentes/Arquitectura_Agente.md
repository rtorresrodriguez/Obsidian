# IA y Agentes

## ¿Qué es un agente IA?

Un agente IA es un sistema que:
1. **Percibe** información del entorno (input del usuario, resultado de herramientas, memoria)
2. **Decide** qué acción tomar (usando un LLM como capa de razonamiento)
3. **Actúa** ejecutando herramientas o generando respuestas
4. **Recuerda** el estado entre pasos (memoria)

Un agente no es solo un LLM que responde preguntas. Es un sistema que puede tomar decisiones, ejecutar acciones y aprender del contexto.

---

## Diferencia entre workflow y agente

| Dimensión | Workflow (n8n) | Agente IA |
|-----------|---------------|-----------|
| Control de flujo | Predeterminado (pasos fijos) | Dinámico (el LLM decide) |
| Manejo de casos no previstos | Limitado | Adaptativo |
| Herramientas | Nodos predefinidos | Cualquier función o API |
| Complejidad | Baja a media | Media a alta |
| Transparencia | Alta (flujo visible) | Menor (black box del LLM) |
| Cuándo usar | Procesos repetitivos y predecibles | Tareas con múltiples caminos o incertidumbre |

**Regla práctica:** Si puedes diagramar el flujo completo de antemano, usa n8n. Si el flujo depende del razonamiento del modelo, usa un agente.

---

## Arquitectura de agente simple

```
┌─────────────────────────────────────────────────────┐
│                   AGENTE SIMPLE                      │
│                                                     │
│  Input del usuario                                  │
│       ↓                                             │
│  [Memoria] → Contexto                               │
│       ↓                                             │
│  LLM (Claude) ← System Prompt + Tools               │
│       ↓                                             │
│  ¿Llamar a una herramienta?                         │
│    → SÍ: Ejecutar Tool → Resultado → LLM (continúa) │
│    → NO: Generar respuesta final                    │
│       ↓                                             │
│  Guardar en memoria                                 │
│       ↓                                             │
│  Respuesta al usuario                               │
└─────────────────────────────────────────────────────┘
```

### Implementación básica en Python

```python
import anthropic
from app.config import settings

client = anthropic.Anthropic(api_key=settings.anthropic_api_key)

# Definir herramientas
tools = [
    {
        "name": "get_user_data",
        "description": "Obtiene datos de un usuario de la base de datos",
        "input_schema": {
            "type": "object",
            "properties": {
                "user_id": {"type": "integer", "description": "ID del usuario"}
            },
            "required": ["user_id"]
        }
    },
    {
        "name": "send_email",
        "description": "Envía un correo electrónico",
        "input_schema": {
            "type": "object",
            "properties": {
                "to": {"type": "string"},
                "subject": {"type": "string"},
                "body": {"type": "string"}
            },
            "required": ["to", "subject", "body"]
        }
    }
]

def run_agent(user_message: str, conversation_history: list) -> str:
    messages = conversation_history + [{"role": "user", "content": user_message}]

    while True:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            system="Eres un asistente que ayuda a gestionar usuarios. Usa las herramientas disponibles para responder.",
            tools=tools,
            messages=messages
        )

        # Si el modelo quiere usar una herramienta
        if response.stop_reason == "tool_use":
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    result = execute_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": str(result)
                    })

            # Agregar respuesta del modelo y resultados al historial
            messages.append({"role": "assistant", "content": response.content})
            messages.append({"role": "user", "content": tool_results})

        else:
            # El modelo ha terminado
            return response.content[0].text

def execute_tool(name: str, inputs: dict):
    if name == "get_user_data":
        # Llamar a la base de datos
        return {"id": inputs["user_id"], "name": "Juan", "email": "juan@example.com"}
    elif name == "send_email":
        # Llamar al servicio de email
        return {"status": "sent", "to": inputs["to"]}
    return {"error": "Herramienta no encontrada"}
```

---

## Arquitectura multiagente

```
┌────────────────────────────────────────────────────────────┐
│                   ARQUITECTURA MULTIAGENTE                  │
│                                                            │
│  ┌─────────────────────────────────────────────────────┐  │
│  │            AGENTE ORQUESTADOR (Orchestrator)        │  │
│  │  - Recibe la tarea del usuario                     │  │
│  │  - Decide qué subagente invocar                    │  │
│  │  - Consolida resultados                            │  │
│  └──────────┬────────────────────────────┬────────────┘  │
│             │                            │               │
│     ┌───────▼──────┐            ┌────────▼──────┐       │
│     │ Agente A     │            │ Agente B      │       │
│     │ (Research)   │            │ (Escritura)   │       │
│     │ Busca datos  │            │ Redacta       │       │
│     └──────────────┘            └───────────────┘       │
│                                                          │
│  Memoria compartida (PostgreSQL / Redis)                 │
└────────────────────────────────────────────────────────────┘
```

### Cuándo usar multiagente

- La tarea es demasiado compleja para un solo contexto de LLM
- Quieres especialización (un agente para investigar, otro para redactar)
- Quieres paralelismo (varios agentes trabajando simultáneamente)
- Necesitas verificación cruzada (un agente verifica el trabajo de otro)

---

## LLM como capa de decisión

El LLM **decide**, no ejecuta. Las herramientas (tools) son las que ejecutan acciones reales.

```
Principio:
  LLM → "Quiero hacer X" → Tool execution → resultado → LLM → "Resultado fue Y, ahora haré Z"

No:
  LLM → acción directa en sistema externo
```

El LLM nunca:
- Escribe directamente en la base de datos
- Ejecuta comandos del sistema
- Accede a archivos directamente

Siempre lo hace a través de herramientas que tú defines y controlas.

---

## Herramientas, memoria y ejecución

### Tipos de herramientas (Tools)

| Tipo | Ejemplo | Implementación |
|------|---------|---------------|
| Lectura de datos | `get_user`, `search_docs` | Query a PostgreSQL |
| Escritura de datos | `create_task`, `update_status` | POST a FastAPI |
| Integración externa | `send_email`, `post_slack` | Llamada a API externa |
| Cómputo | `calculate`, `parse_date` | Función Python local |
| Búsqueda RAG | `search_knowledge_base` | Vector search |

### Tipos de memoria

| Tipo | Duración | Implementación |
|------|---------|---------------|
| En contexto | Durante la conversación | Lista de mensajes en la sesión |
| Corto plazo | Entre sesiones del mismo día | Redis o tabla temporal |
| Largo plazo | Permanente | PostgreSQL o RAG |
| Semántica | Búsqueda por similitud | pgvector o Qdrant |

---

## Prompts base para agentes

### System prompt para agente asistente

```
Eres un asistente especializado en [DOMINIO]. Tu objetivo es [OBJETIVO].

Reglas:
1. Usa las herramientas disponibles para obtener información actualizada antes de responder.
2. Si no tienes suficiente información, pregunta al usuario.
3. Nunca inventes datos; usa solo lo que obtienes de las herramientas.
4. Sé conciso y directo en tus respuestas.
5. Cuando ejecutes una acción importante, confírmala con el usuario primero.

Contexto del sistema:
- Stack técnico: FastAPI + PostgreSQL + n8n
- Entorno: Producción en AWS
```

### System prompt para agente de revisión de código

```
Eres un revisor de código experto. Antes de responder cualquier pregunta de desarrollo:
1. Revisa los principios de arquitectura en [ruta/a/obsidian].
2. Verifica que la solución sigue el stack definido (FastAPI, PostgreSQL, Docker).
3. Alerta si el cambio puede afectar producción.
4. Sugiere documentar el cambio en Obsidian si es significativo.
```

---

## Ver también

**Hub:** [[IA]] — Hub central de inteligencia artificial y agentes
- [[Modelos_y_Costos]] — Qué modelo Claude usar según la tarea y el presupuesto
- [[Memoria_y_RAG]] — Cómo dar memoria persistente al agente
- [[n8n_Como_Orquestador]] — Cómo n8n puede orquestar flujos de agentes
- [[Estructura_Proyecto]] — Cómo integrar el agente en un proyecto FastAPI
- [[Endpoints_Documentados]] — Documentar los endpoints que el agente usa como tools
- [[Prompts_Para_Claude_Code]] — Prompt base para crear un agente IA
