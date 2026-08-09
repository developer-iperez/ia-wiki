---
title: "Claude Code Guide (zebbern)"
---

Guía de referencia de una sola página para [[claude/claude-code|Claude Code]]: instalación, configuración, comandos y funciones avanzadas. Repo: [zebbern/claude-code-guide](https://github.com/zebbern/claude-code-guide).

## Qué cubre

- **Instalación y setup**: instaladores nativos (macOS/Linux/Windows), npm, Docker.
- **Configuración**: variables de entorno, ficheros de config, memoria de proyecto (`CLAUDE.md`).
- **Slash commands**: más de 50 comandos para gestionar sesiones, modelos, revisiones y workflows.
- **Interfaz**: atajos de teclado, modo Vim, edición multilínea.
- **Funciones avanzadas**: Plan Mode, Auto Mode, tareas en segundo plano, worktrees, subagentes.
- **MCP, plugins y skills**: cómo extender capacidades.
- **Revisión de código**: `/code-review`, `/simplify`, `/ultrareview`.

## Comandos/tips más destacados

- `claude` / `npx claude` — abrir REPL interactivo; `claude -p "query"` — modo no interactivo con salida JSON; `claude --continue` — retomar la última conversación.
- `"think"` / `"think hard"` / `"ultrathink"` — más presupuesto de razonamiento antes de responder; `/effort` para ajustar la profundidad de análisis; `/fast` para respuestas más rápidas en Opus.
- `/goal` (condición de fin para tareas autónomas), `/loop` (prompts recurrentes), `--worktree` (aísla cambios en una rama separada), `--bg` (tarea en segundo plano).
- `/memory` — inspeccionar/editar `CLAUDE.md`; `/agents` — dashboard de sesiones; `claude mcp list/add` — configurar MCP; `/plugins` — marketplace de extensiones.
- `/context` — ver uso de tokens; `@archivo` — mencionar archivos concretos; `.claude/rules/` — reglas modulares por directorio.

## Ver también

- [[claude/claude-code-everything|Claude Code: Everything You Need to Know]] — guía complementaria, más orientada a modelos mentales que a referencia de comandos.
- [[claude/comandos|Comandos útiles de Claude Code]]

Sources:
- [zebbern/claude-code-guide en GitHub](https://github.com/zebbern/claude-code-guide)
