---
title: Flujo de trabajo diario con OpenCode
---

Una rutina práctica para usar OpenCode como agente de código, desde la exploración hasta la verificación y el commit. Está pensada para la TUI, pero casi todo aplica también a `opencode run` y a la interfaz web/escritorio.

## 1. Preparar el proyecto

Antes de la primera sesión:

- Crea un `AGENTS.md` con comandos de build/test, arquitectura y convenciones (ver [[opencode/fundamentos/instrucciones|Instrucciones y AGENTS.md]]).
- Define permisos en `opencode.jsonc`: pregunta antes de shell, permite `git status`/`git diff`, deniega `git push` y protege `.env`.
- Configura un par de comandos y skills para las tareas repetitivas.
- Si usas MCP, añade solo los servidores que necesites: sus herramientas consumen contexto.

Ejemplo de base de seguridad:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "opencode-go/kimi-k3",
  "permissions": [
    { "action": "shell", "resource": "*", "effect": "ask" },
    { "action": "shell", "resource": "git status *", "effect": "allow" },
    { "action": "shell", "resource": "git diff *", "effect": "allow" },
    { "action": "shell", "resource": "git push *", "effect": "deny" },
    { "action": "read", "resource": "*.env", "effect": "ask" },
    { "action": "read", "resource": "*.env.example", "effect": "allow" }
  ]
}
```

## 2. Explorar antes de tocar código

Usa el agente **Plan** para entender el problema y proponer un plan sin editar archivos normales. En la TUI: **Ctrl+X A** → `plan`, o `/agents`.

Buenos prompts de exploración:

```text
Map the authentication flow and list the files involved. Do not edit anything.
Explain how the retry logic works and where the timeout is configured.
Find the tests that cover this module and tell me what is missing.
```

Para investigación en paralelo, pide un subagente:

```text
Use the explore subagent to map the payment module and return the key files.
```

## 3. Ejecutar el cambio

Vuelve a **Build** y sé explícito con el resultado esperado, no solo con el archivo:

```text
Change the default request timeout to 30 seconds, update the tests, and run the relevant test suite.
```

- `@archivo` adjunta contexto; `@archivo#20-45` adjunta un rango.
- `!comando` ejecuta shell y deja la salida en la sesión.
- Los comandos personalizados (`/review`, `/test`, etc.) reducen prompts repetidos.
- Si el agente se desvía, **Enter** dirige la sesión activa; **Ctrl+X Enter** encola una instrucción para después.

## 4. Revisar y verificar

- Abre el diff con `/diff` y revisa archivo por archivo.
- Usa un subagente de revisión de solo lectura:

```text
Use the reviewer subagent to review the current diff for correctness, security, and missing tests.
```

- Ejecuta los tests y el linter con `!` o pide al agente que los ejecute.
- No aceptes un cambio solo porque el agente dice que funciona: comprueba la salida real.

## 5. Deshacer si hace falta

OpenCode captura snapshots del filesystem antes y después de cada paso del modelo. Si un cambio no te convence:

- `/undo` (o **Ctrl+X U**) revierte el último mensaje y el trabajo que le siguió; el prompt vuelve al composer.
- `/redo` (o **Ctrl+X R**) cancela el revert.
- Enviar un prompt nuevo acepta el revert y continúa desde ahí.

Los snapshots cubren el directorio activo dentro de un repo Git, incluidos archivos no trackeados no ignorados de hasta 2 MiB. **No** cubren archivos ignorados, efectos de shell, commits, ramas ni bases de datos. Revisa siempre `git diff` antes de continuar y no los uses como copia de seguridad.

## 6. Gestionar el contexto

Cuando la sesión se acerca al límite del modelo:

- La compactación automática está activada por defecto: resume lo antiguo y conserva ~15.000 tokens recientes.
- Puedes compactar a mano con **Ctrl+X C** (`session.compact`) o con la API:

```bash
opencode api post /api/session/ses_example/compact --data '{}'
```

- Si necesitas más detalle reciente, sube `compaction.keep.tokens`. Si quieres compactar antes, sube `compaction.buffer` (por ejemplo `"20%"`).
- Para trabajo muy largo, `experimental.background_compaction` escribe el resumen en segundo plano, pero es experimental.

## 7. Sesiones y pestañas

- **Ctrl+X N** crea una sesión; **Ctrl+X L** o `/sessions` vuelve a una existente.
- **Ctrl+O** abre sesiones y proyectos recientes.
- Las pestañas permiten tener varias sesiones abiertas; **Ctrl+Tab** cambia y **Ctrl+X W** cierra.
- `opencode session export <id>` guarda una sesión; `--sanitize` redacta datos sensibles.
- `opencode session import <fichero>` la recupera en otra máquina o proyecto.

## 8. Automatizar

Para CI o scripts, usa `opencode run` en modo standalone:

```bash
opencode run --standalone --model anthropic/claude-sonnet-5-5 \
  "Review this repository for correctness and summarize any issues."
```

Opciones útiles: `--format json` para parsear, `--file` para adjuntar, `--agent` para elegir agente, `--continue` para seguir la última sesión, `--auto` para aprobar lo no denegado y `--archived` para no ensuciar la lista de sesiones.

## 9. Antes de commitear

1. Revisa `git diff` y el estado de los archivos.
2. Ejecuta tests, linter y typecheck.
3. Comprueba que no se han colado secretos, archivos generados o cambios fuera del alcance.
4. Pide al agente un mensaje de commit si lo necesitas, pero revisa el contenido.
5. Haz commit y push tú mismo si tienes `git push` denegado en permisos; es la opción recomendada.

## Ver también

- [[opencode/fundamentos/cli-tui|CLI y TUI]]
- [[opencode/extension/permisos-politicas|Permisos y políticas]]
- [[opencode/extension/agentes|Agentes y subagentes]]
- [[opencode/extension/comandos|Comandos personalizados]]
- [[opencode/fundamentos/configuracion|Configuración]]

Sources:
- [OpenCode TUI](https://opencode.ai/docs/cli/tui/)
- [OpenCode Compaction](https://opencode.ai/docs/compaction/)
- [OpenCode Snapshots](https://opencode.ai/docs/snapshots/)
- [OpenCode CLI Commands](https://opencode.ai/docs/cli/commands/)
- [OpenCode Permissions](https://opencode.ai/docs/permissions/)
