---
title: MiniCPM-V-4.7-1B
---

Variante de 1B de la familia [MiniCPM-V 4.7](https://huggingface.co/openbmb/MiniCPM-V-4.7-1B) (OpenBMB), modelo multimodal de imagen, multi-imagen y vídeo publicado en Hugging Face como [openbmb/MiniCPM-V-4.7-1B](https://huggingface.co/openbmb/MiniCPM-V-4.7-1B). Pensado para visión local en dispositivos o GPU modesta. La otra variante de la serie es el 35B-A3B MoE.

## Highlights

- **Visión + texto en un 1B**: comprensión de imagen única, multi-imagen y vídeo. El vendor lo sitúa por encima de Qwen3.5-0.8B en puntuación global y eficiencia de inferencia.
- **Compresión visual mixta 4x/16x**: el encoder comprime tokens visuales temprano (intra-ViT, basado en LLaVA-UHD v4), y el usuario elige 4x o 16x según el equilibrio detalle/coste.
- **Despliegue en edge**: el vendor cita iOS, Android, HarmonyOS y soporte day-0 en chips de Rockchip, Houmo, Intel, AMD y Arm.
- **Formatos listos**: BF16, int4 (bitsandbytes), AWQ, GPTQ, GGUF y FP8, además de variantes DSpark para speculative decoding. Los GGUF incluyen una subcarpeta `thinking/` con el template original.
- **Fine-tuning con SRFT**: el vendor publica Self-Replay Fine-Tuning para extender capacidades reduciendo el olvido.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | ~1.3B según la tabla oficial de evaluación (la metadata de Hugging Face muestra 2B params; no está desglosado en la ficha) |
| Capas | no documentado en la ficha. En el GGUF local (bloque texto `qwen35`): 24 capas, GQA 8 Q / 2 KV, head_dim 256 |
| Contexto nativo | no documentado en la ficha. El GGUF local reporta 262.144 tokens |
| Contexto extendido | no documentado |
| Modalidad | texto + imagen + vídeo (imagen única, multi-imagen y vídeo según la ficha) |
| Licencia | Apache 2.0 |
| Frameworks de inferencia | SGLang, vLLM, llama.cpp (GGUF), Ollama, Transformers (`AutoModelForMultimodalLM`), OpenVINO |

## Resumen esquemático de benchmarks

Comparativa según la tabla oficial de la model card, modo Instruct, con compresión visual 4x (valores del vendor, no reproducidos de forma independiente). Escala 0–100:

| Benchmark | MiniCPM-V-4.7-1B | Qwen3.5-0.8B | LFM2.5-VL-1.6B | Gemma-4-E2B (5.1B) |
|---|---|---|---|---|
| MMBench (general) | 83.17 | 68.00 | 65.45 | 66.70 |
| MMStar (general) | 69.53 | 55.90 | 50.70 | 55.70 |
| MMMU (razonamiento visual) | 56.89 | 47.40 | 40.60 | 59.60 |
| MathVista (mates visuales) | 74.30 | 58.60 | 55.70 | 58.50 |
| OCRBench (OCR) | 86.10 | 79.10 | 79.70 | 73.50 |
| Hallusion-Bench (alucinación) | 57.85 | 46.70 | 42.00 | 44.10 |

Lectura rápida: gana a los comparables de su tamaño en casi todo lo general, OCR y alucinación; en MMMU queda por detrás de Gemma-4-E2B, que tiene 5.1B. En la tabla oficial también aparece MiniCPM-V-4.6 (1.3B) con MathVista 75.50 y OmniDocBench 86.93, por encima del 4.7 en esos dos benchmarks.

### Artificial Analysis

- **Sin entrada**: consulté `artificialanalysis.ai/models/minicpm-v-4-7-1b` y `minicpm-v-4-7`, ambos 404. No hay página de AA para MiniCPM-V-4.7-1B; ia-lab también lo marca como "sin página en AA" (`MODELS.md`).

## Configuración recomendada

- **Sampling**: el cookbook de llama.cpp de OpenBMB recomienda `temperature=0.7, top_p=0.8, top_k=100, repeat_penalty=1.05`, usado también en el menú local de ia-lab. La ficha de Hugging Face no documenta sampling.
- **Modo thinking**: el template activa thinking por defecto. Los GGUF del repositorio raíz lo dejan desactivado; la subcarpeta `thinking/` restaura el template original con trazas `[Start thinking] … [End thinking]`.
- **Longitud de salida**: pruebas cortas con 256 tokens; subir si hay thinking.
- **Tools**: el menú local de ia-lab sirve el perfil de visión sin tools, porque sin herramientas el thinking puede entrar en bucles ante datos en vivo. Con tools explícitos vía API, `finish_reason=tool_calls` funciona con y sin mmproj.

```bash
# vLLM
vllm serve openbmb/MiniCPM-V-4.7-1B --port 8000

# SGLang
python3 -m sglang.launch_server --model-path openbmb/MiniCPM-V-4.7-1B --port 30000

# llama.cpp con GGUF oficial (build >= 11621, soporte M-RoPE para visión)
llama-server -m MiniCPM-V-4_7-1B-Q4_K_M.gguf --mmproj mmproj-model-f16.gguf \
  --port 8080 -ngl 99 -c 32768 -ctk q4_0 -ctv q4_0
```

Notas: el GGUF oficial está en [openbmb/MiniCPM-V-4.7-1B-gguf](https://huggingface.co/openbmb/MiniCPM-V-4.7-1B-gguf). En ia-lab, Q4_K_M ocupa 672 MB más un mmproj f16 de 1,06 GB, que va en subcarpeta propia para no emparejarlo con otros modelos. Con mmproj en CPU, `-c 32768` con KV-Q4 cargó con ~2,5 GB libres en la GPU de referencia; `-c 65536` cargó con ~2 GB libres. El recall a profundidad no está verificado, así que los 32K son el valor por defecto de carga, no de calidad demostrada. Build de llama.cpp ≥ 11621, con soporte fusionado el 2026-10-10 (PR #29416).

Sources:
- [openbmb/MiniCPM-V-4.7-1B en Hugging Face](https://huggingface.co/openbmb/MiniCPM-V-4.7-1B)
- [openbmb/MiniCPM-V-4.7-1B-gguf](https://huggingface.co/openbmb/MiniCPM-V-4.7-1B-gguf)
- [MiniCPM-V 4.7 en GitHub (MiniCPM-o docs)](https://github.com/OpenBMB/MiniCPM-o/blob/main/docs/minicpm_v4dot7_en.md)
- [MiniCPM-V-CookBook: MiniCPM-V 4.7](https://github.com/OpenSQZ/MiniCPM-V-CookBook/blob/main/docs/pages/v4.7/overview.md)
