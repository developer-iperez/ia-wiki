---
title: Plugins en OpenCode
---

Un **plugin** es un paquete que modifica el comportamiento de OpenCode o añade funciones nuevas. Un mismo paquete puede tener varias partes: servidor, TUI, escritorio (GUI) y RPC. Es el mecanismo más potente de extensión y el equivalente más cercano a los [[claude/extension/hooks|hooks de Claude Code]], aunque va bastante más allá: puede añadir herramientas, comandos, agentes, integraciones y elementos de interfaz.

## Partes de un plugin

| Parte | Export | Dónde corre | Qué puede hacer |
|---|---|---|---|
| Servidor | `.` o `./server` | Servidor de OpenCode | Herramientas, hooks, integraciones, comandos y agentes. |
| TUI | `./tui` | Interfaz de terminal | Comandos, rutas, slots y renderizadores Markdown. |
| GUI | `./gui-experimental` | OpenCode Desktop | Paneles, comandos, ajustes y integraciones nativas. |
| RPC | `./rpc` | Contrato tipado | Métodos y eventos entre la parte servidor, las otras partes y clientes. |

Todas las partes usan el mismo paquete `@opencode/plugin`.

## Estructura mínima

```ts title=".opencode/plugins/example/index.ts"
import { Plugin } from "@opencode/plugin"

export default Plugin.define({
  id: "example",
  async setup(ctx) {
    await ctx.storage.set("loaded", true)
  },
})
```

Para publicarlo como paquete npm:

```json title="package.json"
{
  "name": "opencode-acme-plugin",
  "version": "1.0.0",
  "type": "module",
  "exports": {
    ".": "./src/index.ts",
    "./tui": "./src/tui.tsx",
    "./gui-experimental": "./dist/gui-experimental.cjs",
    "./rpc": "./src/rpc.ts"
  },
  "files": ["src", "dist"],
  "dependencies": {
    "@opencode/plugin": "latest"
  }
}
```

El `id` sale de la parte servidor (`Plugin.define`). Un paquete solo TUI declara su propio `id`; una parte GUI no declara ninguno y toma el del plugin.

## Instalación y carga

Plugins locales en `.opencode/plugins/` se cargan automáticamente. Para paquetes publicados u otras rutas, usa `plugins` en `opencode.json(c)`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "plugins": [
    "opencode-acme-plugin",
    "opencode-acme-plugin@1.2.0",
    "@acme/opencode-plugin",
    "./plugins/local",
    "../shared/plugin",
    "/absolute/path/plugin",
    "file:///home/me/plugins/local",
    {
      "package": "@acme/opencode-plugin",
      "options": { "agent": "reviewer", "strict": true }
    }
  ]
}
```

- Las rutas relativas se resuelven desde el archivo de configuración que contiene la entrada.
- Las rutas locales deben ser directorios; un archivo suelto se ignora.
- Los arrays de plugins de los configs aplicables se combinan de menor a mayor precedencia; no se reemplazan.
- También se descubren archivos `.ts` y `.js` y directorios de paquete inmediatos dentro de cada `.opencode/plugins/`.

## Gestión por CLI

```bash
opencode plugin add opencode-acme-plugin@1.2.0
opencode plugin list
opencode plugin list --builtin
opencode plugin check
opencode plugin update
opencode plugin update opencode-acme-plugin
opencode plugin remove opencode-acme-plugin@1.2.0
```

`plugin add` acepta nombres npm con versión, tag o rango, y paquetes Git compatibles con npm (GitHub, HTTPS o SSH, incluidas ramas, tags, commits completos y subdirectorios `::path:`). No acepta tarballs ni alias npm.

## Activar, desactivar y controlar

Las entradas se procesan en orden. Prefija un ID o un wildcard con `-` para desactivarlo; `*` afecta a todos y `.*` coincide con un prefijo:

```jsonc
{
  "plugins": ["*", "-opencode.provider.*", "opencode.provider.openai", "-acme.reviewer"]
}
```

Dos plugins internos ignoran las desactivaciones para que un repositorio no pueda saltarse las políticas: `opencode.config.policy` y `opencode.provider.opencode` (la conexión de Console que entrega políticas de organización).

## Recarga

Los cambios en directorios de configuración vigilados se recargan automáticamente. Al arrancar, el servidor carga los plugins de paquete en caché, instala los que faltan en segundo plano y comprueba actualizaciones de plugins npm/Git no fijados. Las versiones npm exactas y los commits Git completos quedan fijados.

Los cambios en dependencias locales no vigiladas pueden requerir reiniciar OpenCode:

```bash
touch .opencode/plugins/concise/index.ts
opencode service restart
```

## Escritorio y TUI

- Un plugin que exporta `./gui-experimental` añade paneles, comandos o ajustes a OpenCode Desktop. Se configura en el `opencode.json(c)` global para que se use en todas las ventanas.
- **Settings → Extensions → Plugins** lista los plugins cuando la app usa su propio servidor. El interruptor de un plugin solo apaga su UI de escritorio; para apagarlo entero, añade `-` y su ID a `plugins`.
- Los plugins solo TUI se configuran en `cli.json` bajo `plugins` y siguen activos aunque la TUI se conecte a un servidor remoto.

## Cuándo usar un plugin y cuándo no

- **Plugin**: necesitas una herramienta nueva, un hook en el ciclo de vida, una integración con un servicio, un comando/agente generado dinámicamente o UI.
- **Skill**: solo necesitas instrucciones y archivos de apoyo para una tarea.
- **Comando**: solo quieres una plantilla de prompt reutilizable.
- **MCP**: ya existe un servidor MCP que expone la capacidad que necesitas.

## Ver también

- [[opencode/extension/mcp|MCP]]
- [[opencode/extension/skills|Skills]]
- [[opencode/extension/comandos|Comandos personalizados]]
- [[opencode/extension/permisos-politicas|Permisos y políticas]]

Sources:
- [OpenCode Plugins](https://opencode.ai/docs/plugins/)
- [OpenCode Build: Plugins](https://opencode.ai/docs/build/plugins/)
- [OpenCode CLI Settings: Plugins](https://opencode.ai/docs/cli/config/#plugins)
