# Orchestrator Prime — GPT Design Protocol v1

> **Estado:** adoptado para Orchestrator Prime MVP v1. Refinamientos posteriores pertenecen al proyecto `AI Work System`.
> **Propósito:** convertir la creación/evolución de GPTs en una capacidad especializada de Orchestrator Prime, sin introducir un GPT Builder separado salvo evidencia que lo justifique.
> **Ámbito:** Custom GPTs de ChatGPT y decisiones de arquitectura relacionadas.
> **No sustituye:** el operating model general, planes de proyecto ni documentación oficial actual del producto.

## 1. Principio

Un GPT se diseña como un sistema verificable, no como un prompt largo.

Objetivo:

```text
intención del Usuario
→ propósito verificable
→ arquitectura mínima
→ candidate
→ evals
→ Preview
→ gate humano
→ adopción
→ persistencia
```

La prioridad es conservar lo que ya funciona y mejorar solo gaps observados.

### Gate previo — ¿hace falta un GPT separado?

Antes de diseñar un nuevo Custom GPT como solución, demostrar que la separación aporta valor material frente a las capacidades ya disponibles.

Comparar como mínimo, cuando sean opciones reales:

- modelo/chat base;
- Orchestrator Prime existente;
- Worker existente;
- una extensión pequeña de Instructions;
- Knowledge o estándar compartido;
- un GPT separado.

La propuesta debe poder responder:

1. ¿qué problema observable existe?;
2. ¿por qué el sistema actual no lo resuelve suficientemente bien?;
3. ¿qué capacidad nueva aporta un GPT separado?;
4. ¿qué coste introduce en Instructions, Knowledge, mantenimiento, routing, evals y riesgo de drift?;
5. ¿cuál es la alternativa mínima capaz de resolver el problema?

Si el valor incremental de un GPT separado no está demostrado:

> **no diseñarlo como target todavía.**

En ese caso, recomendar la extensión mínima del sistema existente y definir cómo verificar si resulta suficiente.

Este gate ocurre **antes** de diseñar nombre, arquitectura interna, Instructions, Knowledge o herramientas del nuevo GPT.

## 2. Evidencia y precedencia

Para decisiones de diseño, priorizar:

1. requisitos explícitos del Usuario;
2. comportamiento verificado en la cuenta/UI real;
3. configuración efectiva del GPT actual;
4. artifacts/Knowledge vigentes;
5. documentación oficial actual;
6. repositorios oficiales y papers relevantes;
7. inferencias declaradas.

Si documentación y comportamiento real se contradicen:
- registrar la contradicción;
- usar la evidencia empírica de la cuenta para decisiones operativas locales;
- no generalizar ese comportamiento a otros usuarios/cuentas.

Nunca presentar una inferencia como capacidad confirmada.

## 3. Descomposición obligatoria

Antes de diseñar, clasificar cada necesidad en uno de estos dueños:

### Instructions
Comportamiento siempre activo:
- rol;
- objetivos;
- límites;
- workflow;
- routing;
- criterios de decisión;
- formato/estilo;
- reglas que deben aplicarse en todas las conversaciones.

### Knowledge
Material de referencia:
- protocolos detallados;
- manuales;
- arquitectura estable;
- estándares especializados;
- documentación o conocimiento que debe consultarse cuando sea relevante.

No depender de Knowledge como único lugar de una regla crítica de comportamiento.

### Capabilities / tools
Habilitar solo capacidades que tengan un uso real:
- web;
- análisis de archivos/código;
- imágenes;
- otras capacidades disponibles.

### Actions / apps
Solo cuando exista una necesidad concreta de acceder o actuar sobre sistemas externos.
No agregar integración por anticipación.

### Estado mutable del proyecto
No convertirlo en comportamiento global del GPT.
Debe vivir en el plan/artifact propietario del proyecto.

## 4. Routing

Las Instructions deben contener routing compacto hacia Knowledge especializado.

Patrón:

```text
trigger observable
→ artifact family
→ consulta de la versión vigente
```

Ejemplos:
- delegación a Worker → Job Packet + autonomy gates;
- persistencia de plan → planning policy;
- diseño/evolución de GPT → este protocolo.

No crear un router separado si unas pocas reglas de routing en Instructions resuelven el problema.

## 5. Diseño de Instructions

Aplicar:

- instrucciones claras, positivas y concretas;
- estructura explícita para flujos de varios pasos;
- cada regla importante una sola vez;
- ejemplos solo cuando corrigen un gap observado;
- evitar repetir protocolos completos que ya tienen dueño;
- mantener el prompt always-on tan lean como permita el comportamiento requerido;
- no hacer una reescritura total cuando un patch pequeño puede probarse mejor.

Si el GPT ya funciona bien:
1. preservar baseline;
2. cambiar una familia de comportamiento por vez;
3. ejecutar los mismos evals;
4. aceptar solo mejoras sin regresiones materiales.

## 6. Diseño de Knowledge

Antes de agregar un archivo:

1. ¿qué consulta concreta habilita?
2. ¿por qué no basta Instructions + Knowledge existente?
3. ¿tiene un único dueño?
4. ¿es suficientemente estable?
5. ¿su coste de mantenimiento/retrieval se justifica?

Preferir:
- Markdown/texto claro;
- headings descriptivos;
- filenames estables y versionados;
- documentos con SRP;
- referencias a owners en lugar de copias.

Evitar:
- duplicar comportamiento crítico en múltiples archivos;
- documentos enormes de propósito mixto;
- cargar todo el repositorio como Knowledge;
- mezclar material público con contexto confidencial.

## 7. Baseline antes de cambios

Para evolucionar un GPT existente registrar:

- nombre/descripcion actuales;
- Instructions actuales;
- Knowledge vigente;
- capabilities habilitadas;
- Actions/apps;
- modelo recomendado;
- comportamiento observado en casos representativos.

No optimizar contra una impresión general si no existe un fallo identificable.

## 8. Eval design

Los evals deben derivarse de failure modes reales o requisitos explícitos.

Cada caso incluye:

```text
ID
objetivo
input
evidencia/contexto disponible
comportamiento esperado
hard failures
criterios de puntuación
```

Tipos de eval útiles:
- instruction following;
- routing correcto;
- selección de fuente de verdad;
- evidencia vs inferencia;
- missing-information behavior;
- scope discipline;
- artifact/YAGNI gate;
- uso correcto de herramientas;
- continuidad desde artifacts;
- resistencia a instrucciones contradictorias de fuentes menos confiables;
- formato/tono cuando sea requisito de producto.

No evaluar únicamente ejemplos fáciles o parecidos a los usados para escribir las Instructions.

## 9. Comparación baseline vs candidate

Mantener constantes, salvo que la hipótesis exija cambiarlas:

- modelo;
- capabilities;
- datos/Knowledge no involucrados;
- inputs de eval;
- criterio de scoring.

Comparar el GPT actual contra el candidato usando exactamente los mismos casos.

Aceptar un candidate cuando:
- corrige el gap objetivo;
- no introduce regresión material;
- no aumenta complejidad sin beneficio;
- sigue siendo comprensible y mantenible.

## 10. Preview y adopción

Antes de `Actualizar`:

1. probar el draft en Preview;
2. ejecutar evals representativos;
3. revisar fallos cualitativamente;
4. cambiar solo lo que explique un fallo observado;
5. re-ejecutar;
6. presentar evidencia al Usuario.

La adopción final requiere gate humano.

Después:
- actualizar el GPT;
- abrir una conversación nueva;
- verificar comportamiento desde arranque limpio;
- persistir en el repositorio la configuración adoptada;
- cerrar la tarea.

## 11. Capacidad GPT Builder dentro de Orchestrator Prime

Default:

> Orchestrator Prime posee la capacidad de diseñar/evaluar GPTs como parte de su rol de orquestación.

Crear un GPT Builder separado solo si supera el **Gate previo — ¿hace falta un GPT separado?** y aparece evidencia de que:
- la especialización requiere Instructions/Knowledge materialmente incompatibles;
- la separación mejora evals de forma repetible;
- reduce carga cognitiva o coste operacional;
- o existe una necesidad real de distribución/ownership independiente.

La estética de una arquitectura más modular no constituye evidencia suficiente.

## 12. Custom GPT vs ChatGPT Project

No asumir que “proyecto” implica “GPT especializado”.

Antes de crear un GPT por proyecto, comparar:

### Custom GPT
Adecuado para:
- expertise reusable;
- comportamiento estable;
- Knowledge curado y relativamente estático;
- uso transversal.

### ChatGPT Project
Candidato fuerte para:
- trabajo largo y evolutivo;
- múltiples chats;
- archivos que cambian;
- contexto e historial del proyecto;
- memoria de proyecto disponible.

La decisión debe basarse en el comportamiento actual del producto y en la necesidad concreta.

## 13. Finalización

Una tarea de diseño de GPT termina cuando:

- la necesidad está definida;
- la arquitectura mínima está justificada;
- el candidate fue evaluado;
- los gaps materiales están resueltos o explícitamente diferidos;
- el Usuario aprobó la adopción;
- la configuración adoptada quedó persistida;
- no se añaden mejoras retroactivamente sin un nuevo problema observable.

## 14. Base de evidencia usada para este candidate

Consultado el 2026-08-26:

- OpenAI Help Center — Creating and editing GPTs.
- OpenAI Help Center — GPTs in ChatGPT.
- OpenAI Help Center — Projects in ChatGPT.
- OpenAI Help Center — Troubleshooting GPTs.
- OpenAI Model guidance — leaner prompts y evaluación incremental.
- OpenAI Model Spec / instruction hierarchy.
- OpenAI Cookbook — evaluation flywheel.
- OpenAI model_spec_evals — eval prompts + rubric/grader structure.
- Liu et al., TACL 2024 — Lost in the Middle.

Estas fuentes guían el protocolo; las capacidades de producto deben volver a verificarse cuando sean materialmente relevantes porque pueden cambiar.
