# Memoria, RAG y Engram

## Diferencia entre Obsidian, RAG y Engram

| Sistema | Qué es | Para qué sirve | Quién lo usa |
|---------|--------|---------------|-------------|
| **Obsidian** | Base de conocimiento estructurada en Markdown | Documentación técnica, arquitectura, procedimientos | Humanos + agentes IA al inicio de una sesión |
| **RAG** | Sistema de búsqueda semántica sobre documentos | Recuperar fragmentos relevantes de documentación extensa | Agentes IA en tiempo real durante una conversación |
| **Engram** | Memoria conversacional persistente entre sesiones | Recordar preferencias, contexto y decisiones de usuarios | Agentes IA para personalización y continuidad |

## Cuándo usar cada uno

### Usa Obsidian cuando:
- Quieres documentar arquitectura, principios y decisiones de diseño
- El agente necesita contexto inicial antes de comenzar a trabajar
- La información es estable y cambia poco (reglas, convenciones, plantillas)
- Quieres que un humano también pueda leer y editar la información

**Ejemplo:** "Antes de programar, revisa mi Obsidian y sigue mi arquitectura base"

### Usa RAG cuando:
- Tienes cientos de documentos que no caben en el contexto del LLM
- La información cambia frecuentemente
- Necesitas búsqueda semántica ("¿qué documentos hablan de autenticación?")
- Quieres que el agente responda preguntas sobre documentación interna de la empresa

**Ejemplo:** Manual de empleados, base de conocimiento de soporte, documentación de APIs

### Usa Engram (o memoria conversacional) cuando:
- Necesitas que el agente recuerde conversaciones previas con un usuario
- Quieres personalización basada en el historial del usuario
- Necesitas persistencia de preferencias entre sesiones
- Estás construyendo un chatbot o asistente personal

**Ejemplo:** "La última vez me dijiste que prefieres respuestas cortas" → el agente lo recuerda

---

## Cómo preparar Obsidian para que sea fuente de conocimiento para un agente

### Estructura que facilita la lectura por agentes

```
✓ Archivos pequeños y específicos (< 500 líneas)
✓ Un tema por archivo
✓ Títulos descriptivos (el nombre del archivo ya dice de qué trata)
✓ Ejemplos de código en bloques ```language
✓ Listas y tablas para información comparativa
✓ Checklist para procedimientos paso a paso
✗ NO: párrafos largos sin estructura
✗ NO: información de múltiples temas en un archivo
✗ NO: imágenes sin descripción textual alternativa
```

### Prompt para que un agente lea Obsidian

```
Antes de responder cualquier pregunta de desarrollo, lee los siguientes archivos:

1. /ruta/a/obsidian/00_Dashboard/INSTRUCCIONES_PARA_AGENTES.md
2. /ruta/a/obsidian/01_Principios_Arquitectura/Principios.md
3. /ruta/a/obsidian/02_Arquitectura_Base_Aplicaciones/Plantilla_Arquitectura.md

Si la tarea involucra [servicio específico], también lee:
- [ruta al archivo relevante]

Sigue las reglas y arquitectura definidas en esos archivos. Si algo en tu respuesta contradice lo documentado, acláralo.
```

---

## Cómo estructurar conocimiento para que un agente lo lea bien

### Patrón de archivo ideal para agentes

```markdown
# Título claro del tema

## Regla principal (una oración)

## Cuándo aplica

## Cómo implementarlo (con código)

```python
# Ejemplo de código
```

## Cuándo NO aplica / Anti-patrones

## Referencias relacionadas
- [[Otro archivo relacionado]]
```

### Principios de escritura para agentes IA

1. **Sé explícito, no implícito.** "Nunca expongas el puerto 5432" es mejor que "recuerda la seguridad".
2. **Usa tablas para decisiones.** Los agentes leen tablas bien; los párrafos largos generan confusión.
3. **Un archivo = un tema.** No mezcles PostgreSQL con Docker en el mismo archivo.
4. **Usa listas de checklists.** Los agentes pueden seguir pasos numerados de forma fiable.
5. **Incluye ejemplos de código.** El código es más preciso que la descripción en prosa.
6. **Evita ambigüedades.** "Usa el patrón estándar" no dice nada. "Usa el patrón definido en `02_Arquitectura_Base/Plantilla_Arquitectura.md`" es útil.

---

## Implementar RAG básico con pgvector

Si necesitas búsqueda semántica sobre documentos, puedes usar PostgreSQL con la extensión pgvector:

```sql
-- Activar extensión
CREATE EXTENSION IF NOT EXISTS vector;

-- Tabla de documentos con embeddings
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    embedding vector(1536),  -- dimensión para text-embedding-3-small de OpenAI
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Índice para búsqueda eficiente
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops);
```

```python
# Guardar embedding
import anthropic

def get_embedding(text: str) -> list[float]:
    # Anthropic no tiene API de embeddings; usar OpenAI o sentence-transformers
    # Opción con sentence-transformers (local, gratis):
    from sentence_transformers import SentenceTransformer
    model = SentenceTransformer('all-MiniLM-L6-v2')
    return model.encode(text).tolist()

def save_document(db, content: str, metadata: dict):
    embedding = get_embedding(content)
    db.execute(
        "INSERT INTO documents (content, embedding, metadata) VALUES (%s, %s, %s)",
        (content, embedding, metadata)
    )
    db.commit()

def search_similar(db, query: str, limit: int = 5) -> list:
    query_embedding = get_embedding(query)
    results = db.execute(
        """
        SELECT content, metadata, 1 - (embedding <=> %s::vector) AS similarity
        FROM documents
        ORDER BY embedding <=> %s::vector
        LIMIT %s
        """,
        (query_embedding, query_embedding, limit)
    ).fetchall()
    return results
```

---

## Buenas prácticas de documentación para IA

- **Actualiza Obsidian cada vez que cambias la arquitectura.** Los agentes leerán información desactualizada como si fuera verdad.
- **Documenta las decisiones y el por qué.** "Usamos PostgreSQL y no MongoDB porque necesitamos relaciones entre entidades y transacciones ACID."
- **Documenta errores resueltos.** Un agente que sepa que "el error X se resuelve haciendo Y" es mucho más útil.
- **Usa el sufijo `_deprecated` para información obsoleta** en lugar de borrarla. El historial ayuda.
- **Cada proyecto en `14_Proyectos/` debe tener su propia decisión de arquitectura** explicada.

---

## Ver también

**Hub:** [[IA]] — Hub central de inteligencia artificial y memoria
- [[Obsidian_Como_Fuente_IA]] — Cómo integrar este Obsidian con Claude Code y Cursor
- [[Arquitectura_Agente]] — Cómo el agente usa la memoria en el loop de decisión
- [[Convenciones]] — Implementar pgvector en PostgreSQL para RAG
- [[COMO_USAR_ESTE_OBSIDIAN]] — Guía completa de uso de este sistema con agentes IA
- [[Sistema_Memoria_Empresarial]] — Proyecto real que implementa RAG con pgvector
