---
tags: [coding, sql, mysql, databases]
type: cheatsheet
source: compiled reference (MySQL / MariaDB)
last-verified: 2026-08-27
---

# MySQL

## Up
- [[SQL]]

MySQL is the most widely deployed open-source relational database (MariaDB is a compatible fork). Uses the InnoDB storage engine by default. Backtick-quoted identifiers, `LIMIT` pagination.

---

## Connect & Admin (CLI)

```bash
mysql -u root -p                       # connect
mysql -u user -p -h host -P 3306 dbname
mysqldump -u user -p dbname > backup.sql          # backup
mysql -u user -p dbname < backup.sql              # restore
```

```sql
SHOW DATABASES;
USE mydb;
SHOW TABLES;
DESCRIBE users;           -- or  SHOW COLUMNS FROM users;
SHOW INDEX FROM users;
SHOW CREATE TABLE users;
SELECT VERSION();
```

---

## Databases, Users & Grants

```sql
CREATE DATABASE shop CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
DROP DATABASE shop;

CREATE USER 'app'@'%' IDENTIFIED BY 'password';
GRANT SELECT, INSERT, UPDATE, DELETE ON shop.* TO 'app'@'%';
GRANT ALL PRIVILEGES ON shop.* TO 'admin'@'localhost';
FLUSH PRIVILEGES;
REVOKE INSERT ON shop.* FROM 'app'@'%';
```

---

## Data Types (common)

| Category | Types |
|---|---|
| Integer | `TINYINT`, `SMALLINT`, `INT`, `BIGINT` (+ `UNSIGNED`) |
| Decimal | `DECIMAL(p,s)`, `FLOAT`, `DOUBLE` |
| String | `CHAR(n)`, `VARCHAR(n)`, `TEXT`, `ENUM(...)` |
| Date/Time | `DATE`, `DATETIME`, `TIMESTAMP`, `TIME`, `YEAR` |
| Other | `JSON`, `BOOLEAN` (alias for `TINYINT(1)`), `BLOB` |

---

## Tables

```sql
CREATE TABLE users (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    email      VARCHAR(255) NOT NULL UNIQUE,
    name       VARCHAR(100),
    role       ENUM('admin','user') DEFAULT 'user',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB;

CREATE TABLE orders (
    id      INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    total   DECIMAL(10,2),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

ALTER TABLE users ADD COLUMN age INT;
ALTER TABLE users MODIFY COLUMN name VARCHAR(150);
ALTER TABLE users DROP COLUMN age;
ALTER TABLE users ADD INDEX idx_email (email);
RENAME TABLE users TO members;
TRUNCATE TABLE logs;      -- fast delete-all (resets AUTO_INCREMENT)
```

---

## CRUD

```sql
INSERT INTO users (email, name) VALUES ('a@x.com', 'Ada');
INSERT INTO users (email, name) VALUES ('b@x.com','Bob'), ('c@x.com','Cy');

-- upsert
INSERT INTO users (id, email) VALUES (1, 'a@x.com')
ON DUPLICATE KEY UPDATE email = VALUES(email);

REPLACE INTO users (id, email) VALUES (1, 'a@x.com');

UPDATE users SET name = 'Ada L.' WHERE id = 1;
DELETE FROM users WHERE id = 5;
SELECT * FROM users WHERE email = 'a@x.com';
```

---

## Querying

```sql
SELECT name, COUNT(*) AS n
FROM orders
JOIN users ON users.id = orders.user_id
WHERE total > 100
GROUP BY name
HAVING n > 2
ORDER BY n DESC
LIMIT 10 OFFSET 20;              -- pagination

SELECT * FROM users WHERE name LIKE 'A%';
SELECT * FROM users WHERE created_at BETWEEN '2026-01-01' AND '2026-12-31';
SELECT * FROM users WHERE role IN ('admin','user');
SELECT IFNULL(name, 'unknown'), COALESCE(a, b, 'n/a') FROM users;
```

### Useful MySQL functions

```sql
CONCAT(first, ' ', last); CONCAT_WS('-', a, b);
UPPER(s); LOWER(s); TRIM(s); SUBSTRING(s, 1, 3); LENGTH(s);
NOW(); CURDATE(); DATE_ADD(d, INTERVAL 7 DAY); DATEDIFF(a, b);
DATE_FORMAT(created_at, '%Y-%m-%d'); YEAR(d); MONTH(d);
GROUP_CONCAT(name SEPARATOR ', ');
JSON_EXTRACT(doc, '$.key'); doc->>'$.key';
```

---

## Window Functions (8.0+)

```sql
SELECT name, total,
       ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY total DESC) AS rn,
       RANK()       OVER (ORDER BY total DESC) AS rnk,
       SUM(total)   OVER (PARTITION BY user_id) AS user_total
FROM orders;

WITH ranked AS (            -- CTEs supported in 8.0+
    SELECT *, ROW_NUMBER() OVER (ORDER BY total DESC) rn FROM orders
)
SELECT * FROM ranked WHERE rn <= 3;
```

---

## Transactions & Indexes

```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;        -- or ROLLBACK;

CREATE INDEX idx_user ON orders (user_id);
CREATE UNIQUE INDEX idx_email ON users (email);
CREATE INDEX idx_multi ON orders (user_id, created_at);
EXPLAIN SELECT * FROM orders WHERE user_id = 5;   -- query plan
ANALYZE TABLE orders;
```

---

## MySQL-Specific Notes

- Identifiers quoted with **backticks**: `` `select` ``. String literals use single quotes.
- Pagination: `LIMIT n OFFSET m` (or `LIMIT m, n`).
- `AUTO_INCREMENT` for surrogate keys; storage engine matters — **InnoDB** (transactions, FKs) vs MyISAM (legacy, no FKs).
- Use `utf8mb4` (not `utf8`) for full Unicode incl. emoji.
- `TIMESTAMP` is UTC-normalised and range-limited (1970–2038); `DATETIME` is not.
- Case sensitivity of table names depends on the OS/`lower_case_table_names` setting.
