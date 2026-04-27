# Registro de cambios en Docker

Documenta aquí cualquier modificación significativa a los archivos de Docker en producción.

## Formato de entrada

```
## YYYY-MM-DD — Descripción breve del cambio

**Quién:** Nombre
**Servicios afectados:** api / postgres / n8n / nginx
**Tipo de cambio:** Agregar servicio / Cambiar configuración / Actualizar imagen / Ajustar volumen

**Qué cambió:**
- Detalle 1
- Detalle 2

**Por qué:**
Razón del cambio.

**Impacto:**
Downtime esperado: X minutos / Sin downtime
Servicios reiniciados: ...

**Verificación post-cambio:**
- [ ] docker compose ps muestra todos los servicios "Up"
- [ ] Health checks pasan
- [ ] Logs sin errores
```

---

## Historial de cambios

*(Agregar entradas aquí en orden cronológico inverso — el más reciente primero)*

---

## Ver también

**Hub:** [[DevOps]] — Hub central de DevOps
- [[Docker_Compose_Base]] — Referencia completa de docker-compose y comandos
- [[Checklist_Pre_Produccion]] — Protocolo completo antes de hacer cualquier cambio
- [[Errores_Comunes]] — Errores conocidos de Docker y cómo resolverlos
