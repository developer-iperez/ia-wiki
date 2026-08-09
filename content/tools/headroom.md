---
title: headroom
---

Herramienta de compresión de contexto para agentes de IA: intercepta y comprime salidas de herramientas, logs, ficheros y chunks de RAG antes de que lleguen al LLM. Repo: [headroomlabs-ai/headroom](https://github.com/headroomlabs-ai/headroom).

## Propósito

Reducir los tokens que un agente envía al modelo comprimiendo el contenido "de por medio" (tool outputs, logs, ficheros, resultados de RAG) sin perder la información relevante para la tarea.

## Optimización obtenida

- **JSON**: 60-95% de reducción de tokens.
- **Agentes de código**: 15-20% de reducción.
- **Casos reales reportados**: desde 47% (exploración de codebase) hasta 92% (búsqueda de código).
- Mantiene la precisión de las respuestas ("same answers"), validado con benchmarks como GSM8K, TruthfulQA y SQuAD v2.

Mecanismo: pipeline con tres compresores especializados — **SmartCrusher** (compresión JSON universal), **CodeCompressor** (compresión AST-aware para Python, JS/TS, Go, Rust, Java, C++) y **Kompress-v2-base** (modelo de Hugging Face entrenado en trazas de agentes).

## Instalación

```bash
uv tool install --python 3.13 "headroom-ai[all]"   # recomendado (uv)
pip install "headroom-ai[all]"                     # alternativa con pip
npm install headroom-ai                            # solo SDK de TypeScript
```

Requiere Python ≥ 3.10.

## Comandos destacados

- `headroom wrap claude` — envuelve un agente de código (Claude Code y ~20 más).
- `headroom proxy --port 8787` — proxy HTTP local, sin tener que tocar el código del agente.
- `headroom doctor` — verificación de salud del sistema.
- `headroom learn --verbosity` — aprende el nivel de concisión preferido a partir de sesiones pasadas.
- `headroom deploy` — despliegue local "llave en mano".
- `headroom dashboard` — panel en vivo con el ahorro conseguido.

## Notas técnicas

- Modos de uso: librería (`compress()` inline), proxy HTTP transparente, servidor MCP, o wrappers específicos de agente (Claude Code, Cursor, Copilot, Aider...).
- **Local-first**: los datos no salen de la máquina.
- **Reversible**: las versiones originales sin comprimir se cachean y se pueden recuperar bajo demanda.
- Compatible con más de 20 agentes (Claude, Codex, Cursor, OpenCode, Cline...).
- Licencia Apache 2.0.

## Ver también

- [[claude/claude-code|Claude Code]] — uno de los agentes soportados vía wrapper.
- [[tools/rtk|rtk (Rust Token Killer)]] — optimización similar pero centrada solo en la salida de comandos de terminal; headroom cubre además JSON, ficheros y RAG.
- [[tools/claude-mem|claude-mem]] — ahorro complementario entre sesiones, en vez de en el contenido que se envía dentro de una sesión.
- [[tools/agentmemory|agentmemory]] — otra opción de memoria persistente entre sesiones, con búsqueda híbrida y grafos de conocimiento.

Sources:
- [headroomlabs-ai/headroom en GitHub](https://github.com/headroomlabs-ai/headroom)
