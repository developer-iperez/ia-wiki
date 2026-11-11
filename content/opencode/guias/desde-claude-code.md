---
title: De Claude Code a OpenCode
---

Si vienes de [[claude/fundamentos/claude-code|Claude Code]], OpenCode resulta familiar: mismo modelo mental de agente que lee el repo, edita archivos, ejecuta comandos y verifica. Pero los nombres, las rutas y algunas piezas cambian. Esta guía resume las equivalencias y las diferencias que más importan al migrar.

## Tabla de equivalencias

| Claude Code | OpenCode V2 | Notas |
|---|---|---|
| `CLAUDE.md` | `AGENTS.md` | OpenCode V2 **no lee `CLAUDE.md` como fallback**. |
| `~/.claude/skills/`, `.claude/skills/` | `.opencode/skills/` o `~/.config/opencode/skills/` | OpenCode también descubre `.claude/skills` y `.agents/skills` por compatibilidad. |
| Subagentes en `.claude/agents/` | `.opencode/agents/<nombre>.md` con `mode: subagent` | No hay descubrimiento automático de agentes de Claude; se migran a mano. |
| Comandos en `.claude/commands/` | `.opencode/commands/<nombre>.md` | Migración manual; el formato es plantilla Markdown. |
| `hooks` de Claude Code | Plugins de OpenCode (parte servidor) | Los hooks no tienen un fichero equivalente 1:1; se implementan como plugin. |
| `.claude/settings.json` (permisos) | `opencode.json(c)` → `permissions` + `policies` | `permissions` decide `allow`/`ask`/`deny`; `policies` hace bloqueos duros. |
| Servidores MCP de Claude | `mcp.servers` en `opencode.json(c)` | Se reconfiguran; usa `opencode mcp add`. |
| `/agents` | `/agents` o **Ctrl+X A** | Equivalente directo. |
| `/mcp` | `/mcps` o `opencode mcp list` | En V2 los servidores van bajo `mcp.servers`. |
| `/hooks` | `opencode plugin list` / gestor de plugins | No hay listado de hooks como en Claude. |
| `/compact` | **Ctrl+X C** (`session.compact`) o API | La compactación es automática por defecto; el atajo fuerza una manual. |
| `/clear` | `/new` o **Ctrl+X N** | OpenCode separa por sesiones. |
| `/cost` | `opencode stats` | `--days`, `--models`, `--cost`, `--json`. |
| `/permissions` | Editar `permissions`/`policies` o responder a los prompts | No hay un comando interactivo equivalente documentado. |
| `/memory` | Editar `AGENTS.md` | No hay comando de memoria equivalente documentado. |
| Claude Code Cloud | Web (`opencode pair`), escritorio, `opencode serve` | La arquitectura cliente-servidor permite conexión remota. |

## Diferencias de fondo

### 1. `AGENTS.md`, no `CLAUDE.md`

Es el cambio que más rompe al principio. OpenCode V2 reconoce `AGENTS.md` únicamente. Si quieres reutilizar el contenido de `CLAUDE.md`, tienes dos opciones:

- Copiar o renombrar el contenido a `AGENTS.md` (lo nativo).
- Declarar `CLAUDE.md` en `instructions` si quieres mantenerlo como fuente:

```jsonc
{
  "instructions": ["CLAUDE.md"]
}
```

Pero lo recomendable es que `AGENTS.md` sea la fuente en OpenCode. Claude Code se documenta con `CLAUDE.md`; si conviven ambos agentes en el repo, mantén los dos sincronizados o enlázalos para no duplicar instrucciones.

### 2. Arquitectura cliente-servidor

OpenCode arranca un servidor compartido que posee sesiones, configuración, permisos y herramientas. La TUI, la web, el escritorio y `opencode run` son clientes. Esto permite:

- Lanzar una tarea desde la web o el móvil y verla en la TUI.
- Conectar varias interfaces al mismo estado.
- Aislar con `--standalone` o apuntar a otro servidor con `--server`.

En Claude Code el proceso y la sesión están más acoplados a la terminal.

### 3. Permisos y políticas

Claude Code usa `settings.json` con allow/deny/ask por herramienta. OpenCode usa reglas ordenadas:

- **Última regla que coincide gana**.
- `permissions` puede `allow`, `ask` o `deny`.
- `policies` solo puede `allow` o `deny`, nunca pregunta y solo endurece.
- Las políticas de sistema/Console/MDM pueden impedir que un repo reactive algo denegado.

Esto da más control, pero exige pensar el orden: regla amplia primero, excepciones después.

### 4. Skills compatibles, agentes y comandos no

OpenCode lee `.claude/skills` y `.agents/skills` además de sus rutas nativas, así que muchas skills de Claude funcionan sin moverlas. En cambio:

- Los agentes de `.claude/agents/` no se descubren automáticamente.
- Los comandos de `.claude/commands/` no se descubren automáticamente.
- Los hooks no se traducen solos; hay que reimplementarlos como plugin.

### 5. Plugins en vez de hooks

Los hooks de Claude son scripts en el ciclo de vida. En OpenCode el equivalente es un **plugin de servidor**, que puede engancharse a eventos, añadir herramientas, comandos, agentes o integraciones. Es más potente, pero también más código.

Si solo necesitas automatizar un comando antes/después de una tarea, empieza por un comando con bloque de shell o por CI; si necesitas interceptar el ciclo del agente, usa un plugin.

## Checklist de migración

1. **Crea `AGENTS.md`** con el contenido útil de `CLAUDE.md` (build, tests, arquitectura, convenciones).
2. **Mueve o copia skills** a `.opencode/skills/`. Si quieres probar rápido, deja las de `.claude/skills/`: OpenCode las descubre.
3. **Migra agentes** a `.opencode/agents/*.md`, con `mode: subagent` para los que no deban ser primarios.
4. **Migra comandos** a `.opencode/commands/*.md`; sustituye `$ARGUMENTS`/posicionales según corresponda.
5. **Traduce permisos** de `settings.json` a `permissions` y, para bloqueos que no se puedan saltar, a `policies`.
6. **Reconfigura MCP** con `opencode mcp add` y revisa OAuth/credenciales.
7. **Reimplementa hooks** como plugins solo si de verdad los necesitas.
8. **Prueba en aislado** con `opencode --standalone` o `opencode run` antes de cambiar tu flujo principal.
9. **Ajusta `cli.json`** para tema, atajos y pestañas; no lo mezcles con `opencode.jsonc`.
10. **Revisa el modelo y proveedor**: `/connect`, `/models`, y si usas OpenCode Go, `opencode-go/<model-id>`.

## Qué no esperar

- No hay un importador automático de configuración de Claude Code a OpenCode.
- OpenCode no lee `.claude/settings.json` ni `.claude/agents` automáticamente.
- Los hooks de Claude no se ejecutan en OpenCode.
- Las rutas de skills son compatibles, pero el ID sale de la ruta; revisa que no haya colisiones.

## Ver también

- [[opencode/fundamentos/instrucciones|Instrucciones y AGENTS.md]]
- [[opencode/extension/skills|Skills]]
- [[opencode/extension/agentes|Agentes y subagentes]]
- [[opencode/extension/permisos-politicas|Permisos y políticas]]
- [[opencode/extension/plugins|Plugins]]
- [[opencode/extension/mcp|MCP]]
- [[claude/index|Claude]]

Sources:
- [OpenCode Instructions](https://opencode.ai/docs/instructions/)
- [OpenCode Skills](https://opencode.ai/docs/skills/)
- [OpenCode Agents](https://opencode.ai/docs/agents/)
- [OpenCode Permissions](https://opencode.ai/docs/permissions/)
- [OpenCode Policies](https://opencode.ai/docs/policies/)
- [OpenCode Plugins](https://opencode.ai/docs/plugins/)
- [OpenCode MCP Servers](https://opencode.ai/docs/mcp-servers/)
