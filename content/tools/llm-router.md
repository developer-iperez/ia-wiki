---
title: llm-router
---

Enrutador de modelos de IA que intercepta los prompts de herramientas de codificación ([[claude/claude-code|Claude Code]], Codex, Gemini CLI, Cursor, VS Code) y los dirige automáticamente al modelo más barato capaz de resolver cada tarea, con fallback entre proveedores si uno falla. Repo: [ypollak2/llm-router](https://github.com/ypollak2/llm-router).

## Propósito

A diferencia de [[tools/rtk|rtk]] o [[tools/headroom|headroom]] (que comprimen lo que se envía al modelo), `llm-router` ataca el problema desde otro ángulo: **elige un modelo más barato o local cuando la tarea no necesita el modelo premium**, reservando la cuota de Claude/GPT-4o para lo que de verdad la requiere. Reclaman un ahorro del **35-80% en prompts rutinarios**.

## Cómo enruta

Tres fases:

1. **Clasificación de complejidad**: heurísticos por regex (gratis, ~70% de acierto) o un modelo barato (Ollama local / Gemini Flash, ~$0.0001) para los casos ambiguos.
2. **Encadenamiento de modelos** en tres capas: **free** (Ollama local) → **budget** (Codex, Gemini Flash) → **premium** (Claude, GPT-4o) — prueba primero el más barato y solo escala si hace falta.
3. **Guardas en paralelo**: circuit breakers, control de presupuesto y control de calidad de la respuesta.

## Instalación e integración

```bash
pip install llm-routing
llm-router install                    # Claude Code (por defecto)
llm-router install --host codex       # Codex CLI
llm-router install --host gemini-cli  # Gemini CLI
llm-router install --host vscode      # VS Code
llm-router install --host cursor      # Cursor
```

Se integra vía hooks que interceptan las llamadas nativas sin cambiar el flujo de trabajo habitual; para VS Code/Cursor también expone herramientas MCP manuales (60 funciones: routing, texto, media, orquestación, monitorización).

## Proveedores soportados

- **Texto**: Ollama (gratis, local), OpenAI, Google Gemini, Anthropic, xAI, DeepSeek, OpenRouter (343 modelos), Groq y más.
- **Multimedia**: fal (imagen/vídeo), ElevenLabs (audio), Stability, Runway.

## Coste y configuración

**No requiere ninguna clave de API** si solo se usa con una suscripción Claude Pro/Max (usa esa cuenta directamente). Las claves son opcionales, para sumar más proveedores:

```bash
export OPENAI_API_KEY="sk-..."
export GEMINI_API_KEY="AIza..."
export OLLAMA_BASE_URL="http://localhost:11434"
export OPENROUTER_API_KEY="sk-or-v1-…"
```

Configuración persistente en `~/.llm-router/config.yaml`.

## Notas técnicas

- **Modo `zero_claude`**: bloquea el consumo de la cuota nativa de Claude Code por defecto, exigiendo el prefijo `claude:` cuando de verdad se quiere usar Claude a propósito — útil para evitar gastos accidentales de la cuota premium.
- **Política `cost_aggressive`**: acceso a los 343 modelos de OpenRouter con ahorros reclamados del 70-85%.
- Ejecución local-first, sin proxy centralizado — los datos no se envían a un servidor intermedio del propio proyecto.
- Incluye auditoría offline para detectar decisiones de enrutamiento subóptimas.
- Rankeado #8 en RouterArena (evaluador independiente de routers de modelos).

## Ver también

- [[tools/rtk|rtk]] y [[tools/headroom|headroom]] — comprimen el contenido que se envía al modelo; `llm-router` en cambio decide *qué modelo* lo recibe. Combinables entre sí.
- [[tools/codeburn|codeburn]] — para medir si el enrutamiento está reduciendo de verdad el coste por sesión/proyecto.
- [[tools/caveman|caveman]] — mencionado en su propia documentación como parte de una pila de compresión (RTK+Caveman) usada por proyectos derivados como OmniRoute.
- [[claude/modelos-claude|Familia de modelos Claude]] — contexto de qué modelo Claude usar en cada caso, la misma decisión que `llm-router` automatiza también hacia proveedores no-Anthropic.

Sources:
- [ypollak2/llm-router en GitHub](https://github.com/ypollak2/llm-router)
