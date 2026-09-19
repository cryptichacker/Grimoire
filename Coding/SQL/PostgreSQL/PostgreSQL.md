---
tags: [coding, sql, postgresql, databases]
type: cheatsheet
source: compiled reference (PostgreSQL)
last-verified: 2026-08-27
---

# PostgreSQL

## Up
- [[SQL]]

PostgreSQL ("Postgres") is a powerful, standards-compliant open-source relational database with rich types (JSONB, arrays, ranges), extensibility, and strong SQL feature coverage. Double-quoted identifiers, `LIMIT`/`OFFSET` pagination.

---

## psql (CLI) Essentials

```bash
psql -U user -d dbname -h host -p 5432
pg_dump -U user dbname > backup.sql              # backup
pg_dump -Fc dbname > backup.dump                 # custom format
psql -U user dbname < backup.sql                 # restore
pg_restore -U user -d dbname backup.dump
```

```
\l          list databases        \dt      list tables
\c dbname   connect               \d table describe table
\dn         list schemas          \di      list indexes
\du         list roles            \df      list functions
\x          toggle expanded view  \timing  toggle query timing
\e          edit in $EDITOR       \q       quit
```

---

## Databases, Roles & Schemas

```sql
CREATE DATABASE shop ENCODING 'UTF8';
CREATE ROLE app LOGIN PASSWORD 'pw';
GRANT SELECT, INSERT ON ALL TABLES IN SCHEMA public TO app;
ALTER ROLE app CREATEDB;

CREATE SCHEMA analytics;
SET search_path TO analytics, public;
```

---

## Data Types (highlights)

| Category | Types |
|---|---|
| Integer | `SMALLINT`, `INTEGER`, `BIGINT`, `SERIAL`/`BIGSERIAL` (auto), `GENERATED ALWAYS AS IDENTITY` |
| Decimal | `NUMERIC(p,s)`, `REAL`, `DOUBLE PRECISION` |
| String | `VARCHAR(n)`, `TEXT`, `CHAR(n)` |
| Date/Time | `DATE`, `TIME`, `TIMESTAMP`, `TIMESTAMPTZ`, `INTERVAL` |
| Rich | `BOOLEAN`, `UUID`, `JSON`/`JSONB`, `ARRAY`, `HSTORE`, `INET`, `TSVECTOR`, ranges |

---

## Tables

```sql
CREATE TABLE users (
    id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email      TEXT NOT NULL UNIQUE,
    name       TEXT,
    tags       TEXT[],                       -- array column
    profile    JSONB,                        -- binary JSON
    created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE orders (
    id      BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE,
    total   NUMERIC(10,2) CHECK (total >= 0)
);

ALTER TABLE users ADD COLUMN age INT;
ALTER TABLE users ALTER COLUMN name SET NOT NULL;
ALTER TABLE users RENAME COLUMN name TO full_name;
DROP TABLE IF EXISTS logs CASCADE;
```

---

## CRUD & Upsert

```sql
INSERT INTO users (email, name) VALUES ('a@x.com', 'Ada')
RETURNING id;                              -- return generated values

-- upsert (ON CONFLICT)
INSERT INTO users (email, name) VALUES ('a@x.com', 'Ada')
ON CONFLICT (email) DO UPDATE SET name = EXCLUDED.name;
INSERT INTO users (email) VALUES ('a@x.com')
ON CONFLICT (email) DO NOTHING;

UPDATE users SET name = 'Ada L.' WHERE id = 1 RETURNING *;
DELETE FROM users WHERE id = 5;
```

---

## Querying — Postgres strengths

```sql
-- CTEs (incl. recursive)
WITH RECURSIVE tree AS (
    SELECT id, parent_id, name FROM categories WHERE parent_id IS NULL
    UNION ALL
    SELECT c.id, c.parent_id, c.name
    FROM categories c JOIN tree t ON c.parent_id = t.id
)
SELECT * FROM tree;

-- Window functions
SELECT name, total,
       RANK() OVER (PARTITION BY user_id ORDER BY total DESC),
       LAG(total) OVER (ORDER BY created_at),
       SUM(total) OVER (ORDER BY created_at ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
FROM orders;

-- DISTINCT ON (first row per group)
SELECT DISTINCT ON (user_id) * FROM orders ORDER BY user_id, created_at DESC;

-- FILTER, GROUPING SETS
SELECT dept,
       COUNT(*) FILTER (WHERE active) AS active_count
FROM staff GROUP BY dept;
```

### JSONB & arrays

```sql
SELECT profile->>'name'            FROM users;   -- text
SELECT profile->'address'->>'city' FROM users;   -- nested
SELECT * FROM users WHERE profile @> '{"vip": true}';   -- containment
SELECT * FROM users WHERE profile ? 'email';            -- key exists
SELECT jsonb_array_elements(profile->'roles') FROM users;

SELECT * FROM users WHERE 'admin' = ANY(tags);          -- array membership
SELECT array_agg(name) FROM users;
SELECT unnest(tags) FROM users;
```

---

## Indexes (Postgres has many kinds)

```sql
CREATE INDEX idx_email ON users (email);
CREATE UNIQUE INDEX ON users (email);
CREATE INDEX idx_gin ON users USING GIN (profile);      -- JSONB / arrays / FTS
CREATE INDEX idx_partial ON orders (user_id) WHERE total > 100;
CREATE INDEX idx_expr ON users (lower(email));           -- expression index
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 5;
```

Index types: **B-tree** (default), **GIN** (JSONB/arrays/full-text), **GiST** (geometric/ranges), **BRIN** (huge append-only tables), **Hash**.

---

## Transactions

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
SAVEPOINT s1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- ROLLBACK TO s1;
COMMIT;                                    -- or ROLLBACK;

-- isolation levels
BEGIN ISOLATION LEVEL SERIALIZABLE;
```

---

## Postgres-Specific Notes

- Identifiers double-quoted (`"table"`); unquoted names fold to **lowercase** (opposite of SQL Server).
- Use `TIMESTAMPTZ` (stored UTC) over `TIMESTAMP` for real timestamps.
- `SERIAL` is legacy; prefer `GENERATED ALWAYS AS IDENTITY`.
- `RETURNING` on INSERT/UPDATE/DELETE avoids a second round-trip.
- Prefer **JSONB** over JSON (indexable, binary). Full-text search built in via `tsvector`/`tsquery`.
- Extensions add power: `CREATE EXTENSION postgis / pg_trgm / uuid-ossp / pgcrypto;`.
- Run `VACUUM (ANALYZE)` / rely on autovacuum to reclaim space and refresh stats.
