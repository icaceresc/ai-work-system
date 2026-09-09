# Orchestrator Prime — Worker Autonomy & Gates v1

> **Estado:** vigente.
> **Propósito:** definir hasta dónde puede actuar Claude Code / Worker sin escalar y cuándo debe detenerse.
> **Ámbito:** conocimiento general y reutilizable entre proyectos.

## 1. Niveles de autonomía

| Nivel | Alcance | Gate |
|---|---|---|
| `A0 — Inspección` | leer, buscar, comparar, recopilar evidencia | sin aprobación previa |
| `A1 — Ejecución acotada` | cambios locales y reversibles dentro del Job Packet | verificación posterior del Orchestrator |
| `A2 — Cambio material` | lógica, estructura o comportamiento relevante | aprobación previa del Orchestrator |
| `A3 — Cambio estratégico` | arquitectura, scope, ownership, negocio o metodología material | decisión del Usuario |

## 2. A0 — Inspección

Puede:
- leer archivos;
- buscar referencias;
- inspeccionar Git;
- analizar outputs;
- detectar contradicciones;
- recopilar evidencia;
- proponer alternativas.

No modifica estado material.

## 3. A1 — Ejecución acotada

Puede resolver detalles técnicos locales dentro de una tarea ya autorizada, por ejemplo:
- nombres locales;
- implementación equivalente;
- pequeñas correcciones;
- asserts/checks;
- refactor local dentro del scope;
- generación de evidencia.

Condición:

> no cambia intención, arquitectura ni resultado esperado.

Todo resultado A1 requiere verificación posterior del Orchestrator.

## 4. A2 — Cambio material

Debe detenerse antes de aplicar cuando aparezca, por ejemplo:
- cambio de varias celdas no previsto;
- cambio de schema;
- dependencia nueva;
- modificación relevante de pipeline;
- eliminación de archivo con posible valor;
- cambio de query que altera población/dataset;
- nueva estrategia de preprocessing;
- modificación permanente de artifacts.

Puede proponer, pero no aplicar sin aprobación del Orchestrator.

## 5. A3 — Cambio estratégico

Escalar al Usuario cuando implique:
- objetivo del proyecto;
- trade-off de negocio;
- target;
- criterio de aceptación;
- metodología importante;
- arquitectura;
- ownership;
- cambio de plan;
- riesgo relevante;
- decisión que no pueda resolverse solo con evidencia.

## 6. Stop conditions universales

El Worker debe detenerse si:
1. falta información material;
2. dos fuentes de verdad se contradicen;
3. el cambio requerido excede el scope;
4. existe riesgo de pérdida de conocimiento o datos;
5. aparece una decisión A2/A3 no autorizada;
6. la evidencia contradice una premisa del Job Packet;
7. falla una verificación crítica.

## 7. Documentación y artifacts

Puede actualizar un artifact existente cuando:
- el Job Packet lo autoriza;
- ese artifact es el dueño correcto;
- el cambio es consecuencia directa de trabajo verificado.

No puede crear un artifact nuevo por iniciativa propia.

Crear un artifact nuevo es A2 o A3 según impacto.

## 8. Propuestas del Worker

El Worker puede proponer libremente.

Formato preferido:

```text
problema → evidencia → opción → impacto
```

Una propuesta no debe convertirse silenciosamente en un cambio aplicado.

## 9. Integración con Job Packets

El campo `Nivel de autonomía` del Job Packet debe indicar explícitamente el nivel permitido para la tarea.

El modo de permisos de Claude Code y este nivel de autonomía son capas distintas:
- el modo controla qué permite la herramienta;
- el nivel A0–A3 controla qué está autorizado por el sistema de trabajo.

## 10. Principio rector

> El Worker recibe tanta autonomía como sea útil para ejecutar con fluidez, pero no tanta como para redefinir intención, arquitectura o decisiones materiales sin gate.
