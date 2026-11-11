---
title: K2-Horizon-3.7B
---

Modelo de la familia [K2-Horizon](https://huggingface.co/collections/IFM/k2-horizon) del Institute of Foundation Models (IFM), publicado en Hugging Face como [IFM/K2-Horizon-3.7B](https://huggingface.co/IFM/K2-Horizon-3.7B) y en formato GGUF como [IFM/K2-Horizon-3.7B-GGUF](https://huggingface.co/IFM/K2-Horizon-3.7B-GGUF). En local usamos el fichero `K2-Horizon-4B-Q4_K_M.gguf` (2,94 GiB) de esa ficha oficial; el nombre "4B" es del fichero, el modelo es de 3,7B de núcleo denso. Es el miembro pequeño de la familia, pensado para agente y código en ventana corta en una GPU de consumo.


> **Estado en ia-lab (2026-11-11):** probado; no instalado actualmente. Su rol de denso de ventana corta lo cubren K2-Horizon-7B (purgado) y Ternary-Bonsai-2-27B.

## Highlights

- **Denso pequeño con contexto nativo de 512K**: 524.288 tokens nativos desde las fases de midtraining; la arquitectura `k2-horizon` es nueva en llama.cpp y requiere build ≥ b11454.
- **Entrenamiento completo y abierto**: datos, receta, código de entrenamiento y checkpoints intermedios publicados (pretrain, midtrain 1–4, RL por expertos fusionados, SFT en 2 fases).
- **Post-entreno con RL por expertos**: experto matemático, de código y STEM-código, fusionados después (ISO en atención, RAM en el resto).
- **Anti-doom-loop (FTPO)**: etapa final de ~110 pasos para eliminar los bucles de generación que alargan las respuestas hasta el límite de salida.
- **Razonamiento por nivel**: `reasoning_effort` (high por defecto) y formato de tool calls seleccionable (`json`, `xml`, `xml_typed`), con thinking en `reasoning_content`.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 3,7B denso (5,06B parámetros totales según el fichero GGUF; la ficha de AA indica 3,7B) |
| Capas | 36, GQA con 32 heads Q y 8 KV, head_dim 128 |
| Contexto nativo | 524.288 tokens (512K) |
| Contexto extendido | no documentado (nativo 512K) |
| Modalidad | solo texto, EN |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | vLLM (`--reasoning-parser k2_horizon`, `--tool-call-parser k2_horizon`), SGLang (recomendado con FA3), Transformers (`trust_remote_code`), llama.cpp (GGUF, build ≥ b11454), Ollama, LM Studio |

## Resumen esquemático de benchmarks

Comparativa según la ficha GGUF oficial (valores del vendor, en %). La columna del modelo es la del fichero que usamos en local:

| Benchmark | K2-Horizon-3.7B | Qwen3.5-4B | Granite 4.2-3B | Nemotron 3 Nano-4B |
|---|---|---|---|---|
| HMMT Feb 2026 (mates) | 70,5 | 61,6 | 57,2 | 34,7 |
| SWE-bench Verified (agente código) | 68,6 | 41,2 | 32,2 | 1,8 |
| GPQA Diamond (ciencia) | 65,4 | 77,1 | 55,9 | 51,3 |
| Terminal-Bench 2.1 (terminal agéntico) | 25,1 | 25,8 | 13,9 | 3,7 |
| BFCL v4 (function calling) | 50,9 | 55,7 | 50,8 | 36,8 |
| HLE (razonamiento experto) | 12,9 | 9,9 | 6,6 | 4,9 |

Lectura rápida: destaca en mates y en SWE-bench Verified frente a comparables de 3–4B; en GPQA queda por detrás de Qwen3.5-4B y en tool-use (BFCL) también. Atención: la ficha GGUF cuantizada muestra caídas frente a BF16 (AIME avg 74,1 vs 91,6; GPQA 58,5 vs 68,6), así que estas cifras son del BF16 o de la ficha general, no del Q4_K_M en local.

### Artificial Analysis: última versión (consultado 2026-11-11)

- **AA Progress Index v1: 242** ([K2 Horizon 3.7B en AA](https://artificialanalysis.ai/models/k2-horizon-3-7b)). Es el índice que AA usa ahora en la comparativa de modelos open source pequeños; no es comparable con el Intelligence Index v4.3.2.
- **AA Intelligence Index v5.0.1: N/A** en la ficha del modelo.
- **Histórico:** Intelligence Index v4.3.2 = 16, consultado vía Wayback el 2026-10-15.

### Histórico: Artificial Analysis v4.3.2 (consulta original)

- **Intelligence Index v4.3.2: 16** (copia de AA del 2026-10-15 en Wayback Machine). AA lo lista como *K2 Horizon 3.7B* (slug `k2-horizon-3-7b`; `k2-horizon-4b` devuelve 404). En la web actual el índice v5.0 aparece como N/A.
- Según la misma página: 3,7B, texto→texto, 524K de contexto, Apache 2.0, reasoning y open weights.

## Configuración recomendada

- **Sampling oficial**: `temperature=1.0`, `top_p=0.95`.
- **Razonamiento**: `reasoning_effort="high"` por defecto (la ficha pide siempre high para reproducir resultados; medium/low solo para investigación). Se pasa por petición en `chat_template_kwargs`.
- **Tool calls**: `tool_call_format` `xml` (por defecto), `json` o `xml_typed` en `chat_template_kwargs`.
- **Longitud de salida**: al menos 32.768 tokens; si se corta el razonamiento la respuesta falla. En nuestras sondas con 8.192 bastó para recall, pero los problemas duros piensan mucho más.

```bash
# llama.cpp (build >= b11454; probado con b11501)
llama-server -m K2-Horizon-4B-Q4_K_M.gguf -c 16384 -ngl 34 \
  --cache-type-k q4_0 --cache-type-v q4_0 --reasoning-format deepseek \
  --temp 1.0 --top-p 0.95 --port 8080

# vLLM (ficha oficial)
vllm serve IFM/K2-Horizon-3.7B --trust-remote-code --dtype bfloat16 \
  --tensor-parallel-size 1 --reasoning-parser k2_horizon \
  --enable-auto-tool-choice --tool-call-parser k2_horizon

# SGLang (receta validada en la ficha)
sglang serve --model-path IFM/K2-Horizon-3.7B --tp 1 --dtype bfloat16 \
  --attention-backend fa3 --reasoning-parser k2_horizon --port 30000
```

Notas de troubleshooting:

- **Build**: con llama.cpp anterior a b11454 la arquitectura `k2-horizon` no carga.
- **Memoria KV**: unos 144 KB/token en f16 (la más cara del repo por token). En 8 GB de VRAM, f16 @8192 en full GPU no cabe y q4 @16384 en full GPU da OOM por el buffer de cómputo; hacen falta 2 capas en CPU (`-ngl 34`) desde ~12K. Para 32K con q4 hay que bajar a `-ngl 24`, a costa de ~2 t/s de generación.
- **Tools**: sin tools declarados en la petición el modelo no llama a ninguna herramienta (criterio nuestro, no incapacidad); con tools explícitos llama bien (`finish_reason=tool_calls` con argumentos correctos en nuestras sondas).
- **Cuantización**: Q4_K_M es el único fichero probado en local; Q5_K_M y Q6_K pesan algo más y la ficha los muestra mejores que Q4 en IFEval y AIME.

Sources:
- [IFM/K2-Horizon-3.7B en Hugging Face](https://huggingface.co/IFM/K2-Horizon-3.7B)
- [IFM/K2-Horizon-3.7B-GGUF en Hugging Face](https://huggingface.co/IFM/K2-Horizon-3.7B-GGUF)
- [K2-Horizon-3.7B en Artificial Analysis](https://artificialanalysis.ai/models/k2-horizon-3-7b)
- [Blog IFM: K2 Horizon](https://ifm.ai/blog/k2/)
