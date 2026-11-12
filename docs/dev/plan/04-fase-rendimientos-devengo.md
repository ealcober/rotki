# Fase 4 — Rendimientos valorados en el devengo

**Resultado:** cada pago de intereses, recompensa de staking o ingreso en cripto queda valorado en euros al
precio de mercado del instante en que se recibe, clasificado como rendimiento del capital mobiliario, y
ese valor es el coste de adquisición de las monedas recibidas para su venta posterior.

## Base fiscal

- Las recompensas de staking directo son rendimientos del capital mobiliario (art. 25.2 LIRPF), tributan
  en la base del ahorro y se valoran a valor de mercado en euros el día de la recepción; varias
  recompensas en el mismo día admiten el cambio medio del día (DGT V1766-22;
  `../04-registro-afirmaciones.md`).
- Que ese valor sea el coste de adquisición en la venta posterior es la afirmación 1 del registro: se
  verifica antes de implementar el punto 4.
- El staking delegado (HYPE) sin pronunciamiento expreso de la DGT: afirmación 5. El perfil ES lo trata
  como el directo y lo marca en el informe como criterio aplicado.

## Puntos

1. **Inventario de eventos de rendimiento** que ya genera rotki: recompensas de staking de ETH beacon,
   `delegatorRewards` de Hyperliquid, intereses de exchanges, `INTEREST_PAYMENT`, `FREE_RECEIPT` y
   `BENEFITS_SAVEBACK` de Trade Republic, y cómo los clasifica `rotkehlchen/accounting/rules.py`.
2. **Precio en el instante.** Verificar la granularidad de los oráculos de precio de rotki para el momento
   exacto (hora, no solo día) y el comportamiento cuando falta precio: ninguna ausencia se convierte en
   cero; se produce un aviso `ERROR` con el evento.
3. **Regla de agregación diaria** opcional (cambio medio del día) para rendimientos muy frecuentes, con
   el detalle por evento conservado para la trazabilidad.
4. **Lote de adquisición:** cada rendimiento recibido abre un lote FIFO con coste igual a su valoración,
   de forma que la venta posterior tribute solo por la diferencia.
5. **Clasificación en el perfil ES:** rendimientos del capital mobiliario separados de las ganancias
   patrimoniales en el informe (fase 6), con subtipos: staking, intereses, airdrop/recompensa.
6. **Rendimientos en efectivo con retención** (Trade Republic `INTEREST_PAYMENT` con `tax`): la retención se
   registra para su deducción.

## Tests

- Evento de recompensa a una hora concreta con precio conocido sintético: valoración exacta, lote creado,
  venta posterior tributa por la diferencia.
- Precio ausente: aviso `ERROR`, informe bloqueado, ningún cero.
- Agregación diaria: suma de valoraciones por evento igual a la agregada con el cambio medio dentro de la
  tolerancia definida.
