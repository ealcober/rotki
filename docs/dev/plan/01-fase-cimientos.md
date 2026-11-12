# Fase 1 — Cimientos

**Resultado:** el fork arranca en la máquina del usuario, la suite pasa, se conoce el rendimiento con
volumen alto, existe un perfil fiscal español y el trabajo vive en un repositorio privado.

## Puntos

1. **Entorno de desarrollo.** `uv sync`, `pnpm install` y arranque web sin Electron según `CLAUDE.md` de
   upstream. Se documenta en `../01-guia-fork-llm.md` cualquier paso específico de Pop!_OS.
2. **Línea base de tests.** Suite de backend y frontend ejecutada sobre `fork/main` recién creada; salida
   guardada fuera de git como referencia.
3. **CI del fork.** Decidir qué workflows de upstream se activan en el fork y cuáles quedan desactivados,
   contando con que el repositorio privado (punto 9) tiene los minutos de Actions limitados
   (afirmación 22).
4. **Benchmark de volumen** (afirmación 3). Perfiles sintéticos de 10⁵, 10⁶ y 5·10⁶ eventos —extensión de
   `tools/scenarios/` de upstream, no un generador nuevo— y medición de: importación, consulta de
   historial, informe de P&L FIFO, tamaño de la base. Predicción escrita antes de medir; el resultado
   decide si la fase 3 necesita inserción por lotes propia y si el cálculo FIFO necesita optimización.
   Las mismas mediciones fijan los presupuestos de respuesta por tramo de volumen del ADR 07.
5. **Perfil fiscal ES.** Conjunto de ajustes contables de rotki para España: FIFO, EUR como moneda,
   permuta cripto-cripto como transmisión, año natural. Se implementa como preset seleccionable, sin
   cambiar los valores por defecto de upstream.
6. **`data/privado/`.** Carpeta local ignorada por git para exports reales; el usuario deposita ahí sus
   CSV de Bybit, KuCoin y demás para completar `../03-formatos-csv.md`.
7. **Acceso en desarrollo.** La instancia escucha solo en loopback; el acceso remoto del usuario va por la
   red privada elegida en el ADR 01. Se verifica que ningún puerto queda expuesto a Internet.
8. **Estado responsive del frontend de rotki** (afirmación 20): recorrido de las vistas que usa el fork
   (importación, historial, informe de P&L, estadísticas) en anchos de móvil, tablet y escritorio, con
   capturas fuera de git. El resultado fija qué adaptaciones de upstream necesita el fork y se anota en
   `../02-mapa-codigo-rotki.md`.
9. **Repositorio privado.** El trabajo del fork se traslada a un repositorio privado (decisión del usuario,
   2026-11-12): GitHub no permite poner privado un fork de un repositorio público (afirmación 21), así que
   se duplica el repositorio y el fork público `ealcober/rotki` queda solo como puente de PRs hacia
   upstream. La guía (`../01-guia-fork-llm.md`, «Ramas y remotos») describe el esquema de remotos
   resultante.

## Ampliación punto a punto

### Punto 1 — Entorno de desarrollo

**Challenge.** Los prerequisitos vienen del `CLAUDE.md` de upstream: Node 22+, pnpm 10+, Python 3.14t,
Rust estable y uv. El riesgo real en Pop!_OS es Python 3.14t (freethreaded), que no viene empaquetado:
`uv python install` lo resuelve sin tocar el sistema.

**Trabajo.**

1. `uv sync` en la raíz; `pnpm install` en la raíz y `pnpm install --frozen-lockfile` en `frontend/`.
2. Arranque backend: `uv run python -m rotkehlchen --api-port 4242 --websockets-port 4333`.
3. Arranque web sin Electron: `pnpm dev:web` desde la raíz.
4. Cualquier dependencia de sistema que falte en Pop!_OS se añade a la guía del fork en el momento de
   descubrirla, con el paquete exacto.

**Hecho cuando:** `curl http://127.0.0.1:4242/api/1/ping` responde y la web carga y permite crear un
usuario local. Lo falsaría cualquier paso de arranque que exija algo no documentado.

### Punto 2 — Línea base de tests

**Challenge.** La suite completa de backend incluye tests con red y cassettes y tarda horas; una línea
base útil es la que se re-ejecuta tras cada sincronización con upstream y cabe en una tarde. El CI de
upstream ya parte los tests en jobs (`.github/workflows/task_backend_tests.yml`,
`task_fe_unit_tests.yml`), que sirven de guía del alcance.

**Alcance de la línea base:** backend `uv run pytest rotkehlchen/tests/unit` y `uv run make lint`;
frontend, desde `frontend/`: `pnpm run lint`, `pnpm run typecheck`, `pnpm run test:unit`; colibri:
`cargo test` en `colibri/`. Salida completa (comando, commit, recuentos, fallos) guardada en
`data/privado/linea-base/<fecha>-<commit>/` (ignorada por git, `.gitignore:47`).

**Regla sobre fallos preexistentes:** un fallo que ya está en `develop` de upstream no es un defecto del
fork; se registra con su salida en la línea base y no bloquea. Un fallo nuevo respecto a la línea base sí
es un punto numerado. Esta distinción es la razón de guardar la línea base.

**Hecho cuando:** existen los ficheros de salida con recuentos y la bitácora (`../05-bitacora.md`) anota
fecha, commit y resumen.

### Punto 3 — CI del fork

**Challenge.** `.github/workflows/` trae 18 workflows (CI, nightly, builds, releases, benchmarks). En un
fork, GitHub los mantiene deshabilitados hasta activarlos. Con el repositorio privado (punto 9) los
minutos de Actions dejan de ser ilimitados (afirmación 22), y los gates ya corren en local (punto 2).

**Decisión:** ningún workflow activado en fase 1. Los gates se ejecutan en local antes de cada push. La
decisión se revisa al llegar la fase 10 (releases y builds públicas). No se edita ningún fichero de
workflow: cada línea tocada en upstream es un conflicto futuro.

**Hecho cuando:** la pestaña Actions del repositorio de trabajo no muestra ningún run tras un push.

### Punto 4 — Benchmark de volumen (afirmación 3)

**Challenge.** El plan pedía un «generador de eventos sintéticos»; upstream ya lo tiene, y más: perfiles
sintéticos deterministas con caché (`tools/scenarios/`, perfiles `empty`, `small` y `whale` con ~400k
eventos sobre 6 años), un harness macro que arranca el backend real y mide operaciones por la API
(`tools/bench/`, con `run` y `compare` base/head, muestras y estadísticas), y microbenchmarks
pytest-benchmark (`rotkehlchen/tests/benchmarks/`, excluidos de la suite normal, `pyproject.toml:539`).
La afirmación implícita «hay que construirlo» cayó y está en el registro. El punto queda redefinido:
**extender perfiles y operaciones, no construir infraestructura.**

**Trabajo.**

1. Perfiles del fork en `tools/scenarios/profiles/`: `fork_hft_1e5`, `fork_hft_1e6` y `fork_hft_5e6`,
   registrados en `registry.py`, reutilizando las distribuciones de `whale.py` escaladas. La caché por
   esquema de `tools/scenarios/cache.py` hace soportable el coste de construcción de 5·10⁶.
2. Operaciones nuevas para `tools/bench/operations.py` (hoy mide historial paginado/filtrado, búsqueda de
   activos, balances y redecode; no mide importación CSV ni informe de P&L): importación de un CSV
   sintético (`import_csv`, `rotkehlchen/data_import/manager.py:86`), informe de P&L FIFO de un año
   natural, y tamaño del fichero de base tras la carga.
3. Métricas por operación y tramo: duración, memoria pico (RSS) y, para el historial, tiempo de la
   primera página. Cada medición repite muestras y reporta mediana y dispersión (lo hace ya
   `tools/bench/stats.py`); el mejor de k intentos no es una medición.

**Predicciones — escritas antes de medir, con el significado de cada resultado:**

| # | Predicción | Si se cumple | Si falla por poco (hasta 2×) | Si falla por mucho (>2×) |
|---|---|---|---|---|
| P1 | Inserción sostenida ≥ 2.000 eventos/s a 10⁶ | la inserción de upstream basta para la fase 3 | lotes de upstream con ajuste de tamaño de transacción | la fase 3 necesita inserción por lotes propia (ADR 03.6) |
| P2 | Primera página del historial < 1 s a 10⁶ y < 3 s a 5·10⁶ (`rotkehlchen/db/filtering.py:182`) | la paginación de upstream vale | índices adicionales | paginación keyset propia antes de la fase 3 |
| P3 | Informe P&L FIFO de un año sobre 10⁶ eventos < 30 min y < 4 GB RSS | el motor contable vale para el informe ES | cálculo incremental (ADR 07.5) entra en la fase 6 | ruta candidata a `colibri` y el informe ES no se promete para HFT hasta resolverlo |
| P4 | Tamaño de base entre 1 y 2 KB/evento (5·10⁶ ≈ 5–10 GB) | el disco local absorbe HFT | retención/limpieza configurable | revisar payloads e índices antes de prometer HFT |

**Dirección del sesgo:** cada predicción optimista que falle se descubre aquí o se descubre en la fase 6
con datos reales del usuario; por eso un presupuesto incumplido es punto numerado y la fase no cierra con
él abierto (ADR 07). El resultado medido va a la bitácora y fija los presupuestos del ADR 07.

### Punto 5 — Perfil fiscal ES

**Ajustes localizados** (`rotkehlchen/db/settings.py`, verificados):

| Ajuste | Default de upstream | Valor ES | Motivo |
|---|---|---|---|
| `main_currency` | USD (`:57`) | EUR | moneda del informe |
| `cost_basis_method` | FIFO (`:70`) | FIFO, fijado explícito | España exige FIFO (manual AEAT, verificado) |
| `taxfree_after_period` | 1 año (`:51`; `None` = deshabilitado, `:476`, consumo en `accounting/cost_basis/base.py:200`) | `None` | España no exime ganancias por antigüedad; dejar el default eximiría ventas de más de un año: infraestima la base imponible, el sesgo más grave posible |
| `include_crypto2crypto` | `True` (`:52`) | `True` | la permuta cripto-cripto tributa (manual AEAT, verificado) |
| `include_gas_costs` | `True` (`:53`) | `True` | gastos inherentes (ADR 09) |
| `include_fees_in_cost_basis` | `True` (`:73`) | `True` | ADR 09, casos 1–4 |
| `calculate_past_cost_basis` | `True` (`:62`) | `True` | el FIFO necesita el inventario completo |
| `eth_staking_taxable_after_withdrawal_enabled` | `True` (`:72`; controla si la retirada del validador tributa, `rotkehlchen/history/events/structures/eth2.py:316`) | sin cambio en fase 1 | el tratamiento ES del staking es de la fase 4 (devengo, afirmación 1); hasta entonces se mantiene el valor conservador de upstream |

`date_display_format` ya es `%d/%m/%Y %H:%M:%S %Z` por defecto (`:58`): sin cambio. El **año natural**
no es un ajuste de rotki: es el rango temporal que la fase 6 fija al generar el informe.

**Diseño del preset** (sin cambiar ningún default de upstream):

- Módulo nuevo `rotkehlchen/fork/tax_profile/` con `ES_TAX_PROFILE: Final[ModifiableDBSettings]`
  (`rotkehlchen/db/settings.py:353`) y `apply_tax_profile(db)` que escribe con `DBHandler.set_settings`
  (`rotkehlchen/db/dbhandler.py:823`).
- En fase 1 el preset se aplica por código (función + test); el selector en la interfaz llega con la
  fase 6, siguiendo «Adding a Frontend Setting» del `CLAUDE.md` de upstream.

**Test (de la suite):** aplicar el perfil y leer los ajustes esperados; un informe sintético con dos
adquisiciones a precios distintos y una venta posterior a un año demuestra que (a) se transmite primero
el lote más antiguo (FIFO) y (b) la ganancia sigue tributando (sin exención por antigüedad). El caso (b)
es el control negativo del preset: con el default de upstream, falla.

### Punto 6 — `data/privado/`

**Verificado:** `.gitignore:47` ya ignora `data/privado/`. Queda crear la estructura local y el flujo.

- Estructura: `data/privado/exports/<exchange>/`, `data/privado/linea-base/` (punto 2) y
  `data/privado/capturas/` (punto 8).
- El usuario deposita sus exports reales; de cada formato se copian **solo cabeceras y tipos** a
  `../03-formatos-csv.md`. Ninguna fila real sale de la carpeta.
- Guarda: `git check-ignore data/privado/x` responde afirmativamente y `git status --porcelain` no lista
  nada tras depositar ficheros. Esa comprobación es el test del punto.

### Punto 7 — Acceso en desarrollo

**Verificado:** la API REST escucha por defecto en `127.0.0.1` (`rotkehlchen/args.py:69`) y el MCP
rechaza hosts que no sean loopback (`rotkehlchen/mcp/server.py:29`).

- Con la pila completa arrancada (backend, frontend dev, colibri), auditar con `ss -tlnp`: ningún socket
  del fork escucha en `0.0.0.0` ni en la IP de la LAN. Un puerto fuera de loopback falsaría el punto.
- El acceso remoto personal (Tailscale o Cloudflare Tunnel) es la pregunta 1 del ADR 01 y sigue abierta;
  hasta decidirla no existe acceso remoto. Recomendación registrada: Tailscale, por no depender de la
  afirmación 6 (DNS) y no exponer ningún puerto público.

### Punto 8 — Estado responsive del frontend (afirmación 20)

- **Vistas a recorrer** (las que usa el fork): importación CSV, historial de eventos, informe de P&L y
  estadísticas. Los puntos de corte son los de `useBreakpoint`
  (`frontend/app/src/modules/core/common/use-area-visibility-store.ts:85`).
- **Método:** `pnpm dev:web` y tres anchos en devtools —390×844 (móvil), 820×1180 (tablet),
  1920×1080 (escritorio)—, capturas a `data/privado/capturas/`.
- **Criterio por vista:** usable = la tarea de la vista se completa (importar un fichero, filtrar el
  historial, leer el informe) sin scroll horizontal roto ni controles inalcanzables.
- **Resultado:** tabla en `../02-mapa-codigo-rotki.md` (vista → estado por ancho → adaptación necesaria),
  que convierte la afirmación 20 en verificada o en puntos de adaptación para las fases con interfaz.

### Punto 9 — Repositorio privado

**Challenge.** GitHub no permite cambiar a privado un fork de un repositorio público; la vía documentada
es duplicar el repositorio (afirmación 21: clon espejo empujado a un repositorio privado nuevo; al
ejecutar, comprobar también la opción «leave fork network» en Settings contra la documentación vigente).

**Diseño de remotos tras la migración:**

| Remoto | Apunta a | Papel |
|---|---|---|
| `origin` | repositorio privado nuevo | trabajo diario del fork; recibe todos los push |
| `upstream` | `rotki/rotki` | sincronización con develop |
| `public` | `ealcober/rotki` (fork público) | solo ramas limpias desde `develop` para PRs a upstream |

**Consecuencias:**

- Los minutos de Actions pasan a estar limitados (afirmación 22) → refuerza la decisión del punto 3.
- AGPL-3.0: sin distribución no hay obligación de publicar; al distribuir (fase 10) se publica el código
  de cada versión distribuida (ADR 01, consecuencias).
- La regla de privacidad de la guía no se relaja: el historial puede acabar público (PRs a upstream,
  publicación de la fase 10), así que ningún dato personal entra en git tampoco en privado.

**Hecho cuando:** `git remote -v` muestra el esquema de la tabla, un push llega al repositorio privado y
el fork público no recibe más commits de trabajo.

## Tests

- Test del preset ES: aplicar el perfil deja los ajustes esperados y un informe sintético usa FIFO y no
  exime ganancias por antigüedad (control negativo: con los defaults de upstream, falla).
- El benchmark es un script de medición, no un test de la suite.

## Documentación

- `../02-mapa-codigo-rotki.md`: añadir ubicación de los ajustes contables.
- `../05-bitacora.md`: resultado del benchmark.
- `../02-mapa-codigo-rotki.md`: estado responsive de las vistas de upstream que usa el fork.
