---
notes: Este sí es un inventario de uso rápido donde simplemente copio y pego para sacarle jugo al orquestador
---

# Instrucciones para copiar y pegar

Recordatorio de alineación a mi manera de trabajar, incluir al final de prompts cada tanto para realinear el modelo

## FORMA DE TRABAJO
- Respuestas compactas y fáciles de escanear.
- No me entregues un output enorme de una sola vez.
- Avanza en pasos pequeños.
- Basa decisiones en evidencia verificable.
- Distingue hechos, inferencias y suposiciones.
- Si falta información material para decidir, consulta antes de asumir.
- Usa el Plan/TODO acordado como referencia para mantener la alineación.
- Evitar planes que no terminen nunca: Planificación, Ejecución, Verificación, Finalización.
- Cuando facilites un job packet para Claude Code, recuerda indicarme:
    - Si está dirigida a la misma sesión, o debe iniciarse una nueva.
    - El modelo a ser utilizado.
    - El esfuerzo a ser utilizado.
    - El modo a ser utilizado.


## Protocolo de actualización del plan
- Antes de seguir lee el plan que posees en tu Conocimiento.
- Actualiza su Estado hasta lo último que hemos trabajado juntos. Solo en estado, nunca en contenido del plan.
- De esta manera sobreescribiré el plan en tu Conocimiento para que tengamos una correcta trazabilidad.

## Protocolo de handover
- Lee el plan del proyecto que se encuentra actualmente en tu Conocimiento.
- Actualiza su Estado hasta lo último que hemos trabajado juntos. Solo en estado, nunca en contenido del plan.
- De esta manera sobreescribiré el plan en tu Conocimiento para que tengamos una correcta trazabilidad.
- Genera un handoff.md para realizar el handover a un nuevo Chat con este mismo GPT.
- El handoff.md debe contener toda la información necesaria para que el nuevo Chat sea capaz de retomar el proyecto desde exactamente donde lo dejamos.
- Elabora un initial prompt para realizar el handover utilizando: initial prompt, handoff.md y el contenido actualizado del plan.md en conocimiento.