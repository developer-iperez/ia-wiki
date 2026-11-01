---
title: Modelos LLM open source
---

Resúmenes de modelos LLM open source pensados para ejecución local (plantilla: `content/templates/modelo-llm.md`).

- [[models/k2-horizon-7b|K2-Horizon-7B]]
- [[models/qwen3.6-35b-a3b|Qwen3.6-35B-A3B]]
- [[models/gemma-4-26b-a4b|Gemma-4-26B-A4B]]
- [[models/k2-horizon-4b|K2-Horizon-3.7B (4B)]]
- [[models/ling-3.0-tiny|Ling-3.0-tiny]]
- [[models/qwen3.5-4b|Qwen3.5-4B]]
- [[models/qwen3.5-9b|Qwen3.5-9B]]
- [[models/minicpm5-2b|MiniCPM5-2B]]
- [[models/minicpm5-1b|MiniCPM5-1B]]
- [[models/g9v3|G9v3-3B]]
- [[models/neohorse-1-4b|NeoHorse-1-4B]]
- [[models/lfm2.5-2.6b|LFM2.5-2.6B]]
- [[models/lfm3-1.7b|LFM3-1.7B]]
- [[models/spark-x2.5-4b|Spark-X2.5-4B]]
- [[models/spark-x2.5-1.7b|Spark-X2.5-1.7B]]
- [[models/granite-4.2-3b|Granite-4.2-3B]]
- [[models/gemma-4-e4b|Gemma-4-E4B]]
- [[models/gemma-4-e2b|Gemma-4-E2B]]
- [[models/minicpm-v-4.7-1b|MiniCPM-V-4.7-1B]]
- [[models/ternary-bonsai-2-4b|Ternary-Bonsai-2-4B]]
- [[models/ternary-bonsai-2-9b|Ternary-Bonsai-2-9B]]

## Comparativa

Tabla mantenida a mano: cada modelo nuevo suma una fila. Detalle completo en su nota.

| Modelo | Parámetros | Contexto nativo | Arquitectura | Índice AA | Fuerte en | Modalidad | Licencia |
|---|---|---|---|---|---|---|---|
| <a href="./k2-horizon-7b">K2-Horizon-7B</a> | ~7B denso | 512K | densa GQA (36 capas) | 21 ‡ | agente/código, ventana media | solo texto | Apache 2.0 |
| <a href="./qwen3.6-35b-a3b">Qwen3.6-35B-A3B</a> | 35B MoE (3B activos) | 262K | híbrida Gated DeltaNet + atención, MoE | 18 ‡ | techo de inteligencia local, agentes | texto+imagen+vídeo | Apache 2.0 |
| <a href="./gemma-4-26b-a4b">Gemma-4-26B-A4B</a> | 25,2B MoE (3,8B activos) | 256K | MoE con ventana deslizante + capa global | 17* ‡ | razonamiento, multimodal | texto+imagen | Apache 2.0 |
| <a href="./k2-horizon-4b">K2-Horizon-3.7B (4B)</a> | 3,7B denso | 512K | densa GQA (36 capas) | 16 ‡ | agente/código en ventana corta | solo texto | Apache 2.0 |
| <a href="./ling-3.0-tiny">Ling-3.0-tiny</a> | 7.9B MoE (1.3B activos) | 128K (256K vía YaRN) | híbrida-lineal KDA+MLA 3:1, 128 expertos (8+1) | 15 (est.) | agentes on-device, velocidad, contexto largo | solo texto | MIT |
| <a href="./minicpm5-2b">MiniCPM5-2B</a> | ~2.5B denso | 128K | Llama GQA (42 capas) | 13 | código, mates, agentes/tool-use | solo texto | Apache 2.0 |
| <a href="./qwen3.5-4b">Qwen3.5-4B</a> | ~4.66B denso | 262K (1M vía YaRN) | híbrida Gated DeltaNet + atención completa | 13 (est.) | instrucciones, contexto largo, visión | texto+imagen+vídeo | Apache 2.0 |
| <a href="./g9v3">G9v3-3B</a> | ~3B denso | 128K | densa GQA (52 capas) | 11* ‡ | QA de documento largo | solo texto | Apache 2.0 |
| <a href="./qwen3.5-9b">Qwen3.5-9B</a> | ~9B denso | 256K | híbrida Gated DeltaNet + atención completa | 11 ‡ | calidad puntual, visión | texto+imagen+vídeo | Apache 2.0 |
| <a href="./gemma-4-e4b">Gemma-4-E4B</a> | 4.5B efectivos (8B total) | 128K | densa híbrida local+global, PLE | 9 (est.) | multimodal compacto, código, agentes | texto+imagen+vídeo+audio | Apache 2.0 |
| <a href="./granite-4.2-3b">Granite-4.2-3B</a> | 3B denso | 128K (512K ext.) | denso GQA (40 capas) | 9 | razonamiento, código, agentes | solo texto | Apache 2.0 |
| <a href="./minicpm5-1b">MiniCPM5-1B</a> | ~1,08B denso | 128K | Llama GQA (24 capas) | 9* ‡ | velocidad máxima, agentes simples | solo texto | Apache 2.0 |
| <a href="./gemma-4-e2b">Gemma-4-E2B</a> | 2.3B efectivos (5.1B total) | 128K | densa híbrida local+global, PLE | 8 (est.) | on-device/móvil, multimodal ligero | texto+imagen+vídeo+audio | Apache 2.0 |
| <a href="./lfm2.5-2.6b">LFM2.5-2.6B</a> | 2.69B denso | 128K | híbrida convolución + GQA | 8 (est.) | agentes/herramientas on-device, eficiencia | solo texto | LFM Open License 1.0 |
| <a href="./lfm3-1.7b">LFM3-1.7B</a> | 1,73B denso | 256K | híbrida convolución + atención (24 capas) | — | agente ligero rápido, 16 idiomas | solo texto | LFM Open License 1.0 |
| <a href="./minicpm-v-4.7-1b">MiniCPM-V-4.7-1B</a> | ~1,3B denso | no documentado | multimodal (visión + texto) | — | visión y texto en GPU modesta | texto+imagen+vídeo | Apache 2.0 |
| <a href="./neohorse-1-4b">NeoHorse-1-4B</a> | ~4B denso | 256K | Qwen3.5-4B post-entrenado | — | agentes, tool use | solo texto | Apache 2.0 |
| <a href="./spark-x2.5-1.7b">Spark-X2.5-1.7B</a> | 1.7B denso | 1M | híbrida full + SWA (1:3) | — (sin entrada) | instrucciones, eficiencia on-device | solo texto | Apache 2.0 |
| <a href="./spark-x2.5-4b">Spark-X2.5-4B</a> | 4B denso | 1M | híbrida full + SWA (1:3) | — (sin entrada) | agentes, código, matemáticas | solo texto | Apache 2.0 |
| <a href="./ternary-bonsai-2-4b">Ternary-Bonsai-2-4B</a> | ~4,2B denso (ternario Q2_0) | 256K | híbrida lineal + completa (3:1) | — | ventana grande en GPU modesta | texto+imagen | Apache 2.0 |
| <a href="./ternary-bonsai-2-9b">Ternary-Bonsai-2-9B</a> | ~9B denso (ternario Q2_0) | 256K | híbrida lineal + completa (3:1) | — | 9B entero en 8 GB de VRAM | texto+imagen | Apache 2.0 |

\* Intelligence Index de [Artificial Analysis](https://artificialanalysis.ai/) (variante reasoning cuando existe; "est." = estimado, pendiente de evaluación independiente).

‡ Valor tomado de las mediciones de ia-lab con el índice v4.3.2 (2026-10-11), no de la página pública de AA: a 2026-11-01 AA muestra el Intelligence Index v5.0 como N/A para estos modelos, así que no se ha podido confirmar. Valores de versiones distintas no son comparables al 100%; detalle en la nota de cada modelo.
