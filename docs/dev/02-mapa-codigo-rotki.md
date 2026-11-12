# Mapa del código de rotki usado por el fork

Cada entrada está verificada contra el árbol de `develop` en el commit `b9fb521563`. Las líneas citadas se
revisan al sincronizar con upstream.

## Arquitectura general

- Backend Python (`rotkehlchen/`) con API REST (puerto 4242) y WebSocket (4333).
- Frontend Vue 3 + TypeScript (`frontend/app/`), empaquetado como Electron o servido como web.
- Servicio Rust `colibri/` para piezas de rendimiento.
- Persistencia: SQLite por usuario (cifrada) y una base global de activos.
- Abstracción central: los **history events**. Trades, depósitos, staking, intereses y transferencias
  on-chain son eventos con tipo y subtipo; la contabilidad recorre esos eventos.

## Contabilidad e informe de P&L

| Pieza | Ubicación | Uso en el fork |
|---|---|---|
| Métodos de coste | `rotkehlchen/types.py:829` (`CostBasisMethod`: FIFO, LIFO, HIFO, ACB) | España exige FIFO; el perfil fiscal ES lo fija |
| Motor contable | `rotkehlchen/accounting/accountant.py:52` (`Accountant`) | fuente del cálculo de ganancias y pérdidas |
| Exportación del informe | `rotkehlchen/accounting/export/csv.py`, `report.py` | base del informe fiscal ES |
| Reglas por tipo de evento | `rotkehlchen/accounting/rules.py`, `entry_type_mappings.py` | clasificación fiscal de intereses y staking |

## Exchanges

| Pieza | Ubicación |
|---|---|
| Conector Bybit | `rotkehlchen/exchanges/bybit.py`; URL base fija en `:143` (`https://api.bybit.com/v5`) |
| Patrón de regiones OKX | `rotkehlchen/exchanges/okx.py:90` (`OkxLocation`), dominio por región en `:124` |
| Propagación de la región | `rotkehlchen/exchanges/manager.py:178`, `:222`, `:242`, `:294`, `:365` |
| Esquema API de la región | `rotkehlchen/api/v1/schemas.py:2194` |
| Guía de upstream para añadir un exchange | `AGENTS.md`, sección «Exchange Addition» |

## Importación CSV

| Pieza | Ubicación |
|---|---|
| Fuentes soportadas | `rotkehlchen/data_import/manager.py:35` (`DataImportSource`) |
| Entrada de importación | `rotkehlchen/data_import/manager.py:86` (`import_csv`) |
| Importadores por formato | `rotkehlchen/data_import/importers/` (Binance, KuCoin, Nexo, Crypto.com, Blockpit, CoinTracking…) |

No existe importador CSV de Bybit, Trade Republic ni Hyperliquid. Cada importador espera un formato fijo
de columnas; la tolerancia a variaciones es trabajo del fork (fase 3).

## Redes

`SupportedBlockchain` en `rotkehlchen/types.py:468`. Estado para las redes del usuario:

| Red | Soporte en rotki | Módulo |
|---|---|---|
| Ethereum (+ staking beacon) | sí | `rotkehlchen/chain/ethereum/`, `ETHEREUM_BEACONCHAIN` |
| Bitcoin | sí | `rotkehlchen/chain/bitcoin/` |
| Solana | sí | `rotkehlchen/chain/solana/` |
| Hyperliquid (HyperCore + HyperEVM) | sí | `rotkehlchen/chain/hyperliquid/`, `rotkehlchen/externalapis/hyperliquid.py` |
| XRP Ledger | **no** | — (fase 5) |

Staking de HYPE: `query_staking_summary` (`rotkehlchen/externalapis/hyperliquid.py:637`) consulta
`delegatorSummary` (delegado, no delegado, retirada pendiente); `delegatorRewards` y `delegatorHistory`
se convierten en eventos (`:1542`, `:1722`).

## Servidor MCP

rotki incluye un servidor MCP propio en `rotkehlchen/mcp/` (dependencia `mcp==1.30.0`).

| Pieza | Ubicación | Notas |
|---|---|---|
| Registro de herramientas | `rotkehlchen/mcp/registry.py:11` (`register_tool`) | `premium=True` por defecto |
| Puerta de suscripción | `rotkehlchen/mcp/premium.py:13` (`NON_MCP_TIERS` = Free, Supporter) | las herramientas premium exigen plan Basic o superior de rotki |
| Transporte HTTP | `rotkehlchen/mcp/server.py:29` (`validate_loopback_host`) | solo escucha en loopback |
| Modos de privacidad | `rotkehlchen/mcp/constants.py:9` (`strict`, `balanced`, `raw`) | anonimiza identificadores antes de llegar al LLM |
| Herramientas existentes | `rotkehlchen/mcp/tools/` | `info` (libre), `get_asset_details`, `get_historical_prices`, `refresh_analytics_data`, `list_tables`, `describe_table`, `query_sql`, `get_event_taxonomy` |

## Visualización

- El frontend incluye `d3-force`, `echarts` y `vue-echarts`.
- `frontend/app/src/modules/statistics/address-interactions/` ya dibuja un grafo de fuerzas de
  interacciones entre direcciones (`interaction-layout.ts`, `AddressInteractionGraph.vue`,
  `AddressInteractionClusters.vue`). Es la base del grafo de flujos del fork (fase 8).

## Incidencias de datos

rotki incluye un motor de incidencias en `rotkehlchen/history/data_issues/`, con tabla `data_issues`
(tipo, gravedad, estado, intentos de remediación, `payload_json`). Es la base del motor de avisos del fork
(fase 7).

| Pieza | Ubicación | Notas |
|---|---|---|
| Tipos de incidencia | `rotkehlchen/history/data_issues/constants.py:5` (`IssueKind`) | saldo actual discrepante, saldo negativo, token con rebase, bridge sin emparejar |
| Estados | `constants.py:12` (`IssueState`) | `open`, `auto_remediating`, `unresolved`, `resolved`, `dismissed` |
| Gravedad | `constants.py:20` (`IssueSeverity`) | un único nivel, `warning`; el fork amplía la jerarquía |
| Escritura de incidencias | `rotkehlchen/history/data_issues/manager.py:141` (`DataIssuesManager`), `write_issue` en `:147` | punto de extensión para los detectores del fork |
| Detectores | `rotkehlchen/history/data_issues/detectors/` | patrón a seguir |
| Remediación automática | `rotkehlchen/tasks/data_issues.py:317` (`run_data_issue_remediation`) | |

## Tareas en segundo plano y paginación

| Pieza | Ubicación | Uso en el fork |
|---|---|---|
| Consultas asíncronas de la API | `rotkehlchen/api/rest.py:398` (argumento `async_query`), `:663` (`_query_async`) | toda operación larga del fork se expone así |
| Tareas periódicas | `rotkehlchen/tasks/` (`manager.py`, `supervisor.py`) | detectores y copias programadas |
| Paginación de filtros | `rotkehlchen/db/filtering.py:182` (`LIMIT`/`OFFSET`) | todo listado del fork pagina en servidor |
| Eventos de P&L persistidos | `rotkehlchen/db/reports.py:84` (`DBAccountingReports`), inserción en `:240` | el informe ES lee eventos ya calculados |
| Servicio Rust | `colibri/src/` | candidato para rutas críticas de rendimiento |

El MCP de upstream carga las tablas analíticas con `pandas` (`rotkehlchen/mcp/analytics.py:14`); las
herramientas del fork agregan en SQL para acotar la memoria (ADR 07).

## Benchmark y datos sintéticos

Upstream trae la infraestructura completa de medición; el benchmark de la fase 1 la extiende, no la
reconstruye.

| Pieza | Ubicación | Notas |
|---|---|---|
| Perfiles sintéticos deterministas | `tools/scenarios/` (`profiles/empty.py`, `small.py`, `whale.py` con ~400k eventos) | caché por esquema en `cache.py`; CLI `uv run python -m tools.scenarios build --profile <p>` |
| Harness macro | `tools/bench/` (`run` y `compare` base/head) | arranca el backend real y mide por la API; muestras y estadísticas en `stats.py` |
| Operaciones medidas | `tools/bench/operations.py:94` (`OPERATIONS`) | historial paginado/filtrado, búsqueda de activos, balances, netvalue, redecode; **no mide importación CSV ni informe de P&L** |
| Microbenchmarks | `rotkehlchen/tests/benchmarks/` | excluidos de la suite normal (`pyproject.toml:539`); `pytest -m benchmark --benchmark-only` |
| Workflow de CI | `.github/workflows/rotki_benchmarks.yml` | opt-in por etiqueta `run-benchmarks` o `[bench]` en el título del PR |

## Comisiones

| Pieza | Ubicación |
|---|---|
| Subtipo de evento de comisión | `rotkehlchen/history/events/structures/types.py:64` (`FEE`), categoría en `:199` |
| Ajuste de comisiones en el coste | `rotkehlchen/db/settings.py:73` (`include_fees_in_cost_basis`, activo por defecto) |

## Sincronización y copias de upstream

- `rotkehlchen/premium/sync.py:76` (`PremiumSyncManager`): sincronización de la base del usuario con los
  servidores de rotki, ligada a su suscripción. El fork no la modifica; su almacenamiento propio es el
  ADR 06.
- La base del usuario es un fichero SQLite cifrado; rotki genera copias locales en cada actualización de
  esquema (`rotkehlchen/db/dbhandler.py:244`, `DB_BACKUP_RE`).

## Documentación de upstream

- `docs/` es Sphinx con `source_suffix = '.rst'` (`docs/conf.py:70`): los `.md` de `docs/dev/` no entran en
  la build de upstream.
- CI de upstream: `.github/workflows/`. En un fork, GitHub mantiene los workflows deshabilitados hasta
  activarlos desde la pestaña Actions.
