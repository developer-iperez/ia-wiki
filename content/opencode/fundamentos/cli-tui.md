---
title: CLI y TUI de OpenCode
---

OpenCode tiene dos formas principales de uso interactivo: la **TUI** de pantalla completa y la **CLI** para lanzar tareas sin abrir la interfaz. Ambas comparten el mismo servidor y las mismas sesiones.

## CLI

Cada comando acepta `--help`. Los más útiles:

| Comando | Para qué sirve |
|---|---|
| `opencode` | Abre la TUI en el directorio actual. |
| `opencode ~/code/proyecto` | Abre la TUI en otro proyecto. |
| `opencode run "prompt"` | Ejecuta una tarea y devuelve la respuesta sin TUI. |
| `opencode mini` | Interfaz interactiva mínima, útil en terminales pequeñas. |
| `opencode session list` | Lista sesiones; `--format json` para scripts. |
| `opencode session export <id>` | Exporta una sesión a JSON; `--sanitize` redacta datos sensibles. |
| `opencode session import <fichero>` | Importa una sesión. |
| `opencode auth list` | Lista proveedores y cuentas conectadas. |
| `opencode auth login <proveedor>` | Inicia sesión en un proveedor. |
| `opencode models` | Lista todos los modelos disponibles. |
| `opencode mcp list` | Estado de los servidores MCP. |
| `opencode plugin list` | Lista plugins; `--builtin` incluye los internos. |
| `opencode stats` | Estadísticas de uso; `--days`, `--models`, `--cost`, `--json`. |
| `opencode serve` | Arranca el servidor API/web. |
| `opencode pair` | Genera enlaces temporales para conectar un navegador o dispositivo. |
| `opencode service status` | Estado del servidor compartido. |
| `opencode service restart` | Reinicia el servidor si está atascado. |
| `opencode reload` | Recarga configuración sin reiniciar el servidor. |
| `opencode api GET /api/session` | Llama a la API del servidor. |
| `opencode acp` | Servidor Agent Client Protocol para integraciones de editor. |
| `opencode debug paths` | Muestra rutas de datos, configuración, caché y logs. |
| `opencode upgrade` | Actualiza OpenCode. |
| `opencode uninstall` | Desinstala; no borra sesiones ni configuración. |

### Automatización con `opencode run`

```bash
opencode run "Explain this repository"
opencode run --model anthropic/claude-sonnet-5-5 "Refactor parseToken"
opencode run --continue "Now handle the expired case"
opencode run --format json "List the TODO comments"
opencode run --file src/server.ts --file src/client.ts "Review these for bugs"
opencode run --agent build "Fix the failing test"
opencode run --session ses_4f2a1c "Check edge cases"
opencode run --auto "Fix the failing test"
```

- `--auto` aprueba automáticamente los permisos que no estén denegados explícitamente.
- `--archived` crea la sesión archivada para que no aparezca en las listas.
- En CI, usa `--standalone` para que el servidor privado reciba la API key del secreto.

## TUI

### Prompt

- **Enter** envía; **Shift+Enter**, **Ctrl+Enter**, **Alt+Enter** o **Ctrl+J** insertan salto de línea.
- Mientras OpenCode trabaja, **Enter** dirige la sesión activa; **Ctrl+X** y luego **Enter** encola el prompt para después.
- `@` busca y adjunta un archivo. Se puede indicar una línea o rango: `@src/auth.ts#20-45`.
- `!` al inicio de una línea vacía entra en modo shell: `!git status`. La salida queda en la sesión.
- `/` abre la lista de comandos. **Ctrl+P** abre la paleta con todas las acciones disponibles.
- `/btw <pregunta>` hace una pregunta lateral usando el contexto de la conversación, pero sin añadirla al historial.

### Atajos principales

El **leader** por defecto es **Ctrl+X**:

| Acción | Atajo |
|---|---|
| Elegir modelo | **Ctrl+X**, luego **M** |
| Elegir agente | **Ctrl+X**, luego **A** |
| Nueva sesión | **Ctrl+X**, luego **N** |
| Listar sesiones | **Ctrl+X**, luego **L** |
| Deshacer | **Ctrl+X**, luego **U** |
| Rehacer | **Ctrl+X**, luego **R** |
| Editor externo | **Ctrl+X**, luego **E** |
| Cerrar pestaña | **Ctrl+X**, luego **W** |
| Siguiente/anterior agente | **Shift+Tab** |
| Modelo reciente | **F2** |
| Sesiones y proyectos recientes | **Ctrl+O** |
| Cambiar pestaña | **Ctrl+Tab** / **Ctrl+Shift+Tab** |

Los atajos se configuran en `~/.config/opencode/cli.json` bajo `keybinds`; el leader se cambia con `keybinds.leader`. También se puede abrir **Ctrl+P → Open settings**. La lista completa de IDs y valores por defecto está en la [referencia de keybinds](https://opencode.ai/docs/cli/keybinds/).

### Sesiones y edición

- **Ctrl+X N** crea una sesión; **Ctrl+X L** o `/sessions` vuelve a una existente.
- Las pestañas están activadas por defecto. **Ctrl+Tab** cambia, **Ctrl+X W** cierra y **Ctrl+Shift+T** reabre.
- `/undo` revierte el último mensaje y el trabajo que le siguió; el prompt vuelve al editor. `/redo` lo restaura. Enviar un prompt nuevo acepta el revert.
- `/editor` abre el prompt actual en `$VISUAL` o `$EDITOR`.
- `/models` y `/agents` son las vías rápidas para cambiar de modelo o de agente.
- `/mcps` gestiona servidores MCP; `/diff` abre el visor de cambios.

## Ver también

- [[opencode/fundamentos/opencode|Qué es OpenCode]] — instalación y arquitectura.
- [[opencode/fundamentos/configuracion|Configuración]] — `opencode.jsonc` y `cli.json`.
- [[opencode/extension/comandos|Comandos personalizados]] — slash commands propios.
- [[opencode/guias/flujo-de-trabajo|Flujo de trabajo diario]] — cómo encadenar plan, edición y verificación.

Sources:
- [OpenCode CLI](https://opencode.ai/docs/cli/)
- [OpenCode CLI Commands](https://opencode.ai/docs/cli/commands/)
- [OpenCode TUI](https://opencode.ai/docs/cli/tui/)
- [OpenCode Keybinds](https://opencode.ai/docs/cli/keybinds/)
- [OpenCode CLI Settings](https://opencode.ai/docs/cli/config/)
