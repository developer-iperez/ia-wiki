---
title: Gemma-3n-E2B
---

Modelo multimodal on-device de [Google DeepMind](https://ai.google.dev/gemma/docs/gemma-3n), publicado en [google/gemma-3n-E2B-it](https://huggingface.co/google/gemma-3n-E2B-it). Es la variante de ~2B efectivos de la familia Gemma 3n, pensada para móviles, portátiles y dispositivos de bajos recursos, con entrada de texto, imagen, vídeo y audio.

> **Estado en ia-lab (2026-11-11):** probado y descartado el 2026-09-15. Sin tool-calling (la plantilla nativa no tiene variable `tools`) y con un contexto útil real de ~17-20K pese a los 32K nativos. Lo cubren Gemma-4-E2B (64K, reasoning y tools) y Spark-X2.5-1.7B/LFM2.5-2.6B en velocidad.

## Highlights

- **MatFormer + PLE**: un checkpoint de 6B brutos que puede ejecutarse con ~2B efectivos mediante activación selectiva y caché de embeddings por capa (PLE).
- **Multimodal completo**: texto, imagen, vídeo y audio de entrada; salida solo texto.
- **32K de contexto** y más de 140 idiomas en el entrenamiento.
- **Carga condicional**: se pueden omitir los parámetros de visión y audio para reducir memoria.
- **Licencia Gemma** (uso comercial permitido bajo los términos de Google).

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 6B brutos / ~2B efectivos (AA: 5,1B totales, 2,3B activos) |
| Capas | MatFormer con PLE; la ficha no publica el número de capas |
| Contexto nativo | 32.768 tokens (32K) |
| Contexto extendido | no documentado |
| Modalidad | texto + imagen + vídeo + audio → texto |
| Licencia | Gemma (términos de Google) |
| Frameworks de inferencia | Transformers ≥4.53, vLLM, SGLang, llama.cpp/GGUF (solo texto en el GGUF probado) |

## Resumen esquemático de benchmarks

Valores del vendor (model card de Google, en precisión completa; `IT` = instruction-tuned):

| Benchmark | Gemma 3n E2B IT | Gemma 3n E4B IT |
|---|---:|---:|
| MMLU | 60,1 | 64,9 |
| MMLU-Pro | 40,5 | 50,6 |
| GPQA Diamond | 24,8 | 23,7 |
| AIME 2025 | 6,7 | 11,6 |
| LiveCodeBench v5 | 18,6 | 25,7 |
| HumanEval | 66,5 | 75,0 |
| MBPP | 56,6 | 63,6 |
| Global-MMLU | 55,1 | 60,3 |
| MGSM | 53,1 | 60,7 |

Medidas locales en la RTX 5060 Laptop 8 GB (2026-09-15, Q4_K_M, 2,81 GiB):

| Medida | Valor |
|---|---:|
| Prompt (pp512) | 267,2 t/s |
| Generación (tg128) | 46,9 t/s |
| Contexto útil | ~17-20K (recall 3/3 a 17K; 2/3 a 24K; mezcla de códigos a 30K) |
| Tool calling | ✗ (la plantilla nativa no soporta `tools`) |

Lectura rápida: es un buen modelo multimodal de bolsillo, pero en texto puro y contexto largo queda por detrás de los 2-4B solo-texto del catálogo. Su limitación decisiva en ia-lab fue la ausencia de tool calling.

### Artificial Analysis (consultado 2026-11-11)

- **AA Progress Index v1: 165** para la variante reasoning y **158** para non-reasoning ([Gemma 3n E2B en AA](https://artificialanalysis.ai/models/gemma-3n-e2b)).
- **AA Intelligence Index v5.0.1: N/A** en la ficha del modelo.
- **Histórico:** Intelligence Index v4.3.2 = 8 (estimado) / 6 (non-reasoning), consultado el 2026-10-27.
- **Contexto que lista AA**: 32K.

## Configuración recomendada

Sampling oficial de la ficha: `temperature=1.0`, `top_k=64`, `top_p=0.95`, `min_p=0.0`.

```bash
# llama.cpp con el GGUF de unsloth (solo texto)
llama-server -m gemma-3n-E2B-it-Q4_K_M.gguf -ngl 99 -fa on -c 32768 \
  --temp 1.0 --top-k 64 --top-p 0.95 --min-p 0.0
```

- **Contexto**: aunque el nativo son 32K, el recall se degrada a partir de ~20K; para QA de documento largo es mejor Ling-3.0-tiny o Gemma-4-E2B.
- **Multimodal**: el GGUF de llama.cpp probado solo soporta texto; para imagen/audio hace falta Transformers, vLLM o SGLang con el procesador original.
- **Herramientas**: no soportadas; no usar como agente.

Sources:
- [Gemma 3n: documentación oficial de Google](https://ai.google.dev/gemma/docs/gemma-3n)
- [google/gemma-3n-E2B-it en Hugging Face](https://huggingface.co/google/gemma-3n-E2B-it)
- [unsloth/gemma-3n-E2B-it-GGUF](https://huggingface.co/unsloth/gemma-3n-E2B-it-GGUF)
- [Gemma 3n E2B en Artificial Analysis](https://artificialanalysis.ai/models/gemma-3n-e2b)
