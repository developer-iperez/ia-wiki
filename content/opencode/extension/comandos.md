---
title: Comandos personalizados en OpenCode
---

Los comandos convierten una plantilla de prompt en un slash command reutilizable. Son el equivalente a los comandos personalizados de Claude Code, pero en OpenCode se definen como archivos Markdown o entradas de configuración y pueden fijar agente, modelo y ejecución en segundo plano.

## Crear un comando

```md title=".opencode/commands/review.md"
Review $ARGUMENTS for bugs and missing tests.
```

En la TUI:

```text
/review src/auth.ts
```

OpenCode envía `Review src/auth.ts for bugs and missing tests.` como prompt de usuario.

## Ubicaciones

| Ámbito | Ruta |
|---|---|
| Global | `~/.config/opencode/commands/` |
| Proyecto | `.opencode/commands/` |

Solo se descubren archivos `.md`. Las rutas anidadas forman el nombre con `/`:

```md title=".opencode/commands/team/review.md"
Review $ARGUMENTS using the team's checklist.
```

Se ejecuta como `/team/review src/auth.ts`. El directorio singular antiguo `command/` también se descubre, pero para archivos nuevos usa `commands/`.

## Frontmatter

```md title=".opencode/commands/review.md"
---
description: Revisa código en busca de errores
agent: plan
model: anthropic/claude-sonnet-5-5#high
---

Review $ARGUMENTS. Report bugs first.
```

| Campo | Obligatorio | Comportamiento |
|---|---|---|
| `template` | Solo en JSON | Plantilla del prompt. En Markdown lo aporta el cuerpo. |
| `description` | No | Texto que se muestra en listas y descubrimiento. |
| `agent` | No | Agente que se selecciona al ejecutar el comando. |
| `model` | No | Override de modelo en formato `proveedor/modelo#variante`. |
| `subagent` | No | `true` ejecuta en una sesión hija en segundo plano; `false` fuerza la sesión actual. |
| `subtask` | No | Alias obsoleto de `subagent`. |

No pongas `template` en el frontmatter de Markdown; el cuerpo es la plantilla.

## Argumentos

- `$ARGUMENTS` se sustituye por la cadena completa de argumentos tal como se escribió.
- `$1`, `$2`, etc. son argumentos posicionales; las comillas simples o dobles agrupan palabras y se eliminan.
- El placeholder de mayor número consume su argumento y todo lo que venga después.
- Los huecos que faltan se sustituyen por cadenas vacías.
- Si no hay `$ARGUMENTS` ni posicionales, OpenCode añade los argumentos no vacíos después de una línea en blanco.

```md title=".opencode/commands/check.md"
Check $1. Focus on $2.
```

`/check src/auth.ts "error handling"` → `Check src/auth.ts. Focus on error handling.`

## Bloques de shell

Envuelve un comando en `!` seguido de backticks para insertar su salida antes de enviar el prompt:

```md title=".opencode/commands/review-diff.md"
Review this diff:

!`git diff --stat && git diff`
```

OpenCode ejecuta cada bloque con el shell configurado en el directorio activo e inserta la salida combinada. Los placeholders se expanden antes.

> **Cuidado:** los bloques de shell se ejecutan al evaluar el comando, fuera del flujo de permisos de herramientas del agente. No pongas argumentos no confiables dentro de un bloque de shell.

## Adjuntos

Las plantillas guardadas no expanden `@path`: queda como texto normal. Para adjuntar un archivo, añádelo desde el composer al invocar el comando; OpenCode conserva esos adjuntos al enviar el prompt expandido.

## Ejecución y modelo

Los comandos expanden argumentos y bloques de shell antes de enviar un prompt de usuario duradero. Por defecto se ejecutan en la sesión actual.

Prioridad de modelo:

1. `model` del comando.
2. Modelo configurado del agente seleccionado.
3. Modelo activo de la sesión.

`agent` cambia el agente de la sesión actual antes de enviar el prompt, salvo que uses `subagent: true`.

## Ejecución en segundo plano

```md title=".opencode/commands/audit.md"
---
description: Audita cambios
agent: general
subagent: true
---

Audit $ARGUMENTS for bugs and missing tests.
```

| Valor | Comportamiento |
|---|---|
| `true` | Siempre usa una sesión hija, aunque el agente sea `primary`. |
| `false` | Siempre usa la sesión actual, aunque el agente sea `subagent`. |
| Omitido | Usa una sesión hija solo si el agente tiene `mode: subagent`. |

El padre sigue disponible y conserva su agente y modelo. Cuando el hijo termina, OpenCode le devuelve el resultado o el fallo.

## Comandos en JSON

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "commands": {
    "audit": {
      "template": "Audit $ARGUMENTS.",
      "description": "Audita un paquete",
      "agent": "general",
      "model": "anthropic/claude-sonnet-5-5#high",
      "subagent": true
    }
  }
}
```

Los comandos Markdown y JSON comparten registro. Las fuentes posteriores reemplazan a las anteriores con el mismo nombre; el proyecto pisa a global y lo cercano pisa a lo lejano. Los cambios se recargan sin reiniciar.

## Ver también

- [[opencode/extension/skills|Skills]]
- [[opencode/extension/agentes|Agentes y subagentes]]
- [[opencode/fundamentos/cli-tui|CLI y TUI]]
- [[opencode/guias/desde-claude-code|Desde Claude Code]]

Sources:
- [OpenCode Commands](https://opencode.ai/docs/commands/)
- [OpenCode Config](https://opencode.ai/docs/config/)
