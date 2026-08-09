---
title: <Nombre del modelo>
---

<Frase de contexto: familia/organización, enlace a la fuente (HF/blog), y para qué tamaño/caso está pensado (ej. "ejecución local en una GPU modesta").>

## Highlights

- <3-5 bullets con los rasgos diferenciales del modelo: arquitectura, entrenamiento, capacidades destacadas>

## Parámetros, contexto y tipo de inferencia

| | |
|---|---|
| Parámetros | <total, denso o MoE (activos/totales)> |
| Capas | <nº capas, tipo de bloques si es relevante (híbrido, MoE, etc.)> |
| Contexto nativo | <tokens> |
| Contexto extendido | <si aplica, método (YaRN, etc.)> |
| Modalidad | <texto / texto+imagen / texto+imagen+vídeo> |
| Licencia | <licencia> |
| Frameworks de inferencia | <vLLM, SGLang, Transformers, llama.cpp, etc.> |

## Resumen esquemático de benchmarks

| Benchmark | <Modelo> | <Modelo comparable 1> | <Modelo comparable 2> |
|---|---|---|---|
| <benchmark de conocimiento> | | | |
| <benchmark de razonamiento> | | | |
| <benchmark de código> | | | |
| <benchmark de instrucciones> | | | |
| <benchmark de contexto largo> | | | |

<1-2 frases de "lectura rápida": para qué destaca / en qué se queda corto frente a los comparables>

## Configuración recomendada

<Tabla o lista de sampling params por modo (thinking/no-thinking, general/código/razonamiento) si el modelo lo distingue>

- <Longitud de salida recomendada>
- <Notas de activación de modos especiales (thinking on/off, etc.)>

```bash
# Comandos de servido local (vLLM / SGLang / llama.cpp / Ollama...)
```

<Notas de troubleshooting: OOM, memoria mínima recomendada, cómo extender contexto, cuantización si aplica>

Sources:
- [<Fuente 1>](url)
- [<Fuente 2>](url)
