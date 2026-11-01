---
title: NeoHorse-1-4B
---

Modelo de [TokenRhythm](https://huggingface.co/TokenRhythm), publicado en Hugging Face como [TokenRhythm/NeoHorse-1-4B](https://huggingface.co/TokenRhythm/NeoHorse-1-4B) (con variante GGUF en [TokenRhythm/NeoHorse-1-4B-GGUF](https://huggingface.co/TokenRhythm/NeoHorse-1-4B-GGUF)). Es un post-entreno agéntico de [Qwen3.5-4B](https://huggingface.co/Qwen/Qwen3.5-4B) orientado a harnesses de agentes, uso de herramientas, código y seguimiento de instrucciones, con solo pesos de texto (sin visión). Se presenta como prototipo hacia la auto-mejora recursiva; ver el [informe técnico (arXiv 2609.08183)](https://arxiv.org/abs/2609.08183).

## Highlights

- **Post-entreno agéntico sobre Qwen3.5-4B**: parte del mismo checkpoint base y arquitectura que Qwen3.5-4B, con fine-tuning orientado a trayectorias de agente, tool-use y código.
- **Enrutado en el entrenamiento**: el informe describe un harness de enrutado que asigna tareas a un pool heterogéneo de modelos, registra interacciones con herramientas y usa la demanda de capacidades para dar forma a la mezcla de entrenamiento (SFT guiado por enrutado y on-policy distillation, según el informe).
- **Curación de datos**: deduplicación exacta y casi duplicada, descontaminación frente a evaluaciones, validación estructural y etiquetado Scene/Goal/Outcome por subescena.
- **Ganancia media reportada**: 64,87 de media en diez benchmarks frente a 58,94 de Qwen3.5-4B (+5,93), según el vendor.
- **Solo texto, con formato GGUF**: los pesos de visión no se incluyen; hay GGUF BF16/F16 y cuantizaciones Q8_0, Q5_K_M y Q4_K_M para llama.cpp, Ollama y LM Studio.

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | ~4B (denso); el GGUF Q4_K_M cargado localmente reporta 4,21B. La ficha no da el número exacto |
| Capas | no documentado en la ficha; la arquitectura es la de Qwen3.5-4B (`qwen3_5_text`, `qwen35` en GGUF) |
| Contexto nativo | 262.144 tokens |
| Contexto extendido | hasta 1.010.000 tokens (la ficha no detalla el método; hereda el de Qwen3.5) |
| Modalidad | solo texto (entrada y salida); sin pesos de visión |
| Licencia | Apache 2.0 (conserva el aviso de copyright de Alibaba Cloud) |
| Frameworks de inferencia | Transformers, vLLM, SGLang, llama.cpp (GGUF), Ollama, LM Studio, Docker Model Runner |

## Resumen esquemático de benchmarks

Valores del vendor (model card de TokenRhythm), no reproducidos de forma independiente. Escala 0–100, protocolo SGLang v0.5.17 con thinking activado:

| Benchmark | NeoHorse-1-4B (vendor) | Qwen3.5-4B | Nanbeige-4.2-3B | Agents-A1-4B |
|---|---|---|---|---|
| tau2-Bench (agente / tool-use) | 88,46 | 84,29 | 85,08 | 81,00 |
| PinchBench (agente) | 77,33 | 71,19 | 66,78 | 75,07 |
| BFCL v4 (tool-use) | 61,79 | 61,02 | 67,28 | 46,60 |
| LiveCodeBench v6 (código) | 59,43 | 53,71 | 72,50 | 56,57 |
| IFEval (instrucciones) | 88,35 | 87,06 | 84,47 | 83,55 |
| Media de 10 benchmarks | 64,87 | 58,94 | 62,31 | 61,46 |

Lectura rápida: mejora frente a su base Qwen3.5-4B en casi todas las filas del vendor, con el mayor salto en agentes (VitaBench +10,50, WorkBuddy +9,79) y en tau2-Bench; en BFCL v4 la mejora es marginal (+0,77) y en código y tool-use puro queda por detrás de Nanbeige-4.2-3B. Los datos de otras filas (QwenClawBench, WorkBuddy, VitaBench, HumanEval, IFBench) están en la model card.

### Artificial Analysis (medición independiente, consultado 2026-11-01)

No hay página de NeoHorse-1-4B en Artificial Analysis: el slug `neohorse-1-4b` devuelve 404 en la web actual y en las copias de Wayback (15 oct, 1 nov y 1 oct 2026). Por tanto no hay Intelligence Index ni métricas independientes; la entrada del menú de AA queda como "—".

## Configuración recomendada

**Parámetros de muestreo** (modo thinking, según la model card):

| temperature | top_p | top_k | min_p | presence_penalty | repetition_penalty |
|---|---|---|---|---|---|
| 1.0 | 0.95 | 20 | 0.0 | 1.5 | 1.0 |

- **Thinking**: se activa con `chat_template_kwargs: {"enable_thinking": true}`. La ficha no documenta el valor por defecto de la plantilla ni un modo no-thinking explícito; en LM Studio se desactiva con `reasoning: "off"`.
- **Longitud de salida**: los ejemplos oficiales usan `max_tokens` de 512 a 2048; para tareas agénticas largas, aumentar `max_tokens` y el contexto del servidor.
- **Tool calling**: en vLLM hace falta `--enable-auto-tool-choice --tool-call-parser qwen3_coder`; en SGLang, `--tool-call-parser qwen3_coder`. Con llama.cpp, mantener `--jinja` para usar la plantilla con herramientas.

```bash
# SGLang (versión usada en el informe: 0.5.17)
python3 -m sglang.launch_server \
  --model-path TokenRhythm/NeoHorse-1-4B \
  --served-model-name neohorse-1-4b \
  --port 30000 --context-length 262144 \
  --reasoning-parser qwen3 --tool-call-parser qwen3_coder

# vLLM
vllm serve TokenRhythm/NeoHorse-1-4B \
  --served-model-name neohorse-1-4b \
  --port 8000 --max-model-len 262144 \
  --reasoning-parser qwen3 \
  --enable-auto-tool-choice --tool-call-parser qwen3_coder

# llama.cpp con GGUF (ejemplo de la ficha, BF16)
llama-server -hf TokenRhythm/NeoHorse-1-4B-GGUF:BF16 -c 4096 -ngl 99 --jinja --port 8080 --alias neohorse-1-4b

# Ollama
ollama run hf.co/TokenRhythm/NeoHorse-1-4B-GGUF:Q4_K_M
```

Notas: la ficha indica que las cuantizaciones se generaron directamente desde BF16 sin matriz de importancia y que solo pasaron comprobaciones básicas de compatibilidad (thinking, tool calls, a 4K de contexto); no hay benchmark de calidad para ellas. En verificación local (ia-lab, septiembre 2026) el Q4_K_M dio velocidades muy parecidas al Qwen3.5-4B de Unsloth (pp512 179 vs 173 t/s; tg128 32 vs 30 t/s, CUDA) y tools funcionando, pero la comparación de calidad no se llegó a hacer y el modelo fue borrado el 2026-10-11. Si el objetivo es elegir entre ambos, hay que medir calidad en la misma tarea antes de cambiar.

Sources:
- [TokenRhythm/NeoHorse-1-4B en Hugging Face](https://huggingface.co/TokenRhythm/NeoHorse-1-4B)
- [TokenRhythm/NeoHorse-1-4B-GGUF en Hugging Face](https://huggingface.co/TokenRhythm/NeoHorse-1-4B-GGUF)
- [NeoHorse-1: Towards Recursive Self-Improvement via Agentic Post-Training with Routing Harness (arXiv 2609.08183)](https://arxiv.org/abs/2609.08183)
- [Qwen/Qwen3.5-4B en Hugging Face](https://huggingface.co/Qwen/Qwen3.5-4B)
