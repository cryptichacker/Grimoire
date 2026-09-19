---
tags: [coding, python, pydantic, validation]
type: cheatsheet
source: compiled reference (Pydantic v2)
last-verified: 2026-09-03
---

# Pydantic

## Up
- [[Python]]

Pydantic is the standard Python **data-validation** library — define a schema with type hints, and it parses/validates/coerces input and serializes output. It underpins [[FastAPI]]. This reflects **v2** (Rust core, much faster). Install: `pip install pydantic` (+ `pydantic-settings` for config).

---

## Models

```python
from pydantic import BaseModel, Field, EmailStr
from datetime import datetime

class User(BaseModel):
    id: int
    name: str = "anon"                       # default
    email: EmailStr
    age: int = Field(gt=0, le=120)           # constraints
    tags: list[str] = []
    created: datetime | None = None

u = User(id="1", email="a@x.com", age=30)    # "1" → 1 (coerced), validated
u.id            # 1 (int)
u.model_dump()          # → dict
u.model_dump_json()     # → JSON string
User.model_validate({"id": 2, "email": "b@x.com", "age": 5})   # parse a dict
User.model_validate_json('{"id":3,"email":"c@x.com","age":9}')  # parse JSON
```

Invalid data raises `ValidationError` with a precise, structured error list.

---

## Field Constraints

```python
from pydantic import Field
name:  str   = Field(min_length=1, max_length=50)
price: float = Field(gt=0)
qty:   int   = Field(ge=0, le=1000)
code:  str   = Field(pattern=r"^[A-Z]{3}$")
items: list  = Field(default_factory=list)      # mutable default
secret: str  = Field(repr=False)                # hide from repr
aliased: str = Field(alias="camelCaseKey")      # map external key
```

Rich types: `EmailStr`, `HttpUrl`, `UUID4`, `IPvAnyAddress`, `SecretStr`, `Json`, `conint`/`constr`, `PositiveInt`, `datetime`/`date`.

---

## Validators

```python
from pydantic import field_validator, model_validator

class Signup(BaseModel):
    password: str
    confirm: str

    @field_validator("password")
    @classmethod
    def strong(cls, v: str) -> str:
        if len(v) < 8: raise ValueError("too short")
        return v

    @model_validator(mode="after")
    def match(self):
        if self.password != self.confirm:
            raise ValueError("passwords differ")
        return self

# computed output field
from pydantic import computed_field
class Rect(BaseModel):
    w: int; h: int
    @computed_field
    @property
    def area(self) -> int: return self.w * self.h
```

- `field_validator(mode="before")` runs pre-coercion; `"after"` (default) post-coercion.
- `model_validator(mode="after")` validates the whole object.

---

## Model Config

```python
from pydantic import ConfigDict

class M(BaseModel):
    model_config = ConfigDict(
        extra="forbid",          # reject unknown fields (or "ignore"/"allow")
        frozen=True,             # immutable, hashable
        str_strip_whitespace=True,
        populate_by_name=True,   # accept field name OR alias
    )
```

---

## Settings (env / .env config)

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_prefix="APP_")
    debug: bool = False
    database_url: str
    api_key: SecretStr

settings = Settings()            # reads env vars (APP_DEBUG, APP_DATABASE_URL, ...)
```

The clean, validated way to load configuration (replaces ad-hoc `os.environ`).

---

## Nested Models & Serialization

```python
class Address(BaseModel):
    city: str
class Person(BaseModel):
    name: str
    address: Address                          # nested validation

p = Person.model_validate({"name": "Ada", "address": {"city": "NYC"}})
p.model_dump(exclude={"address"}, exclude_none=True, by_alias=True)

# TypeAdapter — validate non-model types (lists, dicts) without a BaseModel
from pydantic import TypeAdapter
TypeAdapter(list[int]).validate_python(["1", 2, 3])   # → [1,2,3]
```

---

## Dataclasses & v1 notes
- `from pydantic.dataclasses import dataclass` — validation on stdlib-style dataclasses.
- **v1 → v2 changes:** `.dict()`→`.model_dump()`, `.json()`→`.model_dump_json()`, `@validator`→`@field_validator`, `class Config`→`model_config`, `parse_obj`→`model_validate`.

---

## Tips
- Let Pydantic **coerce at the boundary** (API input, config, files) — inside your code, trust the types.
- Use **`extra="forbid"`** to catch typos/unexpected payload keys.
- `SecretStr` keeps secrets out of logs/reprs; access with `.get_secret_value()`.
- For pure validation of built-in types, `TypeAdapter` avoids boilerplate models.
- Pairs with [[FastAPI]] (request/response models) and [[SQLAlchemy]] (map ORM ↔ schema).
