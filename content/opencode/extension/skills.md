---
title: Skills en OpenCode
---

Una **skill** es un conjunto de instrucciones reutilizables para una tarea concreta. OpenCode la descubre, la anuncia al modelo cuando puede ser relevante y la carga solo cuando el agente la necesita. Es el mismo concepto que las [[claude/extension/skills|Skills de Claude]], con un formato muy parecido y algunas diferencias de descubrimiento.

## Estructura

La forma recomendada es una carpeta con un `SKILL.md` y, al lado, scripts, referencias y plantillas:

```text
.opencode/skills/
└── git-release/
    ├── SKILL.md
    ├── scripts/
    │   └── changelog.ts
    └── references/
        └── release-policy.md
```

```md title=".opencode/skills/git-release/SKILL.md"
---
name: Git Release
description: Prepara notas de release, subida de versión y publicación en GitHub
---

## Workflow

1. Lee `references/release-policy.md`.
2. Resume los cambios desde el tag anterior.
3. Propón la nueva versión antes de tocar archivos.
4. Ejecuta `scripts/changelog.ts` solo tras aprobar la versión.
```

Las rutas dentro de la skill son relativas al directorio que contiene `SKILL.md`. La forma de carpeta es preferible porque da un directorio base privado a los archivos de apoyo.

## Dónde se descubren

| Ámbito | Fuentes |
|---|---|
| Global | `~/.config/opencode/skills` |
| Compatibilidad global | `~/.claude/skills`, `~/.agents/skills` |
| Proyecto | `.opencode/skills` |
| Compatibilidad de proyecto | `.claude/skills`, `.agents/skills` |

Para las fuentes de proyecto, OpenCode busca desde el directorio actual hasta la raíz del proyecto e incluye los directorios que coinciden en cada nivel.

Formatos aceptados:

- Archivos `.md` en la raíz de la fuente, por ejemplo `skills/review.md`.
- Archivos llamados exactamente `SKILL.md` dentro de una carpeta, por ejemplo `skills/git-release/SKILL.md`.
- Una carpeta con `SKILL.md` es una skill; no se buscan más skills dentro.
- Se saltan archivos y carpetas ocultos y `node_modules`.
- La búsqueda se detiene seis directorios por debajo de la fuente.

## Frontmatter

Todos los campos son opcionales en tiempo de ejecución, pero una `description` clara es lo que permite que el modelo descubra la skill.

| Campo | Comportamiento |
|---|---|
| `name` | Nombre visible; por defecto, el ID derivado de la ruta. |
| `description` | Resumen que se muestra al modelo. Sin descripción, la skill no se anuncia. |
| `metadata.opencode/autoinvoke` | `false` la oculta de la lista del modelo; sigue registrada y se puede cargar por ID. |
| `disable-model-invocation` | `true` equivale a `opencode/autoinvoke: false`; si ambos están, gana `opencode/autoinvoke`. |

El **ID** lo determina la ruta, no el `name`:

| Archivo | ID |
|---|---|
| `<fuente>/git-release.md` | `git-release` |
| `<fuente>/git-release/SKILL.md` | `git-release` |
| `<fuente>/teams/release/SKILL.md` | `release` |

Los IDs distinguen mayúsculas de minúsculas. Para skills portables, usa minúsculas y kebab-case, alineado con el nombre del directorio.

## Carga

En cada paso del modelo, OpenCode lista las skills permitidas que tienen descripción y no están ocultas. La lista incluye solo ID, nombre y descripción, no el cuerpo completo.

El modelo carga una skill llamando a la herramienta `skill` con su ID exacto:

```json
{
  "id": "git-release"
}
```

OpenCode comprueba el permiso `skill` del agente, añade el cuerpo Markdown sin frontmatter y proporciona el directorio base y una muestra de hasta diez archivos de apoyo. Los contenidos de esos archivos no se cargan automáticamente; el agente los lee cuando la skill se lo indica.

Para cargarla manualmente, menciónala en el prompt como `@skill-id`.

## Permisos

```jsonc
{
  "permissions": [
    { "action": "skill", "resource": "*", "effect": "allow" },
    { "action": "skill", "resource": "internal-*", "effect": "deny" },
    { "action": "skill", "resource": "experimental-*", "effect": "ask" }
  ]
}
```

| Efecto | Comportamiento |
|---|---|
| `allow` | La anuncia y la carga sin aprobación. |
| `ask` | La anuncia y pide aprobación antes de cargarla. |
| `deny` | La oculta del modelo y rechaza la carga. |

Las reglas se pueden poner también bajo `agents.<id>.permissions` para un agente concreto.

## Precedencia

Si dos fuentes definen el mismo ID, gana la registrada más tarde:

1. Skills integradas.
2. `.claude/skills` (global y ancestros).
3. `.agents/skills` (global y ancestros).
4. `~/.config/opencode/skills`.
5. `.opencode/skills` del proyecto, de la raíz hacia el directorio actual.
6. Entradas explícitas de `skills` en configuración, por prioridad y orden del array.

Evita IDs duplicados salvo que quieras sobrescribir una skill anterior.

## Fuentes adicionales

```jsonc
{
  "skills": [
    "./team-skills",
    "~/shared/opencode-skills",
    "/opt/company-skills",
    "https://example.com/opencode/skills/"
  ]
}
```

- Las rutas relativas se resuelven desde el directorio de trabajo de OpenCode, no desde el archivo de configuración.
- Las URLs se cargan como catálogo HTTP con un `index.json`.
- Las entradas de todos los archivos de configuración se combinan, no se reemplazan.

## Diferencias con Claude Skills

- OpenCode lee también `.claude/skills` y `.agents/skills` como compatibilidad, además de sus rutas nativas.
- El ID sale de la ruta, no del `name`.
- OpenCode no impone por ahora el límite de 64 caracteres, la coincidencia entre nombre y directorio ni un máximo de descripción.
- La skill se carga con la herramienta `skill`, no inyectando todo el cuerpo en cada prompt.
- Los archivos de apoyo se referencian por ruta relativa; no se cargan solos.

## Ver también

- [[opencode/extension/agentes|Agentes y subagentes]]
- [[opencode/extension/comandos|Comandos personalizados]]
- [[opencode/extension/permisos-politicas|Permisos y políticas]]
- [[opencode/guias/desde-claude-code|Desde Claude Code]]

Sources:
- [OpenCode Skills](https://opencode.ai/docs/skills/)
- [OpenCode Permissions](https://opencode.ai/docs/permissions/)
