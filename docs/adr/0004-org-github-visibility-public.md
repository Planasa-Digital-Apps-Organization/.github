# 0004. Adopt `<org>/.github` (public) as org-wide source of truth for reusable workflows, label catalog and community files

Date: 2026-05-29
Status: Proposed

## Context

ADR [0002](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox/blob/master/docs/adr/0002-reusable-template-baseline.md) estableció `claude-sandbox` como template canónico de `.claude/` y safeguards. La primera adopción real (workflow `sensitive-paths-check.yml` con label `area:ai-behavior`) puso de manifiesto dos límites del modelo per-repo:

1. La label `area:ai-behavior` no existe automáticamente en los demás repos del org — habría que crearla a mano por repo con riesgo de drift.
2. La lógica del workflow (label + comment) se duplicaría en cada repo que lo adopte. Cambiar el checklist obligaría editar 8+ ficheros idénticos.

Lo mismo aplica a community files (issue templates, PR template, SECURITY.md, profile/README): si cada repo los define localmente, hay duplicación y drift.

GitHub ofrece un mecanismo nativo para esto: un repositorio especial `<org>/.github` cuyos community files y reusable workflows actúan como defaults para el resto del org. Lo que no estaba claro a priori es qué **visibility** debe tener ese repo.

Alternativas consideradas para la visibility:

- **Private**: máximo aislamiento, pero la herencia desde private `.github` solo aplica a repos privados y es históricamente flaky en GitHub. Descartado.
- **Internal**: inicialmente la opción más natural (alineada con el resto de repos del org). Se montó así y se verificó empíricamente que **no propaga issue templates** a repos hijos pese a estar correctamente configurado:
  - `expediciones-app` con un `.github/ISSUE_TEMPLATE/bug_report.md` propio renderizaba bien el dropdown en `/issues/new/choose`.
  - El mismo fichero como default en `<org>/.github` (internal) no aparecía en ningún repo hijo.
  - Confirmado: el problema es exclusivamente herencia desde internal, no formato del fichero. Es un comportamiento no documentado oficialmente por GitHub pero ampliamente reportado en la community.
- **Public**: la herencia funciona de forma fiable y completa (SECURITY.md, templates, profile/README, reusable workflows). Verificado tras el cambio de visibility con `crm-app`: muestra el dropdown de issue templates heredados sin tener nada local. El trade-off es que workflows + `labels.yml` quedan visibles públicamente — sin secrets dentro, los push rulesets enterprise no se pierden porque no había configurados, y el contenido es doctrina genérica.

## Decision

`Planasa-Digital-Apps-Organization/.github` es el repositorio org-wide source of truth para:

- **Reusable workflows** (`.github/workflows/*.yml`) — invocados por callers per-repo via `uses: Planasa-Digital-Apps-Organization/.github/.github/workflows/<name>.yml@master`.
- **Catálogo de labels** (`.github/labels.yml`) — declarativo, sincronizado a todos los repos no-archivados por `sync-labels.yml`. Requiere org secret `LABEL_SYNC_TOKEN` (fine-grained PAT, Issues:write + Metadata:read).
- **Community files defaults** (`.github/SECURITY.md`, `.github/pull_request_template.md`, `.github/ISSUE_TEMPLATE/*`, `profile/README.md`) — heredados automáticamente por los repos del org sin override propio.

**Visibility: public.** Validado empíricamente con `crm-app` después del cambio.

Convención de credenciales: un PAT por workflow capability, secret nombrado `<CAPABILITY>_TOKEN`, scope mínimo, almacenado como org secret con Repository access limitado al repo `.github`.

Convención de versionado: bootstrap usa `@master` (mutable). Cuando el contrato del workflow se estabilice, cortar tag `v1` y migrar callers a `@v1` para aislar breaking changes.

## Consequences

- **Positivo**:
  - Single source of truth para labels, lógica del workflow, y community files. Un cambio aquí propaga al siguiente run / a la siguiente visita en UI.
  - Onboarding de repo nuevo: caller stub de 8 líneas YAML + bootstrap desde `claude-sanbox` (paso a paso en el README del `.github`).
  - PAT scope sigue least-privilege por capability — token comprometido = blast radius acotado.
  - SECURITY.md y profile/README quedan visibles a usuarios externos — útil para reports de seguridad y como tarjeta de visita.

- **Negativo**:
  - Workflows reusables, `labels.yml` y patterns operacionales son visibles públicamente. Riesgo aceptado: no contienen secrets ni código de producto.
  - Cada capability nueva añade un PAT que hay que rotar. Pendiente: rotación trimestral disciplinada o migración a GitHub App cuando se acumulen ≥3 PATs.
  - Gotcha de fine-grained PAT: al crearlo hay que elegir Resource owner = **org**, no = user. Equivocarse hace que `gh repo list <org>` devuelva entradas vacías con metadata redactada (símbolo silencioso de mala scope). Documentado en el README del `<org>/.github` y los workflows tienen diagnostics que detectan el caso.
  - Callers usan `@master` (mutable). Cualquier cambio en el reusable afecta inmediatamente a todos los callers — riesgo de breakage no anunciado hasta que cortemos `v1`.
  - Enterprise ruleset `Aa Branch General Naming Protection` (id 17021935) bloquea `revert-*` (las ramas auto-generadas por GitHub al revertir desde UI). Workaround documentado: hacer revert manual via feature branch nombrada `feature/<JIRA>-<NNN>-revert-...`. Solución de fondo: ajustar el ruleset para excluir `revert-*` y `dependabot/*`.
  - Internal visibility se exploró antes y se descartó tras prueba empírica — la decisión queda registrada aquí para no re-litigar el debate.

- **Abierto**:
  - Migración a GitHub App cuando el número de PATs supere 3.
  - Estrategia de tagging del reusable (`v1` semver-ish vs. tag por workflow individual).
  - Política de overwrite del sync de labels: actualmente `gh label edit` sobrescribe color/description si difiere — mata drift, también mata personalizaciones legítimas per-repo. Decidir si introducir un campo `protected: true` en `labels.yml` o asumir overwrite total.
  - Próximas capabilities probables a montar bajo el mismo patrón: PR title lint reusable, auto-labeler por path, dependency review, sync de CODEOWNERS templates. Cada una en su ADR cuando llegue.
