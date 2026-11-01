---
title: Qwen3.5-9B
---

Modelo de la familia [Qwen3.5](https://qwen.ai/blog?id=qwen3.5) (Alibaba), publicado en Hugging Face como [Qwen/Qwen3.5-9B](https://huggingface.co/Qwen/Qwen3.5-9B). Es un modelo denso de ~9B pensado para ejecución local en una GPU de consumo y para agentes con tool-use, con capacidad de entrada de imagen y vídeo además de texto.

## Highlights

- **Arquitectura híbrida**: bloques Gated DeltaNet (atención lineal) combinados con Gated Attention en proporción 3:1, con FFN tras cada bloque. Esto reduce el coste de atención en contextos largos frente a una pila de atención completa.
- **Multimodal nativo**: entrenamiento temprano (early fusion) sobre tokens de texto, imagen y vídeo; incluye encoder de visión dentro del mismo checkpoint.
- **Modo thinking por defecto**: genera bloque `<think>` antes de responder; se puede desactivar con `enable_thinking=False` para respuestas directas.
- **Contexto nativo amplio**: 262.144 tokens, extensible hasta ~1.010.000 con YaRN (estático).
- **Tool-use reforzado**: la ficha oficial destaca el rendimiento en tool calling y agentes (BFCL-V4, TAU2-Bench), con parser `qwen3_coder` en SGLang y vLLM.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | ~9B denso (ficha oficial: 9B en el modelo de lenguaje; el visor de HF muestra 10B en safetensors) |
| Capas | 32: 8 bloques × (3 × Gated DeltaNet → FFN, 1 × Gated Attention → FFN); 16 heads Q / 4 KV en atención completa |
| Contexto nativo | 262.144 tokens (256K) |
| Contexto extendido | hasta 1.010.000 tokens con YaRN (`factor` 4.0, configuración en `config.json` o vía `--hf-overrides` en vLLM) |
| Modalidad | texto + imagen + vídeo de entrada, salida solo texto |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | Transformers, vLLM, SGLang, KTransformers (oficiales); llama.cpp/Ollama vía GGUF de terceros (no documentado en la ficha oficial) |

## Resumen esquemático de benchmarks

Comparativa según la model card oficial (valores del vendor, no reproducidos de forma independiente). Escala 0–100:

| Benchmark | Qwen3.5-9B | Qwen3.5-4B | GPT-OSS-20B |
|---|---|---|---|
| MMLU-Pro (conocimiento) | 82.5 | 79.1 | 74.8 |
| GPQA Diamond (ciencia) | 81.7 | 76.2 | 71.5 |
| LiveCodeBench v6 (código) | 65.6 | 55.8 | 74.6 |
| IFEval (instrucciones) | 91.5 | 89.8 | 88.2 |
| AA-LCR (contexto largo) | 63.0 | 57.0 | 30.7 |
| BFCL-V4 (tool-use) | 66.1 | 50.3 | -- |

Lectura rápida: según el vendor, destaca en conocimiento, ciencia, instrucciones, contexto largo y tool-use frente a los comparables de su tabla; en código (LiveCodeBench v6) queda por detrás de GPT-OSS-20B. La tabla oficial incluye también visión (p. ej. MMMU 78.4, OmniDocBench1.5 87.7) y multilingüe, no reproducidos aquí.

### Artificial Analysis (medición independiente, consultado 2026-11-01)

- **Intelligence Index v4.3.2: 11** (copia de AA del 2026-10-15 en Wayback Machine, variante *reasoning*; existe también *non-reasoning*). La [página actual](https://artificialanalysis.ai/models/qwen3-5-9b) muestra el índice v5.0 como N/A.
- Datos de la misma ficha: 9.7B parámetros totales, 262k de ventana, texto+imagen+vídeo de entrada, Apache 2.0, ~61 tokens/s de salida y $0.14 / $0.20 por millón de tokens entrada/salida (mediana de proveedores).

## Configuración recomendada

Sampling oficial, según la ficha de Hugging Face:

| Modo / tarea | temperature | top_p | top_k | presence_penalty |
|---|---|---|---|---|
| Thinking, uso general | 1.0 | 0.95 | 20 | 1.5 |
| Thinking, código preciso (p. ej. WebDev) | 0.6 | 0.95 | 20 | 0.0 |
| Instruct (no-thinking), uso general | 0.7 | 0.8 | 20 | 1.5 |
| Instruct (no-thinking), razonamiento | 1.0 | 1.0 | 40 | 2.0 |

Nota: en el Quickstart oficial, la fila de razonamiento instruct usa `top_p=0.95, top_k=20, presence_penalty=1.5`; los valores de la tabla son los de la sección Best Practices.

- **Thinking**: activo por defecto. Para desactivarlo en la API OpenAI-compatible: `extra_body={"chat_template_kwargs": {"enable_thinking": False}}` (en Alibaba Cloud Model Studio, `"enable_thinking": False` directamente). No soporta los switches `/think` y `/nothink` de Qwen3.
- **Longitud de salida**: la ficha recomienda 32.768 tokens para la mayoría de consultas y hasta 81.920 en benchmarks de mates/programación competitiva. El ejemplo oficial usa `max_tokens=81920` para texto.
- **Contexto**: la ficha aconseja mantener al menos 128K para preservar la capacidad de thinking.

```bash
# vLLM (rama main/nightly, según la ficha)
vllm serve Qwen/Qwen3.5-9B --port 8000 --tensor-parallel-size 1 --max-model-len 262144 --reasoning-parser qwen3

# vLLM con tool calling
vllm serve Qwen/Qwen3.5-9B --port 8000 --max-model-len 262144 --reasoning-parser qwen3 \
  --enable-auto-tool-choice --tool-call-parser qwen3_coder

# vLLM solo texto (libera memoria del encoder de visión para más KV cache)
vllm serve Qwen/Qwen3.5-9B --port 8000 --max-model-len 262144 --reasoning-parser qwen3 --language-model-only

# SGLang
python -m sglang.launch_server --model-path Qwen/Qwen3.5-9B --port 8000 --context-length 262144 \
  --reasoning-parser qwen3 --tool-call-parser qwen3_coder

# Transformers (serve ligero, pruebas)
transformers serve --force-model Qwen/Qwen3.5-9B --port 8000 --continuous-batching
```

Notas: vLLM y SGLang exigen builds recientes (la ficha pide la rama main o nightly). El soporte MTP (`qwen3_next_mtp` en vLLM, NEXTN en SGLang) está documentado como recomendado. Para ejecución local con llama.cpp se usan GGUF de terceros (p. ej. unsloth); en ese caso conviene bajar la capa offload (`-ngl`) y cuantizar el KV cache si la VRAM es justa, ya que el modelo a contexto completo es pesado. Para contextos por encima de 262K, activar YaRN solo cuando haga falta: la ficha advierte que el escalado estático puede degradar textos cortos.

Sources:
- [Qwen/Qwen3.5-9B en Hugging Face](https://huggingface.co/Qwen/Qwen3.5-9B)
- [Blog oficial Qwen3.5](https://qwen.ai/blog?id=qwen3.5)
- [Qwen3.5 9B (Reasoning) en Artificial Analysis](https://artificialanalysis.ai/models/qwen3-5-9b)
