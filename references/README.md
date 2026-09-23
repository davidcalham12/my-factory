# Referencias — índice

Todo lo que ha pasado la profesora o el equipo, con la fecha en que llegó y
para qué se usó. Un fichero por tema cuando hay que desarrollarlo; una fila aquí
cuando basta el enlace.

| fecha | tema | enlace | para qué se usó |
|---|---|---|---|
| 2026-09-17 | Fuente de precios de modelos | https://platform.claude.com/docs/en/about-claude/pricing | `config/pricing.json`; el coste se acota, no se calcula |
| 2026-09-17 | Langfuse — región **US** | https://us.cloud.langfuse.com | el SDK apunta a EU por defecto y devuelve un 401 mudo; tres tropiezos |
| 2026-09-21 | Ontología de generación de novelas — definiciones | https://claude.ai/artifact/4SxM2Tuc3UwihHoxGu9K4y | base de `docs/definitions.md` y `domain-knowledge.md`; capas de contexto, *Scene Context Packet*, *knowledge-state* por personaje |
| 2026-09-21 | Ontología — diagramas Mermaid | https://claude.ai/artifact/9gn89g44w6F5xYRa8LMm2M | árbol de entidades, anatomía, capas de contexto, pipeline con puertas |
| 2026-09-21 | Feature-Sliced Design — overview | https://feature-sliced.design/docs/get-started/overview | arquitectura del frontend (D28, D29) |
| 2026-09-21 | FSD — docs para LLMs | https://feature-sliced.design/docs/llms | contexto para el agente al construir el frontend |
| 2026-09-21 | FSD — skill oficial | https://github.com/feature-sliced/skills — `npx skills add https://github.com/feature-sliced/skills --skill feature-sliced-design` | skill `feature-sliced-design` |
| 2026-09-21 | SQLite — skill | https://github.com/SecureSkills-io/sqlite-skill | evaluada; **rechazada** con motivo en `architecture.md` §7.1 |
| 2026-09-21 | sqlite-vec — skill de búsqueda vectorial | (la profesora la nombró sin URL; localizar en marketplaces y leer entera) | búsqueda vectorial en SQLite (D16, D17) |
| 2026-09-21 | Metodologías de verificación — hoja de referencia | https://claude.ai/artifact/Rass3RVfaN5KSJDdG2FQhR | skill `verification`; 8 métodos de artefacto, 11 de proceso, clasificación **T/A/I/D/U** |
| 2026-09-21 | grill-me / grilling — skill | https://github.com/mattpocock/skills (`skills/productivity/grill-me`, `grilling`) | entrevista por rondas para cerrar decisiones; usada en dos rondas del brief y en cada spec |
| 2026-09-21 | frontend-design — skill oficial de Anthropic | https://github.com/anthropics/skills (`skills/frontend-design/SKILL.md`) | plan de tokens antes de código; lista de *tells* de diseño generado |
| 2026-09-22 | coherencia-docs — skill (compañero) | https://github.com/maujimenez4/MyFactory (`.claude/Skills/coherencia-docs`) | coherencia entre `definitions`, `domain-knowledge`, `architecture`, `verification`, `CLAUDE.md`; 17 + 6 incidencias resueltas en `novaforge-v2` |
| 2026-09-23 | mattpocock/skills — el repo entero | https://github.com/mattpocock/skills | instaladas además: `tdd`, `writing-for-agents`, `code-review`, `handoff`; `to-spec` no instalada, plantilla adoptada |
| 2026-09-23 | ponytail — "lazy senior dev" | https://github.com/DietrichGebert/ponytail | skill instalada (sólo `SKILL.md`, no sus hooks); la escalera YAGNI → stdlib → nativo → una línea |
| 2026-09-23 | rtk — proxy de comandos que reduce tokens | https://github.com/rtk-ai/rtk | **no instalado**: su hook global reescribe el Bash del orquestador (`wc -w`, `python -m …`); si se instala, `exclude_commands = ["wc","python","node","cat"]` |
| 2026-09-23 | Métodos de Spec-Driven Development | ver [`2026-09-23-spec-driven-development.md`](2026-09-23-spec-driven-development.md) | comparar nuestro proceso `AGENTS.md` con los de la comunidad |
| 2026-09-23 | learntla.com — TLA+ | https://learntla.com | referencia que da el examen para el validador formal del sistema |
| 2026-09-23 | Lenguajes de verificación formal, por penetración de mercado | ver [`2026-09-23-formal-verification-languages.md`](2026-09-23-formal-verification-languages.md) | TLA+ y Lean 4 son los del examen; Quint y P como alternativas consideradas |
| 2026-09-23 | EasySpecs — materiales del curso | https://easyspecs.ai | enunciado del examen final; Trust Specs ≈ nuestro `verification.md` |
| 2026-09-23 | Enunciado del examen final (Harness Engineering) | ver [`2026-09-23-examen-final-enunciado.md`](2026-09-23-examen-final-enunciado.md) | lo que aprueba y lo que no; entregables, `/docs`, presentación y slide de presupuesto; el plan que lo decodifica está en `novaforge-v2/docs/brief/storymaker-exam-plan.md` |
