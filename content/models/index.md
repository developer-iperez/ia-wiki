---
title: Modelos LLM open source
---

Resúmenes de modelos LLM open source pensados para ejecución local (plantilla: `content/templates/modelo-llm.md`).

- [[models/qwen3.5-4b|Qwen3.5-4B]]
- [[models/lfm2.5-2.6b|LFM2.5-2.6B]]
- [[models/spark-x2.5-4b|Spark-X2.5-4B]]

## Comparativa

Tabla mantenida a mano: cada modelo nuevo suma una fila. Detalle completo en su nota.

| Modelo | Parámetros | Contexto nativo | Modalidad | Arquitectura | Licencia | Fuerte en |
|---|---|---|---|---|---|---|
| <a href="./qwen3.5-4b">Qwen3.5-4B</a> | ~4.66B denso | 262K (1M vía YaRN) | texto+imagen+vídeo | híbrida Gated DeltaNet + atención completa | Apache 2.0 | instrucciones, contexto largo, visión |
| <a href="./lfm2.5-2.6b">LFM2.5-2.6B</a> | 2.69B denso | 128K | solo texto | híbrida convolución + GQA | LFM Open License 1.0 | agentes/herramientas on-device, eficiencia |
| <a href="./spark-x2.5-4b">Spark-X2.5-4B</a> | 4B denso | 1M | solo texto | híbrida full + SWA (1:3) | Apache 2.0 | agentes, código, matemáticas |
