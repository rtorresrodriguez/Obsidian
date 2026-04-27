# Flujos de n8n documentados

Documenta aquí cada flujo activo en n8n. Usa el formato estándar.

## Formato de documentación

```markdown
## [NOMBRE DEL FLUJO]

**ID en n8n:** (copiar del URL de n8n)
**Estado:** Activo / Pausado / Desarrollo
**Trigger:** Webhook / Schedule (cron) / Email / Manual
**Descripción:** Qué hace en una oración

**Flujo:**
Paso 1 → Paso 2 → Paso 3 → ...

**Endpoints de FastAPI que llama:**
- POST /api/...
- GET /api/...

**Servicios externos que usa:**
- Gmail, Slack, Stripe, etc.

**Variables de entorno que necesita:**
- VARIABLE_1
- VARIABLE_2

**Última modificación:** YYYY-MM-DD
**Notas:**
Cualquier comportamiento especial o limitación.
```

---

## Flujos activos

*(Agregar entradas aquí según se creen los flujos en n8n)*

---

## Checklist antes de activar un flujo en producción

- [ ] El flujo fue probado con datos reales en un entorno de desarrollo
- [ ] Las credenciales de n8n están configuradas (no hardcodeadas en los nodos)
- [ ] El flujo tiene manejo de errores (nodo "Error Trigger" o "IF" para verificar status codes)
- [ ] Si el flujo llama a FastAPI, los endpoints existen y responden correctamente
- [ ] Si el flujo tiene un webhook, la URL fue configurada en el servicio externo
- [ ] El flujo está documentado en este archivo

---

## Ver también

**Hub:** [[DevOps]] — Hub central de DevOps (Docker, Nginx, despliegue)
- [[n8n_Como_Orquestador]] — Patrones de conexión n8n ↔ FastAPI ↔ PostgreSQL con ejemplos
- [[Endpoints_Documentados]] — Los endpoints de FastAPI que los flujos pueden llamar
- [[IA]] — Hub de IA: cómo n8n puede orquestar agentes Claude
- [[Automatizacion_Correos_IA]] — Ejemplo real de flujo de n8n con IA
- [[Agente_IA_FastAPI_n8n_PostgreSQL]] — Ejemplo real de agente orquestado por n8n
