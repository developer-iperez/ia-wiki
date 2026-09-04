---
title: codegraph
---

Grafo de conocimiento del código pre-indexado, que se auto-sincroniza con cada cambio y da a los agentes de IA ([[claude/claude-code|Claude Code]], Codex, Gemini CLI, Cursor, OpenCode, Copilot...) el código exacto que necesitan en una sola llamada: menos tokens, menos tool calls, 100% local. Repo: [colbymchenry/codegraph](https://github.com/colbymchenry/codegraph).

## Propósito

Cuando un agente necesita entender código, lo descubre a la fuerza bruta: grep, glob y Read archivo por archivo, reconstruyendo a mano rutas de llamadas y dependencias antes de empezar el trabajo real. `codegraph` le entrega un índice ya construido con cada símbolo, arista de llamada y dependencia del repo, así que en vez de rastrear ficheros pregunta al grafo y recibe el fuente relevante, los caminos entre símbolos (incluidos saltos de despacho dinámico que grep no puede seguir) y el blast radius del cambio. Contexto quirúrgico en vez de búsqueda archivo por archivo.

## Instalación y uso

Tres pasos (el instalador no necesita Node.js, empaqueta su propio runtime):

```bash
# 1. Instalar el CLI
curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh
# o: npm i -g @colbymchenry/codegraph

# 2. Conectar los agentes (servidor MCP en cada uno)
codegraph install

# 3. Indexar cada proyecto (crea .codegraph/ local)
cd tu-proyecto && codegraph init
```

A partir de ahí no hay que resincronizar: un watcher nativo (FSEvents/inotify) actualiza el grafo con cada guardado tras un debounce de ~2s. Para ver lo mismo que ve el agente en el navegador:

```bash
codegraph ui       # abre http://127.0.0.1:4747
codegraph status   # verifica pendientes de sincronización
```

## Ahorro medido

Benchmarks del propio proyecto (re-medidos en 2026-08 con Claude Opus 4.8 sobre 7 repos reales en 7 lenguajes, agente con y sin CodeGraph): **88% menos tool calls, 53% más rápido, 62% menos tokens, 44% más barato**, con las lecturas de ficheros reducidas a cero en los siete repos. El ahorro depende de cuánto descubrimiento exigía la pregunta (57-78% donde el agente sin grafo necesitaba 28-43 llamadas).

Matiz importante que el propio proyecto documenta: esos números miden tokens *procesados*, no lo que queda residente en la ventana de contexto — las respuestas densas del grafo dejan ~80% más contexto residente al final de sesiones largas. En sesiones largas con ventana pequeña, hay que presupuestarlo.

## Notas técnicas

- Núcleo de parseo en Rust (20 lenguajes verificados byte por byte contra el motor de referencia, con fallback por fichero), índice local en SQLite con búsqueda full-text (FTS5).
- Análisis de impacto (llamantes, llamados, radio a N saltos), rutas framework-aware (17 frameworks web: del patrón URL al handler) y puentes mixtos iOS / React Native / Expo (Swift↔ObjC, TurboModules, Fabric, emisores de eventos).
- 100% local: sin API keys ni servicios externos; índice por proyecto en `.codegraph/`.
- Licencia MIT. `codegraph upgrade` actualiza in place; `codegraph uninstall` revierte la configuración de agentes.

## Ver también

- [[tools/rtk|rtk]] y [[tools/headroom|headroom]] — comprimen lo que se envía al modelo; `codegraph` evita que el agente tenga que descubrir la estructura a ciegas. Combinables.
- [[tools/codeburn|codeburn]] — para medir si el grafo está reduciendo de verdad el coste por sesión/proyecto.
- [[tools/claude-mem|claude-mem]] y [[tools/agentmemory|agentmemory]] — memoria entre sesiones; `codegraph` es conocimiento *del código*, no de la conversación.
- [[claude/mcp|MCP (Model Context Protocol)]] — el mecanismo por el que `codegraph install` conecta el grafo con cada agente.

Sources:
- [colbymchenry/codegraph en GitHub](https://github.com/colbymchenry/codegraph)
- [Documentación de CodeGraph](https://colbymchenry.github.io/codegraph/)
