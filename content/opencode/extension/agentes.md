---
title: Agentes y subagentes en OpenCode
---

Un **agente** en OpenCode es un perfil con nombre que combina un system prompt, un modelo preferido, permisos y detalles de presentación. Es el equivalente a los [[claude/extension/subagentes|subagentes de Claude Code]], pero OpenCode distingue entre agentes **primarios** y **subagentes** y permite definir ambos con el mismo formato.

## Dónde se definen

| Ámbito | Ruta |
|---|---|
| Global | `~/.config/opencode/agents/<nombre>.md` |
| Proyecto | `.opencode/agents/<nombre>.md` |
| Config JSON | Clave `agents` en cualquier `opencode.json(c)` |

Los agentes de proyecto se descubren desde el directorio actual hacia la raíz del proyecto. Una ruta anidada forma parte del ID:

```text
.opencode/agents/team/reviewer.md  →  team/reviewer
```

## Formato Markdown

```md title=".opencode/agents/reviewer.md"
---
description: Revisa cambios en busca de errores y regresiones
mode: subagent
model: anthropic/claude-sonnet-5-5#high
permissions:
  - action: edit
    resource: "*"
    effect: deny
  - action: shell
    resource: "*"
    effect: deny
---

Revisa los cambios actuales. Lista los hallazgos por severidad con archivo y línea.
```

El cuerpo Markdown es el `system` prompt del agente. El frontmatter acepta los mismos campos que la entrada JSON.

## Modos

| Modo | Comportamiento |
|---|---|
| `primary` | Se ejecuta como agente principal de una sesión. Es el valor por defecto de un agente nuevo. |
| `subagent` | Solo se ejecuta en una sesión hija mediante la herramienta `subagent`. |
| `all` | Puede usarse como primario o como subagente. |

Los subagentes arrancan con contexto limpio, en primer plano o en segundo plano. El padre controla qué agentes puede lanzar con el permiso `subagent`; el hijo usa sus propios permisos, no un subconjunto de los del padre.

```jsonc
{
  "agents": {
    "orchestrator": {
      "permissions": [
        { "action": "subagent", "resource": "*", "effect": "deny" },
        { "action": "subagent", "resource": "reviewer", "effect": "allow" }
      ]
    }
  }
}
```

## Agentes integrados

| Agente | Modo | Propósito |
|---|---|---|
| **Build** (`build`) | `primary` | Agente de código por defecto. Acceso amplio; las lecturas de `.env` piden aprobación. |
| **Plan** (`plan`) | `primary` | Explora y planifica sin editar archivos normales. Puede escribir archivos de plan de OpenCode. |
| **General** (`general`) | `subagent` | Investigación y trabajo multi-paso con acceso amplio, sin lanzar más subagentes. |
| **Explore** (`explore`) | `subagent` | Busca y lee código o fuentes web sin editar. |

Hay agentes internos ocultos (`compaction`, `title`, `summary`) que hacen mantenimiento y no se pueden seleccionar. Para sobrescribir un integrado, usa su mismo ID.

## Campos principales

| Campo | Uso |
|---|---|
| `description` | Explica el propósito; importante en subagentes porque el modelo la usa para elegir. |
| `mode` | `primary`, `subagent` o `all`. |
| `model` | `proveedor/modelo` con `#variante` opcional. |
| `system` | System prompt (en Markdown, el cuerpo). |
| `permissions` | Reglas ordenadas `action`/`resource`/`effect`. |
| `steps` | Máximo de pasos de modelo; al llegar, retira herramientas y pide un resumen. |
| `hidden` | Lo oculta de listados y del catálogo de subagentes; no es seguridad. |
| `color` | Color de UI en hex de seis dígitos. |
| `disabled` | Elimina un agente integrado o personalizado en ese punto de la carga. |
| `request` | Headers/body por agente; en V2 se conserva pero aún no se envía con las peticiones. |

## Elegir agente

- **Ctrl+X**, luego **A** en la TUI, o `/agents`.
- **Shift+Tab** cicla agentes.
- `default_agent` fija el primario por defecto. Debe existir, ser visible y soportar uso primario; si no, OpenCode usa `build` y luego el primer primario visible.

```jsonc
{
  "default_agent": "plan",
  "agents": {
    "plan": { "mode": "primary" }
  }
}
```

Cambiar `default_agent` no cambia el agente guardado en una sesión existente.

## Ejemplo: reviewer de solo lectura

```md title=".opencode/agents/reviewer.md"
---
description: Revisa cambios sin modificarlos
mode: subagent
permissions:
  - action: edit
    resource: "*"
    effect: deny
  - action: shell
    resource: "*"
    effect: deny
---

Revisa el diff actual. Prioriza correctness, seguridad y tests que falten.
```

Uso desde la sesión principal:

```text
Usa el subagente reviewer para revisar mis cambios actuales.
```

## Ver también

- [[opencode/extension/skills|Skills]]
- [[opencode/extension/permisos-politicas|Permisos y políticas]]
- [[opencode/fundamentos/modelos-proveedores|Modelos y proveedores]]
- [[opencode/guias/desde-claude-code|Desde Claude Code]]

Sources:
- [OpenCode Agents](https://opencode.ai/docs/agents/)
- [OpenCode Tools: subagent](https://opencode.ai/docs/tools/)
- [OpenCode Permissions](https://opencode.ai/docs/permissions/)
