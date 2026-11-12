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

## Fases

| Fase | Documento | Resultado | Depende de |
|---|---|---|---|
| 1 | `01-fase-cimientos.md` | entorno, suite en verde, benchmark de volumen, perfil fiscal ES | — |
| 2 | `02-fase-bybit-regiones.md` | Bybit Global y Bybit EU seleccionables (candidato a PR upstream) | 1 |
| 3 | `03-fase-importacion-tolerante.md` | detector de formatos, Trade Republic, Bybit CSV, variantes KuCoin | 1 |
| 4 | `04-fase-rendimientos-devengo.md` | intereses y staking valorados en el devengo, clasificados para IRPF | 1 |
| 5 | `05-fase-redes.md` | cobertura verificada de HYPE, ETH, BTC, SOL; XRP Ledger nuevo | 4 |
| 6 | `06-fase-informe-fiscal-es.md` | informe anual ES con trazabilidad; datos del 721 | 3, 4 |
| 7 | `07-fase-conciliacion-agente.md` | motor de avisos, preguntas al usuario, herramientas MCP | 3, 4, 5 |
| 8 | `08-fase-grafo-flujos.md` | grafo interactivo de flujos con categorías y trazabilidad | 6 |
| 9 | `09-fase-telegram.md` | bot de Telegram como cliente del agente | 7 |
| 10 | `10-fase-superficie-publica.md` | web pública en subdominio, distribución, anuncios | 6, 8 |

Camino mínimo para la declaración del usuario: fases 1 → 3 → 4 → 6. Las fases 2, 5, 7, 8 y 9 amplían;
la 10 publica.

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

## Aviso sobre el contenido fiscal

El informe es una ayuda para preparar la declaración; la responsabilidad de lo declarado es del
contribuyente. Las reglas fiscales implementadas citan su fuente y se revisan cada ejercicio.
