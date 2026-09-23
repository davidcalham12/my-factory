---
name: sqlite
description: Using SQLite from Python with the standard library `sqlite3` module and numbered SQL migrations, with no ORM. Use when creating or changing a schema, writing queries, handling transactions and concurrency, or deciding how a run's state is persisted. Covers the settings that decide whether a small local database behaves well or corrupts, and the query patterns that keep parameters out of the SQL string.
---

# SQLite from Python, without an ORM

The standard library's `sqlite3` module and plain SQL. No ORM: the schema is
files you can read, and a migration is a numbered `.sql` file that runs once.

## Why no ORM here

Three reasons, and the third is the one that decides it.

1. The schema is small and stable. An ORM earns its place when models change
   constantly and by many hands; it costs a dependency and a layer of
   indirection otherwise.
2. Migrations as numbered SQL files are readable by anyone, diff cleanly, and
   run identically in tests and in production.
3. **`sqlite-vec` is a virtual table.** ORMs model virtual tables poorly or not
   at all, and the moment you work around your ORM to talk to one, you have the
   cost of the ORM without its benefit.

## Connecting

```python
import sqlite3
from pathlib import Path

def connect(path: Path) -> sqlite3.Connection:
    conn = sqlite3.connect(path, isolation_level=None)  # autocommit; BEGIN by hand
    conn.row_factory = sqlite3.Row                      # rows behave like dicts
    conn.execute("PRAGMA journal_mode = WAL")           # readers do not block the writer
    conn.execute("PRAGMA foreign_keys = ON")            # OFF by default — say it every time
    conn.execute("PRAGMA busy_timeout = 5000")          # wait rather than raise "database is locked"
    return conn
```

Four settings, each for a reason worth knowing:

- **`row_factory = sqlite3.Row`** — rows index by name (`row["title"]`) instead
  of position. Positional access breaks silently when a column is added.
- **`journal_mode = WAL`** — a reader and the writer can work at once. It is
  persistent: set once per database file, not per connection, though repeating
  it is harmless. Not available over a network filesystem.
- **`foreign_keys = ON`** — SQLite ignores foreign keys unless you ask, **per
  connection**. A schema full of `REFERENCES` that enforces nothing is worse
  than one with none, because it reads as a guarantee.
- **`busy_timeout`** — without it a concurrent write raises immediately.

A connection belongs to the thread that made it. Either one connection per
thread, or `check_same_thread=False` with your own lock — never share one
across threads and hope.

## Parameters are never string-formatted

```python
# Correct
conn.execute("SELECT * FROM chapters WHERE run_id = ? AND n = ?", (run_id, n))
conn.execute("INSERT INTO runs (slug, premise) VALUES (:slug, :premise)",
             {"slug": slug, "premise": premise})

# Wrong, and the commonest hole in hand-written SQL
conn.execute(f"SELECT * FROM chapters WHERE run_id = '{run_id}'")
```

**Identifiers cannot be parameterised.** `?` binds values, never a table or
column name. When a name must be dynamic, validate it against a hard-coded
allowlist and interpolate the allowlisted constant, never the input:

```python
TABLES = {"runs", "chapters", "critiques"}          # the allowlist IS the check
if table not in TABLES:
    raise ValueError(f"unknown table: {table}")
conn.execute(f"SELECT * FROM {table} WHERE run_id = ?", (run_id,))
```

A public skill for this was read and rejected for exactly this: it interpolated
table and column names straight from JSON input into four different statements.

## Transactions

With `isolation_level=None` you own the transaction boundaries, which is the
point — implicit ones surprise you at the worst moment.

```python
from contextlib import contextmanager

@contextmanager
def tx(conn: sqlite3.Connection):
    conn.execute("BEGIN")
    try:
        yield conn
        conn.execute("COMMIT")
    except Exception:
        conn.execute("ROLLBACK")
        raise
```

Wrap anything that must be all-or-nothing. Writing a chapter's draft, its five
critique rows and its gate decision is one transaction: a crash between them
leaves a run that the reader cannot explain.

## Migrations

Numbered files, applied in order, recorded so they run once.

```
backend/commons/db/migrations/
├── 001_runs.sql
├── 002_chapters_and_critiques.sql
└── 003_character_knowledge.sql
```

```python
def migrate(conn: sqlite3.Connection, folder: Path) -> list[str]:
    conn.execute("CREATE TABLE IF NOT EXISTS schema_migrations ("
                 "  name TEXT PRIMARY KEY,"
                 "  applied_at TEXT NOT NULL DEFAULT (datetime('now')))")
    done = {r["name"] for r in conn.execute("SELECT name FROM schema_migrations")}
    applied = []
    for path in sorted(folder.glob("*.sql")):
        if path.name in done:
            continue
        with tx(conn):
            conn.executescript(path.read_text(encoding="utf-8"))
            conn.execute("INSERT INTO schema_migrations (name) VALUES (?)", (path.name,))
        applied.append(path.name)
    return applied
```

Rules that keep this honest:

- **A migration is never edited once it has run anywhere.** Fix it with the
  next number. An edited migration means two databases with the same recorded
  history and different schemas.
- **`sorted()` on names, so zero-pad.** `010` before `9` is a real bug.
- **`executescript` commits implicitly before it starts.** That is why the
  `BEGIN` above is outside it; do not nest the two and assume atomicity.
- Run `migrate()` on startup and in test setup. The same function, so the test
  schema cannot drift from the real one.

## Schema notes that matter here

- **Column types are advisory.** SQLite stores what you give it; `INTEGER`
  accepts `"banana"`. Use `CHECK` when a value must be one of a set:
  `verdict TEXT NOT NULL CHECK (verdict IN ('accept','retry','accept_with_warnings'))`.
- **`STRICT` tables** (SQLite 3.37+) enforce declared types. Prefer them for new
  tables: `CREATE TABLE runs (...) STRICT;`
- **Timestamps as TEXT in ISO-8601 UTC** (`2026-09-21T14:03:00Z`). They sort
  correctly as strings and survive a round trip without a timezone library.
- **`INTEGER PRIMARY KEY` is the rowid** and costs nothing extra.
- **Index what you filter on.** A run with thousands of log rows scanned per
  request is a slow panel and an easy fix.
- **Booleans are 0/1.** There is no boolean type.

## Reading rows back

```python
row = conn.execute("SELECT * FROM runs WHERE slug = ?", (slug,)).fetchone()
if row is None:           # fetchone returns None, it does not raise
    raise LookupError(slug)
data = dict(row)          # sqlite3.Row -> dict, for JSON responses
```

Use `fetchall()` only when the result is known to be small; iterate the cursor
otherwise.

## Testing

`sqlite3.connect(":memory:")` gives a fresh database per test, with the same
`migrate()` call. It is fast enough that every test can have its own, which is
better than sharing one and cleaning up.

An in-memory database does not support WAL, and that is fine — nothing is
concurrent in a test.

## What SQLite will not do for you

- **No network access, no users, no permissions.** File permissions are the
  access control.
- **One writer at a time.** WAL lets readers continue; it does not give you two
  writers.
- **`ALTER TABLE` is limited.** Adding a column is fine; changing or dropping
  one means the twelve-step dance (create new table, copy, drop, rename) — write
  it into the migration rather than reaching for a tool.
- **No `DATE` type.** See timestamps above.
