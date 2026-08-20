---
title: Hooks de Claude Code
---

Un hook es un comando de shell (o un endpoint HTTP, una tool MCP, o incluso un prompt evaluado por Claude) que [[claude/claude-code|Claude Code]] ejecuta automáticamente en puntos concretos de su ciclo de vida: antes de correr una herramienta, después de editar un fichero, al terminar la respuesta, etc. A diferencia de pedirle a Claude "siempre haz X después de Y", un hook es una automatización **determinista** — se ejecuta siempre, sin depender de que el modelo decida acordarse.

## Por qué usarlos

- **Seguridad**: bloquear comandos peligrosos (`rm -rf`, `git push --force`...) antes de que se ejecuten, sin depender de que Claude los evite por sí solo.
- **Calidad automática**: formatear o lintear un fichero justo después de editarlo, sin tener que pedirlo cada vez.
- **Notificaciones**: avisar (sonido, Slack, etc.) cuando Claude termina una tarea larga.
- **Integraciones**: es el mecanismo que usan herramientas como [[tools/rtk|rtk]] para interceptar y reescribir comandos (`git status` → `rtk git status`) de forma transparente.

## Eventos disponibles

Se agrupan según cuándo se disparan:

**Por sesión**
- `SessionStart` — al iniciar o reanudar una sesión.
- `SessionEnd` — al terminarla.

**Por turno**
- `UserPromptSubmit` — antes de procesar el prompt del usuario.
- `Stop` — cuando Claude termina de responder.
- `StopFailure` — cuando un turno falla por un error de la API.

**Dentro del bucle agéntico** (por cada llamada a herramienta)
- `PreToolUse` — antes de ejecutar una herramienta; puede **bloquearla**.
- `PermissionRequest` — cuando hace falta una decisión de permiso.
- `PostToolUse` — después de que una herramienta se ejecute con éxito.
- `PostToolUseFailure` — después de que falle.

**Otros**
- `FileChanged`, `CwdChanged`, `ConfigChange` — cambios de estado del entorno.
- `Notification` — cuando Claude Code envía una notificación.
- `PreCompact` / `PostCompact` — antes/después de comprimir el contexto (ver `/compact` en [[claude/comandos|comandos]]).

## Configuración

Se definen en un `settings.json`, con tres niveles: **evento** → **grupo con matcher** → **hook**.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/block-rm.sh",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### Dónde se pueden definir

- `~/.claude/settings.json` — global, para todos tus proyectos.
- `.claude/settings.json` — del proyecto, se puede versionar y compartir con el equipo.
- `.claude/settings.local.json` — del proyecto, pero local (normalmente en `.gitignore`).
- `hooks/hooks.json` de un plugin.

### El `matcher`

Filtra a qué herramientas aplica el hook:

| Patrón | Significado |
|---|---|
| `*` o vacío | coincide con todo |
| `Bash`, `Edit\|Write` | nombre exacto, o varios separados por `\|` |
| `mcp__memory__.*` | regex — útil para herramientas MCP, que se llaman `mcp__<servidor>__<tool>` |

## Entrada y salida

El hook recibe por **stdin** un JSON con datos del evento — por ejemplo, en `PreToolUse` incluye `tool_name`, `tool_input`, `session_id`, `cwd`...

La respuesta se controla con el **exit code**:

- `0` — éxito; si además escribe JSON en stdout, Claude Code lo interpreta.
- `2` — bloquea la acción (válido en `PreToolUse`, `UserPromptSubmit`, `Stop`...).
- cualquier otro — error no bloqueante, continúa el flujo.

El JSON de salida puede incluir, entre otros campos, `hookSpecificOutput.permissionDecision` (`allow` / `deny` / `escalate`) con su razón, o `additionalContext` para añadir información al contexto del modelo.

## Ejemplos prácticos

### 1. Bloquear un comando destructivo

`.claude/hooks/block-rm.sh`:

```bash
#!/bin/bash
COMMAND=$(jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q 'rm -rf'; then
  jq -n '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: "Comandos destructivos bloqueados por política"
    }
  }'
fi
exit 0
```

```json
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "Bash", "hooks": [
        { "type": "command", "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/block-rm.sh" }
      ]}
    ]
  }
}
```

### 2. Formatear el código tras cada edición

```json
{
  "hooks": {
    "PostToolUse": [
      { "matcher": "Edit|Write", "hooks": [
        { "type": "command", "command": "npx", "args": ["prettier", "--write", "${tool_input.file_path}"] }
      ]}
    ]
  }
}
```

### 3. Notificar cuando Claude termina de responder

```json
{
  "hooks": {
    "Stop": [
      { "hooks": [
        { "type": "command", "command": "notify-send", "args": ["Claude Code", "Tarea terminada"] }
      ]}
    ]
  }
}
```

## Variables de entorno útiles dentro de un hook

- `$CLAUDE_PROJECT_DIR` — raíz del proyecto.
- `$CLAUDE_PLUGIN_ROOT` — directorio del plugin, si el hook viene de uno.
- `$CLAUDE_CODE_REMOTE` — `"true"` cuando la sesión corre en Claude Code Cloud.

## Gestión

- `/hooks` (dentro de una sesión) — muestra todos los hooks configurados, sus matchers y de dónde vienen.
- `"disableAllHooks": true` en `settings.json` — desactiva todos los hooks de golpe (útil para depurar).

## Ver también

- [[claude/claude-code|Claude Code]]
- [[claude/comandos|Comandos útiles de Claude Code]]
- [[tools/rtk|rtk (Rust Token Killer)]] — ejemplo real de herramienta construida sobre hooks: reescribe comandos de shell de forma transparente.

Sources:
- [Hooks — Claude Code Docs](https://code.claude.com/docs/en/hooks)
- [Claude Code hooks: Una guía práctica con ejemplos (2026)](https://www.eesel.ai/blog/hooks-in-claude-code)
- [Claude Code Hooks: guía práctica de automatización — DataCamp](https://www.datacamp.com/tutorial/claude-code-hooks)
