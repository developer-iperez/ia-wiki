---
title: MiniCPM5-1B
---

Primer modelo de la familia [MiniCPM5](https://huggingface.co/collections/openbmb/minicpm5) (OpenBMB), publicado en Hugging Face como [openbmb/MiniCPM5-1B](https://huggingface.co/openbmb/MiniCPM5-1B). Es un denso de ~1B pensado para asistente local, agentes de código y tool-use en dispositivos o GPU modesta. Lo sucede el [MiniCPM5-2B](minicpm5-2b), con la misma receta escalada.

## Highlights

- **Denso pequeño, arquitectura estándar**: `LlamaForCausalLM` con 24 capas y GQA (16 Q / 2 KV); carga directa en Transformers, vLLM, SGLang y llama.cpp sin kernels custom ni fork del modelo.
- **Pensamiento híbrido**: una sola checkpoint con modos Think y No-Think vía `enable_thinking` en el chat template.
- **Post-entreno SFT + RL + OPD**: 200B tokens de SFT en deep-thinking, 200B en hybrid-thinking, luego profesores RL (mates, código, QA cerrado, escritura) destilados con On-Policy Distillation. El vendor reporta ~+16 puntos de media en mates, código e instrucciones frente al SFT base.
- **Datos abiertos**: pretrain con Ultra-FineWeb y UltraData-Math, SFT con UltraData-SFT-2605, y gestión de datos por niveles según [Tiered Data Management](https://arxiv.org/pdf/2602.09003).
- **Tool-use con parser dedicado**: emite tool calls estilo XML; SGLang lo convierte a `tool_calls` de OpenAI con `--tool-call-parser minicpm5`. En llama.cpp (GGUF) también sale como `tool_calls` estándar según las pruebas locales de ia-lab.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 1.080.632.832 totales (679.552.512 sin embeddings), denso |
| Capas | 24, GQA con 16 heads Q y 2 KV |
| Contexto nativo | 131.072 tokens (128K) |
| Contexto extendido | no documentado (nativo 128K) |
| Modalidad | solo texto (`text-generation`), EN + ZH |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | Transformers (>=5.6), vLLM (>=0.21), SGLang (>=0.5.12), llama.cpp (GGUF), Ollama, LM Studio, MLX, LiteRT-LM, ArcLight |

## Resumen esquemático de benchmarks

Comparativa según la model card oficial, en modo Think (valores del vendor, no reproducidos de forma independiente). Escala 0–100:

| Benchmark | MiniCPM5-1B | Qwen3-0.6B | Qwen3.5-0.8B | LFM2.5-1.2B |
|---|---|---|---|---|
| MMLU-Pro (conocimiento) | 48.85 | 35.63 | 42.74 | 47.98 |
| AIME-2025 @Avg16 (mates) | 40.42 | 16.25 | 1.04 | 31.88 |
| LiveCodeBench v6 @Avg3 (código) | 33.52 | 16.00 | 5.33 | 21.33 |
| IFEval (instrucciones) | 80.41 | 59.89 | 59.89 | 84.84 |
| BFCLv4 (tool-use) | 25.15 | 25.43 | 25.53 | 10.60 |
| τ²-Bench Telecom (agentic) | 79.53 | 21.10 | 47.70 | 19.60 |

Lectura rápida: destaca en mates, código y agentes (τ²-Bench 79.5 frente a ≤48 de los comparables); en BFCLv4 queda empatado con Qwen3-0.6B y Qwen3.5-0.8B, y en IFEval lo supera LFM2.5-1.2B. Los modelos comparables son de 0.6–1.2B, así que no es una comparación contra modelos de su mismo tamaño exacto ni contra MiniCPM5-2B.

### Artificial Analysis (medición independiente, consultado 2026-11-01)

- [Página del modelo en AA](https://artificialanalysis.ai/models/minicpm5-1b): Intelligence Index **N/A** en la versión actual del índice (v5.0). Tanto la variante reasoning como la non-reasoning aparecen sin valor publicado y sin API pública.
- **Discrepancia**: el menú local de ia-lab (`MODELS.md`) recoge **9\*** para MiniCPM5-1B, pero la página de AA que consulté hoy no muestra ese valor. Es probable que el 9 venga de una versión anterior del índice; no he podido verificarlo.
- Lectura: AA no valida de momento el compuesto para este modelo. Los números de la tabla de arriba son del vendor.

## Configuración recomendada

- **Think** (por defecto): `temperature=0.9, top_p=0.95`, con `enable_thinking=True`.
- **No-Think**: `temperature=0.7, top_p=0.95`, con `enable_thinking=False`.
- **Longitud de salida**: el modelo piensa antes de responder, unos ~9K caracteres en pruebas de ia-lab. Con `max_tokens` ≤2048 la respuesta puede salir vacía; para razonamiento o tools conviene 4096–8192.
- **Modo por defecto en llama.cpp**: la plantilla piensa sin flags (verificado en ia-lab). Para la web UI, añadir `--reasoning-format deepseek` para separar el bloque de razonamiento.

```bash
# vLLM (vllm>=0.21)
vllm serve openbmb/MiniCPM5-1B --port 8000

# SGLang (recomendado para tool calling)
python -m sglang.launch_server --model-path openbmb/MiniCPM5-1B --port 30000 \
  --tool-call-parser minicpm5

# llama.cpp con GGUF oficial (Q8_0 ~1,07 GB)
llama-server -m MiniCPM5-1B-Q8_0.gguf -a MiniCPM5-1B --port 8080 -ngl 99 -c 16384 --jinja

# Transformers
pip install -U "transformers>=5.6" accelerate torch
```

Notas: el contexto nativo es 131K y el KV ocupa ~24 KB por token en f16, así que el 1B cabe bien en GPUs de 8 GB. En una RTX 5060 Laptop (8 GB), ia-lab midió f16 a 131072 con holgura; en una GTX 1650 (~3,5 GB) el techo práctico era 65536. La cuantización KV (q8/q4) debe verificarse por modelo: en este 1B las pruebas de recall en ~96–131K mostraron espirales de thinking tanto en q8 como en q4, así que no es atribuible a la quant. Para GGUF usar [MiniCPM5-1B-GGUF](https://huggingface.co/openbmb/MiniCPM5-1B-GGUF); en Apple Silicon, [versión MLX](https://huggingface.co/openbmb/MiniCPM5-1B-MLX). Para fine-tuning o reproducir el checkpoint previo a RL/OPD, ver [MiniCPM5-1B-SFT](https://huggingface.co/openbmb/MiniCPM5-1B-SFT).

Sources:
- [openbmb/MiniCPM5-1B en Hugging Face](https://huggingface.co/openbmb/MiniCPM5-1B)
- [Evaluation Results of MiniCPM5-1B (imagen de la model card)](https://raw.githubusercontent.com/OpenBMB/MiniCPM/main/assets/minicpm5/public_leaderboard_en.png)
- [MiniCPM GitHub (cookbooks de despliegue, sección MiniCPM5-1B)](https://github.com/OpenBMB/MiniCPM)
- [Tiered Data Management (arXiv 2602.09003)](https://arxiv.org/pdf/2602.09003)
- [MiniCPM5-1B en Artificial Analysis](https://artificialanalysis.ai/models/minicpm5-1b)
