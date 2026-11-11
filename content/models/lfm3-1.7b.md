---
title: LFM3-1.7B
---

Modelo de texto de [Liquid AI](https://huggingface.co/LiquidAI) de la familia [LFM3](https://huggingface.co/collections/LiquidAI/lfm3), publicado en Hugging Face como [LiquidAI/LFM3-1.7B](https://huggingface.co/LiquidAI/LFM3-1.7B). Está pensado para agentes locales, tool calling y salidas estructuradas en dispositivos o GPU modesta, y se basa en su nueva arquitectura tri-híbrida (convoluciones cortas, atención con ventana deslizante y atención global).


> **Estado en ia-lab (2026-11-11):** instalado y recomendado como agente ligero ultra-rápido (1 GB, 131K, tools).

## Highlights

- **Arquitectura tri-híbrida**: 24 capas con 13 convoluciones cortas con puertas, 5 de atención con ventana deslizante y 6 de atención global. Mezcla pensada para bajo coste de memoria en contexto largo.
- **Post-entreno para agentes**: SFT, RL mixto, destilación on-policy multi-dominio (MOPD) desde seis profesores RL especializados (código, mates, tool use, contexto largo, agentes y flujos empresariales) y RL agéntico final. El vendor lo orienta a harnesses como Hermes Agent, Pi u OpenClaw.
- **Edge**: en 4 bits ocupa unos 1 GB de memoria según Liquid AI, y las pruebas de edge del vendor (32K de contexto) lo sitúan con el menor consumo de memoria frente a los modelos comparados.
- **Especialización rápida**: recetas de SFT, DPO, GRPO y CPT con TRL en un solo GPU (cookbook de Liquid AI).
- **Formatos amplios**: BF16, FP8, NVFP4, GGUF, MLX 4 bits y ONNX. Vocabulario de 128K tokens y 16 idiomas declarados.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | 1,73B denso (checkpoint post-entrenado, solo texto) |
| Capas | 24: 13 convoluciones cortas con puertas, 5 de atención con ventana deslizante, 6 de atención global |
| Contexto nativo | 256K tokens (entrada + salida) según la ficha oficial |
| Contexto extendido | no aplica (el 256K es el contexto de la ficha; ver nota de uso práctico abajo) |
| Modalidad | solo texto; 16 idiomas declarados (incluye español, EN, ZH, FR, DE, IT, JA, KO, PT, AR, VI, TH, ID, HI, RU, PL) |
| Licencia | LFM Open License v1.0 (licencia propia de Liquid AI, no Apache 2.0) |
| Frameworks de inferencia | vLLM, SGLang, Transformers (`trust_remote_code=True`), llama.cpp / Ollama / LM Studio (GGUF), MLX, ONNX Runtime, Docker Model Runner |

Nota de uso práctico (datos locales, no del vendor): en la prueba local con llama.cpp y GGUF Q4_K_M, el recall verificado fue 3/3 a 100K con `-c 131072`; el 256K nativo no se ha probado.

## Resumen esquemático de benchmarks

Comparativa según la ficha oficial de Hugging Face (valores del vendor, reproducidos por Liquid AI; no verificados de forma independiente). Escala 0–100:

| Benchmark | LFM3-1.7B | MiniCPM5-2B | LFM2.5-2.6B | Qwen3.5-4B |
|---|---|---|---|---|
| GPQA Diamond (conocimiento científico) | 63.37 | 65.15 | 57.78 | 75.76 |
| LiveCodeBench v6 (código) | 73.46 | 70.52 | 60.57 | 61.99 |
| IFEval (instrucciones) | 96.66 | 87.83 | 95.39 | 92.52 |
| BFCL V4 (tool use) | 71.39 | 62.65 | 55.89 | 55.12 |
| RULER 128K (contexto largo) | 65.56 | 37.96 | 1.15 | 84.63 |
| SWE-bench Verified (OpenCode, agente código) | 43.40 | 34.80 | 18.00 | 41.20 |

Lectura rápida: según el vendor destaca en instrucciones, tool use y código para su tamaño, y en agentes (LiveClaw 45.60, por encima de Qwen3.5 9B con 35.77). Queda por detrás de Qwen3.5-4B en conocimiento científico (GPQA) y en RULER 128K, y en razonamiento difícil (HLE sin herramientas 7.17) está en el rango bajo. El vendor no recomienda el modelo para programación agéntica compleja ni para razonamiento multi-paso largo por su tamaño.

### Artificial Analysis: última versión (consultado 2026-11-11)

- **Sin entrada en AA**: no hay página para este modelo en la comparativa de modelos open source; no hay Progress Index v1 ni Intelligence Index v5.0.1 que citar.
- **Histórico:** tampoco existía un valor v4.3.2 para este modelo en el wiki.

### Histórico: Artificial Analysis v4.3.2 (consulta original)

Artificial Analysis **no tiene entrada** para LFM3-1.7B: el slug `lfm3-1-7b` devuelve 404 en la web actual y en las copias de Wayback (15 oct, 1 nov y 1 oct 2026). No hay Intelligence Index que citar en ninguna versión.

## Configuración recomendada

- **Sampling general** (chat, tool use, agentes, código): `temperature=0.1, top_k=50, repetition_penalty=1.1`.
- **Sampling matemático/STEM**: `temperature=1.0, top_p=0.95, top_k=20`.
- **Longitud de salida**: el modelo piensa por defecto (bloque `<think>` en inglés, a menudo 100-700 tokens incluso en preguntas triviales). Con `max_tokens` de 512 la respuesta puede salir vacía; usar 4096 o más. La plantilla acepta `"enable_thinking": false` por API (verificado localmente).
- **Modo de razonamiento**: la ficha no documenta un flag oficial de activación/desactivación más allá de la plantilla de chat; los detalles de tool calling están en la plantilla del repositorio.

```bash
# vLLM
vllm serve LiquidAI/LFM3-1.7B --port 8000

# SGLang
python3 -m sglang.launch_server --model-path LiquidAI/LFM3-1.7B \
  --host 0.0.0.0 --port 30000

# llama.cpp (GGUF; requiere build con soporte de arquitectura lfm3, ≥b11960 según pruebas locales)
llama-server -m LFM3-1.7B-Q4_K_M.gguf -ngl 99 \
  --temp 0.1 --top-k 50 --repeat-penalty 1.1 -c 131072

# Transformers (requiere trust_remote_code=True)
pip install -U transformers accelerate
```

Notas: para GGUF usar el repositorio oficial [LiquidAI/LFM3-1.7B-GGUF](https://huggingface.co/LiquidAI/LFM3-1.7B-GGUF) (Q4_K_M y Q8_0 probados localmente; Q4_K_M con +23 % de velocidad de decodificación frente a Q8_0 y paridad de calidad en las sondas locales). Para Apple Silicon, [LFM3-1.7B-MLX-4bit](https://huggingface.co/LiquidAI/LFM3-1.7B-MLX-4bit). Para GPU hay también variantes [FP8](https://huggingface.co/LiquidAI/LFM3-1.7B-FP8) y [NVFP4](https://huggingface.co/LiquidAI/LFM3-1.7B-NVFP4) para SGLang. Troubleshooting: si la salida llega vacía o cortada con `finish=length`, subir `max_tokens`. Con `-c 65536` el Q4 ocupa unos 2,5 GB de VRAM y con 131072 unos 3,3 GB (medido en RTX 5060).

Sources:
- [LiquidAI/LFM3-1.7B en Hugging Face](https://huggingface.co/LiquidAI/LFM3-1.7B)
- [LiquidAI/LFM3-1.7B-GGUF en Hugging Face](https://huggingface.co/LiquidAI/LFM3-1.7B-GGUF)
- [Blog de lanzamiento LFM3 (Liquid AI)](https://www.liquid.ai/blog/lfm3)
- [Colección LFM3 en Hugging Face](https://huggingface.co/collections/LiquidAI/lfm3)
