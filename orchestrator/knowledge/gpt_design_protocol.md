---
artifact_id: gpt_design_protocol
artifact_version: 1
artifact_type: knowledge
owner: orchestrator_prime
status: adopted
---

# GPT Design Protocol

> **Propósito:** diseñar, auditar y evolucionar GPTs como sistemas verificables, sin introducir un GPT Builder separado salvo evidencia que lo justifique.
> **Ámbito:** Custom GPTs de ChatGPT y las decisiones de arquitectura relacionadas.
> **Principio:** conservar lo que ya funciona y mejorar solo gaps observados.

```text
problema observable
→ gate: ¿hace falta un GPT separado?
→ arquitectura mínima
→ candidate en branch
→ patch mínimo
→ Preview / sandbox
→ smoke test dirigido al failure mode
→ uso real + feedback del Usuario
→ PASS: adoptar y persistir  |  FAIL: rollback
```

## 1. Gate previo — ¿hace falta un GPT separado?

Antes de diseñar un Custom GPT nuevo como solución, demostrar que la separación aporta valor material frente a lo que ya existe.

Comparar, cuando sean opciones reales:

- modelo/chat base;
- el Orchestrator existente;
- el Worker existente;
- una extensión pequeña de Instructions;
- un archivo de Knowledge o un estándar compartido;
- un GPT separado.

La propuesta debe responder:

1. ¿qué problema observable existe?
2. ¿por qué el sistema actual no lo resuelve suficientemente bien?
3. ¿qué capacidad nueva aporta un GPT separado?
4. ¿qué coste introduce en Instructions, Knowledge, mantenimiento, routing y riesgo de drift?
5. ¿cuál es la alternativa mínima capaz de resolver el problema?

Si el valor incremental no está demostrado:

> **no diseñar el GPT separado todavía.**

Recomendar en su lugar la extensión mínima del sistema existente y definir cómo se comprobará si resulta suficiente.

Este gate ocurre **antes** de diseñar nombre, arquitectura interna, Instructions, Knowledge o herramientas.

La estética de una arquitectura más modular no es evidencia. Por defecto, la capacidad de diseñar y evaluar GPTs pertenece al Orchestrator existente; un GPT Builder separado requiere superar este gate y mostrar que la especialización necesita Instructions/Knowledge materialmente incompatibles, que mejora el resultado de forma repetible, que reduce carga cognitiva o coste operacional, o que existe una necesidad real de ownership independiente.

## 2. Custom GPT vs Project

No asumir que "proyecto" implica "GPT especializado".

- **Custom GPT:** expertise reusable, comportamiento estable, Knowledge curado y relativamente estático, uso transversal.
- **Project (chat con contexto de proyecto):** trabajo largo y evolutivo, múltiples chats, archivos que cambian, historial y memoria del proyecto.

Decidir según el comportamiento actual del producto y la necesidad concreta, no según una preferencia previa.

## 3. Evidencia y precedencia

Esta lista ordena **fuentes de evidencia para decisiones de diseño**, no autoridad de gobierno: una petición explícita del Usuario no anula por sí sola las invariantes operacionales del Orchestrator ni los protocolos adoptados. El Usuario decide objetivos, trade-offs y adopción; los protocolos gobiernan cómo el sistema implementa esas decisiones hasta que se modifiquen y se adopten por su propio gate.

Para decisiones de diseño, priorizar:

1. requisitos explícitos del Usuario;
2. comportamiento verificado en la cuenta/UI real;
3. configuración efectiva del GPT actual;
4. artifacts y Knowledge vigentes;
5. documentación oficial actual;
6. repositorios oficiales y papers relevantes;
7. inferencias declaradas como tales.

Si la documentación y el comportamiento real se contradicen: registrar la contradicción, usar la evidencia de la cuenta para decisiones operativas locales, y no generalizar ese comportamiento a otras cuentas.

Nunca presentar una inferencia como capacidad confirmada. Las capacidades del producto cambian: re-verificarlas cuando sean materialmente relevantes.

## 4. Descomposición obligatoria

Antes de diseñar, asignar cada necesidad a un dueño:

- **Instructions** — comportamiento siempre activo: rol, objetivos, límites, workflow, routing, criterios de decisión, formato. Una regla crítica de comportamiento no puede vivir únicamente en Knowledge.
- **Knowledge** — material de referencia consultado cuando es relevante: protocolos detallados, arquitectura estable, estándares especializados.
- **Capabilities / tools** — habilitar solo lo que tiene un uso real.
- **Actions / apps** — solo ante una necesidad concreta de actuar sobre sistemas externos. No agregar integración por anticipación.
- **Estado mutable del proyecto** — nunca como comportamiento global del GPT; vive en el plan/contexto propietario del proyecto.

## 5. Routing

Las Instructions llevan routing compacto hacia el Knowledge especializado:

```text
trigger observable → artifact → consulta de la versión vigente
```

No crear un router o índice separado si unas pocas reglas de routing en Instructions resuelven el problema.

## 6. Diseño de Instructions

- Instrucciones claras, positivas y concretas.
- Estructura explícita para flujos de varios pasos.
- Cada regla importante, una sola vez.
- Ejemplos solo cuando corrigen un gap observado.
- No repetir protocolos que ya tienen dueño en Knowledge.
- Mantener el prompt always-on tan lean como permita el comportamiento requerido.
- Preferir un patch pequeño y probable antes que una reescritura total.

## 7. Diseño de Knowledge

Antes de agregar un archivo:

1. ¿qué consulta concreta habilita?
2. ¿por qué no basta Instructions + Knowledge existente?
3. ¿tiene un único dueño?
4. ¿es suficientemente estable?
5. ¿su coste de mantenimiento se justifica?

Preferir: Markdown claro, headings descriptivos, filenames estables, un documento por responsabilidad, referencias a owners en lugar de copias.

Evitar: duplicar comportamiento crítico en varios archivos, documentos enormes de propósito mixto, cargar el repositorio entero como Knowledge, mezclar material público con contexto confidencial.

## 8. Baseline antes de cambios

Para evolucionar un GPT existente, registrar antes de tocarlo:

- nombre y descripción actuales;
- Instructions actuales;
- Knowledge vigente;
- capabilities habilitadas;
- Actions/apps;
- modelo recomendado;
- comportamiento observado en los casos que motivan el cambio.

No optimizar contra una impresión general si no existe un fallo identificable. Cambiar una familia de comportamiento por vez.

## 9. Verificación de un cambio

El default es ligero y dirigido:

1. partir de un **problema observable** concreto;
2. trabajar el candidate en una **branch** del repositorio, no sobre el artifact adoptado;
3. aplicar el **patch mínimo** que explica el fallo;
4. probarlo en **Preview o sandbox** (para el Orchestrator, Preview/draft; para el Worker, una sesión nueva y limpia);
5. correr un **smoke test dirigido al failure mode**, no una batería exhaustiva;
6. contrastar con **uso real** y feedback del Usuario;
7. **PASS** → adoptar; **FAIL** → rollback a la versión anterior.

Cuando compares baseline vs candidate, mantén constantes el modelo, las capabilities, el Knowledge no involucrado y los inputs. Aceptar el candidate si corrige el gap objetivo, no introduce regresión material y no aumenta complejidad sin beneficio.

Evals más formales (casos escritos con input, comportamiento esperado y hard failures) solo cuando el riesgo o la ambigüedad los justifiquen. No mantener una suite obligatoria ni automatizar la evaluación sin evidencia de que aporta valor. No crear un GPT evaluador separado salvo que supere el gate de la sección 1.

## 10. Adopción

La adopción final requiere **gate humano**.

Después de aprobar:

1. actualizar el GPT;
2. abrir una conversación nueva y verificar el comportamiento desde arranque limpio;
3. persistir la configuración adoptada en el repositorio;
4. cerrar la tarea.

## 11. Finalización

Una tarea de diseño de GPT termina cuando la necesidad está definida, la arquitectura mínima está justificada, el candidate fue probado, los gaps materiales están resueltos o explícitamente diferidos, el Usuario aprobó la adopción y la configuración quedó persistida.

No añadir mejoras retroactivamente sin un nuevo problema observable.

## 12. Base de evidencia

Este protocolo se apoya en documentación de OpenAI sobre creación y edición de GPTs, GPTs y Projects en ChatGPT, guías de prompting con prompts más lean y evaluación incremental, el Model Spec y su jerarquía de instrucciones, el Cookbook sobre el ciclo de evaluación, y en Liu et al. (TACL 2024), *Lost in the Middle*, consultados en 2026-08-26. También en el uso real del sistema descrito en este repositorio.

Las capacidades de producto cambian: verificar de nuevo antes de apoyarse en ellas para una decisión material.
