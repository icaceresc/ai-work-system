---
artifact_id: planning_protocol
artifact_version: 3
artifact_type: knowledge
owner: orchestrator_prime
status: adopted
---

# Planning Protocol

> **Propósito:** decidir cuándo el trabajo necesita Project control files, y cómo crearlos y mantenerlos para que el trabajo converja y termine.
> **Ámbito:** conocimiento general, reutilizable entre proyectos.
> **Principio:** persistir un plan solo cuando la continuidad futura tenga valor material.

Planificar siempre que sea útil no implica crear siempre un archivo de plan.

## 1. Tarea efímera

Usar planificación efímera dentro del chat cuando la tarea:

- es autocontenida;
- puede completarse razonablemente sin continuidad durable;
- no deja dependencias importantes;
- no necesita un Project control file de estado;
- puede cerrarse con resultado + verificación.

```text
intención → mini-plan temporal → ejecución / Job Packet → verificación → resultado → finalización
```

El mini-plan muere con la tarea.

Una tarea efímera puede producir conocimiento durable. Si ocurre, persistir esa decisión o regla en la fuente de verdad durable que le corresponda, sin convertir por eso la tarea entera en un proyecto persistente.

## 2. Proyecto persistente

Usar Project control files cuando el trabajo:

- abarca varias sesiones;
- acumula decisiones o gates;
- tiene dependencias entre fases;
- coordina múltiples Project control files;
- necesita estado verificable para retomarse;
- haría costoso perder el contexto.

Antes de crear un proyecto persistente, responder explícitamente:

1. ¿Habrá continuidad futura material?
2. ¿Hay estado o gates que deban sobrevivir al chat?
3. ¿Perder este contexto costaría una reconstrucción significativa?

Si la respuesta es no, preferir planificación efímera. No crear un plan persistente preventivamente "por si acaso".

**Promover** una tarea efímera a proyecto persistente si durante la ejecución aparece trabajo multi-sesión, decisiones materiales que deben sobrevivir, varios Project control files coordinados, dependencias nuevas, múltiples gates o necesidad clara de reanudación futura.

## 3. Modelo de Project control files

Layout canónico para repos persistentes gestionados con AI Work System: contrato obligatorio cuando el proyecto usa Project control files.

```text
repo/
└── project/
    ├── context.md
    └── plans/
        └── <mission>.md        # una por misión; otras solo si existen misiones realmente distintas
```

Este es el layout canónico obligatorio. No forma parte del contrato ningún archivo adicional (ideas, issues, backlog u otro): si un proyecto concreto necesita persistir algo más, es una decisión local de ese proyecto, no una extensión del estándar.

`context.md` y cada archivo de `project/plans/` son **Project control files**: gobiernan el estado y el scope del proyecto o de una misión concreta. Project control files viven en el repositorio del proyecto y **nunca** se despliegan ni se sincronizan con el Knowledge del Orchestrator, en ningún flujo adoptado. Si el directorio `project/` queda tracked por Git es una decisión del repo concreto, no una regla de este protocolo.

**Source of truth ≠ mecanismo de transporte.** El repo es siempre el owner de estos archivos, independientemente de cómo el Orchestrator llegue a su contenido. El Orchestrator los consume mediante el acceso disponible en su propio entorno — este protocolo no asume un mecanismo único:

- si el Orchestrator no tiene filesystem propio (por ejemplo, un Custom GPT en un chat web), normalmente el Usuario le proporciona `context.md` y el plan activo como adjuntos o texto pegado;
- si hay un Worker con acceso al repo, el Orchestrator puede delegarle un Job Packet read-only para que inspeccione el repo y transporte evidencia o un resumen referenciado;
- transportar o resumir esa información no cambia su ownership: el repo sigue siendo la fuente de verdad, y el archivo transportado o el resumen no la reemplazan de forma permanente;
- el Orchestrator no debe asumir que conoce contenido del repo que no le fue proporcionado por uno de estos dos caminos.

Este contrato es portable: no depende de qué producto concreto implemente el rol Orchestrator ni de si tiene o no acceso directo a un filesystem.

`context.md` es el mapa durable y el punto de entrada para reconstruir una sesión nueva del Orchestrator: debe declarar explícitamente su plan activo (`active_plan: plans/<mission>.md`, o `active_plan: none` si no hay ninguno). Cada archivo en `project/plans/` representa exactamente una misión finita. Los filenames son estables (`<mission>.md`, `context.md`), sin sufijo `_vN`: cambiar de sesión, cerrar una fase o iterar un plan actualiza el mismo archivo, nunca crea `plan_v2`, `plan_v3` ni variantes.

Historial: si el repo concreto trackea `project/` con Git, cada actualización queda en el historial de Git y es recuperable como en cualquier otro archivo versionado. Si el repo decide no trackearlo, Git no provee historial ni rollback para esos archivos — el estado vive solo en la copia local, sin red de seguridad de versión anterior. `artifact_version` en el frontmatter identifica la revisión lógica adoptada del artifact; por sí solo no implica que exista historial Git ni sustituye a uno. Este protocolo no introduce un mecanismo de backup propio: la decisión de trackear `project/` con Git es la que determina si hay historial recuperable.

**Proyecto no equivale a plan.** Un proyecto puede abarcar varias misiones a lo largo del tiempo; cada plan representa exactamente una. Que algo pertenezca al proyecto no implica que pertenezca al plan activo. Una misión futura no se escribe como plan hasta que se adopta.

Lo prohibido no es que un proyecto tenga varios planes, sino agregar varias misiones dentro de uno.

El plan persistente es dueño de la secuencia, el estado, los gates y la próxima acción autorizada de su misión.

Información que define comportamiento estable del sistema pertenece a Knowledge. Información mutable del proyecto pertenece a estos Project control files. Ninguna decisión estructural importante debe sobrevivir solo en el historial de un chat.

### Reconstrucción de sesión nueva

Una sesión nueva del Orchestrator reconstruye el proyecto en este orden, usando el mecanismo de acceso disponible en su entorno:

1. disponer de `project/context.md` — proporcionado por el Usuario, o inspeccionado y transportado por un Worker delegado;
2. leer su `active_plan`;
3. disponer del plan activo referenciado, por el mismo mecanismo;
4. recuperar solo las fuentes de verdad propietarias que la siguiente tarea realmente necesite, no todas por ritual.

Si falta una fuente material para continuar, el Orchestrator la solicita al Usuario o delega su inspección a un Worker con acceso al repo — no la asume ni la reconstruye desde el historial del chat. El Worker sigue recibiendo Job Packets autocontenidos por defecto.

## 4. Context control file

Debe ser **reconstructivo, no un diario**. Permite retomar el proyecto sin releer conversaciones.

Contenido:

- **Propósito** — qué problema resuelve el proyecto y para quién.
- **Arquitectura y owners** — qué artifact o sistema es dueño de qué información.
- **Estado durable** — lo que es cierto sobre el proyecto con independencia del plan activo. No estado de ejecución del día a día.
- **Decisiones adoptadas** — qué se decidió, por qué y cuándo.
- **Límites** — qué queda explícitamente fuera.
- **Fuentes de verdad** — dónde vive el detalle real.
- **Plan activo** — referencia explícita al plan vigente (`active_plan: plans/<mission>.md`) o `active_plan: none` si no hay ninguno en curso.
- **Cómo reconstruir** — leer este archivo, leer el plan activo y recuperar el detalle desde las fuentes de verdad propietarias. No usar el historial del chat como fuente de verdad.

No duplicar en el contexto detalle que ya vive en su fuente de verdad propietaria.

### Frontmatter recomendado

```yaml
---
artifact_id: context_<project>
artifact_version: <integer>
artifact_type: context
owner: <owner>
status: active
---
```

`artifact_id` identifica el artifact lógico y permanece estable. `artifact_version` representa una revisión lógica del contexto, no cada edición ni cada cambio de sesión. El filename sigue siendo `project/context.md`; nunca `context_v2.md` ni variantes. Este frontmatter y su versión no implican backup ni historial Git (ver §3).

## 5. Plan control file

Un plan tiene **una misión finita**. Debe poder terminar.

Una misión válida:

- tiene un objetivo general único;
- tiene criterios de finalización verificables;
- puede terminar aunque sigan existiendo ideas, mejoras o posibilidades;
- no depende de seguir capturando, revisando o triando trabajo futuro;
- no agrega varias misiones bajo una etiqueta paraguas.

«Dejar el sistema listo para la release pública v1» es una misión válida. «Mejorar el sistema indefinidamente y gestionar todas sus ideas futuras» no lo es: no tiene un DONE verificable. Lo que invalida una misión es su semántica agregadora, no la palabra que la nombre.

Contenido:

- **Objetivo general único** — qué debe quedar resuelto cuando este plan termine.
- **Objetivos o fases delimitados**, con su estado (pendiente / en curso / hecho).
- **Estado** — resumen breve de dónde está el trabajo ahora.
- **Pasos** de la fase en curso.
- **Gates** — qué decisión o verificación debe pasar antes de continuar.
- **Out of scope** — qué NO hace este plan.
- **Siguiente tarea** — una sola tarea concreta, no una lista larga.
- **Criterios de finalización** — condiciones explícitas bajo las cuales el plan se declara terminado.

### Principio crítico — el plan es finito

> Los LLM tienden a expandir el scope. El plan existe para **forzar convergencia y terminar el trabajo**, no para almacenar ideas.

Invariante: **cumplidos sus criterios de finalización, el plan se cierra como `completed`.** Ideas aplazadas, mejoras conocidas, integraciones deseables o misiones potenciales no bloquean el cierre, haya una o cien. No existe trabajo posterior al DONE cuya única finalidad sea gestionar posibilidades futuras.

El plan activo contiene únicamente el trabajo necesario para completar su misión vigente. Cada objetivo, fase, paso o elemento estructural debe contribuir materialmente a al menos un criterio de finalización vigente: si el DONE ya puede alcanzarse sin X, X no pertenece al plan activo.

En particular, una fase cuyo propósito sea revisar, triar, registrar, considerar, clasificar o decidir sobre trabajo situado **después** del DONE vigente no pertenece al plan. Da igual cómo se llame —fase futura, revisión final, backlog, roadmap, inbox, parking lot, trabajo diferido, oportunidades, mejoras—: se evalúa qué hace, no su etiqueta.

No incorporar al plan:

- ideas futuras;
- nice-to-haves;
- automatizaciones opcionales;
- integraciones posibles;
- GPTs futuros;
- mejoras no necesarias para los criterios de finalización actuales;
- backlog general del proyecto.

No uses el plan activo como backlog general.

### Test de admisión

Aplicable a **cualquier** elemento candidato X: tarea, idea, fase, sección, tabla, estado, artifact embebido o cualquier contenido que pretenda entrar al plan.

**Q1 — ¿los criterios de finalización vigentes son inalcanzables sin X?**

- No → X no entra al plan activo.
- Sí → Q2.

**Q2 — ¿X permanece dentro del objetivo y los boundaries vigentes?**

- Sí → puede incorporarse, sujeto a los gates aplicables.
- No → requiere decisión humana: re-scope explícito del plan vigente, o misión nueva en un plan nuevo. No extender indefinidamente el plan vigente.

Mientras esa decisión se toma, no uses el plan activo como contenedor provisional.

No cuentan como necesidad para alcanzar el DONE: no perder una idea, recordarla, tenerla visible, revisarla después, triarla o decidir sobre ella en el futuro. Son motivos legítimos para preservarla, no para admitirla.

Si X sí es imprescindible para el DONE vigente, entra. El test acota el plan, no lo congela.

### Ideas que quedan fuera del plan

Que una idea no entre al plan no significa descartarla.

Si el Usuario pide explícitamente no perder ideas futuras, reconoce la intención y sepárala del scope: la petición legítima es no perderlas, no convertirlas en trabajo comprometido.

Una idea que no pasa Q1 no puede persistirse dentro del plan bajo ninguna forma: ni sección, ni apéndice, ni tabla, ni lista, ni fase, ni máquina de estados, ni backlog, inbox, roadmap o equivalente semántico. Si no hace falta persistencia durable, simplemente queda fuera del plan.

Cuando —y solo cuando— haga falta persistencia durable, el proyecto puede usar una fuente separada para esas ideas, si de verdad la necesita. Este protocolo no define su filename, ubicación ni layout: no es parte del contrato canónico (`project/context.md` + `project/plans/`), y no debe tratarse como si lo fuera. Cualquier fuente de ese tipo que un proyecto decida usar:

- no es un plan;
- no es autoritativa para la ejecución;
- no altera la misión vigente;
- no añade fases;
- no bloquea los criterios de finalización;
- no genera trabajo futuro automáticamente;
- no obliga a crear una misión futura.

Cada idea requiere una decisión posterior explícita antes de convertirse en misión.

No es un componente obligatorio del sistema ni una capa nueva: solo existe si el proyecto concreto la necesita, y no requiere plantilla ni convención estándar.

### Frontmatter recomendado

```yaml
---
artifact_id: plan_<mission>
artifact_version: <integer>
artifact_type: plan
owner: <owner>
status: active | completed
---
```

Ubicación: `project/plans/<mission>.md`. El filename es estable y no lleva sufijo `_vN`. `artifact_version` identifica la revisión lógica del artifact; que Git conserve historial de esos cambios depende de si el repo trackea `project/` (ver §3).

## 6. Actualización del plan

Durante una fase:

- usar el chat y la evidencia de ejecución para el progreso operativo;
- no hacer churn del plan por cada micro-paso.

Que aparezca una idea nueva no es motivo para actualizar el plan. La secuencia «apareció una idea → siguiente tarea: actualizar el plan» no está autorizada.

Al cerrar una fase:

1. actualizar el plan propietario (estado, fase completada, gate alcanzado, siguiente tarea);
2. persistir el Project control file actualizado en su owner repo-local — nunca en el Knowledge del Orchestrator.

La única razón para actualizar el plan fuera del cierre de una fase es que evidencia material nueva lo invalide y exija una decisión o un gate. En ese caso, escalar antes de continuar: no reescribir el plan silenciosamente para acomodar el desvío.

## 7. Autoridad

El plan adoptado vigente es la fuente de verdad del estado, el scope y el progreso **de su misión**. No gobierna las reglas del sistema, los protocolos adoptados, las Instructions ni otras misiones.

Un context o un plan no puede redefinir las invariantes de este protocolo. Si un Project control file las contradice, reportar la inconsistencia en lugar de tratarlo como autoridad superior.

Un draft, un candidate o una copia legacy no se vuelve autoridad solo por estar disponible o aparecer en una búsqueda.
