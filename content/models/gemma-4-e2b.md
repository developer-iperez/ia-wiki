---
title: Gemma-4-E2B
---

Modelo de la familia [Gemma 4](https://huggingface.co/collections/google/gemma-4) (Google DeepMind, abril 2026), publicado en Hugging Face como [google/gemma-4-E2B](https://huggingface.co/google/gemma-4-E2B) (base) y [google/gemma-4-E2B-it](https://huggingface.co/google/gemma-4-E2B-it) (instructivo). Es el miembro más pequeño de la familia: 2.3B de parámetros efectivos (5.1B con embeddings), multimodal (texto + imagen + vídeo + audio → texto) con 128K de contexto, pensado para ejecución on-device (móvil, Raspberry Pi, portátil sin GPU).

> Nota: se incluye aquí porque no existe ningún "Gemma 4 E3B" — la familia E solo tiene [[models/gemma-4-e4b|Gemma-4-E4B]] (4.5B efectivos) y este E2B (2.3B efectivos).

## Highlights

- **El Gemma 4 más pequeño y el único que cabe en un teléfono**: ~2.58 GB en 4-bit, ~48 tok/s en un ROG Phone 9 Pro; Google lo declara apto para Raspberry Pi y Jetson Orin Nano totalmente offline.
- **Parámetros "efectivos" (Per-Layer Embeddings)**: cada capa del decoder tiene su propio embedding pequeño por token; las tablas solo sirven para lookups, de ahí que el conteo efectivo (2.3B) sea menos de la mitad del total (5.1B).
- **Multimodal nativo completo pese al tamaño**: texto, imagen (resolución/aspecto variable, presupuesto de 70–1120 tokens visuales), vídeo (hasta 60 s como frames) y audio (ASR y traducción, hasta 30 s).
- **Razonador configurable**: modo thinking activable/desactivable (`enable_thinking` en la chat template, token `<|think|>` en el system prompt); la familia introduce además soporte nativo del rol `system`.
- **Apache 2.0** (a diferencia de Gemma 3, que usaba Gemma Terms of Use) y variantes QAT/mobile (wNa8o8, QAT Q4_0, LiteRT-LM) optimizadas para NPU/CPU móvil.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 2.3B efectivos (5.1B con embeddings), denso BF16 |
| Capas | 35, híbrida local SWA-512 + global (final siempre global), vocabulario 262K |
| Contexto nativo | 131.072 tokens (128K) |
| Contexto extendido | no documentado (nativo 128K; los medianos de la familia llegan a 256K) |
| Modalidad | texto + imagen + vídeo + audio → texto (`any-to-any` / `image-text-to-text`) |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | Transformers (`AutoModelForMultimodalLM`), vLLM, SGLang, llama.cpp / Ollama / LM Studio (vía cuantizaciones de la comunidad), MLX (vía Unsloth), LiteRT-LM y QAT mobile on-device |

## Resumen esquemático de benchmarks

Comparativa según la model card oficial (modelos instructivos, valores del vendor):

| Benchmark | Gemma-4-E2B | Gemma-4-E4B | Gemma 3n E2B (generación anterior) |
|---|---|---|---|
| MMLU-Pro (conocimiento) | 60.0 | 69.4 | — |
| AIME 2026 sin herramientas (mates) | 37.5 | 42.5 | — |
| LiveCodeBench v6 (código) | 44.0 | 52.0 | — |
| Codeforces ELO (código) | 633 | 940 | — |
| GPQA Diamond (conocimiento) | 43.4 | 58.6 | — |
| τ² medio (agentes) | 24.5 | 42.2 | — |
| BigBench Extra Hard (razonamiento) | 21.9 | 33.1 | — |
| MMMU-Pro (visión) | 44.2 | 52.6 | — |
| MATH-Vision (visión+mates) | 52.4 | 59.5 | — |
| MRCR v2 8-needle 128K (contexto largo) | 19.1 | 25.4 | — |

Lectura rápida: queda por debajo de [[models/gemma-4-e4b|Gemma-4-E4B]] en todas las filas (esperable con la mitad de parámetros efectivos), pero duplica o triplica a la generación anterior en código y agentes (Codeforces 633 frente a 110 de Gemma 3 27B; τ² 24.5 frente a 16.2). Frente a los solo-texto del wiki de tamaño similar ([[models/lfm2.5-2.6b|LFM2.5-2.6B]], [[models/spark-x2.5-1.7b|Spark-X2.5-1.7B]], [[models/minicpm5-2b|MiniCPM5-2B]]) su baza no es el texto puro sino ser el único con visión + audio nativos corriendo en un móvil.

### Artificial Analysis (medición independiente, consultado 2026-09-21)

- [Página del modelo en AA (reasoning)](https://artificialanalysis.ai/models/gemma-4-e2b): Intelligence Index **8 (estimado, v4.3.2)**, puesto #67/142; sin velocidad ni precio publicados (modelo para self-host / on-device, sin proveedores API rastreados).
- En el [artículo de lanzamiento de AA](https://artificialanalysis.ai/articles/gemma-4-everything-you-need-to-know) (abril 2026, metodología anterior) el E2B reasoning marcó **15 puntos** (+10 sobre Gemma 3n E2B), con AA-Omniscience −24 — mejor (menos alucinación) que el 31B (−45). La diferencia con el 8 actual es cambio de versión del índice, no del modelo.
- En el [estudio de AA en móvil](https://artificialanalysis.ai/articles/mobile-phone-intelligence-inference) no aparece aún este modelo (el test es anterior a su lanzamiento); como referencia, [[models/lfm2.5-2.6b|LFM2.5-2.6B]] compartía la mejor nota media (63) en iPhone 17 Pro.
- Lectura: AA lo sitúa en la mediana de su clase (8, mediana 8) — por debajo de [[models/minicpm5-2b|MiniCPM5-2B]] (13) y [[models/granite-4.2-3b|Granite-4.2-3B]] (9); su caso de uso es correr offline donde esos no caben, no ganar el índice.

## Configuración recomendada

**Parámetros de muestreo** (estándar oficial, todos los casos de uso):

| temperature | top_p | top_k |
|---|---|---|
| 1.0 | 0.95 | 64 |

- **Thinking**: se controla con `enable_thinking=True/False` en `apply_chat_template` (o token `<|think|>` al inicio del system prompt). En multi-turno, no reenviar el thinking de turnos anteriores (salvo turnos con tool calls, donde se conserva).
- **Orden de modalidades**: imagen **antes** del texto, audio **después** del texto. Presupuesto visual configurable: 70/140/280/560/1120 tokens (bajo para clasificación/caption/vídeo, alto para OCR/documentos).
- **Límites**: audio máx. 30 s, vídeo máx. 60 s (1 frame/s).

```bash
# Transformers (pruebas rápidas)
pip install -U transformers torch accelerate  # + torchvision librosa para imagen/audio

# vLLM (OpenAI-compatible)
vllm serve google/gemma-4-E2B-it --port 8000

# SGLang
python -m sglang.launch_server --model-path google/gemma-4-E2B-it \
  --host 0.0.0.0 --port 30000

# On-device / móvil (LiteRT-LM)
litert-lm run \
  --from-huggingface-repo=litert-community/gemma-4-E2B-it-litert-lm \
  gemma-4-E2B-it.litertlm --prompt="What is the capital of France?"
```

```python
from transformers import AutoProcessor, AutoModelForMultimodalLM

MODEL_ID = "google/gemma-4-E2B-it"
processor = AutoProcessor.from_pretrained(MODEL_ID)
model = AutoModelForMultimodalLM.from_pretrained(MODEL_ID, dtype="auto", device_map="auto")

messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Write a short joke about saving RAM."},
]
inputs = processor.apply_chat_template(
    messages, tokenize=True, return_dict=True, return_tensors="pt",
    add_generation_prompt=True, enable_thinking=False,
).to(model.device)
outputs = model.generate(**inputs, max_new_tokens=1024)
print(processor.decode(outputs[0][inputs["input_ids"].shape[-1]:], skip_special_tokens=False))
```

Rendimiento on-device orientativo (LiteRT-LM, modelo 2.58 GB): ~47 tok/s de decode en GPU de S26 Ultra / iPhone 17 Pro, ~8 tok/s en CPU de Raspberry Pi 5 16GB. Para PC sin GPU o Apple Silicon, usar las [cuantizaciones GGUF de la comunidad](https://huggingface.co/models?other=base_model%3Aquantized%3Agoogle%2Fgemma-4-E2B) o la [versión MLX 4-bit de Unsloth](https://huggingface.co/unsloth/gemma-4-E2B-it-UD-MLX-4bit). En móvil se pide Snapdragon 8 Gen 3 / Dimensity 9300+ con 12 GB de RAM mínimo.

Sources:
- [google/gemma-4-E2B en Hugging Face](https://huggingface.co/google/gemma-4-E2B)
- [google/gemma-4-E2B-it en Hugging Face](https://huggingface.co/google/gemma-4-E2B-it)
- [Colección Gemma 4](https://huggingface.co/collections/google/gemma-4)
- [Gemma 4 Technical Report (arXiv 2607.02770)](https://arxiv.org/abs/2607.02770)
- [Gemma 4 en Artificial Analysis (reasoning)](https://artificialanalysis.ai/models/gemma-4-e2b)
- [Gemma 4: everything you need to know (AA)](https://artificialanalysis.ai/articles/gemma-4-everything-you-need-to-know)
- [Gemma 4 en Google AI Edge (rendimiento on-device)](https://ai.google.dev/edge/litert-lm/models/gemma-4)
