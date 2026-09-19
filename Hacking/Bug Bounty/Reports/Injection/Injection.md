---
tags: [hacking, bug-bounty, injection, reports]
type: moc
---

# Injection

## Up
- [[Reports]]

Disclosed **injection** reports, split by type. Core idea: untrusted input crosses into a parser/interpreter (SQL engine, browser DOM, template engine, OS shell, deserializer) without proper separation/encoding. See [[Pattern]] → Injection.

## Subtypes
- [[SQLi]] — SQL injection (error/boolean/time-based, UNION, blind, second-order)
- [[XSS]] — Cross-Site Scripting (reflected, stored, DOM)
- [[SSTI]] — Server-Side Template Injection
- [[RCE]] — remote code execution via injection chains (OS command injection, insecure deserialization, file-upload, SSTI→RCE)
