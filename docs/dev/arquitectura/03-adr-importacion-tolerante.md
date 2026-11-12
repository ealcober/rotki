# ADR 03 — Importación tolerante de CSV y XLSX

- **Estado:** propuesto.

## Contexto

Los importadores de rotki (`rotkehlchen/data_import/importers/`) esperan un formato fijo por fuente. Los
exchanges cambian columnas, idioma y formato de fecha entre versiones del export, y algunos (Bybit, Trade
Republic) no tienen importador.

## Decisión propuesta

Una capa previa, `rotkehlchen/data_import/fork_detect/`, que convierte cualquier export reconocible en el
formato de un importador existente o en eventos de rotki:

1. **Lectura robusta:** detección de codificación, BOM, separador, decimal y fila de cabecera; XLSX con
   varias hojas.
2. **Huella de formato:** cada formato conocido se describe en un fichero declarativo
   (`formats/<origen>-<variante>.yaml`) con columnas obligatorias, alias por idioma, tipos y reglas de
   conversión. La detección puntúa el fichero contra cada huella y elige la de mayor puntuación.
3. **Confianza explícita:** por encima del umbral se importa; en la zona intermedia se muestra al usuario
   el mapeo propuesto columna a columna para que lo confirme; por debajo, se rechaza con el motivo. Un
   mapeo confirmado se guarda como variante nueva.
4. **Sin ausencias leídas como valor:** una columna obligatoria vacía o ilegible produce un aviso por fila,
   nunca un cero.
5. **Deduplicación** por identificador de origen (`transaction_id` o equivalente) y, si no existe, por
   huella (fecha, activo, cantidad, tipo).
6. **Escala:** lectura en streaming por bloques e inserción en lotes, para ficheros de millones de filas.

## Consecuencias

- Añadir un formato es añadir un YAML y sus tests sintéticos, sin código nuevo en la mayoría de casos.
- Los importadores de upstream se reutilizan donde existen; la capa del fork solo normaliza.
