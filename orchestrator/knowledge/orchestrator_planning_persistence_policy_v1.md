# Orchestrator Prime — Planning Persistence Policy v1

> **Estado:** aprobado durante Fase 4.1 del plan `scoring_micro`.
> **Propósito:** decidir cuándo una tarea requiere un plan persistente y cuándo basta con planificación efímera en el chat.
> **Ámbito:** conocimiento general y reutilizable entre proyectos.

## 1. Principio

> Persistir un plan solo cuando la continuidad futura tenga valor material.

Planificar siempre que sea útil no implica crear siempre un archivo de plan.

## 2. Dos modos de trabajo

### A. Proyecto persistente

Usar un plan persistente cuando el trabajo:
- abarca varias sesiones;
- acumula decisiones o gates;
- modifica múltiples artifacts;
- requiere continuidad temporal;
- tiene dependencias entre fases;
- perder contexto sería costoso;
- necesita estado verificable para retomar trabajo.

El plan persistente es dueño de:
- secuencia;
- estado;
- gates;
- próxima acción autorizada.

Ejemplo: `scoring_micro`.

### B. Tarea efímera

Usar planificación efímera dentro del chat cuando la tarea:
- es autocontenida;
- puede completarse razonablemente en una sesión;
- no deja dependencias importantes;
- no requiere continuidad futura;
- no necesita un artifact de estado;
- puede cerrarse con resultado + verificación.

Flujo:

```text
intención
→ mini-plan temporal
→ ejecución / Job Packet
→ verificación
→ resultado
→ finalización
```

El mini-plan muere con la tarea.

## 3. Cuándo promover una tarea efímera a plan persistente

Promover si durante la ejecución aparece:
- trabajo multi-sesión;
- decisiones materiales que deben sobrevivir;
- varios artifacts coordinados;
- dependencias nuevas;
- múltiples gates;
- riesgo de perder contexto;
- necesidad clara de reanudación futura.

No crear un plan persistente preventivamente “por si acaso”.

## 4. Persistencia de conocimiento sin plan

Una tarea efímera puede producir conocimiento durable.

Si ocurre:
- persistir la decisión, regla o conocimiento en su artifact/Knowledge propietario;
- no convertir automáticamente toda la tarea en un proyecto persistente.

## 5. Relación con otros módulos

- `orchestrator_roles_operating_model_v1.md`: define quién planifica, verifica y escala.
- `orchestrator_job_packet_protocol_v1.md`: define cómo se delega una tarea madura.
- `orchestrator_worker_autonomy_gates_v1.md`: define qué puede hacer el Worker sin escalar.
- este archivo: define si la planificación debe sobrevivir al chat.

## 6. Regla operativa para el Orchestrator

Antes de crear un nuevo plan persistente, evaluar explícitamente:

1. ¿Habrá continuidad futura material?
2. ¿Hay estado/gates que deban sobrevivir?
3. ¿Perder este contexto costaría reconstrucción significativa?

Si la respuesta es no, preferir planificación efímera.
