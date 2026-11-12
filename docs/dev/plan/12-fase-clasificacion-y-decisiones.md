# Fase 12 — Clasificación y registro de decisiones

**Resultado:** la herramienta clasifica por capas, pregunta lo que no sabe y lo que falta explicando por
qué, y guarda cada respuesta en un registro que se aplica, se puede revertir y se refleja en el informe.
Diseño: `../arquitectura/08-adr-clasificacion-y-decisiones.md`.

## Puntos

1. **Modelo de datos:** preguntas abiertas sobre el motor de incidencias de rotki; tabla propia de
   decisiones con migración de esquema del fork.
2. **Capa 1, reglas deterministas,** incluida la aplicación de decisiones previas a nuevas importaciones.
3. **Capa 2, heurísticas** con puntuación y umbral: titularidad probable de una dirección, emparejado de
   retirada y depósito, cadencia de recompensas.
4. **Estado «sin clasificar»** visible, contado y bloqueante cuando afecta a importes del informe.
5. **Preguntas** con la forma del ADR 08, agrupadas por patrón; opción «no lo sé» con tratamiento
   conservador.
6. **Información que falta:** catálogo de ausencias (precio, coste de adquisición, moneda de comisión,
   zona horaria, titularidad) con sus tratamientos y efectos.
7. **Registro de decisiones:** alta, consulta, reversión con recálculo incremental (ADR 07).
8. **Interfaz:** bandeja de preguntas ordenada por importe afectado; historial de decisiones.

La capa 3 (propuesta del agente) y las propuestas de optimización llegan con la fase 7.

## Tests

- Una decisión sobre una dirección reclasifica todos sus eventos y se aplica a una importación posterior.
- Revertir una decisión devuelve el cálculo al estado anterior.
- Un dato ausente nunca produce un valor por defecto: produce una pregunta.
- «No lo sé» aplica el tratamiento conservador y mantiene la pregunta abierta.
