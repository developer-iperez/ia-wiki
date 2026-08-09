---
title: Qwen3.5-4B
---

Modelo de la familia [Qwen3.5](https://qwen.ai/blog?id=qwen3.5) (Alibaba), publicado en Hugging Face como [Qwen/Qwen3.5-4B](https://huggingface.co/Qwen/Qwen3.5-4B). Es multimodal (texto + imagen + vídeo → texto) y, dentro de la familia, es el tamaño más pequeño pensado para ejecución local en una sola GPU modesta.

## Highlights

- **Base visión-lenguaje unificada**: entrenamiento con fusión temprana de tokens multimodales — a diferencia de generaciones anteriores (Qwen3-VL), donde la parte visual se añadía después, aquí texto e imagen se entrenan de forma conjunta desde el principio.
- **Arquitectura híbrida eficiente**: combina capas de atención lineal (Gated DeltaNet) con capas de atención completa, buscando alto rendimiento de inferencia con poca latencia/coste.
- **RL a gran escala**: entrenamiento por refuerzo sobre entornos de millones de agentes con dificultad creciente, orientado a robustez en tareas reales (agentes, herramientas).
- **Cobertura de 201 idiomas y dialectos**.
- **Modo "thinking" por defecto**: genera un bloque `<think>...</think>` antes de la respuesta final (se puede desactivar, ver más abajo).

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | ~4.66 B (denso, **no** es Mixture-of-Experts — a diferencia de otros modelos Qwen con sufijo "A", este es un modelo denso) |
| Capas | 32, en bloques de 8 × (3 × [Gated DeltaNet → FFN] → 1 × [Atención completa → FFN]) — mayoría de capas con atención lineal y una de cada cuatro con atención completa |
| Contexto nativo | 262.144 tokens |
| Contexto extendido | hasta 1.010.000 tokens vía YaRN (escalado de RoPE), soportado por transformers/vLLM/SGLang/KTransformers |
| Modalidad | texto + imagen + vídeo → texto (`image-text-to-text`) |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | Hugging Face Transformers, vLLM, SGLang, KTransformers |

## Resumen esquemático de benchmarks

Comparativa (según la model card oficial) frente a otros modelos de tamaño similar o algo mayor. Valores aproximados, escala 0–100 salvo que se indique lo contrario:

| Benchmark | Qwen3.5-4B | Qwen3.5-9B | GPT-OSS-20B |
|---|---|---|---|
| MMLU-Pro (conocimiento) | 79.1 | 82.5 | 74.8 |
| GPQA Diamond (razonamiento) | 76.2 | 81.7 | 71.5 |
| LiveCodeBench v6 (código) | 55.8 | 65.6 | 74.6 |
| IFEval (seguir instrucciones) | 89.8 | 91.5 | 88.2 |
| AA-LCR (contexto largo) | 57.0 | 63.0 | 30.7 |
| MMMU (visión, conocimiento) | 77.6 | 78.4 | — (no multimodal) |

Lectura rápida: para su tamaño (4B), rinde notablemente bien en instrucciones y contexto largo, algo por debajo de su hermano de 9B en razonamiento/código duro (esperable), y es la única de las tres comparadas con capacidad visual nativa.

## Configuración recomendada

**Parámetros de muestreo** (varían según modo y tarea):

| Modo | temperature | top_p | top_k | presence_penalty |
|---|---|---|---|---|
| Thinking, tareas generales | 1.0 | 0.95 | 20 | 1.5 |
| Thinking, código preciso | 0.6 | 0.95 | 20 | 0.0 |
| Instruct/no-thinking, general | 0.7 | 0.8 | 20 | 1.5 |
| Instruct/no-thinking, razonamiento | 1.0 | 1.0 | 40 | 2.0 |

- `min_p=0.0`, `repetition_penalty=1.0` en todos los casos. `presence_penalty` entre 0–2 para evitar repeticiones (valores altos pueden mezclar idiomas).
- **Longitud de salida**: 32.768 tokens para uso normal; hasta 81.920 para problemas muy complejos (matemáticas/competición).
- El modelo piensa (`<think>`) por defecto. Para desactivarlo: `chat_template_kwargs: {"enable_thinking": false}` en la API de chat completions.
- En conversaciones multi-turno, no reenviar el contenido de `<think>` de turnos anteriores (ya lo gestiona la chat template por defecto).

**Servir localmente** (ejemplos con GPU única, `tensor-parallel-size 1`):

```bash
# vLLM
vllm serve Qwen/Qwen3.5-4B --port 8000 --tensor-parallel-size 1 \
  --max-model-len 262144 --reasoning-parser qwen3

# SGLang
python -m sglang.launch_server --model-path Qwen/Qwen3.5-4B --port 8000 \
  --tp-size 1 --mem-fraction-static 0.8 --context-length 262144 --reasoning-parser qwen3

# Hugging Face Transformers (opción más ligera para pruebas rápidas)
transformers serve --force-model Qwen/Qwen3.5-4B --port 8000 --continuous-batching
```

Si da error de memoria (OOM), reduce `--max-model-len`/`--context-length`, pero se recomienda mantener al menos 128K para no perder capacidad de razonamiento largo. Para superar los 262K nativos (hasta ~1M) hay que activar YaRN explícitamente vía `--hf-overrides` (vLLM) o modificando `rope_parameters` en `config.json` — ver la [model card](https://huggingface.co/Qwen/Qwen3.5-4B#processing-ultra-long-texts) para el JSON exacto.

Sources:
- [Qwen/Qwen3.5-4B en Hugging Face](https://huggingface.co/Qwen/Qwen3.5-4B)
- [Qwen3.5 blog post](https://qwen.ai/blog?id=qwen3.5)
