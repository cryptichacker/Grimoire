---
tags: [coding, sql, tsql, sqlserver, databases]
type: cheatsheet
source: compiled reference (Transact-SQL)
last-verified: 2026-08-27
---

# T-SQL

## Up
- [[SQL]]

Transact-SQL (T-SQL) is Microsoft's procedural extension to SQL used by [[MSSQL]] SQL Server and Azure SQL. It adds variables, control-of-flow, error handling, stored procedures, functions, triggers, and cursors on top of standard SQL DML/DDL.

---

## Variables & Batches

```sql
DECLARE @count INT = 0;
DECLARE @name NVARCHAR(50), @today DATE = GETDATE();

SET @count = 10;
SELECT @count = COUNT(*) FROM users;      -- assign from a query
PRINT @count;                             -- to messages
SELECT @count AS total;                   -- to results

GO         -- batch separator (SSMS/sqlcmd directive)
GO 5       -- run the previous batch 5 times
```

---

## Control-of-Flow

```sql
-- IF / ELSE
IF @count > 100
    PRINT 'many';
ELSE
BEGIN
    PRINT 'few';
    SET @count = 0;
END

-- WHILE
WHILE @count < 10
BEGIN
    SET @count += 1;
    IF @count = 5 CONTINUE;
    IF @count = 8 BREAK;
END

-- CASE (expression, not a statement)
SELECT name,
    CASE WHEN age < 18 THEN 'minor'
         WHEN age < 65 THEN 'adult'
         ELSE 'senior' END AS band
FROM users;

-- IIF and CHOOSE
SELECT IIF(age >= 18, 'adult', 'minor');
SELECT CHOOSE(2, 'a', 'b', 'c');          -- 'b'

WAITFOR DELAY '00:00:05';                 -- sleep 5s
```

---

## Error Handling (TRY/CATCH)

```sql
BEGIN TRY
    BEGIN TRANSACTION;
    INSERT INTO accounts (id, balance) VALUES (1, 100);
    UPDATE accounts SET balance = balance / 0;      -- error
    COMMIT;
END TRY
BEGIN CATCH
    IF @@TRANCOUNT > 0 ROLLBACK;
    SELECT ERROR_NUMBER()   AS num,
           ERROR_MESSAGE()  AS msg,
           ERROR_LINE()     AS line,
           ERROR_SEVERITY() AS sev;
    THROW;                          -- re-raise the caught error
    -- THROW 50001, 'Custom message', 1;   -- raise a custom error
    -- RAISERROR('Legacy %d', 16, 1, @val);
END CATCH;
```

---

## Stored Procedures

```sql
CREATE OR ALTER PROCEDURE dbo.GetUserOrders
    @UserId INT,
    @MinTotal DECIMAL(10,2) = 0,        -- default value
    @OrderCount INT OUTPUT              -- output parameter
AS
BEGIN
    SET NOCOUNT ON;
    SELECT * FROM orders
    WHERE user_id = @UserId AND total >= @MinTotal;

    SELECT @OrderCount = @@ROWCOUNT;
    RETURN 0;                          -- status code
END;
GO

-- execute
DECLARE @n INT;
EXEC dbo.GetUserOrders @UserId = 5, @MinTotal = 50, @OrderCount = @n OUTPUT;
PRINT @n;
```

---

## User-Defined Functions

```sql
-- scalar function
CREATE OR ALTER FUNCTION dbo.FullName(@first NVARCHAR(50), @last NVARCHAR(50))
RETURNS NVARCHAR(101)
AS
BEGIN
    RETURN @first + ' ' + @last;
END;
GO
SELECT dbo.FullName(first, last) FROM users;

-- inline table-valued function (fast, inlines like a view)
CREATE OR ALTER FUNCTION dbo.OrdersOver(@min DECIMAL(10,2))
RETURNS TABLE
AS
RETURN (SELECT * FROM orders WHERE total > @min);
GO
SELECT * FROM dbo.OrdersOver(100);
```

---

## Temp Tables, Table Variables & CTEs

```sql
-- local temp table (# = session-scoped, ## = global)
CREATE TABLE #tmp (id INT, name NVARCHAR(50));
INSERT INTO #tmp SELECT id, name FROM users;
DROP TABLE #tmp;

-- table variable
DECLARE @t TABLE (id INT, name NVARCHAR(50));
INSERT INTO @t VALUES (1, 'Ada');

-- SELECT INTO (create + fill)
SELECT * INTO #snapshot FROM users;

-- CTE
WITH recent AS (
    SELECT * FROM orders WHERE created_at > DATEADD(DAY, -30, GETDATE())
)
SELECT user_id, SUM(total) FROM recent GROUP BY user_id;
```

---

## Triggers

```sql
CREATE OR ALTER TRIGGER trg_audit
ON dbo.users
AFTER INSERT, UPDATE
AS
BEGIN
    SET NOCOUNT ON;
    INSERT INTO audit_log (user_id, action, at)
    SELECT id, 'change', SYSUTCDATETIME() FROM inserted;  -- 'inserted'/'deleted' pseudo-tables
END;
GO
```

---

## Cursors (use sparingly — prefer set-based SQL)

```sql
DECLARE @id INT, @name NVARCHAR(50);
DECLARE c CURSOR FOR SELECT id, name FROM users;
OPEN c;
FETCH NEXT FROM c INTO @id, @name;
WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT CONCAT(@id, ': ', @name);
    FETCH NEXT FROM c INTO @id, @name;
END
CLOSE c; DEALLOCATE c;
```

---

## Dynamic SQL

```sql
DECLARE @sql NVARCHAR(MAX) = N'SELECT * FROM users WHERE id = @id';
EXEC sp_executesql @sql, N'@id INT', @id = 5;      -- parameterised (safe)
-- avoid EXEC(@concatenated_string) — SQL injection risk
```

---

## System Functions Worth Knowing

```sql
@@ROWCOUNT      -- rows affected by last statement
@@IDENTITY / SCOPE_IDENTITY()   -- last identity value
@@TRANCOUNT     -- open transaction nesting level
@@ERROR         -- error code of last statement
NEWID(); SYSUTCDATETIME(); GETUTCDATE();
ISNULL(x, 0); COALESCE(a, b, c); NULLIF(a, b);
CAST/CONVERT/TRY_CONVERT; STRING_SPLIT('a,b,c', ',');
```

---

## Notes

- T-SQL statements end with optional `;` (increasingly required); batches split by `GO`.
- Set-based operations beat cursors/loops almost always — reach for joins, window functions, and CTEs first.
- Use `sp_executesql` with parameters (never string-concatenated dynamic SQL) to avoid injection and enable plan reuse.
- `SET NOCOUNT ON` in procs/triggers suppresses "N rows affected" chatter and reduces overhead.
- `CREATE OR ALTER` (2016 SP1+) simplifies idempotent deployment of procs/functions/views/triggers.
