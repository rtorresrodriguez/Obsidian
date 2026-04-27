# Proyecto: Sistema interno de memoria empresarial

## Descripción

Base de conocimiento interna de la empresa potenciada con IA. Los empleados pueden hacer preguntas en lenguaje natural y el sistema responde basándose en la documentación interna (manuales, procedimientos, políticas, historial de decisiones). Funciona como un "Google interno con IA".

## Stack técnico

| Componente | Uso |
|-----------|-----|
| FastAPI | API de consulta y gestión del conocimiento |
| PostgreSQL + pgvector | Almacenamiento de documentos y embeddings |
| Claude | Generación de respuestas basadas en contexto (RAG) |
| sentence-transformers | Generación de embeddings (local, sin costo) |
| n8n | Ingesta automática de nuevos documentos |
| Nginx | Reverse proxy con autenticación básica |

## Arquitectura RAG

```
Usuario escribe pregunta
    ↓
FastAPI → generar embedding de la pregunta
    ↓
PostgreSQL (pgvector) → búsqueda semántica
    → Top 5 fragmentos más relevantes
    ↓
Claude:
  System: "Eres el asistente de [EMPRESA]. Responde basándote solo en el contexto."
  Context: [fragmentos recuperados]
  User: [pregunta original]
    ↓
Respuesta con fuentes citadas
    ↓
Log de la consulta (para mejorar el sistema)
```

## Esquema de base de datos

```sql
-- Extensión de vectores
CREATE EXTENSION IF NOT EXISTS vector;

-- Documentos fuente
CREATE TABLE knowledge_documents (
    id SERIAL PRIMARY KEY,
    title VARCHAR(500) NOT NULL,
    source VARCHAR(200),          -- URL, ruta de archivo, etc.
    content TEXT NOT NULL,
    document_type VARCHAR(50),    -- 'manual', 'politica', 'procedimiento'
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    is_active BOOLEAN DEFAULT TRUE
);

-- Chunks (fragmentos) con embeddings
CREATE TABLE knowledge_chunks (
    id SERIAL PRIMARY KEY,
    document_id INTEGER REFERENCES knowledge_documents(id),
    chunk_index INTEGER,
    content TEXT NOT NULL,
    embedding vector(384),        -- dimensión de all-MiniLM-L6-v2
    created_at TIMESTAMP DEFAULT NOW()
);

-- Índice vectorial
CREATE INDEX ON knowledge_chunks USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);

-- Log de consultas
CREATE TABLE query_logs (
    id SERIAL PRIMARY KEY,
    user_id VARCHAR(100),
    query TEXT NOT NULL,
    response TEXT,
    sources_used INTEGER[],       -- IDs de chunks usados
    tokens_used INTEGER,
    response_time_ms INTEGER,
    created_at TIMESTAMP DEFAULT NOW()
);
```

## API endpoints

### POST /api/knowledge/query
Hacer una consulta en lenguaje natural.

### POST /api/knowledge/documents
Agregar un documento nuevo.

### GET /api/knowledge/documents
Listar documentos indexados.

### DELETE /api/knowledge/documents/{id}
Eliminar un documento (y sus chunks).

### POST /api/knowledge/reindex/{id}
Reindexar un documento después de actualizarlo.

## Ingesta de documentos (Pipeline)

```python
# services/ingestion_service.py
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')

def chunk_text(text: str, chunk_size: int = 500, overlap: int = 50) -> list[str]:
    """Dividir texto en fragmentos con overlap."""
    words = text.split()
    chunks = []
    for i in range(0, len(words), chunk_size - overlap):
        chunk = " ".join(words[i:i + chunk_size])
        chunks.append(chunk)
    return chunks

def index_document(db, document_id: int, content: str):
    """Generar embeddings y guardar en la base de datos."""
    chunks = chunk_text(content)
    for i, chunk in enumerate(chunks):
        embedding = model.encode(chunk).tolist()
        db.execute(
            """
            INSERT INTO knowledge_chunks (document_id, chunk_index, content, embedding)
            VALUES (%s, %s, %s, %s::vector)
            """,
            (document_id, i, chunk, embedding)
        )
    db.commit()
```

## Estado del proyecto

- [ ] Setup PostgreSQL con pgvector
- [ ] Pipeline de ingesta de documentos
- [ ] Endpoint de consulta con RAG
- [ ] Interfaz de usuario (simple form HTML o Next.js)
- [ ] Autenticación de usuarios internos
- [ ] n8n para ingesta automática desde Google Drive / Notion
- [ ] Dashboard de uso y métricas
- [ ] Despliegue en AWS

## Decisiones de arquitectura

**¿Por qué sentence-transformers local en lugar de OpenAI embeddings?**
Los documentos internos son confidenciales. Usando un modelo local (all-MiniLM-L6-v2) evitamos enviar texto sensible a APIs externas y eliminamos el costo por embedding.

**¿Por qué pgvector en lugar de Qdrant o Pinecone?**
Ya tenemos PostgreSQL en el servidor. Agregar pgvector evita mantener otro servicio. Para escala mayor (millones de vectores), se puede migrar a Qdrant.

**¿Por qué Claude para la generación y no un modelo local?**
La generación de texto es la parte visible para el usuario y necesita máxima calidad. Claude maneja bien el español y respeta el contexto proporcionado. Los embeddings (parte más costosa en volumen) son locales.

---

## Ver también

- [[Memoria_y_RAG]] — Fundamentos del sistema RAG y cómo implementarlo
- [[Convenciones]] — Configuración de pgvector en PostgreSQL
- [[Estructura_Proyecto]] — Estructura del backend FastAPI de este proyecto
- [[Modelos_y_Costos]] — Por qué Claude Sonnet para generación y modelos locales para embeddings
- [[Plantilla_Proyecto]] — Plantilla base usada para crear este documento
