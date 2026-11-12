# Fase 1 — Cimientos

**Resultado:** el fork arranca en la máquina del usuario, la suite pasa, se conoce el rendimiento con
volumen alto y existe un perfil fiscal español.

## Puntos

1. **Entorno de desarrollo.** `uv sync`, `pnpm install` y arranque web sin Electron según `CLAUDE.md` de
   upstream. Se documenta en `../01-guia-fork-llm.md` cualquier paso específico de Pop!_OS.
2. **Línea base de tests.** Suite de backend y frontend ejecutada sobre `fork/main` recién creada; salida
   guardada fuera de git como referencia.
3. **CI del fork.** Decidir qué workflows de upstream se activan en el fork (minutos gratuitos de GitHub
   Actions en repos públicos) y cuáles quedan desactivados.
4. **Benchmark de volumen** (afirmación 3). Generador de eventos sintéticos (10⁵, 10⁶, 5·10⁶ trades) y
   medición de: importación, consulta de historial, informe de P&L FIFO, tamaño de la base. Predicción
   escrita antes de medir; el resultado decide si la fase 3 necesita inserción por lotes propia y si el
   cálculo FIFO necesita optimización.
   Las mismas mediciones fijan los presupuestos de respuesta por tramo de volumen del ADR 07.
5. **Perfil fiscal ES.** Conjunto de ajustes contables de rotki para España: FIFO, EUR como moneda,
   permuta cripto-cripto como transmisión, año natural. Se implementa como preset seleccionable, sin
   cambiar los valores por defecto de upstream.
6. **`data/privado/`.** Carpeta local ignorada por git para exports reales; el usuario deposita ahí sus
   CSV de Bybit, KuCoin y demás para completar `../03-formatos-csv.md`.
7. **Acceso en desarrollo.** La instancia escucha solo en loopback; el acceso remoto del usuario va por la
   red privada elegida en el ADR 01. Se verifica que ningún puerto queda expuesto a Internet.

## Tests

- Test del preset ES: aplicar el perfil deja los ajustes esperados y un informe sintético usa FIFO.
- El benchmark es un script de medición, no un test de la suite.

## Documentación

- `../02-mapa-codigo-rotki.md`: añadir ubicación de los ajustes contables.
- `../05-bitacora.md`: resultado del benchmark.
