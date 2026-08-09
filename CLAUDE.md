# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Qué es este repositorio

ia-wiki es un vault de notas (Obsidian Flavored Markdown) publicado como sitio estático con [Quartz 5](https://quartz.jzhao.xyz) en GitHub Pages: https://developer-iperez.github.io/ia-wiki

Cada push a `main` dispara `.github/workflows/deploy.yml`, que compila el sitio y lo publica automáticamente.

## Dónde escribir

Todo el contenido del vault vive en `content/`. Es la única carpeta que debes crear, editar o reorganizar en el trabajo normal de sesión.

- Notas nuevas: crea un `.md` en `content/` (o subcarpeta temática) con frontmatter YAML mínimo:
  ```yaml
  ---
  title: Título de la nota
  ---
  ```
- Enlaces internos: usa wikilinks `[[Nombre de la nota]]` (la resolución de enlaces está fijada en `shortest`, como en Obsidian).
- **Toda nota nueva debe quedar referenciada desde `content/index.md`** (directamente, o desde otra nota ya enlazada desde la portada) con un wikilink, para que no quede huérfana ni invisible en el sitio publicado. Esto forma parte de la tarea, no un paso opcional aparte.
- Notas que no deben publicarse todavía: añade `draft: true` en el frontmatter, o colócalas en `content/private/` o `content/templates/` (excluidas del build por `ignorePatterns` en `quartz.config.yaml`).
- **Resúmenes de modelos LLM open source** van en `content/models/`, uno por modelo, y deben seguir la plantilla `content/templates/modelo-llm.md` (secciones: Highlights, Parámetros/contexto/tipo de inferencia, Resumen esquemático de benchmarks, Configuración recomendada).
- No crees ni edites una carpeta `.obsidian/` dentro de `content/` — si el usuario abre el vault en Obsidian local, esa carpeta es suya y ya está excluida del build.

## Qué NO tocar

El motor de Quartz y el despliegue no deben modificarse salvo que el usuario lo pida explícitamente:

- `quartz/`, `quartz.ts` — código del generador de sitio.
- `quartz.config.yaml` — configuración global (baseUrl, tema, plugins, ignorePatterns).
- `.github/workflows/deploy.yml` — pipeline de build y publicación.
- `.claude/settings.json` — permisos de la sesión.

## Comandos

```bash
npx quartz build --serve   # previsualizar el sitio en http://localhost:8080
npx quartz build           # compilar (el mismo comando que ejecuta el CI)
```

## Flujo de trabajo típico

1. Buscar/investigar el tema pedido (búsqueda web si el entorno lo permite).
2. Resumir con tus propias palabras — no copiar/pegar contenido extenso de fuentes externas.
3. Guardar como nota nueva (o actualizar una existente) en `content/`, siguiendo las convenciones de arriba.
4. Enlazar la nota desde `content/index.md` (o desde una nota ya alcanzable desde la portada).
5. `git add content/... && git commit`.
6. **Publica siempre directamente en `main`** (sin pedir confirmación previa ni abrir pull request, salvo que el usuario pida explícitamente lo contrario): `git push origin HEAD:main`. En cuanto el push llega a `main`, el Action publica el sitio en 1–2 minutos.
