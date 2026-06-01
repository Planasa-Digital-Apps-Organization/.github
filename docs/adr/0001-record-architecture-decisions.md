# 0001. Record architecture decisions

Date: 2026-05-16
Status: Accepted

## Context

Este repositorio (`claude-sandbox`) se está estableciendo como template canónico de `.claude/` + AGENTS + skills + rules + agents + hooks + PR safeguards para los proyectos Flutter de Planasa-Digital-Apps-Organization. Decisiones sobre el diseño del template (qué rules, qué denylist, qué safeguards en GitHub) se tomarán a lo largo del tiempo, algunas por humanos, otras inducidas por asistentes IA.

Sin un registro persistente:

- Cada nueva sesión de IA reinventa criterios basándose en "lo que el modelo cree mejor".
- Los humanos olvidan por qué tomaron una decisión hace meses y la revierten sin haber considerado la razón original.
- `CLAUDE.md` crece sin control intentando capturar todo el contexto histórico.

Alternativas consideradas:

- **Status quo (nada)**: rechazado, ya se está viendo el síntoma (CLAUDE.md absorbiendo contexto histórico).
- **Documentación libre en `docs/`**: rechazado, no impone disciplina de "una decisión, un documento, inmutable".
- **Wiki externa**: rechazado, separa la decisión del código y aumenta fricción para asistentes IA que leen el árbol del repo.

## Decision

Mantenemos Architecture Decision Records en `docs/adr/` con el formato Nygard (Context / Decision / Consequences). Convenciones:

- Un ADR por decisión, archivo numerado correlativamente `NNNN-slug.md`.
- Inmutables tras `Status: Accepted`. Si la decisión cambia, se crea un ADR nuevo con `Supersedes [NNNN]` y se actualiza el viejo a `Superseded by [MMMM]` — único cambio permitido tras Accepted.
- Máximo ~1 página por ADR.
- La skill `.claude/skills/adr-writer/` automatiza la creación.

## Consequences

- **Positivo**:
  - Decisiones recuperables sin arqueología de git.
  - Asistentes IA pueden leer `docs/adr/` para entender el "por qué" antes de proponer cambios que reviertan decisiones pasadas.
  - Reduce el tamaño de `CLAUDE.md` (no hay que duplicar contexto histórico).
  - Cada repo derivado del template hereda la disciplina ADR.

- **Negativo**:
  - Requiere disciplina para abrir un ADR en el momento de la decisión, no después.
  - Riesgo de "litter" si se abren ADRs para decisiones triviales — mitigado con la guía de la skill `adr-writer` ("no usar para fixes de bugs ni decisiones reversibles").

- **Abierto**:
  - Política de revisión: ¿se hace una pasada trimestral de ADRs vigentes? Pendiente de decidir.
