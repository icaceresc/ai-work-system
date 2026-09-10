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

## Frenar scope creep

Úsalo cuando notes que el trabajo se está expandiendo más allá de lo acordado.

```text
Esto parece salirse del objetivo. Comprueba si realmente hace falta para
cerrar esto como DONE. Si no hace falta, déjalo fuera.
No metas ideas futuras ni nice-to-haves dentro de esta misión.
```

## Cuestionar o auditar una decisión

Antes de adoptar algo o después de haberlo adoptado.

**Antes de decidir:**

```text
Antes de adoptar esto: busca evidencia en contra, alternativas reales y
señales de sobreingeniería. Dime qué suposiciones estás dando por buenas.
```

**Después de decidir:**

```text
Reconstruye qué decidimos, por qué, con qué evidencia y con qué
limitaciones o riesgos aceptados.
```

## Pedir delegación al Worker

```text
Quiero delegar esto al Worker. Madura tú la tarea y prepárale la
instrucción completa; yo no la redacto.
Dime antes: sesión, modelo, esfuerzo y modo.
Verifica el resultado antes de dármelo por bueno.
```

## Recuperar alineación

Úsalo cuando el resultado ya no coincide con lo acordado.

```text
Esto no es lo que acordamos. Vuelve a las fuentes de verdad vigentes,
identifica el punto exacto donde nos desviamos y corrige solo lo
necesario. No reconstruyas todo desde este chat.
```

## Continuar entre chats

Para cerrar un chat que se degrada o se alarga, y retomarlo donde quedó.

**Antes de cerrar:**

```text
Antes de cerrar este chat:
1. actualiza el Estado del plan que tienes en tu Conocimiento;
2. genera un handoff con lo necesario para retomar exactamente desde aquí;
3. dame el prompt inicial para arrancar el chat nuevo con ese handoff y el plan actualizado.
```

**Al retomar:**

```text
Retoma <proyecto> desde el contexto y el plan que tienes en tu Conocimiento.
No reconstruyas desde conversaciones anteriores.
Dime en 5 líneas: dónde estamos, qué gate está abierto y cuál es la
siguiente tarea.
```

## Dar más autonomía

Para tareas ya definidas, mecánicas y verificables, donde las decisiones materiales ya están tomadas.

```text
Esta parte ya está definida y es mecánica. Dale al Worker autonomía
suficiente para completarla y verificarla sin pedirme aprobación por
microdecisiones.
```

## Cerrar una fase o misión

```text
Cierra esta fase:
- qué quedó hecho y con qué evidencia;
- qué gate se aprobó;
- qué queda explícitamente fuera;
- actualiza el plan propietario y dime si la misión ya cumple sus criterios de finalización.
```
