---
title: Granite-4.2-8B
---

Modelo denso de razonamiento de la familia [Granite 4.2](https://huggingface.co/collections/ibm-granite/granite-42-language-models) (IBM), publicado en [ibm-granite/granite-4.2-8b](https://huggingface.co/ibm-granite/granite-4.2-8b). Es el tamaño intermedio de la serie, con thinking nativo, tool calling y 128K de contexto (512K con extensión).

> **Estado en ia-lab (2026-11-11):** probado y descartado el 2026-09-13. En la RTX 5060 8 GB su KV es muy cara: el techo útil es 16K con KV-Q4 y ~2 t/s de generación; a 24K baja a ~1 t/s. Tools sí, pero no compensa frente a Ling-3.0-tiny o Qwen3.6-35B-A3B.

## Highlights

- **Razonamiento nativo** con `<think>...</think>` y tres modos: thinking (por defecto), low-effort y non-thinking.
- **Tool calling aumentado con razonamiento**: el modelo razona qué herramienta invocar antes de emitir la llamada.
- **128K nativos y hasta 512K** con extensión de contexto.
- **12 idiomas testados**: inglés, alemán, español, francés, japonés, portugués, árabe, checo, italiano, coreano, neerlandés y chino.
- **Apache 2.0** y sucesor directo de Granite 4.1 8B.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 8B densos |
| Capas | 40; hidden 4.096, GQA 32 cabezas Q / 8 KV, head_dim 128, MLP 12.800, RoPE theta 10M |
| Contexto nativo | 131.072 tokens (128K) |
| Contexto extendido | hasta 512K con extensión de contexto (no verificada en local) |
| Modalidad | solo texto → texto |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | Transformers, vLLM (≥0.20 con parser `granite_thinking_parser`), SGLang, llama.cpp (GGUF oficial y de la comunidad) |

## Resumen esquemático de benchmarks

Valores del vendor (model card, BF16) frente a los otros tamaños de la familia:

| Benchmark | Granite 4.2 8B | Granite 4.2 3B | Granite 4.2 30B |
|---|---:|---:|---:|
| SWE-bench Verified | 47,67 | — | 57,00 |
| Terminal-Bench 2.1 | 20,56 | — | 29,24 |
| τ³-bench (media) | 58,06 | 45,78 | 62,00 |
| BFCL v4 | 52,39 | 52,41 | 61,39 |
| AIME25 | 86,67 | 78,33 | 89,17 |
| GPQA | 64,14 | 54,80 | 66,41 |
| LiveCodeBench v6 | 73,24 | 69,71 | 75,77 |
| MMLU-Pro | 74,04 | 67,84 | 77,60 |
| IFBench (prompt) | 79,33 | 74,33 | 77,17 |
| RULER 64K | 80,99 | 67,52 | 89,96 |
| RULER 128K | 71,41 | 55,30 | 81,38 |

Medidas locales en la RTX 5060 Laptop 8 GB (2026-09-13, Q4_K_M, 4,98 GiB):

| Config | Contexto útil | Velocidad | Notas |
|---|---:|---:|---|
| `-ngl 24` (f16) | — | pp 67,1 / tg 8,4 t/s | Techo de offload parcial |
| `-ngl 18`, KV-Q4 | 12K (recall 3/3) | ~2 t/s | 16K de contexto cargan, pero la generación es muy lenta |
| 24K | carga | ~1 t/s | Sonda >15 min sin responder; inviable para chat |

Lectura rápida: es un 8B de razonamiento competente y con buen tool calling, pero su KV (~160 KB/token en f16) lo hace inadecuado para esta GPU. En una GPU con más VRAM sería una alternativa sólida a modelos densos de 8B; aquí lo cubren modelos con KV más barata.

### Artificial Analysis (consultado 2026-11-11)

- **AA Progress Index v1: 211** ([Granite 4.2 8B en AA](https://artificialanalysis.ai/models/granite-4-2-8b)).
- **AA Intelligence Index v5.0.1: 5** en la ficha actual (escala v5, no comparable con v4.x).
- **Histórico:** Intelligence Index v4.3.2 = 11 (estimado), consultado el 2026-10-27.

## Configuración recomendada

Sampling oficial: `temperature=1.0`, `top_p=0.95` en todos los modos y backends. `max_new_tokens=8192` para thinking (subir en problemas complejos).

| Modo | Plantilla | Comportamiento |
|---|---|---|
| Thinking (default) | `enable_thinking=True` | Razonamiento completo en `<think>...</think>` |
| Non-thinking | `enable_thinking=False` | Respuesta directa |
| Low-effort | `enable_thinking=True, low_effort=True` | Razonamiento breve |

```bash
# vLLM con el parser de reasoning de Granite
vllm serve ibm-granite/granite-4.2-8b --max-model-len 131072 \
  --reasoning-parser granite_thinking_parser \
  --reasoning-parser-plugin ./granite_thinking_parser.py

# llama.cpp local (Q4_K_M) — configuración probada en ia-lab
llama-server -m granite-4.2-8b-Q4_K_M.gguf -ngl 18 -c 16384 \
  -ctk q4_0 -ctv q4_0 -fa on --jinja --reasoning-format deepseek \
  --temp 1.0 --top-p 0.95
```

- **KV**: usar KV-Q4 desde el principio; en f16 el contexto útil se queda en ~8K. A 24K la generación cae a ~1 t/s.
- **Herramientas**: verificadas vía API (tools ✓/✓), pero el coste de contexto hace que solo tengan sentido en ventanas cortas.
- **Historial**: en multi-turno el thinking de turnos anteriores se elimina por defecto (`truncate_history_thinking=True`); se puede conservar con `False`.

Sources:
- [ibm-granite/granite-4.2-8b en Hugging Face](https://huggingface.co/ibm-granite/granite-4.2-8b)
- [Colección Granite 4.2 Language Models](https://huggingface.co/collections/ibm-granite/granite-42-language-models)
- [Granite 4.2 Technical Blog](https://huggingface.co/blog/ibm-granite/granite-4-2)
- [Granite 4.2 8B en Artificial Analysis](https://artificialanalysis.ai/models/granite-4-2-8b)
