# Fase 8 — Grafo de flujos

**Resultado:** vista interactiva del periodo fiscal en la que el dinero se ve moverse entre cuentas,
exchanges, wallets y activos, con agrupación por categorías propias y trazabilidad hasta el evento.
Diseño: `../arquitectura/04-adr-visualizacion-grafo.md`.

## Puntos

1. **Endpoint de agregación** del backend: nodos y aristas del periodo con importes en EUR, recuento de
   eventos e identificadores.
2. **Módulo** `frontend/app/src/modules/fork-flow-graph/` sobre el motor de `address-interactions`.
3. **Interacción:** hover con resumen, clic para abrir eventos, arrastrar para separar, soltar sobre otro
   nodo para agrupar, categorías propias persistidas como preferencia de vista.
4. **Filtros:** periodo, activo, tipo de evento, nivel de aviso (resalta lo que señala la fase 7).
5. **Escala:** vista agregada inicial, expansión bajo demanda, canvas por encima de un umbral de
   elementos.
6. **Diseño y accesibilidad:** tema oscuro y claro, responsive, teclado, contraste.

## Tests

- Unitario del agregador: un conjunto sintético de eventos produce el grafo esperado.
- Spec de componente según la política de tests del frontend de upstream.
