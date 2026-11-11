---
title: K2-Horizon-MoVA-36B-A4B
---

Modelo de la familia [K2-Horizon](https://ifm.ai/blog/k2/) (Institute of Foundation Models, IFM), publicado en Hugging Face como [IFM/K2-Horizon-MoVA-36B-A4B](https://huggingface.co/IFM/K2-Horizon-MoVA-36B-A4B). Es el miembro disperso de la familia: un MoE de 36B totales con 4B activos por token y atención Mixture-of-Values (MoVA), con 524K de contexto nativo.

> **Estado en ia-lab (2026-11-11):** probado y descartado el 2026-10-30. Su índice de inteligencia es el más alto del catálogo, pero en 8 GB de VRAM + 32 GB de RAM obliga a expertos en CPU, ocupa 20,82 GiB de disco y deja solo ~24K de contexto útil a ~4 t/s. Queda como referencia de techo, no como modelo instalado.

## Highlights

- **Resultados de clase frontier con 4B activos**: según el vendor, supera a densos de ~30B y a MoE de hasta 15× su tamaño en benchmarks agénticos y de razonamiento, y compite con modelos cerrados.
- **MoE + MoVA**: 100 expertos enrutados (top-8) + 1 compartido, y una atención MoVA con 64 expertos de valores (top-4) que reduce el coste del KV cache.
- **Contexto nativo de 512K** (524.288 tokens) desde las etapas de midtraining.
- **Totalmente abierto**: pesos, datos de entrenamiento, receta, código de entrenamiento y checkpoints intermedios.
- **Solo texto y solo inglés**, licencia Apache 2.0.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 36B totales, 4B activos por token (MoE, 100 expertos, top-8 + 1 compartido; MoVA 64 expertos, top-4) |
| Capas | 48; hidden 2.560, GQA 32 cabezas Q / 8 KV, head_dim 128, RoPE theta 10M |
| Contexto nativo | 524.288 tokens (512K) |
| Contexto extendido | no documentado (el nativo ya es 512K) |
| Modalidad | solo texto → texto |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | vLLM, SGLang, Transformers (con `trust_remote_code`), GGUF de la comunidad para llama.cpp |

## Resumen esquemático de benchmarks

Valores del vendor (model card, en modo reasoning y con el harness propio de IFM; los baselines son de Artificial Analysis). Selección de filas:

| Benchmark | K2-Horizon-MoVA-36B-A4B | Qwen3.6-35B-A3B | G9v3-39A5B | Gemma 4 31B-it |
|---|---:|---:|---:|---:|
| τ³-Banking (agentes) | **26,8** | 9,3 | 22,1 | 14,8 |
| Terminal-Bench 2.1 | **58,6** | 44,9 | 32,6 | 43,4 |
| SciCode | 38,9 | 35,8 | 34,0 | 43,4 |
| HLE (sin tools) | 25,2 | 22,2 | 17,5 | 23,6 |
| GPQA Diamond | 80,8 | 84,1 | 80,5 | 85,7 |
| AA-LCR (contexto largo) | 66,3 | 66,7 | 62,0 | 68,3 |
| AA-Omniscience (accuracy / no alucinación) | 18,8 / 69,2 | 18,8 / 49,5 | 14,9 / 87,0 | 20,0 / 15,0 |

Medidas locales en la RTX 5060 Laptop 8 GB (2026-10-30, Q4_K_M, `llama-bench` y servidor real):

| Config | Contexto útil | Velocidad | Notas |
|---|---:|---:|---|
| `-ngl 99 --cpu-moe` (base) | — | pp 343 / tg 20,6 t/s | Expertos en CPU; el resto en GPU |
| KV-Q4, 16K en GPU | 11,7K (recall 3/3) | ~11 t/s | 495 MB libres; al filo |
| KV en RAM, 32K | 24,5K (recall 3/3) | ~3,7 t/s | Solo lectura puntual |

Lectura rápida: el techo de calidad es real (AA Progress Index 281, el más alto del catálogo), pero el coste local lo hace inviable como modelo diario en 8+32 GB: 24K útiles a ~4 t/s frente a 131K del Qwen3.6 a ~33 t/s. Tools verificados 2/2 vía API.

### Artificial Analysis (consultado 2026-11-11)

- **AA Progress Index v1: 281** ([K2 Horizon MoVA 36B A4B en AA](https://artificialanalysis.ai/models/k2-horizon-mova-36b-a4b)). Es el índice que AA usa ahora en la comparativa de modelos open source pequeños; no es comparable con el Intelligence Index v4.3.2.
- **AA Intelligence Index v5.0.1: N/A** en la ficha del modelo.
- **Histórico:** Intelligence Index v4.3.2 = 25 (medido), consultado el 2026-10-30. Era el índice más alto del catálogo de ia-lab.

## Configuración recomendada

Sampling oficial: `temperature=1.0`, `top_p=0.95`; `reasoning_effort="high"` por defecto. El thinking se devuelve en `reasoning_content` y la respuesta en `content`. Formatos de tool call: `xml` (default), `json` o `xml_typed`.

```bash
# vLLM (2 GPUs, receta oficial)
vllm serve IFM/K2-Horizon-MoVA-36B-A4B --tensor-parallel-size 2 --enable-expert-parallel \
  --trust-remote-code --dtype bfloat16 --reasoning-parser k2_horizon \
  --tool-call-parser k2_horizon --enable-auto-tool-choice

# llama.cpp local (GGUF Q4_K_M de la comunidad) — solo como referencia
llama-server -m K2-Horizon-MoVA-36B-A4B-Q4_K_M.gguf -ngl 99 --cpu-moe \
  -c 32768 -b 512 -ub 512 -fa on --jinja --reasoning-format deepseek \
  --temp 1.0 --top-p 0.95
```

- **Memoria**: en 8 GB, los expertos MoE en CPU son obligatorios; `-ncmoe 48/56/64/80` dejan tg 20-21 t/s, pero `-ncmoe ≤40` no carga. La KV es cara (~192 KB/token en f16): 32K en GPU hace OOM; 16K en GPU o 32K con KV en RAM son los únicos puntos viables.
- **Longitud de salida**: `max_tokens` generoso (≥8192; la ficha usa 32768) porque el modelo piensa largo.
- **Uso**: solo tiene sentido con más VRAM/RAM; en este equipo queda como referencia de calidad, no como instalación.

Sources:
- [IFM/K2-Horizon-MoVA-36B-A4B en Hugging Face](https://huggingface.co/IFM/K2-Horizon-MoVA-36B-A4B)
- [Blog de IFM: K2 Horizon](https://ifm.ai/blog/k2/)
- [K2 Horizon MoVA 36B A4B en Artificial Analysis](https://artificialanalysis.ai/models/k2-horizon-mova-36b-a4b)
- [SGLang K2 Horizon cookbook](https://docs.sglang.io/cookbook/autoregressive/IFM/K2-Horizon)
