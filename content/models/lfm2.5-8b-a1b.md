---
title: LFM2.5-8B-A1B
---

Modelo MoE on-device de [Liquid AI](https://www.liquid.ai/), publicado en [LiquidAI/LFM2.5-8B-A1B](https://huggingface.co/LiquidAI/LFM2.5-8B-A1B). Tiene 8,3B totales con 1,5B activos por token y está afinado para razonamiento, tool use y asistentes locales multilingües.

> **Estado en ia-lab (2026-11-11):** probado y descartado el 2026-09-15. Carga a 128K con `--cpu-moe` y hace ~30 t/s, pero el recall se rompe a 61K (0/3) y Ling-3.0-tiny lo domina en contexto largo (131K con recall 3/3 @79K) con velocidad similar.

## Highlights

- **MoE híbrido**: 24 capas, 18 de convolución con doble gate y 6 de atención GQA; solo 1,5B activos.
- **128K de contexto** y 38 billones de tokens de entrenamiento.
- **Afinado para agentes**: tool use, salidas estructuradas y asistentes multilingües (9 idiomas).
- **Drafter DSpark** opcional de 328M para decodificación especulativa (~2,5× con salidas idénticas).
- **Licencia LFM Open License 1.0** (uso comercial con condiciones).

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 8,3B totales, 1,5B activos (MoE, 32 expertos, top-4) |
| Capas | 24: 18 convoluciones con doble gate + 6 atención GQA (32 Q / 8 KV) |
| Contexto nativo | 128.000 tokens |
| Contexto extendido | no documentado |
| Modalidad | solo texto → texto |
| Licencia | LFM Open License 1.0 |
| Frameworks de inferencia | Transformers, vLLM, SGLang, llama.cpp (GGUF oficial), ONNX, MLX |

## Resumen esquemático de benchmarks

Valores del vendor (model card) frente a modelos de tamaño similar:

| Benchmark | LFM2.5-8B-A1B | Qwen3.5-4B | Gemma-4-E4B | gpt-oss-20b |
|---|---:|---:|---:|---:|
| IFEval | **91,84** | 87,80 | 87,74 | 86,73 |
| IFBench | 56,47 | 50,38 | 39,48 | **58,65** |
| MATH500 | **88,76** | 80,76 | 65,00 | — |
| AIME25 | 42,53 | **54,28** | 34,33 | — |
| BFCLv3 | 64,79 | **71,06** | 57,31 | — |
| BFCLv4 | 49,73 | **54,01** | 33,92 | — |
| Tau² Telecom | **88,07** | 87,72 | 26,75 | — |
| Tau² Retail | 39,82 | **71,93** | 42,11 | — |

Medidas locales en la RTX 5060 Laptop 8 GB (2026-09-15, Q4_K_M, 4,79 GiB):

| Config | Contexto | Velocidad | Notas |
|---|---:|---:|---|
| `-ngl 99 --cpu-moe` | 128K | pp 200-202 / tg 30,5 t/s | Expertos en CPU; el resto en GPU |
| Recall a 20K | 20K | — | 3/3 |
| Recall a 61K | 61K | — | 0/3; el contexto largo no es fiable |
| Tool calling | — | — | ✓/✓ vía API |

Lectura rápida: es rápido y muy bueno en instrucciones y tool use para su tamaño, pero su ventana útil real se queda en ~20K. Ling-3.0-tiny ofrece más contexto útil y velocidad parecida, así que no tiene nicho propio en este equipo.

### Artificial Analysis (consultado 2026-11-11)

- **AA Progress Index v1: 167** ([LFM2.5-8B-A1B en AA](https://artificialanalysis.ai/models/lfm2-5-8b-a1b)).
- **AA Intelligence Index v5.0.1: N/A** en la ficha del modelo.
- **Histórico:** Intelligence Index v4.3.2 = 7, consultado el 2026-10-27.

## Configuración recomendada

Sampling oficial: `temperature=0.2`, `top_k=80`, `repetition_penalty=1.05`.

```bash
# llama.cpp local con expertos en CPU
llama-server -m LFM2.5-8B-A1B-Q4_K_M.gguf -ngl 99 --cpu-moe \
  -c 32768 -b 512 -ub 512 -fa on --jinja \
  --temp 0.2 --top-k 80 --repeat-penalty 1.05
```

- **Contexto**: 32K para chat/agentes; 128K solo para lectura corrida, no para recuperar datos enterrados.
- **Herramientas**: verificadas; usar `max_tokens` generoso porque el modelo razona antes de llamar.
- **Alternativa**: Ling-3.0-tiny para contexto largo; Qwen3.6-35B-A3B para calidad.

Sources:
- [LiquidAI/LFM2.5-8B-A1B en Hugging Face](https://huggingface.co/LiquidAI/LFM2.5-8B-A1B)
- [LiquidAI/LFM2.5-8B-A1B-GGUF](https://huggingface.co/LiquidAI/LFM2.5-8B-A1B-GGUF)
- [LFM2.5-8B-A1B en Artificial Analysis](https://artificialanalysis.ai/models/lfm2-5-8b-a1b)
