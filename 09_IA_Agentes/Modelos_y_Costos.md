# Modelos Claude y Consideraciones de Costo

## Modelos disponibles (Anthropic, abril 2026)

| Modelo | ID | Caso de uso | Velocidad | Costo relativo |
|--------|-----|------------|----------|---------------|
| Claude Opus 4.7 | claude-opus-4-7 | Tareas complejas, multiagente | Medio | Alto |
| Claude Sonnet 4.6 | claude-sonnet-4-6 | Balance calidad/costo (recomendado) | Rápido | Medio |
| Claude Haiku 4.5 | claude-haiku-4-5-20251001 | Tareas simples, clasificación | Muy rápido | Bajo |

**Default del sistema:** `claude-sonnet-4-6`

## Cuándo usar cada modelo

### claude-sonnet-4-6 (default)
- Chatbots y agentes conversacionales
- Generación de texto de calidad media-alta
- Análisis de documentos
- Código de complejidad media

### claude-haiku-4-5-20251001
- Clasificación de correos o documentos
- Extracción de datos estructurados de textos cortos
- Respuestas de sí/no o categorías fijas
- Cuando el volumen es alto y el costo importa

### claude-opus-4-7
- Razonamiento complejo multi-paso
- Agentes que toman decisiones críticas
- Análisis profundo de documentos largos
- Cuando la calidad es más importante que el costo

## Configuración en el código

```python
# config.py
class Settings(BaseSettings):
    anthropic_api_key: str
    default_model: str = "claude-sonnet-4-6"
    fast_model: str = "claude-haiku-4-5-20251001"
    smart_model: str = "claude-opus-4-7"
```

```python
# ai_service.py
import anthropic
from app.config import settings

client = anthropic.Anthropic(api_key=settings.anthropic_api_key)

def classify_text(text: str) -> dict:
    """Usar Haiku para clasificación rápida y barata."""
    response = client.messages.create(
        model=settings.fast_model,
        max_tokens=256,
        messages=[{"role": "user", "content": f"Clasifica: {text}"}]
    )
    return response.content[0].text

def analyze_document(document: str) -> str:
    """Usar Sonnet para análisis balanceado."""
    response = client.messages.create(
        model=settings.default_model,
        max_tokens=2048,
        messages=[{"role": "user", "content": document}]
    )
    return response.content[0].text
```

## Cómo monitorear costos

```python
# Guardar tokens_used en cada llamada al LLM
def log_llm_usage(db, model: str, input_tokens: int, output_tokens: int, task: str):
    db.execute(
        """
        INSERT INTO llm_usage_logs (model, input_tokens, output_tokens, task, created_at)
        VALUES (%s, %s, %s, %s, NOW())
        """,
        (model, input_tokens, output_tokens, task)
    )
    db.commit()

# En el service:
response = client.messages.create(...)
log_llm_usage(db, model, response.usage.input_tokens, response.usage.output_tokens, "classify_email")
```

## Límites de contexto

| Modelo | Contexto máximo | Recomendado para prompts |
|--------|----------------|------------------------|
| Opus 4.7 | 200K tokens | Documentos largos |
| Sonnet 4.6 | 200K tokens | Uso general |
| Haiku 4.5 | 200K tokens | Textos cortos |

**Nota:** 1 token ≈ 4 caracteres en inglés, ≈ 3.5 caracteres en español.
Un documento de 10 páginas ≈ 3,000-4,000 tokens.

---

## Ver también

**Hub:** [[IA]] — Hub central de inteligencia artificial
- [[Arquitectura_Agente]] — Cómo usar estos modelos en el loop de un agente
- [[Estructura_Proyecto]] — Configuración de `default_model` en `config.py`
- [[Automatizacion_Correos_IA]] — Ejemplo de uso de Haiku para clasificación masiva
- [[Sistema_Memoria_Empresarial]] — Ejemplo de uso de Sonnet para generación RAG
- [[Prompts_Para_Claude_Code]] — Prompt para crear agentes con el modelo correcto
