# Frontend: Convenciones y Decisiones

## Stack frontend recomendado

Para proyectos nuevos en este sistema, el frontend por defecto es:

| Componente | Tecnología | Razón |
|-----------|-----------|-------|
| Framework | Next.js (React) | SSR, rutas, fullstack si se necesita |
| Estilos | Tailwind CSS | Utilidades, rápido de usar |
| Estado global | Zustand o React Query | Ligero, sin boilerplate |
| Formularios | React Hook Form + Zod | Validación con schemas |
| HTTP client | fetch nativo o axios | Preferir fetch en Next.js |
| Autenticación | NextAuth.js o JWT propio | Según la complejidad |

## Cuándo construir un frontend

Construye un frontend cuando:
- El usuario final es una persona (no un sistema)
- La interfaz necesita formularios, dashboards o visualizaciones
- n8n no es suficiente para la interacción con el usuario

**No construyas un frontend** si:
- El "cliente" es otro servicio o un agente IA
- n8n puede manejar el flujo completo
- Solo se necesita un formulario simple (considera Typeform, Tally, o forms de n8n)

## Estructura de carpetas (Next.js)

```
frontend/
├── app/                    ← App Router (Next.js 13+)
│   ├── layout.tsx
│   ├── page.tsx
│   └── api/               ← API Routes si se necesitan
├── components/
│   ├── ui/                ← Componentes base (Button, Input, Card)
│   └── features/          ← Componentes de dominio
├── lib/
│   ├── api.ts             ← Funciones para llamar al backend FastAPI
│   └── utils.ts
├── hooks/                 ← Custom hooks
├── types/                 ← TypeScript types
├── .env.local             ← Variables de entorno frontend
├── next.config.js
├── tailwind.config.js
└── package.json
```

## Variables de entorno frontend

```bash
# .env.local (NUNCA subir a git)
NEXT_PUBLIC_API_URL=https://api.tu-dominio.com
NEXT_PUBLIC_APP_NAME=Mi App

# Variables sin NEXT_PUBLIC son solo server-side
API_SECRET_KEY=...
```

## Cómo conectar el frontend con FastAPI

```typescript
// lib/api.ts
const API_URL = process.env.NEXT_PUBLIC_API_URL;

export async function fetchItems() {
  const res = await fetch(`${API_URL}/api/items`);
  if (!res.ok) throw new Error("Error al obtener items");
  return res.json();
}

export async function createItem(data: { name: string; description?: string }) {
  const res = await fetch(`${API_URL}/api/items`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(data),
  });
  if (!res.ok) throw new Error("Error al crear item");
  return res.json();
}
```

## CORS en FastAPI para el frontend

```python
# En main.py, ajustar origins según el dominio del frontend
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://tu-dominio-frontend.com",
        "http://localhost:3000",  # solo para desarrollo
    ],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["*"],
)
```

## Buenas prácticas

- Nunca pongas lógica de negocio en el frontend; ponla en FastAPI
- Nunca conectes el frontend directamente a PostgreSQL
- Siempre valida en el frontend (UX) Y en el backend (seguridad)
- Usa variables de entorno para la URL del backend, nunca la hardcodees
- El frontend solo consume la API; la API nunca devuelve datos sensibles innecesarios

---

## Ver también

**Hub:** [[Backend]] — Hub central del backend (FastAPI + PostgreSQL + APIs)
- [[Herramientas_Sin_Frontend]] — Cuándo usar n8n, Retool o Streamlit en vez de frontend propio
- [[Estructura_Proyecto]] — La API de FastAPI que este frontend consume
- [[Autenticacion_JWT]] — Cómo manejar el Bearer Token desde el frontend
- [[Endpoints_Documentados]] — Los endpoints disponibles para llamar desde el frontend
- [[Checklist_Seguridad]] — Variables de entorno del frontend y CORS
