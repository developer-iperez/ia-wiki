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

| Modelo | Parámetros | Contexto nativo | Arquitectura | Índice AA* | Fuerte en | Modalidad | Licencia |
|---|---|---|---|---|---|---|---|
| <a href="./qwen3.5-4b">Qwen3.5-4B</a> | ~4.66B denso | 262K (1M vía YaRN) | híbrida Gated DeltaNet + atención completa | 13 (est.) | instrucciones, contexto largo, visión | texto+imagen+vídeo | Apache 2.0 |
| <a href="./lfm2.5-2.6b">LFM2.5-2.6B</a> | 2.69B denso | 128K | híbrida convolución + GQA | 8 (est.) | agentes/herramientas on-device, eficiencia | solo texto | LFM Open License 1.0 |
| <a href="./spark-x2.5-4b">Spark-X2.5-4B</a> | 4B denso | 1M | híbrida full + SWA (1:3) | — (sin entrada) | agentes, código, matemáticas | solo texto | Apache 2.0 |
| <a href="./spark-x2.5-1.7b">Spark-X2.5-1.7B</a> | 1.7B denso | 1M | híbrida full + SWA (1:3) | — (sin entrada) | instrucciones, eficiencia on-device | solo texto | Apache 2.0 |
| <a href="./granite-4.2-3b">Granite-4.2-3B</a> | 3B denso | 128K (512K ext.) | denso GQA (40 capas) | 9 | razonamiento, código, agentes | solo texto | Apache 2.0 |
| <a href="./minicpm5-2b">MiniCPM5-2B</a> | ~2.5B denso | 128K | Llama GQA (42 capas) | 13 | código, mates, agentes/tool-use | solo texto | Apache 2.0 |
| <a href="./gemma-4-e4b">Gemma-4-E4B</a> | 4.5B efectivos (8B total) | 128K | densa híbrida local+global, PLE | 9 (est.) | multimodal compacto, código, agentes | texto+imagen+vídeo+audio | Apache 2.0 |
| <a href="./gemma-4-e2b">Gemma-4-E2B</a> | 2.3B efectivos (5.1B total) | 128K | densa híbrida local+global, PLE | 8 (est.) | on-device/móvil, multimodal ligero | texto+imagen+vídeo+audio | Apache 2.0 |

\* Intelligence Index de [Artificial Analysis](https://artificialanalysis.ai/) (variante reasoning cuando existe; "est." = estimado, pendiente de evaluación independiente). Valores consultados entre el 2026-09-09 y el 2026-09-21 con índice v4.3–v4.3.2 — no comparables al 100% entre versiones; detalle y desglose en la nota de cada modelo.
