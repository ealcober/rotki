# Guía del fork para humanos y LLMs

Este repositorio es el fork personal de [rotki](https://github.com/rotki/rotki) mantenido por `ealcober`
(`github.com/ealcober/rotki`). Añade sobre rotki una capa orientada a la **fiscalidad española de
criptoactivos**: importación tolerante de CSV, conciliación asistida de wallets y rendimientos, informe
fiscal para el IRPF (modelo 100) y datos para el modelo 721, visualización de flujos en grafo y una capa
agéntica sobre el servidor MCP de rotki.

Este documento es el punto de entrada. Léelo entero antes de tocar nada.

## Qué es upstream y qué es del fork

| Ámbito | Dueño | Dónde vive |
|---|---|---|
| Código y docs de rotki | upstream (`rotki/rotki`) | todo el árbol salvo lo listado abajo |
| Guía del fork, ADRs, planes, bitácora | fork | `docs/dev/` |
| Instrucciones de agente de upstream | upstream | `CLAUDE.md`, `AGENTS.md` (el fork solo añade una línea de enlace a este documento) |
| Código propio del fork | fork | módulos nuevos con prefijo `fork_` o directorios `fork/` dentro del paquete afectado (ver «Convenciones») |
| Datos personales (CSV, exports, claves) | usuario | `data/privado/` — ignorado por git, **nunca** se commitea |

Las instrucciones de upstream (`CLAUDE.md` / `AGENTS.md` en la raíz) siguen siendo obligatorias para todo
el código: comandos, linters, convenciones de Vue/TypeScript, tests y política de VCR.

## Ramas y remotos

| Rama | Papel |
|---|---|
| `develop` | espejo de `upstream/develop`; no recibe commits propios |
| `fork/main` | rama de trabajo del fork; parte de `develop` |
| `fork/<tema>` | ramas de trabajo cortas que se integran en `fork/main` |

Remotos: `origin` = `ealcober/rotki`, `upstream` = `rotki/rotki`.

Sincronización con upstream:

```bash
git fetch upstream
git switch develop && git merge --ff-only upstream/develop && git push origin develop
git switch fork/main && git merge develop   # los conflictos se resuelven a favor de upstream salvo en código fork_
```

Lo que sirve a cualquier usuario de rotki (por ejemplo, las regiones de Bybit) se desarrolla en una rama
limpia desde `develop` para poder abrir un PR contra upstream sin arrastrar la capa del fork.

## Reglas de trabajo

Aplican las reglas de usuario (`~/.claude/CLAUDE.md`) y, además, estas propias del fork:

1. **Privacidad primero.** El repositorio es público (los forks de repos públicos en GitHub lo son). Ningún
   dato personal entra en git: ni CSV, ni direcciones de wallet propias, ni IBAN, ni nombres. Los formatos
   de exchange se documentan con **cabeceras y tipos**, nunca con filas reales. Los tests usan datos
   sintéticos.
2. **Encajar antes que construir.** Antes de crear un módulo, se localiza el equivalente en rotki
   (`docs/dev/02-mapa-codigo-rotki.md`). Se extiende lo que existe; un sistema paralelo necesita un ADR.
3. **Cambios mínimos en ficheros de upstream.** Cada línea tocada en un fichero de upstream es un conflicto
   potencial en la próxima sincronización. Se prefieren puntos de extensión (registros, decoradores, nuevos
   ficheros) a editar ficheros existentes.
4. **Coste cero.** Ninguna pieza de la arquitectura puede exigir un pago recurrente nuevo
   (`docs/dev/arquitectura/01-adr-despliegue-coste-cero.md`).
5. **Toda cifra fiscal es trazable.** Cada importe del informe enlaza con los eventos que lo producen. Un
   importe sin rastro hasta sus eventos es un defecto.
6. **Ninguna afirmación fiscal sin fuente.** Las reglas fiscales citan la norma o la consulta vinculante de
   la DGT. Lo que no tiene fuente va al registro de `docs/dev/04-registro-afirmaciones.md`.
7. **La interfaz responde siempre.** Toda operación cuyo coste crece con el historial corre en segundo
   plano, con progreso y cancelación, y todo listado se pagina en el servidor
   (`docs/dev/arquitectura/07-adr-escala-y-respuesta.md`).
8. **Nada se decide en silencio.** Lo que la herramienta no sabe catalogar, o la información que falta, se
   pregunta al usuario explicando por qué importa; la respuesta y su motivo quedan en el registro de
   decisiones y se reflejan en el informe (`docs/dev/arquitectura/08-adr-clasificacion-y-decisiones.md`).
9. **Toda comisión se computa una vez y en su sitio.** Cada comisión es un evento ligado a su operación,
   con el tratamiento de `docs/dev/arquitectura/09-adr-comisiones.md`.
10. **Acceso por etapas.** Durante el desarrollo la instancia es de un único usuario y no se expone a
    Internet; el acceso de terceros y los planes de pago llegan con
    `docs/dev/arquitectura/05-adr-monetizacion-y-acceso.md`.

## Convenciones

- Documentos: `docs/dev/**/NN-slug.md`, numeración por carpeta. En `docs/dev/plan/` la numeración empieza en
  `00` (el plan maestro); en el resto de carpetas, en `01`.
- Código Python propio: `rotkehlchen/<paquete>/fork_<tema>.py` o `rotkehlchen/fork/<tema>/`.
- Herramientas MCP propias: `rotkehlchen/mcp/tools/fork_<tema>.py`, registradas con `@register_tool`.
- Frontend propio: `frontend/app/src/modules/fork-<tema>/`, con las mismas convenciones que el resto de
  `modules/`.
- Textos de interfaz: en las locales existentes (`frontend/app/src/locales/`), con español completo.

## Mapa de la documentación del fork

| Documento | Contenido |
|---|---|
| `01-guia-fork-llm.md` | este documento |
| `02-mapa-codigo-rotki.md` | módulos de rotki sobre los que se construye, con fichero:línea |
| `03-formatos-csv.md` | formatos de exportación de exchanges y brokers (solo cabeceras) |
| `04-registro-afirmaciones.md` | afirmaciones sin verificar y afirmaciones caídas |
| `05-bitacora.md` | bitácora fechada del fork |
| `arquitectura/` | ADRs 01–09: despliegue, capa agéntica, importación, visualización, monetización y acceso, almacenamiento en Drive, escala, clasificación y decisiones, comisiones |
| `plan/00-plan-maestro.md` | plan de desarrollo completo y orden de fases |
| `plan/NN-fase-*.md` | una fase por documento |

## Cómo arrancar una sesión de trabajo

1. Leer este documento, `plan/00-plan-maestro.md` y la fase en curso.
2. Leer las secciones de `CLAUDE.md` de upstream que afectan a lo que se va a tocar.
3. Comprobar `git status`, rama `fork/main` (o la rama de tema) y sincronía con `origin`.
4. Trabajar la fase con `/challenge-ampliar` (ampliar puntos) o `/challenge-implementar` (implementar).
5. Actualizar la documentación con `/champliar-docs`.
