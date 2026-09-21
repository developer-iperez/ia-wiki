---
title: Ling-3.0-tiny
---

Modelo de la familia [Ling 3.0](https://huggingface.co/collections/inclusionAI/ling-30) (Ant Group / InclusionAI, agosto 2026), publicado en Hugging Face como [inclusionAI/Ling-3.0-tiny](https://huggingface.co/inclusionAI/Ling-3.0-tiny). Es el miembro ligero de la familia: un MoE de 7.9B totales con solo **1.3B de parámetros activos por token**, razonador híbrido pensado para **agentes on-device y locales** (DGX Spark, MacBook Apple Silicon, Mac mini) sin GPUs de datacenter.

## Highlights

- **MoE diminuto en activos, grande en capacidad**: 128 expertos enrutados (8 activos + 1 compartido por token) — con 1.3B activos rinde cerca de densos de 3–4B y supera en agentes a modelos mucho mayores.
- **Atención híbrida-lineal KDA+MLA (3:1)**: 3 capas Kimi Delta Attention (lineal-recurrente, barata en KV-cache) + 1 capa Multi-Head Latent Attention (caché latente comprimida de 576 dims, ~10× menos memoria KV) por bloque de 4 capas; el diseño que en el hermano mayor ([[models/qwen3.5-4b|Qwen3.5-4B]]-like, Flash 124B) va en ratio 5:1.
- **Razonador híbrido conmutabre por petición**: thinking activado por defecto, desactivable con `enable_thinking: false` en `chat_template_kwargs` — un solo modelo para respuestas rápidas y razonamiento multi-paso.
- **Nativo para agentes**: parsers propios de razonamiento y tool-calling (`ling3` en SGLang/vLLM), function calling nativo y prompt caching; validado en más de 10.000 entornos interactivos a nivel de familia.
- **Licencia MIT** (la más permisiva del wiki) y tres precisiones oficiales (BF16, FP8, INT4) más cuantizaciones GGUF de la comunidad.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 7.9B MoE (1.3B activos por token: 8 expertos enrutados + 1 compartido de 128) |
| Capas | bloques de 4 × (3 × KDA lineal-recurrente + 1 × MLA latente 576-dim); nº total no publicado |
| Contexto nativo | 131.072 tokens (128K) |
| Contexto extendido | hasta 262.144 tokens (256K) vía YaRN (factor 2.0), soportado por SGLang/vLLM |
| Modalidad | solo texto (`text-generation`), conversacional |
| Licencia | MIT |
| Frameworks de inferencia | SGLang (imagen docker y cookbook oficiales), vLLM (compilar desde fuente, `trust-remote-code`), Ollama (PR #17643, vía MLX en Apple Silicon, compilar desde fuente), llama.cpp (GGUF de la comunidad), APIs (OpenRouter, Vercel AI Gateway) |

## Resumen esquemático de benchmarks

Comparativa con valores de Artificial Analysis (misma harness, comparables entre sí; los de Qwen y MiniCPM vienen de sus notas en este wiki):

| Benchmark | Ling-3.0-tiny | MiniCPM5-2B | Qwen3.5-4B |
|---|---|---|---|
| GPQA Diamond (conocimiento) | 73.4 | 70.2 | 77.1 |
| HLE (conocimiento extremo) | 9.3 | 8.9 | 7.8 |
| SciCode (código científico) | 24.2 | 26.3 | ~16–18 |
| AA-LCR (contexto largo) | 58.7 | 59.0 | 55.7 |
| τ³-Banking (agentes) | 20.8 | 20.8 | 8.2 |
| GDPval-AA v2 (trabajo real, Elo) | 772 | — | — |

Lectura rápida: con solo 1.3B activos empata con [[models/minicpm5-2b|MiniCPM5-2B]] en contexto largo y agentes (τ³ 20.8) y lo supera en GPQA (73.4 frente a 70.2); frente a [[models/qwen3.5-4b|Qwen3.5-4B]] gana en agentes y contexto largo pero pierde en conocimiento amplio y código científico. Es solo-texto: si hace falta visión, Qwen3.5-4B o [[models/gemma-4-e4b|Gemma-4-E4B]] siguen siendo la opción.

### Artificial Analysis (medición independiente, consultado 2026-09-21)

- [Página del modelo en AA](https://artificialanalysis.ai/models/ling-3-0-tiny): Intelligence Index **15 (estimado, v4.3.2)**, puesto #17/142; velocidad 57 tok/s (mediana entre proveedores), precio $0.00/$0.00 por 1M tokens in/out (InclusionAI y Novita), latencia al primer token ~3.3 s, contexto 262K.
- La model card oficial declara **25 puntos en el índice v4.1.1** y **16 en el Agentic Index** (por encima incluso del Gemma-4-31B en agentes), con >160 tok/s y ~18 s end-to-end para 500 tokens incluyendo thinking. La diferencia con el 15 actual es cambio de versión del índice (v4.1.1 → v4.3.2), no del modelo — mismo patrón que [[models/gemma-4-e4b|Gemma-4-E4B]]/[[models/gemma-4-e2b|Gemma-4-E2B]].
- Lectura: AA lo coloca como **líder del wiki en índice compuesto** (15, por encima de Qwen3.5-4B y MiniCPM5-2B con 13), rapidísimo por API (132–172 tok/s en InclusionAI) pero muy verboso en el índice (210–220M tokens, el peor del comparador en verbosidad).

## Configuración recomendada

**Parámetros de muestreo** (oficiales, únicos):

| temperature | top_p | top_k |
|---|---|---|
| 1.0 | 0.95 | 20 |

- Piensa por defecto (parser `ling3`). Para desactivarlo en una petición: `"chat_template_kwargs": {"enable_thinking": false}`.
- Tool calling con parsers propios `ling3` (en vLLM: `--tool-call-parser ling3 --reasoning-parser ling3 --enable-auto-tool-choice`).
- En el protocolo AA usan `max_new_tokens=32K` con 256K de contexto (Terminal-Bench 2.1, harness Terminus 2, 3 runs por tarea).

**Servir localmente**:

```bash
# SGLang (imagen oficial con runtime Ling-3.0, receta de baja latencia con YaRN 256K)
docker pull lmsysorg/sglang:dev-Ling-3.0-tiny
docker run --rm --gpus all --ipc=host --shm-size 32g -p 30000:30000 \
  -e HF_TOKEN=<tu-hf-token> lmsysorg/sglang:dev-Ling-3.0-tiny \
  env SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN=1 \
  python3 -m sglang.launch_server --model-path inclusionAI/Ling-3.0-tiny \
    --tp 1 \
    --json-model-override-args '{"rope_scaling":{"rope_type":"yarn","factor":2.0,"rope_theta":6000000,"partial_rotary_factor":0.5,"original_max_position_embeddings":131072}}' \
    --context-length 262144 --speculative-algorithm NEXTN \
    --mem-fraction-static 0.8 --host 0.0.0.0 --port 30000

# vLLM (requiere compilar desde fuente con soporte Ling-3.0)
vllm serve "$MODEL_PATH" --port "$PORT" --trust-remote-code \
  --served-model-name auto --tensor-parallel-size 1 \
  --gpu-memory-utilization 0.85 --enable-prefix-caching \
  --mamba-cache-mode align --enable-auto-tool-choice \
  --tool-call-parser ling3 --reasoning-parser ling3

# Ollama en Apple Silicon (requiere compilar el PR #17643, vía MLX)
git clone https://github.com/ollama/ollama.git && cd ollama
git fetch origin refs/pull/17643/head:bailing-moe-v3 && git switch bailing-moe-v3
cmake -B build . && cmake --build build --parallel 8
OLLAMA_CONTEXT_LENGTH=8192 ./ollama serve
```

Rendimiento local orientativo: FP8 ~100–105 tok/s en DGX Spark y ~86–90 tok/s en MacBook M4 Pro (~8.3 GiB a 8K de contexto); INT4 hasta ~115 tok/s (~6.1 GiB); BF16 ~68–71 tok/s (~14.9 GiB). Funciona con 8 GB de RAM en INT4. Para CPU/edge ligero, usar las [cuantizaciones GGUF de la comunidad](https://huggingface.co/models?other=base_model%3Aquantized%3AinclusionAI%2FLing-3.0-tiny) (ojo: la arquitectura `bailingmoe3` exige builds recientes de llama.cpp/Ollama; los binarios stock antiguos la rechazan). Pesos oficiales por precisión: [Ling-3.0-tiny](https://huggingface.co/inclusionAI/Ling-3.0-tiny) (BF16) y [Ling-3.0-tiny-fp8](https://huggingface.co/inclusionAI/Ling-3.0-tiny-fp8).

Sources:
- [inclusionAI/Ling-3.0-tiny en Hugging Face](https://huggingface.co/inclusionAI/Ling-3.0-tiny)
- [inclusionAI/Ling-3.0-tiny-fp8 en Hugging Face](https://huggingface.co/inclusionAI/Ling-3.0-tiny-fp8)
- [Colección Ling 3.0](https://huggingface.co/collections/inclusionAI/ling-30)
- [Cookbook SGLang para Ling-3.0-tiny](https://docs.sglang.io/cookbook/autoregressive/InclusionAI/Ling-3.0-tiny)
- [Ling 3.0 Tiny en Artificial Analysis](https://artificialanalysis.ai/models/ling-3-0-tiny)
- [Ling 3.0 Tiny: API providers en AA](https://artificialanalysis.ai/models/ling-3-0-tiny/providers)
