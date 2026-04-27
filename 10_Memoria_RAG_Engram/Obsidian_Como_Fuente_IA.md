# Obsidian como fuente de conocimiento para agentes IA

## Cómo usar este Obsidian con Claude Code

### Opción 1: Dar contexto al inicio de la sesión

Al iniciar una sesión con Claude Code, pega el contenido de los archivos relevantes como contexto:

```
Antes de programar, ten en cuenta mi arquitectura base:

[pegar contenido de 01_Principios_Arquitectura/Principios.md]
[pegar contenido de 02_Arquitectura_Base_Aplicaciones/Plantilla_Arquitectura.md]
```

### Opción 2: Referenciar archivos en el prompt

```
Revisa el archivo en G:/Mi unidad/Obsidian/00_Dashboard/INSTRUCCIONES_PARA_AGENTES.md
y sigue las reglas ahí documentadas antes de continuar.
```

### Opción 3: CLAUDE.md en el proyecto

Crea un archivo `CLAUDE.md` en la raíz de cada proyecto con referencias a esta Obsidian:

```markdown
# Instrucciones para Claude Code

Antes de cualquier tarea, leer:
- G:/Mi unidad/Obsidian/00_Dashboard/INSTRUCCIONES_PARA_AGENTES.md
- G:/Mi unidad/Obsidian/01_Principios_Arquitectura/Principios.md

Arquitectura de este proyecto: ver 14_Proyectos/[nombre_proyecto].md en Obsidian
```

---

## Cómo mantener Obsidian actualizado (para que sirva de memoria real)

### Después de cada proyecto o cambio significativo

1. **Si creaste un endpoint nuevo:** Agrega entrada en `11_Integraciones_API/Endpoints_Documentados.md`
2. **Si encontraste un error nuevo:** Documenta en `13_Errores_y_Soluciones/` con la plantilla
3. **Si cambió la arquitectura:** Actualiza `02_Arquitectura_Base_Aplicaciones/`
4. **Si descubriste una mejor práctica:** Agrega a `01_Principios_Arquitectura/`
5. **Si completaste un proyecto:** Actualiza el estado en `14_Proyectos/`

### Frecuencia recomendada de actualización

| Tipo de cambio | Cuándo documentar |
|---------------|------------------|
| Error resuelto | Inmediatamente después de resolverlo |
| Endpoint nuevo | Al terminar de implementarlo |
| Cambio de arquitectura | Antes de hacer el cambio (como decisión) |
| Estado de proyecto | Al terminar cada sprint o hito |
| Nuevas integraciones | Al terminar de integrar |

---

## Script para que Claude Code lea Obsidian automáticamente

Si usas Claude Code CLI, puedes crear un script que genere el contexto:

```bash
#!/bin/bash
# read_obsidian_context.sh
# Genera un contexto de Obsidian para usar en Claude Code

OBSIDIAN_PATH="G:/Mi unidad/Obsidian"

echo "=== INSTRUCCIONES PARA EL AGENTE ==="
cat "$OBSIDIAN_PATH/00_Dashboard/INSTRUCCIONES_PARA_AGENTES.md"

echo ""
echo "=== PRINCIPIOS DE ARQUITECTURA ==="
cat "$OBSIDIAN_PATH/01_Principios_Arquitectura/Principios.md"

echo ""
echo "=== ARQUITECTURA BASE ==="
cat "$OBSIDIAN_PATH/02_Arquitectura_Base_Aplicaciones/Plantilla_Arquitectura.md"
```

Ejecutar antes de una sesión de Claude Code:
```bash
bash read_obsidian_context.sh | pbcopy  # Mac: copiar al clipboard
# Luego pegar al inicio de la sesión
```

---

## Ver también

- [[COMO_USAR_ESTE_OBSIDIAN]] — Guía completa de uso con Claude Code, Cursor y agentes
- [[Prompts_Para_Claude_Code]] — Prompts listos para iniciar sesiones con agentes IA
- [[INSTRUCCIONES_PARA_AGENTES]] — El archivo que el agente debe leer primero
- [[Memoria_y_RAG]] — Diferencia entre Obsidian, RAG y Engram como sistemas de memoria
