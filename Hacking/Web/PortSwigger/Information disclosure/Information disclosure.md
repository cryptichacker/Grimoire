---
tags: [hacking, web, portswigger, information-disclosure]
type: vulnerability-category
source: https://portswigger.net/web-security/information-disclosure
last-verified: 2026-08-25
---

# Information disclosure

## Up
- [[PortSwigger]]

## What it is
Information disclosure (information leakage) is when a website unintentionally reveals sensitive information — user data, business data, or technical details about the app and its infrastructure that aid further attacks.

## Common examples
- **Hidden files/directories** exposed via `robots.txt`, sitemap, or directory listing.
- **Temporary/backup files** — source with `.bak`, `.old`, `~`, `.txt` copies of scripts revealing logic and credentials.
- **Verbose error messages** — stack traces leaking database table/column names, file paths, library versions.
- **Debug / diagnostic pages** left enabled in production, dumping config, environment variables, session data.
- **Source code disclosure** — via backup files or exposed version-control history (`/.git`).
- **Developer comments** in HTML/JS referencing hidden endpoints or credentials.
- **Hardcoded secrets** — API keys, internal IPs, DB credentials in client-side code.
- **Behavioural hints** — different responses that reveal whether a username/resource exists.

## How it arises
1. **Content not removed** — debug comments, test endpoints, TODOs left in production.
2. **Insecure configuration** — verbose errors/debug features enabled, default settings on third-party tech.
3. **Flawed design** — responses that differentiate states (e.g. leaking existence via redirects that still include data).

## Finding and exploiting it
- **Fuzz** interesting parameters and observe anomalies; use Burp Scanner and engagement tools.
- **Engineer informative responses** — deliberately send malformed/unexpected input to trigger errors that leak internals.
- Review `robots.txt`, `/sitemap.xml`, comments, JS source, and probe for backup files and `.git`.
- Check verbose errors for versions/paths, and diff account/error pages for leaked data.

## Prevention
- Educate developers on what counts as sensitive; review before shipping.
- Automate checks to strip comments and disable debug/diagnostic features in production.
- Return **generic** error messages; log details server-side only.
- Audit third-party technology's default configuration and its security implications.

## Labs
5 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Information disclosure in error messages | Apprentice | Trigger a stack trace that leaks the framework/version, then exploit. | |
| Information disclosure on debug page | Apprentice | Find a debug page (via comments/`robots.txt`) exposing secrets/env vars. | |
| Source code disclosure via backup files | Apprentice | Find `/backup` or `.bak` source revealing DB credentials. | |
| Authentication bypass via information disclosure | Practitioner | Leak the header/param the admin panel checks (e.g. via TRACE) and reuse it. | |
| Information disclosure in version control history | Practitioner | Download `/.git`, inspect history for a committed secret/password change. | |
