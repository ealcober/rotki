# Fase 14 — Acceso de terceros y planes

**Resultado:** personas distintas del usuario acceden con los planes Trader, Pro-Trader y High Freq Trader,
pagados en criptoactivos a wallets del usuario.
Diseño: `../arquitectura/05-adr-monetizacion-y-acceso.md`.

## Puntos

1. **Decisiones previas:** preguntas del ADR 05 respondidas; afirmaciones 4, 11, 14, 15, 16 y 17 verificadas;
   revisión con un asesor de las obligaciones de la actividad.
2. **Módulo de permisos** `rotkehlchen/fork/entitlements/` con el proveedor `dev` (se crea en cuanto una
   capacidad de plan exista, para que ningún módulo consulte licencias por su cuenta).
3. **Tabla plan → límite de transacciones** en un único fichero, y contador de transacciones por la unidad
   de cómputo que fije el ADR 05.
4. **Licencias firmadas** verificables sin conexión; proveedor `license`.
5. **Servicio de licencias** mínimo, sin datos fiscales.
6. **Recepción de pagos propia** (ADR 05, punto 6): generación de pedidos, identificación por dirección
   derivada, etiqueta o importe único según la red, vigilante de pagos, cola de revisión y emisión de
   licencia; una red por entrega, empezando por la que fije el usuario. Página de planes en la web pública
   con Pro-Trader destacado.
7. **Beta cerrada** con licencias emitidas a mano antes de abrir la venta.
8. **Condiciones de uso, privacidad y aviso fiscal.**

## Tests

- Con el proveedor `dev`, todas las capacidades responden activas y no se realiza ninguna llamada de red.
- Licencia caducada o manipulada: el plan efectivo es Trader. Superar el límite del plan no trunca ni
  altera ningún cálculo.
- Pago exacto dentro de ventana emite la licencia; pago insuficiente, excesivo o tardío va a la cola de
  revisión y no emite nada; un mismo pago no emite dos licencias.
- El servicio de pagos no contiene ninguna clave privada (comprobación estática y de configuración).
- Ningún módulo fuera de `entitlements` referencia planes (comprobación estática).
