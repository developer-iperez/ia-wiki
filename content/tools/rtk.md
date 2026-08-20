---
title: rtk (Rust Token Killer)
---

Proxy CLI en Rust que se sitúa entre un agente de IA (Claude Code, Copilot, Gemini CLI, Cursor...) y la terminal, comprimiendo la salida de los comandos antes de que llegue al contexto del modelo. Repo: [rtk-ai/rtk](https://github.com/rtk-ai/rtk).

## Propósito

Los agentes de codificación gastan una parte importante de su contexto leyendo salidas de `bash` (listados, logs de tests, diffs, `git status`...), muchas veces con ruido irrelevante. `rtk` reescribe de forma transparente esos comandos por versiones equivalentes pero comprimidas.

## Optimización obtenida

Hasta un **90% menos de tokens** en la salida de los comandos de `bash` que lee el agente, mediante:

- **Filtrado**: elimina ruido, comentarios y espacios innecesarios.
- **Agrupación**: agrupa archivos por directorio, errores por tipo.
- **Truncado**: conserva el contexto relevante y descarta redundancias.
- **Deduplicación**: colapsa líneas repetidas mostrando el conteo.

Importante: el ahorro se mide sobre el output de bash, no sobre la factura total de tokens de la sesión (bash es solo una de las fuentes de contexto).

## Instalación

```bash
brew install rtk                                                 # macOS/Linux (Homebrew)
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh   # script rápido
cargo install --git https://github.com/rtk-ai/rtk                # vía Cargo
```

También hay binarios precompilados en la sección Releases del repo (macOS, Linux, Windows).

Verificación e inicialización:

```bash
rtk --version
rtk init -g          # configura el hook para Claude Code / Copilot (por defecto)
```

## Comandos destacados

- `rtk ls`, `rtk read <archivo>`, `rtk grep "patrón" .` — versión compacta de los comandos de archivos.
- `rtk git status` / `rtk git log` / `rtk git diff` — versión condensada de git.
- `rtk cargo test` / `rtk pytest` / `rtk go test` — solo tests fallidos.
- `rtk gain` — dashboard con el ahorro de tokens conseguido.
- `rtk discover` — analiza el historial de la sesión para detectar oportunidades de optimización no aprovechadas.

## Notas técnicas

- Binario único en Rust, sin dependencias externas (en Windows necesita `ripgrep` para algunos filtros).
- Compatible nativamente con Windows desde la v0.37.2 (no requiere shell Unix).
- Licencia Apache 2.0.

## Ver también

- [[claude/claude-code|Claude Code]] — uno de los agentes con los que se integra vía hook.
- [[claude/hooks|Hooks de Claude Code]] — el mecanismo concreto que usa `rtk init -g` para reescribir los comandos de forma transparente.
- [[claude/comandos|Comandos útiles de Claude Code]] — comandos como `/cost` para medir consumo; `rtk` lo reduce de forma proactiva.
- [[tools/claude-mem|claude-mem]] — ahorro de tokens complementario, pero entre sesiones en vez de en la salida de comandos.
- [[tools/headroom|headroom]] — enfoque más amplio: comprime también JSON, ficheros y RAG, no solo la salida de bash.
- [[tools/codeburn|codeburn]] — complementario: no comprime, mide el consumo y detecta dónde se desperdician tokens.

Sources:
- [rtk-ai/rtk en GitHub](https://github.com/rtk-ai/rtk)
