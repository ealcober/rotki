# Bitácora del fork

Registro fechado de hitos. Cada entrada describe el estado que deja, en presente.

## 2026-11-12

- `ealcober/rotki` es fork de `rotki/rotki`; el clon local tiene `origin` (fork) y `upstream`.
- `develop` es espejo de upstream en el commit `b9fb521563`; `fork/main` es la rama de trabajo.
- `docs/dev/` contiene la guía del fork, el mapa del código de rotki, el catálogo de formatos CSV, el
  registro de afirmaciones, nueve ADR propuestos y el plan de desarrollo (`plan/00` a `plan/14`).
- `data/privado/` está ignorado por git para los exports reales del usuario.
- Los ADR 01, 02, 05, 06 y 09 tienen preguntas abiertas para el usuario; ninguna fase de implementación ha
  empezado.
- La documentación recoge como requisitos: acceso exclusivo del usuario durante el desarrollo, planes de
  pago de 1, 3 y 5 años o anuncios como vías de ingreso, respuesta garantizada con historiales de
  millones de operaciones, preguntas al usuario ante lo no catalogable, registro de decisiones,
  tratamiento completo de comisiones y copias cifradas en el Drive del usuario.
