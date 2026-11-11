---
title: Ternary-Bonsai-2-9B
---

Modelo de la familia [Bonsai 2 Small](https://huggingface.co/collections/prism-ml/bonsai-2-small) (PrismML), publicado en Hugging Face como [prism-ml/Ternary-Bonsai-2-9B-gguf](https://huggingface.co/prism-ml/Ternary-Bonsai-2-9B-gguf). Es un modelo de visión y lenguaje de ~9B con pesos ternarios {-1, 0, +1}, derivado de Qwen3.5-9B, pensado para ejecución local en una GPU de consumo que pueda alojarlo entero en VRAM.


> **Estado en ia-lab (2026-11-11):** probado; no instalado actualmente. El empaquetado conservado de la familia es el 27B PTQ1_0.

## Highlights

- **Pesos ternarios con escala por grupo**: embeddings, proyecciones de atención, MLP y LM head (separado en este tamaño) en ternario con escala FP16 por grupo; normalizaciones y el camino de estado lineal en F32.
- **Tres empaquetados del mismo modelo**: PTQ1_0 (2,00 GB), PQ2_0 (2,42 GB) y Q2_0 (2,56 GB). Solo Q2_0 corre en llama.cpp upstream; PQ2_0 y PTQ1_0 requieren el fork de PrismML.
- **Arquitectura híbrida heredada de Qwen3.5**: 24 capas de atención lineal y 8 de atención completa (3:1), lo que reduce el coste del KV cache frente a un transformer denso.
- **Visión opcional**: proyector mmproj (Q8_0 recomendado) que solo se carga cuando llega una imagen.
- **Speculative decoding opcional**: variante MTP (PQ2_0 con cabeza de predicción multi-token) y drafter DFlash2, ambos con el fork de PrismML.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | ~8,95B language model (32 bloques, hidden 4096, embedding y LM head separados) + ~0,46B torre de visión (27 bloques) |
| Capas | 32 bloques híbridos: 24 de atención lineal + 8 de atención completa (3:1), SwiGLU, RoPE, RMSNorm |
| Contexto nativo | 262.144 tokens (heredado del modelo base) |
| Contexto extendido | no documentado; en la práctica local, KV Q4 a 32.768 tokens funciona con recall 3/3 en 25 K (ver nota) |
| Modalidad | texto + imagen (`image-text-to-text`) |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | llama.cpp (fork PrismML para PQ2_0/PTQ1_0/MTP/DFlash2; upstream b11555+ solo Q2_0), MLX (mlx-vlm ≥ 0.7.8), vLLM (según la ficha) |

## Resumen esquemático de benchmarks

La ficha oficial de Hugging Face **no publica tablas de benchmarks de calidad** (ni MMLU, GPQA, LiveCodeBench ni similares), así que no hay valores del vendor que comparar. Lo que sí hay son cifras de la ficha y medidas locales de velocidad con `llama-bench` (build 11621, CUDA, `-ngl 99`, tamaño de prompt 512 y 128 tokens generados), que no son benchmarks de calidad:

| Medida | Ternary-Bonsai-2-9B (Q2_0) | Ternary-Bonsai-2-4B (Q2_0) |
|---|---|---|
| Tamaño fichero Q2_0 (ficha) | 2,56 GB | 1,21 GB |
| Prompt (pp512), medido localmente | ~98 t/s | ~186 t/s |
| Generación (tg128), medido localmente | ~28 t/s | ~52 t/s |
| Generación a 25 K de contexto, medido localmente | ~10 t/s | no medido |

### Artificial Analysis (consultado 2026-11-11)

- **Sin entrada en AA**: el slug `/models/ternary-bonsai-2-9b` sigue devolviendo 404; no hay Progress Index v1 ni Intelligence Index v5.0.1 que citar.
- **Histórico:** tampoco existía un valor v4.3.2 cuando se añadió el modelo (2026-10-16).


Lectura rápida: no hay una comparativa de calidad publicada que permita situarlo frente a otros 9B; el punto fuerte documentado es el contexto nativo de 262K y la capacidad de caber entero en VRAM con Q2_0. A contexto largo la generación cae de ~28 a ~10 t/s, y el offload parcial lo hunde a la mitad. Cualquier afirmación de calidad habrá que medirla en casa.

## Configuración recomendada

Sampling oficial de la ficha (la misma para todos los empaquetados y MLX):

| Modo | temp | top_p | top_k | min_p | presence_penalty | repetition_penalty |
|---|---|---|---|---|---|---|
| Thinking (por defecto) | 1.0 | 0.95 | 20 | 0.0 | 1.0 | 1.0 |
| No-thinking | 0.7 | 0.80 | 20 | 0.0 | 1.5 | 1.0 |

- **Thinking**: activo por defecto. Aun en prompts triviales genera 3000+ tokens de razonamiento según la ficha; en la configuración local, el 9B razona de forma muy larga (3000+ tokens en trivial), frente a los ~147 del 4B.
- **No-thinking**: enviar `"chat_template_kwargs": {"enable_thinking": false}` en cada petición.
- **Longitud de salida**: con thinking, la ficha recomienda al menos 8.192 tokens; la nota local de la configuración usada sugiere `max_tokens` ≥ 4096 como mínimo, o la respuesta sale vacía.
- **Evitar temperature=0** con thinking activo: puede quedarse atascado razonando.

```bash
# llama.cpp upstream (build >= b11555), Q2_0, configuración local verificada
./llama-server -m Ternary-Bonsai-2-9B-Q2_0.gguf -ngl 99 -fa on -c 32768 \
  -ctk q4_0 -ctv q4_0 -b 256 -ub 128 --reasoning-format deepseek \
  --temp 1.0 --top-p 0.95 --top-k 20 --min-p 0 --presence-penalty 1.5 --port 8080

# Fork PrismML con PQ2_0 + visión (según la ficha oficial)
./bin/llama-server -m Ternary-Bonsai-2-9B-PQ2_0.gguf \
  --mmproj Ternary-Bonsai-2-9B-mmproj-Q8_0.gguf -ngl 99 -fa on -c 32768 --port 8080

# Speculative decoding con MTP (fork PrismML, no medido localmente)
./bin/llama-server -m Ternary-Bonsai-2-9B-PQ2_0-MTP-Q8_0.gguf -ngl 99 -fa on \
  --spec-type draft-mtp --spec-draft-n-max 2 --spec-draft-n-min 0 --spec-draft-p-min 0
```

Notas:
- **Solo Q2_0 en build stock**: `PQ2_0` y `PTQ1_0` se rechazan como "unknown type" fuera del fork de PrismML. Para el build upstream usar siempre el fichero `Q2_0`.
- **Memoria y contexto (medido localmente, VRAM libre según nvidia-smi)**: con `-ngl 99` el modelo cabe entero en GPU. Con KV F16 a 16.384 tokens carga, pero sin margen de escritorio. KV Q8 a 32.768 no carga (el buffer de cómputo se come el ahorro). KV Q4 a 32.768 carga con ~730 MB libres y recupera 3 de 3 agujas a 25 K, a ~80 t/s de prompt y ~10 t/s de generación; es la configuración por defecto de la nota local.
- **Contexto de 64 K o más**: con `-ngl 24` (offload parcial) carga y recupera 3/3 a 46 K, pero el prefill a 47 K tarda más de 10 minutos. Útil solo para lectura puntual de un documento largo, no para chat.
- **Offload parcial**: `-ngl 24` reduce la generación a la mitad (~15 t/s frente a ~27 t/s en corto). Para velocidad, mantener todo en GPU.
- **Tool calling**: verificado en la configuración por defecto (llamada explícita a herramienta con `finish=tool_calls` y argumentos correctos en ~9 s).
- **Visión**: el mmproj BF16 puede tumbar `llama-server` con varias peticiones de imagen simultáneas; usar Q8_0.
- **Apple Silicon**: existe build MLX 2-bit ([Ternary-Bonsai-2-9B-mlx-2bit](https://huggingface.co/prism-ml/Ternary-Bonsai-2-9B-mlx-2bit)), que requiere mlx-vlm ≥ 0.7.8.

Sources:
- [prism-ml/Ternary-Bonsai-2-9B-gguf en Hugging Face](https://huggingface.co/prism-ml/Ternary-Bonsai-2-9B-gguf)
- [Colección Bonsai 2 Small (prism-ml)](https://huggingface.co/collections/prism-ml/bonsai-2-small)
- [PrismML-Eng/Bonsai-demo (fuente de verdad para ejecutar los modelos)](https://github.com/PrismML-Eng/Bonsai-demo)
- [Fork llama.cpp de PrismML](https://github.com/PrismML-Eng/llama.cpp)
- [Artificial Analysis: sin entrada para este modelo (consultado 2026-11-01)](https://artificialanalysis.ai/models/)
