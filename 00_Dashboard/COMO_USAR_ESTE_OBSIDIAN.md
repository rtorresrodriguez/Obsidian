# Cómo usar este Obsidian

Esta guía explica cómo integrar esta base de conocimiento con Claude Code, Cursor y otros agentes IA para que sigan tu arquitectura de forma consistente.

---

## ¿Para qué sirve este Obsidian?

Este sistema no es solo documentación. Es la **memoria base** que define:
- Qué stack usar y por qué
- Cómo estructurar cada tipo de proyecto
- Qué reglas de seguridad aplicar
- Qué errores ya se resolvieron y cómo

Cuando le dices a un agente IA "revisa mi Obsidian", le estás dando el contexto necesario para tomar buenas decisiones sin que tengas que explicar todo desde cero cada vez.

---

## Flujo de trabajo recomendado

### Inicio de sesión con Claude Code

Pega esto al comienzo de cualquier sesión donde vayas a programar:

```
Antes de hacer cualquier tarea, lee estos archivos:
- G:\Mi unidad\Obsidian\00_Dashboard\INSTRUCCIONES_PARA_AGENTES.md
- G:\Mi unidad\Obsidian\01_Principios_Arquitectura\Principios.md
- G:\Mi unidad\Obsidian\02_Arquitectura_Base_Aplicaciones\Plantilla_Arquitectura.md

Sigue las reglas y arquitectura ahí documentadas.
La tarea es: [TU TAREA AQUÍ]
```

### Inicio de sesión enfocada en un servicio

Si la tarea es específica de un servicio, agrega ese archivo:

```
# Para tareas de FastAPI
- G:\Mi unidad\Obsidian\03_Backend_FastAPI\Estructura_Proyecto.md

# Para tareas de Docker
- G:\Mi unidad\Obsidian\06_Docker_DevOps\Docker_Compose_Base.md

# Para tareas de n8n
- G:\Mi unidad\Obsidian\08_n8n_Automatizacion\n8n_Como_Orquestador.md

# Para crear un agente IA
- G:\Mi unidad\Obsidian\09_IA_Agentes\Arquitectura_Agente.md
- G:\Mi unidad\Obsidian\09_IA_Agentes\Modelos_y_Costos.md
```

---

## Usando los Prompts base

La carpeta [[Prompts_Para_Claude_Code]] tiene 7 prompts listos para las situaciones más comunes. En vez de escribir el contexto desde cero cada vez, copia el prompt que corresponde y completa los campos marcados con `[CORCHETES]`.

| Situación | Prompt a usar |
|-----------|--------------|
| Crear nueva app FastAPI | Prompt: Crear nueva aplicación FastAPI |
| Crear un agente IA | Prompt: Crear agente IA |
| Modificar el servidor | Prompt: Antes de modificar el servidor |
| Revisar arquitectura existente | Prompt: Revisar arquitectura existente |
| Revisar seguridad antes de push | Prompt: Revisar seguridad antes de push |
| Generar docker-compose | Prompt: Generar docker-compose |
| Documentar un cambio | Prompt: Revisar y documentar cambio nuevo |

---

## Usando con Cursor

En Cursor, puedes agregar este Obsidian como contexto de dos formas:

### Opción 1: Reglas de proyecto (`.cursorrules`)

Crea un archivo `.cursorrules` en la raíz de cada proyecto:

```
Antes de responder cualquier pregunta de desarrollo, considera las siguientes reglas de arquitectura:

STACK POR DEFECTO:
- Backend: FastAPI (Python)
- Base de datos: PostgreSQL
- Orquestación: n8n
- Reverse proxy: Nginx
- Contenedores: Docker + Compose
- LLM: Claude (Anthropic API, modelo claude-sonnet-4-6)

REGLAS ABSOLUTAS:
- Nunca hardcodees credenciales. Siempre usar .env
- Nunca expongas el puerto 5432 de PostgreSQL al exterior
- Nunca coloques lógica de negocio en los routers de FastAPI
- La estructura de carpetas sigue el patrón: routers/ services/ models/ schemas/
- Siempre valida inputs con Pydantic
- Siempre usar docker compose (v2), no docker-compose (v1)

DOCUMENTACIÓN DE REFERENCIA:
Ver G:\Mi unidad\Obsidian\ para la arquitectura completa.
```

### Opción 2: Mencionar archivos en el chat

En el chat de Cursor, usa `@` para mencionar archivos de este Obsidian:

```
@G:\Mi unidad\Obsidian\01_Principios_Arquitectura\Principios.md
Sigue estos principios para refactorizar el servicio de usuarios.
```

---

## Cómo actualizar este Obsidian

Este sistema solo es útil si está actualizado. Regla simple: **documenta después de cada cambio significativo**.

### Qué documentar y dónde

| Si hiciste esto... | Documenta aquí |
|-------------------|----------------|
| Creaste un endpoint nuevo | [[Endpoints_Documentados]] |
| Creaste una tabla en PostgreSQL | `05_Base_Datos_PostgreSQL/Tablas/` usando [[Plantilla_Tabla_PostgreSQL]] |
| Modificaste docker-compose | [[Registro_Cambios_Docker]] |
| Resolviste un error difícil | [[Errores_Comunes]] usando [[Plantilla_Error]] |
| Terminaste o iniciaste un proyecto | Carpeta `14_Proyectos/` usando [[Plantilla_Proyecto]] |
| Configuraste un dominio nuevo | [[Dominios_Activos]] |
| Integraste una API externa | [[Integraciones_Externas]] |
| Activaste un flujo en n8n | [[Flujos_Documentados]] |

### Cuánto detalle poner

El nivel de detalle correcto es: **lo suficiente para que tú mismo puedas retomar el trabajo después de 6 meses sin recordar los detalles**.

Incluye siempre:
- El **por qué** de la decisión (no solo el qué)
- Un **ejemplo de código** si aplica
- Los **errores comunes** relacionados si los conoces

---

## Navegación en Obsidian

### Graph view

Abre el Graph View de Obsidian para ver cómo están conectadas todas las notas visualmente. Los nodos más conectados son los más importantes (deberían ser el README y las notas de principios).

### Búsqueda rápida

Usa `Ctrl+O` en Obsidian para abrir cualquier nota por nombre. Todos los archivos tienen nombres descriptivos.

### Tags recomendados (para el futuro)

Puedes agregar tags al frontmatter de las notas para filtrarlas:

```yaml
---
tags: [fastapi, backend, autenticacion]
---
```

---

## Estructura de carpetas de un vistazo

```
00_Dashboard/       ← EMPIEZA AQUÍ siempre
01_Principios/      ← Reglas que nunca cambian
02_Arquitectura/    ← Plantillas y checklists
03_FastAPI/         ← Implementación backend
04_Frontend/        ← Implementación frontend
05_PostgreSQL/      ← Base de datos
06_Docker/          ← Contenedores y DevOps
07_Nginx/           ← Reverse proxy y SSL
08_n8n/             ← Automatización
09_IA_Agentes/      ← LLMs y agentes
10_Memoria_RAG/     ← Memoria y búsqueda semántica
11_APIs/            ← Integraciones y endpoints
12_Seguridad/       ← Secretos y firewall
13_Errores/         ← Soluciones conocidas
14_Proyectos/       ← Estado de proyectos activos
15_Prompts/         ← Prompts reutilizables
99_Plantillas/      ← Plantillas en blanco
```

Ver [[README]] para el índice completo con links a cada nota.
