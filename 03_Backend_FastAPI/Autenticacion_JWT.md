# Autenticación JWT en FastAPI

## Flujo de autenticación

```
Cliente → POST /api/auth/login { email, password }
              ↓
         FastAPI verifica credenciales en PostgreSQL
              ↓
         Genera JWT token (firmado con SECRET_KEY)
              ↓
         Devuelve { access_token, token_type: "bearer" }
              ↓
Cliente → GET /api/users/me
  Header: Authorization: Bearer <token>
              ↓
         FastAPI verifica y decodifica el token
              ↓
         Devuelve datos del usuario autenticado
```

## Implementación

### Dependencias

```bash
pip install python-jose[cryptography] passlib[bcrypt]
```

```
# requirements.txt
fastapi>=0.110.0
python-jose[cryptography]>=3.3.0
passlib[bcrypt]>=1.7.4
```

### auth.py

```python
# app/auth.py
from datetime import datetime, timedelta
from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from app.config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/auth/login")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: timedelta = timedelta(hours=24)) -> str:
    to_encode = data.copy()
    to_encode["exp"] = datetime.utcnow() + expires_delta
    return jwt.encode(to_encode, settings.secret_key, algorithm="HS256")

async def get_current_user(token: str = Depends(oauth2_scheme), db = Depends(get_db)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Token inválido o expirado",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, settings.secret_key, algorithms=["HS256"])
        user_id: int = payload.get("sub")
        if user_id is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    
    user = db.query(User).filter(User.id == user_id).first()
    if user is None:
        raise credentials_exception
    return user
```

### Router de autenticación

```python
# app/routers/auth.py
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm
from sqlalchemy.orm import Session
from app.database import get_db
from app.auth import verify_password, create_access_token
from app.models.user import User

router = APIRouter()

@router.post("/login")
def login(form_data: OAuth2PasswordRequestForm = Depends(), db: Session = Depends(get_db)):
    user = db.query(User).filter(User.email == form_data.username).first()
    if not user or not verify_password(form_data.password, user.hashed_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Email o contraseña incorrectos"
        )
    token = create_access_token(data={"sub": str(user.id)})
    return {"access_token": token, "token_type": "bearer"}
```

### Usar en endpoints protegidos

```python
from app.auth import get_current_user
from app.models.user import User

@router.get("/me", response_model=UserResponse)
def get_me(current_user: User = Depends(get_current_user)):
    return current_user

@router.get("/admin-only")
def admin_route(current_user: User = Depends(get_current_user)):
    if not current_user.is_admin:
        raise HTTPException(status_code=403, detail="No autorizado")
    return {"message": "Acceso admin"}
```

## Variables de entorno requeridas

```bash
SECRET_KEY=CLAVE_ALEATORIA_DE_32_CARACTERES_MINIMO
ACCESS_TOKEN_EXPIRE_HOURS=24
```

---

## Ver también

**Hub:** [[Backend]] — Hub central del backend (FastAPI + PostgreSQL + APIs)
- [[Estructura_Proyecto]] — Cómo integrar la autenticación en la estructura de proyecto FastAPI
- [[Checklist_Seguridad]] — Cómo manejar el SECRET_KEY de forma segura en .env
- [[Endpoints_Documentados]] — Cómo documentar los endpoints de auth (/login, /me)
