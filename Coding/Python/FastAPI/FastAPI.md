---
tags: [coding, python, web, api, fastapi]
type: cheatsheet
source: compiled reference (FastAPI)
last-verified: 2026-08-27
---

# FastAPI

## Up
- [[Python]]

FastAPI is a modern, async, type-hint-driven web framework for building APIs. It uses **Pydantic** for validation and **Starlette** for the ASGI core, and auto-generates OpenAPI/Swagger docs. Install: `pip install fastapi uvicorn[standard]`. Compare with [[Flask]] (simpler, sync).

---

## Minimal App

```python
from fastapi import FastAPI

app = FastAPI(title="My API", version="1.0.0")

@app.get("/")
def root():
    return {"message": "Hello"}    # dict → JSON automatically
```

```bash
uvicorn main:app --reload                     # dev, auto-reload
uvicorn main:app --host 0.0.0.0 --port 8000
# fastapi dev main.py   /   fastapi run main.py   (fastapi-cli)
```

Interactive docs are free: **`/docs`** (Swagger UI) and **`/redoc`**. OpenAPI JSON at `/openapi.json`.

---

## Path & Query Parameters

```python
@app.get("/items/{item_id}")           # path param (typed & validated)
def get_item(item_id: int):
    return {"item_id": item_id}

@app.get("/items")                     # query params from function args
def list_items(skip: int = 0, limit: int = 10, q: str | None = None):
    return {"skip": skip, "limit": limit, "q": q}

# validation constraints
from fastapi import Path, Query
def f(item_id: int = Path(..., ge=1),
      q: str = Query(None, min_length=3, max_length=50)): ...
```

Types are enforced automatically — `/items/abc` returns a 422 with a clear error.

---

## Request Bodies with Pydantic

```python
from pydantic import BaseModel, Field, EmailStr

class Item(BaseModel):
    name: str
    price: float = Field(..., gt=0)
    tags: list[str] = []
    email: EmailStr | None = None

@app.post("/items", status_code=201)
def create_item(item: Item):           # parsed + validated from JSON body
    return {"received": item.name, "price": item.price}

@app.put("/items/{item_id}")
def update(item_id: int, item: Item):  # path + body together
    return {"id": item_id, **item.model_dump()}
```

Pydantic v2: `item.model_dump()` → dict, `item.model_dump_json()` → JSON, `Item.model_validate(data)` to parse.

---

## Response Models & Status Codes

```python
class ItemOut(BaseModel):
    id: int
    name: str

@app.get("/items/{id}", response_model=ItemOut)     # filters/validates output
def get(id: int):
    return {"id": id, "name": "x", "secret": "hidden"}   # 'secret' stripped

from fastapi import status
@app.post("/x", status_code=status.HTTP_201_CREATED)
def create(): ...
```

---

## Errors & Validation

```python
from fastapi import HTTPException

@app.get("/items/{id}")
def get(id: int):
    if id not in db:
        raise HTTPException(status_code=404, detail="Item not found")
    return db[id]

# custom handler
from fastapi.responses import JSONResponse
from fastapi.requests import Request

@app.exception_handler(ValueError)
async def value_error_handler(request: Request, exc: ValueError):
    return JSONResponse(status_code=400, content={"error": str(exc)})
```

---

## Dependency Injection

```python
from fastapi import Depends, Header

def get_db():
    db = Session()
    try:
        yield db            # value injected; teardown after response
    finally:
        db.close()

def verify_token(x_token: str = Header(...)):
    if x_token != "secret":
        raise HTTPException(401, "Invalid token")
    return x_token

@app.get("/items")
def read(db=Depends(get_db), token: str = Depends(verify_token)):
    return db.query(...)

# apply to a whole router/app
app = FastAPI(dependencies=[Depends(verify_token)])
```

Dependencies compose, cache within a request, and appear in the OpenAPI docs.

---

## Async Endpoints

```python
import httpx

@app.get("/proxy")
async def proxy():
    async with httpx.AsyncClient() as client:
        r = await client.get("https://api.example.com")
    return r.json()
```

Use `async def` when the handler awaits I/O (HTTP, async DB drivers). Use plain `def` for sync/blocking code — FastAPI runs it in a threadpool so it won't block the event loop.

---

## Other Inputs

```python
from fastapi import Form, File, UploadFile, Cookie

@app.post("/login")
def login(username: str = Form(...), password: str = Form(...)): ...

@app.post("/upload")
async def upload(file: UploadFile = File(...)):
    content = await file.read()
    return {"filename": file.filename, "size": len(content)}

@app.get("/me")
def me(session: str | None = Cookie(None)): ...
```

---

## Middleware, CORS, Routers

```python
from fastapi.middleware.cors import CORSMiddleware
app.add_middleware(CORSMiddleware, allow_origins=["*"],
                   allow_methods=["*"], allow_headers=["*"])

# modular routers (like Flask blueprints)
from fastapi import APIRouter
router = APIRouter(prefix="/users", tags=["users"])

@router.get("/")
def list_users(): ...

app.include_router(router)

# lifespan / startup-shutdown
from contextlib import asynccontextmanager
@asynccontextmanager
async def lifespan(app):
    # startup: connect db
    yield
    # shutdown: close db
app = FastAPI(lifespan=lifespan)
```

---

## Tips

- Everything is driven by type hints — the more precise your types, the better the validation and docs.
- Return a Pydantic model or dict; FastAPI serializes and documents it for you.
- `response_model` doubles as an output filter (great for hiding internal fields).
- Run in production with `uvicorn`/`gunicorn -k uvicorn.workers.UvicornWorker`, multiple workers behind nginx.
- BackgroundTasks (`from fastapi import BackgroundTasks`) run work after the response is sent.
