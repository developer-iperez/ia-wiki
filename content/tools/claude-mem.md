---
title: claude-mem
---

Sistema de memoria persistente y comprimida para agentes de IA (pensado para Claude Code). Repo: [thedotmack/claude-mem](https://github.com/thedotmack/claude-mem).

## Propósito

Captura automáticamente lo que el agente hace durante una sesión, genera resúmenes semánticos de ello y los inyecta en sesiones futuras, para mantener continuidad de conocimiento sobre un proyecto incluso después de que la sesión original termine (evitando volver a explicar contexto ya trabajado).

## Optimización obtenida

Flujo en tres capas para no cargar todo el historial de golpe:

1. **Búsqueda inicial** (~50-100 tokens): índice compacto con IDs.
2. **Contexto cronológico** (~200-300 tokens): vista de los resultados relevantes.
3. **Detalles completos** (~500-1.000 tokens): solo se recuperan las observaciones ya filtradas.

Al filtrar antes de pedir el detalle completo, el proyecto reporta hasta **~10x de ahorro de tokens** frente a cargar el historial completo.

## Instalación

```bash
npx claude-mem install
```

Variantes según el agente/IDE:

```bash
npx claude-mem install --ide opencode       # OpenCode
npx claude-mem install --ide antigravity    # Antigravity CLI
/plugin marketplace add thedotmack/claude-mem   # vía marketplace interno de Claude Code
```

> Importante: `npm install -g claude-mem` solo instala la librería; para que registre los hooks correctamente hay que usar `npx claude-mem install`.

## Comandos destacados

- `mem-search` — búsqueda en lenguaje natural sobre la memoria acumulada.
- Herramientas MCP expuestas: `search`, `timeline`, `get_observations`.
- Diagnóstico: basta con describirle el problema a Claude para que active la reparación automática del propio sistema.

## Notas técnicas

- Requiere Node.js ≥ 20.0.0 y una versión reciente de Claude Code.
- Instala automáticamente **Bun** y **uv** (gestor de paquetes Python) si no están presentes.
- Usa SQLite 3 (incluido) como almacén de la memoria.

## Ver también

- [[claude/claude-code|Claude Code]] — agente principal para el que está pensado.
- [[tools/rtk|rtk (Rust Token Killer)]] — optimización complementaria: rtk reduce tokens en la salida de comandos dentro de una sesión, claude-mem reduce tokens de contexto entre sesiones.
- [[tools/headroom|headroom]] — comprime lo que entra a la sesión (tool outputs, ficheros, RAG); claude-mem comprime lo que persiste entre sesiones.

Sources:
- [thedotmack/claude-mem en GitHub](https://github.com/thedotmack/claude-mem)
