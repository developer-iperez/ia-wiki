---
title: MiniCPM5-2B
---

Modelo de la familia [MiniCPM5](https://huggingface.co/collections/openbmb/minicpm5) (OpenBMB), publicado en Hugging Face como [openbmb/MiniCPM5-2B](https://huggingface.co/openbmb/MiniCPM5-2B). Es el segundo modelo de la serie tras el MiniCPM5-1B, un denso de ~2.5B pensado para asistente local, agentes de código y tool-use en dispositivos o GPU modesta.

## Highlights

- **Denso pequeño con receta escalada**: mismo entrenamiento que el 1B pero con más capacidad; arquitectura estándar `LlamaForCausalLM`, sin kernels custom ni fork del modelo.
- **Post-entreno SFT + RL + OPD**: 400B tokens de SFT con deep-thinking, luego profesores RL especializados (mates, código, agentes, escritura) destilados con On-Policy Distillation desde 16 expertos. Ganancia reportada de ~+11 puntos en razonamiento y ~+7 en agentic frente al SFT base.
- **Datos abiertos (familia UltraData)**: publican UltraX (web pretrain), UltraData-Code (gestión por niveles L0–L3), UltraData-SFT-Agent-2609 (500K muestras agente) y UltraData-RL-2609 (80K+ muestras RL).
- **Tool-use nativo con parser dedicado**: emite tool calls estilo XML; SGLang lo convierte a formato OpenAI con `--tool-call-parser minicpm5` (backend recomendado para agentes).
- **Ecosistema de formatos amplio**: checkpoints Base / Midtrain / SFT / final, más GGUF (llama.cpp/Ollama), MLX 4bit (Apple Silicon), GPTQ 4bit, LiteRT y modelo draft DSpark para speculative decoding.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | ~2.52B denso (1.98B sin embeddings) |
| Capas | 42, GQA con 16 heads Q y 2 KV |
| Contexto nativo | 131.072 tokens (128K) |
| Contexto extendido | no documentado (nativo 128K) |
| Modalidad | solo texto (`text-generation`), EN + ZH |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | Transformers, vLLM, SGLang, llama.cpp (GGUF), Ollama, LM Studio, MLX, LiteRT |

## Resumen esquemático de benchmarks

Comparativa según la model card oficial (valores del vendor, reproducidos internamente salvo † de Artificial Analysis). Escala 0–100:

| Benchmark | MiniCPM5-2B | Qwen3.5-2B | LFM2.5-2.6B | Qwen3.5-4B (ref. mayor) |
|---|---|---|---|---|
| LiveCodeBench v6 (código) | 69.1 | 20.2 | 42.1 | 56.4 |
| AIME 2025 (mates) | 86.5 | 29.6 | 41.9 | 78.8 |
| MATH-500 (mates) | 94.6 | 85.8 | 89.6 | 99.0 |
| IFEval (instrucciones) | 86.7 | 77.5 | 93.4 | 90.2 |
| MMLU-Pro (conocimiento) | 70.8 | 64.3 | 65.2 | 78.0 |
| AA-LCR (contexto largo) | 59.0 | 28.7 | 5.3 | 61.0 |
| BFCL v4 (tool-use) | 66.6 | 43.6 | 61.1 | 56.8 |
| SWE-bench Verified (agente código) | 46.4 | 5.0 | 6.0 | 33.6 |

Lectura rápida: domina con claridad su rango de 2B en código, mates, contexto largo y agentes (SWE-bench, GAIA 88.7); en conocimiento general amplio (MMLU-Pro) queda por debajo del Qwen3.5-4B, y en IFEval puro lo supera LFM2.5-2.6B. Promedio reportado 53.9, por encima de los 4B incluidos en su tabla.

### Artificial Analysis (medición independiente, consultado 2026-09-09)

- [Página del modelo en AA](https://artificialanalysis.ai/models/minicpm5-2b): Intelligence Index **13 (v4.3)**, puesto **#1/47 en clase tiny** (mediana 6); en v4.2 marcó 15, también top open <4B. Conciso: 74M tokens en el índice (#2/47 en verbosidad).
- Los valores con † en la tabla de arriba vienen ya del release oficial de AA según el vendor: SciCode 26.3, HLE 8.9, GPQA-Diamond 70.2, AA-LCR 59.0, τ³-Banking 20.8, Terminal-Bench v2.1 8.6, GDPval-AA v2 19.6. El resto (AIME, MATH-500, LiveCodeBench, SWE-bench, BFCL…) son reproducciones internas de OpenBMB, aún sin réplica externa fila a fila.
- Lectura: AA valida el compuesto (líder tiny), pero no cada fila del vendor; el SWE-bench Verified 46.4 en 2.5B sigue siendo el dato que más conviene reproducir antes de darlo por bueno en producción.

## Configuración recomendada

- **Sampling oficial**: `temperature=1.0, top_p=0.95`.
- **Thinking**: se activa con `enable_thinking=True` en `apply_chat_template`; salida de razonamiento larga para mates/código.
- **Longitud de salida**: ejemplos oficiales usan `max_tokens=128` / `max_new_tokens=128` para pruebas cortas; subir a varios miles en razonamiento y agentes.

```bash
# vLLM (vllm>=0.21)
vllm serve openbmb/MiniCPM5-2B --port 8000

# SGLang (recomendado para tool calling)
python -m sglang.launch_server --model-path openbmb/MiniCPM5-2B --port 30000
python -m sglang.launch_server --model-path openbmb/MiniCPM5-2B --port 30000 \
  --tool-call-parser minicpm5

# SGLang + speculative decoding (draft DSpark, misma salida del target)
python -m sglang.launch_server \
  --model-path openbmb/MiniCPM5-2B --trust-remote-code \
  --speculative-algorithm DSPARK \
  --speculative-draft-model-path openbmb/MiniCPM5-2B-DSpark \
  --speculative-dspark-block-size 7 --port 30000

# Transformers
pip install -U "transformers>=5.6" accelerate torch
```

Notas: al ser arquitectura Llama estándar, carga directa sin código custom. Para GGUF/Ollama/LM Studio usar [MiniCPM5-2B-GGUF](https://huggingface.co/openbmb/MiniCPM5-2B-GGUF); en Apple Silicon, [versión MLX 4bit](https://huggingface.co/openbmb/MiniCPM5-2B-MLX). Variante solo-SFT ([MiniCPM5-2B-SFT](https://huggingface.co/openbmb/MiniCPM5-2B-SFT)) disponible si se quiere el checkpoint previo a RL/OPD.

Sources:
- [openbmb/MiniCPM5-2B en Hugging Face](https://huggingface.co/openbmb/MiniCPM5-2B)
- [MiniCPM4 Tech Report (arXiv 2506.07900)](https://arxiv.org/pdf/2506.07900)
- [Tiered Data Management (arXiv 2602.09003)](https://arxiv.org/pdf/2602.09003)
- [MiniCPM5-2B en Artificial Analysis](https://artificialanalysis.ai/models/minicpm5-2b)
