---
artifact_id: planning_protocol
artifact_version: 1
artifact_type: knowledge
owner: orchestrator_prime
status: adopted
---

# Planning Protocol

> **Propósito:** decidir cuándo el trabajo necesita artifacts persistentes, y cómo crearlos y mantenerlos para que el trabajo converja y termine.
> **Ámbito:** conocimiento general, reutilizable entre proyectos.
> **Principio:** persistir un plan solo cuando la continuidad futura tenga valor material.

Planificar siempre que sea útil no implica crear siempre un archivo de plan.

## 1. Tarea efímera

Usar planificación efímera dentro del chat cuando la tarea:

- es autocontenida;
- puede completarse razonablemente sin continuidad durable;
- no deja dependencias importantes;
- no necesita un artifact de estado;
- puede cerrarse con resultado + verificación.

```text
intención → mini-plan temporal → ejecución / Job Packet → verificación → resultado → finalización
```

El mini-plan muere con la tarea.

Una tarea efímera puede producir conocimiento durable. Si ocurre, persistir esa decisión o regla en su artifact propietario, sin convertir la tarea entera en un proyecto persistente.

## 2. Proyecto persistente

Usar artifacts persistentes cuando el trabajo:

- abarca varias sesiones;
- acumula decisiones o gates;
- tiene dependencias entre fases;
- coordina múltiples artifacts;
- necesita estado verificable para retomarse;
- haría costoso perder el contexto.

Antes de crear un proyecto persistente, responder explícitamente:

1. ¿Habrá continuidad futura material?
2. ¿Hay estado o gates que deban sobrevivir al chat?
3. ¿Perder este contexto costaría una reconstrucción significativa?

Si la respuesta es no, preferir planificación efímera. No crear un plan persistente preventivamente "por si acaso".

**Promover** una tarea efímera a proyecto persistente si durante la ejecución aparece trabajo multi-sesión, decisiones materiales que deben sobrevivir, varios artifacts coordinados, dependencias nuevas, múltiples gates o necesidad clara de reanudación futura.

## 3. Modelo de artifacts

```text
Proyecto
├── context_<project>.md
└── plan_<project>_<mission>.md
```

El contexto es un mapa durable. El plan es una misión finita. Un proyecto puede tener un solo contexto y varios planes sucesivos.

El plan persistente es dueño de la secuencia, el estado, los gates y la próxima acción autorizada. Los planes y contextos de proyectos no son Knowledge general del Orchestrator: son artifacts propietarios de cada proyecto y se cargan cuando ese proyecto está activo.

Información que define comportamiento estable del sistema pertenece a Knowledge. Información mutable del proyecto pertenece a estos artifacts. Ninguna decisión estructural importante debe sobrevivir solo en el historial de un chat.

## 4. Context artifact

Debe ser **reconstructivo, no un diario**. Permite retomar el proyecto sin releer conversaciones.

Contenido:

- **Propósito** — qué problema resuelve el proyecto y para quién.
- **Arquitectura y owners** — qué artifact o sistema es dueño de qué información.
- **Estado durable** — lo que es cierto sobre el proyecto con independencia del plan activo. No estado de ejecución del día a día.
- **Decisiones adoptadas** — qué se decidió, por qué y cuándo.
- **Límites** — qué queda explícitamente fuera.
- **Fuentes de verdad** — dónde vive el detalle real.
- **Cómo reconstruir** — leer este archivo, leer el plan activo, recuperar detalle desde los artifacts propietarios. No usar el historial del chat como fuente de verdad.

No duplicar en el contexto el detalle que ya vive en sus artifacts propietarios.

## 5. Plan artifact

Un plan tiene **una misión finita**. Debe poder terminar.

Contenido:

- **Objetivo general único** — qué debe quedar resuelto cuando este plan termine.
- **Objetivos o fases delimitados**, con su estado (pendiente / en curso / hecho).
- **Estado** — resumen breve de dónde está el trabajo ahora.
- **Pasos** de la fase en curso.
- **Gates** — qué decisión o verificación debe pasar antes de continuar.
- **Out of scope** — qué NO hace este plan.
- **Siguiente tarea** — una sola tarea concreta, no una lista larga.
- **Criterios de finalización** — condiciones explícitas bajo las cuales el plan se declara terminado.

### Principio crítico

> Los LLM tienden a expandir el scope. El plan existe para **forzar convergencia y terminar el trabajo**, no para acumular ideas.

Si aparece una misión materialmente distinta: cerrar o separar el plan y abrir uno nuevo. No extender indefinidamente el plan vigente.

### Frontmatter recomendado

```yaml
---
artifact_id: plan_<project>_<mission>
artifact_version: <integer>
artifact_type: plan
owner: orchestrator_prime
status: active | completed
---
```

El filename es estable y no lleva sufijo `_vN`. Git conserva el historial; `artifact_version` identifica la revisión lógica del artifact.

## 6. Actualización del plan

Durante una fase:

- usar el chat y la evidencia de ejecución para el progreso operativo;
- no hacer churn del plan por cada micro-paso.

Al cerrar una fase:

1. actualizar el plan propietario (estado, fase completada, gate alcanzado, siguiente tarea);
2. persistir el artifact actualizado;
3. reemplazar su copia en el Knowledge del Orchestrator si está desplegada allí, para mantener trazabilidad.

Si aparece información crítica que invalida el plan, escalar antes de continuar: no reescribir el plan silenciosamente para acomodar el desvío.

## 7. Autoridad

El plan adoptado vigente es la fuente de verdad del estado. Un draft, un candidate o una copia legacy no se vuelve autoridad solo por estar disponible o aparecer en una búsqueda.
