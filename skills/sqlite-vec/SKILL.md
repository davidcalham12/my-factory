---
name: sqlite-vec
description: Vector search inside SQLite with the sqlite-vec extension — loading it from Python, creating vec0 virtual tables, storing embeddings as compact float32 BLOBs, and running KNN queries. Use when indexing text for retrieval, deciding what to embed and at what granularity, joining vector hits back to source rows, or working out why a similarity query returns nothing.
---

# sqlite-vec

Vector search as a SQLite extension. Written in C, no dependencies, runs
wherever SQLite runs. Source: `asg017/sqlite-vec`, dual MIT / Apache-2.0.

Everything below is from the official README and docs, not from memory. Where
you need something this file does not cover, read those rather than guessing:
an invented function name fails loudly, but an invented *argument order* returns
plausible nonsense.

## Loading it

```python
import sqlite3
import sqlite_vec

db = sqlite3.connect("novaforge.db")
db.enable_load_extension(True)
sqlite_vec.load(db)
db.enable_load_extension(False)       # close the door again immediately

version, = db.execute("select vec_version()").fetchone()
```

Three things that bite:

- **Extension loading is off by default** and must be re-enabled on **every
  connection**, not once per database. A pooled or per-thread connection that
  skipped this raises `no such module: vec0` on the first query, not on connect.
- **Turn it off again.** `enable_load_extension(True)` left on lets any SQL that
  reaches the connection load a shared library. Open it, load, close it.
- Some Python builds ship without `enable_load_extension` at all. Check once at
  startup and fail with a clear message rather than at the first search.

## Creating a table

```sql
create virtual table chunk_vectors using vec0(
  chunk_id integer primary key,
  embedding float[384]
);
```

The dimension is **fixed at creation** and must match the model exactly —
`all-MiniLM-L6-v2` is 384, `all-mpnet-base-v2` is 768. Changing models means a
new table and a re-index, so record which model wrote a table in an ordinary
table beside it. A vector of the wrong length is rejected, which is the good
case; the bad case is two models with the same dimension silently mixed, and
nothing but a provenance column prevents that.

`vec0` also takes metadata, auxiliary and partition-key columns for filtering
without a join. Start without them: a join to a real table is easier to reason
about, and this is the layer people over-build first.

## Writing vectors

```python
from sqlite_vec import serialize_float32

db.execute(
    "insert into chunk_vectors(chunk_id, embedding) values (?, ?)",
    (chunk_id, serialize_float32(embedding)),
)
```

`serialize_float32` packs a list of floats into the compact binary format.
Vectors can also be given as JSON text (`'[0.1, 0.2, ...]'`), which is useful in
a `.sql` file or when debugging by hand, and wasteful at volume — roughly four
times the bytes and a parse per row.

## Querying — KNN

```python
rows = db.execute(
    """
    select chunk_id, distance
    from chunk_vectors
    where embedding match ?
    order by distance
    limit ?
    """,
    (serialize_float32(query_embedding), k),
).fetchall()
```

The shape is not negotiable and each part earns its place:

- **`match`**, not `=` or a function call. This is what engages the index.
- **`order by distance`** — `distance` is a column the virtual table produces.
  Without the ordering you get k rows, not the nearest k.
- **`limit`** is required for a KNN query. Without it the query errors rather
  than scanning, which is the extension telling you it is not a `WHERE` clause.

**Distance is a distance: smaller is nearer.** It is not a similarity score and
not bounded to 0–1. Do not threshold it with a number copied from another
system; measure what values your own corpus produces before deciding what
"close enough" means, or skip thresholds and take the top k.

## Joining back to the source

A vector table holds vectors. Everything a reader needs lives elsewhere:

```sql
select c.text, c.source_path, v.distance
from chunk_vectors v
join chunks c on c.id = v.chunk_id
where v.embedding match :q
order by v.distance
limit 8;
```

Keep `chunks` as an ordinary table with the text, where it came from, and the
row it belongs to. The vector table is an index, not a store: if you would be
unable to rebuild it from the ordinary tables, the design is the wrong way round.

## Chunking, which decides quality more than the model does

- **Chunk on meaning, not on characters.** A paragraph, a bible entry, one
  summary fact. A 512-character window that cuts a sentence retrieves halves of
  two ideas and neither.
- **Keep chunks comparable in size.** Similarity is skewed by length; one
  enormous chunk among small ones wins or loses queries for the wrong reason.
- **Store the origin on the chunk** — which file, which chapter, which line.
  Retrieval that cannot say where a passage came from is not usable as evidence.
- **Re-embed on change.** A stale vector pointing at edited text is worse than a
  missing one, because it answers confidently.

## What vector search will not do

- **It does not guarantee coverage.** Retrieval returns what is *similar*, not
  what is *relevant*, and never what is *complete*. Where a check must see
  everything — every rule in a rulebook, every beat in a list — pass the whole
  thing and do not retrieve. A rule that was not retrieved is a violation nobody
  looked for.
- **It has no idea what a negation means.** "The door was never opened" and "the
  door was opened" embed close together.
- **Exact lookups belong in SQL.** Search by id, name or date with `WHERE`.

## Debugging an empty or wrong result

1. `select vec_version()` — if this fails the extension is not loaded on *this*
   connection.
2. Is the stored vector the right length? `select vec_length(embedding) from ...`
   against the table's declared dimension.
3. Did the query vector come from the **same model** as the stored ones?
   Different models with the same dimension return confident nonsense, and this
   is the failure that looks like a bad corpus rather than a bug.
4. Is `limit` present? A KNN query without one errors.
5. Are the vectors normalised the same way, if the model expects normalisation?
