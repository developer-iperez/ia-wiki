---
title: IA wiki
---

Bienvenido a mi wiki de notas.

## Claude

### [[claude/index|Claude Code y ecosistema]]

#### [[claude/fundamentos/index|Fundamentos]]

- [[claude/fundamentos/claude-code|Claude Code]]
- [[claude/fundamentos/comandos|Comandos útiles de Claude Code]]
- [[claude/fundamentos/modelos-claude|Familia de modelos Claude]]

#### [[claude/extension/index|Extender Claude Code]]

- [[claude/extension/skills|Skills de Claude]]
- [[claude/extension/subagentes|Subagentes de Claude Code]]
- [[claude/extension/hooks|Hooks de Claude Code]]
- [[claude/extension/mcp|MCP (Model Context Protocol)]]

#### [[claude/guias/index|Guías]]

- [[claude/guias/claude-code-guide-zebbern|Claude Code Guide (zebbern)]]
- [[claude/guias/claude-code-everything|Claude Code: Everything You Need to Know]]

## Modelos LLM open source (ejecución local)

- [[models/index|Comparativa de modelos open source]]
- [[models/k2-horizon-mova-36b-a4b|K2-Horizon-MoVA-36B-A4B]]
- [[models/k2-horizon-7b|K2-Horizon-7B]]
- [[models/qwen3.6-35b-a3b|Qwen3.6-35B-A3B]]
- [[models/gemma-4-26b-a4b|Gemma-4-26B-A4B]]
- [[models/k2-horizon-4b|K2-Horizon-3.7B (4B)]]
- [[models/qwen3.5-9b|Qwen3.5-9B]]
- [[models/ling-3.0-tiny|Ling-3.0-tiny]]
- [[models/qwen3.5-4b|Qwen3.5-4B]]
- [[models/minicpm5-2b|MiniCPM5-2B]]
- [[models/granite-4.2-8b|Granite-4.2-8B]]
- [[models/granite-4.2-3b|Granite-4.2-3B]]
- [[models/g9v3|G9v3-3B]]
- [[models/gemma-4-e4b|Gemma-4-E4B]]
- [[models/llama-3.1-nemotron-nano-4b|Llama-3.1-Nemotron-Nano-4B]]
- [[models/nvidia-nemotron3-nano-4b|NVIDIA-Nemotron-3-Nano-4B]]
- [[models/lfm2.5-2.6b|LFM2.5-2.6B]]
- [[models/lfm2.5-8b-a1b|LFM2.5-8B-A1B]]
- [[models/minicpm5-1b|MiniCPM5-1B]]
- [[models/gemma-3n-e2b|Gemma-3n-E2B]]
- [[models/gemma-4-e2b|Gemma-4-E2B]]
- [[models/lfm3-1.7b|LFM3-1.7B]]
- [[models/minicpm-v-4.7-1b|MiniCPM-V-4.7-1B]]
- [[models/neohorse-1-4b|NeoHorse-1-4B]]
- [[models/smollm3-3b|SmolLM3-3B]]
- [[models/spark-x2.5-1.7b|Spark-X2.5-1.7B]]
- [[models/spark-x2.5-4b|Spark-X2.5-4B]]
- [[models/ternary-bonsai-2-27b|Ternary-Bonsai-2-27B]]
- [[models/ternary-bonsai-2-4b|Ternary-Bonsai-2-4B]]
- [[models/ternary-bonsai-2-9b|Ternary-Bonsai-2-9B]]

## Herramientas (ahorro de tokens / optimización de Claude e IA)

- [[tools/rtk|rtk (Rust Token Killer)]] — comprime la salida de la terminal antes de que llegue al modelo (hasta 90% menos tokens en outputs de bash).
- [[tools/claude-mem|claude-mem]] — memoria persistente entre sesiones: resume lo trabajado para no tener que reexplicar contexto (~10x menos tokens de historial).
- [[tools/headroom|headroom]] — compresión de contexto general (JSON, logs, ficheros, RAG) antes de enviarlo al LLM, en local.
- [[tools/agentmemory|agentmemory]] — memoria persistente con búsqueda híbrida (BM25 + vectorial + grafos) y visor en tiempo real.
- [[tools/codeburn|codeburn]] — mide consumo y coste de IA por modelo/proyecto y detecta patrones de desperdicio de tokens.
- [[tools/caveman|caveman]] — reduce los tokens de salida instruyendo al agente a responder en estilo minimalista (~65% menos en chat).
- [[tools/llm-router|llm-router]] — dirige cada prompt al modelo más barato capaz de resolverlo, reservando el premium para lo difícil (35-80% de ahorro).
- [[tools/codegraph|codegraph]] — grafo de conocimiento del código pre-indexado: el agente pregunta al grafo en vez de rastrear ficheros (62% menos tokens).
- [[tools/graphify|graphify]] — convierte código, docs, SQL y PDFs en un grafo consultable (AST local, sin vectores), con skill /graphify y visor HTML.
