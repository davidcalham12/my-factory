# Spec-Driven Development — métodos en la comunidad

Llegó el 2026-09-23, de la profesora, con la instrucción de guardarlo aquí.
Lista tal cual, y debajo cómo se compara con **nuestro** proceso
(`AGENTS.md`: docs → spec aprobada → plan aprobado → código con TDD).

## Open source

| método | qué es | enlace |
|---|---|---|
| **GitHub Spec Kit** | CLI oficial de GitHub en Python. Flujo *constitution → specify → plan → tasks → implement*; funciona con 30+ agentes de código | https://github.com/github/spec-kit |
| **OpenSpec** | Alternativa ligera a Spec Kit, basada en **cambios** (*propose → apply → archive*). Sirve para brownfield y greenfield; sin Python | https://github.com/Fission-AI/OpenSpec |
| **GSD (Get Shit Done)** | Meta-prompting, ingeniería de contexto y SDD ligero para Claude Code, Codex, Cursor… Apunta al *context rot* | https://github.com/gsd-build/get-shit-done |
| **Superpowers** | Método de desarrollo para agentes hecho de skills componibles. **Saca la spec de la conversación antes de escribir código** | https://github.com/obra/superpowers |
| **BMAD-METHOD** | Framework pesado de ciclo completo con personas-agente (analista, PM, arquitecto, dev, QA). Potente; curva de aprendizaje alta | https://github.com/bmad-code-org/BMAD-METHOD |
| **MUSUBI** | Alto rigor: constitución de 9 artículos, requisitos en formato **EARS**, diagramas **C4**, **ADRs**, y validación de cada feature contra la constitución. Modos greenfield y brownfield | https://github.com/nahisaho/MUSUBI |
| **Agent OS** (Brian Casel) | Lee el código existente, escribe las convenciones que realmente usa y se las da al agente. Bueno para brownfield | https://github.com/buildermethods/agent-os |

## Comerciales

| producto | qué es | enlace |
|---|---|---|
| **EasySpecs** (Xesca Alabart) | SDD para bases de código existentes. Primero documentación técnica y funcional del código real (hasta 98 % de cobertura de líneas); sobre eso, specs de calidad emparejadas con **Trust Specs** (validadores, casos límite, tests de rollback), verificadas por una cascada de comprobaciones deterministas y probabilísticas. Agnóstico de proveedor | https://easyspecs.ai |
| **Kiro** (AWS) | IDE spec-driven sobre VS Code: requisitos, documentos de diseño, *steering files*. Ata al IDE | https://kiro.dev |
| **Tessl** | Plataforma spec-centric para desarrollo AI-native; la spec es el artefacto principal | https://tessl.io |
| **BrainGrid** | Planificación y descomposición de specs, agnóstico del agente; entrega el trabajo al agente de código | https://www.braingrid.ai |
| **CodeMySpec** | SDD centrado en comprobar que el código coincide con la spec | https://codemyspec.com |
| **Augment Code** (Cosmos) | SDD desde el lado del contexto: motor persistente que entiende la arquitectura en bases grandes y coordina varios agentes | https://www.augmentcode.com |

## Cómo se compara con lo nuestro

Lo que hacemos en `novaforge-v2` no es ninguno de estos, pero tiene piezas de
varios. Sirve para explicarlo en la presentación y para saber qué copiar si
hace falta:

| pieza nuestra | equivalente en la comunidad | diferencia |
|---|---|---|
| `AGENTS.md` como reglas de proceso con puertas | la **constitution** de Spec Kit y de MUSUBI | la nuestra pone la aprobación **escrita en el fichero** como acto humano; un "ok" en el chat no vale |
| `grilling` antes de la spec | **Superpowers** saca la spec de la conversación | nosotros preguntamos sólo lo que el repo no responde; los hechos los busca el agente |
| `SPEC-NNN` → `PLAN-NNN` → código | *specify → plan → tasks → implement* de Spec Kit | el plan lista **los tests antes que el código**, o no se aprueba |
| `SPEC-008-leftovers` (arreglos pequeños vía spec corta) | el modelo por **cambios** de OpenSpec | igual espíritu: hasta un arreglo pasa por spec |
| `verification.md` con **T/A/I/D/U** y huecos declarados | las **Trust Specs** de EasySpecs | la nuestra clasifica cada garantía por cómo se demuestra y **declara lo no verificable** |
| `coherencia-docs` | lo más cercano es Agent OS (convenciones reales frente a declaradas) | la nuestra compara documentos entre sí por tipo de afirmación |
| `docs/_authority.md` | — | quién gana en cada tipo de afirmación; no lo vimos en ninguno |

**Recomendación:** no adoptar ninguno entero a dos días de la entrega. Si hay
que citar uno como referencia formal, **Spec Kit** es el más cercano en flujo y
**EasySpecs** el más cercano en verificación — y es la casa de la profesora.
Si después del examen se quiere formalizar, MUSUBI (EARS + ADRs) es el que más
rigor añade a lo que ya tenemos.
