# tools

Utilidades propias escritas durante el curso, copiadas tal cual desde el
proyecto que las produjo, con su origen. Son **referencia**, no la versión
viva: la que corre es la del repositorio de origen, y si las dos difieren manda
aquella. Ninguna necesita credenciales en argumentos; las que hablan con un
servicio las leen del entorno.

| fichero | origen | qué hace | cómo se usa |
|---|---|---|---|
| `export_to_langfuse.py` | `novaforge` (rama `claude-orchestrator`), `tools/` | envía el log de un run terminado a Langfuse: una traza por run, una generación por llamada a subagente con sus tokens, un score por crítico. El coste va **acotado** (`low / estimate / high`), nunca calculado, porque el harness reporta un solo total de tokens por llamada. Escrutinio de cadenas con forma de clave antes de enviar nada | `python tools/export_to_langfuse.py output/<slug> --dry-run` (sin credenciales ni red); sin `--dry-run` lee `LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY` y `LANGFUSE_BASE_URL` del entorno. La región del proyecto es **US**; el SDK apunta a EU por defecto |
| `backfill_tokens.py` | `novaforge`, `tools/` | transcribe a `logs/agents.jsonl` los `subagent_tokens` de una sesión anterior a que el orquestador los registrara. Cada fila queda marcada `tokens_source: reconstructed`. Muestra la regla del proyecto: un dato copiado después no es la misma clase de hecho que uno medido en el momento | `python tools/backfill_tokens.py output/<slug>` |
| `measure.mjs` | `novaforge-v2`, `specs/loops/LOOP-003/` | el instrumento de LOOP-003: por intento, las seis notas, en qué intento pasó, hallazgos resueltos, líneas tocadas frente a citadas. Reporta lo no medible como *not measurable*, nunca como cero. Con `--self-test` reproduce el run guardado | `node tools/measure.mjs <slug>` · `node tools/measure.mjs --self-test` |
| `validate-sheet.mjs` | `novaforge-v2`, `specs/loops/LOOP-003/` | la puerta de la hoja de retroalimentación: rechaza una hoja sin alguna de las seis notas, sin alguno de los cuatro campos por hallazgo, con un hueco de plantilla, que cite un capítulo anterior, o que llegue al intento 3 sin frase literal | `node tools/validate-sheet.mjs <hoja.md>` · `--self-test` |

Copiados el 2026-09-23 desde los commits `464cb69` (novaforge) y `3b8e818`
(novaforge-v2). Los dos `.mjs` esperan la estructura `output/<slug>/` y
`specs/loops/LOOP-003/sheets/<slug>/` del proyecto; fuera de él sirven como
modelo de cómo se escribe un instrumento que se prueba a sí mismo.
