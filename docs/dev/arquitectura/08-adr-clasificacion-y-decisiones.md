# ADR 08 — Clasificación, preguntas al usuario y registro de decisiones

- **Estado:** propuesto.

## Contexto

La herramienta cataloga con inteligencia todo lo que puede, pregunta lo que no sabe, pregunta cómo tratar
la información que falta y por qué, y refleja cada respuesta. El agente, además, propone mejoras.

rotki aporta: tipos y subtipos de evento, reglas contables por tipo de evento
(`rotkehlchen/accounting/rules.py`), libreta de direcciones (`rotkehlchen/db/addressbook.py`) y el motor
de incidencias (`rotkehlchen/history/data_issues/`).

## Decisión propuesta

### Clasificación por capas

| Capa | Qué resuelve | Confianza |
|---|---|---|
| 1. Reglas deterministas | formatos conocidos, tipos de evento, contrapartes de la libreta, decisiones previas del usuario | total |
| 2. Heurísticas | patrones: ingresos recurrentes desde una dirección, ida y vuelta entre dos cuentas, cadencia de recompensas de staking, depósito que casa con una retirada | puntuada |
| 3. Propuesta del agente | casos que las capas 1 y 2 dejan abiertos, sobre datos anonimizados | puntuada, siempre como propuesta |
| 4. Pregunta al usuario | todo lo que queda por debajo del umbral de confianza | — |

Un evento que ninguna capa resuelve queda en estado «sin clasificar», visible y contado. Si afecta a un
importe del informe, bloquea su cierre.

### Forma de una pregunta

Cada pregunta lleva: qué se desconoce; por qué importa (importe afectado en EUR y dirección del sesgo);
opciones, cada una con su consecuencia fiscal; la opción recomendada y su motivo; y la opción «no lo sé»,
que aplica el tratamiento conservador y deja la pregunta abierta. Las preguntas se agrupan por patrón: una
respuesta sobre una dirección resuelve todos sus eventos.

### Información que falta

Precio ausente, coste de adquisición desconocido, moneda de una comisión, zona horaria de un fichero,
titularidad de una dirección: cada ausencia abre una pregunta con sus tratamientos posibles (por ejemplo:
aportar el dato, usar el precio de mercado de la fecha, o coste cero como criterio conservador) y el
efecto de cada uno. Ninguna ausencia se rellena con un valor por defecto.

### Registro de decisiones

Cada respuesta se guarda con: pregunta, tratamiento elegido, motivo dado por el usuario, fuente si la
hay, alcance (evento, dirección o patrón), fecha, cliente (interfaz, agente, Telegram) y eventos
afectados. Una decisión:

- se aplica como regla a las importaciones futuras (capa 1);
- es reversible, y revertirla recalcula;
- aparece en el anexo del informe fiscal, en el grafo y en las herramientas MCP.

### Propuestas de optimización

El agente genera simulaciones sobre los datos del usuario: compensación de pérdidas latentes antes del
cierre del ejercicio (con la regla de recompra aplicada como restricción, afirmación 10), cercanía al
umbral del 721, ahorro de comisiones por hábitos, consolidación de saldos residuales. Cada propuesta
lleva cifras, supuestos, fuente y la etiqueta «simulación». El método FIFO es obligatorio y ninguna
propuesta lo altera. Las propuestas informan; la decisión y la responsabilidad son del usuario.

## Consecuencias

- Las preguntas abiertas y las decisiones se apoyan en el motor de incidencias de rotki más una tabla
  propia de decisiones (migración de esquema del fork).
- El informe fiscal se cierra solo cuando no quedan preguntas que afecten a sus importes (fase 6).
