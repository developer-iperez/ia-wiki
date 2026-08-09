---
title: Claude Code
---

Claude Code es la herramienta de codificación agéntica de Anthropic que corre en la terminal (y se integra con VS Code y los IDEs de JetBrains). A diferencia de los asistentes clásicos que autocompletan línea a línea, funciona como un agente autónomo: lee la base de código completa, planifica cambios que pueden tocar varios archivos a la vez, los aplica, ejecuta comandos y tests, revisa la salida de errores y vuelve a intentarlo si algo falla.

## Instalación y arranque

- Instalación nativa: `curl -fsSL https://claude.ai/install.sh | bash`, seguida de autenticación en el navegador.
- Para arrancar una sesión interactiva en un proyecto basta con escribir `claude` en la terminal, dentro del repositorio.
- También se le puede pasar un prompt inicial directamente, p. ej. `claude "explain this project"`.
- Es recomendable crear un `CLAUDE.md` en la raíz del repo con contexto del proyecto (stack, comandos de build/test, arquitectura) — Claude lo lee automáticamente al empezar una sesión ahí. Este mismo repo tiene uno.

## Qué puede hacer

- Editar múltiples ficheros de forma coherente en una sola tarea.
- Ejecutar comandos de terminal (build, tests, linters) e iterar según el resultado.
- Gestionar flujos de Git: preparar cambios, escribir mensajes de commit, crear ramas y abrir pull requests.
- Conectarse a herramientas externas vía **MCP** (Model Context Protocol): leer documentos de Google Drive, actualizar tickets de Jira, consultar Slack, o cualquier servidor MCP que se configure.
- Trabajar con hasta ~1 millón de tokens de contexto, lo que le permite razonar sobre bases de código grandes.

## Modelo de permisos

Claude Code pide confirmación antes de ejecutar acciones potencialmente sensibles (editar archivos, ejecutar comandos, hacer push). Esos permisos se pueden acotar por proyecto en `.claude/settings.json`, definiendo qué herramientas/comandos se permiten sin preguntar, cuáles requieren aprobación y cuáles quedan bloqueados — así es como este repositorio limita las sesiones cloud a escribir solo dentro de `content/`.

## Ver también

- [[claude/comandos|Comandos útiles de Claude Code]]
- [[claude/skills|Skills de Claude]]
- [[tools/rtk|rtk (Rust Token Killer)]] — para reducir el consumo de tokens de las sesiones
- [[tools/claude-mem|claude-mem]] — para dar continuidad de contexto entre sesiones
- [[tools/headroom|headroom]] — para comprimir tool outputs, ficheros y RAG antes de que lleguen al modelo
- [[tools/agentmemory|agentmemory]] — alternativa a claude-mem para dar continuidad de contexto entre sesiones
- [[tools/codeburn|codeburn]] — para medir el consumo de tokens y coste de las sesiones
- [[tools/caveman|caveman]] — para reducir tokens de salida con un estilo de respuesta minimalista

## Uso desde el móvil (Claude Code Cloud)

La app de Claude (pestaña *Code*) permite lanzar sesiones de Claude Code en la nube sobre un repositorio de GitHub sin necesidad de un ordenador: la sesión clona el repo, trabaja de forma autónoma (puede seguir corriendo aunque se cierre la app) y al terminar el usuario revisa el diff y aprueba el `git push` desde el móvil.

Sources:
- [Descripción general - Claude Code Docs](https://code.claude.com/docs/es/overview)
- [Claude Code: Qué Es y Cómo Usarlo en 2026 — Guía Completa para Desarrolladores | Archi's Academy](https://archisacademy.com/en/blogs/claude-code-como-usar-2026)
- [Guía de Claude Code CLI: instalación, configuración, ...](https://blakecrosley.com/guides/claude-code)
