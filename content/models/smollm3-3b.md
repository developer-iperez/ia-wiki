---
title: SmolLM3-3B
---

Modelo denso de [Hugging Face](https://huggingface.co/HuggingFaceTB/SmolLM3-3B), presentado como un 3B "totalmente abierto" con razonamiento dual, 6 idiomas y contexto largo. Fue uno de los candidatos evaluados por ia-lab para el rango 3B, pero se descartó sin descargarlo.

> **Estado en ia-lab (2026-11-11):** descartado sin descargar el 2026-09-15. No tenía índice AA publicado y su funcionalidad ya estaba cubierta por MiniCPM5-2B (12 en v4.3.2, tools verificados y 64K útiles). Se reevaluaría si AA publica un índice ≥13 o si aparece un GGUF con soporte de tools bien integrado.

## Highlights

- **3B densos** con GQA y NoPE en proporción 3:1, entrenados con 11,2T tokens.
- **Razonamiento dual**: modo thinking activado por defecto, conmutables con `/think` y `/no_think` en el prompt de sistema.
- **Contexto largo**: 64K nativos y hasta 128K con YaRN (256K teóricos).
- **Tool calling** en formato XML o Python, documentado en la ficha.
- **Totalmente abierto**: pesos, datos de preentrenamiento y receta; Apache 2.0.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 3B densos |
| Capas | 36; hidden 2.048, GQA 16 cabezas Q / 4 KV, MLP 11.008, NoPE cada 4 capas |
| Contexto nativo | 65.536 tokens (64K) |
| Contexto extendido | hasta 128K con YaRN (256K teóricos) |
| Modalidad | solo texto → texto |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | Transformers, vLLM, SGLang, llama.cpp (GGUF de la comunidad) |

## Resumen esquemático de benchmarks

Valores del vendor (model card). El modo thinking mejora matemáticas y conocimiento, a costa de más tokens:

| Benchmark | Sin thinking | Con thinking |
|---|---:|---:|
| AIME 2025 | 9,3 | 36,7 |
| GPQA Diamond | 35,7 | 41,7 |
| BFCL (tool calling) | 92,3 | 88,8 |
| Global MMLU | 53,5 | 64,1 |

Lectura rápida: para un 3B, el tool calling es su punto fuerte (BFCL >90 en el harness del vendor) y el thinking duplica con creces el rendimiento en matemáticas. No hay mediciones locales porque se descartó antes de descargarlo; los números son del vendor y no se han replicado.

### Artificial Analysis (consultado 2026-11-11)

- **Sin entrada en AA**: no hay página para SmolLM3-3B en la comparativa de modelos open source pequeños; no hay Progress Index ni Intelligence Index v5.0.1 que citar.
- **Histórico:** tampoco tenía índice v4.3.2 cuando se evaluó en ia-lab (2026-09-15).

## Configuración recomendada

Sampling oficial: `temperature=0.6`, `top_p=0.95`.

```bash
# vLLM con tool calling
vllm serve HuggingFaceTB/SmolLM3-3B --enable-auto-tool-choice \
  --tool-call-parser hermes --max-model-len 65536

# llama.cpp (GGUF de la comunidad) — no probado en ia-lab
llama-server -m SmolLM3-3B-Q4_K_M.gguf -ngl 99 -c 32768 \
  --temp 0.6 --top-p 0.95 --jinja
```

- **Thinking**: activo por defecto. Para desactivarlo, añadir `/no_think` al system prompt; para forzarlo, `/think`.
- **Contexto**: 64K nativos; para 128K hay que activar YaRN en `config.json` (no verificado en local).
- **Herramientas**: soportadas en el formato de la ficha, pero sin GGUF probado en ia-lab; no hay verificación de que el autoparser de llama.cpp las convierta correctamente.

Sources:
- [HuggingFaceTB/SmolLM3-3B en Hugging Face](https://huggingface.co/HuggingFaceTB/SmolLM3-3B)
- [SmolLM3: smol, multilingual, long-context reasoner (arXiv)](https://arxiv.org/abs/2507.06119)
- [Artificial Analysis: sin entrada para este modelo (consultado 2026-11-11)](https://artificialanalysis.ai/models/)
