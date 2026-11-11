---
title: Qué es OpenCode
---

OpenCode es un agente de codificación open source desarrollado por [Anomaly](https://opencode.ai/). Igual que [[claude/fundamentos/claude-code|Claude Code]], trabaja dentro de un proyecto: lee archivos, planifica cambios, edita código, ejecuta comandos y tests, y repite el ciclo según el resultado. La diferencia principal es que OpenCode es agnóstico de proveedor y separa la interfaz del servidor que ejecuta el trabajo.

> Esta sección documenta **OpenCode V2**. La configuración de V1 y su API de plugins son distintas; para migrar, consulta la [guía oficial de migración](https://opencode.ai/docs/migrate-v1/).

## Interfaces

OpenCode se puede usar de varias formas, todas conectadas al mismo servidor:

- **TUI**: la interfaz principal de terminal. Se arranca con `opencode` dentro del proyecto.
- **CLI**: `opencode run "..."` para automatización y CI, y `opencode mini` para una interfaz interactiva mínima.
- **Escritorio**: aplicación nativa para macOS, Windows y Linux.
- **Web**: `opencode pair` genera un enlace temporal para abrir la interfaz web desde otro dispositivo.
- **Docker**: imágenes versionadas, por ejemplo `ghcr.io/anomalyco/opencode:2.0.0`.

## Instalación

Métodos principales:

```bash
# Instalador oficial
curl -fsSL https://opencode.ai/v2/install | bash

# Homebrew
brew install anomalyco/tap/opencode-v2

# npm (instala el binario nativo según la plataforma)
npm install -g @opencode/cli

# Alternativas
bun install -g --trust @opencode/cli
pnpm add -g --allow-build=@opencode/cli @opencode/cli
yarn global add @opencode/cli
```

También hay binarios independientes para macOS, Windows y Linux (glibc y musl) en la [página de instalación](https://opencode.ai/docs/). Los gestores de paquetes de Windows no están soportados; en Windows se usa el binario o el instalador de escritorio.

## Primer arranque

```bash
cd ~/code/mi-proyecto
opencode
```

Dentro de la TUI:

1. `/connect` para conectar un proveedor (API key, OAuth, OpenCode Console o Go).
2. `/models` para elegir modelo.
3. Escribe el prompt y pulsa **Enter**.
4. Usa `@` para adjuntar un archivo y `!` al principio de una línea vacía para ejecutar un comando de shell.

Para trabajar en otro directorio sin cambiar de carpeta:

```bash
opencode ~/code/otro-proyecto
```

## Arquitectura cliente-servidor

OpenCode no es solo un binario de terminal. Por defecto descubre o arranca **un servidor compartido en segundo plano** para el usuario. Ese servidor es el dueño de las sesiones, la configuración, las integraciones, los permisos y la ejecución de herramientas. La TUI, el escritorio, la web y la CLI son clientes que se conectan a él.

- `opencode --standalone` arranca un servidor privado para ese proceso.
- `opencode --server http://localhost:4096` se conecta a un servidor concreto.
- `opencode service status` comprueba el servicio compartido; `opencode service restart` lo reinicia si está atascado.
- La configuración se recarga sin reiniciar; no hace falta reiniciar tras editar `opencode.jsonc`.

Esta separación permite tener la TUI abierta, lanzar tareas desde la web o el móvil y automatizar con `opencode run` contra el mismo estado.

## Configuración mínima

- **Servidor/proyecto**: `opencode.json` u `opencode.jsonc` en la raíz del proyecto, o `.opencode/opencode.json(c)`. Global en `~/.config/opencode/opencode.json(c)`.
- **Terminal**: `~/.config/opencode/cli.json` para temas, atajos, pestañas, diffs y notificaciones. No se mezcla con la configuración del servidor.
- **Instrucciones**: `AGENTS.md` en el proyecto y en `~/.config/opencode/AGENTS.md`. OpenCode V2 **no usa `CLAUDE.md` como fallback**.
- **Permisos**: reglas ordenadas en `permissions` y bloqueos duros en `policies`.

Un `opencode.jsonc` mínimo:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "opencode-go/kimi-k3",
  "default_agent": "build",
  "permissions": [
    { "action": "shell", "resource": "*", "effect": "ask" },
    { "action": "shell", "resource": "git status *", "effect": "allow" },
    { "action": "shell", "resource": "git push *", "effect": "deny" }
  ]
}
```

## Ver también

- [[opencode/fundamentos/cli-tui|CLI y TUI]] — manejo diario y automatización.
- [[opencode/fundamentos/configuracion|Configuración]] — todas las piezas de `opencode.jsonc`.
- [[opencode/fundamentos/instrucciones|Instrucciones y AGENTS.md]] — contexto persistente.
- [[opencode/guias/desde-claude-code|Desde Claude Code]] — si vienes de Claude Code.

Sources:
- [OpenCode: Intro](https://opencode.ai/docs/)
- [OpenCode CLI](https://opencode.ai/docs/cli/)
- [OpenCode Config](https://opencode.ai/docs/config/)
- [OpenCode Troubleshooting](https://opencode.ai/docs/troubleshooting/)
