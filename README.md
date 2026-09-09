# AI Work System

Sistema reproducible de trabajo con IA: un **Human User**, un **Orchestrator** y un **Worker** técnico, bajo delegación explícita y verificable.

## 1. Qué es

No es un framework de código. Es un conjunto mínimo de instrucciones y protocolos en texto plano que se despliegan sobre herramientas que ya existen.

```text
Human User
    ↓ intención, contexto, criterio
Orchestrator
    ↓ instrucción madura / Job Packet
Worker
    ↓ evidencia técnica
Orchestrator
    ↓ verificación y síntesis
Human User — decide solo cuando hace falta criterio humano
```

Principio: **el Orchestrator absorbe la complejidad, el Worker ejecuta, el Usuario no coordina modelos ni herramientas a mano salvo necesidad.**

El Worker es un **rol**, no un producto. Implementaciones ya usadas: Claude Code y Antigravity CLI.

## 2. Qué contiene

| Path | Quién lo usa | Para qué | Estado | Cómo se despliega |
|---|---|---|---|---|
| `README.md` | Human User | Instalar, usar y evolucionar el sistema | Requerido | Se lee. No se despliega. |
| `orchestrator/instructions.md` | Orchestrator | Comportamiento always-on: UX, workflow, evidencia, criticidad, routing, delegación, adopción | Requerido | Se pega en el campo *Instructions* del GPT |
| `orchestrator/knowledge/worker_protocol.md` | Orchestrator | Contrato Orchestrator ↔ Worker: Job Packet, autonomía A0–A3, stop conditions, verificación | Requerido | Se sube como Knowledge del GPT |
| `orchestrator/knowledge/planning_protocol.md` | Orchestrator | Tarea efímera vs proyecto persistente; cómo crear context y plan | Requerido | Se sube como Knowledge del GPT |
| `orchestrator/knowledge/gpt_design_protocol.md` | Orchestrator | Crear, auditar y evolucionar GPTs con gates y rollback | Requerido | Se sube como Knowledge del GPT |
| `orchestrator/cheatsheet.md` | Human User | Prompts copy-paste para realinear, cerrar fases o pedir delegación | Opcional | Se lee. No se despliega. |
| `worker/system_prompt.md` | Worker | Worker Core portable: rol, autonomía, integridad, seguridad, código, comunicación, verificación | Requerido | Se despliega como archivo de instrucciones del harness |
| `worker/environment_adapter.example.md` | Worker | Plantilla del Environment Adapter: sistemas, límites y routing del entorno | Opcional | Se copia **fuera** de este repo y se adapta |
| `assets/` | Human User | Imágenes de perfil del GPT | Opcional | Se sube al crear el GPT |
| `.gitignore` | Repo | Evita commitear secretos, temporales y config de editor | Requerido | — |

Los contextos y planes de proyectos **no** viven aquí: son artifacts propietarios de cada proyecto.

## 3. Instalar el Orchestrator

1. Crear un Custom GPT.
2. **Instructions:** pegar el contenido de `orchestrator/instructions.md`.
3. **Knowledge:** subir los tres archivos de `orchestrator/knowledge/`.
4. **Capabilities:** el baseline adoptado tiene habilitadas búsqueda web, intérprete de código / análisis de datos y generación de imágenes. Las dos primeras las usan los protocolos (evidencia desde documentación oficial, análisis de archivos y repositorios); la tercera es opcional.
5. **Actions:** ninguna por defecto. Añadir solo ante una necesidad concreta de actuar sobre un sistema externo.
6. Opcional: usar una imagen de `assets/` como perfil.

Los contextos y planes de un proyecto se suben al Knowledge del GPT solo mientras ese proyecto está activo, y se reemplazan al cerrar cada fase.

## 4. Instalar el Worker

Fuente de verdad: **`worker/system_prompt.md`**. El mismo contenido sirve para cualquier ejecutor compatible; solo cambia el archivo donde se deposita.

- **Claude Code:** desplegar el contenido como `CLAUDE.md` en el scope deseado (global para todo el trabajo, o dentro de un proyecto).
- **Antigravity CLI:** desplegar el contenido en el archivo de instrucciones que soporte el harness (`AGENTS.md` o el equivalente correspondiente al scope).

Verificar el deployment abriendo una sesión nueva y limpia y comprobando que el Worker carga el Core y, si existe, el Environment Adapter.

## 5. Environment Adapter

Opcional. Sirve cuando el entorno tiene sistemas, límites o rutas propias que el Worker debe conocer: qué existe, qué es read-only, qué paths son legibles o escribibles, y dónde está el detalle.

1. Copiar `worker/environment_adapter.example.md` fuera de este repositorio.
2. Completar los sistemas, restricciones y rutas reales del entorno.
3. Enlazarlo según el mecanismo del harness (por ejemplo, un import desde el archivo de instrucciones del Worker).

Puede apuntar a un router propio —un índice estructurado de herramientas y fuentes— cuando el entorno sea complejo. Las credenciales nunca van en el adapter ni en Git: variables de entorno o un mecanismo local aprobado.

## 6. Uso

El Usuario explica su intención al Orchestrator, no una tarea técnica ya traducida.

El Orchestrator decide:

- resolverlo él mismo;
- guiar al Usuario paso a paso;
- delegar en el Worker con un Job Packet;
- abrir un proyecto persistente.

Cuando delega, indica antes al Usuario **sesión, modelo, esfuerzo y modo**. Después verifica la evidencia del Worker antes de darla por buena.

## 7. Proyectos

**Tarea efímera:** autocontenida, sin continuidad durable. Se planifica en el chat y muere con la tarea.

**Proyecto persistente:** varias sesiones, decisiones, gates o artifacts coordinados. Usa dos artifacts:

```text
context_<project>.md    → mapa durable: propósito, owners, estado durable, decisiones, límites
plan_<project>_<mission>.md → una misión finita: fases, gates, siguiente tarea, criterios de finalización
```

Un plan tiene que poder terminar. Si aparece una misión materialmente distinta, se abre un plan nuevo en vez de extender el vigente. Detalle en `orchestrator/knowledge/planning_protocol.md`.

## 8. Evolución del CORE

```text
main
  → branch de cambio
    → candidate (patch mínimo)
      → Preview / sesión limpia
        → smoke test dirigido + uso real
          → PASS: merge   |   FAIL: rollback
```

- **Orchestrator:** probar el cambio en Preview/draft antes de actualizar el GPT.
- **Worker:** probar el cambio en una sesión nueva y limpia antes de confiar en él.

No mergear a `main` sin evidencia de que el cambio pasó su verificación mínima. La adopción final es un gate humano.

## 9. Versionado

SemVer para el repositorio completo, mediante tags de Git:

- `v0.x.y` mientras no exista una primera release pública estable;
- `v1.0.0` para la primera release estable;
- después, *patch* para correcciones, *minor* para capacidades nuevas compatibles, *major* para cambios que rompen el despliegue existente.

Los filenames son estables y no llevan sufijo `_vN`. Git conserva el historial. Los archivos de Knowledge llevan `artifact_version` en su frontmatter para identificar su revisión lógica.

## 10. Público vs privado

Este repositorio es público y contiene únicamente comportamiento general y portable.

Fuera de él, en un deployment privado: el Environment Adapter real, routers y config local, hosts e infraestructura, credenciales, y los contextos y planes de proyectos concretos.
