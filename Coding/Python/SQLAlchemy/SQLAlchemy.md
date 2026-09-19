---
tags: [coding, python, sqlalchemy, orm, database]
type: cheatsheet
source: compiled reference (SQLAlchemy 2.0)
last-verified: 2026-09-03
---

# SQLAlchemy

## Up
- [[Python]]

SQLAlchemy is Python's most powerful SQL toolkit + ORM. Two layers: **Core** (SQL expression language) and the **ORM** (map classes ↔ tables). This reflects the modern **2.0 style** (`select()`, `Session`). Install: `pip install sqlalchemy`. Pairs with [[SQL]] (the query-language notes) and Alembic for migrations.

---

## Engine & Connection

```python
from sqlalchemy import create_engine, text

engine = create_engine("postgresql+psycopg://user:pass@host/db", echo=True)
# sqlite:///app.db  •  mysql+pymysql://...  •  postgresql+asyncpg://... (async)

with engine.connect() as conn:                # Core
    rows = conn.execute(text("SELECT * FROM users WHERE id=:id"), {"id": 1})
    for row in rows: print(row.id, row.name)
    conn.commit()
```

---

## ORM Models (Declarative, 2.0 typed)

```python
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship
from sqlalchemy import ForeignKey, String

class Base(DeclarativeBase): pass

class User(Base):
    __tablename__ = "users"
    id:    Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(255), unique=True)
    name:  Mapped[str | None]                       # nullable via Optional
    posts: Mapped[list["Post"]] = relationship(back_populates="author")

class Post(Base):
    __tablename__ = "posts"
    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str]
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"))
    author: Mapped["User"] = relationship(back_populates="posts")

Base.metadata.create_all(engine)                    # create tables
```

---

## Session & CRUD

```python
from sqlalchemy.orm import Session
from sqlalchemy import select

with Session(engine) as session:
    # CREATE
    u = User(email="a@x.com", name="Ada")
    session.add(u); session.add_all([...])
    session.commit()

    # READ (2.0 select)
    u = session.get(User, 1)                        # by PK
    stmt = select(User).where(User.name == "Ada").order_by(User.id)
    users = session.scalars(stmt).all()             # list[User]
    one = session.scalars(stmt).first()

    # UPDATE
    u.name = "Ada L."; session.commit()

    # DELETE
    session.delete(u); session.commit()
```

`session.scalars(stmt)` returns ORM objects; `session.execute(stmt)` returns Row tuples.

---

## Querying

```python
from sqlalchemy import select, func, and_, or_

select(User).where(User.email.like("%@x.com"))
select(User).where(and_(User.name == "Ada", User.id > 1))
select(User).where(User.id.in_([1, 2, 3]))
select(func.count()).select_from(User)              # aggregate
select(User).join(Post).where(Post.title == "hi")   # join
select(User).order_by(User.id.desc()).limit(10).offset(20)

# group by + having
select(Post.user_id, func.count()).group_by(Post.user_id).having(func.count() > 2)

# eager loading (avoid N+1)
from sqlalchemy.orm import selectinload, joinedload
select(User).options(selectinload(User.posts))
```

---

## Relationships & Cascades

```python
posts: Mapped[list["Post"]] = relationship(
    back_populates="author",
    cascade="all, delete-orphan",     # delete posts when user deleted
    lazy="selectin",                  # loading strategy
)
u.posts.append(Post(title="x"))       # append → auto-inserts on commit
```

Loading strategies: `select`(lazy, N+1 risk), `selectin`(one extra query, good default), `joined`(single JOIN), `raise`(force explicit).

---

## Transactions

```python
with Session(engine) as session, session.begin():   # commits on exit, rolls back on error
    session.add(obj)
# or manual: session.commit() / session.rollback()
```

---

## Async (2.0)

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
engine = create_async_engine("postgresql+asyncpg://...")
async with AsyncSession(engine) as s:
    result = await s.scalars(select(User))
    users = result.all()
```

Pairs with [[asyncio]] / [[FastAPI]].

---

## Migrations (Alembic)

```bash
pip install alembic
alembic init migrations
# edit env.py → target_metadata = Base.metadata
alembic revision --autogenerate -m "add users"
alembic upgrade head          # apply ; alembic downgrade -1 to revert
alembic history / current
```

---

## Tips
- Use the **2.0 style** (`select()`, `Session`, `Mapped[]`) — the legacy `Query` API is deprecated.
- Beware the **N+1 query** problem — use `selectinload`/`joinedload` for relationships you'll access.
- One `Session` per unit of work / request; don't share sessions across threads.
- `session.get(Model, pk)` is the fast path for primary-key lookups.
- Let **Alembic autogenerate** migrations, but always review the generated script.
