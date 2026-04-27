# Cuándo usar cada servicio

Guía de referencia rápida para tomar decisiones de arquitectura.

## Árbol de decisión

```
¿Necesito exponer datos o lógica via HTTP?
    → SÍ → FastAPI
    → NO → ¿Necesito conectar servicios o automatizar flujos?
                → SÍ → n8n
                → NO → ¿Necesito persistir datos relacionales?
                            → SÍ → PostgreSQL
                            → NO → Script Python directo
```

## Tabla de decisión rápida

| Necesidad | Solución recomendada |
|-----------|---------------------|
| API REST con validación | FastAPI |
| Automatización periódica (cron) | n8n |
| Webhook de servicio externo | n8n → FastAPI |
| Almacenamiento de datos | PostgreSQL |
| Caché de respuestas | Redis (si está disponible) o dict en memoria |
| Reverse proxy + SSL | Nginx + Certbot |
| Aislamiento de servicios | Docker |
| Orquestación de contenedores simple | Docker Compose |
| LLM para generar texto | Anthropic API (Claude) |
| Búsqueda semántica en documentos | pgvector o Qdrant |
| Memoria conversacional entre sesiones | Engram o tabla en PostgreSQL |
| Documentación de APIs | FastAPI Swagger automático + este Obsidian |

## Combinaciones comunes probadas

### Patrón 1: API interna

```
Cliente → FastAPI → PostgreSQL
```
Usar cuando: Necesitas una API simple de CRUD.

### Patrón 2: API con orquestación

```
Evento externo → n8n (webhook) → FastAPI → PostgreSQL
```
Usar cuando: Un evento externo (Stripe, WhatsApp, correo) debe disparar lógica interna.

### Patrón 3: Agente IA

```
Usuario → FastAPI → [LLM + Tools] → PostgreSQL
                  ↑
                n8n (memoria / context retrieval)
```
Usar cuando: Necesitas un chatbot o agente que tome decisiones y persista resultados.

### Patrón 4: Automatización completa

```
Trigger (cron/evento) → n8n → FastAPI → PostgreSQL
                                      → LLM
                                      → Email/WhatsApp
```
Usar cuando: Procesos automáticos sin intervención humana.

## Anti-patrones a evitar

| Anti-patrón | Consecuencia | Alternativa |
|-------------|-------------|-------------|
| n8n con queries SQL directas a producción | Riesgo de datos inconsistentes, sin validación | n8n → FastAPI endpoint |
| LLM escribiendo directamente en DB | Sin validación, riesgo de inyección | LLM → Tool → FastAPI → DB |
| Todo en un solo archivo main.py | Imantenible, difícil de testear | Separar en routers/services/models |
| Secrets en variables de entorno sin `.env.example` | Credenciales olvidadas al desplegar | Siempre mantener `.env.example` actualizado |
| Docker para scripts de una vez | Overhead innecesario | Script Python directo con venv |

---

## Ver también

**Hub:** [[Arquitectura]] — Hub central de arquitectura y principios
- [[Principios]] — El razonamiento detrás de estas decisiones
- [[Plantilla_Arquitectura]] — Cómo combinar estos servicios en una app real
- [[Docker_Compose_Base]] — Referencia de configuración de Docker
- [[Estructura_Proyecto]] — Estructura interna de una app FastAPI
- [[n8n_Como_Orquestador]] — Cuándo y cómo usar n8n con FastAPI
- [[Convenciones]] — Convenciones de PostgreSQL
