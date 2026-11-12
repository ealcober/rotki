# Formatos de exportación de exchanges y brokers

Catálogo de formatos que el importador tolerante (fase 3) debe reconocer. Solo se documentan **cabeceras,
tipos y valores de enumeración**; ninguna fila real entra en el repositorio. Los ficheros de ejemplo del
usuario viven en `data/privado/` (ignorado por git).

## Estado del catálogo

| Origen | Formato | Estado | Importador en rotki |
|---|---|---|---|
| Trade Republic — «Exportación de transacción» | CSV, coma, comillas dobles, UTF-8 | cabeceras verificadas sobre un export real de 2026 | no |
| Bybit | por obtener | pendiente: el usuario aporta exports a `data/privado/bybit/` | no (solo API) |
| KuCoin | por obtener | pendiente: el usuario aporta exports a `data/privado/kucoin/` | sí (`importers/kucoin.py`), formato fijo |
| Binance — «Exportar Historial de Depósito» y «Exportar historial de compra» | XLSX, una hoja | cabeceras verificadas sobre exports reales de 2021 | sí para CSV (`importers/binance.py`), otro formato |
| «Historial Trades de Exportación-AAAA-MM-DD hh_mm_ss.xlsx» (origen por confirmar con el usuario; el formato coincide con el historial spot de MEXC) | XLSX, hoja `Sheet1` | cabeceras verificadas sobre un export real de 2025 | no |

## Trade Republic — «Exportación de transacción.csv»

Relevante para el modelo 721: Trade Republic custodia fuera de España.

| Columna | Tipo | Notas |
|---|---|---|
| `datetime` | ISO 8601 UTC con fracción (`…Z`) | marca temporal del evento |
| `date` | `AAAA-MM-DD` | fecha contable |
| `account_type` | enum | `DEFAULT` |
| `category` | enum | `CASH`, `TRADING`, `DELIVERY` |
| `type` | enum | ver tabla siguiente |
| `asset_class` | enum o vacío | `CRYPTO`, `STOCK` |
| `name` | texto | nombre del activo o de la contraparte |
| `symbol` | texto | ticker cripto (`BTC`) o ISIN en acciones |
| `shares` | decimal con signo | negativo en ventas |
| `price` | decimal | precio unitario en `currency` |
| `amount` | decimal con signo | importe en `currency`; negativo en compras y pagos |
| `fee` | decimal con signo o vacío | comisión |
| `tax` | decimal con signo o vacío | retención aplicada |
| `currency` | ISO 4217 | `EUR` |
| `original_amount`, `original_currency`, `fx_rate` | decimal, ISO 4217, decimal | operaciones en divisa |
| `description` | texto libre | contiene el identificador del plan de ahorro y la cantidad |
| `transaction_id` | UUID | identificador único: clave de deduplicación |
| `counterparty_name`, `counterparty_iban`, `payment_reference` | texto | **datos personales**: el importador los descarta |
| `mcc_code` | entero o vacío | categoría del comercio en pagos con tarjeta |

Valores de `type` observados y su tratamiento previsto:

| `type` | Significado | Evento rotki previsto |
|---|---|---|
| `BUY`, `SELL` | compraventa (incluye ejecuciones de plan de ahorro) | trade contra EUR |
| `FREE_RECEIPT` | entrega gratuita de cripto (categoría `DELIVERY`) | ingreso con valor de mercado en la recepción |
| `BENEFITS_SAVEBACK` | recompensa en cripto por gasto con tarjeta | ingreso con valor de mercado en la recepción |
| `INTEREST_PAYMENT` | intereses sobre efectivo, con retención en `tax` | rendimiento del capital mobiliario en EUR |
| `DIVIDEND_EQUIVALENT_PAYMENT` | distribución sobre acción | fuera del alcance cripto; se reporta aparte |
| `TRANSFER_INSTANT_INBOUND`, `TRANSFER_INSTANT_OUTBOUND` | transferencias bancarias | movimiento de efectivo, sin efecto fiscal |
| `CARD_TRANSACTION`, `CARD_TRANSACTION_INTERNATIONAL` | pagos con tarjeta | fuera del alcance; se descarta |

## Binance — exports XLSX con interfaz en español

El nombre del fichero está traducido; las cabeceras están en inglés.

| Fichero | Cabeceras |
|---|---|
| `Exportar Historial de Depósito.xlsx` (depósitos fiat) | `Date(UTC)`, `Coin`, `Amount`, `Status`, `Payment Method`, `Indicated Amount`, `Fee`, `Order ID` |
| `Exportar historial de compra.xlsx` (compra de cripto con fiat) | `Date(UTC)`, `Method`, `Amount`, `Price`, `Fees`, `Final Amount`, `Status`, `Transaction ID` |

Clave de deduplicación: `Order ID` y `Transaction ID`. Solo las filas con `Status` completado generan evento.

## «Historial Trades de Exportación» — historial spot con cabeceras en español

| Columna | Tipo | Notas |
|---|---|---|
| `Pares` | texto | par con guion bajo: `BASE_QUOTE` |
| `Tiempo` | `AAAA-MM-DD hh:mm:ss` | sin zona en el dato; la zona se confirma con el usuario antes de importar |
| `Dirección` | enum traducido | `Compra`, `Venta` |
| `Precio Completo` | decimal | precio de ejecución en la moneda cotizada |
| `Monto Ejecutado` | decimal | cantidad de la moneda base |
| `Total` | decimal | importe en la moneda cotizada |
| `Comisión` | decimal | la moneda de la comisión no figura en el fichero: se infiere y se marca con un aviso |
| `Rol` | enum | `Maker`, `Taker` |

El fichero no trae identificador de operación: la deduplicación usa la huella (par, tiempo, dirección,
precio, cantidad) y las ejecuciones parciales idénticas en el mismo segundo se cuentan por número de
apariciones.

## Variaciones que el importador absorbe

Inventario de variaciones a cubrir con tests sintéticos (fase 3):

- separador `,` / `;` / tabulador, y decimal `.` / `,`;
- cabeceras traducidas (español, inglés) o con mayúsculas y espacios distintos;
- columnas añadidas, reordenadas o eliminadas entre versiones del export;
- fechas en ISO, `DD/MM/AAAA hh:mm:ss` o epoch en milisegundos, con y sin zona;
- BOM UTF-8 y codificación Windows-1252;
- XLSX con una o varias hojas y filas de título antes de la cabecera.
