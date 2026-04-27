# Instrucciones para Agentes IA

> Este archivo es el punto de entrada obligatorio para cualquier agente IA, Claude Code o Cursor que opere sobre proyectos de este sistema.
> Hubs del sistema: [[Arquitectura]] · [[Backend]] · [[DevOps]] · [[IA]] · [[MAPA_MENTAL]]

---

## 1. Carpetas que debes revisar primero (en orden)

Antes de escribir una sola línea de código o ejecutar cualquier comando, lee estos archivos en este orden:

1. [[README]] — Estado del sistema y servicios activos
2. [[Principios]] — Reglas de diseño y separación de responsabilidades
3. [[Cuando_Usar_Cada_Servicio]] — Árbol de decisión de tecnologías
4. [[Plantilla_Arquitectura]] — Arquitectura por defecto para apps nuevas
5. [[Checklist_Seguridad]] — Siempre antes de tocar producción

Si la tarea involucra algún servicio específico, también lee:
- Backend: [[Estructura_Proyecto]]
- Base de datos: [[Convenciones]]
- Docker: [[Docker_Compose_Base]]
- Nginx/SSL: [[Reverse_Proxy]]
- n8n: [[n8n_Como_Orquestador]]
- Agentes IA: [[Arquitectura_Agente]]

---

## 2. Reglas que debes seguir antes de programar

### Reglas absolutas (no negociables)

- **NUNCA** hardcodees credenciales, contraseñas o API keys en el código.
- **NUNCA** expongas puertos sin validar con `12_Seguridad/Puertos_y_Firewall.md`.
- **NUNCA** modifiques `docker-compose.yml` de producción sin confirmación explícita del usuario.
- **NUNCA** hagas `git push` a `main` sin preguntar primero.
- **NUNCA** borres volúmenes de Docker sin confirmación. Contienen datos de producción.
- **NUNCA** reinicies Nginx o PostgreSQL en producción sin avisar.
- **NUNCA** instales dependencias globales en el servidor sin confirmación.

### Reglas de desarrollo

- Siempre usa variables de entorno via `.env`. Nunca valores directos en el código.
- Siempre crea endpoints bajo un router, no directamente en `main.py`.
- Siempre valida inputs con Pydantic antes de procesar.
- Siempre usa transacciones cuando escribas múltiples tablas en PostgreSQL.
- Siempre usa `docker compose` (v2) en lugar de `docker-compose` (v1 deprecated).
- Siempre documenta cualquier endpoint nuevo en [[Endpoints_Documentados]].
- Si creas una tabla nueva en PostgreSQL, documéntala en `05_Base_Datos_PostgreSQL/Tablas/` usando [[Plantilla_Tabla_PostgreSQL]].

---

## 3. Arquitectura por defecto

Cuando el usuario pida crear una nueva aplicación o servicio, usa esta arquitectura por defecto:

```
[Cliente / n8n / Frontend]
        ↓
[Nginx] → reverse proxy con SSL
        ↓
[FastAPI] → lógica de negocio, rutas, validación
        ↓
[PostgreSQL] → persistencia de datos
        ↑
[n8n] → orquestación de flujos y automatización
        ↑
[LLM / Agente IA] → capa de decisión y generación
```

### Stack por defecto

| Capa | Tecnología |
|------|-----------|
| Backend | FastAPI (Python) |
| Base de datos | PostgreSQL |
| Orquestación | n8n |
| Reverse proxy | Nginx |
| Contenedores | Docker + Docker Compose |
| SSL | Let's Encrypt + Certbot |
| Infraestructura | AWS EC2 |
| LLM | Claude (Anthropic API) |
| Memoria/RAG | Según el proyecto (ver [[Memoria_y_RAG]]) |

### Estructura de carpetas para un proyecto Python/FastAPI

```
mi_proyecto/
├── app/
│   ├── main.py
│   ├── routers/
│   ├── services/
│   ├── models/
│   ├── schemas/
│   └── database.py
├── .env
├── .env.example
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
└── README.md
```

---

## 4. Cosas que NO debes tocar sin confirmación explícita

| Acción | Razón |
|--------|-------|
| Modificar `docker-compose.yml` de producción | Puede bajar todos los servicios |
| Reiniciar PostgreSQL | Puede interrumpir conexiones activas |
| Cambiar configuración de Nginx | Puede dejar el dominio inaccesible |
| Renovar/revocar certificados SSL | Afecta acceso HTTPS de todos los dominios |
| Borrar volúmenes Docker | Pérdida irreversible de datos |
| Cambiar variables de entorno en producción | Puede romper servicios dependientes |
| Hacer `DROP TABLE` o `ALTER TABLE` en producción | Riesgo de pérdida de datos |
| Modificar crons de n8n activos | Puede interrumpir automatizaciones críticas |
| Hacer push a rama `main` | Puede disparar despliegue automático |

---

## 5. Cómo documentar cualquier cambio nuevo

Después de cualquier cambio significativo, documenta siguiendo esta guía:

### Si creaste un endpoint nuevo
→ Agrega entrada en [[Endpoints_Documentados]]

### Si creaste una tabla nueva en PostgreSQL
→ Agrega esquema en `05_Base_Datos_PostgreSQL/Tablas/` usando la plantilla [[Plantilla_Tabla_PostgreSQL]]

### Si modificaste docker-compose
→ Anota el cambio en [[Registro_Cambios_Docker]]

### Si encontraste y resolviste un error
→ Documenta en [[Errores_Comunes]] usando la plantilla [[Plantilla_Error]]

### Si creaste un proyecto nuevo
→ Crea un archivo en `14_Proyectos/` usando la plantilla [[Plantilla_Proyecto]]

### Si configuraste Nginx o un dominio nuevo
→ Documenta en [[Dominios_Activos]]

### Si agregaste una integración con API externa
→ Documenta en [[Integraciones_Externas]]

---

## Nota final para el agente

Este sistema de documentación es la fuente de verdad. Si algo en el código contradice lo documentado aquí, **pregunta al usuario** cuál es correcto antes de proceder. No asumas. No inventes arquitecturas. Si tienes dudas, revisa [[Principios]] y [[Cuando_Usar_Cada_Servicio]], y luego pregunta.
