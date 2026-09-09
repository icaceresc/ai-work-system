# AI Work System — Architecture v1

> **Estado:** vigente.
> **Propósito:** definir una arquitectura portable para trabajar con un Usuario, un Orchestrator IA y uno o más Workers técnicos.
> **Principio:** separar comportamiento general de contexto de entorno y de proyecto desde el inicio.

## 1. Arquitectura

```text
Human User
    ↓
External AI Orchestrator
    ↓
Job Packet / bounded delegation
    ↓
Worker
    ↓
Technical evidence
    ↓
External AI Orchestrator
    ↓
Verification / synthesis / gate
    ↓
Human User
```

## 2. Orchestrator

El Orchestrator es un agente IA/LLM externo al Worker.

Es agnóstico de proveedor/modelo. Su implementación puede cambiar sin alterar la arquitectura.

Es dueño de:
- maduración de intención;
- planificación;
- scope;
- gates;
- selección de herramientas;
- delegación;
- verificación global;
- síntesis;
- continuidad cuando el trabajo es persistente.

El contexto específico de trabajo, proyecto personal o freelance entra mediante planes y conocimiento relevante, no se codifica en la identidad del Orchestrator.

## 3. Worker

El Worker es el ejecutor técnico.

Es dueño de:
- inspección;
- implementación;
- ejecución autorizada;
- verificación técnica local;
- evidencia;
- propuestas.

No redefine por iniciativa propia:
- objetivo;
- plan;
- scope material;
- arquitectura;
- gates globales.

## 4. Worker Core

Debe ser portable entre trabajo, proyectos personales y freelance.

Contiene únicamente reglas generales y estables:
- rol Worker;
- relación con External AI Orchestrator;
- autonomía y escalamiento;
- integridad de información;
- seguridad base;
- estándares de código;
- comunicación según audiencia;
- disciplina de artifacts;
- evidencia y finalización.

No contiene:
- nombres de empresa;
- endpoints internos;
- rutas corporativas;
- datasets concretos;
- reglas de un proyecto;
- infraestructura específica.

## 5. Environment Adapters

Extienden las capacidades del Worker según el entorno.

Ejemplos:

### Work adapter
- router de herramientas;
- diccionario de datos;
- Trino/datalake;
- MLflow corporativo;
- restricciones internas;
- infraestructura aprobada.

### Personal adapter
- herramientas y servicios personales autorizados.

### Freelance adapter
- configuración específica del cliente, separada del core y de otros clientes.

Un adapter no debe contaminar el Worker Core.

## 6. Project Configuration

Cada proyecto puede agregar instrucciones locales.

Ejemplos:
- estructura de carpetas;
- registries;
- IDs;
- invariantes del dominio;
- workflow de notebooks;
- artifacts propietarios;
- restricciones específicas.

Estas reglas no se promueven al Worker Core salvo que demuestren ser verdaderamente generales.

## 7. Job Packet

El Job Packet contiene la autorización concreta de una tarea:
- objetivo;
- contexto necesario;
- scope;
- cambios permitidos/prohibidos;
- autonomía;
- resultado;
- criterios;
- verificaciones;
- stop conditions;
- output.

No es almacenamiento permanente de conocimiento.

## 8. Comunicación por audiencia

### User-facing
Prioriza:
- español;
- conclusión primero;
- bloques cortos;
- comprensión;
- aprendizaje progresivo;
- rigor técnico sin sobrecarga.

### Orchestrator-facing
Prioriza:
- evidencia;
- paths;
- checks;
- métricas;
- contradicciones;
- estado;
- decisiones pendientes;
- formato solicitado por el Job Packet.

## 9. Principios de diseño

- Pareto: conservar lo que aporta valor material.
- YAGNI: no crear capacidades, reglas o artifacts “por si acaso”.
- Un dueño por información.
- Configuración portable primero; adapters después.
- La seguridad importante no debe depender solo de instrucciones textuales cuando existe enforcement técnico razonable.
- El sistema debe reducir trabajo futuro y carga cognitiva del Usuario.

## 10. Portabilidad

Baseline adoptado:

```text
AI Work System/                  # contenedor externo, no es un repositorio Git
├── ai_work_system/               # repo portable/publicable
└── ai_work_system_private/       # repo privado independiente (deployment, entorno real)
```

`ai_work_system_private` no forma parte del repo público y no se documenta en detalle aquí.

Estructura conceptual del repo público (`ai_work_system/`):

```text
ai_work_system/
├── README.md
├── orchestrator/
│   ├── instructions
│   ├── manifest
│   ├── cheatsheet
│   └── knowledge/
├── worker/
│   ├── CLAUDE.md
│   └── environment_adapter.example.md
├── templates/
├── docs/
├── evals/
└── assets/
```

La configuración específica de un entorno de trabajo real (routers, credenciales, infraestructura corporativa) pertenece al deployment privado del Worker, no al repo público.

## 11. Regla de evolución

Antes de agregar una nueva regla o artifact preguntar:

1. ¿resuelve un problema observado?
2. ¿tiene un dueño claro?
3. ¿será usado suficientemente para justificar su coste?
4. ¿puede resolverse con una regla existente?

Si no aporta valor material, no se agrega.
