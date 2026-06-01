# Git flow lite — runbook

Procedimiento operativo para el modelo de ramas de los proyectos Planasa
Digital Apps. La doctrina autoritativa la enforza la skill
[`commit-flow`](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox/blob/master/.claude/skills/commit-flow/SKILL.md); este runbook es
la versión legible humana del día a día. **Ejecuta las acciones pidiéndoselas
a Claude en lenguaje natural** — la skill rechaza lo que no encaje.

## TL;DR

- **Modelo:** git flow lite, **master-based**.
- **Long-lived branches:** `master` (producción) + `develop` (vehículo de release).
- **Short-lived branches** (`feature/*`, `hotfix/*`, `release/*`): **siempre
  nacen de `master`**, nunca de `develop`.
- **Default release:** PR `develop → master`. Tag `vX.Y.Z`.
- **Plan B (release parcial):** construye `release/x.y.z` desde `master`,
  mergea sólo las `feature/*` confirmadas, PR `release/x.y.z → master`.
  Resetea `develop` a `master` después.
- **Skills activas:** `commit-flow` (siempre) + `commit-flow-flutter`
  (auto-activa si hay `pubspec.yaml`).
- **Herramientas que usa el modelo:** sólo `git merge --no-ff` y `git pull`.
  No rebase, no cherry-pick, no squash por defecto.

## Prefijos JIRA permitidos

Lista cerrada (hardcoded en `commit-flow`):

```
BCR | BTS | ID | MAM | GIT
```

Cuando aparezca un proyecto nuevo, hay que añadir el prefijo a la skill.
Roadmap: sustituir hardcoded por una consulta al Atlassian MCP
(`mcp__claude_ai_Atlassian_Rovo__authenticate`) que liste proyectos
accesibles al usuario.

## Día a día

### 1. Empezar una feature nueva

> *"empieza el ticket BCR-1234 — añadir empty state al carrito"*

`commit-flow` hará:
1. Validar que `BCR` está en la lista de prefijos permitidos.
2. Verificar que `HEAD` está en `master` y actualizado; `git pull` si no.
3. Proponer `feature/BCR-1234-cart-empty-state` y crearla tras tu OK.

Si propones un nombre no conforme, la skill **para y ofrece corrección** —
nunca renombra silenciosamente.

### 2. Commit de trabajo

> *"haz commit"*

`commit-flow` hará:
1. `git status` y `git diff --staged`.
2. Stagear paths explícitos (nunca `git add -A`).
3. Extraer `BCR-1234` del nombre de rama.
4. Elegir `<type>` según la naturaleza del cambio (`feat | fix | refactor | docs | …`).
5. Componer `<type>(<scope>): <description> [BCR-1234]`.
6. Si `commit-flow-flutter` está activa: correr `dart format --set-exit-if-changed`,
   `flutter analyze`, y los pattern guards (no `print(` en `lib/`, no `late?`,
   no `TODO` sin JIRA, no business logic en `build`).
7. Mostrar el mensaje y commitear.
8. Footer `Co-Authored-By: Claude` cuando Claude escribió la mayoría
   sustantiva del cambio.

### 3. Abrir el PR de integración a `develop`

> *"abre PR a develop"*

`commit-flow` delega en la skill [`pr-body`](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox/blob/master/.claude/skills/pr-body/SKILL.md)
para redactar el cuerpo. Hará:
1. Confirmar que la rama está pusheada y trackeando un remoto.
2. Construir título y descripción (resumen + test plan + link a JIRA).
3. Target = `develop`.

Este es el momento de **integración**. CI/QA valida tu feature junto a sus
pares. **No es la release.**

### 4. Preparar una release

> *"prepara la release 1.5.0"*

`commit-flow` preguntará qué camino aplica:

- **Default — todo lo que hay en `develop` va a prod:**
  PR `develop → master`. Tag `v1.5.0` en `master` tras merge.

- **Plan B — `develop` contiene features que NO van a este deploy:**
  1. Crear `release/1.5.0` desde el `master` actual.
  2. Mergear sólo las `feature/*` confirmadas para este release (merges
     limpios porque `release/1.5.0` y cada `feature/*` comparten `master`
     como ancestro).
  3. PR `release/1.5.0 → master`. Tag `v1.5.0`.
  4. **Tarea pendiente manual:** resetear `develop` a `master` y re-mergear
     las features aún en vuelo. La skill te lo recuerda; **no** ejecuta el
     reset automáticamente.

Regla de decisión: *"si `develop` == lo que va a prod → Default; si no →
Plan B"*.

### 5. Hotfix urgente

> *"hotfix MAM-9 — el TOTP falla por desfase de reloj"*

`commit-flow` hará:
1. Crear `hotfix/MAM-9-totp-clock-skew` desde el `master` actual.
2. Tras tu fix y commit, abrir PR `hotfix/MAM-9-… → master`.
3. Tras merge, recordarte que hagas **forward-merge `master` → `develop`**
   para que QA en `develop` refleje producción.

## Safeguards — lo que las skills rechazan

- Branchear desde `develop`, desde otra `feature/*` (excepto stacked
  declarado), o desde una rama release.
- Commitear directamente a `master`, `develop`, o `release/*` — todo vía PR.
- Pushear directamente a `master`, `develop`, o `release/*`.
- Usar `git add -A` o cualquier staging por defecto catch-all.
- Renombrar silenciosamente una rama no conforme.
- Rebasear o cherry-pickear silenciosamente.
- Editar código para que `dart format` / `flutter analyze` pase sin tu OK.

## Drift warnings que las skills emiten

- `feature/*` con más de **14 días** desde su base en `master` → sugiere
  rebase sobre el `master` actual (no lo ejecuta automáticamente).
- Dos `feature/*` abiertas modificando el mismo archivo → aviso de
  conflicto probable en integración. Sólo informativo.
- Tras release vía Plan B → recordatorio de resetear `develop` y
  re-mergear las features aún en vuelo.

## Cuando hay que salirse del modelo

Pide explícitamente y asume el override. Ejemplos:

- *"force push my feature, lo necesito"* → la skill avisa y confirma antes
  de correr `git push --force-with-lease`.
- *"haz cherry-pick del commit X a release/1.5.0"* → confirmado manualmente,
  caso a caso. La skill deja constancia del override en el mensaje
  (`Cherry-picked-from: <hash>`).

El default es siempre el camino seguro. Los overrides son explícitos y
visibles en la historia de git.

## Referencias externas que inspiraron este modelo

- [netresearch/git-workflow-skill](https://github.com/netresearch/git-workflow-skill)
- [conventional-jira-commits](https://mcpmarket.com/es/tools/skills/conventional-jira-commits)
- [KINTO Tech: Claude Code Jira Automation](https://blog.kinto-technologies.com/posts/2025-12-05-claude-code-jira-automation-en/)
