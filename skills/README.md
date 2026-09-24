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
| `feature-sliced-design` (+ 9 references) | feature-sliced/skills | MIT | dónde va cada cosa en el frontend (capas, slices, API pública, imports hacia abajo) |
| `sqlite` | escrita para novaforge-v2 | — | `sqlite3` estándar, migraciones numeradas, transacciones, sin ORM |
| `sqlite-vec` | escrita para novaforge-v2 desde el README de asg017/sqlite-vec y los docs de Python | extensión: MIT / Apache-2.0 | vec0, BLOBs float32, KNN, unir aciertos a sus filas |
| `verification` | construida para novaforge-v2 desde el brief §2.2 y el Anexo D | — | `verification.md`: garantías con T/A/I/D/U, criticidad, huecos declarados |
| `fastapi` | escrita para novaforge-v2 | — | un folder por feature + commons, SSE, tests sin red ni coste |
| `react` | escrita para novaforge-v2 | — | React + TS + Vite: efectos, fetching, SSE, error boundaries |
| `sentence-transformers` | escrita para novaforge-v2 | — | embeddings locales: modelo, dimensión, lotes, normalización |
| `pydantic` | escrita aquí desde los docs oficiales de Pydantic v2 | — | validar en un borde: `extra="forbid"` como contrato, `errors()` como lista de preguntas, dónde vive una regla |
| `langfuse` | escrita aquí desde los docs del SDK Python v4 | — | sesiones, spans, scores de validadores, prompts versionados, máscara; la trampa de la región US |
| `playwright` | escrita aquí desde los docs oficiales de Playwright Python | — | imprimir HTML a PDF con Chromium; separar el renderizador puro del navegador |

Añadidas el 2026-09-23 desde la máquina virtual de construcción (estaban en su
`~/.claude/skills/` desde el 21–22; siete de las que arriba figuraban como
pendientes). Añadidas el 2026-09-23 para el examen, escritas desde la documentación oficial
de cada tecnología porque no hay skill publicada fiable: `pydantic`, `langfuse`,
`playwright` (que cubre el hueco que la lista llamaba `html-to-pdf`: el PDF se
imprime con Chromium y no añade dependencia nueva). Pendientes: `lean4` y
`tlaplus`, que esperan a que `elan` y un JDK se instalen en la máquina, y
`playwright-mcp`, que es configuración de servidor MCP y no una técnica.

Evaluadas y no instaladas: `to-spec` (publica a un gestor de incidencias; se
tomó su plantilla), `SecureSkills-io/sqlite-skill` (rechazada, motivo en
`novaforge-v2/docs/architecture.md` §7.1), `rtk` (binario; riesgo para el
orquestador, ver `references/`).
| `fastmcp` | StoryMaker `.claude/skills/fastmcp` (`b95abbf`) | — | construir el servidor MCP del proyecto (SPEC-EXAM-005 O2) con FastMCP |
| `novaforge` | StoryMaker `.claude/skills/novaforge` (`b95abbf`) | — | el procedimiento que conduce una novela entera con un solo orquestador (seis etapas, gate por capítulo) |
| `storymaker` (+ `units/`) | StoryMaker `.claude/skills/storymaker` (`b95abbf`) | — | el mismo procedimiento partido en unidades para el conductor por etapas (SPEC-EXAM-003); `units/chapter.md` es un capítulo de principio a fin |
