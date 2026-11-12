# Fase 9 — Telegram

**Resultado:** el usuario conversa con el agente desde Telegram sin coste y sin exponer datos a terceros.

## Puntos

1. **Bot** que corre junto a la instancia (misma máquina), habla con el MCP local y con un modelo local
   (Ollama) o con el cliente que el usuario elija; polling largo, sin webhook público.
2. **Autenticación:** solo el `chat_id` del usuario; cualquier otro remitente es ignorado.
3. **Capacidades:** resumen fiscal, avisos abiertos, responder preguntas del motor de conciliación
   («¿esta wallet es tuya?» con botones), estado de importaciones.
4. **Escrituras:** únicamente las que pasan por `fork_propose_fix` → confirmación en el chat →
   `fork_apply_fix`.
5. **Privacidad:** modo `balanced` del MCP por defecto; nunca se envían claves ni identificadores
   completos.

## Tests

- Mensaje de un `chat_id` no autorizado no produce ninguna llamada al MCP.
- Flujo propuesta → confirmación → aplicación con fixture sintética.
