# Fase 14 — Acceso de terceros y planes

**Resultado:** personas distintas del usuario acceden con planes de 1, 3 y 5 años.
Diseño: `../arquitectura/05-adr-monetizacion-y-acceso.md`.

## Puntos

1. **Decisiones previas:** preguntas del ADR 05 respondidas; afirmaciones 4, 11, 14, 15 y 16 verificadas;
   revisión con un asesor de las obligaciones de la actividad.
2. **Módulo de permisos** `rotkehlchen/fork/entitlements/` con el proveedor `dev` (se crea en cuanto una
   capacidad de plan exista, para que ningún módulo consulte licencias por su cuenta).
3. **Tabla capacidad → plan** en un único fichero.
4. **Licencias firmadas** verificables sin conexión; proveedor `license`.
5. **Servicio de licencias** mínimo, sin datos fiscales.
6. **Pasarela de pago** y página de planes en la web pública, con el plan de 3 años destacado.
7. **Beta cerrada** con licencias emitidas a mano antes de abrir la venta.
8. **Condiciones de uso, privacidad y aviso fiscal.**

## Tests

- Con el proveedor `dev`, todas las capacidades responden activas y no se realiza ninguna llamada de red.
- Licencia caducada, manipulada o de otro plan: la capacidad responde inactiva y el cálculo fiscal sigue
  disponible.
- Ningún módulo fuera de `entitlements` referencia planes (comprobación estática).
