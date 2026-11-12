# Fase 7 — Conciliación y agente

**Resultado:** la herramienta revisa toda la situación del usuario, clasifica discrepancias por gravedad,
pregunta lo que no puede decidir y propone correcciones justificadas; el agente MCP hace lo mismo en
conversación. Diseño: `../arquitectura/02-adr-capa-agentica-mcp.md`.

## Puntos

1. **Motor de avisos** con la jerarquía `ERROR` / `INCONGRUENCIA` / `AVISO` / `INFO`, cada aviso con
   eventos, importe, dirección del sesgo y opciones.
2. **Detectores iniciales:** venta sin coste de adquisición; retirada sin depósito emparejado; depósito
   desde dirección desconocida con flujo recurrente («¿esta wallet es tuya?»); saldo negativo; precio
   ausente; wallet de ahorro o staking no conectada (saldos en exchange que salen a una dirección y no
   vuelven); recompensas de staking sin evento de precio.
3. **Preguntas al usuario** desde el frontend: respuesta guardada como regla (dirección propia,
   contraparte, categoría) que reclasifica los eventos afectados y recalcula.
4. **Herramientas MCP del fork** (`rotkehlchen/mcp/tools/fork_*.py`): `fork_reconciliation_report`,
   `fork_unknown_counterparties`, `fork_tax_summary_es`, `fork_propose_fix`, `fork_apply_fix`; todas con
   la anonimización de rotki respetada y la escritura con confirmación.
5. **Registro de auditoría** de cada corrección aplicada: quién, cuándo, desde qué cliente, qué cambió.
6. **Conexión de clientes:** Claude Desktop y Claude Code del usuario al MCP local; prueba de extremo a
   extremo de una conversación de conciliación.

## Tests

- Por detector: fixture sintética que dispara el aviso y otra que no; gravedad y sesgo esperados.
- `fork_apply_fix` sin identificador de propuesta previa es rechazada.
