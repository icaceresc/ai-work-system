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
| `orchestrator/knowledge/development_protocol.md` | Orchestrator | Cómo desarrollar sobre un repositorio: spec proporcional, baseline y branch, TDD proporcional, calidad y verificación antes de adoptar | Requerido | Se sube como Knowledge del GPT |
| `orchestrator/knowledge/model_development_protocol.md` | Orchestrator | Overlay de modelado y ML: problema y target, validación y leakage, baseline, experimentos incrementales, reproducibilidad | Requerido | Se sube como Knowledge del GPT |
| `orchestrator/knowledge/gpt_design_protocol.md` | Orchestrator | Crear, auditar y evolucionar GPTs con gates y rollback | Requerido | Se sube como Knowledge del GPT |
| `orchestrator/cheatsheet.md` | Human User | Prompts copy-paste para realinear, cerrar fases o pedir delegación | Opcional | Se lee. No se despliega. |
| `worker/worker_core.md` | Worker | Comportamiento portable del rol Worker: autonomía, integridad, seguridad, código, comunicación, verificación | Requerido | Se despliega como archivo de instrucciones del ejecutor |
| `worker/environment_adapter.example.md` | Worker | Plantilla del Environment Adapter: sistemas, límites y routing del entorno | Opcional | Se copia **fuera** de este repo y se adapta |
| `assets/` | Human User | Imágenes de perfil del GPT | Opcional | Se sube al crear el GPT |
| `.gitignore` | Repo | Evita commitear secretos, temporales y config de editor | Requerido | — |

Los contextos y planes de proyectos **no** viven aquí: son artifacts propietarios de cada proyecto.

## 3. Instalar el Orchestrator

1. Crear un Custom GPT.
2. **Instructions:** pegar el contenido de `orchestrator/instructions.md`.
3. **Knowledge:** subir los cinco archivos de `orchestrator/knowledge/`.
4. **Capabilities:** el baseline adoptado tiene habilitadas búsqueda web, intérprete de código / análisis de datos y generación de imágenes. Las dos primeras las usan los protocolos (evidencia desde documentación oficial, análisis de archivos y repositorios); la tercera es opcional.
5. **Actions:** ninguna por defecto. Añadir solo ante una necesidad concreta de actuar sobre un sistema externo.
6. Opcional: usar una imagen de `assets/` como perfil.

Los cinco Knowledge canónicos llevan frontmatter (`artifact_id`, `artifact_version`, `artifact_type`, `owner`, `status`); las Instructions no lo necesitan.

Los contextos y planes de un proyecto se suben al Knowledge del GPT solo mientras ese proyecto está activo, y se reemplazan al cerrar cada fase.

## 4. Instalar el Worker

Fuente de verdad: **`worker/worker_core.md`**.

Este archivo **no reemplaza el system prompt del producto**. Es una capa portable de instrucciones que se suma a lo que el ejecutor ya trae:

```text
instrucciones y capacidades nativas del agente
+ Worker Core
+ Environment Adapter (opcional)
+ contexto del proyecto
= comportamiento efectivo del Worker
```

Identidad del agente, herramientas, skills, subagentes y comportamiento de interfaz siguen siendo del producto. Este repo no los documenta ni los sustituye.

Worker Core portable no implica comportamiento idéntico entre agentes: cada ejecutor trae su propio system prompt, capacidades y comportamiento de interfaz, que este repo no controla.

### Claude Code — reference Worker implementation

Validación conductual PASS con el Worker Core actual. Recomendado para trabajo real.

Desplegar el contenido de `worker/worker_core.md` en:

```text
~/.claude/CLAUDE.md
```

Sin Environment Adapter, no hace falta nada más. Con Adapter, referenciarlo explícitamente desde ese archivo mediante el mecanismo de referencia que soporta Claude Code; la referencia se resuelve y el Adapter puede dirigir después al router y a los helpers del entorno.

### Antigravity CLI 1.2.0 — experimental / best-effort

Implementación compatible con el Worker Core: la carga del Core vía `~/.gemini/config/AGENTS.md` está comprobada. La paridad de comportamiento con Claude Code **no** está demostrada. Este repo no afirma causa del comportamiento observado en uno u otro ejecutor.

Desplegar el contenido de `worker/worker_core.md` en:

```text
~/.gemini/config/AGENTS.md
```

Comprobado: un archivo en esa ubicación afecta el comportamiento global de las sesiones nuevas.

Sin Environment Adapter, no hace falta nada más. Con Adapter, **incorporar sus instrucciones inline en ese mismo `AGENTS.md`**.

Por qué inline: en el deployment probado no quedó demostrado que un `environment_adapter.md` separado en la misma carpeta se cargue automáticamente, ni que `AGENTS.md` soporte referencias a otros archivos como Claude Code. Esto se documenta para la versión 1.2.0; no se afirma nada sobre versiones futuras.

### Verificar

Abrir una sesión nueva y limpia y comprobar que el Worker aplica las reglas del Core y, si corresponde, las del Adapter.

## 5. Environment Adapter

Opcional. Sirve cuando el entorno tiene sistemas, herramientas, fuentes de verdad, rutas o restricciones propias que el Worker debe conocer. Evita que adivine infraestructura, que la redescubra en cada tarea y que esa configuración privada acabe dentro del Core portable.

```text
Worker Core
    ↓
Environment Adapter (opcional)
    ↓
router (opcional)
    ↓
references / helpers / tools
```

1. Copiar `worker/environment_adapter.example.md` fuera de este repositorio.
2. Completar los sistemas, restricciones y rutas reales del entorno.
3. Desplegarlo según lo verificado para tu ejecutor (sección 4): referencia desde el archivo de instrucciones en Claude Code, inline en Antigravity CLI 1.2.0.

El adapter define un contrato lógico, no un mecanismo de carga universal. El router no es obligatorio; para entornos complejos, un índice estructurado —por ejemplo un JSON que mapee cada sistema a su referencia o helper aprobado— ha funcionado bien en uso real. Las credenciales nunca van en el adapter ni en Git: variables de entorno o un mecanismo local aprobado.

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

**Proyecto persistente:** varias sesiones, decisiones, gates o artifacts coordinados.

```text
PROJECT
├── context_<project>.md              → mapa durable: propósito, owners, estado durable, decisiones, límites
├── plan_<project>_<mission>.md       → una misión finita: fases, gates, siguiente tarea, criterios de finalización
├── otros planes, si existen misiones distintas
└── ideas_<project>.md                → opcional
```

**Proyecto no es lo mismo que plan.** Un proyecto puede abarcar varias misiones a lo largo del tiempo; cada plan representa exactamente una misión finita y tiene que poder terminar. Misiones distintas van en planes distintos, nunca como extensión del vigente. Ideas futuras no pertenecen al plan activo: `ideas_<project>.md` es opcional, no autoriza ejecución y no bloquea el cierre del plan. Detalle en `orchestrator/knowledge/planning_protocol.md`.

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

`artifact_version` identifica la revisión **adoptada** de ese Knowledge file, no cada commit ni cada iteración de una branch de cambio. Los commits dentro de una candidate branch no la incrementan; se incrementa cuando una modificación semántica llega a `main` después de que la versión anterior ya estaba adoptada. Git conserva el historial fino de cada cambio; el tag SemVer versiona el AI Work System completo. Los Knowledge actuales están en `artifact_version: 1` porque es su primera revisión canónica adoptable.

## 10. Público vs privado

Este repositorio es público y contiene únicamente comportamiento general y portable.

Fuera de él, en un deployment privado: el Environment Adapter real, routers y config local, hosts e infraestructura, credenciales, y los contextos y planes de proyectos concretos.
