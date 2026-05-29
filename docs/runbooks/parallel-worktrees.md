# Worktrees paralelos — runbook

Procedimiento operativo para trabajar en **varias ramas a la vez** del mismo
repo, cada una en su propia sesión de CLI (Claude Code o terminal normal), sin
`git stash` ni cambios de rama constantes. Complementa al runbook
[`git-flow.md`](./git-flow.md): el modelo de ramas no cambia, solo el *cómo*
las trabajas en paralelo.

## TL;DR

- Un **git worktree** = otro directorio de trabajo que comparte el mismo `.git`
  (historial, remotos, refs) pero tiene **su propia rama** checkout.
- Una sesión de CLI vive en **un** directorio → para N ramas en paralelo, N
  worktrees, un CLI en cada uno.
- **Vía recomendada (Opción A):** `claude --worktree <name>` — Claude Code crea
  y entra en el worktree por ti, con limpieza automática.
- **Vía manual (Opción B):** `git worktree add` + `claude` (o cualquier editor)
  cuando quieras controlar nombre de rama y ubicación.
- **Doctrina:** las ramas siguen naciendo de `master` y el naming sigue siendo
  obligatorio **al pushear** (lo enforza el ruleset, no es local).

## Cuándo usarlo

- Revisar un PR sin abandonar tu feature en curso.
- Un hotfix urgente mientras tienes una feature a medias.
- Comparar dos enfoques en ramas distintas a la vez.
- Lanzar tests largos en una rama mientras editas otra.

Si solo cambias de rama de vez en cuando, no necesitas worktrees: `git switch`
basta. Los worktrees son para **paralelismo real**.

---

## Opción A — `claude --worktree` (recomendada)

Claude Code tiene soporte propio de worktrees (no es solo un wrapper de git).

```bash
# Terminal 1 — feature de login
claude --worktree feature-login      # forma corta: claude -w feature-login

# Terminal 2 — en paralelo, otra cosa
claude --worktree bugfix-totp
```

Qué hace:

- Crea el worktree en `.claude/worktrees/<name>/` sobre una rama nueva
  `worktree-<name>` y **arranca Claude ya dentro**.
- La base por defecto es `origin/HEAD` (= `master` aquí), árbol limpio — lo que
  **encaja con la doctrina** "toda rama nace de `master`".
- **Limpieza automática al salir:** si el worktree no tiene commits, cambios ni
  untracked, se borra solo. Si tiene algo, te pregunta si conservarlo.
- Si omites el nombre, Claude genera uno (`bright-running-fox`).

Variantes útiles:

```bash
claude --worktree "#42"     # worktree a partir del PR #42 (pull/42/head)
claude --worktree "https://github.com/<org>/<repo>/pull/42"   # idem por URL
```

> ⚠️ **Importante en estos repos:** la rama por defecto que crea el modo A se
> llama `worktree-<name>`, que **NO** cumple el regex de naming
> (`^feature/[A-Z0-9]{2,10}-[0-9]{1,4}...`). En local da igual, pero **al
> pushear el ruleset lo rechaza**. Antes de subir, renombra a una rama
> conforme:
>
> ```bash
> git branch -m feature/BCR-10-login
> git push -u origin feature/BCR-10-login
> ```
>
> O directamente pídeselo a Claude ("renómbrala a feature/BCR-10-login y
> súbela") y la skill `commit-flow` valida el nombre.

### `.worktreeinclude` (copiar config local)

Ficheros gitignored (p.ej. `.env`, secretos locales) **no** viajan a un worktree
nuevo. Para copiarlos automáticamente, crea un `.worktreeinclude` en la raíz
(sintaxis tipo `.gitignore`):

```text
.env
.env.local
config/secrets.local.json
```

Solo se procesa con la **Opción A** (y subagentes/desktop), no con `git
worktree` manual. Nunca lo commitees con secretos dentro — es para rutas, no
para valores.

---

## Opción B — `git worktree` manual + CLI

Cuando quieres control total del nombre de rama (conforme desde el inicio) y la
ubicación. **Es la vía más limpia en estos repos** porque la rama ya nace con
nombre válido.

```bash
# desde el repo, con master actualizado
git worktree add ..\sandbox-bcr10 -b feature/BCR-10-login master
cd ..\sandbox-bcr10
claude            # o code . / tu editor

# en otra terminal, en paralelo
git worktree add ..\sandbox-mam9 -b hotfix/MAM-9-totp-skew master
cd ..\sandbox-mam9
claude
```

Gestión y limpieza:

```bash
git worktree list                    # ver todos los worktrees montados
git worktree remove ..\sandbox-bcr10 # quitar uno (debe estar limpio)
git worktree prune                   # limpiar referencias muertas
```

Notas:

- Crea siempre con base **`master`** explícita (`... master`) para no heredar
  otra rama por error.
- No procesa `.worktreeinclude`: copia tu config local a mano si la necesitas.
- No puedes tener la **misma** rama en dos worktrees a la vez.

---

## A vs B — cuál elegir

| | Opción A (`claude --worktree`) | Opción B (`git worktree` manual) |
| --- | --- | --- |
| Setup | Un comando, Claude entra solo | Dos pasos (add + cd + cli) |
| Nombre de rama | `worktree-<name>` (renombrar antes de push) | Conforme desde el inicio |
| Ubicación | `.claude/worktrees/<name>/` | Donde tú decidas |
| Limpieza | Automática / con prompt | Manual (`worktree remove`) |
| `.worktreeinclude` | ✅ | ❌ |
| Desde un PR | ✅ `--worktree "#42"` | Manual (`git fetch` + `add`) |
| Ideal para | Sesiones rápidas, revisar PRs | Ramas de trabajo largas con nombre fijo |

Regla práctica: **revisar/experimentar → A; trabajar una feature de verdad →
B** (o A + renombrado inmediato).

---

## Gotchas comunes (ambas opciones)

- **`.git` compartido, working dirs aislados:** no hay conflictos en el
  historial, pero la misma rama no puede estar en dos worktrees.
- **`.claude/` se comparte** entre todos los worktrees: settings, permisos,
  hooks (`post-dart-edit.ps1`), MCP y skills son comunes. Un cambio en
  `.claude/settings.json` afecta a todas las sesiones.
- **El naming solo se valida en el push**, no en local. Cualquier nombre de
  rama funciona localmente; el rechazo llega al subir.
- **Entornos de dev por worktree:** cada directorio es un checkout limpio —
  reinstala dependencias / regenera artefactos si tu stack lo necesita
  (cuando este template se use en proyectos Flutter: `flutter pub get` en cada
  worktree).
- **No borres la carpeta de un worktree a mano.** Usa `git worktree remove` (o
  deja que la Opción A limpie) para no dejar referencias colgando.

---

## Receta rápida (Opción A, día a día)

```bash
# 1. revisar un PR mientras sigo en mi feature
claude --worktree "#5"
#    ... lees, comentas, sales -> se limpia solo si no tocaste nada

# 2. empezar trabajo paralelo
claude --worktree login
#    dentro: trabajas, y antes de subir:
#    "renombra la rama a feature/BCR-10-login y súbela"
```

## Referencias

- [Claude Code — Worktrees](https://code.claude.com/docs/en/worktrees)
- [Claude Code — Parallel sessions with worktrees](https://code.claude.com/docs/en/common-workflows#run-parallel-sessions-with-worktrees)
- [Claude Code — Worktree settings (`worktree.baseRef`)](https://code.claude.com/docs/en/settings#worktree-settings)
- [`git worktree` (docs oficiales de git)](https://git-scm.com/docs/git-worktree)
- Modelo de ramas: [`git-flow.md`](./git-flow.md) · skill [`commit-flow`](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox/blob/master/.claude/skills/commit-flow/SKILL.md)
