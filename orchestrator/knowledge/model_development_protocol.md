---
artifact_id: model_development_protocol
artifact_version: 2
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

Optimiza **tiempo hasta una decisión defendible**, no cantidad de análisis ni completitud del artifact. La fiabilidad vive en la evidencia; la simplicidad, en la interfaz que consume el humano.

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

**Primera unidad cuando el problema aún no está definido.** Si target, población, unidad de observación, horizonte o momento de predicción no están suficientemente establecidos, todavía no se delega cleaning material, split, feature engineering, baseline, entrenamiento ni tuning: cualquiera de esos trabajos codifica una definición del problema que nadie adoptó.

Si en ese punto hace falta un Worker, la primera delegación es preferentemente **read-only, de recopilación de evidencia**:

- inspeccionar el notebook o el código existente;
- inspeccionar la documentación y los datos disponibles;
- identificar qué está ya comprobado, y con qué evidencia;
- identificar las ambigüedades que quedan abiertas;
- detectar variables post-evento o candidatas a leakage;
- devolver evidencia, sin decidir.

Después, el Orchestrator y el Usuario interpretan esa evidencia y adoptan la definición del problema. Recién entonces empieza el trabajo que depende de ella.

## 3. Datos, validación y leakage

El diseño de validación debe representar cómo se usará el modelo. Cuando afecta la validez de la evaluación es una decisión metodológica material: el Orchestrator debe poder explicar por qué el split elegido representa el uso futuro, y su gate precede a su implementación.

No imponer split aleatorio. Elegir según la estructura real de los datos: temporal, out-of-time, por grupo o entidad, aleatorio, u otra.

Mientras existan alternativas materiales sin resolver, el Worker puede:

- inspeccionar la estructura temporal, de grupo o de entidad de los datos;
- medir cardinalidades, volúmenes y distribuciones;
- proponer alternativas con sus consecuencias;
- aportar la evidencia que permita decidir.

Lo que no debe hacer es implementar una estrategia de split —ni el código que la presupone— antes de que el Orchestrator y el Usuario adopten la decisión. Un split implementado «provisionalmente» se convierte en el split adoptado por inercia y contamina todo lo que se mida después.

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

### EDA orientado a decisión

El EDA existe para **reducir incertidumbre que afecta una decisión posterior**, no para producir un catálogo exhaustivo de tablas y gráficos.

Antes de cada bloque exploratorio, formular qué pregunta responde y qué decisión podría cambiar con su resultado. Si el análisis no puede cambiar una decisión vigente, justificar por qué sigue siendo necesario o dejarlo fuera del critical path.

Dimensiones frecuentes —no checklist obligatorio—:

- población, cobertura y representatividad;
- calidad y observabilidad del target;
- distribuciones, missingness y anomalías con impacto material;
- relación histórica entre variables candidatas y target;
- redundancia o dependencia entre variables;
- drift o cambios temporales relevantes;
- evidencia necesaria para target, población, split, features o evaluación.

Preferir comparaciones con denominadores explícitos, cortes temporales consistentes y reglas congeladas cuando la comparación histórica dependa de ellas.

Cuando una figura comunica mejor el patrón, preferirla sobre una tabla extensa. Mantener tablas cuando el valor exacto sea la evidencia que importa. No imprimir outputs grandes solo porque fueron útiles para depurar.

## 4. Baseline antes de complejidad

Antes de un modelo complejo, definir un baseline defendible: una heurística, una predicción constante o naive, un modelo simple, la solución actualmente en uso, u otro comparador razonable.

El baseline existe para responder una pregunta: **¿la complejidad adicional compra valor?**

- No adoptar un modelo más complejo solo porque existe o porque está disponible en la librería.
- La complejidad debe justificar una mejora material según el criterio del proyecto, no una diferencia marginal en una métrica elegida después de ver los resultados.
- Si el baseline ya es suficiente para la decisión que se quiere apoyar, ese es un resultado válido y hay que reportarlo como tal.

## 5. Métricas y evaluación

La métrica se elige antes de ver los resultados y se justifica por el problema, no por el modelo.

Relacionarla con: tipo de problema, distribución del target, coste relativo de cada tipo de error, uso previsto y decisión real que se apoya.

**El orden importa.** Primero el problema, el uso previsto y el tipo y coste de error que importa. Solo después:

- la métrica o métricas primarias;
- las métricas diagnósticas realmente necesarias;
- la estrategia de threshold, cuando la decisión lo requiera.

No imponer como checklist universal el paquete convencional del tipo de problema —ROC AUC, precision, recall, F1, PR AUC, matriz de confusión y equivalentes—: es la respuesta por defecto de la literatura, no una decisión sobre este problema. Preseleccionarlo antes de saber qué error cuesta más es elegir la métrica por el modelo, precisamente lo que esta sección prohíbe. Cuestionar ese default no significa sustituirlo por otro paquete fijo.

El Worker puede calcular métricas exploratorias cuando ayuden a decidir. Una lista convencional calculada por comodidad no se convierte por eso en criterio de aceptación.

Cuando varias vistas son relevantes —por ejemplo el error global y el error en el segmento que importa—, ninguna métrica única gobierna la decisión automáticamente.

Separar tres cosas al reportar:

| Nivel | Contenido |
|---|---|
| **Hecho** | el resultado medido |
| **Inferencia** | por qué se cree que ocurrió |
| **Decisión** | si la mejora justifica la adopción |

El test final sirve para estimar el desempeño, no para elegir hyperparameters ni features. Cada vez que se decide algo mirando el test, ese test deja de ser una estimación honesta.

## 6. Experimentación incremental

### Throughput analítico y criterio de parada

El objetivo operacional es llegar rápido a una **decisión suficientemente sustentada**, sin sacrificar validez.

```text
pregunta concreta
→ evidencia mínima fiable
→ representación útil
→ interpretación
→ decisión / gate
→ siguiente pregunta
```

- Rigor proporcional al impacto potencial de estar equivocado.
- Una excepción merece tiempo proporcional a su capacidad de cambiar la decisión.
- Alcanzada evidencia suficiente para el gate vigente, decidir y avanzar; no perfeccionar el artifact por inercia.
- No añadir métricas, visualizaciones, features, infraestructura o análisis porque «podrían ser interesantes» si no pueden cambiar una decisión de la misión.
- Velocidad no significa saltar contratos, leakage checks o validación. Significa evitar trabajo que no compra información accionable.
- Si una nueva evidencia contradice una decisión adoptada, reabrir el gate correspondiente; no proteger la velocidad a costa de ignorar señal material.

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

**Frontera de la unidad.** El tamaño de una unidad no lo fija solo lo que el Usuario puede seguir: lo fija el próximo gate metodológico. Una unidad lógica **no puede cruzar una decisión metodológica material que todavía no está adoptada**. Puede contener varias acciones o celdas técnicas coherentes, pero termina **antes** del siguiente gate.

```text
problema / target / unidad de observación / horizonte
→ GATE

diseño de validación / split
→ GATE

definición del baseline
→ implementación + evidencia

experimento de feature o de modelo
→ evidencia + interpretación
```

Es un ejemplo de dónde caen los gates en un caso típico, no una lista de fases obligatorias: `planning_protocol.md` sigue siendo dueño de los planes y el plan concreto decide su estructura.

Cuando las decisiones metodológicas relevantes ya están adoptadas, esta regla no restringe nada: no quedan gates por cruzar (§ 8).

```text
1. El Orchestrator define y, si hay decisión material, debate la siguiente unidad.
2. El Usuario adopta la decisión material que abre esa unidad; que el Worker considere resuelta la ambigüedad no sustituye esa adopción.
3. El Worker implementa solo esa unidad o el bloque autorizado.
4. Se ejecuta y se verifica.
5. El Worker devuelve evidencia compacta.
6. El Orchestrator interpreta el resultado con el Usuario.
7. Se avanza a la siguiente unidad cuando el gate correspondiente está pasado.
```

No generar en silencio un notebook completo para pedirle después al Usuario que lo audite hacia atrás. La excepción es una tarea explícitamente autorizada como mecánica o reconstructiva.


### Notebook human-first

Cuando el notebook sea un artifact humano, su función primaria es ser una **interfaz ejecutable del análisis**, no la memoria del agente, un log de depuración ni un expediente de auditoría.

Patrón preferido para una unidad visible:

```text
PREGUNTA
→ CÓDIGO ACOTADO
→ FIGURA / OUTPUT ÚTIL
→ INTERPRETACIÓN BREVE
```

No es una plantilla rígida, pero sí una prueba de diseño: un lector competente debe poder recorrer el notebook y entender por qué existe cada bloque.

Reglas:

- una celda o bloque visible debe tener una función concreta;
- preferir gráficos, tablas compactas y captions breves cuando reduzcan carga cognitiva;
- definir un término técnico cuando sea necesario para interpretar el resultado, no antes por completitud;
- evitar Markdown largo que preserve historia de depuración, decisiones ya cerradas, excepciones de bajo impacto o defensas metodológicas repetidas;
- checks de integridad que no sean parte del argumento analítico deben ser silenciosos (`assert`, tests o equivalente) o resumirse de forma compacta;
- no mostrar grandes tablas intermedias si el lector no necesita inspeccionarlas;
- la complejidad de ejecución puede vivir debajo; la complejidad cognitiva no debe filtrarse innecesariamente al lector.

El notebook puede seguir siendo totalmente reproducible sin contener toda la memoria reconstructiva del trabajo.

**Granularidad del Job Packet.** Con el modo guiado activo, un Job Packet inspecciona o implementa solo hasta el próximo gate material. Un Job Packet demasiado amplio no se corrige agregándole stop conditions: las stop conditions son guardrails para lo que no se previó, no un sustituto de dividir bien el trabajo. Si el Job Packet necesita una stop condition para no cruzar un gate que ya se sabe que está ahí, ese gate es el final del Job Packet.

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

### Companion analítico opcional

Si un notebook humano empieza a cargarse con detalle reconstructivo que sí tiene valor para continuidad, Orchestrator o Worker, se puede separar un companion analítico —por ejemplo `*.analysis.md`— **solo cuando resuelva ese problema observable**.

Ese companion:

- vive junto al artifact analítico o donde el repo concreto defina, no convierte `project/` en un contenedor genérico;
- no es plan, backlog ni Project control file;
- es reconstructivo y mutable, no un diario cronológico;
- puede conservar decisiones analíticas, mapa de evidencia, hallazgos de data quality, caveats y referencias a SQL/código;
- referencia al owner autoritativo cuando la información ya pertenece a otro artifact;
- no duplica contexto, plan, spec ni historia de chat.

Si el notebook sigue siendo claro sin companion, no crear uno.

### Verificación proporcional al cambio

Verificar exhaustivamente **lo que cambió y su closure mínima de dependencias**. No exigir una re-ejecución total por ritual cuando artifacts congelados, fingerprints, manifests u otras evidencias ya demuestran que lo no tocado permanece igual.

Una verificación mínima válida puede ser:

```text
estado limpio
→ dependencias necesarias
→ unidad modificada completa
→ invariantes relevantes
→ diff / schema / artifact checks
```

Re-ejecutar notebook o pipeline completo cuando el cambio pueda afectar materialmente otras partes, cuando la evidencia congelada no sea suficiente o cuando el criterio de aceptación lo exija.

La verificación proporcional reduce coste sin reducir responsabilidad: si la closure mínima no puede demostrar la afirmación que se quiere hacer, ampliar la verificación.

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

La evidencia y la explicación no tienen que vivir en el mismo lugar. **Source → transformación → dataset → figura → claim** debe poder reconstruirse, aunque la interfaz humana muestre solo la parte necesaria para comprender y defender el claim.

## 13. Principio rector

> Un resultado que no se puede explicar no se puede adoptar.

Problema antes que modelo. Validación antes que métrica. Baseline antes que complejidad. Un cambio material por vez. Y la unidad de avance es la que el Usuario puede entender y defender.
