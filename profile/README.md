# Planasa Digital Apps

Org de las aplicaciones digitales internas de Planasa. Aquí viven los repositorios de producto, herramientas y plantillas que el equipo de Digital Apps mantiene.

## Qué hay dentro

- **Aplicaciones de producto** — repos por dominio (expediciones, partes, GMAO, CRM, Quality, etc.).
- **`claude-sanbox`** — sandbox + plantilla canónica de Claude Code (skills, rules, agents, hooks, safeguards). Cualquier proyecto nuevo que adopte Claude Code parte de aquí. Ver [ADR 0002](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox/blob/master/docs/adr/0002-reusable-template-baseline.md).
- **`.github`** (este repo) — defaults org-wide: reusable workflows, catálogo declarativo de labels, community files heredados. Ver [ADR 0004](https://github.com/Planasa-Digital-Apps-Organization/.github/blob/master/docs/adr/0004-org-github-visibility-public.md).

## Doctrina

- **Branching**: git-flow lite, master-based. Toda short-lived branch nace de `master`. `develop` es vehículo de release. Detalle en el [runbook](https://github.com/Planasa-Digital-Apps-Organization/.github/blob/master/docs/runbooks/git-flow.md).
- **Commits**: Conventional Commits + sufijo JIRA `[<PROJ>-<NNN>]`. Tipos: `feat | fix | docs | style | refactor | test | chore`. Prefijos JIRA permitidos: `BCR | BTS | ID | MAM | GIT`.
- **PRs**: plantilla heredada de este repo (`## Summary / ## Why / ## Test plan`). Si tocan paths sensibles (`.claude/**`, `CLAUDE.md`, `AGENTS.md`, `.github/**`) → sección `## Notes` obligatoria.

## Puesta en marcha de un repo nuevo

1. Copia `.claude/`, `AGENTS.md`, `.github/CODEOWNERS`, `.github/pull_request_template.md` y `docs/{adr,runbooks}/` desde `claude-sanbox`.
2. Adopta el reusable workflow `sensitive-paths-check.yml` añadiendo el caller stub a tu repo (snippet en el [README de este `.github`](https://github.com/Planasa-Digital-Apps-Organization/.github)).
3. Ajusta `.github/CODEOWNERS` con el team del proyecto.
4. Configura branch protection / rulesets en `master` y `develop`.
