# Plan maestro del fork

Plan de desarrollo completo. Cada fase vive en su propio documento de esta carpeta; el número de fichero
coincide con el número de fase. Este documento fija el objetivo, el orden y los criterios comunes.

## Objetivo

Una herramienta de fiscalidad cripto para residentes en España, construida sobre rotki, que:

1. importa exports CSV/XLSX de exchanges y brokers aunque el formato varíe;
2. integra las redes del usuario (Hyperliquid/HYPE, Ethereum, Bitcoin, Solana, XRP Ledger);
3. valora cada interés y recompensa de staking al precio de mercado del momento del devengo;
4. concilia: detecta wallets propias sin declarar, wallets de ahorro, staking y discrepancias, y pregunta;
5. produce el informe fiscal anual (datos para el modelo 100 y el 721) con trazabilidad hasta el evento;
6. muestra los flujos del periodo como grafo interactivo;
7. ofrece un agente (MCP) con acceso a todo el contexto, también por Telegram;
8. cuesta cero euros de infraestructura.
9. responde siempre, también con historiales de millones de operaciones;
10. pregunta lo que no sabe catalogar o lo que falta, explica por qué y refleja cada decisión en el informe;
11. computa todas las comisiones, en todos los casos, una sola vez;
12. propone mejoras de la declaración y de los hábitos del usuario como simulaciones con cifras;
13. guarda copias cifradas e informes en el Drive del propio usuario;
14. admite acceso de terceros por planes de pago de 1, 3 y 5 años, y anuncios en la web pública.

## Fases

| Fase | Documento | Resultado | Depende de |
|---|---|---|---|
| 1 | `01-fase-cimientos.md` | entorno, suite en verde, benchmark de volumen, perfil fiscal ES | — |
| 2 | `02-fase-bybit-regiones.md` | Bybit Global y Bybit EU seleccionables (candidato a PR upstream) | 1 |
| 3 | `03-fase-importacion-tolerante.md` | detector de formatos, Trade Republic, Bybit CSV, variantes KuCoin | 1 |
| 4 | `04-fase-rendimientos-devengo.md` | intereses y staking valorados en el devengo, clasificados para IRPF | 1 |
| 5 | `05-fase-redes.md` | cobertura verificada de HYPE, ETH, BTC, SOL; XRP Ledger nuevo | 4 |
| 6 | `06-fase-informe-fiscal-es.md` | informe anual ES con trazabilidad; datos del 721 | 3, 4, 11, 12 |
| 7 | `07-fase-conciliacion-agente.md` | motor de avisos, propuestas de optimización, herramientas MCP | 3, 4, 5, 12 |
| 8 | `08-fase-grafo-flujos.md` | grafo interactivo de flujos con categorías y trazabilidad | 6 |
| 9 | `09-fase-telegram.md` | bot de Telegram como cliente del agente | 7 |
| 10 | `10-fase-superficie-publica.md` | web pública en subdominio, distribución, anuncios | 6, 8 |
| 11 | `11-fase-comisiones.md` | tratamiento de comisiones en todos los casos, con invariantes | 1 |
| 12 | `12-fase-clasificacion-y-decisiones.md` | clasificador por capas, preguntas al usuario, registro de decisiones | 3, 4 |
| 13 | `13-fase-respaldo-drive.md` | copias cifradas e informes en el Drive del usuario | 1 |
| 14 | `14-fase-acceso-y-planes.md` | licencias, planes de 1, 3 y 5 años, acceso de terceros | 10, 13 |

Camino mínimo para la declaración del usuario: fases 1 → 3 → 4 → 11 → 12 → 6. Las fases 2, 5, 7, 8, 9 y 13
amplían; la 10 publica y la 14 abre el acceso a terceros. El número de fase es el orden de creación del
documento; el orden de ejecución lo fija la columna «Depende de».

## Criterios comunes a todas las fases

- **Método:** cada fase se amplía con `/challenge-ampliar` (los puntos de su documento son el esqueleto) y
  se implementa con `/challenge-implementar`. La documentación se actualiza con `/champliar-docs`.
- **Hecho significa:** tests reales en verde (sin mocks de la función bajo prueba, datos sintéticos),
  linters de upstream en verde, documentación en presente, commit pusheado.
- **Decisiones abiertas:** las preguntas de los ADR (`../arquitectura/`) se cierran antes de empezar la
  fase que depende de ellas.
- **Afirmaciones sin verificar:** `../04-registro-afirmaciones.md`. Una fase no arranca sobre una
  afirmación de la que depende y que sigue sin verificar.
- **Privacidad:** ningún dato real del usuario entra en git.
- **Respuesta:** ninguna operación bloquea la interfaz; cada fase declara su presupuesto de tiempo y
  memoria por tramo de volumen y lo mide con el benchmark de la fase 1 (`../arquitectura/07-adr-escala-y-respuesta.md`).
- **Decisiones:** ningún valor por defecto sustituye a un dato desconocido; se pregunta y se registra
  (`../arquitectura/08-adr-clasificacion-y-decisiones.md`).
- **Acceso:** hasta la fase 14 la instancia es de un único usuario y solo escucha en loopback o en la red
  privada del usuario.

## Aviso sobre el contenido fiscal

El informe es una ayuda para preparar la declaración; la responsabilidad de lo declarado es del
contribuyente. Las reglas fiscales implementadas citan su fuente y se revisan cada ejercicio.
