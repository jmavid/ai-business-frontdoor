# Informe de validación MCP — oassis.dev

**Fecha de ejecución:** 2026-10-03 (UTC)
**Objetivo MCP:** `https://api.oassis.dev/mcp`
**Documentación proporcionada:** `https://api.oassis.dev/openapi.json` y
`https://api.oassis.dev/llms.txt`

## Resultado

No fue posible iniciar una sesión con el servidor MCP desde este entorno. La
llamada MCP `initialize` a `https://api.oassis.dev/mcp` fue denegada por el
proxy de salida antes de alcanzar el servidor. Por tanto, no se pudo enviar
`tools/list`, obtener los esquemas de entrada ni ejecutar llamadas a las
herramientas.

Esto es una limitación de conectividad del entorno de pruebas, **no** un
resultado funcional o de seguridad sobre `api.oassis.dev`.

## Llamada MCP ejecutada

| Caso | Solicitud MCP | Resultado observado | Estado |
| --- | --- | --- | --- |
| Negociación de sesión | JSON-RPC `initialize` (protocolo `2025-03-26`) mediante `POST /mcp` | `CONNECT tunnel failed, response 403`; el proxy `envoy` devolvió `403 Forbidden` | Bloqueado |
| Descubrimiento de herramientas | JSON-RPC `tools/list` | No ejecutable: requiere una sesión creada por `initialize` | Bloqueado |
| Validación de parámetros | JSON-RPC `tools/call` para cada herramienta | No ejecutable: `tools/list` no devolvió los esquemas de parámetros | Bloqueado |

## Cobertura de parámetros

No aplica todavía: la respuesta de `tools/list` es la fuente MCP de los nombres
de herramientas y sus esquemas JSON. Como el servidor no recibió `initialize`,
no fue posible obtener dichos esquemas ni construir llamadas `tools/call`
válidas. Inventar valores para los parámetros produciría un informe no
verificable y podría ejecutar operaciones con efectos secundarios.

## Siguiente paso recomendado

Ejecutar la validación desde una red con acceso a `https://api.oassis.dev/mcp`.
La secuencia debe ser exclusivamente MCP: `initialize`, notificación
`notifications/initialized`, `tools/list` y `tools/call` para cada caso. Si
alguna herramienta tiene coste o efectos secundarios, se requiere además una
cuenta y datos de prueba autorizados.

1. Acceso de red desde el ejecutor al endpoint MCP.
2. Credenciales de prueba con permisos y datos no productivos, si alguna
   herramienta requiere autenticación o consume saldo.
3. Autorización explícita para ejecutar herramientas que no sean de solo lectura.

Con esa información se pueden cubrir, por herramienta, los casos válidos,
campos obligatorios ausentes, tipos y límites, valores de enumeraciones,
autenticación y respuestas de error, sin enviar mutaciones a producción.

## Comando MCP reproducible

```bash
curl --connect-timeout 10 --max-time 30 \
  -X POST https://api.oassis.dev/mcp \
  -H 'Accept: application/json, text/event-stream' \
  -H 'Content-Type: application/json' \
  -H 'MCP-Protocol-Version: 2025-03-26' \
  --data '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"validator","version":"1.0.0"}}}'
```
