# Fase 2 — Regiones de Bybit (Global y EU)

**Resultado:** al conectar Bybit se elige entre Global y EU, y las peticiones van al dominio de la entidad
elegida. Se desarrolla en rama limpia desde `develop` para proponerlo a upstream.

Plan detallado previo: documento «Dev Plan - Integración Bybit en Rotki». Este documento lo adapta al
código verificado.

## Puntos

1. **Verificación previa** (afirmación 2): dominio y firma de la API de Bybit EU; issue en upstream
   describiendo la propuesta.
2. **Enum `BybitLocation`** en `rotkehlchen/exchanges/bybit.py`, con la forma de `OkxLocation`
   (`okx.py:90`). Alcance del enum (solo `GLOBAL` y `EEA`, o más entidades regionales de Bybit): decisión
   pendiente del usuario.
3. **URL base por región:** `bybit.py:143` deja de ser fija y se resuelve desde `bybit_location`, con
   `GLOBAL` por defecto.
4. **Propagación:** `exchanges/manager.py` (mismos cinco puntos que `okx_location`: `:178`, `:222`, `:242`,
   `:294`, `:365`) y `api/v1/schemas.py` (junto a `:2194`).
5. **Persistencia:** el valor viaja en los extras del exchange como `OKX_LOCATION_KEY`; se verifica si
   hace falta migración de base de datos o basta el valor por defecto en lectura.
6. **Frontend:** selector de región en el alta de Bybit, replicando el de OKX; textos en las locales.
7. **Documentación de usuario** y entrada en `docs/changelog.rst` de upstream.

## Tests

- Unitario: cada valor del enum resuelve a su dominio.
- API: alta de Bybit con región EEA y lectura posterior, calcando el test de OKX en
  `rotkehlchen/tests/api/test_exchanges.py`.
- Retrocompatibilidad: una credencial guardada sin región carga como `GLOBAL`.
- VCR según la política de upstream; el usuario graba los cassettes con su clave de solo lectura.
