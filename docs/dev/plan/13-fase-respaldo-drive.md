# Fase 13 — Copias en el Drive del usuario

**Resultado:** el usuario guarda copias cifradas de su base y sus informes en su propio Drive y puede
restaurarlas. Diseño: `../arquitectura/06-adr-almacenamiento-drive-usuario.md`.

## Puntos

1. **Verificación previa** de las afirmaciones 12 y 13 del registro.
2. **Interfaz de almacenamiento** `rotkehlchen/fork/storage/` y proveedor de carpeta local.
3. **Instantánea consistente** de la base cifrada, con versión de esquema y huella de integridad.
4. **Retención** configurable y limpieza de copias antiguas.
5. **Restauración** con verificación previa y copia local de la base sustituida.
6. **Exportación de informes** a la carpeta de copias.
7. **Proveedor Google Drive** con OAuth y ámbito `drive.file` (necesario para el acceso de terceros; el
   usuario en desarrollo usa el proveedor de carpeta local).
8. **Tareas en segundo plano** con progreso y cancelación (ADR 07).

## Tests

- Copia y restauración de una base sintética: el estado restaurado es idéntico al copiado.
- Una copia con la huella alterada es rechazada y la base actual queda intacta.
- La retención elimina exactamente las copias que su política indica.
