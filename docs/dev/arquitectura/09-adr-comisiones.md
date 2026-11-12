# ADR 09 — Tratamiento de comisiones

- **Estado:** propuesto, pendiente de verificar su base fiscal (afirmaciones 7, 8 y 9).

## Contexto

Las comisiones se computan correctamente en todos los casos. rotki modela la comisión como evento con
subtipo `FEE` (`rotkehlchen/history/events/structures/types.py:64`) y ofrece el ajuste
`include_fees_in_cost_basis` (`rotkehlchen/db/settings.py:73`).

## Decisión propuesta

### Matriz de casos

| # | Caso | Tratamiento en el perfil ES |
|---|---|---|
| 1 | Compra con comisión en la moneda de pago | suma al valor de adquisición |
| 2 | Compra con comisión descontada del activo comprado | la cantidad recibida es la neta; el coste total se reparte sobre ella |
| 3 | Venta con comisión en la moneda cobrada | minora el valor de transmisión |
| 4 | Permuta cripto-cripto con comisión en uno de los dos activos | como 2 o 3 según el lado |
| 5 | Comisión pagada en un tercer activo (token del exchange, gas) | transmisión de ese activo a valor de mercado, con su ganancia o pérdida; su valor suma al coste o minora la venta de la operación principal |
| 6 | Comisión de red entre wallets propias | transmisión del activo gastado; su cómputo como gasto es una decisión del usuario registrada (ADR 08) |
| 7 | Comisión de retirada cobrada por un exchange | como 6 |
| 8 | Comisiones y funding de derivados | integran el resultado de la posición |
| 9 | Comisión del validador en staking | el rendimiento es el neto recibido |
| 10 | Transacción fallida con gas consumido | transmisión del activo gastado, sin operación principal |
| 11 | Reembolso de comisión (rebate) | menor coste de la operación a la que corresponde |
| 12 | Comisión cuya moneda o importe no figura en el fichero | pregunta al usuario (ADR 08) |

### Invariantes

1. Toda comisión es un evento ligado a su operación principal.
2. Cada comisión se computa una sola vez: en el coste, en la transmisión o como transmisión propia.
3. Por activo y cuenta, el saldo derivado de los eventos, comisiones incluidas, coincide con el saldo
   observado; la diferencia es un aviso `INCONGRUENCIA`.
4. Una comisión sin precio en su instante es un aviso `ERROR`.
5. El informe fiscal muestra el total de comisiones por tratamiento y permite abrir sus eventos.

## Consecuencias

- El perfil ES fija el ajuste de rotki y añade los tratamientos que el ajuste no cubre (casos 5 a 12).
- Cada fila de la matriz tiene un test con resultado calculado a mano (fase 11).

## Preguntas abiertas para el usuario

1. Criterio por defecto para los casos 6 y 7 mientras la afirmación 9 siga sin verificar.
