---
name: fastapi
description: Building a FastAPI backend organised as one folder per feature plus a commons folder, with Pydantic models, dependency injection, background work in-process, Server-Sent Events for live progress, and tests that run without network or cost. Use when adding an endpoint, streaming progress to a browser, structuring a router, or deciding where request-scoped state belongs.
---

# FastAPI

## Layout: one folder per feature, plus commons

```
backend/
├── commons/          shared machinery, no feature knowledge
│   ├── db/           connection, migrations
│   ├── config/       loading specs and config files
│   └── llm/          the model client and its mock
├── runs/
│   ├── router.py     the endpoints
│   ├── models.py     Pydantic request/response shapes
│   ├── service.py    the work; knows nothing about HTTP
│   └── repository.py the SQL
└── main.py           assembles the app from the routers
```

The rule that keeps this honest: **`service.py` must not import from FastAPI.**
If the work needs `Request`, `HTTPException` or a `Depends`, the boundary has
slipped and the logic can no longer be tested or reused without a web server.
The router translates HTTP to arguments and results back to HTTP; that is all it
does.

`commons/` holds what every feature needs and no feature owns. A thing that
knows about chapters does not belong there however widely it is used.

```python
# main.py
from fastapi import FastAPI
from runs.router import router as runs_router

app = FastAPI(title="NovaForge")
app.include_router(runs_router, prefix="/api/runs", tags=["runs"])
```

## Endpoints

```python
from fastapi import APIRouter, HTTPException, Depends, status
from pydantic import BaseModel, Field

router = APIRouter()

class StartRun(BaseModel):
    premise: str = Field(min_length=10, max_length=2000)
    profile: str = "tiny"
    tone: str | None = None          # None means: read the genre off the premise

class RunCreated(BaseModel):
    id: str
    slug: str

@router.post("", response_model=RunCreated, status_code=status.HTTP_201_CREATED)
def start_run(body: StartRun, svc: RunService = Depends(get_run_service)) -> RunCreated:
    try:
        run = svc.start(body.premise, body.profile, body.tone)
    except AlreadyRunning as exc:
        raise HTTPException(status.HTTP_409_CONFLICT, str(exc)) from exc
    return RunCreated(id=run.id, slug=run.slug)
```

- **Declare `response_model`.** It is the contract, it filters out fields the
  caller should not see, and it is what the generated OpenAPI shows.
- **Validate at the edge with Pydantic**, not with `if` statements inside the
  service. A `Field(min_length=...)` is a 422 with a precise message for free.
- **Raise `HTTPException` in the router only.** Services raise their own domain
  errors; the router maps them to status codes. That mapping is the router's job
  and the reason it exists.
- `def` gets a threadpool, `async def` runs on the event loop. **A blocking call
  inside `async def` stalls every other request** — for synchronous database and
  file work, plain `def` is the right and safe choice.

## Dependencies

```python
from functools import lru_cache
from typing import Annotated
from fastapi import Depends

@lru_cache
def get_settings() -> Settings:
    return Settings()                      # read once, reused

def get_db(settings: Annotated[Settings, Depends(get_settings)]):
    conn = connect(settings.db_path)
    try:
        yield conn                         # request-scoped
    finally:
        conn.close()
```

A `yield` dependency closes its resource after the response, including when the
handler raised. Override them in tests with `app.dependency_overrides[get_db] = ...`,
which is the cleanest seam FastAPI gives you — a test swaps the real database or
the real model client for a fake without touching the code under test.

## Background work, in-process

For a single-user tool with a queue of one, a background task in the same
process is the right size. Anything more is infrastructure nobody asked for.

```python
@router.post("/{run_id}/start")
def start(run_id: str, background: BackgroundTasks, svc = Depends(get_run_service)):
    svc.mark_queued(run_id)                # state change BEFORE returning
    background.add_task(svc.execute, run_id)
    return {"status": "queued"}
```

**Persist state at every step, not at the end.** A background task that keeps
progress in memory loses everything to a restart, and a long run will meet one.
Write to the database after each stage and each attempt, so a crashed run is
still readable and, ideally, resumable.

`BackgroundTasks` runs after the response is sent and dies with the process. For
work measured in hours that is a real limit — accept it deliberately, and make
the persisted state good enough that a restart can pick up rather than repeat.

## Server-Sent Events

For progress that flows one way, server to browser, SSE beats WebSockets: it is
plain HTTP, it reconnects on its own, and there is no protocol to debug.

```python
import json
from fastapi.responses import StreamingResponse

def sse(event: str, data: dict) -> str:
    return f"event: {event}\ndata: {json.dumps(data)}\n\n"

@router.get("/{run_id}/events")
def events(run_id: str, svc = Depends(get_run_service)):
    def stream():
        yield sse("open", {"run_id": run_id})
        for update in svc.follow(run_id):        # blocks until the next update
            yield sse(update.kind, update.payload)
        yield sse("done", {"run_id": run_id})

    return StreamingResponse(
        stream(),
        media_type="text/event-stream",
        headers={"Cache-Control": "no-cache", "X-Accel-Buffering": "no"},
    )
```

The details that decide whether it works:

- **Two newlines end an event.** One, and the browser waits forever.
- **`X-Accel-Buffering: no`** stops a proxy holding the stream until it fills.
- **Send something periodically**, even a comment line `: ping\n\n`, or an idle
  connection is dropped by something in the middle.
- **The stream is a view, never the record.** A client that missed an event must
  be able to recover everything by reading the run's state from the database. If
  the stream is the only place a fact exists, a reconnect loses it.
- On the browser: `new EventSource("/api/runs/x/events")`, then
  `es.addEventListener("progress", ...)`, and `es.close()` when done — it
  reconnects automatically otherwise.

## Errors

```python
class DomainError(Exception): ...
class NotFound(DomainError): ...
class BudgetExceeded(DomainError): ...

@app.exception_handler(NotFound)
def not_found(request, exc):
    return JSONResponse(status_code=404, content={"error": str(exc)})
```

One handler per domain error beats `try/except` in every route. Return a
consistent body — `{"error": "..."}` everywhere — so the frontend has one shape
to handle.

## Tests

```python
from fastapi.testclient import TestClient

def test_start_run_returns_slug(client):
    r = client.post("/api/runs", json={"premise": "A lighthouse keeper...", "profile": "tiny"})
    assert r.status_code == 201
    assert r.json()["slug"] == "a-lighthouse-keeper"
```

```python
# conftest.py
@pytest.fixture
def client(tmp_path):
    app.dependency_overrides[get_db] = lambda: memory_db()
    app.dependency_overrides[get_llm] = lambda: MockEngine()   # no network, no cost
    with TestClient(app) as c:
        yield c
    app.dependency_overrides.clear()
```

**Every test runs against the mock engine and costs nothing.** A suite that needs
a credential is a suite that does not run in CI, and a suite that does not run in
CI is documentation.

## Settings

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    db_path: str = "novaforge.db"
    anthropic_api_key: str | None = None      # from the environment, never a file
    use_mock_engine: bool = True              # real engine is opt-in

    class Config:
        env_file = ".env"
```

Credentials come from the environment. Never a default in code, never committed,
never logged — and `.env` is in `.gitignore` before it is created, not after.
