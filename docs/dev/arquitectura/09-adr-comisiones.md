# ADR 09 — Tratamiento de comisiones

- **Estado:** propuesto. Base fiscal en `../04-registro-afirmaciones.md`: gastos inherentes y transferencias
  entre monederos propios, verificados; afirmaciones 8 y 9, sin verificar.

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
| 6 | Comisión de red entre wallets propias | sin efecto en el IRPF (DGT V0648-24): las unidades gastadas salen del inventario sin ganancia ni pérdida |
| 7 | Comisión de retirada cobrada por un exchange | hacia una wallet propia, como 6; si la retirada forma parte de una compraventa o permuta, gasto inherente a esa operación (casos 1 a 4) |
| 8 | Comisiones y funding de derivados | integran el resultado de la posición |
| 9 | Comisión del validador en staking | el rendimiento es el neto recibido |
| 10 | Transacción fallida con gas consumido | transmisión del activo gastado, sin operación principal |
| 11 | Reembolso de comisión (rebate) | menor coste de la operación a la que corresponde |
| 12 | Comisión cuya moneda o importe no figura en el fichero | pregunta al usuario (ADR 08) |
| 13 | Comisiones de préstamo (*borrowing fees*) | no deducibles (DGT V0648-24); se registran y se muestran aparte |

### Invariantes

1. Toda comisión es un evento ligado a su operación principal.
2. Cada comisión se computa una sola vez: en el coste, en la transmisión o como transmisión propia.
3. Por activo y cuenta, el saldo derivado de los eventos, comisiones incluidas, coincide con el saldo
   observado; la diferencia es un aviso `INCONGRUENCIA`.
4. Una comisión sin precio en su instante es un aviso `ERROR`.
5. El informe fiscal muestra el total de comisiones por tratamiento y permite abrir sus eventos.

## Consecuencias

- El perfil ES fija el ajuste de rotki y añade los tratamientos que el ajuste no cubre (casos 5 a 13).
- Cada fila de la matriz tiene un test con resultado calculado a mano (fase 11).

## Criterios por defecto y alternativas

Donde la fuente no cierra el tratamiento, el perfil ES aplica un criterio por defecto y ofrece las
alternativas; el usuario puede cambiarlo y su elección queda en el registro de decisiones (ADR 08).

| Caso | Por defecto | Alternativas | Motivo del defecto |
|---|---|---|---|
| 5 — comisión en un tercer activo | transmisión de ese activo a valor de mercado | sin efecto, como gasto menor | pagar con un criptoactivo es entregarlo a cambio de un servicio; es el criterio completo y el que menos infraestima la base (afirmación 8) |
| 6 y 7 — transferencia entre wallets propias | sin efecto en el IRPF | transmisión a valor de mercado con ganancia o pérdida | criterio expreso de la DGT (V0648-24); la mecánica de inventario es la afirmación 9 |
| 10 — transacción fallida | como el caso 5 | sin efecto | el activo se entrega al validador igual que en una comisión pagada |
