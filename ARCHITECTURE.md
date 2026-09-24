# AI Work System — Architectural North Star

AI Work System define una metodología portable para convertir intención humana en trabajo técnico verificable. Este documento describe su dirección arquitectónica; no sustituye los protocolos operativos ni afirma que todas sus primitivas futuras ya estén implementadas.

```text
                    AI WORK SYSTEM

                    ┌───────────┐
                    │   Human   │
                    └─────┬─────┘
                          │
                  responsibility
                          │
                    ┌─────▼─────┐
                    │Orchestrator│
                    └─────┬─────┘
                          │
              ┌───────────┴───────────┐
              │                       │
          Skills                  Capabilities
      "how to work"             "what I can do"
              │                       │
              │                 Deployment Adapter
              │                       │
              │              ┌────────┴────────┐
              │              │                 │
              │           Native             MCP
              │           tools             servers
              │
              └────────── Worker(s)
                           │
                           ▼
                       Evidence
                           │
                           ▼
                       Auto Evals
                           │
                     PASS / Gate
```

Los harnesses y runtimes son intercambiables: Codex / Claude / ChatGPT / LangGraph-based systems / future runtimes.

## Principios

- **Human responsibility.** El humano conserva responsabilidad, criterio y decisiones materiales.
- **Orchestrator.** Traduce intención en trabajo delimitado, coordina, verifica y escala los gates humanos.
- **Skills.** Representan workflows portables: “cómo trabajar”.
- **Capabilities.** Representan capacidades lógicas: “qué puede hacerse”.
- **Deployment Adapter.** Adapta el CORE agnóstico al entorno real. Un deployment puede poseer sus runtimes, secrets, tools, resources, permisos y bindings concretos.
- **Native / MCP.** Una capability puede exponerse mediante herramientas nativas u otros bindings. MCP es un binding/protocolo candidato importante, no una dependencia obligatoria del CORE.
- **Workers.** Ejecutan trabajo técnico delimitado; no redefinen por sí mismos intención, arquitectura ni metodología.
- **Evidence.** El trabajo debe producir evidencia verificable.
- **Auto Evals.** La dirección es automatizar verificaciones y evaluaciones donde sea razonable, escalando al humano solo las decisiones que requieren criterio.
- **Harness neutrality.** AI Work System no depende de Codex, Claude, ChatGPT, LangGraph ni de otro runtime particular.

## CORE y deployments

El CORE define reglas y contratos portables. Cada deployment conecta las capabilities y resources propias de su entorno; sus detalles concretos —incluidos sistemas, secretos, permisos y bindings— no pertenecen al CORE. Por ello, una implementación de deployment puede variar sin modificar la metodología central. La analogía de un hub USB describe solo esta separación: el CORE mantiene el contrato y cada deployment conecta lo que le corresponde.

## Estado actual y North Star

**Implemented today:** roles Human/Orchestrator/Worker, protocols, project control, deployment adapter pattern y trabajo basado en evidencia.

**North Star / evolving primitives:** portable Skills, logical capabilities/resources, automated evals y MCP bindings.

Este North Star no afirma que exista hoy un capability registry formal, un MCP server, un generic eval framework, un portable Skill ya adoptado ni bootstrap automático de deployments.
