---
title: agentmemory
---

Sistema de memoria persistente para agentes de IA de codificación, con búsqueda híbrida y captura automática vía hooks. Repo: [rohitg00/agentmemory](https://github.com/rohitg00/agentmemory).

## Propósito

Evitar que el agente "olvide" todo entre sesiones: captura arquitectura, preferencias y decisiones ya explicadas, para no tener que repetírselas cada vez que se abre una sesión nueva.

## Optimización obtenida

- Reduce el consumo estimado de ~22K tokens por sesión a ~1.900 tokens al año (~92% menos), lo que se traduce, según el proyecto, en pasar de ~500 USD/año a ~10 USD/año.
- Recuperación de contexto mediante **búsqueda híbrida** (BM25 + vectorial + grafos de conocimiento), con 95,2% de precisión en recall@5 sobre el benchmark LongMemEval-S.
- **Captura automática**: 12 hooks integrados registran observaciones durante el uso de herramientas, sesiones y otros eventos del agente, sin intervención manual.

## Instalación

```bash
npm install -g @agentmemory/agentmemory
agentmemory                          # arranca el servidor (puerto 3111)
agentmemory connect claude-code      # conecta con el agente
npx skills add rohitg00/agentmemory  # instala 15 skills nativos
```

## Comandos destacados

- `agentmemory demo` — carga datos de ejemplo y prueba la búsqueda semántica.
- `agentmemory connect <agente>` — conecta vía MCP con un agente concreto.
- `agentmemory stop` — detiene el servidor.
- `agentmemory upgrade` — actualiza dependencias.

## Notas técnicas

- Requiere Node.js ≥ 20.
- Incluye **iii-engine** (v0.11.2) o alternativamente Docker.
- Usa SQLite, sin dependencias externas de base de datos.
- Puertos: 3111 (API REST) y 3113 (visor en tiempo real de observaciones, sesiones y grafos de conocimiento, en `http://localhost:3113`).
- Compatible con 15+ agentes (Claude Code, Cursor, GitHub Copilot CLI, Cline, Windsurf...).
- Opcional: clave API de un proveedor LLM (Anthropic, OpenAI, Gemini...) para algunas funciones.

## Ver también

- [[claude/claude-code|Claude Code]] — uno de los agentes soportados.
- [[tools/claude-mem|claude-mem]] — mismo objetivo (memoria persistente entre sesiones); agentmemory añade búsqueda híbrida con grafos de conocimiento y un visor en tiempo real.
- [[tools/headroom|headroom]] — optimización complementaria: comprime lo que entra en la sesión, no lo que persiste entre sesiones.
- [[tools/codeburn|codeburn]] — para medir el ahorro real conseguido con agentmemory y otras optimizaciones.

Sources:
- [rohitg00/agentmemory en GitHub](https://github.com/rohitg00/agentmemory)
