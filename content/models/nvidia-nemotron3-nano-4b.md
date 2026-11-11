---
title: NVIDIA-Nemotron-3-Nano-4B
---

Modelo pequeño de NVIDIA publicado en [nvidia/NVIDIA-Nemotron-3-Nano-4B-GGUF](https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Nano-4B-GGUF). Es un híbrido Mamba-2 + atención de 3,97B, comprimido desde Nemotron-Nano-9B-v2 con el framework Elastic, pensado para agentes en el borde (Jetson Thor, GeForce RTX, DGX Spark).

> **Estado en ia-lab (2026-11-11):** probado y retirado como alternativa el 2026-09-13. Funciona bien en ventana corta (20K útiles, ~28 t/s, tools ✓/✓), pero lo dominan LFM2.5-2.6B (más rápido y más contexto) y Gemma-4-E2B (más capaz). Se conserva la receta por si se necesita diversidad NVIDIA/Mamba.

## Highlights

- **Híbrido Mamba-2 + Transformer**: solo 4 capas de atención en una red de 3,97B, lo que abarata mucho el KV cache (~16 KB/token).
- **262K de contexto nativo** y RULER 128K de 91,2 en Q4_K_M según el vendor.
- **Razonamiento conmutable** por prompt de sistema (on/off), con trazas `[Start thinking]`.
- **Tool calling** verificado en ia-lab (ficheros + web).
- **Licencia NVIDIA Nemotron Open Model License**, uso comercial permitido.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 3,97B |
| Capas | Híbrido Mamba-2 + MLP con 4 capas de atención (arquitectura `nemotron_h`) |
| Contexto nativo | 262.144 tokens (262K) |
| Contexto extendido | no documentado; en local el techo práctico es 20.480 tokens |
| Modalidad | solo texto → texto |
| Licencia | NVIDIA Nemotron Open Model License |
| Frameworks de inferencia | llama.cpp (GGUF oficial), Transformers, vLLM |

## Resumen esquemático de benchmarks

Valores del vendor en modo reasoning-off (model card, Q4_K_M incluido):

| Benchmark | BF16 | Q4_K_M |
|---|---:|---:|
| IFBench-Prompt | 43,2 | 46,9 |
| IFBench-Instruction | 44,2 | 49,6 |
| IFEval-Prompt | 82,8 | 81,5 |
| IFEval-Instruction | 88,0 | 83,9 |
| HaluEval | 62,2 | 62,4 |
| RULER 128K | 91,1 | 91,2 |

Medidas locales en la RTX 5060 Laptop 8 GB (2026-09-04, Q4_K_M, 2,64 GiB):

| Medida | Valor |
|---|---:|
| Prompt (pp512) | 151,7 t/s |
| Generación (tg128) | 27,5 t/s |
| Generación en corto | ~28 t/s |
| Contexto útil | 20.480 tokens (`-b 256 -ub 128`; 24K y 32K fallan con `cudaMalloc failed`) |
| Tool calling | ✓/✓ (ficheros + web) |

Lectura rápida: es un híbrido Mamba-2 eficiente y con buen retrieval en ventana corta, pero su techo práctico de 20K y la competencia de modelos con más contexto y mejor calidad lo dejaron fuera del inventario.

### Artificial Analysis (consultado 2026-11-11)

- **AA Progress Index v1: 176** ([NVIDIA Nemotron 3 Nano 4B en AA](https://artificialanalysis.ai/models/nvidia-nemotron-3-nano-4b)).
- **AA Intelligence Index v5.0.1: 2** en la ficha actual (escala v5, no comparable con v4.x).
- **Histórico:** sin valor v4.3.2 en el wiki; AA lo incorporó con la metodología nueva.

## Configuración recomendada

La ficha no recomienda sampling explícito; ia-lab usa los valores estándar de modelos con reasoning: `temperature=0.6`, `top_p=0.95`, `top_k=20`.

```bash
# llama.cpp local — configuración probada en ia-lab
llama-server -m NVIDIA-Nemotron-3-Nano-4B-Q4_K_M.gguf \
  -ngl 99 -c 20480 -b 256 -ub 128 -fa on \
  --temp 0.6 --top-p 0.95 --top-k 20 \
  --reasoning-format deepseek --jinja
```

- **Contexto**: no pasar de 20.480 tokens en 8 GB; el batch por defecto agranda el buffer de cómputo y provoca OOM aunque haya VRAM libre.
- **Razonamiento**: se controla desde el prompt de sistema; `--reasoning-format deepseek` separa `[Start thinking]` en la web UI.
- **Herramientas**: verificadas con `file_glob_search` y `duckduckgo_search`; usar `max_tokens` generoso.

Sources:
- [nvidia/NVIDIA-Nemotron-3-Nano-4B-GGUF en Hugging Face](https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Nano-4B-GGUF)
- [NVIDIA Nemotron 3 Nano 4B en Artificial Analysis](https://artificialanalysis.ai/models/nvidia-nemotron-3-nano-4b)
- [Technical report de Nemotron 3 Nano (arXiv 2512.20848)](https://arxiv.org/abs/2512.20848)
