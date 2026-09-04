---
title: Familia de modelos Claude
---

Anthropic organiza sus modelos en niveles según capacidad, velocidad y coste. En agosto de 2026 la generación vigente es la "Claude 5": **Fable 5** (el más capaz, primer modelo de un nivel nuevo llamado "Mythos-class"), **Opus 5**, **Sonnet 5** — el modelo con el que estás leyendo esta nota — y **Haiku 4.5** (aún no reemplazado por un "Haiku 5" en el momento de escribir esto).

## Tabla resumen

| Modelo | Model ID | Contexto | Precio input /1M tokens | Precio output /1M tokens | Lanzamiento | Cutoff de conocimiento |
|---|---|---|---|---|---|---|
| Claude Fable 5 | `claude-fable-5` | 1M | $10 | $50 | 9 jun 2026 | — (no publicado con precisión) |
| Claude Opus 5 | `claude-opus-5` | 1M | $5 | $25 | 24 jul 2026 | mayo 2026 |
| Claude Sonnet 5 | `claude-sonnet-5` | 1M | $3 (promo $2 hasta 31 ago 2026) | $15 (promo $10) | 30 jun 2026 | enero 2026 |
| Claude Haiku 4.5 | `claude-haiku-4-5` | 200K | $1 | $5 | oct 2025 | feb 2025 |

Precios de la API directa de Anthropic (primera parte); en Amazon Bedrock, Google Vertex AI o Microsoft Foundry pueden variar. Los precios y modelos cambian con frecuencia — comprobar siempre la [página oficial de precios](https://www.anthropic.com/pricing) y el [listado de modelos](https://platform.claude.com/docs/en/about-claude/models/overview) para datos actualizados.

## Qué es cada uno

- **Claude Fable 5** — el modelo más capaz que Anthropic ha publicado de forma general, primero de un nivel nuevo por encima de Opus ("Mythos-class"). Especialmente por delante en tareas largas y complejas (ingeniería de software, investigación). Incorpora salvaguardas de seguridad más agresivas que el resto: algunas consultas sensibles se redirigen automáticamente a Opus 4.8 (ocurre en menos del 5% de las sesiones). Existe también **Claude Mythos 5** (`claude-mythos-5`), el mismo modelo base con las salvaguardas relajadas, disponible solo para un grupo reducido de organizaciones de ciberdefensa e infraestructura crítica.
- **Claude Opus 5** — el buque insignia "normal": máxima capacidad de razonamiento a un precio bastante menor que Fable 5. Es el que Anthropic recomienda por defecto salvo que se pida explícitamente otro.
- **Claude Sonnet 5** — el equilibrio entre capacidad y coste/velocidad; el modelo que usa por defecto [[claude/claude-code|Claude Code]] para la mayoría de tareas de programación del día a día.
- **Claude Haiku 4.5** — el más rápido y barato, con ventana de contexto menor (200K frente a 1M del resto). Ideal para tareas sencillas, de alto volumen, o como "clasificador" barato antes de escalar a un modelo mayor.

## Cuándo usar cada uno

- **Fable 5 / Opus 5**: tareas que requieren razonamiento profundo, planificación a largo plazo o trabajo agéntico complejo con muchos pasos — donde el coste del modelo es pequeño comparado con el coste de un error.
- **Sonnet 5**: la opción por defecto razonable para programar, revisar código, escribir y la mayoría de tareas cotidianas — buena relación calidad/precio.
- **Haiku 4.5**: clasificación, extracción de datos, resúmenes cortos, subagentes que hacen tareas mecánicas repetidas muchas veces, o cualquier caso donde la latencia/coste importan más que el techo de capacidad.

Un patrón habitual: usar Haiku para explorar/clasificar y escalar a Sonnet u Opus solo cuando hace falta — es exactamente lo que automatiza [[tools/llm-router|llm-router]].

## Elegir el modelo en la práctica

**En Claude Code**, con el slash command `/model` (ver [[claude/comandos|comandos]]), o fijando el modelo por defecto de un [[claude/subagentes|subagente]] en su frontmatter (`model: haiku`).

**Vía API** (ejemplo en TypeScript):

```typescript
const response = await client.messages.create({
  model: "claude-sonnet-5",
  max_tokens: 16000,
  messages: [{ role: "user", content: "Resume este documento" }],
});
```

## `effort`: controlar cuánto "piensa" un modelo

Los modelos de la generación 2026 (Opus 5, Sonnet 5, Fable 5) admiten un parámetro `output_config.effort` (`low` / `medium` / `high` / `xhigh` / `max`) que regula cuánto razonamiento interno usa el modelo antes de responder — a más esfuerzo, más calidad pero también más coste y latencia. En vez de cambiar de modelo, a veces basta con bajar el `effort` para una tarea sencilla, o subirlo para una difícil manteniendo el mismo modelo.

## Ver también

- [[claude/claude-code|Claude Code]]
- [[claude/comandos|Comandos útiles de Claude Code]] — el comando `/model`.
- [[claude/subagentes|Subagentes de Claude Code]] — cada subagente puede fijar su propio modelo.
- [[tools/llm-router|llm-router]] — automatiza justo la elección "qué modelo para cada tarea" descrita arriba.
- [[tools/codeburn|codeburn]] — para medir el coste real por modelo una vez en uso.

Sources:
- [Models overview — Claude Platform Docs](https://platform.claude.com/docs/en/about-claude/models/overview)
- [Introducing Claude Fable 5 and Claude Mythos 5 — Claude Platform Docs](https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5)
- [Claude Fable 5 and Claude Mythos 5 \ Anthropic](https://www.anthropic.com/news/claude-fable-5-mythos-5)
- [Introducing Claude Sonnet 5 \ Anthropic](https://www.anthropic.com/news/claude-sonnet-5)
- [Anthropic's Claude Opus 5 AI model rivals Fable 5 and is cheaper — CNBC](https://www.cnbc.com/2026/07/24/anthropic-claude-opus-5-ai-fable-5-cost.html)
- [Precios oficiales de Anthropic](https://www.anthropic.com/pricing)
