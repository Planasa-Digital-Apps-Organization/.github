# Planasa Digital Apps — Organization defaults

Repositorio especial `.github` para
[Planasa-Digital-Apps-Organization](https://github.com/Planasa-Digital-Apps-Organization).
Centraliza los _defaults_ que toda la organización hereda: reusable
workflows, community files, catálogo de labels y plantillas de issues/PRs.

Mantenedor: **Planasa Digital Apps**. Las decisiones arquitectónicas que
respaldan este repo viven como ADRs en
[`claude-sanbox/docs/adr/`](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox/tree/master/docs/adr).

## Contenido

| Categoría | Path | Función |
| --- | --- | --- |
| **Landing pública** | `profile/README.md` | Página del org (`github.com/<org>`). Tarjeta de visita. |
| **Catálogo de labels** | `.github/labels.yml` | Source of truth declarativo de labels org-wide. |
| **Sync de labels** | `.github/workflows/sync-labels.yml` | Propaga `labels.yml` a todos los repos no-archivados. Triggers: push a `master` con cambios, schedule semanal, `workflow_dispatch`. |
| **Reusable: sensitive paths** | `.github/workflows/sensitive-paths-check.yml` | `workflow_call`. Etiqueta `area:ai-behavior` + checklist comment en PRs que tocan paths sensibles. Invocada por stub per-repo. |
| **Community defaults** | `.github/SECURITY.md`, `.github/pull_request_template.md`, `.github/ISSUE_TEMPLATE/*` | Heredados automáticamente por todos los repos del org que no tengan los suyos propios. |

## Setup de un proyecto nuevo basado en `claude-sanbox`

`claude-sanbox` es el template canónico (ADR
[0002](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox/blob/master/docs/adr/0002-reusable-template-baseline.md)).
Para bootstrappear un repo nuevo:

### 1. Crear el repo

```bash
gh repo create Planasa-Digital-Apps-Organization/<nombre-proyecto> \
  --internal \
  --description "<descripción>"
```

> Visibility `internal` es el default para producto. Public solo cuando
> hay justificación documentada.

### 2. Copiar el baseline desde `claude-sanbox`

Clona ambos repos y copia los directorios/archivos canónicos:

```bash
# Desde la raíz del repo nuevo, clonado vacío:
git clone https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox.git /tmp/claude-sanbox

cp -r /tmp/claude-sanbox/.claude .
cp /tmp/claude-sanbox/AGENTS.md .
cp /tmp/claude-sanbox/CLAUDE.md .
cp -r /tmp/claude-sanbox/.github .
cp -r /tmp/claude-sanbox/docs .
cp /tmp/claude-sanbox/.gitignore .

rm -rf /tmp/claude-sanbox
```

### 3. Ajustes específicos del proyecto

- **`.github/CODEOWNERS`** — sustituir `@mescartin` por el team responsable
  del proyecto (e.g. `@Planasa-Digital-Apps-Organization/<team-slug>`).
- **`.claude/settings.json`** — añadir entradas al allowlist específicas
  del proyecto. Ejemplos:
  - `Bash(gh api repos/Planasa-Digital-Apps-Organization/<nombre-proyecto>:*)`
  - Comandos de build/test propios del stack (ej. `Bash(flutter test:*)`).
- **`CLAUDE.md`** — sección _Purpose_ específica del proyecto al inicio;
  el resto de la doctrina (ramas, commits, skills) se mantiene.

### 4. Workflows del repo nuevo

Añadir al menos el caller stub del `sensitive-paths-check`:

```yaml
# .github/workflows/sensitive-paths-check.yml
name: Flag sensitive paths
on:
  pull_request:
    paths:
      - '.claude/**'
      - 'CLAUDE.md'
      - 'AGENTS.md'
      - '.github/**'
      - '.gitignore'
permissions:
  contents: read
  pull-requests: write
jobs:
  flag:
    uses: Planasa-Digital-Apps-Organization/.github/.github/workflows/sensitive-paths-check.yml@master
```

### 5. Branch protection y rulesets

- Confirmar que los enterprise rulesets aplican al repo (heredados
  automáticamente).
- Configurar branch protection adicional si el proyecto lo requiere
  (e.g. status checks específicos del CI del proyecto).
- `master` y `develop` deben requerir _Code Owners review_.

### 6. Primer commit y push

Convención de bootstrap (ver ADR 0003 y
[`docs/runbooks/git-flow.md`](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox/blob/master/docs/runbooks/git-flow.md)):

```bash
git checkout -b feature/<JIRA>-<NNN>-bootstrap-template
git add .
git commit -m "chore(template): bootstrap from claude-sanbox [<JIRA>-<NNN>]"
git push -u origin feature/<JIRA>-<NNN>-bootstrap-template
gh pr create --base develop --title "..." --body "..."
```

## Secret necesario

`.github/workflows/sync-labels.yml` requiere el org secret
`LABEL_SYNC_TOKEN`. Configurado en `Settings → Secrets and variables →
Actions` de la organización, con `Repository access` restringido a este
repo.

PAT requerido: **fine-grained**, _Resource owner_ = la organización,
_Repository access_ = All repositories, permisos:

- Repository → Metadata: Read
- Repository → Issues: Read and write

Rotación recomendada: trimestral. Más detalle sobre el modelo de
credenciales en ADR 0003.

## Visibility

Este repo es **public**. La decisión y el porqué están en ADR
[0004](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox/blob/master/docs/adr/0004-org-github-visibility-public.md):
con `internal`, la herencia de issue templates no propaga a los repos
hijos (limitación de GitHub no documentada oficialmente).

Contenido visible públicamente:

- Workflows reusables (`.github/workflows/`)
- Catálogo de labels (`.github/labels.yml`)
- Community files (templates de issues/PR, SECURITY.md)
- Esta documentación

**No** se expone:

- Secretos (siguen en el org secret store).
- Código de producto (vive en repos privados/internal del org).

## Versionado de los reusable workflows

Hoy los callers referencian `@master`:

```yaml
uses: Planasa-Digital-Apps-Organization/.github/.github/workflows/<name>.yml@master
```

Esto significa que cualquier cambio en el reusable afecta inmediatamente
a todos los callers. Cuando el contrato de cada workflow se estabilice,
cortaremos tags `v1`/`v2` y los callers migrarán a `@v1`. Pendiente.

## Contribuir cambios a este repo

Todo cambio sigue el flujo estándar Planasa Digital Apps:

1. Branch `feature/<JIRA>-<NNN>-<slug>` o `hotfix/<JIRA>-<NNN>-<slug>`
   desde `master`.
2. PR a `develop` (cambios no urgentes) o `release/*` (cherry-pick para
   release en curso).
3. Review humano obligatorio (CODEOWNERS).
4. Squash merge.

Cambios que tocan paths bajo `.github/workflows/**` o `labels.yml`
disparan el workflow `sensitive-paths-check.yml` automáticamente —
checklist obligatorio en el body del PR.
