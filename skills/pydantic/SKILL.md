---
name: pydantic
description: Validating data at a boundary with Pydantic v2 — models and field constraints, model_validate and model_dump, the error list a ValidationError carries, strict mode, field and model validators, forbidding unknown fields, and JSON Schema for a tool's contract. Use when defining the shape of an input a model or a user supplies, rejecting malformed data at an edge, or publishing the schema a tool's output must satisfy.
---

# Pydantic v2

`pydantic` **2.x** (`2.13.5` here). v1's `parse_obj`, `.dict()`, `.json()`,
`@validator` and `class Config` are the previous major version; if you recall
those names, you are recalling v1.

| v1 | v2 |
|---|---|
| `Model.parse_obj(d)` | `Model.model_validate(d)` |
| `Model.parse_raw(s)` | `Model.model_validate_json(s)` |
| `m.dict()` / `m.json()` | `m.model_dump()` / `m.model_dump_json()` |
| `@validator` | `@field_validator` |
| `@root_validator` | `@model_validator` |
| `class Config:` | `model_config = ConfigDict(...)` |
| `Model.schema()` | `Model.model_json_schema()` |

## A model, and the constraints on it

```python
from pydantic import BaseModel, ConfigDict, Field

class Recipient(BaseModel):
    model_config = ConfigDict(extra="forbid")

    alias: str = Field(min_length=1, max_length=80)
    age: int = Field(ge=0, le=120)
    pronouns: str
    traits: list[str] = Field(default_factory=list, max_length=12)
```

**`extra="forbid"` is the setting that makes a schema a contract.** The default
is `ignore`: a misspelled key is silently dropped and the field keeps its
default, which is how a required instruction goes missing without anyone
noticing. `allow` keeps unknown keys in `__pydantic_extra__`. For anything that
crosses a trust boundary — a model's JSON, a user's form, a tool's output —
forbid.

## Validating

```python
Recipient.model_validate({"alias": "Ana", "age": 8, "pronouns": "she/her"})
Recipient.model_validate_json('{"alias": "Ana", "age": 8, "pronouns": "she/her"}')
```

Use `model_validate_json` on raw JSON rather than `json.loads` then
`model_validate`: it parses and validates in one pass and its error positions
point into the text.

**Strict mode** refuses coercion — `"8"` stops being an acceptable `int`:

```python
Recipient.model_validate(data, strict=True)
```

Coercion is convenient at a form and dangerous at a machine boundary, where a
string where a number belongs usually means the producer is confused.

## The error is a list, and it is the useful part

```python
from pydantic import ValidationError

try:
    Brief.model_validate(payload)
except ValidationError as exc:
    for error in exc.errors():
        print(error["loc"], error["type"], error["msg"])
```

`errors()` gives one entry per problem, each with the path (`loc`) to the field.
Reporting that list is what turns "the brief is invalid" into a set of questions
to ask; reporting `str(exc)` throws the structure away.

## Validators

```python
from pydantic import field_validator, model_validator

class Brief(BaseModel):
    model_config = ConfigDict(extra="forbid")
    recipient: Recipient
    tone: str
    free_text: str = ""

    @field_validator("tone")
    @classmethod
    def normalise(cls, v: str) -> str:
        return v.strip().casefold()

    @model_validator(mode="after")
    def age_and_tone_agree(self):
        adult = {"novela negra adulta", "erótico", "thriller violento"}
        if self.recipient.age < 13 and self.tone in adult:
            raise ValueError(f"age {self.recipient.age} with tone {self.tone!r}")
        return self
```

`@field_validator` needs `@classmethod` under it and sees one field.
`@model_validator(mode="after")` sees the built instance and is where a rule
*between* fields belongs.

**Where a rule lives is a decision, not a detail.** A rule inside the model
fires on every construction, which is right for an invariant that is always
true. A rule that produces a *report* — what is missing, what contradicts what,
what to ask next — belongs in a function beside the model, so the caller can
receive the whole list instead of the first exception. Validate the shape in
the model; decide the answer in code you can test on its own.

## JSON Schema — publishing the contract

```python
schema = Brief.model_json_schema()
```

A plain jsonable dict. It is what a tool's declared input looks like, what a
front end can generate a form from, and what a test compares against a form's
field names so the two cannot drift.

## Serialising

```python
m.model_dump()                       # dict
m.model_dump(exclude_none=True)      # omit what was never set
m.model_dump_json(indent=2)          # str
```

`exclude_none=True` is the one to reach for when *absent* is a meaningful
answer: a key omitted says nobody measured it, where `null` in the output can
read as a measured nothing, and `0` certainly does.
