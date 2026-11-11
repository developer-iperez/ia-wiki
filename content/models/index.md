---
title: Modelos LLM open source
---

Resúmenes de modelos LLM open source pensados para ejecución local (plantilla: `content/templates/modelo-llm.md`). El catálogo incluye modelos instalados, probados pero no instalados y descartados; el estado de cada uno se resume en la sección [Estado en ia-lab](#estado-en-ia-lab-2026-11-11).

La tabla está ordenada por el **Progress Index v1 de Artificial Analysis** (consultado el 2026-11-11), de mayor a menor. Los modelos sin entrada en AA van al final, en orden alfabético. Los valores históricos del Intelligence Index v4.3.2 se conservan en la ficha de cada modelo.

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

## Comparativa

Tabla mantenida a mano: cada modelo nuevo suma una fila. Detalle completo en su nota.

| Modelo | Parámetros | Contexto nativo | Arquitectura | Índice AA (Progress v1) | Fuerte en | Modalidad | Licencia |
|---|---|---|---|---|---|---|---|
| <a href="./k2-horizon-mova-36b-a4b">K2-Horizon-MoVA-36B-A4B</a> | 36B MoE (4B activos) | 512K | MoE + MoVA (100 expertos, top-8) | 281 | agentes, código, contexto largo | solo texto | Apache 2.0 |
| <a href="./k2-horizon-7b">K2-Horizon-7B</a> | ~7B denso | 512K | densa GQA (36 capas) | 263 | agente/código, ventana media | solo texto | Apache 2.0 |
| <a href="./qwen3.6-35b-a3b">Qwen3.6-35B-A3B</a> | 35B MoE (3B activos) | 262K | híbrida Gated DeltaNet + atención, MoE | 261 | techo de inteligencia local, agentes | texto+imagen+vídeo | Apache 2.0 |
| <a href="./gemma-4-26b-a4b">Gemma-4-26B-A4B</a> | 25,2B MoE (3,8B activos) | 256K | MoE con ventana deslizante + capa global | 243 | razonamiento, multimodal | texto+imagen | Apache 2.0 |
| <a href="./k2-horizon-4b">K2-Horizon-3.7B (4B)</a> | 3,7B denso | 512K | densa GQA (36 capas) | 242 | agente/código en ventana corta | solo texto | Apache 2.0 |
| <a href="./qwen3.5-9b">Qwen3.5-9B</a> | ~9B denso | 256K | híbrida Gated DeltaNet + atención completa | 240 | calidad puntual, visión | texto+imagen+vídeo | Apache 2.0 |
| <a href="./ling-3.0-tiny">Ling-3.0-tiny</a> | 7,9B MoE (1,3B activos) | 128K (256K vía YaRN) | híbrida-lineal KDA+MLA 3:1, 128 expertos (8+1) | 231 | agentes on-device, velocidad, contexto largo | solo texto | MIT |
| <a href="./qwen3.5-4b">Qwen3.5-4B</a> | ~4,66B denso | 262K (1M vía YaRN) | híbrida Gated DeltaNet + atención completa | 231 | instrucciones, contexto largo, visión | texto+imagen+vídeo | Apache 2.0 |
| <a href="./minicpm5-2b">MiniCPM5-2B</a> | ~2,5B denso | 128K | Llama GQA (42 capas) | 221 | código, mates, agentes/tool-use | solo texto | Apache 2.0 |
| <a href="./granite-4.2-8b">Granite-4.2-8B</a> | 8B denso | 128K (512K ext.) | denso GQA (40 capas) | 211 | razonamiento, código, agentes | solo texto | Apache 2.0 |
| <a href="./granite-4.2-3b">Granite-4.2-3B</a> | 3B denso | 128K (512K ext.) | denso GQA (40 capas) | 191 | razonamiento, código, agentes | solo texto | Apache 2.0 |
| <a href="./g9v3">G9v3-3B</a> | ~3B denso | 128K | densa GQA (52 capas) | 189 | QA de documento largo | solo texto | Apache 2.0 |
| <a href="./gemma-4-e4b">Gemma-4-E4B</a> | 4,5B efectivos (8B total) | 128K | densa híbrida local+global, PLE | 183 | multimodal compacto, código, agentes | texto+imagen+vídeo+audio | Apache 2.0 |
| <a href="./llama-3.1-nemotron-nano-4b">Llama-3.1-Nemotron-Nano-4B</a> | 4,5B denso | 128K | Llama GQA (32 capas) | 183 | razonamiento, RAG, tool calling | solo texto | NVIDIA Open Model License |
| <a href="./nvidia-nemotron3-nano-4b">NVIDIA-Nemotron-3-Nano-4B</a> | 3,97B | 262K | híbrida Mamba-2 + 4 atención | 176 | retrieval en ventana corta, edge | solo texto | NVIDIA Nemotron Open Model License |
| <a href="./lfm2.5-2.6b">LFM2.5-2.6B</a> | 2,69B denso | 128K | híbrida convolución + GQA | 171 | agentes/herramientas on-device, eficiencia | solo texto | LFM Open License 1.0 |
| <a href="./lfm2.5-8b-a1b">LFM2.5-8B-A1B</a> | 8,3B MoE (1,5B activos) | 128K | híbrida conv + GQA (24 capas) | 167 | agentes/tool use, multilingüe | solo texto | LFM Open License 1.0 |
| <a href="./minicpm5-1b">MiniCPM5-1B</a> | ~1,08B denso | 128K | Llama GQA (24 capas) | 167 | velocidad máxima, agentes simples | solo texto | Apache 2.0 |
| <a href="./gemma-3n-e2b">Gemma-3n-E2B</a> | 6B brutos (2,3B efectivos) | 32K | MatFormer/PLE | 165 | multimodal on-device | texto+imagen+vídeo+audio | Gemma |
| <a href="./gemma-4-e2b">Gemma-4-E2B</a> | 2,3B efectivos (5,1B total) | 128K | densa híbrida local+global, PLE | 165 | on-device/móvil, multimodal ligero | texto+imagen+vídeo+audio | Apache 2.0 |
| <a href="./lfm3-1.7b">LFM3-1.7B</a> | 1,73B denso | 256K | híbrida convolución + atención (24 capas) | — | agente ligero rápido, 16 idiomas | solo texto | LFM Open License 1.0 |
| <a href="./minicpm-v-4.7-1b">MiniCPM-V-4.7-1B</a> | ~1,3B denso | no documentado | multimodal (visión + texto) | — | visión y texto en GPU modesta | texto+imagen+vídeo | Apache 2.0 |
| <a href="./neohorse-1-4b">NeoHorse-1-4B</a> | ~4B denso | 256K | Qwen3.5-4B post-entrenado | — | agentes, tool use | solo texto | Apache 2.0 |
| <a href="./smollm3-3b">SmolLM3-3B</a> | 3B denso | 64K (128K vía YaRN) | GQA + NoPE (36 capas) | — | razonamiento dual, tool calling | solo texto | Apache 2.0 |
| <a href="./spark-x2.5-1.7b">Spark-X2.5-1.7B</a> | 1,7B denso | 1M | híbrida full + SWA (1:3) | — | instrucciones, eficiencia on-device | solo texto | Apache 2.0 |
| <a href="./spark-x2.5-4b">Spark-X2.5-4B</a> | 4B denso | 1M | híbrida full + SWA (1:3) | — | agentes, código, matemáticas | solo texto | Apache 2.0 |
| <a href="./ternary-bonsai-2-27b">Ternary-Bonsai-2-27B</a> | ~27,36B denso (ternario PTQ1_0) | 262K | híbrida lineal + completa (~75/25) | — | agente/código de gama alta en 8 GB | texto+imagen | Apache 2.0 |
| <a href="./ternary-bonsai-2-4b">Ternary-Bonsai-2-4B</a> | ~4,2B denso (ternario Q2_0) | 256K | híbrida lineal + completa (3:1) | — | ventana grande en GPU modesta | texto+imagen | Apache 2.0 |
| <a href="./ternary-bonsai-2-9b">Ternary-Bonsai-2-9B</a> | ~9B denso (ternario Q2_0) | 256K | híbrida lineal + completa (3:1) | — | 9B entero en 8 GB de VRAM | texto+imagen | Apache 2.0 |

**Índice AA** = [Artificial Analysis](https://artificialanalysis.ai/) **Progress Index v1**, consultado el 2026-11-11 en las tablas de modelos open source *small* (4B-40B) y *tiny* (≤4B). Es la métrica que AA usa ahora en esa comparativa y **no es comparable** con el Intelligence Index v4.3.2 ni con el v5.0.1: son escalas distintas. El v5.0.1 solo tiene valor para unos pocos modelos del catálogo (Granite 4.2 3B/8B, Nemotron 3 Nano 4B y Qwen3.8 27B); para el resto aparece como N/A. Los valores v4.3.2 anteriores se conservan como histórico en cada ficha. `—` = sin página en AA.

## Estado en ia-lab (2026-11-11)

Inventario y veredictos de `ia-lab/MODELS.md`. Las velocidades son las medidas en la RTX 5060 Laptop 8 GB + 32 GB de RAM.

| Modelo | Estado | Fecha | Notas |
|---|---|---|---|
| Ternary-Bonsai-2-27B | **Instalado, recomendado** | 2026-11-01 | 64K full-GPU a ~36 t/s y tools; 131K/262K con capas en CPU; requiere fork PrismML |
| Qwen3.6-35B-A3B | **Instalado, recomendado** | 2026-10-27 | 131K/262K; ~33 t/s con expertos MoE en CPU |
| Ling-3.0-tiny | **Instalado, recomendado** | 2026-10-27 | 131K reales, recall 3/3 @79K, ~95-100 t/s, tools |
| LFM3-1.7B | **Instalado, recomendado** | 2026-10-28 | 1 GB, 131K, ~186 t/s corto / ~90 @100K, tools |
| Gemma-4-E4B | Instalado, mantener | 2026-10-27 | 131K full-GPU, ~54-56 t/s, respuestas directas sin thinking |
| Gemma-4-26B-A4B | Instalado, mantener (pesado) | 2026-10-27 | 32K a ~44 t/s con `-ncmoe 24`; solo si hace falta 262K |
| MiniCPM5-1B | Instalado, mantener | 2026-09-04 | 1 GB, ~80 t/s, tools; velocidad máxima y agentes simples |
| K2-Horizon-MoVA-36B-A4B | Descartado | 2026-10-30 | 24K útiles a ~4 t/s, 20,82 GiB; inviable en 8+32 GB |
| K2-Horizon-7B | Purgado | 2026-11-01 | Cubierto por Bonsai 27B (64K, ~36 t/s, más calidad); receta conservada |
| K2-Horizon-3.7B (4B) | Probado, no instalado | 2026-10-08 | Denso con KV cara; 16K útiles y tools |
| Qwen3.5-9B | Descartado | 2026-10-11 | ~3-7 t/s; K2 3.7B y Qwen3.6 lo superan |
| Qwen3.5-4B | Probado, no instalado | 2026-09-12 | 32K útiles; su rol lo cubre Qwen3.6-35B-A3B |
| MiniCPM5-2B | Probado, no instalado | 2026-09-09 | 64K KV-Q4 y recall 3/3 @47K; se conserva el 1B por velocidad |
| G9v3-3B | Descartado | 2026-10-11 | Sin tool-calling; Ling cubre el QA de documento largo |
| NeoHorse-1-4B | Descartado | 2026-10-11 | Clon funcional de Qwen3.5-4B sin nicho propio |
| LFM2.5-2.6B | Probado, no instalado | 2026-08-06 | Sustituido por LFM3-1.7B (más rápido, 131K, tools) |
| Spark-X2.5-4B | Descartado | 2026-09-15 | Dominado por Spark-X2.5-1.7B en tamaño, velocidad y contexto |
| Spark-X2.5-1.7B | Probado, no instalado | 2026-09-09 | Fue agente diario; hoy LFM3-1.7B y Ling cubren ese hueco |
| Granite-4.2-3B | Probado, no instalado | 2026-08-26 | Buen 3B, pero sin nicho frente a MiniCPM5 |
| Granite-4.2-8B | Descartado | 2026-09-13 | KV cara: 16K a ~2 t/s, 24K a ~1 t/s |
| Gemma-4-E2B | Probado, no instalado | 2026-09-12 | 64K full-GPU y ~35 t/s; su nicho lo cubre Gemma-4-E4B |
| Gemma-3n-E2B | Descartado | 2026-09-15 | Sin tools; contexto útil real 17-20K |
| MiniCPM-V-4.7-1B | Probado, no instalado | 2026-10-14 | Visión 1B; se sirve como dos perfiles (visión / tools) |
| Ternary-Bonsai-2-4B | Probado, no instalado | 2026-10-16 | La familia se conserva en el 27B PTQ1_0 |
| Ternary-Bonsai-2-9B | Probado, no instalado | 2026-10-16 | La familia se conserva en el 27B PTQ1_0 |
| LFM2.5-8B-A1B | Descartado | 2026-09-15 | Recall 0/3 @61K; Ling lo domina en contexto largo |
| Llama-3.1-Nemotron-Nano-4B | Descartado | 2026-09-12 | Sin tools en llama.cpp y solo 12-13K útiles |
| NVIDIA-Nemotron-3-Nano-4B | Retirado como alternativa | 2026-09-13 | 20K útiles y tools; dominado por LFM2.5-2.6B y Gemma-4-E2B |
| SmolLM3-3B | Descartado sin descargar | 2026-09-15 | Sin índice AA y cubierto por MiniCPM5-2B |

## Candidatos pendientes

Modelos del catálogo de ia-lab que aún no se han descargado o probado. No tienen ficha propia todavía; se resumen aquí con su índice AA cuando existe.

| Modelo | AA Progress v1 | Por qué interesa | Estado |
|---|---:|---|---|
| [Qwen3.8-27B](https://huggingface.co/Qwen/Qwen3.8-27B) | 303 (xhigh) / 290 (medium) / 283 (low) / 262 (non) | Denso híbrido 27B, 262K, visión y MTP drafter; base del Ternary-Bonsai-2-27B. UD-Q4_K_M 15,3 GiB; techo local, pero lento en 8 GB | Candidato #2 pendiente |
| [gpt-oss-20b](https://huggingface.co/openai/gpt-oss-20b) | 217 (high) / 205 (low) | MoE 21B/3,6B, 128K, MXFP4 11,28 GiB; harmony obligatorio. No cabe en 8 GB, expertos en RAM | Pendiente de probar |
| [Nemotron 3.5 Bolt E4B 4GB](https://huggingface.co/nvidia/NVIDIA-Nemotron-3.5-Bolt-E4B-4GB) | — (sin puntuación) | Post-entreno de Gemma-4-E4B, 8B/4B, 64K, Q4_K_M 6,18 GiB y drafter DFlash oficial | Candidato #3 pendiente |
| [Dory1.0-4B](https://huggingface.co/nvidia/NVIDIA-Nemotron-Labs-Dory1.0-4B-GGUF) | — (estimación NVIDIA 13,8/17,7/18,4 en v4.3.2) | 4B looped con 256K, Q5_K_M 3,67 GB; bloqueado por soporte de llama.cpp | Bloqueado |
