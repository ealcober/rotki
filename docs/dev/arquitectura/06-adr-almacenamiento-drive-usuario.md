# ADR 06 — Almacenamiento en el Drive del usuario

- **Estado:** propuesto, pendiente de aprobación del usuario.

## Contexto

El usuario quiere poder guardar su información en su propio Google Drive. La base de rotki es un fichero
SQLite cifrado por usuario en el equipo local. rotki ofrece su propia sincronización con sus servidores,
ligada a su suscripción (`rotkehlchen/premium/sync.py:76`); el fork no la toca.

## Decisión propuesta

**El Drive del usuario es el destino de copias cifradas y de informes; la base en uso vive siempre en
disco local.**

1. **Copias, no base en vivo.** La aplicación genera instantáneas consistentes de la base (API de backup
   de SQLite), ya cifradas con la clave del usuario, y las sube. La base abierta se mantiene fuera de
   carpetas sincronizadas (afirmación 13).
2. **Interfaz de almacenamiento** `rotkehlchen/fork/storage/` con proveedores intercambiables:

   | Proveedor | Etapa | Mecanismo |
   |---|---|---|
   | carpeta local | desarrollo | escribe la instantánea en una carpeta que el cliente de Drive del equipo ya sincroniza |
   | Google Drive | acceso de terceros | OAuth del usuario con ámbito `drive.file`: la app solo ve los ficheros que ella crea (afirmación 12) |

3. **Contenido de una copia:** instantánea cifrada, versión de esquema, huella de integridad y fecha.
   Retención configurable (por defecto, las N últimas y una por mes).
4. **Restauración** con verificación de integridad y de versión de esquema antes de sustituir nada; la
   base actual se conserva como copia local previa.
5. **Informes en Drive:** el informe fiscal (CSV, JSON, PDF) se puede exportar a la misma carpeta, con su
   anexo de decisiones.
6. **Tareas en segundo plano:** copia y restauración siguen las reglas del ADR 07 (progreso, cancelación).
7. **Agente:** el conector de Drive del cliente del usuario puede leer los informes exportados que el
   usuario elija; la base cifrada no es legible para el agente.

## Consecuencias

- El proveedor de la aplicación nunca almacena datos fiscales de usuarios: copias e informes están en el
  Drive de cada uno, cifradas con su clave.
- Añadir otro destino (otra nube, un NAS) es añadir un proveedor.
- Las copias en Drive están disponibles en todos los planes (ADR 05).
- Perder la clave de la base hace irrecuperables las copias; la interfaz lo advierte al activar la función.

## Preguntas abiertas para el usuario

1. Política de retención por defecto.
