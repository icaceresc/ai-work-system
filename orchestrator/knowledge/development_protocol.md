---
artifact_id: development_protocol
artifact_version: 1
artifact_type: knowledge
owner: orchestrator_prime
status: adopted
---

# Development Protocol

> **Propósito:** gobernar el trabajo de desarrollo sobre repositorios mantenidos: que el cambio nazca de una intención entendida, sea pequeño y verificable, y pueda revertirse.
> **Ámbito:** conocimiento general, reutilizable entre proyectos y stacks.
> **Principio:** delegar la ejecución no delega la responsabilidad.

Este protocolo no prescribe stack, framework, librería ni herramienta: prescribe cómo se decide, se implementa, se verifica y se adopta un cambio. Las reglas concretas del stack pertenecen al repositorio del proyecto (§ 9).

Complementa `worker_protocol.md`, dueño del contrato de delegación —Job Packet, autonomía A0–A3, stop conditions, formato de evidencia—. Aquí vive qué se hace dentro de ese contrato cuando el trabajo es desarrollo. No se repite.

## 1. Workflow

```text
INTENT
→ UNDERSTAND
→ SPECIFY
→ BASELINE
→ ISOLATE
→ VERIFY FAILURE / DEFINE TEST
→ IMPLEMENT MINIMUM
→ VERIFY
→ EXPLAIN
→ HUMAN GATE (cuando corresponda)
→ MERGE / ROLLBACK
→ DONE
```

| Etapa | Qué ocurre |
|---|---|
| INTENT | El Usuario plantea un problema o una intención, no una tarea ya traducida. |
| UNDERSTAND | Entender el objetivo real e inspeccionar el contexto suficiente antes de proponer solución (§ 2). |
| SPECIFY | Fijar comportamiento esperado, qué no cambia, constraints y criterios de aceptación (§ 3). |
| BASELINE | Registrar el estado de partida: repo, branch, HEAD, working tree, comportamiento actual (§ 5). |
| ISOLATE | Aislar el trabajo en una branch de cambio antes de que un agente edite (§ 5). |
| VERIFY FAILURE / DEFINE TEST | Reproducir el fallo, o definir la evidencia que demostrará el resultado (§ 7). |
| IMPLEMENT MINIMUM | Implementar el cambio más pequeño que satisface la spec (§ 8). |
| VERIFY | Ejecutar las verificaciones definidas y revisar diff y efectos laterales (§ 11). |
| EXPLAIN | Traducir la evidencia a algo que el Usuario pueda entender y defender (§ 10). |
| HUMAN GATE | Aprobación humana cuando la decisión o la adopción lo requieren (§ 4). |
| MERGE / ROLLBACK | Adoptar el cambio demostrado, o descartarlo (§ 5). |

La secuencia es obligatoria como razonamiento, no como ceremonia. Un cambio trivial la recorre en segundos y sin artifacts; un cambio material la recorre de forma explícita. Lo que no está autorizado es saltarse en silencio una etapa material.

## 2. UNDERSTAND

Antes de escribir código:

- identificar qué se quiere conseguir realmente, no solo lo que se pidió;
- separar el problema de la solución propuesta;
- inspeccionar el contexto suficiente: código afectado, convenciones del repo, configuración vigente, tests existentes, historial cuando sea material;
- detectar restricciones reales: interfaces públicas, consumidores, datos, compatibilidad, entorno de ejecución;
- identificar qué decisiones materiales siguen abiertas;
- cuestionar la hipótesis cuando haya evidencia para hacerlo.

No preguntar al Usuario lo que puede resolverse inspeccionando el entorno de forma segura. Preguntar solo lo que la inspección no resuelve y sí cambia la decisión.

Contexto suficiente no es todo el contexto: leer lo que la tarea requiere, no el repositorio entero.

## 3. SPECIFY

Antes de una implementación no trivial debe estar claro:

- qué comportamiento o resultado se quiere;
- qué **no** cambia;
- qué constraints aplican;
- cuáles son los criterios de aceptación;
- cómo se verificará.

*Spec-driven* significa que la intención y los criterios preceden al código. No significa que exista siempre un archivo llamado spec.

La spec es proporcional al cambio:

| Cambio | Dónde vive la spec |
|---|---|
| Trivial y local | el propio pedido, con el comportamiento esperado enunciado |
| Acotado | dentro del Job Packet: resultado esperado, criterios de aceptación, verificaciones |
| Material o multifase | puede justificar un archivo de spec en el repositorio del proyecto |

No crear un `spec.md` por cada cambio por ritual. Si la spec no cabe en un Job Packet o debe sobrevivir a varias sesiones, hay un artifact; si no, no lo hay.

Si falta un elemento material de la spec y la inspección no lo resuelve: detenerse y preguntar antes de implementar. Implementar sobre una spec incompleta produce trabajo desechable con apariencia de progreso.

## 4. Debate y acuerdo

Hay decisiones que no se delegan sin acordarlas antes: arquitectura, metodología, trade-off material, varias soluciones con consecuencias distintas, cambio difícil de revertir, o cualquier decisión que afecte cómo el Usuario interpretará y defenderá el resultado.

En esos casos el Orchestrator presenta la decisión de forma compacta, debate las alternativas con el Usuario y obtiene el gate **antes** de que el Worker implemente:

```text
problema → evidencia → opciones reales → consecuencias → recomendación → decisión
```

La comparación debe ser honesta: alternativas reales, no una opción preferida y dos de relleno.

No se escalan los nombres locales, los detalles de implementación ni las microdecisiones técnicas reversibles. El criterio general de escalación vive en las Instructions; aquí solo se fija que, en desarrollo, el acuerdo sobre *cómo* se resolverá el problema precede a la delegación de la implementación.

## 5. Baseline, branch y merge

Aplica a repositorios Git mantenidos.

Antes de editar, comprobar como mínimo:

- repo root;
- branch actual;
- HEAD o baseline cuando sea material;
- estado del working tree.

Flujo por defecto:

```text
baseline aprobado
→ branch de cambio
→ cambios pequeños
→ tests y verificación
→ revisión de evidencia
→ merge autorizado
```

- `main` no es el workspace normal de un agente cuando existe una branch de cambio viable.
- Working tree inesperadamente dirty: detenerse y reportar. No mezclar cambios ajenos con los propios.
- Si la tarea ya asignó una branch, usar esa; no crear otra por ritual.
- Si el repositorio no usa Git, es desechable o tiene otro workflow adoptado, adaptarse al entorno real en lugar de imponer Git artificialmente.

Merge:

- que los tests pasen no autoriza el merge;
- el Worker reporta evidencia, el Orchestrator la verifica, el Usuario aprueba cuando el cambio o la adopción lo requieren;
- preferir fast-forward cuando la historia y el estado del baseline lo permiten;
- si el baseline avanzó o aparece un conflicto material: detenerse y resolverlo de forma explícita, nunca con un merge a ciegas.

Push, deploy, tags y cambios de configuración del control de versiones: solo con la autorización correspondiente.

El historial de Git da rollback de archivos versionados. Nada más (§ 6).

## 6. Git no protege los efectos externos

Una branch aísla el estado versionado del repositorio. **No** revierte:

- escrituras en bases de datos;
- llamadas a APIs;
- object storage;
- sistemas de tracking de experimentos;
- infraestructura;
- filesystem fuera del repositorio;
- servicios remotos;
- secretos leídos o expuestos;
- cualquier otro side effect.

Esos efectos siguen gobernados por `worker_protocol.md`: nivel de autonomía, stop conditions y los permisos reales del entorno. Una branch no sustituye a un gate.

Cuando exista la opción, recorrer primero el camino reversible: read-only, sandbox, entorno de test, muestra local, *dry run*. Si el efecto externo es inevitable e irreversible, es un gate, no un detalle de implementación.

## 7. TDD proporcional

TDD es una herramienta, no un dogma.

Aplicarlo cuando el comportamiento es determinista —una funcionalidad nueva o un bug reproducible— y automatizar el test es razonable:

1. definir el comportamiento esperado;
2. escribir o identificar un test que falla por la razón correcta;
3. implementar lo mínimo para hacerlo pasar;
4. ejecutar los tests relevantes;
5. refactorizar si aporta claridad sin cambiar comportamiento;
6. volver a ejecutar los tests.

Red → Green → Refactor.

«Falla por la razón correcta» es parte del método: un test que falla por un import roto o una fixture mal construida no demuestra nada sobre el comportamiento.

Para un bug, preferir un regression test que reproduzca el fallo **antes** del fix, cuando hacerlo sea razonable y no tenga un coste desproporcionado. Un bug reproducido es un bug entendido.

Cuando no existe infraestructura de tests, o automatizar el test cuesta más que el riesgo que mitiga: no crear un framework de tests preventivamente. Definir en su lugar evidencia alternativa verificable —comando reproducible, assertion, fixture pequeña, comparación before/after, inspección estructurada u otro acceptance check.

La exigencia no es «que haya tests»: es que exista una verificación reproducible del comportamiento acordado, proporcional al riesgo.

Escribir tests después solo para justificar código ya hecho no es TDD. Puede ser cobertura legítima; no confundir las dos cosas al reportar evidencia.

## 8. Calidad de código

Cuando dos criterios chocan, este es el orden:

1. convenciones adoptadas del repositorio;
2. comportamiento correcto y verificado;
3. simplicidad;
4. claridad y legibilidad;
5. robustez proporcional al riesgo;
6. mantenibilidad real;
7. diff mínimo.

El repositorio gana a la preferencia del agente: código que parece escrito por el mismo equipo vale más que código «mejor» pero ajeno al estilo adoptado.

Invariantes:

- explícito antes que ingenioso;
- YAGNI: sin parámetros de reserva, sin extension points especulativos, sin abstracciones para un futuro hipotético;
- reutilizar antes de generalizar; generalizar con casos reales, no con casos imaginados;
- ninguna dependencia nueva sin necesidad material y su gate;
- nombres que expliquen la intención;
- unidades y tipos coherentes y explícitos;
- fallar de forma visible cuando ocultar el error sería peligroso;
- no capturar ni ignorar errores en silencio sin una razón escrita;
- no refactorizar código no relacionado mientras se arregla otra cosa;
- no optimizar sin una medición que lo justifique;
- preservar las interfaces existentes salvo decisión explícita.

Esto no es una guía de estilo. El estilo concreto lo fija el repositorio (§ 9).

## 9. Configuración técnica: dueño en el repositorio

Distinguir protocolo de configuración ejecutable.

- Este Knowledge: **cómo trabajar**. Estable y portable entre proyectos.
- Repositorio del proyecto: **qué reglas concretas rigen su stack**. Ejecutables y verificables por herramientas.

Pertenecen al repositorio, cuando apliquen: configuración de formatter, linter, type checker, test runner, dependencias y empaquetado, versiones de lenguaje y herramientas, CI, convenciones de directorios, umbrales de cobertura.

Antes de introducir un estándar nuevo, inspeccionar y respetar la configuración existente. Si el repositorio ya declara sus reglas, esas son las reglas. En Python, por ejemplo, esa declaración puede vivir en un `pyproject.toml`, pero no se impone si el repositorio usa otro mecanismo.

Si el repositorio no declara nada y la tarea necesita una regla, esa regla es una decisión de adopción del proyecto: se propone, no se impone desde este protocolo.

## 10. Aprendizaje y accountability del Usuario

El sistema existe para acelerar el desarrollo del Usuario, no para sustituir su comprensión. El Usuario firma el trabajo y debe poder defenderlo fuera del chat.

Para toda decisión técnica no trivial, el Usuario debe poder responder:

1. qué problema resolvimos;
2. qué opción elegimos;
3. por qué esa y no las otras;
4. qué cambió;
5. cómo sabemos que funciona;
6. qué limitaciones quedan.

Reparto de trabajo: el Worker produce el detalle técnico y la evidencia; el Orchestrator la sintetiza en el lenguaje y el nivel que el Usuario necesita. Lo suficiente para responder esas seis preguntas, sin convertir cada tarea en una clase teórica.

Regla: acelerar la ejecución sin eliminar los puntos de comprensión relevantes.

Esto no expresa desconfianza en el Usuario. Es la condición para que el resultado sea auditable y defendible: delegar la tarea, no la responsabilidad.

## 11. Completion

Código escrito no es DONE. Una tarea de desarrollo termina cuando:

- el resultado satisface la spec acordada;
- las verificaciones relevantes se ejecutaron y pasan;
- los efectos laterales materiales se revisaron;
- el diff es consistente con el scope: nada de más, nada de menos;
- la evidencia está disponible y es reproducible;
- las decisiones materiales abiertas están resueltas o escaladas;
- el Usuario y el Orchestrator entienden qué se adopta;
- el merge o la adopción ocurrió con la autorización correspondiente.

Los checks fallidos o saltados se reportan; no se omiten. La verificación local es del Worker; la aceptación global no.

## 12. Principio rector

> Desarrollar más rápido sin desarrollar a ciegas.

Spec antes de implementación material. Evidencia antes de adopción. Branch antes de que un agente edite un repositorio mantenido. Simple antes que complejo. Un cambio justificado antes que muchos cambios enredados.
