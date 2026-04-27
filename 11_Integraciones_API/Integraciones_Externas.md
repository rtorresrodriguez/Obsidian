# Integraciones con APIs externas

Documenta aquí cada integración con servicios externos que use el sistema.

## Anthropic (Claude)

**Propósito:** LLM para agentes IA, clasificación y generación de texto.
**Documentación:** https://docs.anthropic.com
**Credencial:** `ANTHROPIC_API_KEY` en `.env`

```python
import anthropic

client = anthropic.Anthropic(api_key=settings.anthropic_api_key)

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hola"}]
)
text = response.content[0].text
```

**Límites:**
- Rate limit: ver dashboard de Anthropic Console
- Max tokens por request: según el modelo (ver `09_IA_Agentes/Modelos_y_Costos.md`)

---

## Gmail (via n8n)

**Propósito:** Leer y enviar correos electrónicos.
**Configuración:** En n8n → Credentials → Gmail OAuth2
**Requiere:** App Password de Google o OAuth2

**Nodos de n8n:**
- `Gmail Trigger` → Para recibir correos nuevos
- `Gmail` (Send) → Para enviar correos

---

## Plantilla para documentar nueva integración

```markdown
## [Nombre del Servicio]

**Propósito:** Para qué se usa en el sistema.
**Documentación:** URL de la documentación oficial
**Credencial:** Nombre de la variable de entorno en `.env`
**Quién la usa:** FastAPI / n8n / ambos

### Ejemplo de uso

```python
# Código de ejemplo
```

### Errores comunes

| Error | Causa | Solución |
|-------|-------|---------|
| 401 | API key inválida | Regenerar en el dashboard del servicio |
| 429 | Rate limit | Implementar retry con exponential backoff |
```

---

## Cómo implementar retry con exponential backoff

Para APIs externas que pueden fallar por rate limit o problemas temporales:

```python
import time
import httpx

def call_with_retry(func, max_retries=3, base_delay=1):
    for attempt in range(max_retries):
        try:
            return func()
        except httpx.HTTPStatusError as e:
            if e.response.status_code == 429:  # Rate limit
                delay = base_delay * (2 ** attempt)  # 1s, 2s, 4s
                time.sleep(delay)
            elif e.response.status_code >= 500:  # Error del servidor externo
                if attempt == max_retries - 1:
                    raise
                time.sleep(base_delay)
            else:
                raise  # Error del cliente (4xx), no reintentar
    raise Exception("Máximo de reintentos alcanzado")

---

## Ver también

**Hub:** [[Backend]] — Hub central del backend
- [[Endpoints_Documentados]] — Los endpoints internos de FastAPI (no externos)
- [[Checklist_Seguridad]] — Cómo manejar las API keys de servicios externos en `.env`
- [[Estructura_Proyecto]] — Dónde colocar los servicios de integración en la estructura FastAPI
- [[n8n_Como_Orquestador]] — n8n como alternativa para integraciones sin código
```
