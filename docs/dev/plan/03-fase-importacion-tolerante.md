# Fase 3 — Importación tolerante

**Resultado:** el usuario arrastra varios CSV/XLSX y la herramienta identifica cada formato, propone el
mapeo cuando duda, importa sin duplicar y avisa de lo que no pudo leer. Diseño: `../arquitectura/03-adr-importacion-tolerante.md`.

## Puntos

1. **Lector robusto:** codificación, BOM, separador, decimal, fila de cabecera, XLSX multihoja.
2. **Huellas declarativas** `formats/*.yaml` y motor de puntuación con umbrales de confianza.
3. **Formatos iniciales:** Trade Republic (cabeceras ya catalogadas), Bybit (spot, derivados, depósitos y
   retiradas, earn), variantes de KuCoin que el importador de upstream rechaza, exports XLSX localizados
   en Drive.
4. **Flujo de confirmación** en el frontend: tabla columna → campo, con vista previa de filas ya
   convertidas; el mapeo confirmado se guarda como variante.
   Los datos que el fichero no trae se preguntan (fase 12).
5. **Deduplicación** por identificador de origen o huella de fila; reimportar un fichero no cambia nada.
6. **Informe de importación:** filas importadas, duplicadas, descartadas y con aviso, cada una con su
   motivo. Los datos personales de contrapartes (nombre, IBAN) se descartan en la lectura.
7. **Volumen:** streaming e inserción por lotes, dimensionados con el benchmark de la fase 1; tarea en
   segundo plano con progreso, cancelación y reanudación (ADR 07).
8. **Importación por lotes de varios ficheros** con detección de solapes entre ellos (mismo periodo
   exportado dos veces, API y CSV del mismo exchange).

## Tests

- Un fichero sintético por formato y por cada variación de `../03-formatos-csv.md`.
- Propiedad: importar dos veces produce el mismo estado.
- Una columna obligatoria vacía produce aviso por fila y ningún valor por defecto.

## Documentación

- `../03-formatos-csv.md`: cada formato pasa a «cabeceras verificadas» con su tabla.
