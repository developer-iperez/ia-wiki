---
title: OpenCode
---

Notas sobre OpenCode V2, el agente de codificación open source de [Anomaly](https://opencode.ai/). Funciona como interfaz de terminal (TUI), aplicación de escritorio, interfaz web y CLI, y comparte un mismo servidor en segundo plano para sesiones, configuración, permisos y herramientas.

Esta sección sigue la misma estructura que la de [[claude/index|Claude]]: fundamentos, mecanismos de extensión y guías prácticas.

### [[opencode/fundamentos/index|Fundamentos]]

- [[opencode/fundamentos/opencode|Qué es OpenCode]] — instalación, interfaces, arquitectura cliente-servidor y primer arranque.
- [[opencode/fundamentos/cli-tui|CLI y TUI]] — comandos de terminal, atajos, sesiones y automatización.
- [[opencode/fundamentos/configuracion|Configuración]] — `opencode.jsonc`, precedencia, ajustes de terminal y variables.
- [[opencode/fundamentos/modelos-proveedores|Modelos y proveedores]] — catálogo, proveedores locales, Console y Go.
- [[opencode/fundamentos/instrucciones|Instrucciones y AGENTS.md]] — contexto persistente del proyecto.

### [[opencode/extension/index|Extender OpenCode]]

- [[opencode/extension/agentes|Agentes y subagentes]] — agentes primarios, subagentes y perfiles especializados.
- [[opencode/extension/skills|Skills]] — instrucciones reutilizables con `SKILL.md`.
- [[opencode/extension/comandos|Comandos personalizados]] — plantillas de prompts con argumentos y shell.
- [[opencode/extension/plugins|Plugins]] — servidor, TUI, escritorio, hooks y herramientas.
- [[opencode/extension/mcp|MCP]] — servidores locales y remotos, OAuth y Code Mode.
- [[opencode/extension/permisos-politicas|Permisos y políticas]] — reglas `allow`/`ask`/`deny` y bloqueos duros.

### [[opencode/guias/index|Guías]]

- [[opencode/guias/flujo-de-trabajo|Flujo de trabajo diario]] — del plan a la verificación con snapshots y compactación.
- [[opencode/guias/desde-claude-code|Desde Claude Code]] — equivalencias, diferencias y checklist de migración.

Sources:
- [OpenCode Docs](https://opencode.ai/docs/)
- [OpenCode V2: índice de documentación](https://opencode.ai/docs/llms.txt)
- [OpenCode Console](https://opencode.ai/console)
