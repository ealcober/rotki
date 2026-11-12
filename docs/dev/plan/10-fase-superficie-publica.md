# Fase 10 — Superficie pública

**Resultado:** un subdominio de `ealcober.com` con la web del proyecto, guías fiscales, descargas e
ingresos por anuncios; los datos de cada usuario siguen en su máquina.
Diseño: `../arquitectura/01-adr-despliegue-coste-cero.md`.

## Puntos

1. **Decisiones previas:** respuestas a las preguntas del ADR 01 y verificación de las afirmaciones 4 y 6.
2. **Web estática** (generador a elegir) desplegada en un plan gratuito que admita uso comercial;
   contenido: qué hace la herramienta, guías del 100 y el 721, instalación (escritorio y Docker),
   changelog.
3. **Distribución:** releases del fork en GitHub (binarios de rotki con la capa ES) y imagen Docker.
4. **Anuncios y analítica** respetuosos con el RGPD; consentimiento donde proceda.
   Los anuncios viven solo en la web pública, nunca dentro de la aplicación con datos fiscales
   (`../arquitectura/05-adr-monetizacion-y-acceso.md`).
5. **Licencia:** aviso AGPL-3.0, enlace al código, créditos a rotki.
6. **Soporte:** issues en GitHub; sin datos de usuarios en servidores propios.

## Tests

- Build de la web reproducible en CI; comprobación de enlaces.
