# Cheatsheet — prompts para copiar y pegar

Para el Usuario. Bloques cortos que realinean al Orchestrator o fuerzan un paso concreto. Copiar el que haga falta y pegarlo tal cual.

## Realinear la forma de trabajo

Úsalo cuando las respuestas se vuelven largas, dispersas o dejan de apoyarse en evidencia.

```text
Realinea tu forma de trabajo:
- respuestas compactas, conclusión primero;
- no me entregues todo de una vez, avanza en pasos pequeños;
- distingue hecho comprobado, inferencia, suposición y recomendación;
- si falta información material para decidir, pregúntame antes de asumir;
- una sola siguiente tarea aplicable por vez.
```

## Cuestionar el scope

Úsalo cuando notes que el trabajo se está expandiendo.

```text
Antes de seguir: ¿esto sigue dentro del objetivo del plan vigente?
Dime qué parte es scope original, qué parte es scope nuevo, y qué pasa si dejamos fuera lo nuevo.
Si es una misión distinta, propón cerrarla como plan aparte en vez de extender este.
```

## Pedir delegación al Worker

```text
¿Esta tarea conviene delegarla al Worker?
Si sí, prepárame el Job Packet e indícame antes: sesión, modelo, esfuerzo y modo.
```

## Revisar el estado del plan

```text
Lee el plan que tienes en tu Conocimiento.
Actualiza su Estado hasta lo último que hemos trabajado. Solo el Estado, no el contenido del plan.
Devuélveme el archivo actualizado para sobreescribirlo en tu Conocimiento.
```

## Retomar un proyecto en un chat nuevo

```text
Chat nuevo. Retoma <proyecto> desde el contexto y el plan que tienes en tu Conocimiento.
No reconstruyas desde conversaciones anteriores.
Dime en 5 líneas: dónde estamos, qué gate está abierto y cuál es la siguiente tarea.
```

## Handover a un chat nuevo

Úsalo cuando el chat se degrada o se vuelve demasiado largo.

```text
Antes de cerrar este chat:
1. actualiza el Estado del plan que tienes en tu Conocimiento;
2. genera un handoff con lo necesario para retomar exactamente desde aquí;
3. dame el prompt inicial para arrancar el chat nuevo con ese handoff y el plan actualizado.
```

## Cerrar una fase o un proyecto

```text
Cierra esta fase:
- qué quedó hecho y con qué evidencia;
- qué gate se aprobó;
- qué queda explícitamente fuera;
- actualiza el plan propietario y dime si el plan ya cumple sus criterios de finalización.
```
