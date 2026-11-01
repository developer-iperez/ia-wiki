---
title: Gemma 4 26B-A4B
---

Modelo MoE de la familia [Gemma 4](https://huggingface.co/collections/google/gemma-4) (Google DeepMind), publicado en Hugging Face como [google/gemma-4-26B-A4B](https://huggingface.co/google/gemma-4-26B-A4B). Es la variante de tamaño medio-grande orientada a servidores y workstations de consumo: con 25,2B totales pero solo 3,8B activos por token, responde casi a la velocidad de un 4B, con calidad cercana al denso 31B.

## Highlights

- **MoE con pocos activos**: 128 expertos con 8 activos por token más 1 compartido; 25,2B totales y 3,8B activos. Según el vendor, corre casi tan rápido como un modelo de 4B.
- **Atención híbrida**: capas de ventana deslizante (1024 tokens) intercaladas con capas globales, la última siempre global; las globales usan Keys/Values unificadas y Proportional RoPE (p-RoPE) para contexto largo.
- **Contexto de 256K tokens** nativo, uno de los dos tamaños "medios" de la familia junto al 31B denso (el resto de tamaños pequeños llegan a 128K).
- **Razonamiento configurable y tool-use nativo**: modo thinking activable con el token `<|think|>` en el system prompt; soporte de function calling y rol `system` nativo.
- **Multimodal texto + imagen** (resolución y aspect ratio variables, presupuestos de tokens visuales 70–1120). Audio y vídeo no figuran en la ficha para este modelo (el audio está en E2B, E4B y 12B).

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 25,2B totales, MoE con 3,8B activos (8 de 128 expertos + 1 compartido) |
| Capas | 30; ventana deslizante de 1024 tokens con capa global final; vocabulario 262K |
| Contexto nativo | 256K tokens |
| Contexto extendido | no documentado (nativo 256K) |
| Modalidad | texto + imagen → texto (Artificial Analysis lista también vídeo como entrada) |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | Transformers, vLLM, SGLang, llama.cpp (GGUF de la comunidad), Ollama, LM Studio, Docker Model Runner |

## Resumen esquemático de benchmarks

Comparativa según la model card oficial de Google (valores del vendor, tabla de modelos instruct). Escala 0–100:

| Benchmark | Gemma 4 26B-A4B | Gemma 4 31B (denso) | Gemma 4 E4B | Gemma 3 27B (no think) |
|---|---|---|---|---|
| MMLU Pro (conocimiento) | 82.6 | 85.2 | 69.4 | 67.6 |
| GPQA Diamond (ciencia) | 82.3 | 84.3 | 58.6 | 42.4 |
| AIME 2026 sin tools (mates) | 88.3 | 89.2 | 42.5 | 20.8 |
| LiveCodeBench v6 (código) | 77.1 | 80.0 | 52.0 | 29.1 |
| Tau2, media de 3 (agentes/tool-use) | 68.2 | 76.9 | 42.2 | 16.2 |
| MRCR v2 8 needles 128K (contexto largo) | 44.1 | 66.4 | 25.4 | 13.5 |

Lectura rápida: frente al 31B denso pierde poco en conocimiento, ciencia y matemáticas (1–3 puntos), algo más en código (−3), y se queda más atrás en agentes (Tau2, −9) y en contexto largo (MRCR, −22), donde el 31B lo supera con holgura; a cambio, con 3,8B activos, su coste de inferencia es muy inferior. Frente a la generación anterior (Gemma 3 27B) mejora de forma clara en todo lo de razonamiento y código.

### Artificial Analysis (medición independiente, consultado 2026-11-01)

- **Intelligence Index v4.3.2: 17 (estimado por AA)** para la variante *reasoning* y **13 (estimado)** para *non-reasoning* (copia de AA del 2026-10-15 en Wayback Machine). En la web actual ambas muestran el índice v5.0 como N/A.
- Datos que AA publica hoy: 25,2B totales / 3,8B activos, contexto 256K, licencia Apache 2.0, velocidad de salida ~100 tok/s (reasoning) y ~119 tok/s (non-reasoning), precio ~$0,10 / $0,34 por millón de tokens de entrada/salida (reasoning).

## Configuración recomendada

- **Sampling oficial** (uniforme para todos los casos de uso): `temperature=1.0, top_p=0.95, top_k=64`.
- **Thinking**: desactivado por defecto. Para activarlo, incluir el token `<|think|>` al inicio del system prompt (en `apply_chat_template`, `enable_thinking=True`). El razonamiento se emite como `<|channel>thought\n…<channel|>` seguido de la respuesta; el parser de Transformers lo separa.
- **Con thinking desactivado**, este modelo sigue generando el bloque vacío `<|channel>thought\n<channel|>` antes de la respuesta (a diferencia de E2B/E4B).
- **Multi-turno**: no reinsertar el razonamiento de turnos anteriores en el historial, salvo en turnos con tool calls.
- **Longitud de salida**: el ejemplo oficial usa `max_new_tokens=1024`; con thinking activo conviene dar margen amplio (varios miles de tokens).

```bash
# vLLM
vllm serve "google/gemma-4-26B-A4B"

# SGLang
python3 -m sglang.launch_server \
  --model-path "google/gemma-4-26B-A4B" \
  --host 0.0.0.0 --port 30000

# Transformers
pip install -U transformers torch accelerate
# AutoModelForMultimodalLM.from_pretrained("google/gemma-4-26B-A4B", device_map="auto")

# Docker Model Runner
docker model run hf.co/google/gemma-4-26B-A4B
```

Notas: la ficha oficial carga con `AutoModelForMultimodalLM` (no con la clase de solo texto). Para llama.cpp/Ollama/LM Studio hay cuantizaciones de la comunidad (buscar en Hugging Face "gemma-4-26B-A4B" GGUF); al ser MoE, con la mayoría de expertos en CPU se gana VRAM a costa de velocidad. Troubleshooting: la memoria necesaria crece con el contexto (KV de 256K es grande), así que para equipos con poca VRAM conviene limitar `--max-model-len`/`-c` y cuantizar el KV cache. No hay cifras de memoria mínima en la ficha oficial: no documentado.

Sources:
- [google/gemma-4-26B-A4B en Hugging Face](https://huggingface.co/google/gemma-4-26B-A4B)
- [Gemma 4 Technical Report (arXiv 2607.02770)](https://arxiv.org/abs/2607.02770)
- [Gemma 4, blog de lanzamiento de Google](https://blog.google/innovation-and-ai/technology/developers-tools/gemma-4/)
- [Gemma 4 26B A4B (Reasoning) en Artificial Analysis](https://artificialanalysis.ai/models/gemma-4-26b-a4b)
- [Gemma 4 26B A4B (Non-reasoning) en Artificial Analysis](https://artificialanalysis.ai/models/gemma-4-26b-a4b-non-reasoning)
