# Fase 5 — Redes del usuario

**Resultado:** las redes del usuario quedan cubiertas de punta a punta: saldos, historial, staking y
rendimientos, verificados con direcciones de prueba públicas.

## Puntos

1. **Hyperliquid (prioridad).** Verificar con una dirección de prueba: saldos spot y perp en HyperCore,
   staking de HYPE (`query_staking_summary`, `externalapis/hyperliquid.py:637`), recompensas de
   delegación como eventos (`:1722`), funding, HyperEVM. Documentar huecos.
2. **Ethereum:** staking beacon, tokens, L2 que use el usuario; recompensas y MEV como rendimientos.
3. **Bitcoin:** direcciones y xpub; sin rendimientos salvo wrapped.
4. **Solana:** staking nativo y recompensas de epoch como rendimientos en el devengo.
5. **XRP Ledger (nuevo).** rotki no lo soporta (`types.py:468`). Diseño siguiendo el patrón de una cadena
   no EVM existente (Bitcoin o Solana): `SupportedBlockchain.XRP`, módulo `rotkehlchen/chain/xrp/`, API
   pública de XRPL para saldos e historial, eventos de pago y trustlines. Candidato a PR upstream.
6. **Detección de wallets propias:** cruzar retiradas de exchanges con direcciones destino y proponer al
   usuario añadirlas (alimenta la fase 7).

## Tests

- Por red: fixtures sintéticas de respuesta de API y test de deserialización a eventos; tests VCR según la
  política de upstream con direcciones públicas sin relación con el usuario.
