# ADR 01 — Despliegue con coste cero

- **Estado:** propuesto, pendiente de aprobación del usuario.
- **Restricción dura:** ningún pago recurrente nuevo (el usuario ya paga Google Workspace Business).

## Contexto

rotki es una aplicación **local-first y de un solo usuario por instancia**: backend Python con SQLite
cifrada por usuario, frontend Electron o web servido por el propio backend, y servidor MCP que solo escucha
en loopback (`rotkehlchen/mcp/server.py:29`). Toda la lógica fiscal y los datos viven en la máquina que
ejecuta la instancia.

Requisitos del usuario: uso personal inmediato; app pública con anuncios en un subdominio de
`ealcober.com` más adelante; soporte de volúmenes de alta frecuencia (millones de operaciones); acceso por
Telegram; capa agéntica con acceso total al contexto del usuario.

Vías de ingreso previstas: planes de pago de 1, 3 y 5 años, anuncios en la web pública, o ambas (ADR 05).
Durante el desarrollo solo accede el usuario.

## Opciones evaluadas

| Opción | Encaje con rotki | Volumen alto | Coste | Uso comercial | Veredicto |
|---|---|---|---|---|---|
| Google Apps Script | nulo: no ejecuta Python ni SQLite | no: ejecuciones de 6 min y cuotas diarias | 0 | sí | descartada |
| Vercel Hobby | nulo: funciones efímeras, sin proceso persistente ni disco | no | 0 | **no** (solo personal no comercial) | descartada |
| VM Oracle Always Free + rotki en Docker | alto | medio (2 OCPU / 12 GB) | 0 con riesgo de reclamación de recursos | sí | reserva |
| **rotki en la máquina del usuario** (Pop!_OS) | total: es el modelo de rotki | alto: CPU, RAM y disco propios | 0 | — | **elegida para uso personal** |
| SaaS multiusuario propio | requiere reescribir el modelo de un usuario por instancia | alto solo pagando infraestructura | > 0 | sí | descartada por coste y por RGPD |

## Decisión propuesta

**Dos superficies, una sola base de código:**

1. **Instancia personal (fases 1–9).** El fork corre en la máquina del usuario (modo desarrollo o Docker).
   Acceso remoto privado sin abrir puertos:
   - **Tailscale** (plan personal gratuito) para acceso desde el móvil y otros equipos; o
   - **Cloudflare Tunnel + Cloudflare Access** en `fiscal.ealcober.com`, con login restringido al correo del
     usuario (depende de la afirmación 6 del registro).
2. **Superficie pública (fase 10).** La parte pública es **distribución, no alojamiento de datos**: web
   estática en un subdominio de `ealcober.com` con documentación, guías fiscales, descargas del fork y
   anuncios. Cada usuario ejecuta su propia instancia (escritorio o Docker), así que el volumen de cada
   trader de alta frecuencia lo absorbe su propia máquina y el coste de servidor es nulo con cualquier
   número de usuarios. Los datos fiscales nunca salen del equipo del usuario, lo que reduce la exposición
   al RGPD a la analítica de la web estática.

## Consecuencias

- La capa agéntica y el bot de Telegram se ejecutan junto a la instancia (ADR 02 y fase 9).
- La licencia AGPL-3.0 de rotki obliga a publicar el código de cualquier versión modificada que se ofrezca
  por red; el fork ya es público, así que la obligación queda cubierta mientras `origin` esté al día.
- Si la superficie pública llega a necesitar cómputo de servidor, se reabre este ADR.
- Los planes de pago se apoyan en licencias verificables sin conexión y en un servicio mínimo de
  licencias; los datos fiscales siguen en el equipo de cada usuario (ADR 05).
- La persistencia fuera del equipo es opcional y va al Drive del propio usuario como copia cifrada
  (ADR 06).
- La restricción de coste cero se refiere a pagos recurrentes fijos; las comisiones por venta de un
  intermediario de pagos son proporcionales al ingreso y quedan fuera de ella (afirmación 14).

## Preguntas abiertas para el usuario

1. ¿Tailscale o Cloudflare Tunnel para el acceso remoto personal?
2. ¿El DNS de `ealcober.com` está en Cloudflare o en otro proveedor?
3. ¿Se acepta el modelo «web pública = distribución + contenidos con anuncios; datos siempre en local»?
