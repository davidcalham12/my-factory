---
name: verification
description: Produce a verification.md for a system — every guarantee it claims, classified T/A/I/D/U (Test, Analysis, Inspection, Demonstration, Unverifiable), each with a criticality level that fixes its minimum letter, plus the known gaps and accepted risks the system ships with. Use when writing or reviewing a verification, validation or evaluation plan, when asked how a guarantee is proved, when deciding how much verification a claim deserves, or when a claim needs its honest classification.
---

# Verification, validation, evaluation

> **Designing solutions with AI is an exercise in BEST EFFORT. The skill of a
> good AI solutions engineer lies in managing to build systems that are
> *reliable*.**

That is the principle, and it decides the shape of the document. **Reliability
does not come from nothing failing. It comes from knowing where it can fail, what
happens when it does, and how you would find out.** A system with known, bounded
gaps is reliable; a system with "no gaps" is one whose gaps nobody looked for.

So the deliverable is not a list of what is verified. It is that list **plus the
list of what is not** — §"Known gaps" below is the section that makes the rest
worth reading, and the one most plans omit.

Given a system's context, produce `verification.md`: every guarantee the system
claims, what class of evidence can support it, by what method, and what that
method leaves behind that someone else could check.

> **Provenance of this skill.** Its catalogue comes from a verification
> methodologies document summarised in the NovaForge v2 build brief, §2.2. The
> source artifact was not reachable when this was written, so the method lists
> below are as that summary gave them. If the artifact becomes available, read it
> and correct anything here that differs — and say what changed rather than
> quietly replacing it.

## The five classes

A guarantee is verified by one of five, and the letter is a claim about
**evidence**, not about effort.

| letter | class | what it means | what it leaves |
|---|---|---|---|
| **T** | Test | executed, with a pass/fail the machine decides | a test that runs in CI and fails on regression |
| **A** | Analysis | derived without executing: types, static analysis, proof | a compiler or analyser run, with its output |
| **I** | Inspection | a person read it and confirmed it | a named reviewer, a date, what was read |
| **D** | Demonstration | shown working once, under observation | a recorded run, its artefacts, its conditions |
| **U** | Unverifiable | cannot be established by any of the above | a written statement of *why*, and what is claimed instead |

### The selection rule

**Prefer Test over Analysis, Analysis over Inspection, Inspection over
Demonstration — and declare Unverifiable without embarrassment when it is.**

The catalogue is neutral; this ordering is the judgement. It follows from what
each class survives: a Test runs again tomorrow, an Analysis runs again when the
code changes, an Inspection is true of the version someone read, and a
Demonstration is true of one run that already happened.

**Never promote a guarantee to a stronger letter than its evidence supports.**
This is the failure this document exists to prevent. A property that depends on
a model's judgement is not a Test merely because a test file calls it: the test
proves the plumbing ran, not that the judgement was right. Classifying it T
makes the plan *less* trustworthy than having no plan, because it converts an
open question into a closed one on paper.

Where a guarantee splits, split it. "The gate approves good chapters" is often
two claims: *this run's gate produced these scores* (D, from a recorded run) and
*this text would pass any run* (U, because the judges are models and do not
reproduce). Write both lines.

## Criticality: how much verification a guarantee deserves

**Not everything deserves the same effort, and knowing what to care about is half
the work.** Every guarantee carries a level, and the level fixes the **minimum
acceptable letter**:

| level | meaning | minimum letter |
|---|---|---|
| **critical** | if it breaks, the system's thesis or its safety falls | **T or A** |
| **important** | if it breaks, the result visibly worsens | **I** |
| **incidental** | if it breaks, it is fixed later without damage | anything, **U** included |

The column exists so the decision is written down rather than implied. Without
it, a plan spends its effort evenly and leaves the one guarantee everything rests
on sitting at **D**.

**The rule that makes the level do work: a critical guarantee whose best honest
letter is I or D automatically opens a row in "Known gaps".** It is not promoted
to T to satisfy the table, and it is not dropped from the document. It is named
as an accepted risk, with the cost of closing it.

Expect this rule to find things. Applied to a document that already existed, it
surfaced three guarantees below their minimum that nobody had flagged — which is
the point: the criterion notices what a reader would not.

## The catalogue

### Verifying artefacts — 8 methods

| method | verifies | class |
|---|---|---|
| type checking | shapes and contracts at boundaries | A |
| static analysis | patterns reachable without running | A |
| symbolic execution | paths across ranges of input, not examples | A |
| formal verification | a property over all states, proved | A |
| unit and integration tests | behaviour, executed | T |
| property-based testing | an invariant over generated input | T |
| mutation testing | that the tests would notice a change | T |
| contract testing | that two sides still agree | T |

### Verifying an agent's process — 11 methods

| method | verifies | class |
|---|---|---|
| runtime traceability | what actually happened, per call | D |
| evals | behaviour over a fixed suite, scored | T or D |
| sandboxing | that a capability is absent, not merely unused | A or T |
| guardrails | that a forbidden action is refused | T |
| human review | judgement no machine renders | I |
| multi-agent verification | that an independent judge agrees | D |
| CI/CD gates | that nothing ships without the above | T |
| progressive deployment | behaviour under real load, in stages | D |
| red-teaming | resistance to someone trying to break it | D |
| model checking | a property over a modelled state space | A |

*(The source summary names eleven; ten are listed. Restore the missing one from
the artifact when it is available rather than inventing it.)*

**Evals are T or D and the difference matters.** A suite with a deterministic
scorer, run in CI, is T. A suite scored by a model is D — it demonstrates what
happened on that run. Same file, different letter, decided by the scorer.

## The shape of the document

Produce these sections, in this order:

1. **The epigraph** — best effort, and its consequence.
2. **How to read it** — the five letters, the three levels.
3. **The guarantees** — promise · level · letter · method · evidence.
4. **Known gaps and accepted risks** — the five fields below.
5. **Code before agent** — what moved from judgement to script.
6. **What each validator stops propagating.**
7. **What changed since the last version.**

### Known gaps and accepted risks

The most important section, and the one usually missing. One row per gap, five
fields:

| field | what goes in it |
|---|---|
| what is not verified | in one sentence |
| why it is accepted | the cost of verifying it, or the impossibility |
| scope of the damage | what breaks if it fails, and **how far it reaches** |
| how we would find out | the signal that gives it away: a log, a metric, a user — or "there is none" |
| who reviews it, and when | or "nobody, and that is accepted" |

**A gap that is listed is an engineering decision. A gap that is not listed is a
defect.** That sentence is the whole section.

Two rules around it:

- **Rows arrive from specs.** Each spec lists the gaps it leaves; they come here
  and stay.
- **Never remove a row except by the evidence that closed it.** Deleting one
  because it reads badly is the failure the document exists to prevent.

The third field is the one that gets written lazily. "It could fail" is not a
scope; "**bounded by one call rather than by zero**" is.

### Code before agent

**When a check can be done by a script, it is done by a script.** An agent judges
only where judgement is needed. A script gives **T**; an agent gives **D** at
best, so every move from agent to code improves reliability and cost at once.

Keep a table of what has moved — check · was · is — and name the next candidates.
And make it a standing rule wherever the build process is written down: **before
proposing an agent for a task, say why a script will not do.**

### What each validator stops propagating

A validator does not "check" something. **It stops a failure reaching the next
step.** Write that sentence for each one — "stops an incomplete sheet reaching
the writer" is worth more than "validates sheets", because it says what is lost
when it is removed.

## How to produce verification.md

1. **List the guarantees from what the system claims**, not from what the code
   has. A guarantee nobody wrote down cannot be verified, and a system with no
   stated guarantees needs that noted at the top rather than an empty document.
2. **For each, ask what would falsify it.** A guarantee with no failing case is
   not a guarantee; it is a description. Write the falsifier down — it is
   usually the test.
3. **Assign the letter from the evidence available**, using the ordering above.
4. **Name the method and the evidence**, specifically: which file, which run,
   which reviewer. "Tested" is not evidence; `backend/tests/test_context.py::test_writer_packet_has_no_prose` is.
5. **Assign the criticality level**, and check the minimum letter. Anything
   below it opens a gap row — that is the check, and it is mechanical.
6. **Say what is not covered.** A verification plan that lists only what it
   verifies reads as complete and is not.

### The shape of a row

```markdown
### G3 — The writer never receives prior prose

**Level:** critical — the system's central claim rests on it
**Class:** T — meets the minimum. Had the honest letter been I or D, this row
would also open a gap, and the gap would say what closing it costs.
**Method:** type + unit test. The writer's `ContextPacket` has no field that can
carry prose; `tests/test_context.py::test_writer_packet_rejects_prose` fails if
the constructor gains one.
**Evidence:** the test, in CI, on every commit; prompt size logged per chapter,
which must stay flat as the book grows.
**Not covered:** that the summary passed in its place contains no prose
paraphrase of a previous chapter. That is **U** — it is a judgement about text.
```

Note the last line. Most real guarantees have an edge the method does not reach,
and the row is more useful for saying so than for the letter.

## Downgrades to expect, and to write plainly

These are the cases where a project usually claims more than it has:

- **A capability removed by construction versus by convention.** "The agent
  cannot reach X because its tool list has no reader" is A — the capability is
  absent. "The code does not pass X" is T, and only while the test exists. When
  a rewrite converts the first into the second, the letter changes and the
  document must say so rather than inheriting the old language.
- **A model's judgement, anywhere in the chain.** D for "this run", U for "in
  general".
- **A threshold with no measured basis.** Unverifiable until something measures
  it; a number in a config is a decision, not evidence.
- **"It has never failed."** Not a class. Absence of a known failure is not
  verification, and saying how many times it ran is the honest version.
