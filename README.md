# my-factory

Herramientas, skills y referencias del curso *Harness Engineering*. Es el
segundo repositorio del examen final y **el sitio donde va todo enlace y toda
recomendación que pase la profesora**, para poder consultarlo después.

```
my-factory/
├── README.md
├── references/          enlaces y recomendaciones, un fichero por tema, fechados
│   ├── README.md        el índice: qué hay, cuándo llegó, para qué sirve
│   └── YYYY-MM-DD-<tema>.md
├── skills/              las skills instaladas en ~/.claude/skills/, tal cual, con su origen
│   └── README.md
└── tools/               utilidades propias (exportador a Langfuse, instrumentos de medida) — ver tools/README.md
```

## Reglas

1. **Cada enlace que llegue va a `references/` con fecha y una línea de para qué
   sirve**, en el mismo commit en que se use por primera vez.
2. **Cada skill que se instale va a `skills/` con su origen y licencia** en el
   `README` de la carpeta. Se lee entera antes de instalarse; si ejecuta código,
   se anota.
3. Lo que aquí se guarda es **referencia**, no decisión. Las decisiones viven en
   los `docs/` del proyecto que las tomó.

## Proyectos que usan esto

- `novaforge-v2` — el harness de novelas (rama `backend-v1` viva).
- `storyMaker` — el proyecto del examen final (por crear).
