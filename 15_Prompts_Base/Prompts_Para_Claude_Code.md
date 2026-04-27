# Prompts Base para Claude Code y Agentes IA

Copia y pega estos prompts al inicio de una sesión con Claude Code, Cursor, o cualquier agente IA.

---

## Prompt: Antes de modificar el servidor

```
Antes de hacer cualquier cambio en el servidor de producción, necesito que:

1. Leas los siguientes archivos de mi base de conocimiento en Obsidian:
   - 00_Dashboard/INSTRUCCIONES_PARA_AGENTES.md
   - 12_Seguridad/Checklist_Seguridad.md
   - 02_Arquitectura_Base_Aplicaciones/Checklist_Nueva_App.md (sección "Checklist antes de tocar producción")

2. Sigas estas reglas:
   - No reinicies servicios sin avisarme primero
   - No modifiques docker-compose.yml sin mostrarme el diff antes
   - No borres volúmenes Docker bajo ninguna circunstancia
   - Confírmame cada acción destructiva antes de ejecutarla

3. Me digas qué vas a hacer antes de hacerlo, paso a paso.

La tarea es: [DESCRIBIR LA TAREA AQUÍ]
```

---

## Prompt: Crear nueva aplicación FastAPI

```
Voy a crear una nueva aplicación con FastAPI. Necesito que sigas mi arquitectura base.

Primero, lee estos archivos de mi Obsidian:
- 01_Principios_Arquitectura/Principios.md
- 02_Arquitectura_Base_Aplicaciones/Plantilla_Arquitectura.md
- 03_Backend_FastAPI/Estructura_Proyecto.md
- 12_Seguridad/Checklist_Seguridad.md

Reglas que debes seguir:
- Usa la estructura de carpetas exacta definida en Plantilla_Arquitectura.md
- Separa routers, services, models y schemas
- Nunca pongas lógica de negocio en los routers
- Siempre valida inputs con Pydantic
- Siempre usa variables de entorno via .env (nunca hardcodees)
- Incluye .env.example con todas las variables (sin valores reales)
- Incluye Dockerfile y docker-compose.yml
- El puerto de FastAPI es 8000

Descripción de la aplicación:
- Nombre: [NOMBRE]
- Objetivo: [QUÉ HACE]
- Entidades principales: [TABLAS/MODELOS]
- Necesita autenticación: [SÍ/NO]
- Necesita LLM: [SÍ/NO - y cuál]
- Se integrará con n8n: [SÍ/NO]
```

---

## Prompt: Crear agente IA

```
Necesito crear un agente IA. Sigue mi arquitectura base de agentes.

Lee primero:
- 09_IA_Agentes/Arquitectura_Agente.md
- 01_Principios_Arquitectura/Principios.md

Reglas para el agente:
- El LLM es Claude (Anthropic API, modelo claude-sonnet-4-6)
- El agente NUNCA escribe directamente en la base de datos; usa tools que llaman a la API de FastAPI
- La memoria del agente se guarda en PostgreSQL
- El agente debe tener un system prompt que describa claramente su rol y restricciones
- Usa el SDK oficial de Anthropic (pip install anthropic)
- Implementa el patrón de agentic loop (while stop_reason == "tool_use")

Descripción del agente:
- Nombre: [NOMBRE]
- Rol: [QUÉ HACE EL AGENTE]
- Herramientas que necesita: [LISTA DE TOOLS]
- Tipo de memoria: [en contexto / PostgreSQL / RAG]
- Integración con n8n: [SÍ/NO]
```

---

## Prompt: Revisar arquitectura existente

```
Necesito que revises la arquitectura de [PROYECTO/SERVICIO] y la compares con mi arquitectura base.

Lee primero:
- 01_Principios_Arquitectura/Principios.md
- 01_Principios_Arquitectura/Cuando_Usar_Cada_Servicio.md
- 02_Arquitectura_Base_Aplicaciones/Plantilla_Arquitectura.md

Luego revisa el código en [RUTA] y respóndeme:
1. ¿La estructura de carpetas sigue el patrón recomendado?
2. ¿Hay lógica de negocio en los routers?
3. ¿Los inputs están validados con Pydantic?
4. ¿Hay credenciales hardcodeadas?
5. ¿El puerto de PostgreSQL está expuesto innecesariamente?
6. ¿Hay algo que debería estar en un servicio separado?

Sé específico: menciona el archivo y la línea donde encuentres problemas.
```

---

## Prompt: Documentar errores

```
Encontré un error que quiero documentar para el futuro. Ayúdame a documentarlo siguiendo el formato de mi Obsidian.

Lee la plantilla en: 99_Plantillas/Plantilla_Error.md

El error es:
- Servicio afectado: [FastAPI / PostgreSQL / Docker / Nginx / n8n / Certbot / SSH]
- Síntoma: [QUÉ APARECE EN PANTALLA O EN LOGS]
- Contexto: [CUÁNDO OCURRIÓ, QUÉ ESTABA HACIENDO]
- Solución que funcionó: [LO QUE HICE PARA RESOLVERLO]

Genera el documento Markdown completo para guardarlo en 13_Errores_y_Soluciones/.
```

---

## Prompt: Revisar seguridad antes de push

```
Antes de hacer git push, necesito que revises el código por problemas de seguridad.

Lee: 12_Seguridad/Checklist_Seguridad.md

Revisa el código en [RUTA o describe el cambio] y verifica:
1. ¿Hay credenciales, API keys o contraseñas en el código?
2. ¿Hay archivos .env en el staging area?
3. ¿Los inputs de usuario están validados antes de usarse?
4. ¿Hay queries SQL construidas con f-strings o concatenación? (SQL injection)
5. ¿Los endpoints nuevos tienen autenticación si la necesitan?
6. ¿Las respuestas de error revelan información sensible?
7. ¿El .gitignore está actualizado?

Dame un reporte de los problemas encontrados y cómo corregirlos.
```

---

## Prompt: Generar docker-compose

```
Necesito que generes un docker-compose.yml para mi proyecto.

Lee primero: 06_Docker_DevOps/Docker_Compose_Base.md

El proyecto tiene los siguientes servicios:
- [LISTAR SERVICIOS: FastAPI / PostgreSQL / n8n / Nginx / Redis / etc.]

Reglas obligatorias:
- Usar "restart: unless-stopped" en todos los servicios
- PostgreSQL NUNCA debe exponer el puerto 5432 al exterior
- FastAPI y n8n tampoco exponen puertos directamente (van via Nginx)
- Solo Nginx expone los puertos 80 y 443
- Usar variables de entorno via env_file: .env (nunca valores directos)
- Los datos de PostgreSQL deben estar en un volumen nombrado
- Los datos de n8n deben estar en un volumen nombrado
- Todos los servicios en la misma red (bridge)
- PostgreSQL debe tener healthcheck configurado

Nombre del proyecto: [NOMBRE]
```

---

## Prompt: Revisar y documentar cambio nuevo

```
Acabo de hacer el siguiente cambio en el sistema:
[DESCRIBIR EL CAMBIO]

Ayúdame a documentarlo correctamente en mi Obsidian siguiendo las instrucciones de:
00_Dashboard/INSTRUCCIONES_PARA_AGENTES.md (sección "Cómo documentar cualquier cambio nuevo")

El cambio afecta:
- [ ] Un endpoint nuevo → documentar en 11_Integraciones_API/
- [ ] Una tabla nueva en PostgreSQL → documentar en 05_Base_Datos_PostgreSQL/
- [ ] Un cambio en Docker → registrar en 06_Docker_DevOps/
- [ ] Un error resuelto → documentar en 13_Errores_y_Soluciones/
- [ ] Un proyecto nuevo o actualizado → actualizar en 14_Proyectos/
- [ ] Una configuración de Nginx → documentar en 07_Nginx_SSL_Dominios/

Genera el Markdown listo para pegar en la carpeta correspondiente.
```

---

## Ver también

- [[INSTRUCCIONES_PARA_AGENTES]] — Reglas completas que los agentes deben seguir
- [[COMO_USAR_ESTE_OBSIDIAN]] — Guía de integración con Claude Code y Cursor
- [[Plantilla_Proyecto]] — Para documentar proyectos nuevos
- [[Plantilla_Error]] — Para documentar errores resueltos
- [[Plantilla_Endpoint]] — Para documentar endpoints nuevos
