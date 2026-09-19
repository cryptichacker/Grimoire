---
tags: [coding, sql, mssql, sqlserver, databases]
type: cheatsheet
source: compiled reference (Microsoft SQL Server)
last-verified: 2026-08-27
---

# MSSQL

## Up
- [[SQL]]

Microsoft SQL Server (MSSQL) is Microsoft's enterprise relational database. Its procedural dialect is **[[T-SQL]]** (covered separately) — this note focuses on the engine, DDL/DML, administration, and SQL-Server-specific query syntax. Bracket-quoted identifiers, `TOP`/`OFFSET…FETCH` pagination.

---

## Connect (CLI tools)

```bash
sqlcmd -S localhost -U sa -P 'Password' -d master      # classic client
sqlcmd -S server -E                                    # Windows auth
# newer: the `sqlcmd` (go) tool, or `mssql-cli`
```

```sql
SELECT @@VERSION;
SELECT name FROM sys.databases;             -- list databases
USE mydb;
SELECT name FROM sys.tables;                -- list tables
EXEC sp_help 'dbo.users';                   -- describe a table
EXEC sp_helpindex 'dbo.users';
SELECT * FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME='users';
```

---

## Databases, Schemas & Security

```sql
CREATE DATABASE Shop;
DROP DATABASE Shop;
CREATE SCHEMA sales;

CREATE LOGIN app WITH PASSWORD = 'pw';                 -- server level
CREATE USER app FOR LOGIN app;                         -- database level
GRANT SELECT, INSERT ON SCHEMA::dbo TO app;
ALTER ROLE db_datareader ADD MEMBER app;
```

---

## Data Types (common)

| Category | Types |
|---|---|
| Integer | `TINYINT`, `SMALLINT`, `INT`, `BIGINT`, `BIT` (boolean) |
| Decimal | `DECIMAL(p,s)`/`NUMERIC`, `MONEY`, `FLOAT`, `REAL` |
| String | `CHAR`, `VARCHAR(n)`, `VARCHAR(MAX)`, `NCHAR`, `NVARCHAR` (Unicode), `TEXT`(legacy) |
| Date/Time | `DATE`, `TIME`, `DATETIME2`, `DATETIME`, `DATETIMEOFFSET`, `SMALLDATETIME` |
| Other | `UNIQUEIDENTIFIER` (GUID), `XML`, `VARBINARY(MAX)`, `SQL_VARIANT` |

Prefer `NVARCHAR` for Unicode text and `DATETIME2` over legacy `DATETIME`.

---

## Tables

```sql
CREATE TABLE dbo.users (
    id         INT IDENTITY(1,1) PRIMARY KEY,      -- auto-increment
    email      NVARCHAR(255) NOT NULL UNIQUE,
    name       NVARCHAR(100),
    is_active  BIT DEFAULT 1,
    created_at DATETIME2 DEFAULT SYSUTCDATETIME()
);

CREATE TABLE dbo.orders (
    id      INT IDENTITY PRIMARY KEY,
    user_id INT FOREIGN KEY REFERENCES dbo.users(id) ON DELETE CASCADE,
    total   DECIMAL(10,2) CHECK (total >= 0)
);

ALTER TABLE dbo.users ADD age INT;
ALTER TABLE dbo.users ALTER COLUMN name NVARCHAR(150);
ALTER TABLE dbo.users DROP COLUMN age;
EXEC sp_rename 'dbo.users.name', 'full_name', 'COLUMN';
TRUNCATE TABLE dbo.logs;
```

---

## CRUD, Pagination & Upsert

```sql
INSERT INTO dbo.users (email, name) VALUES ('a@x.com', 'Ada');
INSERT INTO dbo.users (email, name)
OUTPUT INSERTED.id                       -- return generated values
VALUES ('b@x.com', 'Bob');

UPDATE dbo.users SET name = 'Ada L.' WHERE id = 1;
DELETE FROM dbo.users WHERE id = 5;

-- TOP-N and pagination
SELECT TOP 10 * FROM dbo.users ORDER BY created_at DESC;
SELECT * FROM dbo.users
ORDER BY id
OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;   -- pagination (2012+)

-- upsert via MERGE
MERGE dbo.users AS tgt
USING (SELECT 'a@x.com' AS email, 'Ada' AS name) AS src
ON tgt.email = src.email
WHEN MATCHED THEN UPDATE SET tgt.name = src.name
WHEN NOT MATCHED THEN INSERT (email, name) VALUES (src.email, src.name);
```

---

## Querying — SQL Server syntax

```sql
SELECT u.name, COUNT(*) AS n
FROM dbo.orders o
JOIN dbo.users u ON u.id = o.user_id
WHERE o.total > 100
GROUP BY u.name
HAVING COUNT(*) > 2
ORDER BY n DESC;

-- CTE + window functions
WITH ranked AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY total DESC) rn
    FROM dbo.orders
)
SELECT * FROM ranked WHERE rn = 1;

SELECT ISNULL(name, 'n/a'), COALESCE(a, b, 'x') FROM dbo.users;
SELECT IIF(total > 100, 'big', 'small') FROM dbo.orders;   -- inline if
SELECT CONCAT(first, ' ', last), FORMAT(created_at, 'yyyy-MM-dd') FROM dbo.users;
```

### Handy SQL Server functions

```sql
GETDATE(); SYSUTCDATETIME(); DATEADD(DAY, 7, d); DATEDIFF(DAY, a, b);
LEN(s); SUBSTRING(s,1,3); CHARINDEX('x', s); REPLACE(s,'a','b');
CAST(x AS INT); TRY_CAST(x AS INT); CONVERT(VARCHAR, d, 23);
NEWID();                              -- new GUID
STRING_AGG(name, ', ');               -- 2017+
```

---

## Indexes & Performance

```sql
CREATE CLUSTERED INDEX ix_created ON dbo.orders (created_at);   -- physical order
CREATE NONCLUSTERED INDEX ix_user ON dbo.orders (user_id) INCLUDE (total);
CREATE UNIQUE INDEX ux_email ON dbo.users (email);
DROP INDEX ix_user ON dbo.orders;

SET STATISTICS IO, TIME ON;
-- View the plan in SSMS/Azure Data Studio (Ctrl-M) or:
SET SHOWPLAN_ALL ON;
```

A table has at most one **clustered** index (defines row storage order — usually the PK); the rest are **nonclustered**.

---

## Transactions & Error Handling

```sql
BEGIN TRANSACTION;
BEGIN TRY
    UPDATE dbo.accounts SET balance -= 100 WHERE id = 1;
    UPDATE dbo.accounts SET balance += 100 WHERE id = 2;
    COMMIT;
END TRY
BEGIN CATCH
    ROLLBACK;
    THROW;      -- re-raise (or  RAISERROR(...))
END CATCH;
```

---

## MSSQL-Specific Notes

- Identifiers use **square brackets**: `[Order Details]`. Batches separated by `GO` (a sqlcmd/SSMS directive, not T-SQL).
- `IDENTITY(seed, increment)` for auto keys; `SCOPE_IDENTITY()` fetches the last one.
- Editions: Express (free, capped), Standard, Enterprise; also **Azure SQL Database** / **Managed Instance** in the cloud.
- System databases: `master`, `model`, `msdb`, `tempdb`.
- Prefer `NVARCHAR`/`DATETIME2`; `TEXT`/`NTEXT`/`IMAGE` are deprecated.
- Backups: `BACKUP DATABASE Shop TO DISK = 'shop.bak';` / `RESTORE DATABASE`.
