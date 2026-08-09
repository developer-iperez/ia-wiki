---
title: Comandos útiles de Claude Code
---

Resumen de los comandos más útiles del día a día con [[claude/claude-code|Claude Code]], separados entre los que se lanzan desde la terminal (antes de entrar en una sesión) y los slash commands que se usan dentro de una sesión ya iniciada.

## Terminal

- `npm i -g @anthropic-ai/claude-code` — instalación vía npm (alternativa al instalador nativo `curl -fsSL https://claude.ai/install.sh | bash`).
- `claude --version` — comprobar la versión instalada.
- `claude` — arrancar una sesión interactiva en el directorio actual.
- `claude "prompt"` — arrancar una sesión con un mensaje inicial ya escrito.
- `claude -p "pregunta"` — modo *print*: responde una vez y termina, sin sesión interactiva (útil para scripts).
- `claude -c` — continuar la sesión más reciente de este directorio.
- `claude -r "<id>"` — retomar una sesión concreta por su ID.

## Slash commands (dentro de una sesión)

- `/help` — lista todos los comandos disponibles.
- `/clear` — limpia el historial de la conversación (útil cuando Claude empieza a mezclar contexto de tareas anteriores).
- `/compact` — comprime los mensajes anteriores en un resumen para liberar contexto sin perder el hilo, cuando te acercas al límite.
- `/context` — muestra cuánto contexto se está usando.
- `/model` — cambia de modelo (p. ej. a uno más barato/rápido si el coste importa).
- `/permissions` — consulta o cambia los permisos de la sesión (lo mismo que se puede fijar de antemano en `.claude/settings.json`).
- `/memory` — edita el `CLAUDE.md` del proyecto directamente desde la sesión.
- `/review` — pide una revisión de código sobre los cambios actuales.
- `/cost` — muestra el consumo de tokens/coste de la sesión; conviene revisarlo en sesiones largas, porque un bucle agéntico puede gastar más de lo esperado.
- `/status` — estado de la cuenta.
- `/agents` — gestiona subagentes especializados.
- `/mcp` — gestiona los servidores MCP conectados (herramientas externas: Google Drive, Jira, Slack, etc.).

## Comandos personalizados

Se pueden crear comandos propios añadiendo ficheros en `~/.claude/commands/` (o `.claude/commands/` a nivel de proyecto): cada fichero se convierte en un slash command nuevo, lo que permite convertir flujos de trabajo repetitivos en una sola invocación.

Para capacidades más elaboradas que un simple comando (con sus propias instrucciones y activación automática por contexto), ver [[claude/skills|Skills de Claude]].

Sources:
- [CLAUDE CODE CLI — COMANDOS VERIFICADOS (Jun 2026)](https://gist.github.com/IoTeacher/292ef3e9cf11414ef968e1ec44ed886b)
- [Claude Code Slash Commands 2026: Complete List + Custom Commands](https://www.heyuan110.com/posts/ai/2026-03-05-claude-code-slash-commands/)
- [Claude Code Commands: A Practical Guide for 2026 | DataCamp](https://www.datacamp.com/tutorial/claude-code-slash-commands)
