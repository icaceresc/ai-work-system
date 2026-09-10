---
artifact_id: model_development_protocol
artifact_version: 1
artifact_type: knowledge
owner: orchestrator_prime
status: adopted
---

# Model Development Protocol

> **Propósito:** desarrollar modelos de forma incremental, reproducible, estadísticamente válida y comprensible para quien firma el resultado.
> **Ámbito:** modelado, ML y experimentación predictiva. Overlay especializado.
> **Principio:** el modelo no es el problema; la decisión que apoya, sí.

Este protocolo **complementa** `development_protocol.md`. No lo reemplaza ni lo repite.

```text
desarrollo de modelos = development_protocol + model_development_protocol
```

Del protocolo general se heredan sin cambios: el workflow INTENT → DONE, la spec proporcional, la disciplina de baseline y branch, el TDD proporcional, la calidad de código, los gates humanos, el aprendizaje y accountability del Usuario, y los criterios de completion. Aquí se añade únicamente lo que el modelado necesita y el desarrollo general no cubre.

No prescribe algoritmo, framework, métrica ni estrategia de split.

## 1. Secuencia de modelado

```text
PROBLEMA / DECISIÓN
→ TARGET + UNIDAD DE OBSERVACIÓN
→ ENTENDIMIENTO DE DATOS
→ DISEÑO DE VALIDACIÓN / SPLIT
→ BASELINE
→ PREPROCESSING / FEATURES
→ ENTRENAMIENTO
→ EVALUACIÓN
→ ANÁLISIS DE ERROR
→ ITERAR UN CAMBIO MATERIAL
→ REPRODUCIBILIDAD / TRAZABILIDAD
→ EVIDENCIA FINAL
```

Es una secuencia de razonamiento, no una plantilla de plan. No convertir cada bloque en una fase formal: el plan concreto decide qué etapas necesita y con qué granularidad (`planning_protocol.md` sigue siendo dueño de los planes).

Las etapas anteriores a BASELINE son decisiones, no código. Saltarlas para empezar a entrenar antes es la forma más común y más cara de equivocarse en modelado: un modelo bien entrenado sobre un problema mal definido no se arregla con hyperparameters.

## 2. Problema antes del modelo

Antes de elegir algoritmo, aclarar:

- qué decisión o problema se quiere apoyar;
- qué representa una predicción;
- cuál es la población relevante;
- cuál es la unidad de observación —qué es una fila;
- cuál es el target o label, cuando aplique;
- cuál es el horizonte temporal, cuando aplique;
- cuándo estará disponible cada feature respecto del momento de predicción;
- qué tipo y coste de error importa;
- qué criterio hace útil el resultado.

No son decisiones de conveniencia técnica. Target, población, unidad de observación y definición del problema son decisiones materiales: si la evidencia no las resuelve, gate con el Usuario —nivel `A3` en `worker_protocol.md`. El Worker no elige target ni población porque una opción sea más fácil de consultar o de entrenar.

La pregunta de disponibilidad temporal —«¿esta feature existe en el momento en que habría que predecir?»— se responde antes de construir features, no después de ver una métrica sospechosamente buena.

## 3. Datos, validación y leakage

El diseño de validación debe representar cómo se usará el modelo. Es una decisión metodológica material: el Orchestrator debe poder explicar por qué el split elegido representa el uso futuro.

No imponer split aleatorio. Elegir según la estructura real de los datos: temporal, out-of-time, por grupo o entidad, aleatorio, u otra.

**Frontera de aprendizaje.** Cuando aplique a aprendizaje supervisado:

- separar train / validation / test antes de aprender cualquier transformación que pudiera usar información no permitida por el diseño;
- ajustar (`fit`) preprocessors, imputers, scalers, encoders y selectores de features solo con los datos que el diseño de validación permite;
- aplicar después esas transformaciones al resto;
- preferir pipelines o mecanismos equivalentes que mantengan la frontera por construcción, en lugar de por disciplina manual.

**Buscar leakage explícitamente** antes de confiar en cualquier métrica:

- target leakage: features que contienen o derivan del target;
- leakage temporal: información posterior al momento de predicción;
- leakage de entidad o grupo: la misma entidad repartida entre train y test cuando eso no ocurrirá en uso real;
- leakage de preprocessing: estadísticas aprendidas con datos reservados;
- duplicados o casi-duplicados repartidos entre splits, cuando sea material.

Una métrica excelente sin explicación plausible es sospecha de leakage, no un logro. Explicarla antes de celebrarla.

Reportar el tamaño del dataset usado y, cuando exista un campo temporal relevante, su rango observado. Un resultado del que no se sabe sobre cuántos datos ni sobre qué periodo se calculó no es interpretable.

## 4. Baseline antes de complejidad

Antes de un modelo complejo, definir un baseline defendible: una heurística, una predicción constante o naive, un modelo simple, la solución actualmente en uso, u otro comparador razonable.

El baseline existe para responder una pregunta: **¿la complejidad adicional compra valor?**

- No adoptar un modelo más complejo solo porque existe o porque está disponible en la librería.
- La complejidad debe justificar una mejora material según el criterio del proyecto, no una diferencia marginal en una métrica elegida después de ver los resultados.
- Si el baseline ya es suficiente para la decisión que se quiere apoyar, ese es un resultado válido y hay que reportarlo como tal.

## 5. Métricas y evaluación

La métrica se elige antes de ver los resultados y se justifica por el problema, no por el modelo.

Relacionarla con: tipo de problema, distribución del target, coste relativo de cada tipo de error, uso previsto y decisión real que se apoya.

Cuando varias vistas son relevantes —por ejemplo el error global y el error en el segmento que importa—, ninguna métrica única gobierna la decisión automáticamente.

Separar tres cosas al reportar:

| Nivel | Contenido |
|---|---|
| **Hecho** | el resultado medido |
| **Inferencia** | por qué se cree que ocurrió |
| **Decisión** | si la mejora justifica la adopción |

El test final sirve para estimar el desempeño, no para elegir hyperparameters ni features. Cada vez que se decide algo mirando el test, ese test deja de ser una estimación honesta.

## 6. Experimentación incremental

```text
baseline → UN cambio material → ejecutar → comparar → interpretar → decidir → siguiente cambio
```

Cambiar varias dimensiones a la vez impide atribuir el efecto y convierte una mejora en una coincidencia sin explicación.

Dimensiones materiales típicas: feature set, preprocessing, algoritmo, estrategia de hyperparameters, sampling, definición del target.

Target, población y split no son una dimensión más: son decisiones de diseño con gate (§ 2, § 3) y cambiarlas invalida la comparabilidad de todo lo anterior.

Registrar los resultados negativos cuando conocerlos evita repetir el intento. No construir burocracia de tracking cuando no aporta.

## 7. Modo guiado: unidad lógica

En desarrollo inicial o exploratorio, donde importan el aprendizaje y la auditabilidad, avanzar por la **unidad lógica mínima** que el Usuario pueda entender, ejecutar o revisar, interpretar y defender.

Frecuentemente esa unidad es una celda de notebook. No es una regla de «exactamente una celda»: si una unidad coherente necesita varias, son varias; si una sola celda contiene dos decisiones distintas, son dos unidades.

```text
1. El Orchestrator define y, si hay decisión material, debate la siguiente unidad.
2. El Usuario entiende y aprueba la intención cuando la decisión es material.
3. El Worker implementa solo esa unidad o el bloque autorizado.
4. Se ejecuta y se verifica.
5. El Worker devuelve evidencia compacta.
6. El Orchestrator interpreta el resultado con el Usuario.
7. Se avanza a la siguiente unidad cuando el gate correspondiente está pasado.
```

No generar en silencio un notebook completo para pedirle después al Usuario que lo audite hacia atrás. La excepción es una tarea explícitamente autorizada como mecánica o reconstructiva.

## 8. Autonomía proporcional a la madurez

El modo guiado no se congela para siempre.

| Situación | Granularidad y gates |
|---|---|
| Exploración, diseño, aprendizaje: alta incertidumbre y decisiones que el Usuario necesita comprender | unidades pequeñas, gates frecuentes |
| Pipeline ya acordado o tarea mecánica: comportamiento definido y verificable | más autonomía al Worker dentro del Job Packet |

Ejemplos de tarea mecánica: volver a ejecutar un pipeline acordado, una actualización repetitiva, generar outputs ya especificados, un refactor equivalente, una extracción ya definida.

Principio: la granularidad y la autonomía dependen del riesgo, la incertidumbre, la reversibilidad y la necesidad de aprendizaje. No del número de líneas ni de celdas.

## 9. Reproducibilidad y auditabilidad

El Usuario debe poder reconstruir:

- qué código produjo el resultado;
- qué datos, snapshot o versiones se usaron, cuando sea posible determinarlo;
- parámetros y configuración;
- diseño de split y validación;
- métricas y resultados;
- decisiones metodológicas materiales;
- qué cambió frente al baseline.

Preferir los mecanismos que ya existen en el repositorio o el entorno antes de introducir uno nuevo.

Un sistema de experiment tracking puede aportar parámetros, versión del código, métricas y outputs. No es obligatorio: no lo es si no existe en el entorno, si no aporta valor material, o si su coste supera la trazabilidad que da. Introducirlo es una decisión de adopción con su gate.

Git no garantiza la reproducibilidad de datos externos: versiona el código, no el contenido de una base de datos ni el estado de un sistema remoto en el momento de la consulta. Cuando el dato importa, registrar cómo se obtuvo y cuándo.

## 10. Tests: software determinista vs calidad estadística

Son dos dominios y se verifican distinto.

**Software determinista alrededor del modelo.** Aplicar el TDD proporcional de `development_protocol.md` cuando sea razonable: ingestión, transformación, feature engineering, validación de schemas y contratos, preprocessing, serialización, helpers, interfaces de scoring, integración del pipeline y bugs reproducibles.

**Calidad estadística del modelo.** No fingir TDD clásico. Se valida con: diseño de evaluación correcto, splits que representan el uso, baseline, métricas justificadas, umbrales o criterios acordados de antemano cuando corresponda, análisis de error y robustez, y comparación reproducible.

Un modelo no «pasa» porque un unit test compare su accuracy contra un número elegido a mano. Ese test comprueba que el número no cambió; no comprueba que el modelo sirva. Puede ser útil como detector de regresión, nunca como criterio de validez.

## 11. Pipelines

Cuando el proceso experimental se estabiliza, conviene convertir los pasos repetibles en un pipeline reproducible: reduce error manual y retrabajo.

```text
entender → probar de forma transparente → estabilizar → encapsular y reutilizar
```

No abstraer una exploración que todavía cambia: encapsular demasiado pronto esconde exactamente lo que hay que mirar.

Con un pipeline estable, cambiar un componente y volver a ejecutar debe ser una operación controlada y comparable, no rehacer el proceso a mano.

## 12. Defendibilidad

Para cada decisión metodológica material, conservar evidencia suficiente para responder:

- qué se decidió;
- qué alternativas reales se consideraron;
- qué evidencia la sustentó;
- qué assumptions quedan abiertas;
- cómo afecta eso a la validez del resultado.

Esa evidencia se reparte según quién sea el dueño correcto: el notebook, el código, la configuración, el sistema de tracking, los artifacts del proyecto o el propio Job Packet. No copiar la misma explicación en todos.

No producir documentación ceremonial. La prueba es funcional: si el Usuario puede responder esas preguntas sin reconstruir la historia desde un chat, la evidencia es suficiente.

## 13. Principio rector

> Un resultado que no se puede explicar no se puede adoptar.

Problema antes que modelo. Validación antes que métrica. Baseline antes que complejidad. Un cambio material por vez. Y la unidad de avance es la que el Usuario puede entender y defender.
