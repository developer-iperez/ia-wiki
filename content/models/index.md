---
title: Modelos LLM open source
---

Resúmenes de modelos LLM open source pensados para ejecución local (plantilla: `content/templates/modelo-llm.md`).

- [[models/qwen3.5-4b|Qwen3.5-4B]]
- [[models/lfm2.5-2.6b|LFM2.5-2.6B]]
- [[models/spark-x2.5-4b|Spark-X2.5-4B]]
- [[models/spark-x2.5-1.7b|Spark-X2.5-1.7B]]
- [[models/granite-4.2-3b|Granite-4.2-3B]]
- [[models/minicpm5-2b|MiniCPM5-2B]]
- [[models/gemma-4-e4b|Gemma-4-E4B]]
- [[models/gemma-4-e2b|Gemma-4-E2B]]

## Comparativa

Tabla mantenida a mano: cada modelo nuevo suma una fila. Detalle completo en su nota.

| Modelo | Parámetros | Contexto nativo | Modalidad | Arquitectura | Licencia | Fuerte en |
|---|---|---|---|---|---|---|
| <a href="./qwen3.5-4b">Qwen3.5-4B</a> | ~4.66B denso | 262K (1M vía YaRN) | texto+imagen+vídeo | híbrida Gated DeltaNet + atención completa | Apache 2.0 | instrucciones, contexto largo, visión |
| <a href="./lfm2.5-2.6b">LFM2.5-2.6B</a> | 2.69B denso | 128K | solo texto | híbrida convolución + GQA | LFM Open License 1.0 | agentes/herramientas on-device, eficiencia |
| <a href="./spark-x2.5-4b">Spark-X2.5-4B</a> | 4B denso | 1M | solo texto | híbrida full + SWA (1:3) | Apache 2.0 | agentes, código, matemáticas |
| <a href="./spark-x2.5-1.7b">Spark-X2.5-1.7B</a> | 1.7B denso | 1M | solo texto | híbrida full + SWA (1:3) | Apache 2.0 | instrucciones, eficiencia on-device |
| <a href="./granite-4.2-3b">Granite-4.2-3B</a> | 3B denso | 128K (512K ext.) | solo texto | denso GQA (40 capas) | Apache 2.0 | razonamiento, código, agentes |
| <a href="./minicpm5-2b">MiniCPM5-2B</a> | ~2.5B denso | 128K | solo texto | Llama GQA (42 capas) | Apache 2.0 | código, mates, agentes/tool-use |
| <a href="./gemma-4-e4b">Gemma-4-E4B</a> | 4.5B efectivos (8B total) | 128K | texto+imagen+vídeo+audio | densa híbrida local+global, PLE | Apache 2.0 | multimodal compacto, código, agentes |
| <a href="./gemma-4-e2b">Gemma-4-E2B</a> | 2.3B efectivos (5.1B total) | 128K | texto+imagen+vídeo+audio | densa híbrida local+global, PLE | Apache 2.0 | on-device/móvil, multimodal ligero |
