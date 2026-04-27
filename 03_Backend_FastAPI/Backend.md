# Hub: Backend

> Este archivo es el hub central del backend. Desde aquí navega a cualquier aspecto de la implementación.
> Hubs del sistema: [[Arquitectura]] · [[DevOps]] · [[IA]] · [[MAPA_MENTAL]]

---

## ¿Qué cubre el backend de este sistema?

El backend abarca tres capas interconectadas:

```
ENTRADA           LÓGICA              DATOS
─────────         ─────────           ─────────
HTTP Request  →   FastAPI         →   PostgreSQL
(validado por     (Router →           (tablas, índices,
 Pydantic)         Service →           relaciones,
                   Model)              backups)

APIS EXTERNAS                     AUTENTICACIÓN
─────────────                     ─────────────
Anthropic API ←── AIService       JWT Bearer Token
Gmail, Stripe ←── HTTPx client    OAuth2 via FastAPI
```

---

## FastAPI: referencia rápida

| Componente | Archivo | Responsabilidad |
|-----------|---------|----------------|
| Router | `app/routers/*.py` | Recibe HTTP, llama al Service |
| Service | `app/services/*.py` | Lógica de negocio |
| Model | `app/models/*.py` | SQLAlchemy ORM, estructura de tablas |
| Schema | `app/schemas/*.py` | Pydantic, validación entrada/salida |
| Database | `app/database.py` | Engine, SessionLocal, get_db |
| Config | `app/config.py` | Pydantic Settings, variables de entorno |

> Estructura completa con código: [[Estructura_Proyecto]]

---

## Base de datos: referencia rápida

| Tema | Nota |
|------|------|
| Convenciones de nombres y tipos | [[Convenciones]] |
| Cómo documentar una tabla nueva | [[Plantilla_Tabla_PostgreSQL]] |
| Índice de tablas actuales | [[Tablas/README\|Tablas documentadas]] |
| Backups y seguridad del puerto 5432 | [[Convenciones]] |

> PostgreSQL se conecta al backend via `DATABASE_URL` en `.env`. El puerto 5432 **nunca** está expuesto al exterior — ver [[Puertos_y_Firewall]].

---

## APIs y autenticación

| Tema | Nota |
|------|------|
| JWT: implementación completa | [[Autenticacion_JWT]] |
| Documentar endpoints del sistema | [[Endpoints_Documentados]] |
| Plantilla de endpoint | [[Plantilla_Endpoint]] |
| Integraciones con APIs externas | [[Integraciones_Externas]] |

---

## Frontend (cuando aplica)

| Tema | Nota |
|------|------|
| Stack frontend recomendado (Next.js) | [[Convenciones_Frontend]] |
| Cuándo NO construir un frontend | [[Herramientas_Sin_Frontend]] |

---

## Sub-notas del backend

### FastAPI
- [[Estructura_Proyecto]] — main.py, routers, services, models, schemas con código real
- [[Autenticacion_JWT]] — JWT completo: hash de passwords, generación y verificación de tokens

### PostgreSQL
- [[Convenciones]] — Nombres, tipos, relaciones, Alembic, backups, seguridad
- [[Tablas/README|Tablas documentadas]] — Índice de todas las tablas del sistema
- [[Plantilla_Tabla_PostgreSQL]] — Plantilla para documentar una tabla nueva

### APIs e integraciones
- [[Endpoints_Documentados]] — Registro de todos los endpoints del sistema
- [[Endpoints_Documentados|Plantilla de endpoint]] — Formato para documentar un endpoint
- [[Integraciones_Externas]] — Anthropic, Gmail y otras APIs externas
- [[Plantilla_Endpoint]] — Plantilla en blanco para un endpoint

### Frontend
- [[Convenciones_Frontend]] — Next.js, Tailwind, conexión con FastAPI, CORS
- [[Herramientas_Sin_Frontend]] — n8n, Retool, Streamlit como alternativas

---

## Relación con otros hubs

```
Backend
    ├── se expone via  →  [[DevOps]]     (Nginx hace el proxy)
    ├── usa            →  [[IA]]          (Claude API, agentes)
    ├── sigue reglas de → [[Arquitectura]] (principios de capas)
    └── es llamado por → [[n8n_Como_Orquestador]] (automatizaciones)
```

---

## Reglas de desarrollo backend (resumen ejecutivo)

1. **Router solo rutea** — Sin lógica de negocio. Solo llama al Service.
2. **Service tiene la lógica** — No llama a la DB directamente. Usa el Model.
3. **Pydantic en cada endpoint** — `request_model` y `response_model` siempre.
4. **Variables de entorno** — Cualquier valor configurable va en `.env`. Ver [[Checklist_Seguridad]].
5. **Documentar endpoints nuevos** — En [[Endpoints_Documentados]] siempre.
6. **Documentar tablas nuevas** — En `05_Base_Datos_PostgreSQL/Tablas/` con [[Plantilla_Tabla_PostgreSQL]].
