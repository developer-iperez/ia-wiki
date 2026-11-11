---
title: Configuración de OpenCode
---

OpenCode separa dos capas de configuración:

- **Servidor y proyecto**: `opencode.json` / `opencode.jsonc`. Controla modelos, agentes, permisos, MCP, plugins, instrucciones, compactación, snapshots, etc.
- **Terminal (TUI)**: `~/.config/opencode/cli.json`. Controla tema, atajos, pestañas, diffs, notificaciones y ajustes de la interfaz. No hay configuración local de CLI por proyecto.

## `opencode.jsonc`

Añade siempre el esquema para validación y autocompletado:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "opencode-go/kimi-k3",
  "default_agent": "build"
}
```

Se acepta JSON o JSONC (comentarios y comas finales). Los cambios se recargan automáticamente; no hace falta reiniciar. Para forzar una recarga completa: `/reload` o `opencode reload`.

## Ubicaciones y precedencia

| Ámbito | Ruta |
|---|---|
| Sistema (todos los usuarios) | Linux `/etc/opencode/`; macOS `/Library/Application Support/opencode`; Windows `%ProgramData%\opencode` |
| Global del usuario | `~/.config/opencode/opencode.json(c)` |
| Proyecto | `opencode.json(c)` o `.opencode/opencode.json(c)` en cualquier directorio del proyecto |

OpenCode busca desde el directorio actual hacia la raíz. Primero fusiona los `opencode.json(c)` directos de la raíz más lejana al directorio actual, y después los de `.opencode/` en ese mismo orden. Por eso **todo `.opencode` pisa a todo config directo**. La configuración de sistema tiene la precedencia más alta.

Ejemplo de menor a mayor precedencia al arrancar desde `acme/packages/web`:

```text
~/.config/opencode/opencode.json
/home/user/projects/acme/opencode.json
/home/user/projects/acme/packages/web/opencode.json
/etc/opencode/opencode.json
```

Los ajustes que no colisionan se conservan. Las **políticas** invierten la precedencia: las fuentes más amplias se evalúan después y pueden bloquear a las más concretas (ver [[agentes/opencode/extension/permisos-politicas|Permisos y políticas]]).

## Campos principales

| Campo | Para qué sirve |
|---|---|
| `model` | Modelo por defecto en formato `proveedor/modelo` (y `#variante` en agentes/comandos). |
| `default_agent` | Agente primario por defecto; si no existe, usa `build` y luego el primer primario visible. |
| `providers` | Proveedores y modelos: endpoints, credenciales, headers, variantes, límites. |
| `agents` | Agentes primarios/subagentes: prompt, modelo, permisos, pasos. |
| `commands` | Comandos slash como plantillas de prompt. |
| `skills` | Directorios o catálogos HTTP adicionales de skills. |
| `instructions` | Archivos, globs o URLs de instrucciones además de `AGENTS.md`. |
| `permissions` | Reglas ordenadas `allow`/`ask`/`deny` por agente y recurso. |
| `policies` | Bloqueos duros de proveedor, herramienta, MCP, skill, plugin o búsqueda web. |
| `mcp` | Servidores MCP locales y remotos bajo `mcp.servers`. |
| `plugins` | Plugins de servidor/TUI/GUI y sus opciones. |
| `references` | Directorios o repositorios de contexto con nombre. |
| `formatter` | Formateo automático tras `write`, `edit` o `patch`. |
| `snapshots` | Capturas de filesystem para `/undo` y `/redo`. |
| `compaction` | Compactación automática del historial y tokens recientes a conservar. |
| `websearch` | Proveedor de búsqueda web o `false` para desactivarla. |
| `watcher` | Archivos y directorios ignorados por el watcher. |
| `worktree` | Directorio padre para worktrees locales. |

## Variables

Dentro de cualquier string de configuración:

```jsonc
{
  "model": "{env:OPENCODE_MODEL}",
  "providers": {
    "anthropic": {
      "settings": { "apiKey": "{env:ANTHROPIC_API_KEY}" }
    },
    "local": {
      "settings": { "apiKey": "{file:~/.secrets/local-key}" }
    }
  }
}
```

- `{env:VAR}` se sustituye por la variable de entorno; si no existe, queda vacío.
- `{file:ruta}` se sustituye por el contenido del archivo. Rutas relativas al config, absolutas o con `~/`.
- Útil para no meter secretos en el repositorio.

## Configuración de terminal: `cli.json`

Vive en `~/.config/opencode/cli.json` (o `$XDG_CONFIG_HOME/opencode/cli.json`). Ejemplo:

```json
{
  "$schema": "https://opencode.ai/v2/cli.json",
  "theme": { "name": "tokyonight", "mode": "system" },
  "keybinds": { "leader": "ctrl+x" },
  "tabs": { "mode": "auto", "scope": "cwd" },
  "diffs": { "source": "branch", "view": "auto" },
  "attention": { "notifications": true, "sound": true }
}
```

- `OPENCODE_CLI_CONFIG_CONTENT` permite inyectar JSON inline que se fusiona sobre el archivo global.
- Los ajustes de `cli.json` no se deben poner en `opencode.jsonc`.
- La configuración del servidor se recarga sola; la de `cli.json` también se recarga mientras la TUI está abierta.

## Compactación y snapshots

Dos ajustes que conviene conocer desde el principio:

- **Compactación**: por defecto se activa al acercarse al límite de contexto, conserva ~15.000 tokens recientes y deja un 15 % de la ventana libre. Se configura con `compaction.auto`, `compaction.keep.tokens` y `compaction.buffer`.
- **Snapshots**: permiten que `/undo` restaure también archivos, no solo la conversación. Están activados por defecto y requieren un repositorio Git. No son una copia de seguridad ni revierten efectos de shell.

## Ver también

- [[agentes/opencode/fundamentos/instrucciones|Instrucciones y AGENTS.md]]
- [[agentes/opencode/fundamentos/modelos-proveedores|Modelos y proveedores]]
- [[agentes/opencode/extension/permisos-politicas|Permisos y políticas]]
- [[agentes/opencode/guias/flujo-de-trabajo|Flujo de trabajo diario]]

Sources:
- [OpenCode Config](https://opencode.ai/docs/config/)
- [OpenCode CLI Settings](https://opencode.ai/docs/cli/config/)
- [OpenCode Compaction](https://opencode.ai/docs/compaction/)
- [OpenCode Snapshots](https://opencode.ai/docs/snapshots/)
