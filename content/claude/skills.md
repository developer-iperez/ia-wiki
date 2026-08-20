---
title: Skills de Claude
---

Una **Skill** es un módulo de capacidad instalable que amplía lo que [[claude/claude-code|Claude Code]] (o Claude en general) sabe hacer por defecto. En vez de repetir las mismas instrucciones en cada conversación, se empaquetan una única vez en una carpeta y Claude las aplica automáticamente cuando detecta que vienen al caso.

La analogía útil: si un prompt suelto es una nota adhesiva que se escribe y se tira, una skill es un procedimiento documentado — con versión y responsable — que estandariza cómo se hace siempre una tarea concreta.

## Estructura de una skill

Una skill es una carpeta que contiene, como mínimo, un fichero `SKILL.md` con:

- **Frontmatter YAML** (metadatos) al principio del fichero, entre `---`.
- **Cuerpo en Markdown** debajo, con las instrucciones propiamente dichas.

### Campos del frontmatter

Solo `name` y `description` son obligatorios:

- `name` — identificador de la skill; solo minúsculas, números y guiones, máx. 64 caracteres.
- `description` — qué hace la skill **y** cuándo debe activarse (qué frases o situaciones la disparan). Es el campo más importante: Claude decide si usar la skill leyendo esta descripción, así que cuanto más específica y accionable, mejor. Máx. 1024 caracteres.
- `allowed-tools` (opcional) — lista de herramientas pre-aprobadas que la skill puede usar, en formato `ToolName(patrón)`, p. ej. `Bash(curl:*) Read Write`.
- `license`, `compatibility`, `metadata` (opcionales) — autor, versión, dependencias (p. ej. un servidor MCP concreto), etc.

Ejemplo mínimo:

```yaml
---
name: customer-onboarding
description: >
  Gestiona el alta completa de un cliente: creación de cuenta, cobro
  y email de bienvenida. Úsala cuando el usuario diga "dar de alta un
  cliente" o "crear cuenta nueva".
allowed-tools: Bash(curl:*) Read Write
---
# Instrucciones de la skill...
```

## Dónde viven

- **Personales**: `~/.claude/skills/` — disponibles en todos los proyectos.
- **De proyecto**: `.claude/skills/` dentro del repositorio — se comparten con quien clone el repo (como el resto de la configuración de `.claude/`).

## Cómo se activan

No hace falta invocarlas a mano: Claude las detecta automáticamente comparando la petición del usuario con el campo `description` de cada skill instalada, y las carga solo cuando encajan. Esto evita cargar contexto innecesario en tareas que no la necesitan.

## Buenas prácticas al escribirlas

- La `description` debe cubrir tanto la **capacidad** ("qué hace") como el **disparador** ("cuándo usarla") — sin esto, Claude no sabrá cuándo activarla.
- Instrucciones específicas y accionables, no vaguedades genéricas.
- Una skill, una responsabilidad clara — igual que una función bien diseñada.

## Ver también

- [[claude/comandos|Comandos útiles de Claude Code]]
- [[claude/subagentes|Subagentes de Claude Code]] — mismo formato de fichero (Markdown + frontmatter), pero un subagente abre una sesión aislada aparte en vez de añadir contexto a la actual.

Sources:
- [Cómo Crear Claude Skills: Guía Completa (2026)](https://www.bleap.finance/es/blog/como-crear-claude-skills)
- [Skills de Claude Code: guía completa y las mejores de 2026](https://www.albertopampin.es/blog/skills-de-claude-code/)
- [Agent Skills Cheat Sheet (2026) - SKILL.md Format & Authoring Reference | Webfuse](https://www.webfuse.com/agent-skills-cheat-sheet)
- [The SKILL.md Frontmatter Reference — Introduction to Agent Skills](https://www.anthropiccertifications.com/courses/introduction-to-agent-skills/skill-frontmatter-reference)
