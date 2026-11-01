---
title: Qwen3.6-35B-A3B
---

Modelo MoE de la familia [Qwen3.6](https://huggingface.co/collections/Qwen/qwen36) (Alibaba / Qwen Team), publicado como primer peso abierto de la serie 3.6 en [Qwen/Qwen3.6-35B-A3B](https://huggingface.co/Qwen/Qwen3.6-35B-A3B) y en el [blog de Qwen](https://qwen.ai/blog?id=qwen3.6-35b-a3b). Tiene 35B totales con 3B activos por token, pensado para agentes de código y uso de herramientas en local con una GPU de consumo más RAM de sistema.

## Highlights

- **Enfoque en código agentico**: según la ficha oficial, mejora el trabajo con flujos de frontend y razonamiento a nivel de repositorio; el release prioriza estabilidad y utilidad real frente a la versión 3.5.
- **Preservación del pensamiento**: opción `preserve_thinking` para retener el razonamiento de mensajes históricos, pensada para agentes (mejor consistencia y menos tokens redundantes).
- **Arquitectura híbrida**: 40 capas en bloques `10 × (3 × Gated DeltaNet → MoE, 1 × Gated Attention → MoE)`, con 256 expertos de los que se activan 8 enrutados + 1 compartido.
- **Multimodal**: encoder de visión integrado (imagen y vídeo, entrada); la salida es solo texto.
- **Contexto largo**: 262.144 tokens nativos, ampliable a ~1.010.000 con YaRN (con la advertencia de la ficha sobre escalado estático).

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 35B totales, 3B activos (MoE, 256 expertos, 8 enrutados + 1 compartido) |
| Capas | 40; híbrido Gated DeltaNet + Gated Attention, ambos seguidos de MoE |
| Contexto nativo | 262.144 tokens (262K) |
| Contexto extendido | hasta ~1.010.000 tokens con YaRN (`factor` 4.0, configurable) |
| Modalidad | texto + imagen + vídeo (entrada), salida solo texto |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | Transformers, vLLM (≥0.19), SGLang (≥0.5.10), KTransformers; GGUF para llama.cpp/Ollama/LM Studio es de la comunidad (no documentado en la ficha oficial) |

## Resumen esquemático de benchmarks

Valores del vendor (ficha oficial de Hugging Face / blog de Qwen), no reproducidos de forma independiente. Columnas: Qwen3.6-35B-A3B frente a su predecesor directo y a dos modelos de tamaño similar que aparecen en la misma tabla del vendor:

| Benchmark | Qwen3.6-35B-A3B | Qwen3.5-35B-A3B | Gemma4-26B-A4B | Qwen3.5-27B (denso) |
|---|---|---|---|---|
| SWE-bench Verified (agente código) | 73.4 | 70.0 | 17.4 | 75.0 |
| Terminal-Bench 2.0 (terminal) | 51.5 | 40.5 | 34.2 | 41.6 |
| MMLU-Pro (conocimiento) | 85.2 | 85.3 | 82.6 | 86.1 |
| GPQA (ciencia, razonamiento) | 86.0 | 84.2 | 82.3 | 85.5 |
| AIME26 (mates) | 92.7 | 91.0 | 88.3 | 92.6 |
| TAU3-Bench (agente general) | 67.2 | 68.9 | 59.0 | 68.4 |

Lectura rápida: destaca en código agentico y terminal frente a su predecesor (SWE-bench Verified 73.4 y Terminal-Bench 51.5) y compite de tú a tú con el denso Qwen3.5-27B en conocimiento y mates; en TAU3-Bench queda ligeramente por debajo del 3.5-35B-A3B y del 27B denso.

### Artificial Analysis (medición independiente, consultado 2026-11-01)

- **Intelligence Index v4.3.2: 18** (copia de AA del 2026-10-15 en Wayback Machine, variante *reasoning*). La [página actual](https://artificialanalysis.ai/models/qwen3-6-35b-a3b) muestra el índice v5.0 como N/A.
- Datos de AA visibles hoy: 36B totales / 3B activos, contexto 262k, texto+imagen+vídeo de entrada, licencia Apache 2.0, ~133 tokens/s en la API de Alibaba (precio $0.38 entrada / $2.25 salida por millón).

## Configuración recomendada

Sampling oficial, según la ficha de Hugging Face (el modelo piensa por defecto):

| Modo | temperature | top_p | top_k | presence_penalty |
|---|---|---|---|---|
| Thinking, uso general | 1.0 | 0.95 | 20 | 1.5 |
| Thinking, código de precisión (p. ej. WebDev) | 0.6 | 0.95 | 20 | 0.0 |
| Instruct / no-thinking | 0.7 | 0.80 | 20 | 1.5 |

- **Longitud de salida**: la ficha recomienda 32.768 tokens para la mayoría de consultas y hasta 81.920 para problemas muy complejos (mates, competición).
- **Activación de modos**: thinking es el comportamiento por defecto. Para desactivarlo, `chat_template_kwargs: {"enable_thinking": false}` (en Model Studio, `enable_thinking: false` directamente). Qwen3.6 no soporta los *soft switches* `/think` y `/nothink` de Qwen3.
- **Contexto**: la ficha aconseja mantener al menos 128K de contexto para preservar las capacidades de razonamiento.

```bash
# vLLM (vllm>=0.19)
vllm serve Qwen/Qwen3.6-35B-A3B --port 8000 --max-model-len 262144 --reasoning-parser qwen3

# vLLM con tool calling
vllm serve Qwen/Qwen3.6-35B-A3B --port 8000 --max-model-len 262144 \
  --reasoning-parser qwen3 --enable-auto-tool-choice --tool-call-parser qwen3_coder

# SGLang (sglang>=0.5.10)
python -m sglang.launch_server --model-path Qwen/Qwen3.6-35B-A3B --port 8000 \
  --context-length 262144 --reasoning-parser qwen3 --tool-call-parser qwen3_coder

# Transformers (servidor ligero para pruebas)
pip install "transformers[serving]"
transformers serve Qwen/Qwen3.6-35B-A3B --port 8000 --continuous-batching
```

Notas: la ficha oficial usa 8 GPUs con tensor parallel (`--tp-size 8` / `--tensor-parallel-size 8`) para el contexto completo; en una sola GPU hay que reducir `--max-model-len`. Para solo texto, vLLM admite `--language-model-only` y libera memoria del encoder de visión. MTP (speculative decoding) está soportado con `--speculative-config '{"method":"qwen3_next_mtp","num_speculative_tokens":2}'` en vLLM.

En local con GGUF (cuantización Q4 de unsloth, ~20,8 GiB) el preset de ia-lab usa 131K de contexto en lugar de 262K por memoria; para extender más allá de 262K, la ficha documenta YaRN, pero es escalado estático y puede degradar textos cortos, así que solo conviene activarlo si el caso lo exige.

Sources:
- [Qwen/Qwen3.6-35B-A3B en Hugging Face](https://huggingface.co/Qwen/Qwen3.6-35B-A3B)
- [Blog de Qwen: Qwen3.6-35B-A3B](https://qwen.ai/blog?id=qwen3.6-35b-a3b)
- [Qwen3.6-35B-A3B en Artificial Analysis](https://artificialanalysis.ai/models/qwen3-6-35b-a3b)
