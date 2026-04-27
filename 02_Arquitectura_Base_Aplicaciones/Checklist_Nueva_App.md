# Checklist antes de crear una nueva aplicación

Completa esta lista antes de escribir código.

## Definición del proyecto

- [ ] Tengo claro el objetivo de la aplicación en una oración
- [ ] Sé quién va a usarla (usuario final, otro servicio, agente IA)
- [ ] Sé qué datos necesita guardar
- [ ] Sé qué datos necesita exponer
- [ ] Sé si necesita autenticación
- [ ] Sé si necesita integrarse con servicios externos

## Decisiones de arquitectura

- [ ] Revisé [[Cuando_Usar_Cada_Servicio]]
- [ ] Confirmé que FastAPI es la capa correcta para este proyecto
- [ ] Decidí si necesita n8n (automatización) o no
- [ ] Decidí si necesita LLM y cuál modelo usar
- [ ] Decidí el esquema de base de datos (tablas, relaciones)
- [ ] Decidí si necesita Docker (casi siempre sí para servicios persistentes)
- [ ] Decidí si necesita Nginx (si va a estar en producción con dominio)

## Seguridad antes de empezar

- [ ] Tengo un `.env.example` con todas las variables necesarias (sin valores reales)
- [ ] Verifiqué que `.env` está en `.gitignore`
- [ ] Sé cuáles puertos va a usar y confirmé que no colisionan
- [ ] Si es en producción, confirmé que el puerto no está expuesto al exterior sin Nginx

## Setup del proyecto

- [ ] Creé la estructura de carpetas siguiendo [[Plantilla_Arquitectura]]
- [ ] Inicialicé el repositorio git
- [ ] Creé `requirements.txt` o `pyproject.toml`
- [ ] Creé `Dockerfile`
- [ ] Creé `docker-compose.yml`
- [ ] Creé el README del proyecto

## Documentación

- [ ] Creé el archivo del proyecto en `14_Proyectos/` usando [[Plantilla_Proyecto]]
- [ ] Documenté el objetivo, stack y arquitectura del proyecto

---

# Checklist antes de tocar producción

**Alto riesgo. No omitas ningún paso.**

## Evaluación previa

- [ ] Leí [[INSTRUCCIONES_PARA_AGENTES]]
- [ ] El cambio está probado en local o en entorno de desarrollo
- [ ] Sé exactamente qué archivos o servicios voy a modificar
- [ ] Tengo un plan de rollback si algo falla

## Backups

- [ ] Hice backup de la base de datos PostgreSQL antes de migrar
- [ ] Sé dónde está el backup y cómo restaurarlo
- [ ] Si modifico `docker-compose.yml`, guardé una copia del archivo actual

## Durante el cambio

- [ ] Estoy conectado al servidor via SSH con la clave correcta
- [ ] Tengo otra sesión SSH abierta como respaldo
- [ ] Si voy a reiniciar un servicio, sé el tiempo estimado de downtime
- [ ] Informé al usuario/equipo del downtime si es necesario

## Validación post-cambio

- [ ] El servicio modificado responde correctamente (`curl` o navegador)
- [ ] Los logs no muestran errores (`docker logs <container>`)
- [ ] Si modifiqué Nginx, el SSL sigue funcionando (HTTPS responde)
- [ ] Si modifiqué PostgreSQL, las conexiones funcionan
- [ ] Si modifiqué n8n, los flujos activos siguen corriendo
- [ ] Documenté el cambio en la sección correspondiente de Obsidian

---

## Ver también

**Hub:** [[Arquitectura]] — Hub central de arquitectura y principios de diseño
- [[Plantilla_Arquitectura]] — Template de arquitectura para seguir al crear la app
- [[Cuando_Usar_Cada_Servicio]] — Árbol de decisión de tecnologías
- [[Checklist_Pre_Produccion]] — El checklist siguiente: antes de desplegar en producción
- [[Checklist_Seguridad]] — Verificaciones de seguridad antes de hacer push
- [[Docker_Compose_Base]] — Docker-compose base para el nuevo proyecto
- [[Estructura_Proyecto]] — Estructura de carpetas de FastAPI a seguir
