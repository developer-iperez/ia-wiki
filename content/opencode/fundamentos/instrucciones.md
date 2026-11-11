---
title: Instrucciones y AGENTS.md
---

OpenCode usa **`AGENTS.md`** como mecanismo nativo de instrucciones persistentes del proyecto. Es el equivalente funcional a `CLAUDE.md` en Claude Code, pero con una diferencia importante: **OpenCode V2 no lee `CLAUDE.md` como fallback**. Si vienes de Claude Code, tienes que renombrarlo, copiarlo o declararlo explícitamente.

## Qué poner en `AGENTS.md`

Instrucciones que deban aplicarse siempre en ese ámbito:

- Comandos de build, test, lint y typecheck.
- Arquitectura y convenciones del repositorio.
- Dónde vive cada tipo de código.
- Requisitos de verificación antes de dar una tarea por terminada.
- Cosas que no se deben tocar (generados, vendor, migraciones, etc.).

```md title="AGENTS.md"
# Project instructions

- Run `bun typecheck` after changing TypeScript.
- Keep database queries in `src/database`.
- Do not edit generated files directly.
- Add a test for every bug fix.
```

## Dónde vive

| Ámbito | Ruta |
|---|---|
| Global | `~/.config/opencode/AGENTS.md` |
| Proyecto | `AGENTS.md` en la raíz del proyecto |
| Subproyecto | `packages/web/AGENTS.md`, etc. |

OpenCode carga el archivo global y luego todos los `AGENTS.md` desde el directorio actual hacia el home (o hasta la raíz del proyecto si el workspace está fuera del home). Los combina en este orden:

```text
~/.config/opencode/AGENTS.md
packages/web/AGENTS.md
packages/AGENTS.md
AGENTS.md
```

No resuelve conflictos: si dos archivos dicen cosas distintas, ambas llegan al modelo. Mantén lo global para lo que aplica a todos los proyectos y lo específico cerca del código.

## Descubrimiento perezoso

Los `AGENTS.md` que están por debajo del workspace se cargan cuando el agente lee o lista esa zona:

```text
mi-proyecto/                ← workspace actual
├── AGENTS.md               cargado al inicio
└── packages/web/
    ├── AGENTS.md           se carga al leer esta zona
    └── src/app.ts
```

Se cargan de cerca a lejos y se deduplican. Si editas un `AGENTS.md` anidado después de que se haya cargado, el cambio no se detecta automáticamente: abre una sesión nueva si necesitas que se aplique ya.

## Actualizaciones en caliente

Los archivos globales y los que están hacia arriba sí se actualizan mientras la sesión está viva. Antes de la siguiente petición al modelo, OpenCode detecta el cambio y añade una actualización de instrucciones. Si borras todos los `AGENTS.md` ambientales, la sesión entiende que las instrucciones anteriores ya no aplican.

## Instrucciones adicionales

La configuración `instructions` permite añadir archivos, globs o URLs:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "CONTRIBUTING.md",
    "docs/guidelines/*.md",
    "~/rules/opencode.md",
    "https://example.com/instructions.md"
  ]
}
```

- Las rutas relativas se buscan desde el workspace hasta la raíz del proyecto.
- `~/` se resuelve desde el home.
- Las URLs se descargan al cargar la configuración, con timeout de 5 segundos.
- Se combinan con los `AGENTS.md`; no los sustituyen.
- Con `OPENCODE_DISABLE_PROJECT_CONFIG=1`, se omite el descubrimiento de `AGENTS.md` de proyecto, pero no el global.

## Orden de ensamblado

El prompt final se construye aproximadamente así:

1. System prompt del agente o proveedor.
2. Guía de herramientas de Code Mode, si está activo.
3. Guía de MCP, referencias y skills.
4. `AGENTS.md` globales y de proyecto, y las `instructions` configuradas.
5. Contexto integrado de fecha y entorno.
6. Instrucciones específicas de la sesión enviadas por la API.

## Convivencia con Claude Code

Si mantienes los dos agentes en el mismo repo:

- Lo nativo en OpenCode es `AGENTS.md`; lo nativo en Claude Code es `CLAUDE.md`.
- Puedes declarar `CLAUDE.md` en `instructions` para reutilizar su contenido en OpenCode, pero sigue siendo un archivo de Claude y no se actualiza solo en el lado de OpenCode.
- Para skills, OpenCode sí lee `.claude/skills` y `.agents/skills` como rutas de compatibilidad (ver [[opencode/extension/skills|Skills]]).
- Para agentes, comandos y permisos, la migración es manual; tienes una tabla de equivalencias en [[opencode/guias/desde-claude-code|Desde Claude Code]].

## Ver también

- [[opencode/fundamentos/configuracion|Configuración]]
- [[opencode/guias/desde-claude-code|Desde Claude Code]]
- [[opencode/extension/agentes|Agentes y subagentes]]

Sources:
- [OpenCode Instructions](https://opencode.ai/docs/instructions/)
- [OpenCode Config](https://opencode.ai/docs/config/)
