---
title: Permisos y políticas en OpenCode
---

OpenCode tiene dos capas de control que conviene no confundir:

- **Permisos (`permissions`)**: reglas por agente y recurso que pueden `allow`, `ask` o `deny`. Son el mecanismo normal para decidir qué puede hacer cada agente.
- **Políticas (`policies`)**: sentencias binarias `allow`/`deny` que **solo endurecen** lo que los permisos ya permiten. Nunca piden confirmación y nunca conceden acceso por sí solas. Sirven para bloqueos duros, gestión por organización o MDM.

## Permisos

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "permissions": [
    { "action": "shell", "resource": "*", "effect": "ask" },
    { "action": "shell", "resource": "git status *", "effect": "allow" },
    { "action": "shell", "resource": "git diff *", "effect": "allow" },
    { "action": "shell", "resource": "git push *", "effect": "deny" }
  ]
}
```

Cada regla tiene tres campos:

| Campo | Significado |
|---|---|
| `action` | Acción o herramienta: `shell`, `edit`, `read`, `webfetch`, `subagent`, `skill`… |
| `resource` | Path, comando, URL, query, ID de skill o de agente. |
| `effect` | `allow`, `ask` o `deny`. |

**La última regla que coincide gana.** Por eso las reglas amplias van primero y las excepciones después. Si no coincide ninguna, OpenCode usa `ask`.

Los wildcards son de valor completo: `*` coincide con cero o más caracteres (incluido `/`) y `?` con exactamente uno. Un patrón de shell que termina en ` *` también coincide con el comando sin argumentos:

```jsonc
{ "action": "shell", "resource": "git status *", "effect": "allow" }
```

Cubre `git status` y `git status --short`.

### Acciones integradas

| Acción | Recurso |
|---|---|
| `read` | Ruta interna relativa o ruta externa canónica absoluta. |
| `edit` | Ruta objetivo de `edit`, `write` y `patch`. |
| `glob` | Patrón solicitado. |
| `grep` | Expresión regular solicitada, no la ruta de búsqueda. |
| `shell` | Comando producido por el scanner; un comando compuesto puede generar varios. |
| `subagent` | ID del agente objetivo. |
| `skill` | ID de la skill. |
| `question` | `*`. |
| `webfetch` | URL solicitada. |
| `websearch` | Query de búsqueda. |
| `experimental_image_generate` | Modelo de imagen seleccionado. |
| `external_directory` | Límite de directorio externo, normalmente acabado en `/*`. |
| `<server>_<tool>` | Herramienta MCP; recurso `*`. |
| `opencode_list_mcp_resources` | Nombre del servidor MCP. |
| `opencode_read_mcp_resource` | `<server>:<uri>`. |
| `execute` | `*`; denegarlo elimina Code Mode y las herramientas que solo se alcanzan por él. |

### Reglas por defecto

Todos los agentes, incluidos los personalizados, parten de esta política base:

```jsonc
[
  { "action": "*", "resource": "*", "effect": "allow" },
  { "action": "read", "resource": "*.env", "effect": "ask" },
  { "action": "read", "resource": "*.env.*", "effect": "ask" },
  { "action": "read", "resource": "*.env.example", "effect": "allow" }
]
```

Los agentes integrados añaden matices:

| Agente | Política adicional |
|---|---|
| `build` | Permite preguntas. |
| `plan` | Permite preguntas; deniega edits salvo archivos bajo `~/.opencode/plan`. |
| `general` | Deniega preguntas y lanzar subagentes. |
| `explore` | Solo permite shell, reads, globs, grep, webfetch, websearch y directorios externos; pide para `.env`. |
| `title`, `summary` | Deniegan todo. |
| `compaction` | Mantiene la política base. |

Como ninguna regla por defecto pide `external_directory`, leer o editar fuera del proyecto no pregunta. Para cambiarlo:

```jsonc
{
  "permissions": [
    { "action": "external_directory", "resource": "*", "effect": "ask" },
    { "action": "external_directory", "resource": "~/projects/reference/*", "effect": "allow" },
    { "action": "read", "resource": "~/projects/reference/*", "effect": "allow" },
    { "action": "edit", "resource": "~/projects/reference/*", "effect": "deny" }
  ]
}
```

### Permisos por agente

Las reglas globales se aplican antes; las del agente se añaden después y pueden refinar:

```jsonc
{
  "permissions": [
    { "action": "shell", "resource": "*", "effect": "ask" },
    { "action": "shell", "resource": "git status *", "effect": "allow" }
  ],
  "agents": {
    "reviewer": {
      "mode": "subagent",
      "permissions": [
        { "action": "edit", "resource": "*", "effect": "deny" }
      ]
    }
  }
}
```

Un subagente personalizado usa sus propios permisos, no un subconjunto de los del padre.

### Aprobaciones

Cuando una regla resuelve a `ask`, el cliente puede responder:

| Opción | Respuesta | Resultado |
|---|---|---|
| Permitir una vez | `once` | Solo esa petición. |
| Permitir siempre | `always` | Guarda los patrones propuestos como reglas `allow` del proyecto. |
| Rechazar | `reject` | Rechaza esa petición y las demás pendientes de la sesión. |

Las aprobaciones guardadas son reglas durables y locales al proyecto. Nunca pisan un `deny` configurado.

> **Aviso:** `shell` se ejecuta con los permisos del usuario del host. La inferencia de directorios a partir del texto del comando es best-effort; prefiere una allowlist estrecha de comandos en vez de patrones que intenten reconocer todo lo peligroso.

## Políticas

```jsonc
{
  "policies": [
    { "action": "provider.use", "resource": "openai", "effect": "deny" }
  ]
}
```

Cada sentencia tiene `action`, `resource` y `effect` (`allow` o `deny`). **La última que coincide gana**; si no coincide ninguna, se permite. Las sentencias inválidas se descartan con un aviso en el log.

### Acciones de política

| Acción | Recurso |
|---|---|
| `provider.use` | ID del proveedor. |
| `tool.use` | `<acción>:<valor>` de la comprobación de permiso. |
| `integration.connect` | ID de integración que aparece en `/connect`. |
| `integration.use` | `<integración>#<cuenta>`. |
| `mcp.use` | Nombre del servidor MCP. |
| `skill.use` | ID de la skill. |
| `plugin.use` | Nombre de paquete sin versión o ruta absoluta. |
| `websearch.use` | ID del proveedor de búsqueda. |

### Ejemplos

**Solo un proveedor:**

```jsonc
{
  "policies": [
    { "action": "provider.use", "resource": "*", "effect": "deny" },
    { "action": "provider.use", "resource": "anthropic", "effect": "allow" }
  ]
}
```

**Bloquear comandos peligrosos aunque un agente los tenga permitidos:**

```jsonc
{
  "policies": [
    { "action": "tool.use", "resource": "shell:git push *", "effect": "deny" },
    { "action": "tool.use", "resource": "edit:*.env", "effect": "deny" },
    { "action": "tool.use", "resource": "webfetch:*", "effect": "deny" }
  ]
}
```

Un `tool.use` denegado falla con `Blocked by configuration policy`; no pregunta. Se aplica después de las reglas de agente y de las aprobaciones guardadas, así que pisa tanto un `ask` como un “Allow always”. Un `tool.use` con `allow` no concede acceso: solo levanta un `deny` anterior más amplio, y luego deciden las reglas del agente.

**Bloquear MCP, skills o plugins:**

```jsonc
{
  "policies": [
    { "action": "mcp.use", "resource": "*", "effect": "deny" },
    { "action": "mcp.use", "resource": "github", "effect": "allow" },
    { "action": "plugin.use", "resource": "opencode-acme-cli", "effect": "deny" }
  ]
}
```

### Precedencia

Las políticas invierten la precedencia normal: las fuentes más amplias se evalúan después y pueden bloquear a las más concretas.

| Autoridad | Fuente |
|---|---|
| 1 (mayor) | Preferencias gestionadas de macOS. |
| 2 | `opencode.json(c)` de sistema. |
| 3 | Workspace conectado de OpenCode Console. |
| 4 | `~/.config/opencode/opencode.json(c)` global. |
| 5 | `opencode.json(c)` directo; los directorios externos ganan a los internos. |
| 6 (menor) | `.opencode/opencode.json(c)`; los externos ganan a los internos. |

Un repositorio no puede reactivar un proveedor que denegaste globalmente. Las políticas de un workspace de Console se aplican en un minuto aproximadamente y no pueden desactivarse desde `plugins`.

## Ver también

- [[agentes/opencode/extension/agentes|Agentes y subagentes]]
- [[agentes/opencode/extension/mcp|MCP]]
- [[agentes/opencode/fundamentos/configuracion|Configuración]]
- [[agentes/opencode/guias/flujo-de-trabajo|Flujo de trabajo diario]]

Sources:
- [OpenCode Permissions](https://opencode.ai/docs/permissions/)
- [OpenCode Policies](https://opencode.ai/docs/policies/)
- [OpenCode Config](https://opencode.ai/docs/config/)
