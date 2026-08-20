---
title: MCP (Model Context Protocol)
---

MCP es el estándar abierto creado por Anthropic (noviembre de 2024) que permite a un modelo de IA conectarse con herramientas externas, bases de datos y APIs mediante una interfaz común, en vez de necesitar una integración a medida para cada servicio. La analogía habitual: MCP es "el USB-C de la IA" — un conector universal. En diciembre de 2025 Anthropic donó el protocolo a la Linux Foundation, consolidándolo como estándar de facto; en 2026 lo usan Claude, ChatGPT, Gemini, Cursor, GitHub Copilot y VS Code, con más de 10.000 servidores públicos activos.

## Por qué importa

Sin MCP, conectar un agente como [[claude/claude-code|Claude Code]] a Jira, una base de datos o Figma implicaba copiar y pegar datos manualmente en la conversación. Con un servidor MCP conectado, el agente puede leer y actuar directamente sobre ese sistema:

- "Implementa la funcionalidad del ticket JIRA ENG-4521 y crea una PR en GitHub."
- "Consulta Sentry para ver el impacto del último despliegue."
- "Busca en la base de datos PostgreSQL los usuarios que no han comprado en 90 días."

## Arquitectura

Tres piezas, comunicadas con JSON-RPC 2.0:

- **Host**: la aplicación que aloja al LLM (p. ej. Claude Code).
- **Client**: el componente, dentro del host, que gestiona la conexión con cada servidor.
- **Server**: el proceso que expone las capacidades de una herramienta concreta (GitHub, una base de datos, un sistema de ficheros...).

El transporte puede ser:

- **stdio** — el servidor corre como proceso local en tu máquina (ideal para scripts propios o acceso directo al sistema de ficheros).
- **HTTP** (`streamable-http`) — el recomendado para servidores remotos/en la nube.
- **SSE** — igual que HTTP pero **deprecado**; úsese HTTP si el servidor lo soporta.
- **WebSocket** — para servidores remotos que necesitan enviar eventos al agente sin que este pregunte primero.

## Las tres primitivas

Un servidor MCP puede exponer:

1. **Tools** — funciones que el modelo puede invocar (con un nombre y un JSON Schema de entrada). Ejemplo: una tool `create_issue(title, body)` que crea un ticket.
2. **Resources** — datos de solo lectura que el modelo puede consultar como contexto, similar a un endpoint `GET` (p. ej. el contenido de un fichero o el esquema de una tabla).
3. **Prompts** — plantillas de instrucciones predefinidas que guían al modelo para una tarea concreta del servidor.

## Conectar un servidor MCP a Claude Code

```bash
# Servidor remoto HTTP (recomendado para servicios en la nube)
claude mcp add --transport http notion https://mcp.notion.com/mcp

# Con autenticación por token
claude mcp add --transport http github https://api.githubcopilot.com/mcp/ \
  --header "Authorization: Bearer TU_TOKEN"

# Servidor local (stdio) — todo lo que va tras "--" se pasa tal cual al comando
claude mcp add --env AIRTABLE_API_KEY=TU_CLAVE --transport stdio airtable \
  -- npx -y airtable-mcp-server
```

Gestión:

```bash
claude mcp list          # listar servidores configurados y su estado
claude mcp get notion    # detalle de un servidor
claude mcp remove notion # quitarlo
/mcp                     # (dentro de una sesión) ver estado y autenticar por OAuth si hace falta
```

### Ámbitos (`--scope`)

| Ámbito | Dónde se carga | Se comparte con el equipo | Se guarda en |
|---|---|---|---|
| `local` (por defecto) | solo en el proyecto actual | No | `~/.claude.json` |
| `project` | solo en el proyecto actual | Sí, vía `.mcp.json` en el repo | `.mcp.json` |
| `user` | en todos tus proyectos | No | `~/.claude.json` |

Para compartir un servidor con quien clone el repo, se añade con `--scope project`, lo que genera un `.mcp.json` en la raíz — Claude Code pide aprobación antes de usar servidores de ese fichero la primera vez, por seguridad.

### Ejemplo de `.mcp.json`

```json
{
  "mcpServers": {
    "shared-server": {
      "type": "http",
      "url": "https://example.com/mcp"
    },
    "db": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@bytebase/dbhub", "--dsn", "${DATABASE_URL}"]
    }
  }
}
```

Admite expansión de variables de entorno (`${VAR}` o `${VAR:-valor_por_defecto}`), útil para no meter claves en el repo.

## Ejemplo práctico: consultar una base de datos

```bash
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://readonly:pass@prod.db.com:5432/analytics"
```

(usar siempre un usuario de solo lectura en el connection string). Una vez conectado, basta con preguntar en lenguaje natural:

> "¿Cuál es la facturación total de este mes?"
> "Muéstrame el esquema de la tabla orders."

## Buenas prácticas y seguridad

- Verifica que confías en un servidor antes de conectarlo — uno que recupera contenido externo (una página web, un email) puede exponerte a **prompt injection**.
- Usa credenciales de solo lectura cuando el servidor pueda evitarlo (p. ej. bases de datos).
- Los servidores de `.mcp.json` (ámbito `project`) requieren aprobación explícita antes de usarse, precisamente para que clonar un repo ajeno no ejecute servidores sin tu consentimiento.

## Dónde encontrar o crear servidores

- Directorio de conectores revisados por Anthropic: [claude.ai/directory](https://claude.ai/directory).
- Guía oficial para construir un servidor propio: [modelcontextprotocol.io/docs/develop/build-server](https://modelcontextprotocol.io/docs/develop/build-server).
- Documentación completa del protocolo: [modelcontextprotocol.io](https://modelcontextprotocol.io/introduction).
- Referencia de MCP en Claude Code: [code.claude.com/docs/en/mcp](https://code.claude.com/docs/en/mcp).

## Ver también

- [[claude/claude-code|Claude Code]] — el agente donde se configuran y usan estos servidores.
- [[claude/comandos|Comandos útiles de Claude Code]] — incluye `/mcp`, el comando para gestionar servidores dentro de una sesión.

Sources:
- [MCP (Model Context Protocol): Que Es, Como Funciona y Por Que Es el USB-C de la IA [Guia 2026] | Javadex](https://www.javadex.es/blog/mcp-model-context-protocol-guia-completa-2026)
- [Qué es MCP (Model Context Protocol) y por qué en 2026 es una decisión de infraestructura](https://blog.shakersworks.com/que-es-mcp-model-context-protocol)
- [Connect Claude Code to tools via MCP — Claude Docs](https://code.claude.com/docs/en/mcp)
- [Model Context Protocol — sitio oficial](https://modelcontextprotocol.io/introduction)
