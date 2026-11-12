# ADR 05 — Monetización y control de acceso

- **Estado:** propuesto, pendiente de aprobación del usuario.

## Contexto

El proyecto tiene tres etapas de acceso y dos vías de ingreso posibles:

- **Desarrollo:** solo accede el usuario.
- **Después:** acceso de terceros por planes según volumen de transacciones (Trader, Pro-Trader y High
  Freq Trader) **y** anuncios en la web pública. Los planes se cobran en criptoactivos, en wallets del
  usuario, con un sistema de recepción propio.

Condicionantes:

- rotki es de un usuario por instancia (`rotkehlchen/rotkehlchen.py:199`) y los datos viven en el equipo
  de quien la ejecuta (ADR 01).
- El fork es AGPL-3.0 (`LICENSE.md`) y su código es público: cualquiera puede compilarlo. Lo que se vende
  es lo que el código público no da por sí solo: builds firmadas y actualizadas, reglas fiscales revisadas
  cada ejercicio, servicios (licencias, copias en Drive), guías y soporte (afirmación 16). Es el modelo que
  usa el propio rotki con su suscripción (`rotkehlchen/premium/`).
- El producto de pago lleva nombre y marca propios (afirmación 15).

## Decisión propuesta

1. **Un único punto de decisión de permisos.** El módulo `rotkehlchen/fork/entitlements/` responde a
   «¿tiene este usuario la capacidad X?». Ningún otro módulo consulta planes ni licencias. Proveedores:

   | Proveedor | Etapa | Comportamiento |
   |---|---|---|
   | `dev` | desarrollo | todas las capacidades activas; instancia local de un usuario |
   | `license` | acceso de terceros | licencia firmada (Ed25519) con plan y fecha de fin, verificable sin conexión |

2. **Etapas de acceso.**

   | Etapa | Quién accede | Cómo se garantiza |
   |---|---|---|
   | 0 — desarrollo | el usuario | instancia en loopback o red privada (fase 1, punto 7); proveedor `dev` |
   | 1 — beta cerrada | invitados | licencias emitidas a mano; mismo binario |
   | 2 — pública | cualquiera con plan | servicio de licencias y pasarela de pago (fase 14) |

3. **Planes.** Tres niveles que se distinguen solo por el volumen de transacciones:

   | Plan | Volumen | Precio |
   |---|---|---|
   | Trader | hasta un límite gratuito | 0 € |
   | Pro-Trader (plan destacado) | hasta un límite alto | 49,99 € |
   | High Freq Trader | ilimitado | 99,99 € |

   Los límites, el periodo que cubre cada precio y su relación con las duraciones de 1, 3 y 5 años son
   preguntas abiertas. La duración plurianual encaja con el periodo durante el que un contribuyente
   necesita conservar y poder rehacer sus cálculos (afirmación 11).
4. **Qué se limita y qué no.** Todos los planes tienen las mismas capacidades (agente, grafo, copias en
   Drive, exportaciones). El único eje es el volumen, y la tabla plan → límite vive en un único fichero.
   La corrección fiscal no depende del plan: al superar el límite todo se sigue importando y calculando
   completo, sin truncar; lo que exige el plan correspondiente es cerrar y exportar el informe del
   ejercicio (comportamiento propuesto, pendiente de confirmar).
5. **Servicio de licencias mínimo.** Emite y renueva licencias; guarda correo, plan y fechas; nunca recibe
   datos fiscales. Corre en un plan gratuito que admita uso comercial (afirmación 4).
6. **Cobro propio en criptoactivos**, en wallets del usuario por red (las redes y direcciones las fija el
   usuario):
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

- El desarrollo no se ve afectado: con el proveedor `dev` no existe ninguna comprobación de licencia.
- Cambiar un límite de volumen es editar una tabla, sin tocar ningún otro módulo.
- Vender acceso es una actividad económica con obligaciones propias (alta, IVA, condiciones de uso,
  RGPD): se revisan con un asesor antes de la etapa 2.

## Preguntas abiertas para el usuario

1. Límite de transacciones del plan Trader y del plan Pro-Trader.
2. Unidad de cómputo del límite: transacciones por ejercicio fiscal o acumuladas.
3. Periodo que cubren 49,99 € y 99,99 € (un ejercicio, un año natural, pago único) y encaje con las
   duraciones de 1, 3 y 5 años.
4. Confirmación del comportamiento al superar el límite (punto 4).
5. Redes, activos y wallets de cobro.
6. Nombre y marca del producto.
