---
title: Gemma-4-E4B
---

Modelo de la familia [Gemma 4](https://huggingface.co/collections/google/gemma-4) (Google DeepMind, abril 2026), publicado en Hugging Face como [google/gemma-4-E4B](https://huggingface.co/google/gemma-4-E4B) (base) y [google/gemma-4-E4B-it](https://huggingface.co/google/gemma-4-E4B-it) (instructivo). Es el compacto "grande" de la familia E (on-device): 4.5B de parámetros efectivos (8B con embeddings), multimodal (texto + imagen + vídeo + audio → texto) con 128K de contexto, pensado para ejecución local en una sola GPU modesta o en móvil de gama alta.

> Nota: no existe ningún "Gemma 4 E3B". La familia E solo tiene dos miembros: [[models/gemma-4-e2b|Gemma-4-E2B]] (2.3B efectivos) y este E4B (4.5B efectivos).

## Highlights

- **Parámetros "efectivos" (Per-Layer Embeddings)**: cada capa del decoder tiene su propio embedding pequeño por token; las tablas son grandes pero solo sirven para lookups, de ahí que el conteo efectivo (4.5B) sea mucho menor que el total (8B).
- **Multimodal nativo completo en tamaño compacto**: texto, imagen (resolución/aspecto variable, presupuesto de 70–1120 tokens visuales), vídeo (hasta 60 s como frames) y audio (ASR y traducción, hasta 30 s) en un solo checkpoint.
- **Razonador configurable**: modo thinking activable/desactivable (`enable_thinking` en la chat template, token `<|think|>` en el system prompt); la familia introduce además soporte nativo del rol `system`.
- **Atención híbrida eficiente**: sliding-window local (512) intercalada con atención global (última capa siempre global), con claves/valores unificados y p-RoPE para contexto largo con poca memoria.
- **Apache 2.0** (a diferencia de Gemma 3, que usaba Gemma Terms of Use) y variantes QAT/mobile (wNa8o8, QAT Q4_0, LiteRT-LM) para servir en teléfono, Raspberry Pi o portátil.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 4.5B efectivos (8B con embeddings), denso BF16 |
| Capas | 42, híbrida local SWA-512 + global (final siempre global), vocabulario 262K |
| Contexto nativo | 131.072 tokens (128K) |
| Contexto extendido | no documentado (nativo 128K; los medianos de la familia llegan a 256K) |
| Modalidad | texto + imagen + vídeo + audio → texto (`any-to-any` / `image-text-to-text`) |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | Transformers (`AutoModelForMultimodalLM`), vLLM, SGLang, Docker Model Runner, llama.cpp / Ollama / LM Studio (vía cuantizaciones de la comunidad), LiteRT-LM y QAT mobile on-device |

## Resumen esquemático de benchmarks

Comparativa según la model card oficial (modelos instructivos, valores del vendor):

| Benchmark | Gemma-4-E4B | Gemma-4-E2B | Gemma 3 27B (sin thinking) |
|---|---|---|---|
| MMLU-Pro (conocimiento) | 69.4 | 60.0 | 67.6 |
| AIME 2026 sin herramientas (mates) | 42.5 | 37.5 | 20.8 |
| LiveCodeBench v6 (código) | 52.0 | 44.0 | 29.1 |
| Codeforces ELO (código) | 940 | 633 | 110 |
| GPQA Diamond (conocimiento) | 58.6 | 43.4 | 42.4 |
| τ² medio (agentes) | 42.2 | 24.5 | 16.2 |
| BigBench Extra Hard (razonamiento) | 33.1 | 21.9 | 19.3 |
| MMMU-Pro (visión) | 52.6 | 44.2 | 49.7 |
| MATH-Vision (visión+mates) | 59.5 | 52.4 | 46.0 |
| MRCR v2 8-needle 128K (contexto largo) | 25.4 | 19.1 | 13.5 |

Lectura rápida: supera a Gemma 3 27B en casi todo salvo MMLU-Pro/MMMLU puros, con un salto grande en código (LiveCodeBench 52.0 frente a 29.1) y agentes (τ² 42.2 frente a 16.2); frente a [[models/gemma-4-e2b|Gemma-4-E2B]] gana en todas las filas a cambio de ~2× memoria (~3.65 GB el modelo frente a ~2.58 GB). Es la única opción compacta del wiki con visión + audio nativos; en texto puro y contexto de 1M, [[models/spark-x2.5-4b|Spark-X2.5-4B]] o [[models/qwen3.5-4b|Qwen3.5-4B]] siguen por delante en contexto largo.

### Artificial Analysis (medición independiente, consultado 2026-09-21)

- [Página del modelo en AA (reasoning)](https://artificialanalysis.ai/models/gemma-4-e4b): Intelligence Index **9 (estimado, v4.3.2)**, puesto #51/142; velocidad ~42 tok/s (mediana DeepInfra, único proveedor API), precio $0.02/$0.10 por 1M tokens in/out, latencia al primer token ~0.85 s.
- [Variante non-reasoning](https://artificialanalysis.ai/models/gemma-4-e4b-non-reasoning): Intelligence Index **7 (estimado, v4.3.2)**, puesto #20/75 de su clase, ~43 tok/s.
- En el [artículo de lanzamiento de AA](https://artificialanalysis.ai/articles/gemma-4-everything-you-need-to-know) (abril 2026, metodología anterior) el E4B reasoning marcó **19 puntos** (+13 sobre Gemma 3n E4B), con AA-Omniscience −20 — mejor (menos alucinación) que el 31B (−45) y a la par de modelos mucho mayores. La diferencia con el 9 actual es cambio de versión del índice, no del modelo.
- Lectura: AA confirma el perfil del vendor (razonador compacto y barato, poco verboso en tokens), pero con índice compuesto por debajo de [[models/minicpm5-2b|MiniCPM5-2B]] (13) e igualado con [[models/granite-4.2-3b|Granite-4.2-3B]] (9); su ventaja diferencial es la multimodalidad (imagen+audio) que esos modelos no tienen.

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
vllm serve google/gemma-4-E4B-it --port 8000

# SGLang
python -m sglang.launch_server --model-path google/gemma-4-E4B-it \
  --host 0.0.0.0 --port 30000

# Docker Model Runner
docker model run hf.co/google/gemma-4-E4B-it

# On-device / móvil (LiteRT-LM)
litert-lm run \
  --from-huggingface-repo=litert-community/gemma-4-E4B-it-litert-lm \
  gemma-4-E4B-it.litertlm --prompt="What is the capital of France?"
```

```python
from transformers import AutoProcessor, AutoModelForMultimodalLM

MODEL_ID = "google/gemma-4-E4B-it"
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

Para CPU o despliegue ligero, usar las [cuantizaciones GGUF de la comunidad](https://huggingface.co/models?other=base_model%3Aquantized%3Agoogle%2Fgemma-4-E4B) con llama.cpp, Ollama o LM Studio. En PC se recomienda GPU de 8 GB mínimo (12–16 GB para 128K de contexto); por debajo hay offload a RAM y la velocidad cae en picado.

Sources:
- [google/gemma-4-E4B en Hugging Face](https://huggingface.co/google/gemma-4-E4B)
- [google/gemma-4-E4B-it en Hugging Face](https://huggingface.co/google/gemma-4-E4B-it)
- [Colección Gemma 4](https://huggingface.co/collections/google/gemma-4)
- [Gemma 4 Technical Report (arXiv 2607.02770)](https://arxiv.org/abs/2607.02770)
- [Gemma 4 en Artificial Analysis (reasoning)](https://artificialanalysis.ai/models/gemma-4-e4b)
- [Gemma 4 E4B non-reasoning en Artificial Analysis](https://artificialanalysis.ai/models/gemma-4-e4b-non-reasoning)
- [Gemma 4: everything you need to know (AA)](https://artificialanalysis.ai/articles/gemma-4-everything-you-need-to-know)
