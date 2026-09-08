# Orchestrator Prime — MVP Manifest v1

> **Estado:** listo para adopción.
> **Fecha:** 2026-08-27
> **Propósito:** permitir reconstruir el MVP de Orchestrator Prime desde artifacts explícitos, sin depender de un chat histórico.

## 1. Arquitectura

```text
Orchestrator Prime
├── Instructions always-on
│   ├── reducción de carga cognitiva
│   ├── trabajo incremental
│   ├── evidencia
│   ├── Planificación → Ejecución → Verificación → Finalización
│   ├── routing por familias de Knowledge
│   ├── diseño/evolución de GPTs
│   ├── metadata de Job Packets
│   └── gate de adopción YAGNI/Pareto
├── Knowledge bajo demanda
│   ├── roles / operating model
│   ├── planning persistence
│   ├── Worker job packets
│   ├── Worker autonomy / gates
│   └── GPT design protocol
├── Tools
│   ├── búsqueda web
│   ├── generación de imágenes
│   └── intérprete de código / análisis de datos
└── Actions
    └── ninguna
```

## 2. Instructions adoptadas

Archivo de reconstrucción:

`orchestrator_prime_instructions_mvp_v1.md`

## 3. Knowledge general adoptado

- `orchestrator_roles_operating_model_v2.md`
- `orchestrator_planning_persistence_policy_v1.md`
- `orchestrator_job_packet_protocol_v1.md`
- `orchestrator_worker_autonomy_gates_v1.md`
- `orchestrator_gpt_design_protocol_v1.md`

Los planes y contextos de proyectos no forman parte del conocimiento general del Orchestrator. Son artifacts propietarios de cada proyecto.

## 4. Routing

Las Instructions mantienen routing por **familias de conocimiento**, no una entrada por cada archivo.

Ejemplos:

```text
roles / ownership / gobierno
→ orchestrator_roles_...

Worker delegation
→ orchestrator_job_packet_... + orchestrator_worker_autonomy_...

planning persistence
→ orchestrator_planning_...

GPT design
→ orchestrator_gpt_...

project state
→ contexto del proyecto + plan activo
```

Agregar un nuevo archivo a una familia existente no exige modificar las Instructions salvo que aparezca una nueva categoría de comportamiento.

## 5. Modelo de proyectos persistentes

```text
Proyecto
├── context_<project>.md
│   ├── propósito y alcance duradero
│   ├── arquitectura / owners
│   ├── contexto necesario para reconstruir el proyecto
│   ├── decisiones y restricciones de alto nivel
│   └── puntero al plan activo
└── planes finitos y secuenciales
    ├── plan_<project>_01_...
    ├── plan_<project>_02_...
    └── plan_<project>_NN_...  ← activo
```

El contexto es un **mapa durable**, no un almacén que duplique todos los detalles técnicos.
Los detalles permanecen en sus artifacts propietarios.
Un plan termina; si el proyecto continúa con otro objetivo, se crea un plan nuevo.

## 6. Evidencia de MVP

Validado durante la recuperación de `scoring_micro`:

- Orchestrator → Worker mediante Job Packets;
- ownership y gates;
- planificación persistente vs tareas efímeras;
- Worker Core + Environment Adapter + configuración de proyecto;
- routing candidato de Knowledge;
- comparación baseline vs candidate para capacidad de diseño de GPTs.

## 7. Limitación conocida

El gate:

> demostrar el valor incremental antes de diseñar un GPT separado

está explícito en Instructions y GPT Design Protocol, pero en el eval `Code Steward` el modelo todavía tendió a aceptar demasiado pronto la premisa de crear un GPT.

Estado:

**no bloquea el MVP general de orquestación.**

Se transfiere como failure mode a evaluar/refinar dentro del proyecto `AI Work System`.

## 8. Fuera del MVP

No bloquear trabajo operativo por:

- full eval suite de GPT design;
- normalización de filenames;
- refinamiento del router;
- organización definitiva del repositorio público/privado;
- Code for Humans;
- automatización de evals;
- hooks/actions/integraciones no justificadas.

Estos objetivos pertenecen a planes futuros de `AI Work System`.

## 9. Regla de reconstrucción

Si se pierde el contexto de diseño:

1. leer este manifest;
2. cargar las Instructions adoptadas;
3. cargar únicamente los Knowledge generales vigentes;
4. para un proyecto concreto, recuperar `context_<project>.md` y su plan activo;
5. reconstruir detalle desde los artifacts propietarios;
6. no usar chats históricos como SSOT.
