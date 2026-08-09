---
title: codeburn
---

Herramienta local y gratuita que rastrea el uso de tokens y coste de IA en 36 herramientas (Claude Code, Cursor, Codex, Gemini...), desglosado por modelo, proyecto y tarea. Repo: [getagentseal/codeburn](https://github.com/getagentseal/codeburn).

## Propósito

A diferencia de las herramientas de compresión (que reducen tokens sobre la marcha), codeburn da **visibilidad**: analiza sin conexión los ficheros de sesión que ya existen en la máquina para mostrar dónde se está gastando el presupuesto de IA, y detecta patrones de desperdicio para poder corregirlos.

## Optimización obtenida

No comprime nada directamente; su valor está en la detección y las sugerencias:

- **Detección de desperdicio**: archivos releídos varias veces, ratio lectura/edición bajo, salidas de bash sin acotar, servidores MCP conectados pero sin usar.
- **One-shot rate**: qué porcentaje de ediciones funcionan a la primera, sin reintentos.
- **Comparación de modelos**: tasa de éxito y coste por llamada entre los modelos que se estén usando.
- **Guard**: límites de presupuesto opcionales, con avisos y bloqueos en Claude Code.
- `codeburn optimize --apply` puede aplicar automáticamente correcciones para el desperdicio detectado (reversibles).

## Instalación

```bash
npx codeburn              # ejecución directa, sin instalar
npm install -g codeburn   # instalación global
brew install codeburn     # macOS, vía Homebrew
```

Requiere Node.js ≥ 22.13. Para Cursor y OpenCode necesita `better-sqlite3` (se instala automáticamente).

## Comandos destacados

- `codeburn` — dashboard interactivo (últimos 7 días).
- `codeburn overview` — resumen mensual legible y copiable.
- `codeburn optimize` — detecta patrones de desperdicio con sugerencias (`--apply` para aplicarlas).
- `codeburn compare` — compara rendimiento entre modelos.
- `codeburn web` — dashboard web en `localhost:4747`.
- `codeburn guard install` — configura límites de presupuesto.
- `codeburn mcp` — expone `get_usage` y `get_savings` como herramientas MCP para agentes de IA.

## Notas técnicas

- Todo en local: lee JSONL de Claude y bases SQLite de Cursor/OpenCode/Codex; no sube datos.
- Precios vía LiteLLM (actualizados a diario), con valores de respaldo para Claude y GPT.
- Categorización automática en 13 categorías (Coding, Debugging, Feature Dev, Testing...) sin llamadas a un LLM.
- Sincronización entre varios dispositivos (portátil, sobremesa, máquina de trabajo) mediante emparejamiento por PIN.
- Historial de cambios reversible (`codeburn act list`, `undo`).
- Licencia MIT.

## Ver también

- [[claude/claude-code|Claude Code]] — una de las 36 herramientas que puede analizar.
- [[tools/rtk|rtk (Rust Token Killer)]] — mientras rtk reduce tokens de forma proactiva, codeburn mide el consumo y detecta dónde se desperdician.
- [[tools/agentmemory|agentmemory]] y [[tools/claude-mem|claude-mem]] — reducen tokens entre sesiones; codeburn sirve para medir el efecto de ese tipo de optimizaciones.

Sources:
- [getagentseal/codeburn en GitHub](https://github.com/getagentseal/codeburn)
