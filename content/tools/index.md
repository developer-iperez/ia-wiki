---
title: Herramientas
---

Herramientas para ahorrar tokens y optimizar procesos con Claude/IA.

- [[tools/rtk|rtk (Rust Token Killer)]] — comprime la salida de la terminal antes de que llegue al modelo (hasta 90% menos tokens en outputs de bash).
- [[tools/claude-mem|claude-mem]] — memoria persistente entre sesiones: resume lo trabajado para no tener que reexplicar contexto (~10x menos tokens de historial).
- [[tools/headroom|headroom]] — compresión de contexto general (JSON, logs, ficheros, RAG) antes de enviarlo al LLM, en local.
- [[tools/agentmemory|agentmemory]] — memoria persistente con búsqueda híbrida (BM25 + vectorial + grafos) y visor en tiempo real.
- [[tools/codeburn|codeburn]] — mide consumo y coste de IA por modelo/proyecto y detecta patrones de desperdicio de tokens.
- [[tools/caveman|caveman]] — reduce los tokens de salida instruyendo al agente a responder en estilo minimalista (~65% menos en chat).
- [[tools/llm-router|llm-router]] — dirige cada prompt al modelo más barato capaz de resolverlo, reservando el premium para lo difícil (35-80% de ahorro).
- [[tools/codegraph|codegraph]] — grafo de conocimiento del código pre-indexado: el agente pregunta al grafo en vez de rastrear ficheros (62% menos tokens).
- [[tools/graphify|graphify]] — convierte código, docs, SQL y PDFs en un grafo consultable (AST local, sin vectores), con skill /graphify y visor HTML.
