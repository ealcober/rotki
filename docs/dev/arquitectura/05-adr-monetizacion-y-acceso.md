# ADR 05 — Monetización y control de acceso

- **Estado:** propuesto, pendiente de aprobación del usuario.

## Contexto

El proyecto tiene tres etapas de acceso y dos vías de ingreso posibles:

- **Desarrollo:** solo accede el usuario.
- **Después:** acceso de terceros por planes según volumen de transacciones, años de historial y periodo
  de acceso (Trader, Pro-Trader y High Freq Trader) **y** anuncios en la web pública. Los planes se
  cobran en criptoactivos, en wallets del usuario, con un sistema de recepción propio.

Condicionantes:

- rotki es de un usuario por instancia (`rotkehlchen/rotkehlchen.py:199`) y los datos viven en el equipo
  de quien la ejecuta (ADR 01).
- El fork es AGPL-3.0 (`LICENSE.md`) y su código es público: cualquiera puede compilarlo. Lo que se vende
  es lo que el código público no da por sí solo: builds firmadas y actualizadas, reglas fiscales revisadas
  cada ejercicio, servicios (licencias, copias en Drive), guías y soporte (afirmación 16). Es el modelo que
  usa el propio rotki con su suscripción (`rotkehlchen/premium/`).
- El producto de pago lleva nombre y marca propios (afirmación 15). Dominio provisional:
  `tax.papertrading.es`.

## Decisión propuesta

1. **Un único punto de decisión de permisos.** El módulo `rotkehlchen/fork/entitlements/` responde a
   «¿tiene este usuario la capacidad X?». Ningún otro módulo consulta planes ni licencias. Proveedores:

   | Proveedor | Etapa | Comportamiento |
   |---|---|---|
   | `dev` | desarrollo | todas las capacidades activas; instancia local de un usuario |
   | `license` | acceso de terceros | licencia firmada (Ed25519) con plan y fecha de fin, verificable sin conexión |
   | `admin` | cualquier etapa | licencia firmada con rol de administrador: ninguna restricción de plan, límite ni periodo |

   El administrador (el usuario) queda excluido de todas las restricciones en todas las etapas.

2. **Etapas de acceso.**

   | Etapa | Quién accede | Cómo se garantiza |
   |---|---|---|
   | 0 — desarrollo | el usuario | instancia en loopback o red privada (fase 1, punto 7); proveedor `dev` |
   | 1 — beta cerrada | invitados | licencias emitidas a mano; mismo binario |
   | 2 — pública | cualquiera con plan | servicio de licencias y pasarela de pago (fase 14) |

3. **Planes.** Tres niveles con las mismas capacidades, que se distinguen en tres ejes:

   | Plan | Ejercicios fiscales | Transacciones por ejercicio | Periodo de acceso | Precio |
   |---|---|---|---|---|
   | Trader | 1 | 100 | indefinido | 0 € |
   | Pro-Trader | 2 | 10.000 | 1 mes | 49,99 € |
   | High Freq Trader (plan destacado) | sin límite | sin límite | 12 meses | 99,99 € |

   - **Ejercicios fiscales:** años naturales con eventos importados. Un usuario puede trabajar un solo
     ejercicio o varios; el plan fija cuántos admite a la vez. Los años anteriores al primer ejercicio
     admitido entran como **lotes de apertura** (activo, cantidad, fecha y coste de adquisición), que no
     cuentan como transacciones y son imprescindibles para que el FIFO sea correcto. Un lote de apertura
     sin coste abre una pregunta (ADR 08).
   - **Transacciones por ejercicio:** cada operación tal como la ve el usuario cuenta una vez: un trade
     con su comisión, una recompensa, un depósito, una retirada. No cuentan los eventos ignorados por el
     usuario (`rotkehlchen/db/dbhandler.py:2016`) ni los de activos marcados como spam
     (`rotkehlchen/globaldb/handler.py:517`).
   - **Periodo de acceso:** tiempo durante el que el plan está activo. Pro-Trader cubre la campaña de una
     declaración; High Freq Trader, un año completo. Al terminar, el plan efectivo es Trader y los datos
     siguen en el equipo del usuario.
   - **Límites para que casi todos paguen:** 100 transacciones por ejercicio se superan con un DCA semanal
     en dos activos (104) o con una sola posición de staking con recompensa diaria (365); el plan
     gratuito queda como prueba con datos reales. La proporción real de usuarios que supera el límite es
     la afirmación 18 y se mide en la beta cerrada.
   - **Plan destacado: High Freq Trader.** Es el de menor precio por mes (unos 8,33 €), sin límites;
     Pro-Trader queda como pase para una sola declaración.
   - Las cifras de la tabla viven en un único fichero de configuración.
4. **Al alcanzar un límite.** Todos los planes tienen las mismas capacidades (agente, grafo, copias en
   Drive, exportaciones); los límites son solo los del punto 3. Al alcanzar el límite de transacciones o
   de ejercicios:
   - la importación se detiene en el límite y no se guardan más eventos; el informe de importación
     cuenta las filas no guardadas por fichero;
   - el informe fiscal del ejercicio afectado no se puede exportar;
   - toda cifra que la aplicación muestra de ese ejercicio lleva la marca «incompleto» con el número de
     filas no guardadas: un historial truncado produce un FIFO incorrecto y no se presenta como
     resultado;
   - al pasar a un plan superior, el usuario reimporta los ficheros y la deduplicación (ADR 03) añade
     solo lo que faltaba.
5. **Servicio de licencias mínimo.** Emite y renueva licencias; guarda correo, plan y fechas; nunca recibe
   datos fiscales. Corre en un plan gratuito que admita uso comercial (afirmación 4).
6. **Cobro propio en criptoactivos**, en wallets del usuario. Se construye primero la infraestructura,
   independiente de la red, con un adaptador por red; las direcciones las fija el usuario después.
   Redes iniciales y forma propuesta de identificar cada pedido (afirmación 19):

   | Red | Identificación del pedido |
   |---|---|
   | Bitcoin | dirección derivada por pedido desde la clave pública extendida |
   | Ethereum | dirección derivada por pedido desde la clave pública extendida |
   | Hyperliquid (HYPE) | dirección derivada por pedido; los fondos llegan a HyperCore o a HyperEVM |
   | XRP Ledger | una dirección y etiqueta de destino por pedido |
   | Solana | una dirección y clave de referencia por pedido |

   Reglas comunes:
   - precio fijado en euros y cotización en el activo de pago válida durante una ventana corta;
   - identificación de cada pedido sin intervención manual: dirección derivada por pedido donde la red
     lo permite con solo la clave pública, y etiqueta de destino o importe único donde no;
   - el servicio guarda únicamente claves públicas y direcciones: ninguna clave privada sale de las
     wallets del usuario;
   - un vigilante de pagos confirma el ingreso tras las confirmaciones exigidas por red y emite la
     licencia; pagos insuficientes, excesivos o fuera de ventana quedan en una cola de revisión;
   - cada cobro se registra con su valor en euros en el momento de recibirlo (afirmación 14) y es
     importable por la propia herramienta;
   - la detección usa APIs y nodos públicos gratuitos (afirmación 17).
7. **Anuncios** solo en la web pública (fase 10), nunca dentro de la aplicación. Son compatibles con los
   planes: la web con anuncios es además el canal de captación.

## Consecuencias

- El desarrollo no se ve afectado: con los proveedores `dev` y `admin` no existe ninguna restricción.
- Cambiar un límite, un periodo o un precio es editar un fichero de configuración, sin tocar ningún otro
  módulo.
- Añadir una red de cobro es añadir un adaptador.
- Vender acceso es una actividad económica con obligaciones propias (alta, IVA, condiciones de uso,
  RGPD): se revisan con un asesor antes de la etapa 2.

## Preguntas abiertas para el usuario

1. Direcciones y claves públicas de cobro por red.
2. Nombre y marca del producto, sobre el dominio provisional `tax.papertrading.es`.
