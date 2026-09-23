---
name: sentence-transformers
description: Producing text embeddings locally with sentence-transformers — choosing a model and knowing its dimension, encoding in batches, normalisation, and keeping the model behind an interface so it can be swapped. Use when indexing text for vector search, deciding what to embed, or working out why similarity scores look wrong.
---

# sentence-transformers

Embeddings computed on this machine. No network call, no per-token cost, and it
runs in CI — which is why it was chosen over a hosted embedding API for a corpus
this size.

## The basics

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("all-MiniLM-L6-v2")      # downloads once, then cached
vectors = model.encode(["first chunk", "second chunk"])   # -> ndarray (2, 384)
```

`encode` takes a list and returns one row per input. Passing a single string
returns a single vector, which is a shape difference that causes a confusing
error one call later — pass lists and index the result.

## Choosing the model, and writing the dimension down

| model | dimension | size | when |
|---|---|---|---|
| `all-MiniLM-L6-v2` | 384 | ~80 MB | the default: fast, small, good enough for retrieval |
| `all-mpnet-base-v2` | 768 | ~420 MB | noticeably better, several times slower |
| `multi-qa-MiniLM-L6-cos-v1` | 384 | ~80 MB | tuned for question → passage retrieval |

**The dimension is a hard contract with the vector store.** A `vec0` table
declares `float[384]` at creation and cannot change. So:

- Record which model produced a table, in an ordinary column beside it.
- **Two different models with the same dimension are the dangerous case**: the
  store accepts the vectors and every search returns confident nonsense. Nothing
  but that recorded provenance catches it.
- Changing model means a new table and a full re-index. Plan it as a migration.

## Encoding at volume

```python
vectors = model.encode(
    chunks,
    batch_size=32,
    normalize_embeddings=True,
    show_progress_bar=False,        # off in a server or a test
    convert_to_numpy=True,
)
```

- **Encode in one call, not in a loop.** Batching is where the speed is; a
  per-item call is an order of magnitude slower.
- **`normalize_embeddings=True` makes every vector unit length**, so cosine
  similarity and dot product agree. Decide once and apply it to *everything* —
  a corpus half normalised is a corpus whose distances are meaningless.
- **Load the model once**, at startup, and reuse it. Constructing
  `SentenceTransformer` reads hundreds of megabytes from disk.
- The first run downloads weights from the network and caches them under
  `~/.cache/huggingface`. In CI, either allow that once or pre-cache it —
  "local" means no *per-call* network, not none ever.

## Behind an interface

The model is a choice that should be reversible without touching callers.

```python
from typing import Protocol

class Embedder(Protocol):
    dimension: int
    name: str
    def embed(self, texts: list[str]) -> list[list[float]]: ...

class LocalEmbedder:
    def __init__(self, model_name: str = "all-MiniLM-L6-v2"):
        self._model = SentenceTransformer(model_name)
        self.name = model_name
        self.dimension = self._model.get_sentence_embedding_dimension()

    def embed(self, texts: list[str]) -> list[list[float]]:
        return self._model.encode(
            texts, normalize_embeddings=True, convert_to_numpy=True
        ).tolist()
```

`dimension` comes from the model rather than from a constant, so it cannot drift
from what the model actually produces.

For tests, a fake that needs no weights and no download:

```python
class FakeEmbedder:
    dimension, name = 384, "fake"
    def embed(self, texts):                       # deterministic, from the text
        return [[((hash(t) >> i) % 1000) / 1000 for i in range(self.dimension)]
                for t in texts]
```

It will not retrieve sensibly — it is not meant to. It proves the plumbing:
that vectors of the right length reach the store and come back joined to their
rows. Anything about retrieval *quality* needs the real model and is a separate,
slower test.

## What to embed

- **Chunk on meaning**: a paragraph, a bible entry, one summary fact. A
  fixed-width window that cuts a sentence stores half of two ideas.
- **Keep chunks comparable in size.** Length skews similarity, and one huge chunk
  among small ones wins or loses queries for reasons unrelated to content.
- **Store the origin with the chunk** — file, chapter, line. A retrieval that
  cannot say where a passage came from is not usable as evidence.
- **Embed the text as a reader meets it.** Stripping punctuation and casing
  helped older methods and hurts these models, which were trained on prose.

## Reading the numbers honestly

- With normalised vectors, cosine similarity is in [-1, 1] and in practice
  unrelated text sits around 0.0–0.3, related around 0.5+. **Measure your own
  corpus before choosing a threshold**; a number copied from another project
  means nothing here.
- **`sqlite-vec` returns a distance, not a similarity.** Smaller is nearer. Do
  not compare it against a similarity threshold.
- **Negation is invisible.** "The door was never opened" and "the door was
  opened" embed close together. Where a check depends on a negative, retrieval is
  the wrong instrument.
- **Retrieval is never exhaustive.** Where something must be checked against
  *every* rule, pass all the rules and do not retrieve — a rule not retrieved is
  a violation nobody looked for.
