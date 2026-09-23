---
name: langfuse
description: Tracing an LLM application with Langfuse from Python — sessions, traces and spans, scores from validators, prompts as versioned objects, and masking. Use when instrumenting a run for observability, recording a validator's score, versioning a prompt, or working out why a trace never appears in the dashboard.
---

# Langfuse, from Python

The SDK is `langfuse`, **major version 4** (`4.15.4` on this machine). v3 and
earlier are a different API: `Langfuse(...)` constructors, `trace()`,
`generation()`, `span()` as methods on the client. **If you recall those, you
are recalling v3.** v4 is an OpenTelemetry client underneath, and everything
hangs off `get_client()` and context managers.

## Credentials come from the environment, and only from there

```bash
LANGFUSE_PUBLIC_KEY=pk-lf-...
LANGFUSE_SECRET_KEY=sk-lf-...
LANGFUSE_BASE_URL=https://cloud.langfuse.com      # EU, the default
# LANGFUSE_BASE_URL=https://us.cloud.langfuse.com # US
```

**The region is the trap.** The SDK points at the EU host by default; a US
project's keys against the EU host fail with a mute 401 and no trace ever
appears. Set `LANGFUSE_BASE_URL` explicitly, every time.

**Never take a key from a chat message, a file or an argument.** Read them from
the environment and check by presence, never by printing:

```python
import os
missing = [k for k in ("LANGFUSE_PUBLIC_KEY", "LANGFUSE_SECRET_KEY") if not os.environ.get(k)]
if missing:
    raise SystemExit(f"not configured: {', '.join(missing)}")   # the names, never the values
```

`.env.example` lists the names with empty values. A key in a repository is a
key to rotate.

## The client, and whether it is even talking

```python
from langfuse import get_client

langfuse = get_client()
if not langfuse.auth_check():
    raise SystemExit("Langfuse rejected these credentials or this host")
```

`auth_check()` is worth the call in any script that runs unattended: the failure
it catches is silent otherwise.

## Traces and spans

```python
with langfuse.start_as_current_observation(as_type="span", name="write-chapter") as span:
    span.update(output="...")

    with langfuse.start_as_current_observation(
        as_type="generation", name="chapter-writer", model="claude-haiku-4-5"
    ) as generation:
        generation.update(output=draft)
```

Nesting is by nesting: an observation opened inside another is its child. The
decorator form is `@observe()` on a function.

## Sessions — grouping the traces of one novel

```python
from langfuse import get_client, propagate_attributes

langfuse = get_client()
with langfuse.start_as_current_observation(as_type="span", name="novel") as root:
    with propagate_attributes(session_id="run-a1b2c3"):
        ...   # every observation created in here carries that session_id
```

A session groups traces and replays them in order. The id is any ASCII string
under 200 characters — a run's slug or id is the natural one.

## Scores — what a validator decided

```python
# from anywhere, when you hold the id
langfuse.create_score(
    name="continuity", value=8, trace_id=trace_id,
    observation_id=observation_id,          # optional: omit to score the trace
    data_type="NUMERIC",                    # NUMERIC | CATEGORICAL | BOOLEAN | TEXT
    comment="the critic's note",
    score_id=f"{run_id}:{chapter}:continuity",   # idempotency: re-running does not duplicate
)

# inside a span
with langfuse.start_as_current_observation(as_type="span", name="gate") as span:
    span.score(name="prose", value=7, data_type="NUMERIC")
    span.score_trace(name="gate_passed", value=False, data_type="BOOLEAN")
```

Numeric values are floats; `CATEGORICAL` and `TEXT` take strings. **A score
nobody produced has no row** — do not send a 0 for a validator that did not run,
because 0 and *absent* are different claims and the dashboard cannot tell them
apart afterwards.

`score_id` is the field that makes an exporter re-runnable. Derive it from the
thing being scored.

## Prompts as versioned objects

```python
langfuse.create_prompt(
    name="chapter-writer",
    type="text",
    prompt="Write chapter {{n}} of {{title}} …",
    labels=["production"],          # the label get_prompt() fetches by default
)

prompt = langfuse.get_prompt("chapter-writer")            # the production label
compiled = prompt.compile(n=3, title="The Ledger")
```

Creating with a name that exists adds a **version**; it does not overwrite. That
is what makes "which prompt wrote this chapter" answerable, and it is the
difference between a tuning iteration you can report and one you cannot.

## Masking

```python
def masking_function(*, data, **kwargs):
    if isinstance(data, str) and data.startswith("SECRET_"):
        return "REDACTED"
    if isinstance(data, dict):
        return {k: masking_function(data=v) for k, v in data.items()}
    if isinstance(data, list):
        return [masking_function(data=item) for item in data]
    return data

langfuse = Langfuse(mask=masking_function)
```

It runs synchronously as attributes are created and covers what the SDK's own
calls set. It does **not** see raw OpenTelemetry attributes from third-party
instrumentation; for those the newer `mask_otel_spans` is the broader hook.

Mask before you send, not after. There is no unsend.

## Flushing

```python
langfuse.flush()      # a script that exits without this loses its last spans
```

The SDK batches in the background. Any short-lived process — an exporter, a CLI,
a test — must flush before it ends.

## Exporting after the fact

Langfuse is usually live instrumentation. When the thing being traced is a
subprocess whose turns you cannot hook — an orchestrator you launch and read —
a **post-hoc exporter** is the honest arrangement: read the run's own log and
database, build the session, the traces and the spans from it, attach the
scores, flush. Say in the documentation that the timestamps are the export's
and not the run's, because they are, and a reader comparing them to a wall clock
deserves to know which they are looking at.
