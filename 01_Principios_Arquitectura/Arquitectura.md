# Hub: Arquitectura

> Este archivo es el hub central de arquitectura. Desde aquí navega a cualquier decisión de diseño del sistema.
> Hubs del sistema: [[Backend]] · [[DevOps]] · [[IA]] · [[MAPA_MENTAL]]

---

## ¿Qué define la arquitectura de este sistema?

El sistema sigue tres principios no negociables:

**1. Separación de capas**
Cada servicio tiene una responsabilidad. FastAPI valida y rutea, los Services tienen la lógica, los Models acceden a datos, PostgreSQL solo almacena. Nadie invade la capa de otro.

**2. Acceso externo solo por Nginx**
Los servicios internos (FastAPI, n8n, PostgreSQL) nunca están expuestos directamente a internet. Todo tráfico entra por Nginx en los puertos 80/443. El resto está en la red privada de Docker.

**3. Configuración en variables de entorno**
Cero credenciales en código. Cero valores hardcodeados. Todo en `.env`, nunca en git.

---

## Stack por defecto

| Capa | Tecnología | Cuándo cambiar |
|------|-----------|---------------|
| API Backend | FastAPI (Python) | Solo si el equipo no usa Python |
| Base de datos | PostgreSQL | Agregar Redis para caché si escala |
| Orquestación | n8n | Agregar Celery si hay colas complejas |
| Reverse proxy | Nginx | Solo si se migra a un Load Balancer |
| Contenedores | Docker + Compose | Solo si se migra a Kubernetes |
| SSL | Let's Encrypt + Certbot | Si se requiere wildcard cert |
| Infraestructura | AWS EC2 | Solo si se requiere serverless |
| LLM | Claude Sonnet 4.6 | Ver [[Modelos_y_Costos]] |

---

## Cuándo usar qué

```
¿Necesito exponer lógica via HTTP?           → FastAPI
¿Necesito automatizar flujos o webhooks?      → n8n
¿Necesito persistir datos relacionales?       → PostgreSQL
¿Necesito búsqueda semántica?                 → pgvector + embeddings
¿Necesito memoria conversacional?             → PostgreSQL o Engram
¿El servicio corre continuamente?             → Docker Compose
¿Necesito dominio propio con HTTPS?           → Nginx + Certbot
¿Necesito generar texto o razonar?            → Claude (Anthropic API)
```

> Guía completa: [[Cuando_Usar_Cada_Servicio]]

---

## Sub-notas de arquitectura

### Principios y decisiones
- [[Principios]] — Separación de capas, capas del sistema, buenas prácticas y anti-patrones
- [[Cuando_Usar_Cada_Servicio]] — Árbol de decisión: FastAPI, n8n, PostgreSQL, Docker, RAG
- [[Mapa_del_Sistema]] — Diagrama de infraestructura AWS completo
- [[MAPA_MENTAL]] — Vista lógica de todos los flujos de información

### Plantillas y checklists
- [[Plantilla_Arquitectura]] — Template de arquitectura FastAPI + PostgreSQL + n8n + LLM
- [[Checklist_Nueva_App]] — Lista completa antes de escribir la primera línea de código
- [[Checklist_Pre_Produccion]] — Lista antes de cualquier cambio en producción

### Documentos de apoyo
- [[INSTRUCCIONES_PARA_AGENTES]] — Reglas que los agentes IA deben seguir
- [[COMO_USAR_ESTE_OBSIDIAN]] — Cómo integrar este sistema con Claude Code y Cursor

---

## Relación con otros hubs

```
Arquitectura
    ├── Backend      → [[Backend]]    (FastAPI, PostgreSQL, APIs)
    ├── DevOps       → [[DevOps]]     (Docker, Nginx, Seguridad)
    └── IA           → [[IA]]         (Agentes, LLMs, Memoria)
```

La arquitectura define las reglas. Los otros hubs implementan esas reglas en sus dominios.

---

## Anti-patrones arquitectónicos a evitar

| Anti-patrón | Qué usar en su lugar |
|-------------|---------------------|
| Lógica de negocio en el router | Moverla al Service |
| n8n conectado directo a PostgreSQL | n8n → FastAPI → PostgreSQL |
| LLM escribiendo en la DB directamente | LLM → Tool → FastAPI → DB |
| Credenciales en el código fuente | Variables de entorno en `.env` |
| Puerto 5432 o 8000 expuesto al exterior | Solo Nginx expone puertos |
| Un `main.py` de 500 líneas | Separar en routers/services/models |

> Ver todos los anti-patrones con ejemplos en [[Principios]]
