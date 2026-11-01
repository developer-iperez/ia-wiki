---
title: K2-Horizon-7B
---

Modelo medio de la familia [K2-Horizon](https://huggingface.co/collections/IFM/k2-horizon-gguf) (IFM, Institute of Foundation Models), publicado en Hugging Face como [IFM/K2-Horizon-7B-GGUF](https://huggingface.co/IFM/K2-Horizon-7B-GGUF) (versión GGUF del [IFM/K2-Horizon-7B](https://huggingface.co/IFM/K2-Horizon-7B) original). Es un denso decoder-only de ~7B core con 512K de contexto, pensado para agentes, código y razonamiento en una GPU de consumo con cuantización Q4_K_M.

## Highlights

- **Denso de referencia con 512K de contexto**: 524.288 tokens nativos desde las fases de midtraining; no hace falta extensión para la ventana completa.
- **Checkpoints intermedios publicados**: se liberan checkpoints de entrenamiento (`mid_*_final`, `base_final`) para estudiar la evolución de las capacidades.
- **Totalmente abierto**: datos y receta de preentreno y midtraining (IFM/K2-Horizon-Pretrain-Data, IFM/K2-Horizon-Midtrain-Data), código de entrenamiento y recursos de evaluación públicos.
- **Razonamiento siempre activo con nivel configurable**: `reasoning_effort` (high por defecto; medium/low vía `chat_template_kwargs`) y el pensamiento se devuelve en `reasoning_content`.
- **Tool-calling con parser dedicado**: formatos `xml` (defecto), `json` y `xml_typed`; en vLLM y SGLang se usa `--tool-call-parser k2_horizon`.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | ~7B denso según la ficha (el GGUF figura como 9B al contar embeddings y cabezales) |
| Capas | 36, GQA con 32 heads Q y 8 KV, head_dim 128 (dato de ia-lab, no de la ficha) |
| Contexto nativo | 524.288 tokens (512K) |
| Contexto extendido | no documentado (nativo 512K) |
| Modalidad | solo texto (entrada y salida) |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | llama.cpp (GGUF, build con soporte `k2-horizon`, mainline desde b11454), vLLM, SGLang, Transformers, Ollama, LM Studio, Jan, Docker Model Runner |

## Resumen esquemático de benchmarks

Comparativa según la ficha oficial de Hugging Face (valores del vendor, escala 0–100; los comparables son los tres modelos de la propia tabla):

| Benchmark | K2-Horizon-7B | Qwen3.5-9B | Gemma 4-12B | Granite 4.2-8B |
|---|---|---|---|---|
| HMMT Feb 2026 (mates) | 73.3 | 65.7 | 63.1 | 66.5 |
| SWE-bench Verified (agente código) | 70.6 | 50.8 | 30.6 | 47.7 |
| HLE (razonamiento experto) | 18.6 | 14.9 | 15.7 | 9.7 |
| LCR (contexto largo) | 68.0 | 65.3 | 61.7 | 43.3 |
| Terminal-Bench 2.1 (terminal agéntica) | 39.1 | 29.2 | 27.3 | 18.4 |

Lectura rápida: destaca en mates, código de agente (SWE-bench) y contexto largo frente a los 8–12B de su clase; en conocimiento experto (HLE) la ventaja sobre Qwen3.5-9B es modesta. La ficha no publica en su tabla principal BFCL ni IFEval de la versión GGUF; en la tabla de cuantizaciones (BF16 vs Q4_K_M) el Q4_K_M da IFEval 82.9, MMLU-Pro 71.6, GPQA-Diamond 67.1 y AIME 26 89.1.

### Artificial Analysis (medición independiente, consultado 2026-11-01)

- [Página del modelo en AA](https://artificialanalysis.ai/models/k2-horizon-7b): en la consulta del 2026-11-01 el Intelligence Index v5.0 aparece como **N/A** (sin puntuación publicada), con contexto de 524k, Apache 2.0 y 7B de parámetros.
- Dato local: la medición de ia-lab con el índice v4.3.2 dio **AA 21** (ver `MODELS.md`), +5 sobre K2-Horizon-3.7B. Esa cifra no la he podido confirmar en la página de AA, que ahora usa v5.0.
- Lectura: AA no valida hoy el compuesto en v5.0; los benchmarks del vendor (SWE-bench 70.6, Terminal-Bench 2.1 39.1) son la referencia disponible, y conviene repetirlos antes de darlos por buenos en producción.

## Configuración recomendada

- **Sampling oficial**: `temperature=1.0, top_p=0.95`.
- **Razonamiento**: `reasoning_effort="high"` por defecto (la ficha pide high en todas las peticiones; medium/low sacrifican precisión). Se pasa en `chat_template_kwargs`.
- **Longitud de salida**: al menos 32.768 tokens de `max_tokens`; si el razonamiento se corta, la respuesta cuenta como fallida.
- **Formato de tools**: `tool_call_format` = `xml` (defecto), `json` o `xml_typed` vía `chat_template_kwargs`.

```bash
# llama.cpp (llama-server), como se usa en ia-lab: RTX 5060 8GB, Q4_K_M, KV-Q4
llama-server -m K2-Horizon-7B-Q4_K_M.gguf \
  -ngl 99 -c 32768 -ctk q4_0 -ctv q4_0 -b 256 -ub 128 \
  --temp 1.0 --top-p 0.95 --reasoning-format deepseek

# Alternativa rápida con margen de VRAM (KV-Q8, ~16K)
llama-server -m K2-Horizon-7B-Q4_K_M.gguf \
  -ngl 99 -c 16384 -ctk q8_0 -ctv q8_0 -b 256 -ub 128 \
  --temp 1.0 --top-p 0.95 --reasoning-format deepseek

# llama.cpp directo desde Hugging Face
llama-server -hf IFM/K2-Horizon-7B-GGUF:Q4_K_M

# vLLM (receta oficial de la ficha)
vllm serve IFM/K2-Horizon-7B \
  --trust-remote-code --dtype bfloat16 --max-model-len 131072 \
  --tensor-parallel-size 1 \
  --reasoning-parser k2_horizon \
  --enable-auto-tool-choice --tool-call-parser k2_horizon

# SGLang (revisión validada en el cookbook oficial)
sglang serve --model-path IFM/K2-Horizon-7B \
  --revision 69ada542b68fe13d767479db2ab9421baff88681 \
  --tp 1 --dtype bfloat16 --attention-backend fa3 \
  --reasoning-parser k2_horizon --host 0.0.0.0 --port 30000
```

Notas: en RTX 5060 8GB con `-c 32768` y KV-Q4 el contexto útil medido es ~19K con recall 3/3 (a 25/50/75 %), a unos 14 t/s de generación y 1,1 GB libres; a partir de ~30K hay que pasar a `-ngl 32` (4 capas en CPU, ~5 t/s). Con f16 a 8K cabe entero en GPU. Ollama: `ollama run hf.co/IFM/K2-Horizon-7B-GGUF:Q4_K_M`. Versiones anteriores de llama.cpp sin la arquitectura `k2-horizon` no cargan el GGUF.

Sources:
- [IFM/K2-Horizon-7B-GGUF en Hugging Face](https://huggingface.co/IFM/K2-Horizon-7B-GGUF)
- [IFM/K2-Horizon-7B (modelo original)](https://huggingface.co/IFM/K2-Horizon-7B)
- [Introducing K2 Horizon (blog IFM)](https://ifm.ai/blog/k2/)
- [K2-Horizon-7B en Artificial Analysis](https://artificialanalysis.ai/models/k2-horizon-7b)
