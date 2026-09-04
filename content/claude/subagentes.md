---
title: Subagentes de Claude Code
---

Un subagente es una instancia aislada de [[claude/claude-code|Claude Code]] que la sesión principal lanza para encargarse de una tarea concreta: tiene su propia ventana de contexto, sus propios permisos de herramientas y, opcionalmente, su propio modelo. El agente principal solo ve el resumen final que devuelve el subagente, nunca sus pasos intermedios — así la conversación principal no se llena de logs de búsquedas, contenido de ficheros leídos, etc.

## Por qué usarlos

- **Preservan contexto**: una exploración larga (buscar en el código, leer muchos ficheros) se queda "encapsulada" en el subagente; el hilo principal solo recibe la conclusión.
- **Restringen herramientas**: se le puede dar a un subagente acceso de solo lectura, o solo a un subconjunto de comandos, aunque la sesión principal tenga más permisos.
- **Controlan coste**: un subagente puede usar un modelo más barato (p. ej. Haiku) para tareas mecánicas, mientras la sesión principal sigue en un modelo más potente.
- **Se pueden paralelizar**: pedir "analiza seguridad y rendimiento de este proyecto" puede lanzar dos subagentes a la vez, cada uno con su propio contexto.
- **Reutilizables**: un mismo fichero de subagente sirve en cualquier proyecto donde se copie.

## Formato de fichero

Un subagente es un Markdown con frontmatter YAML, muy similar a una [[claude/skills|skill]]:

```markdown
---
name: security-auditor
description: Security expert that audits code for vulnerabilities and best practices. Use after implementing authentication or handling sensitive data.
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a security auditor specializing in vulnerability detection.

When reviewing code:
1. Search for common vulnerabilities (SQL injection, XSS, CSRF)
2. Check for exposed credentials or API keys
3. Verify authentication/authorization implementation

For each finding: explica el riesgo y su severidad, muestra el código
vulnerable y propone una versión corregida.
```

### Campos del frontmatter

| Campo | Obligatorio | Descripción |
|---|---|---|
| `name` | Sí | Identificador único (minúsculas, guiones). |
| `description` | Sí | Cuándo debe delegarse en este subagente — Claude la usa para decidir si lo invoca automáticamente. |
| `tools` | No | Lista de herramientas permitidas (allowlist). Si se omite, hereda todas. |
| `disallowedTools` | No | Denylist: todas las herramientas excepto las listadas. |
| `model` | No | `sonnet`, `opus`, `haiku`, o `inherit` (el de la sesión principal, por defecto). |
| `permissionMode` | No | `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, `plan`. |
| `skills` | No | Skills a precargar en el contexto del subagente. |
| `memory` | No | `user`, `project` o `local`, para persistencia entre sesiones. |
| `maxTurns` | No | Máximo de turnos antes de detenerse. |
| `isolation` | No | `worktree` para aislarlo en un git worktree propio. |

## Dónde se guardan

Por prioridad (de mayor a menor):

1. Managed settings de la organización.
2. Flag `--agents` en la propia CLI (solo esa sesión).
3. `.claude/agents/` — del proyecto actual (versionar en git para compartir con el equipo).
4. `~/.claude/agents/` — personal, disponible en todos tus proyectos.
5. Directorio `agents/` de un plugin.

## Subagentes integrados

Claude Code trae tres por defecto, sin necesidad de crear nada:

| Agente | Herramientas | Uso |
|---|---|---|
| **Explore** | solo lectura | búsqueda y análisis rápido del código |
| **Plan** | solo lectura | investigación previa en modo planificación |
| **General-purpose** | todas | tareas complejas de varios pasos |

## Cómo crear uno

**Pidiéndoselo a Claude** (la forma más simple):

```text
Create a personal code-improver subagent in ~/.claude/agents/ that scans
files and suggests improvements. It should be read-only and use Sonnet.
```

**A mano**: crear el `.md` con frontmatter en `.claude/agents/` o `~/.claude/agents/` — Claude Code detecta los cambios automáticamente, sin reiniciar sesión.

**Solo para una sesión**, vía CLI:

```bash
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer",
    "prompt": "You are a senior code reviewer...",
    "tools": ["Read", "Grep", "Glob"],
    "model": "sonnet"
  }
}'
```

## Cómo se invocan

- **Delegación automática**: Claude decide solo, en base a la `description`, cuándo conviene usar un subagente. Basta con pedir la tarea en lenguaje natural: *"Have the code-reviewer look at my recent changes"*.
- **Mención explícita**: para forzar un subagente concreto, `@"security-auditor (agent)" review the authentication changes`.
- **Sesión entera como un subagente**: `claude --agent code-reviewer`, o fijándolo en `.claude/settings.json` con `"agent": "code-reviewer"`.

## Restringir qué subagentes puede invocar otro

Un subagente puede a su vez invocar otros (hasta 3 niveles de profundidad por defecto). Para limitarlo a unos concretos:

```yaml
tools: Agent(worker, researcher), Read, Bash
```

## Subagentes vs Skills

Ambos son ficheros Markdown con frontmatter que Claude activa según una `description`, pero resuelven cosas distintas:

- Una [[claude/skills|skill]] añade **conocimiento/procedimiento** al contexto de la sesión actual — sigue siendo la misma conversación.
- Un **subagente** abre una **sesión aislada aparte**, con su propio contexto y (opcionalmente) su propio modelo, y solo devuelve el resultado final.

Un subagente puede además precargar skills concretas (campo `skills` del frontmatter).

## Buenas prácticas

- Descripciones claras y específicas — es lo único que usa Claude para decidir cuándo delegar.
- Dar solo las herramientas necesarias (allowlist), tanto por seguridad como para que el subagente se mantenga enfocado.
- Guardar en `.claude/agents/` (versionado) los que use todo el equipo, y en `~/.claude/agents/` los personales que quieras reutilizar entre proyectos.

## Ver también

- [[claude/claude-code|Claude Code]]
- [[claude/comandos|Comandos útiles de Claude Code]] — incluye `/agents`, el comando para gestionarlos dentro de una sesión.
- [[claude/skills|Skills de Claude]]
- [[claude/modelos-claude|Familia de modelos Claude]] — para elegir bien el campo `model` de cada subagente.

Sources:
- [Crear subagentes personalizados — Claude Code Docs](https://code.claude.com/docs/es/sub-agents)
- [Claude Code Agents y Subagents: Como Crear Agentes IA Autonomos [Tutorial 2026] | Javadex](https://www.javadex.es/blog/claude-code-agents-subagents-crear-agentes-ia-tutorial-2026)
- [Claude Code subagents: desarrollo en paralelo (guía 2026) — IAcademy](https://iacedemy.com/blog/claude-code-subagents/)
- [Subagentes en el SDK — Claude API Docs](https://platform.claude.com/docs/es/agent-sdk/subagents)
