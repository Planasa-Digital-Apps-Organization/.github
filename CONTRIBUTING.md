# Contribuir a Planasa Digital Apps

Esta guía aplica a cualquier repositorio de la
[Planasa Digital Apps Organization](https://github.com/Planasa-Digital-Apps-Organization)
que no tenga su propio `CONTRIBUTING.md`. Cubre el flujo de PR estándar
para cambios de código.

Para roles y gobernanza ver [`GOVERNANCE.md`](.github/GOVERNANCE.md).
Para canales de soporte y dónde plantear cada tipo de petición ver
[`SUPPORT.md`](.github/SUPPORT.md).

## ¿Primera contribución? Empieza aquí

→ **[Guía paso a paso para tu primer PR](docs/contributing/getting-started.md)**

End-to-end desde clonar el repo hasta mergear, con comandos copy-paste,
ejemplos buenos y errores comunes.

## Referencia rápida

| Tema | ✅ Do | ❌ Don't | Detalle |
| --- | --- | --- | --- |
| Ramas | `feature/BCR-1234-add-customer-export` | `feature/customer-export`, `bcr1234`, `marcos/wip`, `dev` | [git-flow runbook](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox/blob/master/docs/runbooks/git-flow.md) |
| Commits | `feat(api): add customer export endpoint [BCR-1234]` | `Update files`, `WIP`, `feat: export` (sin JIRA) | [commit-style rule](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox/blob/master/.claude/rules/commit-style.md) |
| PRs | Título igual al subject del commit, base `develop`, `## Test plan` rellenado | PR a `master` directo, sin `## Test plan`, título genérico tipo "Update README" | [git-flow runbook](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox/blob/master/docs/runbooks/git-flow.md) |
| Reviews | Esperar approval de CODEOWNERS + 1 maintainer; responder comments antes de remerge | Auto-merge sin review, "trust me", ignorar comments sin réplica | [GOVERNANCE.md](.github/GOVERNANCE.md) |

Prefijos JIRA permitidos: `BCR`, `BTS`, `ID`, `MAM`, `GIT`.

## Reportar bugs y proponer features

- **Bug**: abre un issue con la plantilla _Bug report_.
- **Feature**: abre un issue con la plantilla _Feature request_.
- **Vulnerabilidad de seguridad**: **no abras issue público**. Sigue
  [`SECURITY.md`](.github/SECURITY.md).
- **Duda no urgente o de soporte**: ver [`SUPPORT.md`](.github/SUPPORT.md)
  para el canal apropiado.

## Reviews

Todos los PRs requieren review humano. `CODEOWNERS` routea automáticamente
a los responsables del path afectado.

Lo que se espera en cada PR:

- Título y commit subject según _Referencia rápida_ (Conventional Commits
  con sufijo `[JIRA-NNN]`).
- `## Summary`, `## Why`, `## Test plan` rellenados — la plantilla la
  heredas automáticamente.
- Si tocas `.claude/**`, `CLAUDE.md`, `AGENTS.md`, `.github/**` o
  `.gitignore`: sección adicional `## Notes` explicitando el cambio. El
  workflow `Flag sensitive paths` etiquetará el PR con `area:ai-behavior`
  y dejará un checklist como comment.

Las ramas `master`, `develop` y `release/*` están protegidas por enterprise
rulesets. Sólo se mergea con la review aprobada.

## Política de idioma

El criterio es **la audiencia y la capa del contenido, no la extensión del
archivo**. Tres capas:

### Capa 1 — Documentación humana y políticas

`README.md`, `profile/README.md`, este `CONTRIBUTING.md`,
`docs/contributing/*`, y los community files de `.github/`
(`CODE_OF_CONDUCT.md`, `GOVERNANCE.md`, `SECURITY.md`, `SUPPORT.md`).

→ **Español como lengua base**, incluidos los encabezados. La prosa es
española; los tecnicismos siguen las reglas de terminología de abajo.

### Capa 2 — Superficies que rellena o lee un contribuidor

Plantillas de issue, plantilla de PR, comentario del bot `sensitive-paths`,
el `about` de `ISSUE_TEMPLATE/config.yml`.

→ **Prosa de ayuda, placeholders y descripciones en español.** El inglés se
reserva para:

- **Encabezados estructurales `##`** (`## Summary`, `## Why`, `## Notes`,
  `## Observed behavior`, …). Actúan como contrato citado por la doctrina y
  por el workflow, así que se mantienen estables en inglés.
- **Tokens de convención**: prefijos de commit (`feat:`, `fix:`), nombres de
  label (`type:bug`, `area:ai-behavior`).

### Capa 3 — Ficheros de máquina e infraestructura

`labels.yml`, `.github/workflows/*.yml`, código JS inline.

→ **Inglés.** Es el idioma del ecosistema GitHub Actions y se lee en contexto
de código. **Excepción**: cualquier string que GitHub renderice a una persona
(p. ej. el cuerpo del comentario del bot o el `about` de un contact link) sube
a Capa 2 y va en español.

## Reglas de terminología (prosa española, Capas 1 y 2)

- **Sustantivos-producto de Git/GitHub/CI se mantienen en inglés**, sin
  traducir ni cursiva: issue, pull request / PR, branch, merge, label,
  workflow, ruleset, commit, secret, runbook, advisory.
- **Identificadores de código, CLI y paths nunca se traducen ni se flexionan**:
  `master`, `develop`, `workflow_dispatch`, `.github/`.
- **Nada de verbos spanglish.** No "bootstrappear / mergear / commitear". Usar
  *verbo español + sustantivo inglés*: "inicializar el repo", "hacer merge"
  o "fusionar", "crear un commit", "crear un tag".
- **Acrónimos en mayúscula tal cual**: ADR, PR, CI, SLA, RCE, PAT.
- **Si existe término español asentado y sin ambigüedad, úsalo en prosa**:
  "plantilla" en lugar de "template", salvo cuando "issue template" sea el
  nombre propio de la feature de GitHub.

## Código de conducta

Se aplica el [`CODE_OF_CONDUCT.md`](.github/CODE_OF_CONDUCT.md) heredado
(adaptado del Contributor Covenant v2.1).

## ¿Dudas?

- Email directo: **`mescartin@planasa.com`**
- Canal genérico (Slack/Teams `#digital-apps`): **pendiente de definir**.
- Dudas estructurales sobre doctrina, branching o convenciones:
  [runbooks y ADRs](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox/tree/master/docs)
  en `claude-sanbox`.
