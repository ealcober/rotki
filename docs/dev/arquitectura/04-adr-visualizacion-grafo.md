# ADR 04 — Visualización de flujos en grafo

- **Estado:** propuesto.

## Contexto

El usuario quiere ver, para un periodo fiscal, por dónde se mueve el dinero: nodos (cuentas, exchanges,
wallets, contrapartes, activos) y aristas (transferencias, trades, recompensas), con interacción para
separar, juntar y agrupar nodos en categorías propias, y que sirva para trazar cada importe del informe.

rotki ya incluye `d3-force` y un grafo de interacciones entre direcciones en
`frontend/app/src/modules/statistics/address-interactions/`.

## Decisión propuesta

1. **Módulo nuevo** `frontend/app/src/modules/fork-flow-graph/`, construido sobre el motor de
   `address-interactions` (layout determinista con `d3-force`) en lugar de una librería nueva.
2. **Datos desde el backend:** un endpoint del fork agrega eventos del periodo en nodos y aristas con
   importe en EUR, número de eventos e identificadores de los eventos, para que cada arista abra su lista
   de eventos (trazabilidad).
3. **Agrupación solo visual:** las categorías del usuario se guardan como preferencias de vista; no
   alteran eventos ni cálculos.
4. **Escala:** con más de unos pocos miles de nodos, el grafo arranca agregado por cuenta y activo y se
   expande bajo demanda; el renderizado pasa a canvas cuando el número de elementos lo exige.
   La agregación se calcula en el backend sobre totales precalculados, con los presupuestos del ADR 07.
5. **Diseño:** estética «panel de control» oscura y luminosa, con modo claro; responsive con panel de
   detalle en hoja inferior en móvil. Se aplican las convenciones de diseño y accesibilidad del frontend de
   rotki.

## Consecuencias

- El grafo es también la vista de navegación del agente: cada aviso de la jerarquía (ADR 02) puede
  resaltar sus nodos y aristas.
