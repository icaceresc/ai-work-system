# Orchestrator Prime — Job Packet Protocol v1

> Estado: candidato de Fase 3.2, sujeto a refinamiento por experiencia real.
> Propósito: convertir una intención ya madurada por el Orchestrator en una tarea ejecutable y verificable para Claude Code.
> Principio: un Job Packet es la salida del razonamiento del Orchestrator, no una reacción automática al último mensaje del Usuario.

## 1. Ejecución — metadata para el Usuario, NO se pega en Claude Code

Antes de entregar un Job Packet, el Orchestrator debe indicar:

| Campo | Uso |
|---|---|
| Sesión | misma sesión / nueva sesión |
| Modelo | modelo recomendado |
| Esfuerzo | nivel recomendado |
| Modo | modo de permisos recomendado |

Esta metadata la configura el Usuario fuera del prompt.

### Modos relevantes de Claude Code

- `plan`: exploración/read-only.
- `default` / Manual: solicita permisos según corresponda.
- `acceptEdits`: permite ediciones y operaciones comunes sin pedir cada edición.
- `auto`: ejecuta con comprobaciones de seguridad en segundo plano cuando está disponible.
- `dontAsk`: solo permite herramientas previamente autorizadas.
- `bypassPermissions`: omite la capa de permisos. `--dangerously-skip-permissions` es equivalente.

### Regla de selección

- auditoría/exploración → preferir `plan`;
- implementación revisada por diff → `acceptEdits` o modo corporativamente permitido;
- automatización restringida → `dontAsk` cuando exista allowlist adecuada;
- `bypassPermissions` solo cuando el Usuario lo haya habilitado conscientemente.

El modo y el Job Packet son capas distintas:
- el modo controla qué acciones permite Claude Code;
- el Job Packet controla qué intenta hacer el Worker.

Un Job Packet no es una barrera de seguridad equivalente al sistema de permisos.

## 2. Proceso previo obligatorio del Orchestrator

Antes de generar un Job Packet:

1. identificar qué quiere conseguir realmente el Usuario;
2. separar objetivo de solución propuesta;
3. contrastar la idea contra evidencia, restricciones y estado del arte cuando sea material;
4. determinar qué información falta;
5. considerar el entorno real: stack, infraestructura, herramientas disponibles, políticas y prácticas del equipo;
6. decidir si Claude Code es la herramienta adecuada;
7. delimitar la parte concreta que debe ejecutar el Worker;
8. establecer cómo se verificará el resultado.

Si falta información material, el Orchestrator pregunta antes de inventarla.

## 3. Template del Job Packet — contenido que SÍ recibe Claude Code

```text
# JOB PACKET — <proyecto>
## <tarea>

### 1. Objetivo
### 2. Contexto
### 3. Fuentes de verdad relevantes
### 4. Estado inicial esperado
### 5. Scope
### 6. Cambios permitidos
### 7. Cambios prohibidos
### 8. Nivel de autonomía
### 9. Resultado esperado
### 10. Criterios de aceptación
### 11. Verificaciones obligatorias
### 12. Stop conditions
### 13. Output requerido
```

## 4. Campos obligatorios y opcionales

Obligatorios:
- Objetivo
- Contexto suficiente
- Scope
- Cambios permitidos
- Cambios prohibidos
- Nivel de autonomía
- Resultado esperado
- Criterios de aceptación
- Verificaciones obligatorias
- Stop conditions
- Output requerido

Opcionales cuando aportan valor:
- Fuentes de verdad relevantes
- Estado inicial esperado
- archivos concretos
- baseline/commit
- secuencia exacta
- IDs
- rollback
- evidencia cuantitativa especial

No convertir opciones en burocracia.

## 5. Principios de diseño

- Reducir ambigüedad, no maximizar longitud.
- Pedir evidencia, no logs masivos.
- Usar Stop conditions contra scope creep.
- Un dueño por información; referencias/IDs antes que copias.
- Adaptarse al entorno real, no a un stack ideal.

## 6. Evidencia empírica inicial

El protocolo se deriva de Job Packets usados durante la recuperación de `scoring_micro`.

Casos cubiertos:
- auditoría read-only;
- migración estructurada de documentación;
- gobernanza del Worker;
- micro-correcciones;
- commits controlados.

Durante este proceso Claude Code fue ejecutado con `--dangerously-skip-permissions` / `bypassPermissions`.

Resultado observado:
- los scopes y límites de los Job Packets mantuvieron las tareas dentro de los cambios autorizados en las fases verificadas;
- cada resultado fue revisado por el Orchestrator con evidencia y Git.

Esto no demuestra que el Job Packet sea una barrera de seguridad: `bypassPermissions` omite la capa de permisos.

## 7. Regla para sesiones y modelos

- Mantener la misma sesión cuando el contexto acumulado es valioso.
- No cambiar de modelo en una sesión larga solo porque una tarea sea pequeña.
- Ajustar esfuerzo antes que cambiar modelo innecesariamente.
- Preferir nueva sesión cuando cambia materialmente el problema, el contexto se degrada o se quiere validar configuración global desde arranque limpio.
- Registrar siempre para el Usuario: sesión, modelo, esfuerzo y modo.

## 8. Evolución

Este archivo evoluciona con evidencia real de uso.

Cuando exista el router de Knowledge:
- este archivo será conocimiento general/transversal;
- se activará cuando haya delegación a Claude Code;
- conocimientos específicos de Data Science, Data Engineering, MVPs u otros dominios se cargarán por separado según contexto.
