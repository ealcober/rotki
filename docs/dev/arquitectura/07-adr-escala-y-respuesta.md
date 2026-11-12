# ADR 07 — Escala y respuesta

- **Estado:** propuesto.

## Contexto

La aplicación está pensada para usuarios con muchísimas transacciones, incluidos traders de alta
frecuencia con millones de operaciones, y la interfaz responde siempre. El rendimiento real de rotki con
ese volumen es la afirmación 3 del registro y se mide en la fase 1.

Piezas de rotki sobre las que se apoya (`../02-mapa-codigo-rotki.md`): consultas asíncronas de la API,
tareas periódicas, paginación en los filtros, eventos de P&L persistidos y el servicio Rust `colibri`.

## Reglas de diseño

Aplican a todo el código del fork.

1. **Trabajo largo en segundo plano.** Toda operación cuyo coste crece con el historial (importar,
   recalcular, generar informe, agregar el grafo, copiar a Drive, ejecutar detectores) es una tarea
   asíncrona con progreso, cancelación y resultado consultable. La petición HTTP que la lanza vuelve de
   inmediato.
2. **Paginación en servidor.** Ningún endpoint devuelve una colección sin límite. El frontend virtualiza
   las listas.
3. **Streaming y lotes.** Las importaciones leen por bloques e insertan por lotes en transacciones
   acotadas, con punto de reanudación tras cada lote confirmado.
4. **Memoria acotada.** Ningún camino carga el historial completo en memoria; las agregaciones se hacen
   en SQL. Las herramientas MCP del fork siguen esta regla.
5. **Cálculo incremental.** Un cambio (nueva importación, respuesta a una pregunta, corrección) recalcula
   desde el primer evento afectado de los activos afectados. El diseño concreto depende de la medición de
   la fase 1.
6. **Totales precalculados** por periodo, activo y cuenta, que alimentan informe, grafo y agente.
7. **Degradación explícita.** Cuando una vista supera su presupuesto, se muestra agregada y lo dice; el
   detalle se pide bajo demanda.
8. **Agrupación para la vista, detalle para el cálculo.** Las ejecuciones parciales de una orden se pueden
   mostrar agrupadas; el cálculo fiscal usa siempre cada ejecución.

## Presupuestos

Cada fase declara, antes de medir, su presupuesto por tramo de volumen (10⁵, 10⁶ y 5·10⁶ eventos):
tiempo hasta la primera respuesta de la interfaz, duración de la tarea en segundo plano y memoria máxima.
El benchmark de la fase 1 los mide; un presupuesto incumplido es un punto numerado de la fase, y no se da
por cerrada con él abierto.

## Consecuencias

- Las rutas que el benchmark señale como críticas son candidatas a implementarse en `colibri`.
- Los tests incluyen un caso de volumen por fase, ejecutado fuera de la suite rápida.
