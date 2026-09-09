# AI Work System

Sistema portable para trabajar con un Orchestrator IA (ChatGPT) y un Worker técnico (Claude Code), bajo un modelo de delegación explícita y verificable.

## 1. Qué es

AI Work System es la especificación y los artifacts necesarios para reconstruir, desde cero, un flujo de trabajo donde:
- el Usuario aporta intención, contexto y criterio;
- un Orchestrator IA madura esa intención, planifica y delega en tareas acotadas;
- un Worker técnico ejecuta esas tareas y produce evidencia verificable;
- el Orchestrator verifica y sintetiza antes de devolver el resultado al Usuario.

No es un framework de código. Es un conjunto de instrucciones, protocolos y templates versionados en texto plano.

## 2. Arquitectura

```text
Human User
    ↓
Orchestrator (Orchestrator Prime en ChatGPT)
    ↓
Job Packet / delegación acotada
    ↓
Worker (Claude Code)
    ↓
Evidencia técnica
    ↓
Orchestrator — verificación / síntesis
    ↓
Human User — decide cuando corresponde
```

Detalle completo en [`docs/ai_work_system_architecture_v1.md`](docs/ai_work_system_architecture_v1.md).

Claude Code es la implementación de referencia del rol Worker, no la única posible.

## 3. Componentes y qué artifact los implementa

| Componente | Artifact |
|---|---|
| Instructions del Orchestrator | `orchestrator/orchestrator_prime_instructions_mvp_v1.md` |
| Manifest de reconstrucción del Orchestrator | `orchestrator/orchestrator_prime_mvp_manifest_v1.md` |
| Knowledge del Orchestrator | `orchestrator/knowledge/*.md` |
| Cheatsheet de uso rápido | `orchestrator/cheatsheet.md` |
| Worker Core (portable) | `worker/CLAUDE.md` |
| Environment Adapter (template) | `worker/environment_adapter.example.md` |
| Templates de proyecto | `templates/*.md` |
| Arquitectura general | `docs/ai_work_system_architecture_v1.md` |
| Evals del Orchestrator | `evals/*.md` |

## 4. Desplegar Orchestrator Prime en ChatGPT

1. Crear un Custom GPT.
2. **Instructions:** pegar el contenido de `orchestrator/orchestrator_prime_instructions_mvp_v1.md`.
3. **Knowledge:** subir los archivos listados en la sección "Knowledge general adoptado" de `orchestrator/orchestrator_prime_mvp_manifest_v1.md`.
4. **Capabilities:** las indicadas en el manifest (búsqueda web, generación de imágenes, intérprete de código/análisis de datos). No habilitar Actions salvo necesidad concreta.
5. Guardar/publicar según corresponda a tu cuenta.

Para reconstruir el Orchestrator desde cero sin depender de un chat histórico, seguir la sección "Regla de reconstrucción" del manifest.

## 5. Desplegar el Worker de referencia (Claude Code)

1. Colocar `worker/CLAUDE.md` como configuración global o de proyecto de Claude Code (según cómo organices tu propio setup local).
2. El Worker Core importa un Environment Adapter (`@environment_adapter.md`). Sin ese archivo, el Worker sigue operando con las reglas generales del Core, pero sin routing a herramientas/entornos específicos.

## 6. Adaptar el Worker al entorno

1. Copiar `worker/environment_adapter.example.md` a la ubicación que tu Environment Adapter real debe ocupar (fuera de este repo público).
2. Completar los routers, restricciones y sistemas propios de tu entorno.
3. No subir ese adapter real a un repositorio público: contiene, por diseño, lo específico de tu entorno.

## 7. Qué pertenece al repo público vs al deployment privado

Repo público (`ai_work_system/`, este repositorio):
- Worker Core;
- Instructions/Knowledge/manifest del Orchestrator;
- templates genéricos;
- documentación de arquitectura;
- evals.

Deployment privado (fuera de este repo, p. ej. `ai_work_system_private/` como repositorio independiente):
- Environment Adapter real;
- routers/config local;
- credenciales, hosts, infraestructura corporativa;
- contexto y planes de proyectos concretos.

## 8. Cómo iniciar un proyecto

### Efímero
Tarea autocontenida, sin necesidad de continuidad entre sesiones. Planificación dentro del chat, sin artifact de estado. Ver `orchestrator/knowledge/orchestrator_planning_persistence_policy_v1.md`.

### Persistente
Trabajo multi-sesión con decisiones y gates que deben sobrevivir:
1. Copiar `templates/context_project.md` → `context_<project>.md`.
2. Copiar `templates/plan_project.md` → `plan_<project>.md`.
3. Mantener el contexto como mapa durable y el plan como estado de ejecución.

## 9. Cómo verificar el deployment

- Orchestrator: abrir una conversación nueva y correr un subconjunto de los casos en `evals/`. Confirmar que responde según lo esperado antes de dar por bueno un cambio.
- Worker: abrir una sesión nueva de Claude Code y confirmar que carga el Worker Core y (si existe) el Environment Adapter correctamente.

## 10. Workflow de evolución (lifecycle de cambio)

```text
main
  → branch de cambio
    → patch pequeño
      → commit
        → eval / smoke test
          → sandbox
            → PASS/FAIL
              → merge o rollback
                → tag (si corresponde)
```

- **Orchestrator:** usar Preview/draft como sandbox antes de adoptar un cambio de Instructions/Knowledge.
- **Worker:** usar una sesión nueva y limpia como sandbox antes de confiar en un cambio de configuración.

No mergear a `main` sin evidencia de que el cambio pasó su verificación mínima.

## 11. Versionado

- `v0.x.y` mientras no exista una primera release pública estable.
- `v1.0.0` marca la primera release estable.

Los archivos individuales pueden llevar su propio sufijo `_vN` mientras ese artifact esté en evolución activa; esto es independiente del versionado semántico del repositorio en su conjunto.
