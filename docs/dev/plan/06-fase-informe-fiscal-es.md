# Fase 6 — Informe fiscal español

**Resultado:** un informe anual con los datos que el usuario traslada al modelo 100 y, si procede, al
modelo 721, en el que cada cifra abre la lista de eventos que la producen.

## Alcance

- **Modelo 100 (IRPF):** ganancias y pérdidas patrimoniales por transmisión (FIFO, base del ahorro),
  rendimientos del capital mobiliario (fase 4), retenciones soportadas, pérdidas pendientes de
  compensar.
- **Modelo 721:** valor conjunto a 31 de diciembre de la cripto custodiada por terceros fuera de España
  por entidad, con el umbral de 50.000 € y la lista de entidades (Bybit, KuCoin, Trade Republic…).
- Fuera de alcance: rellenar los modelos; el usuario los cumplimenta aparte.

## Puntos

1. **Estructura del informe:** secciones, totales y, por cada total, la consulta que lo reproduce.
2. **Trazabilidad:** identificadores de eventos en cada línea; exportación CSV y JSON con esa relación.
3. **Saldos a 31/12 por custodio** y marca de custodia extranjera por entidad (configurable).
4. **Avisos bloqueantes:** el informe no se cierra con `ERROR` o `INCONGRUENCIA` abiertos (fase 7); se
   emite como borrador con la lista de pendientes.
5. **Justificación:** anexo con los criterios fiscales aplicados y su fuente, y con cada decisión del
   registro de decisiones que afecta al ejercicio: qué se preguntó, qué se eligió, por qué y a qué
   importes afecta (fase 12).
6. **Vista en el frontend** y exportación (CSV, JSON, PDF).

## Tests

- Caso sintético de un ejercicio completo con resultado calculado a mano: compra, permuta, staking, venta;
  totales exactos y trazabilidad de cada total a sus eventos.
- Umbral del 721: 50.000,00 € no obliga; 50.000,01 € obliga y lista todas las entidades (la obligación nace
  cuando el valor conjunto supera los 50.000 €).
