# Cuándo no necesitas un frontend

Una de las decisiones más importantes es determinar si realmente necesitas construir un frontend.

## Alternativas a construir un frontend custom

### n8n como interfaz

n8n tiene su propia interfaz web. Para muchos flujos de automatización interna, n8n es suficiente:
- Formularios via webhooks de n8n + notificación al operador
- Dashboards básicos con el nodo HTML de n8n

### Interfaces sin código

| Herramienta | Caso de uso | Integración con FastAPI |
|------------|------------|------------------------|
| Typeform / Tally | Formularios de captura | Webhook → n8n → FastAPI |
| Retool | Dashboards internos | Conecta directo a PostgreSQL o via API |
| Streamlit | Prototipado rápido de IA | Llama a FastAPI |
| Gradio | Demos de modelos IA | Llama a FastAPI |

### Swagger UI de FastAPI (gratis)

FastAPI genera automáticamente una interfaz Swagger en `/docs`. Para uso interno o pruebas, esto puede ser suficiente.

```
https://api.tu-dominio.com/docs    ← Swagger UI
https://api.tu-dominio.com/redoc   ← ReDoc (alternativa)
```

## Decisión: ¿Frontend propio o alternativa?

```
¿El usuario es interno (empleado) o externo (cliente)?
    → Interno → Considera Retool, n8n, o Swagger
    → Externo → Probablemente necesitas frontend propio

¿La interacción es simple (formulario, lista) o compleja (dashboard, mapas, real-time)?
    → Simple → Typeform, Tally, n8n form
    → Compleja → Frontend propio (Next.js)

¿El tiempo al mercado es crítico?
    → SÍ → Empieza con Retool o n8n, migra después si necesitas
    → NO → Construye el frontend adecuado desde el inicio
```

---

## Ver también

**Hub:** [[Backend]] — Hub central del backend
- [[Convenciones_Frontend]] — Cuando sí necesitas construir un frontend propio (Next.js)
- [[n8n_Como_Orquestador]] — n8n como alternativa de interfaz para flujos internos
- [[Cuando_Usar_Cada_Servicio]] — Árbol de decisión más amplio para elegir tecnologías
