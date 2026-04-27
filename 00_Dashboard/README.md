# Sistema de Memoria Técnica y Arquitectónica

> **Para agentes IA:** Lee primero [[INSTRUCCIONES_PARA_AGENTES]] antes de cualquier acción.
> **Para humanos:** Lee [[COMO_USAR_ESTE_OBSIDIAN]] para saber cómo aprovechar este sistema.
> **Mapa visual del sistema:** [[MAPA_MENTAL]]

---

## Hubs de navegación rápida

| Hub | Cubre | Notas principales |
|-----|-------|------------------|
| [[Arquitectura]] | Principios, plantillas, checklists | [[Principios]] · [[Plantilla_Arquitectura]] · [[Cuando_Usar_Cada_Servicio]] |
| [[Backend]] | FastAPI, PostgreSQL, APIs, Auth | [[Estructura_Proyecto]] · [[Convenciones]] · [[Endpoints_Documentados]] |
| [[DevOps]] | Docker, Nginx, Seguridad, Despliegue | [[Docker_Compose_Base]] · [[Reverse_Proxy]] · [[Checklist_Seguridad]] |
| [[IA]] | Agentes, LLMs, Memoria, RAG | [[Arquitectura_Agente]] · [[Modelos_y_Costos]] · [[Memoria_y_RAG]] |

---

## Índice de navegación

### Arquitectura y principios

| Nota | Qué encontrarás |
|------|----------------|
| [[Principios]] | Separación de responsabilidades, capas, anti-patrones |
| [[Cuando_Usar_Cada_Servicio]] | Árbol de decisión: FastAPI vs n8n vs PostgreSQL vs Docker |
| [[Plantilla_Arquitectura]] | Diagrama y flujo de datos para apps nuevas |
| [[Checklist_Nueva_App]] | Lista completa antes de escribir código |
| [[Checklist_Pre_Produccion]] | Lista antes de tocar el servidor |
| [[Mapa_del_Sistema]] | Diagrama de toda la infraestructura AWS |

### FastAPI / Backend

| Nota | Qué encontrarás |
|------|----------------|
| [[Estructura_Proyecto]] | Carpetas, main.py, routers, services, models, schemas |
| [[Autenticacion_JWT]] | Implementación completa de JWT con FastAPI |

### Docker y DevOps

| Nota | Qué encontrarás |
|------|----------------|
| [[Docker_Compose_Base]] | docker-compose completo, Dockerfile, comandos útiles |
| [[Registro_Cambios_Docker]] | Historial de cambios en Docker en producción |

### PostgreSQL

| Nota | Qué encontrarás |
|------|----------------|
| [[Convenciones]] | Nombres, tipos, relaciones, backups, seguridad 5432 |
| [[Tablas/README\|Tablas documentadas]] | Índice de tablas del sistema |

### Nginx, SSL y Dominios

| Nota | Qué encontrarás |
|------|----------------|
| [[Reverse_Proxy]] | Config Nginx, Certbot, Let's Encrypt, checklist SSL |
| [[Dominios_Activos]] | Dominios en producción, cómo agregar uno nuevo |

### n8n y Automatización

| Nota | Qué encontrarás |
|------|----------------|
| [[n8n_Como_Orquestador]] | Cómo conectar n8n con FastAPI y PostgreSQL, ejemplos |
| [[Flujos_Documentados]] | Registro de flujos activos en n8n |

### IA y Agentes

| Nota | Qué encontrarás |
|------|----------------|
| [[Arquitectura_Agente]] | Agente simple, multiagente, tools, memoria |
| [[Modelos_y_Costos]] | Modelos Claude disponibles y cuándo usar cada uno |
| [[Memoria_y_RAG]] | Diferencias Obsidian / RAG / Engram, implementación |
| [[Obsidian_Como_Fuente_IA]] | Cómo usar este Obsidian con Claude Code y agentes |

### Seguridad

| Nota | Qué encontrarás |
|------|----------------|
| [[Checklist_Seguridad]] | Variables de entorno, .gitignore, qué nunca subir a git |
| [[Puertos_y_Firewall]] | Qué puertos abrir, UFW, AWS Security Group |

### Errores y Soluciones

| Nota | Qué encontrarás |
|------|----------------|
| [[Errores_Comunes]] | DNS, Nginx, Docker, PostgreSQL, Certbot, SSH, Git |

### Integraciones API

| Nota | Qué encontrarás |
|------|----------------|
| [[Endpoints_Documentados]] | Formato y registro de endpoints del sistema |
| [[Integraciones_Externas]] | Anthropic, Gmail y plantilla para nuevas integraciones |

### Frontend

| Nota | Qué encontrarás |
|------|----------------|
| [[Convenciones_Frontend]] | Next.js, Tailwind, conexión con FastAPI, CORS |
| [[Herramientas_Sin_Frontend]] | Cuándo usar n8n, Retool o Streamlit en vez de frontend |

### Proyectos activos

| Nota | Qué encontrarás |
|------|----------------|
| [[Agente_IA_FastAPI_n8n_PostgreSQL]] | Agente conversacional con memoria |
| [[Automatizacion_Correos_IA]] | Pipeline de clasificación de correos con Claude |
| [[Sistema_Memoria_Empresarial]] | RAG interno con pgvector |

### Prompts base para agentes

| Nota | Qué encontrarás |
|------|----------------|
| [[Prompts_Para_Claude_Code]] | 7 prompts listos para iniciar sesiones con Claude Code |

### Plantillas en blanco

| Nota | Qué encontrarás |
|------|----------------|
| [[Plantilla_Proyecto]] | Plantilla para documentar un nuevo proyecto |
| [[Plantilla_Error]] | Plantilla para registrar un error resuelto |
| [[Plantilla_Endpoint]] | Plantilla para documentar un endpoint de API |
| [[Plantilla_Tabla_PostgreSQL]] | Plantilla para documentar una tabla de la base de datos |

---

## Estado del servidor AWS

| Parámetro | Valor |
|-----------|-------|
| Proveedor | AWS EC2 |
| Tipo de instancia | (completar: t3.medium / t3.large) |
| Sistema operativo | Ubuntu 22.04 LTS |
| IP Elástica | (completar) |
| Región | (completar: us-east-1 / sa-east-1) |
| Acceso SSH | Via clave .pem |

## Servicios en producción

| Servicio | Puerto | Estado | Notas |
|----------|--------|--------|-------|
| FastAPI | 8000 | Activo | Expuesto via Nginx |
| PostgreSQL | 5432 | Activo | Solo red interna Docker |
| n8n | 5678 | Activo | Expuesto via Nginx |
| Nginx | 80/443 | Activo | Único punto de entrada externo |
| Docker | — | Activo | Motor de todos los servicios |

## Dominios activos

| Dominio | Servicio | SSL |
|---------|---------|-----|
| (completar) | FastAPI via Nginx | Let's Encrypt |
| (completar) | n8n via Nginx | Let's Encrypt |

## Reglas de oro

1. No tocar producción sin revisar [[Checklist_Pre_Produccion]]
2. No crear servicios sin revisar [[Cuando_Usar_Cada_Servicio]]
3. No exponer puertos sin revisar [[Puertos_y_Firewall]]
4. Todo cambio nuevo debe documentarse — ver [[INSTRUCCIONES_PARA_AGENTES]]

## Última actualización

- Fecha: (completar al actualizar)
- Quién actualizó: (completar)
- Qué cambió: (completar)
