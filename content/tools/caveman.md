---
title: caveman
---

Skill/plugin de optimización de tokens de **salida**, compatible con Claude Code, Gemini, Cursor y más de 30 agentes de codificación. Repo: [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman).

## Propósito

A diferencia de `rtk` o `headroom`, que comprimen lo que el modelo **lee** (salida de comandos, ficheros, RAG), `caveman` actúa sobre lo que el modelo **escribe**: instruye al agente para que responda en un estilo minimalista, sin prosa narrativa innecesaria, preservando la exactitud técnica en código y comandos.

## Optimización obtenida

- **~65% de reducción** en tokens de salida en respuestas tipo chat.
- **~8.5% de reducción** en ejecuciones agénticas completas de código (donde la mayor parte del gasto ya es código/herramientas, no prosa).

## Notas técnicas

- Funciona vía ingeniería de prompts (instrucciones de estilo), no como proxy ni post-procesado.
- Se instala como skill/plugin en el agente correspondiente.

## Ver también

- [[claude/claude-code|Claude Code]] — uno de los agentes con los que se integra.
- [[tools/rtk|rtk (Rust Token Killer)]] y [[tools/headroom|headroom]] — comprimen tokens de entrada; `caveman` comprime tokens de salida.

Sources:
- [JuliusBrussee/caveman en GitHub](https://github.com/JuliusBrussee/caveman)
