# Examen final · Harness Engineering — enunciado

Llegó el 2026-09-23, pegado por David en la sesión de construcción. Se guarda
tal cual, salvo un dato personal: la dirección de email de entrega se ha
sustituido por `[EMAIL_ELIMINADO]` (RGPD); David la tiene en el enunciado
original. El plan que lo decodifica está en
`novaforge-v2/docs/brief/storymaker-exam-plan.md`.

---

## Contexto

Una empresa quiere vender novelas personalizadas para regalar: a un hijo, a la
pareja, para celebrar una boda, un aniversario o una jubilación. Tu tarea es
diseñar y construir el sistema agéntico que las genera, y presentarlo como
solución técnico-comercial.

El objetivo del cliente tiene dos dimensiones igual de importantes:

**Personalización.** La novela debe incorporar de forma natural los datos del
destinatario: su nombre, su historia, los detalles que el comprador ha
aportado. El destinatario debe reconocerse en la novela y sentir que fue
escrita para él.

**Calidad narrativa mínima.** La novela no tiene que ser un best seller, pero
debe ser agradable de leer. No se aceptan novelas con problemas narrativos
evidentes: inconsistencias de personajes, saltos temporales sin sentido,
capítulos que se contradicen, prosa mecánica o repetitiva, o finales abruptos.
El lector debe poder leerla de principio a fin sin tropezar. La personalización
no justifica una mala escritura.

El sistema no puede optimizar solo para que los datos aparezcan: debe también
producir una historia que funcione como historia. Los validadores, el rol de
editor y el LLM-as-judge deben reflejar este equilibrio.

Proyecto individual. Novelas de 10 capítulos de unas 1.000–1.500 palabras cada
uno. La complejidad está en el harness, no en la extensión.

## Entregables

- Repositorio **storyMaker** — el proyecto de novelas personalizadas: código,
  README, brief de ejemplo reproducible, `.env.example` y carpeta `/docs` con
  toda la documentación de proceso.
- Repositorio **MyFactory** — herramientas y utilidades del curso, ya iniciado
  durante las prácticas.
- La presentación formal y sus anexos en la carpeta `/presentacion/`,
  commiteados antes del plazo de entrega del repo storyMaker.

Formato de los archivos en `/presentacion/`:

- el deck principal en PDF y en el formato original editable (PowerPoint,
  Keynote o similar);
- los anexos como ficheros individuales nombrados de forma descriptiva (por
  ejemplo, `anexo-tla-spec.pdf`, `anexo-evals-tabla.pdf`);
- un `README.md` en la misma carpeta que liste el contenido y el idioma elegido.

## Entrega final

La entrega se hace enviando un email a `[EMAIL_ELIMINADO]` con el asunto:

    [Harness Engineering] Entrega final — <nombre del estudiante>

El email debe incluir:

- el link al commit final del repositorio storyMaker;
- el link al commit final del repositorio MyFactory;
- una frase de no más de tres líneas resumiendo la decisión de diseño más
  importante que tomaste.

No se aceptan entregas fuera de plazo. El commit final es el que cuenta, no la
hora del email.

## Alcance del proyecto

### 1. Configuración

- Un agente entrevistador recoge los datos del destinatario: nombre, edad,
  rasgos, recuerdos, género, tono y extensión. También recoge las palabras o
  temas que el cliente no quiere que aparezcan.
- Detecta los datos que faltan y al menos un tipo de contradicción (por
  ejemplo, edad frente a género o tono).
- El usuario puede pegar texto libre (una anécdota, una carta) del que se
  extraen hechos. Ese texto se trata como contenido no confiable.
- El resultado de la entrevista es un brief estructurado y validado con schema.

### 2. Lectura interactiva (web o PDF)

La novela se entrega como web o como PDF interactivo. En ambos casos debe
incluir:

- un índice de capítulos navegable;
- una ficha de personajes y lugares generada desde la story bible, con enlaces
  al capítulo donde aparece cada uno;
- una portada con dedicatoria personalizada.

Si es web, el lector puede seleccionar un fragmento o un hecho y pedir un
cambio desde la propia página ("el perro se llama Nala"). El sistema identifica
los capítulos que usan ese hecho, regenera solo esos sin romper la continuidad
y marca en la lectura qué capítulos han cambiado respecto a la versión anterior.

Si es PDF, el cambio se pide desde fuera del documento (formulario o CLI) y se
genera una nueva versión del PDF. Esa versión incluye una página inicial de
"novedades" con los capítulos modificados y enlaces internos a cada uno.

En ambas opciones se conserva la versión anterior de la novela.

### 3. Harness

- Tres roles como mínimo: planner, writer y editor/critic.
- Un archivo de instrucciones `CLAUDE.md`, una skill reutilizable y dos hooks:
  uno de validación del capítulo y otro de policy.
- Tools con schema validado.
- Retries con límite.
- Registro de tokens y coste por novela a través de Langfuse (ver sección de
  observabilidad).

### 4. Memoria

- Una story bible en SQLite (obligatorio) en la que cada hecho registra en qué
  capítulos se usa. Incluye una tabla de cronología (eventos, momento,
  personajes, lugar) que alimenta el validador formal.
- Resúmenes por capítulo para construir el contexto de los siguientes.
- Checkpoint por capítulo: si la generación falla, se reanuda desde el último
  capítulo completado.

### 5. Validación y evaluación

El sistema debe incluir validadores de cuatro tipos. Cada validador tiene un
nombre, se ejecuta en un punto concreto del harness (hook, rol editor o gate
antes de publicar una versión) y envía su resultado a Langfuse como score.

**a) Validadores programáticos (deterministas).** Mínimo tres, por ejemplo:

- el brief y la salida de cada rol cumplen su schema;
- el nombre del destinatario y los personajes aparecen escritos exactamente
  como en la story bible;
- la longitud de cada capítulo está dentro del rango;
- cada elemento personalizado obligatorio del brief aparece en al menos un
  capítulo, comprobado contra la tabla de hechos de SQLite;
- el guardrail de palabras prohibidas (ver sección 7);
- validación visual via browser MCP: el agente abre la novela en el browser,
  navega por los capítulos y verifica que el índice, la ficha de personajes y
  la portada renderizan correctamente; si detecta un error visual, lo registra
  como fallo y lo devuelve al writer o al rol correspondiente.

**b) Validadores no programáticos (semánticos).** Mínimo dos:

- un LLM-as-judge con rúbrica que evalúe continuidad, tono, calidad narrativa
  (arco de la historia, coherencia de personajes, ritmo entre capítulos) y que
  la personalización esté integrada de forma natural y no forzada, con una
  puntuación por criterio y una justificación;
- una revisión humana de al menos una novela completa, con la misma rúbrica,
  para comparar el juicio humano con el del LLM.

**c) Validador formal de la historia (Lean 4).** La cronología de la historia
se modela formalmente y se verifica:

- a partir de la story bible en SQLite se genera un fichero Lean con los hechos
  temporales: eventos, momento, personajes presentes, lugar, fechas de
  nacimiento;
- se definen en Lean al menos dos invariantes, por ejemplo: los eventos
  respetan el orden temporal declarado; la edad de un personaje en cada evento
  es coherente con su fecha de nacimiento; un personaje no está en dos lugares
  en el mismo momento; un personaje no aparece después de un evento que lo
  excluye (muerte, partida definitiva);
- la verificación se ejecuta de forma automática (`lake build` o `lean`) y, si
  falla, la versión de la novela no se publica y el fallo vuelve al editor como
  feedback;
- debe mostrarse al menos un caso real en el que el validador formal detecta
  una incoherencia que los otros validadores no detectaron, o justificar por
  qué no se encontró ninguno.

**d) Validador formal del sistema (TLA+).** Mientras Lean verifica la
coherencia de la historia, TLA+ verifica el comportamiento del harness.
Referencia: learntla.com.

- Una especificación en TLA+ o PlusCal del flujo de generación como máquina de
  estados: configuración → planificación → escritura de capítulo → validación →
  publicación de versión, incluyendo retries, reanudación desde checkpoint y
  regeneración por cambio del lector.
- Al menos tres invariantes de seguridad, por ejemplo: nunca se publica una
  versión con un capítulo que no ha pasado todos los validadores; la
  reanudación desde checkpoint no duplica ni pierde capítulos; la versión
  anterior de la novela se conserva siempre tras una regeneración; el número
  de reintentos nunca supera el límite.
- Al menos una propiedad de liveness: toda generación termina publicando una
  versión o deteniéndose con error; nunca queda en un bucle infinito.
- Verificación con el model checker TLC sobre un modelo pequeño (por ejemplo,
  5 capítulos y 2 reintentos), con la configuración incluida en el repo.
- La especificación debe corresponder al código: el README explica qué estado
  o transición del código implementa cada acción de la especificación.
- Si TLC encontró algún contraejemplo durante el desarrollo, se documenta junto
  con el cambio que hizo en el código.

**Evaluación del sistema:**

- Cinco briefs de prueba, incluido al menos uno adversarial (injection en el
  texto libre) y uno diseñado para provocar una incoherencia temporal.
- Una tabla que muestre, por brief, qué validadores pasaron y cuáles fallaron.
- Una iteración de tuning documentada, con los resultados antes y después.

### 6. Observabilidad

- Cada generación de novela es una traza en Langfuse, agrupada por sesión (una
  sesión por novela, incluyendo la entrevista y las regeneraciones posteriores).
- Cada rol (entrevistador, planner, writer, editor) y cada llamada a tool
  aparece como span con nombre identificable.
- Tokens, coste y latencia visibles por llamada, por capítulo y por novela.
- Los resultados de todos los validadores (programáticos, semánticos y Lean) se
  envían a Langfuse como scores asociados a la traza correspondiente. TLC se
  ejecuta en desarrollo, no en cada generación.
- Los prompts versionados en Langfuse, de forma que la iteración de tuning
  muestre qué versión de prompt produjo cada resultado.

### 7. Guardrails

- Un guardrail de palabras prohibidas, aplicado en código sobre cada capítulo
  antes de aceptarlo:
  - listas guardadas en SQLite, en tres niveles: globales (insultos, términos
    ofensivos) y por novela, definidas por el cliente en la configuración (por
    ejemplo, el nombre de una expareja o un tema que no quiere que aparezca);
  - la detección normaliza el texto antes de comparar: mayúsculas, acentos,
    plurales y variantes simples;
  - si hay coincidencia, el capítulo se devuelve al writer para reescribirlo,
    con un límite de intentos; si se agota el límite, la generación se detiene
    y se informa;
  - cada coincidencia queda registrada en el audit log y en Langfuse;
  - tests que cubran al menos un caso de cada nivel y un caso de variante
    (acento o plural).
- Un audit log de las decisiones del policy engine.
- Uso de un máximo de 100.000 tokens concurrentes.

## Opcional (suma nota)

- **Servidor MCP** para consultar y descargar novelas, con FastMCP como plugin
  del FastAPI existente. Tools sugeridas: `list_novels`, `get_chapter`,
  `list_versions`, `query_story_bible`, `download_novel`. Requisitos: cada tool
  con schema validado; servidor de solo lectura; cada llamada registrada en
  Langfuse; el README explica cómo conectarlo a un cliente MCP. Si hay login,
  respeta la identidad del usuario autenticado.
- **Tools de escritura sobre el servidor MCP** (pedir un cambio del lector
  desde un cliente MCP), con permisos y confirmación.
- **Nuevos linters de prosa**: repeticiones y muletillas; frases demasiado
  largas o legibilidad inadecuada; abuso de adverbios, clichés o expresiones
  típicas de texto generado por IA; consistencia de estilo (tiempo verbal,
  narrador, tratamiento). Sobre Vale o LanguageTool con reglas propias, o desde
  cero.
- **Linter propio para edición manual**: en el editor web, extensión de VS Code
  o servidor LSP; comprueba contra la story bible y las palabras prohibidas;
  una edición que cambia un hecho actualiza la bible y vuelve a pasar por los
  validadores (incluido Lean) antes de publicar.
- **Invariantes adicionales en Lean**, o demostraciones generales (para
  cualquier cronología).
- **Especificación TLA+ del servidor MCP** o de la concurrencia entre
  regeneraciones simultáneas.
- **Login de usuarios con SQLite**: registro e inicio de sesión con email y
  contraseña hasheada (bcrypt o similar); sesión con token (JWT o similar);
  cada novela, configuración y entrada del audit log asociada al propietario;
  el servidor MCP respeta la identidad; tests de que un usuario no accede a las
  novelas de otro.
- **Análisis de seguridad con agentes o skills**: prompt injection,
  exfiltración de datos entre novelas o clientes, dependencias (`pip audit`,
  `npm audit`), secrets leak en el historial de commits; informe en
  `/docs/security-report.md` con severidad y cambio que lo resolvió.

## Fuera de alcance

Pagos, cuentas de usuario, impresión física, ilustraciones, audio y despliegue
en producción.

## Los repositorios deben incluir también

- **Novela de ejemplo generada**: el PDF de una novela completa de 10 capítulos,
  generada con el brief de ejemplo del README, commiteada en
  `/ejemplos/novela-ejemplo.pdf`. Si el formato de lectura elegido es web, se
  incluye igualmente el PDF exportado.
- **`/docs` en storyMaker** con la documentación de proceso. No se corrige el
  resultado, se corrige el razonamiento que llevó a él:
  - Spec inicial: qué se decidió construir y por qué, antes de escribir código.
  - Trade-offs: cada decisión de diseño relevante como decisión: opciones,
    criterios y elección (single-agent vs multi-agent, formato de la story
    bible, modelo de lectura, integración de TLA+ con el flujo real,
    invariantes de Lean priorizados).
  - Explainers: uno por cada concepto del curso aplicado. Breves.
  - Diagramas: arquitectura del harness, máquina de estados de TLA+, esquema
    SQLite, tabla de validadores con su punto de ejecución.
  - Registro de iteraciones: qué cambió tras cada eval o contraejemplo de TLC o
    Lean, y por qué. Un log de decisiones con causa y efecto.
  - Red-team log: casos adversariales probados, qué validador los detectó (o
    no) y cómo se resolvió.
- **Vídeo de demo** en cualquier formato, en `/presentacion/` o enlazado desde
  su `README.md`.
- **Sin API keys en ningún repo.** Usar `.env.example`.
- **Claude Code.** El repo debe reflejar ese uso:
  - `CLAUDE.md` en la raíz, cuidado y legible: es parte del examen;
  - la carpeta `.claude/` con memoria y comandos personalizados, commiteada;
  - `.claude/mcp.json` (o equivalente) con un servidor MCP de inspección de
    browser (Chrome MCP, Playwright MCP o similar);
  - el uso real del browser MCP documentado en `/docs`: qué inspeccionó el
    agente, qué detectó y qué cambio provocó;
  - las skills usadas o creadas en el repo y referenciadas desde `/docs`;
  - subagentes y comandos propios documentados en `/docs` con propósito y
    resultado.

**Un proyecto sin evals con resultados medibles, o sin documentación de
proceso en `/docs`, no aprueba.**

## Presentación (10 minutos)

Propuesta formal a cliente con imagen corporativa propia: nombre de empresa,
logotipo, paleta de colores y tipografía consistentes. No se acepta plantilla
genérica. Idioma: inglés o castellano; términos técnicos en inglés. Anexos al
final del deck, valorados en la nota. Slide obligatoria de portada: empresa
presentadora, cliente ficticio, fecha y nombre del estudiante.

| bloque | tiempo | contenido mínimo |
|---|---|---|
| Portada y contexto | 0,5 min | empresa presentadora, cliente, ocasión |
| Problema y cliente | 1 min | quién compra, para qué ocasiones, por qué las alternativas no funcionan |
| Configuración y lectura | 1 min | cómo se configura, cómo se lee y corrige (web o PDF) |
| Arquitectura del harness | 2 min | roles y bucle, gestión del contexto, story bible, tools, hooks, justificación |
| Validación, evaluación y observabilidad | 2 min | los cuatro tipos y dónde actúan; tabla por brief; un fallo detectado por Lean; propiedades TLC; mejora tras el tuning; una traza real en Langfuse |
| Guardrails | 0,5 min | palabras prohibidas con ejemplo, datos personales, injection, audit log |
| Presupuesto y coste | 1 min | ver abajo |
| Demo y cierre | 1 min | un cambio del lector propagado; riesgos y siguientes pasos |
| Contraportada | — | contacto de la empresa presentadora |

Después, 5 minutos de preguntas técnicas.

### Slide de presupuesto y coste (obligatoria)

- Coste unitario por novela: tokens (medido en Langfuse sobre ejecuciones
  reales), infraestructura y margen operativo.
- Precio de venta al cliente final y margen resultante.
- Coste del proyecto de desarrollo: horas por fase (diseño, desarrollo,
  validación, despliegue), tarifa hora y total.
- Escenarios de volumen: margen para tres volúmenes mensuales distintos.
- Análisis de sensibilidad: tokens +50 %; más de tres revisiones por novela.

Los números deben ser reales o razonados, no inventados.

### Evidencias obligatorias en la presentación

1. La tabla de resultados de las evals, con números.
2. El coste real por novela, obtenido de Langfuse, y el margen resultante.
3. La demo de un cambio del lector propagado a los capítulos afectados.
