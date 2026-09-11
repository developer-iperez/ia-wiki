---
title: graphify
---

Convierte cualquier codebase —código, docs, esquemas SQL, configs y PDFs— en un grafo de conocimiento consultable, con un skill `/graphify` para Claude Code, Cursor, Codex y Gemini CLI: parseo AST local y determinista, cada arista explicada, sin vector store. Repo: [Graphify-Labs/graphify](https://github.com/Graphify-Labs/graphify).

## En qué consiste

En vez de que el agente descubra el proyecto a fuerza de `grep` y lecturas archivo por archivo, `graphify` construye un índice en forma de grafo y el agente lo consulta en lugar de leer ficheros en bruto. El flujo típico es:

```
/graphify .
```

y se generan tres artefactos en `graphify-out/`:

- `graph.html` — visualización navegable del grafo (nodos, comunidades, búsqueda).
- `GRAPH_REPORT.md` — resumen: conceptos clave (god nodes), conexiones sorprendentes, preguntas sugeridas.
- `graph.json` — el grafo completo, consultable sin releer los ficheros.

A partir de ahí el agente (o el usuario en terminal) pregunta al grafo:

- `graphify query "qué conecta auth con la base de datos?"` — devuelve un subgrafo acotado a la pregunta.
- `graphify path "UserService" "DatabasePool"` — camino más corto entre dos conceptos.
- `graphify explain "RateLimiter"` — ficha de un concepto (origen, comunidad, conexiones).

Cada arista lleva etiqueta de confianza: `EXTRACTED` (explícita en el fuente) o `INFERRED` (resuelta por graphify), para distinguir lo leído de lo deducido. También extrae el "porqué": comentarios `# NOTE:` / `# WHY:` / `# HACK:`, docstrings y referencias a ADRs/RFCs se convierten en nodos vinculados al código.

Alcance de ingesta: unas 37-40 gramáticas tree-sitter para código, más manifests (`pyproject.toml`, `go.mod`, `pom.xml`...), configs MCP, docs Markdown/HTML/TXT, PDFs, imágenes, vídeo/audio y URLs de YouTube, esquemas SQL, Terraform/HCL y otros lenguajes vía extras opcionales.

## Beneficios

- **Menos lecturas a ciegas**: el agente recibe el fragmento relevante en una llamada en vez de reconstruir llamadas y dependencias con decenas de greps y reads.
- **Código 100% local y gratis**: el parseo de código es AST determinista con tree-sitter, sin LLM y sin que nada salga de la máquina; construir el grafo cuesta 0 créditos LLM.
- **Sin vectores**: no hay embeddings ni vector store que mantener; es un grafo real que se recorre (nodos, aristas, comunidades Leiden, god nodes).
- **Trazabilidad**: cada conexión está explicada y etiquetada (`EXTRACTED` / `INFERRED` / `AMBIGUOUS`).
- **Más que código**: docs, PDFs, imágenes y transcripciones caen al mismo grafo, útil para onboarding y arquitectura.
- **Benchmarks publicados**: en su propio harness (juez validado, 90,6% acuerdo) reportan recall@10 de 0,497 en LOCOMO (frente a 0,048 de mem0), 45,3% QA en LOCOMO, 76% en LongMemEval-S (empate con RAG denso) y subida de cobertura de 70,8% a 82,0% en preguntas sobre ERPNext (~1M LOC). Ver `BENCHMARKS.md` del repo.
- **Multiasistente**: 20+ plataformas (Claude Code, Cursor, Codex, Gemini CLI, Copilot, OpenCode, Aider...), con hooks o ficheros de instrucciones para que el asistente consulte primero el grafo.

## Limitaciones

- **Lo no-código necesita LLM**: docs, PDFs e imágenes requieren un pase semántico con el modelo del IDE o una API key configurada (Gemini, Claude, OpenAI, DeepSeek, Ollama local, Bedrock...). Solo el corpus de código es totalmente offline; para forzarlo existe `--code-only`.
- **Instalación fragmentada**: el paquete PyPI se llama `graphifyy` (con doble `y`) aunque el comando sea `graphify`; cada tipo de contenido extra pide un extra (`[pdf]`, `[office]`, `[video]`, `[sql]`, `[terraform]`...) y Python ≥ 3.10 más `uv` o `pipx`.
- **Mantenimiento del grafo**: si no se instalan los hooks de git (`graphify hook install`) o no se corre `graphify update .` tras `pull`/`merge`, el grafo se desfasa. Los rebuilds en segundo plano pueden tardar segundos en repos grandes.
- **Escala**: `graph.json` tiene un tope por defecto de 512 MiB (ampliable con `GRAPHIFY_MAX_GRAPH_BYTES`); repos enormes exigen presupuestarlo.
- **Proyecto en evolución**: versionado tipo `v8`, plataforma comercial en early access (`graphify.com` / `app.graphify.com`); cabe esperar cambios y fricción ocasional.
- **Matices por plataforma**: en PowerShell se usa `graphify .` (sin `/` inicial), en Codex `$graphify`; cada asistente tiene su comando de instalación y su mecanismo always-on (hooks vs `AGENTS.md`).
- **Benchmarks de parte**: las cifras vienen del harness del propio proyecto; útiles como referencia, no como verdad independiente.

## Instalación y uso

Requisitos: Python 3.10+, `uv` (recomendado) o `pipx`.

```bash
# 1. Instalar el CLI (el paquete es graphifyy, con doble y)
uv tool install graphifyy
# Alternativas: pipx install graphifyy | pip install graphifyy

# 2. Registrar el skill en el asistente
graphify install
# Por plataforma: graphify install --platform codex|opencode|gemini|cursor|...
# Solo para el repo actual: graphify install --project

# 3. Construir el grafo (dentro del asistente: /graphify .)
graphify extract .
```

Comandos habituales:

```bash
/graphify .                       # construir grafo de la carpeta actual
/graphify ./docs --update         # re-extraer solo lo cambiado
/graphify . --cluster-only        # recalcular comunidades sin re-extraer
/graphify . --no-viz              # sin HTML, solo informe + JSON
/graphify . --wiki                # generar wiki Markdown desde el grafo
graphify query "qué conecta auth con la db?"
graphify path "UserService" "DatabasePool"
graphify explain "RateLimiter"
graphify add https://arxiv.org/abs/1706.03762  # añadir paper / vídeo al grafo
graphify hook install             # rebuild automático en commit y checkout
graphify update .                 # sincronizar tras git pull / merge
graphify merge-graphs a.json b.json  # combinar grafos
graphify prs                      # dashboard de PRs con impacto en el grafo
graphify uninstall [--purge]      # quitar la integración
```

Para que el asistente use siempre el grafo:

```bash
graphify claude install     # Claude Code (hooks + instrucciones)
graphify codex install      # Codex (vía AGENTS.md)
graphify opencode install   # OpenCode
graphify cursor install     # Cursor (.cursor/rules con alwaysApply)
graphify gemini install     # Gemini CLI
```

Extras por tipo de contenido (instalar solo lo necesario):

```bash
uv tool install "graphifyy[pdf]"    # PDFs
uv tool install "graphifyy[office]" # .docx y .xlsx
uv tool install "graphifyy[video]"  # transcripción vídeo/audio
uv tool install "graphifyy[sql]"    # esquemas SQL
uv tool install "graphifyy[mcp]"    # servidor MCP stdio
uv tool install "graphifyy[all]"    # todo
```

Servir el grafo por MCP/HTTP para el equipo:

```bash
python -m graphify.serve graphify-out/graph.json
python -m graphify.serve graphify-out/graph.json --transport http --port 8080
```

Se recomienda commitear `graphify-out/` (salvo `cost.json` y opcionalmente `cache/`) y usar `.graphifyignore` (sintaxis tipo `.gitignore`, se fusiona con este) para excluir generados.

## Ver también

- [[tools/codegraph|codegraph]] — alternativa centrada solo en código (Rust + SQLite, watcher con auto-sync); graphify cubre además docs, PDFs, SQL y multimedia, con informe y visor HTML.
- [[tools/claude-mem|claude-mem]] y [[tools/agentmemory|agentmemory]] — memoria entre sesiones; graphify es conocimiento *del proyecto*, no de la conversación.
- [[tools/headroom|headroom]] y [[tools/rtk|rtk (Rust Token Killer)]] — comprimen lo que se envía al modelo; graphify evita que haya que leerlo a ciegas. Combinables.
- [[tools/codeburn|codeburn]] — para medir si el grafo reduce de verdad el coste por sesión/proyecto.
- [[claude/mcp|MCP (Model Context Protocol)]] — mecanismo por el que se expone el grafo a los agentes.

Sources:
- [Graphify-Labs/graphify en GitHub](https://github.com/Graphify-Labs/graphify)
- [BENCHMARKS.md del proyecto](https://github.com/Graphify-Labs/graphify/blob/v8/BENCHMARKS.md)
- [graphify.com](https://graphify.com) (plataforma, early access en [app.graphify.com](https://app.graphify.com/login))
