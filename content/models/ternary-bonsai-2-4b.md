---
title: Ternary-Bonsai-2-4B
---

Modelo de la familia [Bonsai 2 Small](https://huggingface.co/collections/prism-ml/bonsai-2-small) (PrismML), publicado en Hugging Face como [prism-ml/Ternary-Bonsai-2-4B-gguf](https://huggingface.co/prism-ml/Ternary-Bonsai-2-4B-gguf). Es un modelo de visión y lenguaje de ~4B con pesos ternarios {-1, 0, +1}, derivado de Qwen3.5-4B, pensado para ejecución local en una GPU modesta o en portátil.

## Highlights

- **Pesos ternarios con escala por grupo**: embeddings, proyecciones de atención, MLP y LM head (atado) en ternario con escala FP16 por grupo; normalizaciones y el camino de estado lineal en F32.
- **Tres empaquetados del mismo modelo**: PTQ1_0 (0,95 GB), PQ2_0 (1,15 GB) y Q2_0 (1,21 GB). Solo Q2_0 corre en llama.cpp upstream; PQ2_0 y PTQ1_0 requieren el fork de PrismML.
- **Arquitectura híbrida heredada de Qwen3.5**: 3 capas de atención lineal por cada 1 de atención completa (24 + 8), lo que contiene el coste del KV cache en contextos largos.
- **Visión opcional**: proyector mmproj (Q8_0 recomendado) que solo se carga cuando llega una imagen.
- **Speculative decoding opcional**: variante MTP (PQ2_0 con cabeza de predicción multi-token) y drafter DFlash2, ambos con el fork de PrismML.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | ~4,21B language model (32 bloques, hidden 2560, embedding y LM head atados) + ~0,33B torre de visión (24 bloques) |
| Capas | 32 bloques híbridos: 24 de atención lineal + 8 de atención completa (3:1), SwiGLU, RoPE, RMSNorm |
| Contexto nativo | 262.144 tokens (heredado del modelo base) |
| Contexto extendido | no documentado; en la práctica local, KV Q8 a 65.536 tokens funciona con recall 3/3 en 41 K (ver nota) |
| Modalidad | texto + imagen (`image-text-to-text`) |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | llama.cpp (fork PrismML para PQ2_0/PTQ1_0/MTP/DFlash2; upstream b11555+ solo Q2_0), MLX (mlx-vlm ≥ 0.7.8), vLLM (según la ficha) |

## Resumen esquemático de benchmarks

La ficha oficial de Hugging Face **no publica tablas de benchmarks de calidad** (ni MMLU, GPQA, LiveCodeBench ni similares), así que no hay valores del vendor que comparar. Lo que sí hay son cifras de la ficha y medidas locales de velocidad con `llama-bench` (build 11621, CUDA, `-ngl 99`, tamaño de prompt 512 y 128 tokens generados), que no son benchmarks de calidad:

| Medida | Ternary-Bonsai-2-4B (Q2_0) | Ternary-Bonsai-2-9B (Q2_0) |
|---|---|---|
| Tamaño fichero Q2_0 (ficha) | 1,21 GB | 2,56 GB |
| Prompt (pp512), medido localmente | ~186 t/s | ~98 t/s |
| Generación (tg128), medido localmente | ~52 t/s | ~28 t/s |

Artificial Analysis: no tiene página para este modelo (`/models/ternary-bonsai-2-4b` responde 404 el 2026-11-01), así que no hay medición independiente.

Lectura rápida: no hay una comparativa de calidad publicada que permita situarlo frente a otros 4B; el dato técnico fuerte es el contexto nativo de 262K y que la generación en GPU es aproximadamente el doble que la del 9B. Cualquier afirmación de calidad habrá que medirla en casa.

## Configuración recomendada

Sampling oficial de la ficha (la misma para todos los empaquetados y MLX):

| Modo | temp | top_p | top_k | min_p | presence_penalty | repetition_penalty |
|---|---|---|---|---|---|---|
| Thinking (por defecto) | 1.0 | 0.95 | 20 | 0.0 | 1.0 | 1.0 |
| No-thinking | 0.7 | 0.80 | 20 | 0.0 | 1.5 | 1.0 |

- **Thinking**: activo por defecto. Aun en prompts triviales genera 3000+ tokens de razonamiento según la ficha; en la configuración local, el 4B piensa de forma más concisa (~147 tokens en un caso trivial).
- **No-thinking**: enviar `"chat_template_kwargs": {"enable_thinking": false}` en cada petición (respuestas cortas, ~1 s).
- **Longitud de salida**: con thinking, la ficha recomienda al menos 8.192 tokens; clientes con `max_tokens` bajo recibirán respuestas vacías o truncadas.
- **Evitar temperature=0** con thinking activo: puede quedarse atascado razonando.

```bash
# llama.cpp upstream (build >= b11555), Q2_0, configuración local verificada
./llama-server -m Ternary-Bonsai-2-4B-Q2_0.gguf -ngl 99 -fa on -c 65536 \
  -ctk q8_0 -ctv q8_0 -b 256 -ub 128 --reasoning-format deepseek \
  --temp 1.0 --top-p 0.95 --top-k 20 --min-p 0 --presence-penalty 1.5 --port 8080

# Fork PrismML con PQ2_0 + visión (según la ficha oficial)
./bin/llama-server -m Ternary-Bonsai-2-4B-PQ2_0.gguf \
  --mmproj Ternary-Bonsai-2-4B-mmproj-Q8_0.gguf -ngl 99 -fa on -c 32768 --port 8080

# Speculative decoding con MTP (fork PrismML, ver nota de rendimiento)
./bin/llama-server -m Ternary-Bonsai-2-4B-PQ2_0-MTP-Q8_0.gguf -ngl 99 -fa on \
  --spec-type draft-mtp --spec-draft-n-max 2 --spec-draft-n-min 0 --spec-draft-p-min 0
```

Notas:
- **Solo Q2_0 en build stock**: `PQ2_0` y `PTQ1_0` se rechazan como "unknown type" fuera del fork de PrismML. Para el build upstream usar siempre el fichero `Q2_0`.
- **Memoria y contexto (medido localmente, VRAM libre según nvidia-smi)**: con KV Q8 a 65.536 tokens el modelo carga con ~810 MB libres y recupera 3 de 3 agujas a 41,6 K; con F16 a 65.536 no carga. Con `-c 131072` y KV Q8 no carga; con KV Q4 carga, pero la recuperación a fondo no se ha verificado porque el prefill tarda demasiado.
- **Prefill a contexto largo**: a ~47 K tokens el prefill tarda más de 10 minutos. Útil para lectura puntual de documentos largos, no para chat.
- **Speculative decoding en local**: con `ngram-simple` la aceptación fue de solo 2–4 % y la velocidad bajó de 40–41 a 29–38 t/s, así que no compensa en esta configuración. El MTP oficial no se ha medido localmente.
- **Visión**: el mmproj BF16 puede tumbar `llama-server` con varias peticiones de imagen simultáneas; usar Q8_0.
- **Apple Silicon**: existe build MLX 2-bit ([Ternary-Bonsai-2-4B-mlx-2bit](https://huggingface.co/prism-ml/Ternary-Bonsai-2-4B-mlx-2bit)), que requiere mlx-vlm ≥ 0.7.8.

Sources:
- [prism-ml/Ternary-Bonsai-2-4B-gguf en Hugging Face](https://huggingface.co/prism-ml/Ternary-Bonsai-2-4B-gguf)
- [Colección Bonsai 2 Small (prism-ml)](https://huggingface.co/collections/prism-ml/bonsai-2-small)
- [PrismML-Eng/Bonsai-demo (fuente de verdad para ejecutar los modelos)](https://github.com/PrismML-Eng/Bonsai-demo)
- [Fork llama.cpp de PrismML](https://github.com/PrismML-Eng/llama.cpp)
- [Artificial Analysis: sin entrada para este modelo (consultado 2026-11-01)](https://artificialanalysis.ai/models/)
