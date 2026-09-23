# Lenguajes para validadores formales y demostradores — por penetración de mercado

Llegó el 2026-09-23, de la profesora. Lista tal cual, ordenada como la mandó, y
debajo qué usamos nosotros y por qué.

| lenguaje | qué es | dónde se usa | tendencia | enlace |
|---|---|---|---|---|
| **TLA+** | especificación y model checking | AWS, Microsoft, Oracle, Intel, bases de datos distribuidas | ↑ crecimiento moderado | https://foundation.tlapl.us |
| **Lean 4** | asistente de pruebas y lenguaje de programación | matemáticas, AWS, DeepMind, startups de IA (Harmonic) | ↑↑ el que más crece | https://lean-lang.org |
| **Rocq** (antes Coq) | asistente de pruebas | CompCert, academia, criptografía | → estable, pierde peso frente a Lean | https://rocq-prover.org |
| **Isabelle/HOL** | asistente de pruebas | seL4 (microkernel verificado), academia | → estable | https://isabelle.in.tum.de |
| **Dafny** | lenguaje verificable, estilo C#/Python | AWS, benchmarks de IA | ↑ creciendo | https://dafny.org |
| **SPARK** (Ada) | subconjunto verificable de Ada | aeroespacial, defensa, ferroviario | → nicho regulado | https://www.adacore.com/about-spark |
| **B-Method / Event-B** | especificación por refinamiento | metro y ferrocarril (Alstom, Siemens) | → / ↓ legado | https://www.atelierb.eu · https://www.event-b.org |
| **P** | modelado de máquinas de estados | AWS (S3, DynamoDB…) | ↑ creciendo | https://p-org.github.io/P/ |
| **Alloy** | especificación ligera | academia, diseño de modelos de datos | → estable | https://alloytools.org |
| **F\*** | tipos dependientes | criptografía (HACL\*, en Firefox, Linux, Windows) | → nicho | https://fstar-lang.org |
| **Verus** | verificación de código Rust | sistemas en Rust, Microsoft Research | ↑↑ crece rápido desde base pequeña | https://github.com/verus-lang/verus |
| **Kani** | model checker para Rust | AWS, librería estándar de Rust | ↑ creciendo | https://github.com/model-checking/kani |
| **Quint** | TLA con sintaxis moderna | blockchain, protocolos de consenso | ↑↑ crece rápido desde base pequeña | https://quint-lang.org · https://github.com/informalsystems/quint |
| **Agda / Idris 2** | tipos dependientes | investigación en teoría de tipos | → académico | https://agda.readthedocs.io · https://www.idris-lang.org |
| **Z, VDM, PVS** | especificación clásica | NASA (PVS), sistemas industriales heredados | ↓ en declive | https://www.overturetool.org (VDM) · https://pvs.csl.sri.com (PVS); Z es estándar ISO sin web única |

## Qué usamos, y por qué

El examen fija dos, y son los dos que la lista marca como los de mayor tirón:

- **Lean 4** para la **historia**: la cronología de la story bible (eventos,
  momento, personajes, lugares, fechas de nacimiento) se exporta desde SQLite a
  un fichero Lean y se comprueban invariantes sobre esa cronología concreta —
  orden temporal, edad coherente con la fecha de nacimiento, nadie aparece
  después del evento que lo excluye. Se ejecuta con `lake build` en la puerta
  de publicación; si falla, la versión no se publica.
- **TLA+** para el **sistema**: el flujo del harness como máquina de estados
  (configuración → planificación → escritura → validación → publicación, con
  reintentos, checkpoint y regeneración por cambio del lector), invariantes de
  seguridad y una propiedad de liveness, comprobadas con **TLC** sobre un modelo
  pequeño (5 capítulos, 2 reintentos). Se ejecuta en desarrollo, no en cada
  generación.

Dos de la lista merecen mención en la presentación como alternativas
consideradas:

- **Quint**: es TLA+ con sintaxis moderna y mejor tooling. Si TLA+ clásico se
  hace cuesta arriba en dos días, Quint produce el mismo modelo y se comprueba
  con Apalache/TLC. Riesgo: el examen dice TLA+ o PlusCal; consultar antes.
- **P**: máquinas de estados con la misma familia de problemas que nuestro
  harness (es lo que AWS usa para S3). No lo pide el examen; sirve como
  comparación en `trade-offs.md`.

Regla de la profesora que aplica aquí: **cada lenguaje que entre lleva su
skill** (`lean4`, `tlaplus`).
