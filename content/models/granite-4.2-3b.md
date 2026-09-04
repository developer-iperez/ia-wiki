---
title: Granite-4.2-3B
---

Modelo compacto de razonamiento de la familia [Granite 4.2](https://huggingface.co/ibm-granite/granite-4.2-3b) (IBM Granite Team, agosto 2026), publicado en Hugging Face como [ibm-granite/granite-4.2-3b](https://huggingface.co/ibm-granite/granite-4.2-3b). Con solo 3B de parámetros densos y razonamiento chain-of-thought nativo, está pensado para **razonamiento, código y agentes on-device**: 128K de contexto nativo (ampliable a 512K) y tres modos de thinking para equilibrar profundidad y latencia.

## Highlights

- **Razonamiento nativo con tres modos**: thinking completo (por defecto), non-thinking y low-effort, conmutables por petición vía `enable_thinking` / `low_effort` en la chat template.
- **Tool calling con razonamiento integrado**: el modelo razona qué herramienta invocar y por qué antes de llamar, con buen rendimiento en BFCL-V4 (52.41) y τ³-bench (45.78, muy por encima de [[models/spark-x2.5-1.7b|Spark-X2.5-1.7B]]).
- **Muy fuerte en código e instrucciones para su tamaño**: LiveCodeBench v6 69.71 e IFBench 74.33, lo mejor entre los compactos del wiki.
- **12 idiomas testados**, incluido español (el rendimiento puede variar fuera del inglés).
- **Integración directa con harnesses agénticos**: recetas oficiales para OpenCode, Pi y OpenHands contra su API OpenAI-compatible.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 3B (denso, BF16) |
| Capas | 40, Transformer denso con GQA (40 heads, 8 KV), RoPE θ=10M, MLP SwiGLU 8192 |
| Contexto nativo | 131.072 tokens (128K) |
| Contexto extendido | hasta 512K |
| Modalidad | solo texto (`text-generation`), conversacional |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | Transformers, vLLM (parser propio `granite_thinking_parser`), SGLang, llama.cpp (GGUF de la comunidad), Ollama/LM Studio |

## Resumen esquemático de benchmarks

Comparativa (según la model card oficial; los valores con * vienen de las model cards públicas de cada modelo, no de re-medición propia; AIME mezcla ediciones 2025/2026 según lo publicado por cada familia):

| Benchmark | Granite-4.2-3B | Spark-X2.5-1.7B | LFM2.5-2.6B |
|---|---|---|---|
| BFCL-V4 (function calling) | 52.41 | 46.9 | 56.88 |
| τ³-bench (agentes) | 45.78 | 20.1 | — |
| AIME (matemáticas) | 78.33 (AIME25) | 69.4 (AIME 2026) | 51.87 (AIME25) |
| LiveCodeBench v6 (código) | 69.71 | — | 59.41 |
| IFBench (seguir instrucciones) | 74.33 | 66.3 | 59.17 |
| GPQA (conocimiento) | 54.80 | 43.8 | — |
| Contexto largo | 55.30 (RULER 128K) | 24.3 (AA-LCR) | — |

Lectura rápida: el mejor compacto del wiki en agentes (τ³-bench duplica al Spark 1.7B), código e instrucciones; en contexto largo nativo (128K, 512K extendido) queda por detrás del 1M de Spark-X2.5, y con 12 idiomas testados frente a los 200+ de Spark. Para matemáticas puras sigue por encima de [[models/lfm2.5-2.6b|LFM2.5-2.6B]].

## Configuración recomendada

**Parámetros de muestreo** (los mismos en todos los modos y backends, según la model card):

| temperature | top_p | max_new_tokens thinking | max_new_tokens no-thinking |
|---|---|---|---|
| 1.0 | 0.95 | 8192 | 2048 |

- `do_sample=True` siempre (requerido con temperature > 0).
- **Modos de thinking** (parámetros de la chat template): `enable_thinking=True` (defecto, razonamiento completo), `enable_thinking=False` (respuesta directa), `enable_thinking=True, low_effort=True` (razonamiento breve).
- En multi-turno, el thinking de turnos anteriores se recorta por defecto (`truncate_history_thinking=True`) para ahorrar contexto.
- Tool calling con parser `qwen3_coder` (en vLLM: `--tool-call-parser qwen3_coder --enable-auto-tool-choice`).

**Servir localmente**:

```bash
# vLLM (requiere vLLM v0.20+ y el fichero granite_thinking_parser.py del repo del modelo)
vllm serve ibm-granite/granite-4.2-3b \
  --served-model-name granite-4.2-3b --dtype bfloat16 \
  --max-model-len 131072 \
  --reasoning-parser granite_thinking_parser \
  --reasoning-parser-plugin ./granite_thinking_parser.py \
  --tool-call-parser qwen3_coder --enable-auto-tool-choice

# SGLang (v0.5.18+)
python -m sglang.launch_server --model-path ibm-granite/granite-4.2-3b \
  --dtype bfloat16 --context-length 131072 \
  --reasoning-parser auto --tool-call-parser auto

# Transformers (pruebas rápidas)
pip install accelerate transformers  # más: torch
```

Para CPU o despliegue ligero, usar las [cuantizaciones GGUF de la comunidad](https://huggingface.co/models?other=base_model%3Aquantized%3Aibm-granite%2Fgranite-4.2-3b) con llama.cpp, Ollama o LM Studio. El razonamiento de 8K tokens por defecto consume contexto rápido: si da OOM, baja `--max-model-len`/`--context-length` o usa el modo low-effort.

Sources:
- [ibm-granite/granite-4.2-3b en Hugging Face](https://huggingface.co/ibm-granite/granite-4.2-3b)
- [Granite 4.2 Technical Blog](https://huggingface.co/blog/ibm-granite/granite-4-2)
- [Colección Granite 4.2 Language Models](https://huggingface.co/collections/ibm-granite/granite-42-language-models)
