---
title: LFM2.5-2.6B
---

Modelo de la familia [LFM2.5](https://www.liquid.ai/blog/lfm2-5-2-6b) (Liquid AI), publicado en Hugging Face como [LiquidAI/LFM2.5-2.6B](https://huggingface.co/LiquidAI/LFM2.5-2.6B). Es un modelo híbrido pensado para **despliegue on-device** (edge): agentes, uso de herramientas y flujos de contexto largo en hardware modesto (CPU de portátil, incluso teléfono).

## Highlights

- **Arquitectura híbrida LFM2**: 30 capas — 22 bloques de convolución corta con "double gating" + 8 bloques de atención GQA — orientada a inferencia rápida y bajo consumo de memoria.
- **Post-entrenamiento agéntico**: cuatro etapas (SFT en dos rondas, especialización por dominio con modelos "teacher", destilación on-policy multi-dominio y RL agéntico dentro de los harnesses agénticos más usados), lo que lo hace competitivo en uso de herramientas y tareas multi-paso frente a modelos hasta 4× más grandes.
- **Es un modelo de razonamiento puro**: siempre piensa antes de responder, añadiendo un bloque `<think>` automáticamente vía chat template (no se puede desactivar).
- **Inferencia muy eficiente**: 220 tok/s en Apple M5 Max y 113 tok/s en CPU AMD Ryzen, con menos de 2.5 GB de memoria; en GPU (H100) alcanza cerca de 15K tokens/s de salida en alta concurrencia.
- **16 idiomas soportados**, incluido español.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 2.69 B (denso) |
| Capas | 30 (22 bloques de convolución corta "double-gated" + 8 de atención GQA) |
| Contexto nativo | 131.072 tokens (128K) |
| Modalidad | solo texto (`text-generation`), conversacional |
| Licencia | LFM Open License 1.0 (`license: other`, ver `LICENSE` del repo) |
| Frameworks de inferencia | Transformers (≥5.0.0), vLLM, SGLang, llama.cpp (GGUF), MLX (Apple Silicon), LM Studio, ONNX Runtime |

## Resumen esquemático de benchmarks

Comparativa (según la model card oficial) frente a otros modelos sub-10B:

| Benchmark | LFM2.5-2.6B | Qwen3.5-4B | Qwen3.5-9B |
|---|---|---|---|
| AIME25 (matemáticas) | 51.87 | 49.33 | 56.07 |
| LiveCodeBench v6 (código) | 59.41 | 60.85 | 69.86 |
| IFBench (seguir instrucciones) | 59.17 | 48.40 | 56.47 |
| Multi-IF (instrucciones multi-turno) | 80.07 | 55.67 | 62.55 |
| BFCLv4 (function calling) | 56.88 | 50.56 | 60.13 |
| ToolSandbox (uso de herramientas) | 77.83 | 75.55 | 76.44 |

Lectura rápida: con menos de un tercio de los parámetros de Qwen3.5-9B, LFM2.5-2.6B iguala o supera a modelos bastante más grandes en instrucciones y uso de herramientas/agentes, aunque se queda algo por debajo en código y matemáticas puras. Los propios autores no lo recomiendan para programación agéntica ni tareas intensivas en conocimiento.

## Configuración recomendada

**Parámetros de muestreo** (únicos, no varían por modo):

| temperature | top_k | repetition_penalty |
|---|---|---|
| 0.1 | 50 | 1.1 |

- Usa formato tipo ChatML (`<|im_start|>`/`<|im_end|>`), compatible con `tokenizer.apply_chat_template()`.
- El modelo siempre razona: añade automáticamente un bloque `<think>` al inicio de la respuesta del asistente.
- Tool calling en formato "Pythonic" por defecto (`<|tool_call_start|>[func(arg=...)]<|tool_call_end|>`); puede forzarse salida en JSON indicándolo en el system prompt.
- Casos de uso recomendados: agentes, uso de herramientas, extracción de datos, RAG, flujos de contexto largo. No recomendado para programación agéntica compleja ni tareas muy intensivas en conocimiento.

```python
# Transformers (>=5.0.0)
from transformers import AutoModelForCausalLM, AutoTokenizer, TextStreamer

model_id = "LiquidAI/LFM2.5-2.6B"
model = AutoModelForCausalLM.from_pretrained(model_id, device_map="auto", dtype="bfloat16")
tokenizer = AutoTokenizer.from_pretrained(model_id)

input_ids = tokenizer.apply_chat_template(
    [{"role": "user", "content": "What is C. elegans?"}],
    add_generation_prompt=True, return_tensors="pt", tokenize=True,
)["input_ids"].to(model.device)

model.generate(input_ids, do_sample=True, temperature=0.1, top_k=50,
                repetition_penalty=1.1, max_new_tokens=512,
                streamer=TextStreamer(tokenizer, skip_prompt=True, skip_special_tokens=True))
```

Para CPU/local sin GPU, usar la variante [LFM2.5-2.6B-GGUF](https://huggingface.co/LiquidAI/LFM2.5-2.6B-GGUF) con llama.cpp o LM Studio; en Apple Silicon, la variante [LFM2.5-2.6B-MLX](https://huggingface.co/LiquidAI/LFM2.5-2.6B-MLX). Cabe en menos de 2.5 GB de memoria incluso sin cuantizar agresivamente, por lo que no suele requerir troubleshooting de OOM salvo en dispositivos muy limitados.

Sources:
- [LiquidAI/LFM2.5-2.6B en Hugging Face](https://huggingface.co/LiquidAI/LFM2.5-2.6B)
- [LFM2.5-2.6B blog post](https://www.liquid.ai/blog/lfm2-5-2-6b)
- [LFM2 Technical Report (arXiv:2511.23404)](https://arxiv.org/abs/2511.23404)
