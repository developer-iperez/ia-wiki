---
title: Modelos y proveedores en OpenCode
---

OpenCode es agnóstico de proveedor. El catálogo de modelos viene de [models.dev](https://models.dev), de las integraciones de proveedor y de tu propia configuración. Para elegir modelo:

```text
/models
```

El selector solo muestra modelos habilitados cuyo proveedor esté disponible en el proyecto actual. La elección afecta a la sesión, no reescribe la configuración.

## Referencias de modelo

Formato: `proveedor/modelo` con variante opcional `#variante`.

```text
anthropic/claude-sonnet-5-5
openai/gpt-6.1-sol#high
openrouter/anthropic/claude-sonnet-5.5#high
opencode-go/kimi-k3
opencode/gpt-6.1-sol
ollama/gemma3:4b
lmstudio/google/gemma-4-26b-a4b
vllm/Qwen/Qwen3-Coder-30B-A3B-Instruct
```

- El proveedor termina en el primer `/`.
- El modelo puede contener más `/`.
- La variante va después de `#` y suele representar esfuerzo de razonamiento o presupuesto de tokens.
- Los IDs distinguen mayúsculas de minúsculas.

## Modelo por defecto

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "opencode-go/kimi-k3"
}
```

- La selección de una sesión manda sobre el valor por defecto.
- `opencode run --model ...` elige modelo solo para esa ejecución.
- Agentes y comandos pueden fijar su propio `model`.
- El `model` raíz no conserva la variante; para variantes, selecciónala en sesión, agente o comando.

## Proveedores

Para la mayoría de proveedores: `/connect`, elegir proveedor, pegar credencial y luego `/models`.

Proveedores integrados habituales: Anthropic, OpenAI (incluido “Sign in with ChatGPT”), Google, Google Vertex, Azure, Amazon Bedrock, GitHub Copilot, OpenRouter, xAI, Ollama, LM Studio y vLLM. La lista exacta y los campos de cada uno están en la [guía de proveedores](https://opencode.ai/docs/providers/).

### Proveedor personalizado compatible con OpenAI

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "providers": {
    "acme": {
      "name": "Acme",
      "env": ["ACME_API_KEY"],
      "package": "@opencode/ai/providers/openai-compatible",
      "settings": {
        "baseURL": "https://llm.acme.example/v1"
      },
      "models": {
        "qwen3-coder": {
          "name": "Qwen 3 Coder"
        }
      }
    }
  }
}
```

Campos útiles del proveedor: `name`, `env`, `package`, `canonical`, `compatibility`, `settings`, `headers`, `body` y `models`. Cada modelo puede sobrescribir `modelID`, `name`, `capabilities`, `limit`, `cost`, `variants`, `headers`, `body`, `compatibility` y `disabled`.

### Endpoint, headers y timeouts

```jsonc
{
  "providers": {
    "anthropic": {
      "settings": {
        "baseURL": "https://llm-proxy.example.com/anthropic",
        "headerTimeout": 600000,
        "chunkTimeout": false
      },
      "headers": { "X-Team": "platform" },
      "body": { "metadata": { "application": "opencode" } }
    }
  }
}
```

- `headerTimeout` y `chunkTimeout` van en milisegundos; `false` los desactiva.
- `timeout` limita la petición completa.
- `settings` es específico del paquete del proveedor; un campo solo funciona si ese paquete lo soporta.

## Modelos locales

OpenCode descubre automáticamente Ollama, LM Studio y vLLM en sus direcciones por defecto:

- Ollama: `http://127.0.0.1:11434`
- LM Studio: `http://127.0.0.1:1234/v1`
- vLLM: `http://127.0.0.1:8000/v1`

```jsonc
{
  "model": "ollama/gemma3:4b"
}
```

Para otro host o puerto, cambia `settings.baseURL`. En vLLM, el descubrimiento no puede inferir tool support: los modelos descubiertos empiezan con tools desactivadas, aunque el servidor las tenga con `--enable-auto-tool-choice` y `--tool-call-parser`.

Para cualquier otro servidor compatible con OpenAI, define un proveedor con `package: "@opencode/ai/providers/openai-compatible"`, `baseURL` y al menos un modelo con `modelID`, `capabilities` y `limit` reales.

## OpenCode Console y Go

Son dos servicios opcionales de OpenCode; no son necesarios para usar OpenCode con otros proveedores.

### Console

[OpenCode Console](https://opencode.ai/console) es una pasarela de pago por uso con modelos probados por el equipo de OpenCode. Se conecta como un proveedor más:

1. Crear cuenta, añadir crédito y copiar la API key.
2. `/connect` → proveedor de pago de OpenCode.
3. `/models` para elegir.

Los modelos se referencian como `opencode/<model-id>`, por ejemplo `opencode/gpt-6.1-sol`. Hay modelos gratuitos por tiempo limitado; se pueden ocultar con `OPENCODE_DISABLE_FREE_MODELS=1`.

### Go

[OpenCode Go](https://opencode.ai/console/go) es una suscripción opcional para modelos de coding:

| Plan | Precio | Uso incluido |
|---|---:|---|
| Go | 10 $/mes | Acceso de bajo coste a los modelos de Go |
| Go Plus | 40 $/mes | Límites más altos en los mismos modelos |
| Go Teams | 20 $/miembro/mes | Cada miembro tiene sus propios límites, 5 $/mes más que Go por modelo |

Se conecta con `/connect` → **OpenCode Go** y se selecciona con `/models`. En configuración, el modelo se referencia como `opencode-go/<model-id>`:

```jsonc
{
  "model": "opencode-go/kimi-k3"
}
```

Los límites son mensuales en dólares por modelo; cada modelo tiene además ventanas de 5 horas (20 %) y semanal (50 %). Al agotar el límite se puede seguir con los modelos gratuitos o activar *Extra Usage* con saldo de Console. Los modelos y precios cambian con frecuencia; consulta la [página oficial de Go](https://opencode.ai/docs/console/go/).

## Variantes y razonamiento

Las variantes agrupan ajustes de un mismo modelo, normalmente esfuerzo de razonamiento:

```bash
opencode run --model openai/gpt-6.1-sol#high "Review this migration plan"
```

También se pueden definir variantes propias:

```jsonc
{
  "providers": {
    "openai": {
      "models": {
        "gpt-6.1-sol": {
          "variants": [
            { "id": "fast", "settings": { "reasoningEffort": "low" } },
            { "id": "deep", "settings": { "reasoningEffort": "high", "reasoningSummary": "auto" } }
          ]
        }
      }
    }
  }
}
```

Cada variante puede contener `settings`, `headers` y `body`; se aplican después de los valores del proveedor y del modelo.

## Prompt caching

OpenCode coloca breakpoints de caché automáticamente en rutas conocidas, como Anthropic. Si sirves Claude a través de un proxy OpenAI-compatible (LiteLLM, por ejemplo), indícalo:

```jsonc
{
  "providers": {
    "litellm": {
      "package": "@opencode/ai/providers/openai-compatible",
      "settings": { "baseURL": "https://litellm.example.com/v1" },
      "models": {
        "claude-sonnet-5-5": {
          "compatibility": { "cacheFormat": "anthropic" }
        }
      }
    }
  }
}
```

La vida de la caché se ajusta con `settings.cache.ttl` (`"5m"`, `"1h"`, etc.). Un TTL más largo suele costar más de escritura.

## Ver también

- [[agentes/opencode/fundamentos/configuracion|Configuración]]
- [[agentes/opencode/extension/agentes|Agentes y subagentes]]
- [[agentes/opencode/guias/desde-claude-code|Desde Claude Code]]

Sources:
- [OpenCode Models](https://opencode.ai/docs/models/)
- [OpenCode Providers](https://opencode.ai/docs/providers/)
- [OpenCode Console Models](https://opencode.ai/docs/console/models/)
- [OpenCode Go](https://opencode.ai/docs/console/go/)
- [models.dev](https://models.dev)
