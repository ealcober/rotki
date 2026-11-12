# Fase 11 — Comisiones

**Resultado:** todas las comisiones del historial quedan computadas una vez y con el tratamiento del
perfil ES, con invariantes que lo comprueban. Diseño: `../arquitectura/09-adr-comisiones.md`.

## Puntos

1. **Base fiscal:** verificación de las afirmaciones 8 y 9 del registro antes de implementar; hasta
   entonces rigen los criterios por defecto del ADR 09, seleccionables por el usuario.
2. **Inventario:** cómo representa rotki hoy cada caso de la matriz del ADR 09 (evento `FEE`, ajuste
   `include_fees_in_cost_basis`) y qué casos quedan sin cubrir.
3. **Tratamientos del perfil ES** para los casos 5 a 13 de la matriz, con sus criterios por defecto y
   alternativas.
4. **Enlace comisión → operación principal** en los importadores del fork y verificación en los de
   upstream que se usen.
5. **Invariantes** del ADR 09 como comprobaciones que generan avisos.
6. **Totales de comisiones** por tratamiento, disponibles para el informe (fase 6) y el agente (fase 7).

## Tests

- Un caso sintético por fila de la matriz, con resultado calculado a mano.
- Cambiar el criterio de un caso con alternativas recalcula y deja la decisión en el registro.
- Propiedad: en un historial sintético aleatorio, cada comisión aparece exactamente una vez en el
  cálculo.
- Caso de volumen según el ADR 07.

## Documentación

- `../02-mapa-codigo-rotki.md`: ubicación exacta del cómputo de comisiones en el motor contable.
