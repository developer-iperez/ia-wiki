---
title: Llama-3.1-Nemotron-Nano-4B-v1.1
---

Modelo denso de NVIDIA derivado de [Llama-3.1-Minitron-4B-Width-Base](https://huggingface.co/nvidia/Llama-3.1-Minitron-4B-Width-Base), publicado en [nvidia/Llama-3.1-Nemotron-Nano-4B-v1.1](https://huggingface.co/nvidia/Llama-3.1-Nemotron-Nano-4B-v1.1). Está post-entrenado para razonamiento, chat, RAG y tool calling, y cabe en una sola GPU de consumo.

> **Estado en ia-lab (2026-11-11):** probado y descartado el 2026-09-12. Su plantilla nativa rompe el autoparser de llama.cpp y no permite tool-calling; además, el contexto útil se queda en ~12-13K con una KV cara (~128 KB/token en f16). Lo supera NVIDIA-Nemotron-3-Nano-4B (20K útiles y tools).

## Highlights

- **4,5B densos** derivados de Llama 3.1 8B mediante compresión Minitron.
- **128K de contexto** y solo ~2,58 GiB en Q4_K_M.
- **Razonamiento conmutable**: thinking on/off con prompts de sistema; recomendaciones de sampling distintas para cada modo.
- **Post-entrenamiento agéntico**: SFT de matemáticas, código, razonamiento y tool calling, más RL con RPO.
- **Licencia NVIDIA Open Model License** + Llama 3.1 Community License.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 4,5B densos |
| Capas | 32; hidden 3.072, GQA 32 cabezas Q / 8 KV, head_dim 128, MLP 9.216 |
| Contexto nativo | 131.072 tokens (128K) |
| Contexto extendido | no documentado |
| Modalidad | solo texto → texto |
| Licencia | NVIDIA Open Model License (+ Llama 3.1 Community License) |
| Frameworks de inferencia | Transformers, vLLM (parser `llama_nemotron_json`), llama.cpp (GGUF de la comunidad) |

## Resumen esquemático de benchmarks

Valores del vendor (model card), reasoning on/off:

| Benchmark | Reasoning off | Reasoning on |
|---|---:|---:|
| MT-Bench | 7,4 | 8,0 |
| MATH500 | 71,8 % | 96,2 % |
| AIME25 | 13,3 % | 46,3 % |
| GPQA-Diamond | 33,8 % | 55,1 % |
| IFEval (strict prompt / instruction) | 70,1 / 78,5 % | 75,5 / 82,6 % |
| BFCL v2 Live | 63,6 % | 67,9 % |
| MBPP 0-shot | 61,9 % | 85,8 % |

Medidas locales en la RTX 5060 Laptop 8 GB (2026-09-12, Q4_K_M, 2,58 GiB):

| Medida | Valor |
|---|---:|
| Generación en corto | ~36 t/s |
| Contexto útil | ~12-13K (KV ~128 KB/token en f16) |
| Tool calling | ✗ (la plantilla nativa rompe el autoparser del build 10875) |

Lectura rápida: es un 4B de razonamiento decente y muy rápido en corto, pero su contexto útil y la falta de tool-calling lo dejan sin nicho frente a Nemotron-3-Nano-4B y los 4B de Qwen/K2.

### Artificial Analysis (consultado 2026-11-11)

- **AA Progress Index v1: 183** ([Llama 3.1 Nemotron Nano 4B v1.1 en AA](https://artificialanalysis.ai/models/llama-3-1-nemotron-nano-4b-reasoning)).
- **AA Intelligence Index v5.0.1: N/A** en la ficha del modelo.
- **Histórico:** Intelligence Index v4.3.2 = 7, consultado el 2026-10-27.

## Configuración recomendada

Sampling oficial: `temperature=0.6`, `top_p=0.95` para reasoning on; greedy decoding para reasoning off. Todos los benchmarks del vendor se hicieron con 32K de secuencia.

```bash
# llama.cpp local (Q4_K_M) — configuración probada en ia-lab
llama-server -m Llama-3.1-Nemotron-Nano-4B-v1.1-Q4_K_M.gguf \
  -ngl 99 -c 16384 -ctk q4_0 -ctv q4_0 -fa on \
  --temp 0.6 --top-p 0.95 --jinja
```

- **Contexto**: 16K es el techo práctico con KV-Q4; a 24K la generación se vuelve muy lenta.
- **Herramientas**: aunque el modelo las soporta en vLLM con su parser y plantilla propias, en llama.cpp la plantilla nativa no se integra con el autoparser; se probó sin éxito. No usarlo como agente en llama.cpp.
- **Uso**: chat/razonamiento corto en inglés; para código o agentes hay opciones mejores en el catálogo.

Sources:
- [nvidia/Llama-3.1-Nemotron-Nano-4B-v1.1 en Hugging Face](https://huggingface.co/nvidia/Llama-3.1-Nemotron-Nano-4B-v1.1)
- [Llama 3.1 Nemotron Nano 4B v1.1 en Artificial Analysis](https://artificialanalysis.ai/models/llama-3-1-nemotron-nano-4b-reasoning)
