# skills

Las skills instaladas en `~/.claude/skills/`, copiadas tal cual. Cada una se leyó
entera antes de instalarse. Ninguna ejecuta código.

| skill | origen | licencia | para qué |
|---|---|---|---|
| `grill-me` + `grilling` | mattpocock/skills | MIT | entrevista por rondas hasta cerrar el árbol de decisiones; `grill-me` es el atajo, `grilling` el contenido |
| `frontend-design` | anthropics/skills | ver LICENSE.txt del repo | plan de tokens (paleta, tipografía, layout, principios) antes de código; *tells* de diseño generado |
| `coherencia-docs` | maujimenez4/MyFactory | — | coherencia entre los documentos de contexto; informe → plan → edición aprobada; lee `docs/_authority.md` |
| `tdd` (+ `tests.md`, `mocking.md`) | mattpocock/skills | MIT | red → green por rebanadas verticales; tests en *seams* acordados; sin tautologías |
| `writing-for-agents` (+ `SKILL-MECHANICS.md`) | mattpocock/skills | MIT | cómo escribir `CLAUDE.md`, `AGENTS.md` y skills |
| `code-review` | mattpocock/skills | MIT | revisión en dos ejes (estándares · spec) con subagentes |
| `handoff` | mattpocock/skills | MIT | traspaso compacto entre sesiones, redactando secretos y PII |
| `ponytail` | DietrichGebert/ponytail | MIT | escalera YAGNI → ya existe → stdlib → nativo → dependencia → una línea → mínimo. **Sólo la skill; sus hooks de Node no se instalan** |

Pendientes de instalar (regla de la profesora: una por tecnología que entre):
`feature-sliced-design`, `sqlite-vec`, `verification` (se construye desde el
artifact), `fastapi`, `react`, `sentence-transformers`, `lean4`, `tlaplus`,
`langfuse`, `playwright-mcp`, `html-to-pdf`, `pydantic`.

Evaluadas y no instaladas: `to-spec` (publica a un gestor de incidencias; se
tomó su plantilla), `SecureSkills-io/sqlite-skill` (rechazada, motivo en
`novaforge-v2/docs/architecture.md` §7.1), `rtk` (binario; riesgo para el
orquestador, ver `references/`).
