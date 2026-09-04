---
title: Spark-X2.5-4B
---

Modelo de la familia [Spark-X2.5](https://huggingface.co/XHToken/Spark-X2.5-4B) (XHToken / equipo SparkLLM, 2026), publicado en Hugging Face como [XHToken/Spark-X2.5-4B](https://huggingface.co/XHToken/Spark-X2.5-4B). Es un modelo compacto de propósito general orientado a **agentes y código on-device**: 4B de parámetros con 1M de contexto nativo, pensado para ejecución local en una sola GPU (también NPUs Ascend y Apple Silicon).

## Highlights

- **Atención híbrida eficiente**: combina 1 capa de atención completa con 3 de sliding-window (SWA) — el equilibrio entre rendimiento, velocidad de inferencia y tamaño de KV-cache, con 1M de contexto nativo sin trucos de extrapolación.
- **Entrenado para agentes**: pre-entreno de ~20T de tokens (con etapa dedicada de contexto largo hasta 1M) + SFT y RL a gran escala; los autores lo integran directamente con Codex, [[claude/claude-code|Claude Code]], OpenClaw y Hermes.
- **Líder de su tamaño en código agéntico y mates**: supera a [[models/qwen3.5-4b|Qwen3.5-4B]] (y a veces al de 9B) en SWE-Bench Pro/Multilingual, τ³-bench, MCP-Atlas y AIME 2026.
- **Más de 200 idiomas** y compatibilidad amplia de hardware (NVIDIA, Huawei Ascend, Hygon, HOUMO.AI).
- **Piensa por defecto**: modo thinking activable/desactivable vía `chat_template_kwargs` (igual que la familia Qwen3).

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 4B (denso, BF16) |
| Capas | arquitectura híbrida 1 × atención completa + 3 × sliding-window |
| Contexto nativo | 1.048.576 tokens (1M, sin YaRN ni extensiones) |
| Modalidad | solo texto (`text-generation`), conversacional |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | Transformers (`trust_remote_code=True`), vLLM, SGLang, llama.cpp (requiere el fork de XHToken), MLX (proyecto Spark-MLX-LLM), Ollama/LM Studio (vía GGUF y builds propios) |

## Resumen esquemático de benchmarks

Comparativa (según la model card oficial; los valores con * vienen de las model cards públicas de cada modelo, no de re-medición propia):

| Benchmark | Spark-X2.5-4B | Qwen3.5-4B | Qwen3.5-9B |
|---|---|---|---|
| BFCL-V4 (function calling) | 65.1 | 50.3* | 66.1* |
| τ³-bench (agentes) | 30.4 | 6.7 | 9.3 |
| MCP-Atlas (agentes MCP) | 54.6 | 40.8* | 47.4* |
| SWE-Bench Verified (código) | 41.6 | 38.8* | 53.1* |
| SWE-Bench Multilingual (código) | 53.3 | 27.7 | 43.3 |
| AIME 2026 (matemáticas) | 90.7 | 83.0 | 88.2 |
| IFBench (seguir instrucciones) | 75.0 | 59.2 | 64.5 |
| AA-LCR (contexto largo) | 56.3 | 57.0* | 63.0* |
| GPQA (conocimiento) | 67.4 | 67.2 | 77.2 |

Lectura rápida: domina claramente en tareas agénticas, código multilingüe y matemáticas frente a Qwen3.5-4B, al que duplica o triplica en varios benchmarks de agentes; en contexto largo (AA-LCR) y conocimiento general (GPQA) queda a la par del 4B y por debajo del 9B. Es solo-texto: si hace falta visión, [[models/qwen3.5-4b|Qwen3.5-4B]] sigue siendo la opción.

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
vllm serve XHToken/Spark-X2.5-4B --port 30000 --trust-remote-code \
  --served-model-name spark25 --tensor-parallel-size 1 \
  --gpu-memory-utilization 0.7 --enable-prefix-caching \
  --chat-template /models/Spark-X2.5-4B/chat_template.jinja

# SGLang
python -m sglang.launch_server --model-path XHToken/Spark-X2.5-4B \
  --served-model-name spark25 --tool-call-parser spark25 \
  --reasoning-parser qwen3 --tp-size 1 --mem-fraction-static 0.8 \
  --context-length 1048576 --host 0.0.0.0 --port 30000

# Transformers (pruebas rápidas, requiere trust_remote_code por el custom_code)
pip install transformers --upgrade  # más: trust_remote_code=True
```

El contexto de 1M exige mucha VRAM: si da OOM, reduce `--context-length`/`--max-model-len` (el grueso del uso agéntico rinde igual con 128-256K). Para CPU o Apple Silicon existen cuantizaciones GGUF de la comunidad ([11 variantes](https://huggingface.co/models?other=base_model%3Aquantized%3AXHToken%2FSpark-X2.5-4B)) y el proyecto [Spark-MLX-LLM](https://github.com/XHToken/Spark-MLX-LLM) — ojo, Ollama/LM Studio requieren compilar sus forks con el llama.cpp de XHToken, no valen los binarios stock. Fine-tuning con [LlamaFactory](https://github.com/XHToken/LlamaFactory).

Sources:
- [XHToken/Spark-X2.5-4B en Hugging Face](https://huggingface.co/XHToken/Spark-X2.5-4B)
- [Colección Spark-X2.5 (10 checkpoints y variantes)](https://huggingface.co/collections/XHToken/spark-x25)
