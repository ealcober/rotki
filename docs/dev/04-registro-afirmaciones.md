# Registro de afirmaciones

## Afirmaciones sin verificar

Cada una se verifica antes de que una fase dependa de ella. Al verificarse, sale de esta lista y su
contenido pasa al documento que corresponda con su fuente.

| # | Afirmación | De qué depende | Cómo se verifica |
|---|---|---|---|
| 1 | El valor de mercado en euros en la fecha de recepción de una recompensa de staking o de intereses en cripto es su valor de adquisición para la transmisión posterior (FIFO). | Fases 4 y 6 | Lectura de la consulta V1766-22 y de consultas posteriores de la DGT sobre criptoactivos recibidos en especie |
| 2 | Bybit EU expone la API v5 con el mismo esquema de firma que `api.bybit.com` y otro dominio base. | Fase 2 | Documentación oficial de Bybit EU y una petición firmada de solo lectura con clave real |
| 3 | rotki mantiene un rendimiento usable con un millón de eventos en una base de usuario. | Fases 3 y 10 | Benchmark con datos sintéticos (fase 1) |
| 4 | Cloudflare Pages y Workers en plan gratuito permiten uso comercial con anuncios. | Fase 10 | Lectura de los términos vigentes de Cloudflare |
| 5 | El staking delegado (como el de HYPE) se califica igual que el staking directo, como rendimiento del capital mobiliario. | Fase 4 | La DGT no se ha pronunciado sobre el delegado según Cuatrecasas; seguimiento de nuevas consultas |
| 6 | El dominio `ealcober.com` puede servir un subdominio vía Cloudflare (DNS gestionado por Cloudflare o delegación del subdominio). | Fase 10 | Revisión de la configuración DNS actual del dominio |

## Afirmaciones verificadas que fijan decisiones

| Afirmación | Fuente |
|---|---|
| Las recompensas de staking directo son rendimientos del capital mobiliario (art. 25.2 LIRPF), base del ahorro, valoradas a valor de mercado en euros el día de la recepción; varias recompensas el mismo día admiten el cambio medio del día. | DGT V1766-22, resumida en [Cuatrecasas](https://www.cuatrecasas.com/es/spain/art/tributacion-del-staking-en-irpf) |
| El plan Hobby de Vercel es solo para uso personal no comercial. | [Vercel Hobby](https://vercel.com/docs/plans/hobby) |
| El Always Free de Oracle Cloud para Ampere A1 queda en 1.500 OCPU-h y 9.000 GB-h al mes (2 OCPU / 12 GB). | [Linuxiac](https://linuxiac.com/?p=214114) |
| El MCP de rotki exige suscripción Basic o superior para sus herramientas premium. | `rotkehlchen/mcp/premium.py:13` |
| rotki no soporta XRP Ledger. | `rotkehlchen/types.py:468` |

## Afirmaciones caídas

| Afirmación | Medición que la tumbó | Lección |
|---|---|---|
| «Vercel gratis sirve para la app pública con anuncios» | Términos del plan Hobby: uso no comercial | Verificar términos de uso, no solo cuotas |
| «Oracle Always Free da 4 OCPU / 24 GB» | Documentación de Oracle con los nuevos límites | Las cifras de guías de terceros caducan |
| «Hay que construir la capa MCP desde cero» | `rotkehlchen/mcp/` existe con registro, privacidad y transporte | Mapear upstream antes de diseñar |
