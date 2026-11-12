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
| 8 | Pagar una comisión en un criptoactivo distinto del negociado (BNB, KCS, HYPE como gas) es una transmisión de ese activo a valor de mercado, con su propia ganancia o pérdida. | Fase 11 | Consultas de la DGT sobre permutas y pagos con criptoactivos |
| 9 | Las unidades gastadas como comisión en una transferencia entre wallets propias salen del inventario FIFO sin generar ganancia ni pérdida, y su coste de adquisición no se traslada a otras unidades. Es la lectura operativa de «irrelevante a efectos del IRPF» (V0648-24). | Fase 11 | Lectura del texto íntegro de la consulta V0648-24 en el buscador de la DGT |
| 10 | La regla de recompra de dos meses (art. 33.5 LIRPF) no alcanza a los criptoactivos. | Fase 7, propuestas de optimización | Lectura del artículo y de consultas de la DGT; hasta verificarla, las propuestas la aplican como restricción |
| 11 | El plazo de prescripción de cuatro años (art. 66 LGT) justifica conservar datos y ofrecer planes plurianuales. | Fase 14 | Lectura del artículo |
| 12 | El ámbito `drive.file` de la API de Google Drive limita la app a los ficheros que ella crea y evita la auditoría de seguridad exigida a los ámbitos restringidos. | Fase 13 | Documentación vigente de Google sobre ámbitos OAuth y verificación de apps |
| 13 | Una base SQLite abierta dentro de una carpeta sincronizada por un cliente de nube puede corromperse. | Fase 13 | Documentación de SQLite sobre corrupción de ficheros; la decisión del ADR 06 no depende del resultado, solo su justificación |
| 14 | Cobrar los planes en criptoactivos mantiene las obligaciones de facturación e IVA en euros del vendedor, y cada cobro es un ingreso valorado en euros en el momento de recibirlo. | Fase 14 | Revisión con un asesor fiscal antes de abrir la venta |
| 15 | El nombre y la marca «rotki» no pueden usarse para un producto derivado de pago. | Fase 14 | Política de marca de rotki y consulta a sus mantenedores |
| 16 | La AGPL-3.0 permite cobrar por el acceso a builds y servicios de un producto derivado mientras el código fuente siga disponible. | Fase 14 | Lectura de `LICENSE.md` y de las FAQ de la FSF sobre venta de software libre |
| 17 | Los pagos entrantes en cada red de cobro se detectan con fiabilidad usando solo APIs y nodos públicos gratuitos. | Fase 14 | Prueba por red con pagos reales de importe mínimo, cuando el usuario fije las redes |

## Afirmaciones verificadas que fijan decisiones

| Afirmación | Fuente |
|---|---|
| Las recompensas de staking directo son rendimientos del capital mobiliario (art. 25.2 LIRPF), base del ahorro, valoradas a valor de mercado en euros el día de la recepción; varias recompensas el mismo día admiten el cambio medio del día. | DGT V1766-22, resumida en [Cuatrecasas](https://www.cuatrecasas.com/es/spain/art/tributacion-del-staking-en-irpf) |
| El plan Hobby de Vercel es solo para uso personal no comercial. | [Vercel Hobby](https://vercel.com/docs/plans/hobby) |
| El Always Free de Oracle Cloud para Ampere A1 queda en 1.500 OCPU-h y 9.000 GB-h al mes (2 OCPU / 12 GB). | [Linuxiac](https://linuxiac.com/?p=214114) |
| El MCP de rotki exige suscripción Basic o superior para sus herramientas premium. | `rotkehlchen/mcp/premium.py:13` |
| rotki no soporta XRP Ledger. | `rotkehlchen/types.py:468` |
| Los gastos y tributos originados por la operación, con relación directa y satisfechos por el contribuyente, se tienen en cuenta en los valores de adquisición y de transmisión. | [Manual práctico de Renta, AEAT](https://sede.agenciatributaria.gob.es/Sede/ayuda/manuales-videos-folletos/manuales-practicos/irpf-2022/c11-ganancias-perdidas-patrimoniales/monedas-virtuales/compra-venta-monedas-virtuales-tributacion-inversor.html) |
| El intercambio entre monedas virtuales distintas es una permuta; las unidades de una misma moneda son bienes homogéneos y se transmiten primero las adquiridas en primer lugar (FIFO). | mismo manual de la AEAT |
| La comisión pagada por una simple transferencia entre dos monederos del mismo titular es irrelevante a efectos del IRPF; la comisión de retirada que forma parte de una compraventa o permuta es gasto inherente a esa operación; las comisiones de préstamo (*borrowing fees*) no son deducibles. | DGT V0648-24, resumida en [Cuatrecasas](https://www.cuatrecasas.com/es/spain/fiscalidad/art/defi-dgt-analiza-diversas-cuestiones) |
| rotki incluye un motor de incidencias de datos con gravedad única `warning`. | `rotkehlchen/history/data_issues/constants.py:20` |
| Vercel Hobby considera uso comercial la inclusión de anuncios. | [Vercel Fair Use Guidelines](https://vercel.com/docs/limits/fair-use-guidelines) |

## Afirmaciones caídas

| Afirmación | Medición que la tumbó | Lección |
|---|---|---|
| «Vercel gratis sirve para la app pública con anuncios» | Términos del plan Hobby: uso no comercial | Verificar términos de uso, no solo cuotas |
| «Oracle Always Free da 4 OCPU / 24 GB» | Documentación de Oracle con los nuevos límites | Las cifras de guías de terceros caducan |
| «Hay que construir la capa MCP desde cero» | `rotkehlchen/mcp/` existe con registro, privacidad y transporte | Mapear upstream antes de diseñar |
| «El motor de avisos se construye desde cero» | `rotkehlchen/history/data_issues/` existe con tabla, estados, detectores y remediación | Mapear upstream antes de diseñar, también lo que no se buscaba |
