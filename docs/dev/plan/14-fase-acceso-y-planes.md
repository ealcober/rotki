# Fase 14 — Acceso de terceros y planes

**Resultado:** personas distintas del usuario acceden con los planes Trader, Pro-Trader y High Freq Trader,
pagados en criptoactivos a wallets del usuario.
Diseño: `../arquitectura/05-adr-monetizacion-y-acceso.md`.

## Puntos

1. **Decisiones previas:** preguntas del ADR 05 respondidas; afirmaciones 4, 11, 14, 15, 16 y 17 verificadas;
   revisión con un asesor de las obligaciones de la actividad.
2. **Módulo de permisos** `rotkehlchen/fork/entitlements/` con los proveedores `dev` y `admin` (se crea en
   cuanto exista un límite de plan, para que ningún módulo consulte licencias por su cuenta).
3. **Configuración de planes** en un único fichero (ejercicios, transacciones por ejercicio, periodo de
   acceso, precio) y contador de transacciones por ejercicio con las exclusiones del ADR 05.
4. **Aplicación de límites:** corte de la importación en el límite, bloqueo de la exportación del
   informe, marca «incompleto» en las cifras del ejercicio afectado, lotes de apertura para los años
   anteriores al primer ejercicio admitido.
5. **Licencias firmadas** verificables sin conexión; proveedor `license`.
6. **Servicio de licencias** mínimo, sin datos fiscales.
7. **Recepción de pagos propia** (ADR 05, punto 6): generación de pedidos, identificación por dirección
   derivada, etiqueta o importe único según la red, vigilante de pagos, cola de revisión y emisión de
   licencia. Primero la infraestructura común; después un adaptador por red: Bitcoin, Ethereum,
   Hyperliquid, XRP Ledger y Solana. Página de planes en la web pública con el plan destacado que fije el
   ADR 05.
8. **Beta cerrada** con licencias emitidas a mano antes de abrir la venta; mide la proporción de
   usuarios que supera cada límite (afirmación 18).
9. **Condiciones de uso, privacidad y aviso fiscal.**

## Tests

- Con los proveedores `dev` y `admin`, ningún límite se aplica y no se realiza ninguna llamada de red.
- Licencia caducada o manipulada, o periodo de acceso terminado: el plan efectivo es Trader.
- Importación que cruza el límite: se guardan exactamente los eventos hasta el límite, el informe de
  importación cuenta el resto, la exportación del informe fiscal queda bloqueada y las cifras del
  ejercicio llevan la marca «incompleto».
- Al subir de plan y reimportar el mismo fichero, se añaden solo los eventos que faltaban.
- Eventos ignorados y de activos spam no cuentan para el límite.
- Un lote de apertura sin coste abre una pregunta y no se trata como coste cero.
- Por cada adaptador de red: pago exacto dentro de ventana emite la licencia; pago insuficiente, excesivo o
  tardío va a la cola de revisión y no emite nada; un mismo pago no emite dos licencias.
- El servicio de pagos no contiene ninguna clave privada (comprobación estática y de configuración).
- Ningún módulo fuera de `entitlements` referencia planes (comprobación estática).
