---
title: "Claude Code: Everything You Need to Know"
---

Guía práctica de [[claude/claude-code|Claude Code]] centrada en **modelos mentales claros**, desde el setup hasta la orquestación multi-agente. Repo: [wesammustafa/Claude-Code-Everything-You-Need-to-Know](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know).

## Qué cubre

1. **Fundamentos**: instalación, autenticación, primeros prompts.
2. **Prompt engineering**: flujo Explorar → Planificar → Codificar → Commit, desarrollo guiado por tests, niveles de esfuerzo.
3. **Slash commands**: comandos integrados y creación de skills personalizadas.
4. **Skills**: plantillas de workflow reutilizables; visión general del ecosistema.
5. **Hooks**: automatización del ciclo de vida (pre/post uso de herramientas, eventos de sesión).
6. **Subagentes y trabajo en paralelo**: git worktrees, agentes especializados, orquestación.
7. **Agent Teams**: función experimental para coordinar varios especialistas.
8. **Workflows dinámicos**: orquestación de agentes a gran escala mediante JavaScript.
9. **Model Context Protocol (MCP)**: integraciones universales con herramientas externas.
10. **Fast Mode, frameworks y FAQ**: optimización de rendimiento y material de referencia.

## Modelos mentales clave

- **Los 5 puntos de extensión se combinan entre sí**:
  - **Skills** — para prompts que se repiten 3 o más veces.
  - **Hooks** — para disparos automáticos del ciclo de vida.
  - **Subagentes** — para subtareas aisladas en paralelo.
  - **Workflows** — para coordinación multi-agente compleja.
  - **Servidores MCP** — para acceso a herramientas externas.
- **El "esfuerzo" (`/effort`) como dial de comportamiento**, no solo un presupuesto de tokens: cambia la profundidad de pensamiento, el apetito por usar herramientas y la longitud de la respuesta, desde `low` hasta `ultracode`.
- **El "problema N×M" que resuelve MCP**: sin un estándar común, conectar *n* apps con *m* herramientas da *n×m* integraciones frágiles; MCP lo convierte en una arquitectura modular *N+M* (la misma idea que las Web APIs o LSP aplicada a agentes).
- Insiste en **priorizar la calidad del contexto sobre el cómputo bruto**, con ejemplos reales tomados del propio directorio `.claude/` del repositorio.

## Ver también

- [[claude/claude-code-guide-zebbern|Claude Code Guide (zebbern)]] — guía complementaria, más orientada a referencia rápida de comandos.
- [[claude/skills|Skills de Claude]]
- [[claude/comandos|Comandos útiles de Claude Code]]

Sources:
- [wesammustafa/Claude-Code-Everything-You-Need-to-Know en GitHub](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know)
