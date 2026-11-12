# ADR 02 — Capa agéntica sobre el MCP de rotki

- **Estado:** propuesto, pendiente de aprobación del usuario.

## Contexto

rotki incluye un servidor MCP (`rotkehlchen/mcp/`) con registro de herramientas por decorador, anonimización
configurable (`strict`, `balanced`, `raw`), consultas SQL de solo lectura sobre tablas analíticas, y
transporte `stdio` o HTTP en loopback. Sus herramientas premium exigen suscripción Basic o superior de
rotki (`rotkehlchen/mcp/premium.py:13`).

El fork necesita un agente que: revise toda la situación del usuario, pregunte por wallets desconocidas,
detecte discrepancias, las clasifique por gravedad y proponga correcciones justificadas.

## Decisión propuesta

1. **Se extiende el MCP de rotki**, no se crea uno paralelo. Las herramientas del fork viven en
   `rotkehlchen/mcp/tools/fork_*.py` y se registran con `@register_tool`.
2. **Herramientas del fork** (contrato detallado en la fase 7):

   | Herramienta | Efecto | Tipo |
   |---|---|---|
   | `fork_reconciliation_report` | discrepancias por gravedad con eventos que las sostienen | lectura |
   | `fork_unknown_counterparties` | direcciones con flujos relevantes que no son del usuario | lectura |
   | `fork_tax_summary_es` | resumen fiscal ES del año con trazabilidad | lectura |
   | `fork_propose_fix` | propuesta de corrección con su efecto en el informe | lectura |
   | `fork_apply_fix` | aplica una corrección aprobada | **escritura** |

3. **Escritura con confirmación.** Toda herramienta que modifique datos devuelve primero una propuesta con
   identificador y efecto calculado; solo `fork_apply_fix` con ese identificador escribe, y deja rastro en
   un registro de auditoría consultable.
4. **El LLM no se paga aparte.** Clientes previstos, todos sin coste nuevo:
   - Claude Desktop o Claude Code del usuario conectados al MCP local (incluido en su suscripción);
   - un modelo local con Ollama para el bot de Telegram cuando el usuario no está en el escritorio.

## Decisión pendiente del usuario: la puerta premium

Las herramientas existentes de rotki (`query_sql`, `describe_table`…) exigen suscripción Basic. Opciones:

| Opción | Coste | Upstream | Comentario |
|---|---|---|---|
| A. Suscribirse a rotki Basic | > 0 | compatible | contradice la regla de coste cero |
| B. Herramientas del fork con `premium=False` y acceso propio a la base del usuario | 0 | no se propone upstream | duplica parte de `analytics`; respeta el modelo de negocio de rotki en sus herramientas |
| C. Quitar la puerta premium en el fork | 0 | incompatible | la AGPL lo permite; deja de apoyar económicamente a rotki y diverge del código de upstream |

Recomendación: **B**. Las herramientas del fork cubren lo fiscal español, que rotki no ofrece, sin tocar
la puerta de las herramientas de upstream.

## Jerarquía de avisos

| Nivel | Significado | Ejemplo | Bloquea el informe |
|---|---|---|---|
| `ERROR` | el informe sería incorrecto | venta sin coste de adquisición; saldo negativo | sí |
| `INCONGRUENCIA` | los datos se contradicen entre fuentes | retirada de un exchange sin depósito correspondiente | sí, hasta resolver o justificar |
| `AVISO` | dato probable pero no confirmado | dirección con ingresos recurrentes no marcada como propia | no |
| `INFO` | optimización o contexto | recompensas de staking agrupables por día | no |

Cada aviso lleva: eventos implicados, importe afectado en EUR, dirección del sesgo (sobreestima o
infraestima la base imponible) y opciones de resolución.
