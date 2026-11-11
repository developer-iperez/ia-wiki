---
title: Ternary-Bonsai-2-27B
---

Modelo de la familia [Bonsai 2](https://huggingface.co/collections/prism-ml/bonsai-2) (PrismML), publicado en Hugging Face como [prism-ml/Ternary-Bonsai-2-27B-gguf](https://huggingface.co/prism-ml/Ternary-Bonsai-2-27B-gguf). Es un modelo de ~27B derivado de Qwen3.8-27B con pesos ternarios {-1, 0, +1}, pensado para ejecutar un 27B completo en una GPU de consumo de 8 GB o en un portátil.

> **Estado en ia-lab (2026-11-11):** instalado y recomendado como agente/código de gama alta local. Es el modelo conservado de la familia Bonsai 2; los empaquetados PQ2_0 y Q2_0 se probaron y descartaron el 2026-11-01.

## Highlights

- **Pesos ternarios g128**: embeddings, proyecciones de atención, MLP y LM head en {-1, 0, +1} con una escala FP16 compartida por grupo de 128 pesos. Solo el camino de estado lineal y las normalizaciones quedan en mayor precisión.
- **Tres empaquetados con los mismos pesos**: PTQ1_0 (trits densos, 5,95 GB), PQ2_0 (slots de 2 bits, 7,21 GB) y Q2_0 (formato estándar de llama.cpp, 7,63 GB). La calidad es idéntica; cambia el almacenamiento y la velocidad.
- **98,2 % de la inteligencia FP16**: 84,78 de media en 14 benchmarks en modo thinking, frente a 86,32 del Qwen3.8-27B FP16 y 72,59 de un IQ2_XXS convencional. BFCL v3 74,92.
- **Contexto de 262K** heredado de Qwen3.8-27B, con una arquitectura híbrida (~75 % atención lineal, ~25 % atención completa) que abarata el KV cache.
- **Visión opcional**: torre de visión de ~0,46B publicada como mmproj Q8_0 (~0,63 GB), que solo se carga cuando llega una imagen.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 27,36B totales: 24,35B de backbone (64 bloques) + 2,54B de embedding/LM head + 0,46B de torre de visión |
| Capas | 64 bloques híbridos: ~75 % atención lineal y ~25 % atención completa, SwiGLU, RoPE, RMSNorm |
| Contexto nativo | 262.144 tokens (262K) |
| Contexto extendido | no documentado; en local se ha verificado 64K full-GPU, 131K y 262K con capas en CPU |
| Modalidad | texto + imagen (mmproj opcional); salida solo texto |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | llama.cpp (fork PrismML para PTQ1_0/PQ2_0; upstream b11556+ solo para Q2_0), MLX (variante 2-bit) |

## Resumen esquemático de benchmarks

Evaluación del vendor con EvalScope + vLLM sobre H100, en modo thinking. La referencia FP16 es el Qwen3.8-27B original:

| Benchmark / categoría | Qwen3.8-27B FP16 | UD-Q4_K_XL | IQ2_XXS | Ternary-Bonsai-2-27B |
|---|---:|---:|---:|---:|
| MMLU-Redux | 91,46 | 93,35 | 88,93 | 89,09 |
| AIME25 | 96,67 | 92,91 | 66,67 | 95,00 |
| AIME26 | 94,58 | 93,00 | 57,50 | 95,83 |
| LiveCodeBench | 90,05 | 87,96 | 56,40 | 90,07 |
| IFBench (prompt-loose) | 71,00 | 65,65 | 53,76 | 74,00 |
| BFCL v3 | 76,74 | 75,05 | 70,28 | 74,92 |
| MMMU-Pro | 81,73 | 81,73 | 65,19 | 75,49 |
| OCR Bench v2 | 60,99 | 65,45 | 61,70 | 56,88 |
| **Media (14)** | **86,32** | **85,18** | **72,59** | **84,78** |

Medidas locales en la RTX 5060 Laptop 8 GB (2026-11-01, KV-Q4, `-fa on`, `-b 256 -ub 128`):

| Config | VRAM | Contexto | t/s corto | t/s gen @20,7K | Prefill @20,7K |
|---|---:|---:|---:|---:|---:|
| `-ngl 99 -c 65536` | 7.381 MiB | 64K full-GPU | 36,1 | 32,3 | 504 t/s |
| `-ngl 48 -c 131072` | 6.677 MiB | 131K (48/64 capas GPU) | 12,1 | 5,1 | 224 t/s |
| `-ngl 36 -c 262144` | 6.743 MiB | 262K (36/64 capas GPU) | 8,1 | 3,8 | 153 t/s |

Recall 3/3 en las tres configuraciones con un código oculto a 20,7K tokens. El offload parcial castiga la generación; para chat/agentes conviene el default de 64K full-GPU.

### Artificial Analysis (consultado 2026-11-11)

- **Sin entrada en AA**: el slug `/models/ternary-bonsai-2-27b` devuelve 404 y no hay Progress Index ni Intelligence Index v5.0.1. El único agregado de calidad disponible es el 84,78 del vendor (98,2 % del FP16), que no es directamente comparable con el índice de AA.

## Configuración recomendada

Sampling oficial de la ficha (el modelo piensa por defecto y usa `xhigh` como esfuerzo inicial):

| Modo | temp | top_p | top_k | min_p | presence_penalty | repetition_penalty |
|---|---:|---:|---:|---:|---:|---:|
| Thinking | 1.0 | 0.95 | 20 | 0.05 | 0.0 | 1.0 |
| Instruct / no-thinking | 0.7 | 0.80 | 20 | 0.0 | 1.5 | 1.0 |

- **Longitud de salida**: `max_tokens` ≥ 4096 como mínimo; 8192-16384 recomendado para thinking. Con 256 tokens se corta a mitad del razonamiento.
- **Esfuerzo**: `xhigh` por defecto; `medium` para respuestas más cortas. `low` no está soportado y se comporta casi como `xhigh`.
- **Herramientas**: verificado con `sandbox_list_files` (`finish_reason=tool_calls`, args `{"path":"."}`). La ficha avisa de llamadas malformadas o bucles puntuales; usar `max_tokens` generoso.

```bash
# Fork PrismML (PTQ1_0/PQ2_0) — configuración local recomendada
./llama-server -m Ternary-Bonsai-2-27B-PTQ1_0.gguf \
  -ngl 99 -fa on -c 65536 -ctk q4_0 -ctv q4_0 -b 256 -ub 128 \
  --temp 1.0 --top-p 0.95 --top-k 20 --min-p 0.05 --port 8080

# llama.cpp stock (b11556+) con Q2_0
llama-server -m Ternary-Bonsai-2-27B-Q2_0.gguf \
  -ngl 99 -fa on -c 32768 --temp 1.0 --top-p 0.95 --top-k 20 --min-p 0.05
```

- **Runtime**: PTQ1_0 y PQ2_0 solo cargan con el fork [PrismML-Eng/llama.cpp](https://github.com/PrismML-Eng/llama.cpp); el llama.cpp stock los rechaza. Q2_0 funciona en stock desde b11556, pero builds anteriores lo cargan sin avisar y generan basura.
- **Memoria**: PTQ1_0 es el más pequeño y el único que deja margen para 64K de contexto en 8 GB. PQ2_0 no carga a 8K en esta GPU; Q2_0 necesita `-ngl 60` y baja a ~24 t/s.
- **KV en RAM**: `--no-kv-offload` carga 131K/262K pero genera basura en el build probado (prism-b12128); no usarlo.

Sources:
- [prism-ml/Ternary-Bonsai-2-27B-gguf en Hugging Face](https://huggingface.co/prism-ml/Ternary-Bonsai-2-27B-gguf)
- [Colección Bonsai 2 (PrismML)](https://huggingface.co/collections/prism-ml/bonsai-2)
- [PrismML-Eng/Bonsai-demo (recetas de ejecución)](https://github.com/PrismML-Eng/Bonsai-demo)
- [Fork llama.cpp de PrismML](https://github.com/PrismML-Eng/llama.cpp)
- [Artificial Analysis: sin entrada para este modelo (consultado 2026-11-11)](https://artificialanalysis.ai/models/)
