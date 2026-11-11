---
title: MCP en OpenCode
---

OpenCode se conecta a servidores [Model Context Protocol](https://modelcontextprotocol.io/) y expone sus capacidades: herramientas, prompts, recursos e instrucciones. Es el mismo estándar que usa [[claude/extension/mcp|Claude Code]], pero la configuración de V2 cambia: los servidores van anidados bajo `mcp.servers`.

## Añadir un servidor

```bash
# Servidor remoto en el proyecto actual
opencode mcp add context7 --url https://mcp.context7.com/mcp

# Servidor remoto global
opencode mcp add context7 --global --url https://mcp.context7.com/mcp

# Servidor local por stdio
opencode mcp add everything -- npx -y @modelcontextprotocol/server-everything

# Con variables de entorno
opencode mcp add everything --env LOG_LEVEL=info -- npx -y @modelcontextprotocol/server-everything

# Estado y autenticación
opencode mcp list
opencode mcp auth sentry
opencode mcp logout sentry
```

`opencode mcp add` escribe en la configuración del proyecto; `--global` lo hace disponible en todos. Los servidores remotos usan OAuth por defecto. Si `mcp list` dice `needs authentication`, abre `/mcps`, selecciona el servidor y completa el login.

## Configuración manual

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "servers": {
      "context7": {
        "type": "remote",
        "url": "https://mcp.context7.com/mcp"
      },
      "everything": {
        "type": "local",
        "command": ["npx", "-y", "@modelcontextprotocol/server-everything"],
        "cwd": ".",
        "environment": {
          "LOG_LEVEL": "info",
          "MCP_API_KEY": "{env:MCP_API_KEY}"
        }
      }
    }
  }
}
```

- Para desconectar sin borrar la definición: `"disabled": true`. En V2 no existe `enabled`.
- Un config de mayor precedencia reemplaza el objeto entero del servidor con el mismo nombre. Si necesitas otra cuenta o conexión, usa otro nombre.
- Los secretos van en variables de entorno con `{env:NAME}`, no en el JSON.

### Servidor local

| Campo | Obligatorio | Descripción |
|---|---|---|
| `type` | Sí | `"local"`. |
| `command` | Sí | Ejecutable y argumentos. |
| `cwd` | No | Directorio del proceso; relativo al workspace. |
| `environment` | No | Variables añadidas al entorno heredado. |
| `disabled` | No | No conecta si es `true`. |
| `codemode` | No | `false` expone las herramientas directamente en vez de vía Code Mode. Por defecto `true`. |
| `timeout` | No | Timeouts por servidor. |
| `protocol` | No | `legacy` (defecto), `auto` o `2026-07-28`. |
| `pool` | No | Comparte una conexión entre directorios. Por defecto `false`. |

### Servidor remoto

| Campo | Obligatorio | Descripción |
|---|---|---|
| `type` | Sí | `"remote"`. |
| `url` | Sí | Endpoint HTTP absoluto (Streamable HTTP). |
| `headers` | No | Cabeceras HTTP; usa `{env:NAME}` para secretos. |
| `oauth` | No | Ajustes OAuth o `false` para desactivarlo. |
| `disabled` | No | No conecta si es `true`. |
| `codemode` | No | `false` expone las herramientas directamente. |
| `timeout` | No | Timeouts por servidor. |
| `protocol` | No | `legacy`, `auto` o `2026-07-28`. |
| `pool` | No | Comparte conexión entre directorios. |

## OAuth

OAuth está activo por defecto en remotos. OpenCode descubre el servidor de autorización, usa PKCE y refresca tokens; las credenciales se guardan fuera de la configuración del proyecto.

- Si el navegador está en otra máquina (SSH, contenedor, servidor remoto), copia la URL de callback de la barra de direcciones y pégala en el prompt de login.
- OpenCode se identifica con cliente pre-registrado, CIMD o registro dinámico, en ese orden.
- Para un cliente pre-registrado, usa los campos snake_case de V2: `client_id`, `client_secret`, `scope`, `callback_port`, `redirect_uri`, `auth_server_metadata_url`.

## Code Mode

Por defecto, las herramientas MCP se agrupan bajo el espacio de nombres del servidor y se llaman desde `execute` en Code Mode:

```text
tools.context7.resolve_library_id(...)
```

Esto reduce el ruido en el prompt: el modelo no ve todas las herramientas MCP como herramientas nativas. Si necesitas que estén en la lista nativa del proveedor, pon `codemode: false` en ese servidor.

## Nombres y prompts

OpenCode nombra cada herramienta como `<servidor>_<herramienta>`, sustituyendo caracteres no alfanuméricos por `_`:

```text
server: context 7
tool:   resolve.library/id
name:   context_7_resolve_library_id
```

Los prompts MCP se convierten en comandos `<servidor>:<prompt>`:

```text
/context_7:find_docs react "server components"
```

Elige nombres de servidor cortos y únicos después de la normalización.

## Permisos

Las herramientas MCP usan la acción `<servidor>_<herramienta>` con recurso `*`:

```jsonc
{
  "permissions": [
    {
      "action": "context7_*",
      "resource": "*",
      "effect": "deny"
    }
  ]
}
```

También se pueden controlar los recursos MCP:

| Acción | Recurso |
|---|---|
| `opencode_list_mcp_resources` | Nombre del servidor. |
| `opencode_read_mcp_resource` | `<servidor>:<uri>`. |

```jsonc
{
  "permissions": [
    {
      "action": "opencode_read_mcp_resource",
      "resource": "docs:*",
      "effect": "allow"
    }
  ]
}
```

## Timeouts, protocolo y conexiones compartidas

```jsonc
{
  "mcp": {
    "timeout": {
      "startup": 45000,
      "catalog": 30000,
      "execution": 600000
    },
    "servers": {
      "modern": {
        "type": "remote",
        "url": "https://mcp.example.com/mcp",
        "protocol": "auto",
        "pool": true
      }
    }
  }
}
```

| Timeout | Defecto | Aplica a |
|---|---:|---|
| `startup` | 30 s | Conexión e inicialización. |
| `catalog` | 30 s | Listar herramientas, prompts, recursos y plantillas. |
| `execution` | 12 h | Llamadas a herramientas, prompts y recursos. |

`protocol` permite hablar con servidores de la revisión MCP 2026-07-28. `auto` prueba `server/discover` y cae a `legacy` si no lo soporta. `pool: true` comparte una conexión entre directorios con la misma configuración; solo funciona con la revisión 2026-07-28.

## Instrucciones del servidor

Si un servidor devuelve instrucciones al inicializar, OpenCode las añade al system prompt dentro de `<mcp_instructions>`. Se omiten si el agente deniega todas las herramientas de ese servidor o si deniega `execute` para un servidor en Code Mode.

## Ver también

- [[opencode/extension/permisos-politicas|Permisos y políticas]]
- [[opencode/extension/plugins|Plugins]]
- [[opencode/fundamentos/configuracion|Configuración]]
- [[opencode/guias/desde-claude-code|Desde Claude Code]]

Sources:
- [OpenCode MCP Servers](https://opencode.ai/docs/mcp-servers/)
- [OpenCode Permissions](https://opencode.ai/docs/permissions/)
- [Model Context Protocol](https://modelcontextprotocol.io/)
