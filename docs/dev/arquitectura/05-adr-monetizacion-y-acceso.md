# ADR 05 — Monetización y control de acceso

- **Estado:** propuesto, pendiente de aprobación del usuario.

## Contexto

El proyecto tiene tres etapas de acceso y dos vías de ingreso posibles:

- **Desarrollo:** solo accede el usuario.
- **Después:** acceso de terceros por planes de pago de 1, 3 y 5 años (el de 3 años es el plan
  destacado), anuncios en la web pública, o ambas vías.

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

3. **Planes.** Duraciones de 1, 3 y 5 años; el de 3 años es el plan ancla: mejor precio por año que el de
   1 y presentado como recomendado. La duración plurianual encaja con el periodo durante el que un
   contribuyente necesita conservar y poder rehacer sus cálculos (afirmación 11). Precios: decisión del
   usuario.
4. **Qué se limita y qué no.** La corrección fiscal no depende del plan: ningún plan produce un cálculo
   peor. Lo que distingue planes son capacidades de conveniencia (agente, grafo, copias en Drive,
   exportación PDF, número de ejercicios abiertos). La tabla capacidad → plan vive en un único fichero.
5. **Servicio de licencias mínimo.** Emite y renueva licencias; guarda correo, plan y fechas; nunca recibe
   datos fiscales. Corre en un plan gratuito que admita uso comercial (afirmación 4).
6. **Cobro** mediante un intermediario que actúe como vendedor registrado y cobre por transacción
   (afirmación 14).
7. **Anuncios** solo en la web pública (fase 10), nunca dentro de la aplicación. Son compatibles con los
   planes: la web con anuncios es además el canal de captación.

## Consecuencias

- El desarrollo no se ve afectado: con el proveedor `dev` no existe ninguna comprobación de licencia.
- Añadir o mover una capacidad de plan es editar una tabla, sin tocar el módulo que la implementa.
- Vender acceso es una actividad económica con obligaciones propias (alta, IVA, condiciones de uso,
  RGPD): se revisan con un asesor antes de la etapa 2.

## Preguntas abiertas para el usuario

1. ¿Planes de pago, anuncios o ambos en la etapa pública?
2. ¿Qué capacidades distinguen los planes y cuáles son siempre gratuitas?
3. Precios de los planes de 1, 3 y 5 años.
4. Nombre y marca del producto.
