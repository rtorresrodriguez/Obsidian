# Estructura ideal de un proyecto FastAPI

## Estructura de carpetas

```
app/
├── main.py           ← Punto de entrada
├── config.py         ← Variables de entorno
├── database.py       ← Conexión a PostgreSQL
├── dependencies.py   ← Dependencias inyectables (get_db, auth)
├── routers/          ← Endpoints HTTP agrupados por dominio
├── services/         ← Lógica de negocio
├── models/           ← Tablas SQLAlchemy
└── schemas/          ← Validación Pydantic
```

---

## main.py

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.routers import users, items, ai
from app.database import engine, Base

# Crear tablas si no existen (en producción usar Alembic)
Base.metadata.create_all(bind=engine)

app = FastAPI(
    title="Mi API",
    description="Descripción de la API",
    version="1.0.0",
)

# CORS - ajustar origins en producción
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Cambiar en producción
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Montar routers
app.include_router(users.router, prefix="/api/users", tags=["users"])
app.include_router(items.router, prefix="/api/items", tags=["items"])
app.include_router(ai.router, prefix="/api/ai", tags=["ai"])

@app.get("/health")
def health_check():
    return {"status": "ok"}
```

---

## config.py

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    secret_key: str
    anthropic_api_key: str
    environment: str = "development"

    class Config:
        env_file = ".env"

settings = Settings()
```

---

## database.py

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from app.config import settings

engine = create_engine(settings.database_url)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

---

## Routers

```python
# app/routers/items.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from app.database import get_db
from app.schemas.item import ItemCreate, ItemResponse
from app.services.item_service import ItemService

router = APIRouter()

@router.post("/", response_model=ItemResponse, status_code=201)
def create_item(item: ItemCreate, db: Session = Depends(get_db)):
    return ItemService.create(db, item)

@router.get("/{item_id}", response_model=ItemResponse)
def get_item(item_id: int, db: Session = Depends(get_db)):
    item = ItemService.get_by_id(db, item_id)
    if not item:
        raise HTTPException(status_code=404, detail="Item no encontrado")
    return item
```

---

## Services

```python
# app/services/item_service.py
from sqlalchemy.orm import Session
from app.models.item import Item
from app.schemas.item import ItemCreate

class ItemService:
    @staticmethod
    def create(db: Session, data: ItemCreate) -> Item:
        item = Item(**data.model_dump())
        db.add(item)
        db.commit()
        db.refresh(item)
        return item

    @staticmethod
    def get_by_id(db: Session, item_id: int) -> Item | None:
        return db.query(Item).filter(Item.id == item_id).first()

    @staticmethod
    def get_all(db: Session, skip: int = 0, limit: int = 100) -> list[Item]:
        return db.query(Item).offset(skip).limit(limit).all()
```

---

## Models (SQLAlchemy)

```python
# app/models/item.py
from sqlalchemy import Column, Integer, String, DateTime, func
from app.database import Base

class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(255), nullable=False)
    description = Column(String(1000))
    created_at = Column(DateTime, server_default=func.now())
    updated_at = Column(DateTime, onupdate=func.now())
```

---

## Schemas (Pydantic)

```python
# app/schemas/item.py
from pydantic import BaseModel
from datetime import datetime

class ItemBase(BaseModel):
    name: str
    description: str | None = None

class ItemCreate(ItemBase):
    pass

class ItemResponse(ItemBase):
    id: int
    created_at: datetime

    class Config:
        from_attributes = True
```

---

## Ejemplo de endpoint completo con LLM

```python
# app/routers/ai.py
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from app.database import get_db
from app.schemas.ai import AIRequest, AIResponse
from app.services.ai_service import AIService

router = APIRouter()

@router.post("/chat", response_model=AIResponse)
async def chat(request: AIRequest, db: Session = Depends(get_db)):
    response = await AIService.chat(db, request.message, request.session_id)
    return AIResponse(response=response)
```

```python
# app/services/ai_service.py
import anthropic
from app.config import settings

client = anthropic.Anthropic(api_key=settings.anthropic_api_key)

class AIService:
    @staticmethod
    async def chat(db, message: str, session_id: str) -> str:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            messages=[{"role": "user", "content": message}]
        )
        return response.content[0].text
```

---

## Buenas prácticas FastAPI

| Práctica | Descripción |
|----------|-------------|
| Siempre usa `response_model` | Garantiza que el schema de salida sea el correcto |
| Usa `status_code` explícito | No dejes que FastAPI asuma el código HTTP |
| Usa `Depends(get_db)` | Nunca crees sesiones de DB dentro del router |
| Usa `HTTPException` | Para errores conocidos con códigos HTTP correctos |
| Usa `async def` donde aplique | Para operaciones I/O (llamadas externas, DB async) |
| Documenta con docstrings cortos | FastAPI los usa en Swagger automáticamente |
| Divide por dominio en routers | Un router por entidad o funcionalidad |
| Valida con Pydantic siempre | Nunca confíes en datos de entrada sin validar |

---

## Ver también

**Hub:** [[Backend]] — Hub central del backend (FastAPI + PostgreSQL + APIs)
- [[Autenticacion_JWT]] — Cómo agregar autenticación JWT a esta estructura
- [[Plantilla_Arquitectura]] — Cómo este proyecto encaja en la arquitectura general
- [[Convenciones]] — Convenciones de PostgreSQL para los modelos
- [[Docker_Compose_Base]] — Cómo dockerizar esta estructura
- [[Checklist_Seguridad]] — Seguridad antes de desplegar
- [[Endpoints_Documentados]] — Cómo documentar los endpoints que crees
- [[Modelos_y_Costos]] — Referencia de modelos Claude para el servicio de IA
- [[Arquitectura_Agente]] — Cómo integrar un agente IA en la capa de servicios
- [[n8n_Como_Orquestador]] — Cómo n8n llama a los endpoints de esta estructura
