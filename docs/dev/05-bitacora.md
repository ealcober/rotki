# Bitácora del fork

Registro fechado de hitos. Cada entrada describe el estado que deja, en presente.

## 2026-11-12

- `ealcober/rotki` es fork de `rotki/rotki`; el clon local tiene `origin` (fork) y `upstream`.
- `develop` es espejo de upstream en el commit `b9fb521563`; `fork/main` es la rama de trabajo.
- `docs/dev/` contiene la guía del fork, el mapa del código de rotki, el catálogo de formatos CSV, el
  registro de afirmaciones, nueve ADR propuestos y el plan de desarrollo (`plan/00` a `plan/14`).
- `data/privado/` está ignorado por git para los exports reales del usuario.
- Los ADR 01, 02, 05 y 06 tienen preguntas abiertas para el usuario; ninguna fase de implementación ha
  empezado.
- La documentación recoge como requisitos: acceso exclusivo del usuario durante el desarrollo, planes
  Trader, Pro-Trader (49,99 €) y High Freq Trader (99,99 €) cobrados en cripto más anuncios, respuesta
  garantizada con historiales de millones de operaciones, preguntas al usuario ante lo no catalogable,
  registro de decisiones, tratamiento completo de comisiones y copias cifradas en el Drive del usuario.
- Decisiones del usuario: planes y anuncios a la vez; mismas capacidades en todos los planes; cobro propio
  en wallets; comisiones entre wallets propias sin efecto en el IRPF por defecto (DGT V0648-24).
- Decisiones del usuario sobre planes: Trader 1 ejercicio, Pro-Trader 2 ejercicios y 1 mes de acceso,
  High Freq Trader ejercicios ilimitados y 12 meses; al alcanzar un límite no se guarda más ni se exporta
  el informe; el administrador queda fuera de toda restricción; redes de cobro Bitcoin, Ethereum,
  Hyperliquid, XRP Ledger y Solana, con la infraestructura común primero.
- Límites confirmados (Trader 100 y Pro-Trader 10.000 transacciones por ejercicio); plan destacado High
  Freq Trader; web pública en el dominio provisional `tax.papertrading.es`. Pendientes: direcciones de
  cobro y nombre del producto.
- Requisito: la aplicación funciona en móvil, tablet y escritorio (guía, regla 11).
