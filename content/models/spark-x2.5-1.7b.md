---
title: Spark-X2.5-1.7B
---

Hermano pequeño de [[models/spark-x2.5-4b|Spark-X2.5-4B]], de la familia [Spark-X2.5](https://huggingface.co/XHToken/Spark-X2.5-1.7B) (XHToken / equipo SparkLLM, 2026), publicado en Hugging Face como [XHToken/Spark-X2.5-1.7B](https://huggingface.co/XHToken/Spark-X2.5-1.7B). Es un modelo compacto de propósito general orientado a **agentes y código on-device**: 1.7B de parámetros con 1M de contexto nativo, pensado para ejecución local en hardware muy modesto (una sola GPU pequeña, NPUs Ascend, Apple Silicon e incluso CPU).

## Highlights

- **Misma arquitectura eficiente que el 4B**: 1 capa de atención completa + 3 de sliding-window (SWA), con 1M de contexto nativo sin trucos de extrapolación.
- **Entrenado para agentes**: pre-entreno de ~20T de tokens (con etapa dedicada de contexto largo hasta 1M) + SFT y RL a gran escala con consolidación MOPD; integración con Codex, [[claude/claude-code|Claude Code]], OpenClaw y Hermes.
- **Muy fuerte en instrucciones para su tamaño**: supera a Qwen3.5-4B en IFBench (66.3) y lo roza en IFEval (89.5), con τ²-bench de 65.3.
- **Más de 200 idiomas** y compatibilidad amplia de hardware (NVIDIA, Huawei Ascend, Hygon, HOUMO.AI).
- **Piensa por defecto**: modo thinking activable/desactivable vía `chat_template_kwargs` (igual que la familia Qwen3).

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 1.7B (denso, BF16) |
| Capas | arquitectura híbrida 1 × atención completa + 3 × sliding-window |
| Contexto nativo | 1.048.576 tokens (1M, sin YaRN ni extensiones) |
| Modalidad | solo texto (`text-generation`), conversacional |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | Transformers (`trust_remote_code=True`), vLLM, SGLang, llama.cpp (requiere el fork de XHToken), MLX (proyecto Spark-MLX-LLM), Ollama/LM Studio (vía GGUF y builds propios) |

## Resumen esquemático de benchmarks

Comparativa (según la model card oficial; los valores con * vienen de las model cards públicas de cada modelo, no de re-medición propia):

| Benchmark | Spark-X2.5-1.7B | Spark-X2.5-4B | Qwen3.5-2B |
|---|---|---|---|
| BFCL-V4 (function calling) | 46.9 | 65.1 | 43.6* |
| τ³-bench (agentes) | 20.1 | 30.4 | 4.1 |
| MCP-Atlas (agentes MCP) | 23.4 | 54.6 | 14.8 |
| SWE-Bench Verified (código) | 28.3 | 41.6 | 6.8 |
| SWE-Bench Multilingual (código) | 23.3 | 53.3 | 5.0 |
| AIME 2026 (matemáticas) | 69.4 | 90.7 | 30.8 |
| IFBench (seguir instrucciones) | 66.3 | 75.0 | 41.3* |
| AA-LCR (contexto largo) | 24.3 | 56.3 | 25.6* |
| GPQA (conocimiento) | 43.8 | 67.4 | 44.6 |

Lectura rápida: brilla en instrucciones (IFBench por encima de Qwen3.5-4B) y supera a Qwen3.5-2B en casi todo, pero queda claramente por debajo de su hermano de 4B en código agéntico (SWE-Bench Pro 10.4 frente a 44.4), contexto largo (AA-LCR 24.3 frente a 56.3) y conocimiento general (GPQA 43.8 frente a 67.4). Es la opción cuando el hardware manda y el 4B no cabe.

## Configuración recomendada

**Parámetros de muestreo** (únicos, según la model card):

| temperature | top_p | top_k |
|---|---|---|
| 1.0 | 0.95 | -1 |

- Piensa por defecto (parser de razonamiento tipo Qwen3). Para desactivarlo en una petición: `"chat_template_kwargs": {"enable_thinking": false}`.
- Tool calling con parser propio `spark25` (en SGLang: `--tool-call-parser spark25`).

**Servir localmente** (ejemplos con GPU única):

```bash
# vLLM
vllm serve XHToken/Spark-X2.5-1.7B --port 30000 --trust-remote-code \
  --served-model-name spark25 --tensor-parallel-size 1 \
  --gpu-memory-utilization 0.7 --enable-prefix-caching \
  --chat-template /models/Spark-X2.5-1.7B/chat_template.jinja

# SGLang
python -m sglang.launch_server --model-path XHToken/Spark-X2.5-1.7B \
  --served-model-name spark25 --tool-call-parser spark25 \
  --reasoning-parser qwen3 --tp-size 1 --mem-fraction-static 0.8 \
  --context-length 1048576 --host 0.0.0.0 --port 30000

# Transformers (pruebas rápidas, requiere trust_remote_code por el custom_code)
pip install transformers --upgrade  # más: trust_remote_code=True
```

El contexto de 1M exige mucha VRAM: si da OOM, reduce `--context-length`/`--max-model-len` (el grueso del uso agéntico rinde igual con 128-256K). Para CPU o Apple Silicon existen cuantizaciones GGUF de la comunidad (ver [cuantizaciones derivadas](https://huggingface.co/models?other=base_model%3Aquantized%3AXHToken%2FSpark-X2.5-1.7B)) y el proyecto [Spark-MLX-LLM](https://github.com/XHToken/Spark-MLX-LLM) — ojo, Ollama/LM Studio requieren compilar sus forks con el llama.cpp de XHToken, no valen los binarios stock. Fine-tuning con [LlamaFactory](https://github.com/XHToken/LlamaFactory).

Sources:
- [XHToken/Spark-X2.5-1.7B en Hugging Face](https://huggingface.co/XHToken/Spark-X2.5-1.7B)
- [Colección Spark-X2.5 (10 checkpoints y variantes)](https://huggingface.co/collections/XHToken/spark-x25)
