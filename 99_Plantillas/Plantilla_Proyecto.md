# Proyecto: [NOMBRE DEL PROYECTO]

> Fecha de inicio: YYYY-MM-DD
> Estado: [En desarrollo / Activo / Pausado / Completado]
> Responsable: [Nombre]

## Descripción

En una o dos oraciones: qué hace este proyecto y para quién.

## Objetivo

El problema que resuelve y el valor que entrega.

## Stack técnico

| Componente | Tecnología | Versión |
|-----------|-----------|--------|
| Backend | FastAPI | - |
| Base de datos | PostgreSQL | 15 |
| Orquestación | n8n | - |
| Frontend | (si aplica) | - |
| LLM | Claude claude-sonnet-4-6 | - |
| Infraestructura | AWS EC2 | - |

## Arquitectura

```
[Diagrama de arquitectura con texto/ASCII]
```

## Estructura de carpetas

```
nombre_proyecto/
├── app/
│   ├── main.py
│   ├── routers/
│   ├── services/
│   ├── models/
│   └── schemas/
├── .env.example
├── requirements.txt
├── Dockerfile
└── docker-compose.yml
```

## Esquema de base de datos

### Tabla: [nombre_tabla]

| Columna | Tipo | Nullable | Descripción |
|---------|------|----------|-------------|
| id | SERIAL | NO | Clave primaria |
| - | - | - | - |

## URLs y acceso

| Entorno | URL | Notas |
|---------|-----|-------|
| Producción | https://... | - |
| n8n | https://n8n.../workflow/... | - |

## Variables de entorno necesarias

Ver `.env.example` en el repositorio.

## Decisiones de arquitectura

**¿Por qué [DECISIÓN]?**
[RAZÓN]

## Checklist de despliegue

- [ ] Variables de entorno configuradas en producción
- [ ] Migraciones de base de datos ejecutadas
- [ ] Docker compose up -d corriendo
- [ ] Nginx configurado con dominio
- [ ] SSL activo con Let's Encrypt
- [ ] Health check respondiendo
- [ ] n8n flujos activos

## Estado y notas

### YYYY-MM-DD
Descripción del estado actual o cambios recientes.

## Problemas conocidos

- (Listar issues abiertos con links si están en GitHub)

---

*Plantilla — guardar el documento completado en `14_Proyectos/`*
**Ver también:** [[Arquitectura]] · [[Plantilla_Arquitectura]] · [[Checklist_Nueva_App]] · [[INSTRUCCIONES_PARA_AGENTES]]
