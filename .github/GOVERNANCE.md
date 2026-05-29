# Gobernanza

Este documento se aplica a todos los repositorios de la
[Planasa Digital Apps Organization](https://github.com/Planasa-Digital-Apps-Organization)
que no definan un `GOVERNANCE.md` propio, y describe cómo se toman las decisiones
y se aprueban los cambios.

## Roles

| Rol | Quién | Responsabilidad |
| --- | --- | --- |
| **Administradores del org** | Equipo de Digital Apps (IT) | Configuración de la organización: secrets, rulesets, teams, herencia de community files. |
| **`CODEOWNERS`** | Definidos por repo en `.github/CODEOWNERS` | Revisión obligatoria de los paths que les corresponden antes de mergear. |
| **Mantenedores** | Equipo asignado a cada producto | Triage de issues, revisión de PRs, releases. |
| **Colaboradores** | Cualquier miembro del org | Proponen cambios vía PR siguiendo la doctrina. |

## Cómo se aprueban los cambios

El flujo estándar para **todos** los repos (incluido este `.github`):

1. Rama corta nacida de `master` (`feature/* | hotfix/* | release/*`).
2. Pull Request con la plantilla heredada (`## Summary / ## Why / ## Test plan`).
3. Revisión de los `CODEOWNERS` afectados; los paths sensibles
   (`.claude/**`, `CLAUDE.md`, `AGENTS.md`, `.github/**`, `.gitignore`) la
   requieren siempre.
4. Merge sólo con la branch protection / ruleset satisfecha.

Detalle del branching en el
[runbook git-flow](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox/blob/master/docs/runbooks/git-flow.md).

## Decisiones de arquitectura (ADR)

Las decisiones estructurales se registran como ADR en
[`claude-sanbox/docs/adr/`](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox/tree/master/docs/adr).
Una vez en estado **Accepted**, un ADR no se reescribe: sólo cambia su `Status`
(p. ej. a _Superseded_ por un ADR posterior). Relevantes para este repo:

- **ADR 0002** — `claude-sanbox` como plantilla canónica reutilizable.
- **ADR 0003** — este repo `.github` como hogar de los reusable workflows y
  defaults org-wide.

## Qué se hereda desde este repo

GitHub propaga automáticamente a cualquier repo que no tenga el suyo propio:
`CODE_OF_CONDUCT.md`, `SECURITY.md`, `SUPPORT.md`, este `GOVERNANCE.md`, las
plantillas de issues y el `pull_request_template.md`. Además se distribuyen el
catálogo de labels (`sync-labels.yml`) y los reusable workflows. El `CODEOWNERS`
**no** se hereda: cada repo define el suyo.

## Decisiones conscientes sobre community files

- **`CONTRIBUTING.md`**: pendiente. La doctrina de contribución (Conventional
  Commits + sufijo JIRA, branching) vive de momento en el
  [perfil del org](../profile/README.md) y en los runbooks de `claude-sanbox`.
- **`FUNDING.yml`**: no se incluye. La organización es interna y no acepta
  patrocinio.
- **`SECURITY.md`**: en borrador. El buzón de contacto y los SLA están
  pendientes de ratificación por el equipo; hasta entonces se mantienen como
  placeholders marcados en el propio fichero.
