---
tags: [hacking, bug-bounty, injection, ssti, reports]
type: log
---

# SSTI

## Up
- [[Injection]]

Disclosed **Server-Side Template Injection** reports (often escalating to RCE). See [[Pattern]] → Injection (SSTI).

## Reports

### 2026-09-24 — Management Console editor privilege escalation to root SSH via nomad template injection and audit-forward (GitHub) — n/a (High)
- Source: [HackerOne #2332623](https://hackerone.com/reports/2332623)
- Type: Template injection chained to command injection and privilege escalation (GitHub Enterprise Server)
- Summary: On GitHub Enterprise Server, a user holding only the Management Console *editor* role could inject a Nomad template expression through the audit-log-forwarding configuration. The appliance rendered that template server-side while building the forwarding job, escalating a deliberately limited console role all the way to administrative SSH access on the appliance.
- Technique / pattern: Configuration fields on appliance management consoles are frequently rendered as templates by the orchestration layer that writes the service definition — Nomad and Consul templates, systemd unit files, log-forwarder configs, cron entries. Treat every free-text setting (syslog destinations, custom headers, hostnames, certificate fields) as a template sink and probe it with the *orchestrator's* expression syntax rather than generic web payloads. The privilege boundary worth attacking is the one inside the admin UI: an "editor" who cannot SSH but can write config that a root-level process will interpolate.
- Takeaway: Appliance management consoles have internal role tiers, and the values a lower tier may write are usually interpolated into a higher-tier execution context. Fingerprint the orchestration layer first, then test its template syntax in every field that role is allowed to edit.

### 2026-09-22 — Server Side Template Injection (The Plugin People) — $100
- Source: [Bugcrowd #c5699c3d](https://bugcrowd.com/disclosures/c5699c3d-69b6-49f5-b036-f53a5d3e3c6e/server-side-template-injection-ssti)
- Type: Server-Side Template Injection (P4)
- Summary: A server-side template injection was reported and fixed in The Plugin People's systems; the public disclosure does not name the endpoint or template engine.
- Technique / pattern: General method: submit a harmless arithmetic expression in engine syntax and check whether it comes back evaluated rather than literal, then fingerprint the engine to judge whether impact is info disclosure or code execution.
- Takeaway: Never build templates from user input; pass input as context variables to fixed templates and use sandboxed/logic-less modes.

### 2026-09-21 — Unrestricted File System Access via Twig Template Injection on dev-ucrm-billing-demo.ubnt.com (Ubiquiti Inc.) — n/a
- Source: [HackerOne #301406](https://hackerone.com/reports/301406)
- Type: SSTI - Twig template injection -> file system read (CVE-2017-0913)
- Summary: Twig template injection in Ubiquiti's UCRM billing demo instance allowed local file inclusion / unrestricted file system access; Ubiquiti noted exploitation required admin credentials and the demo ran in an isolated container.
- Technique / pattern: Admin-editable templates (invoice/email/notification templates) were abused to reach Twig include/source-style functionality and read server files. The public report does not disclose the exact payload.
- Takeaway: User-editable templates are an SSTI surface even when RCE is not reached - sandbox the template engine and restrict loader paths so include/source cannot read arbitrary files.

### 2026-09-18 — Server-side template injection in lodash _.template allows code execution on the server (lodash / Node.js) — n/a
- Source: [HackerOne #904672](https://hackerone.com/reports/904672)
- Type: SSTI (library-level template sink, JavaScript)
- Summary: The _.template function in lodash 4.17.15 compiled attacker-influenced template input into executable JavaScript, allowing code execution on the server in applications that passed untrusted data into it.
- Technique / pattern: Audit dependencies, not only application code, for template-compilation sinks: _.template, Handlebars.compile, ejs.render, pug.compile and similar take a template string and produce code. Trace whether any request-derived value reaches the template argument (as opposed to the data argument) - that distinction is the whole vulnerability.
- Takeaway: The template is code and the data is data. A widely used utility library can be the SSTI sink, so dependency review belongs in the same pass as reviewing your own render calls.

### 2026-09-18 — Server-side template injection in the Action View UJS test server (Ruby on Rails) — n/a
- Source: [HackerOne #942103](https://hackerone.com/reports/942103)
- Type: SSTI (ERB template rendered from user input)
- Summary: In `actionview/test/ujs/server.rb` (Rails 6.0.3.2) an echo-style route passed request-controlled input into template rendering, so injected template syntax was evaluated server-side and could lead to code execution on a host running that test server.
- Technique / pattern: Grep a repository for render/eval calls whose argument is built from `params` rather than a fixed template name, and extend the search to test harnesses, fixtures and dev servers — code excluded from the security review is still code someone will run.
- Takeaway: Template engines execute, they do not just interpolate: any path where user input becomes the template (not the template's data) is RCE, and auxiliary/test servers shipped inside a framework are a real attack surface when they escape the lab.

### 2026-09-18 — H1514 Server-Side Template Injection via Handlebars leading to RCE (Shopify) — n/a
- Source: [HackerOne #423541](https://hackerone.com/reports/423541)
- Type: SSTI (Handlebars / JavaScript template engine, escalated to RCE)
- Summary: A Shopify surface evaluated user-supplied input as a Handlebars template; the researcher escalated the template injection to remote code execution and published the full chain.
- Technique / pattern: Confirm evaluation with arithmetic probes, fingerprint the engine from its syntax and error text, then walk that engine's documented object and helper model — for Handlebars, the compiler internals and prototype access — until an execution primitive is reachable.
- Takeaway: JavaScript template engines are as exploitable as Jinja2/Twig/Smarty: treat any confirmed expression evaluation as a probable RCE, and fingerprint the engine before writing payloads instead of reusing Python-oriented ones.

### 2026-09-17 — SSTI leads to Command injection (curl) — n/a
- Source: [HackerOne #3584149](https://hackerone.com/reports/3584149)
- Type: SSTI escalating to OS command injection
- Summary: Attacker-influenced input was rendered by a template engine whose evaluation context reached process execution, turning a template-injection primitive into command execution.
- Technique / pattern: Confirm evaluation with an arithmetic probe ({{7*7}} / ${7*7} / #{7*7}), fingerprint the engine from its error output, then walk the object graph the engine exposes until an execution primitive (subprocess/shell helper) is reachable.
- Takeaway: Treat every confirmed template-evaluation finding as potential RCE — once the engine evaluates attacker input, escalation to command execution is the normal outcome, not the exceptional one.

### 2026-09-16 — uber.com may RCE by Flask Jinja2 Template Injection (Uber) — n/a
- Source: [hackerone #125980](https://hackerone.com/reports/125980)
- Type: SSTI (Server-Side Template Injection, Jinja2/Flask → RCE)
- Summary: Setting the rider profile name to a Jinja2 expression (e.g. `{{ '7'*7 }}`) caused Uber's backend to evaluate the template when rendering confirmation emails, proving server-side template injection with RCE potential.
- Technique / pattern: Inject template syntax (`{{7*7}}`) into fields later rendered by a server-side engine (emails, PDFs, notifications); an evaluated math expression confirms SSTI, then escalate via the engine's object model.
- Takeaway: Never pass user-controlled input into server-side templates as code; use logic-less/sandboxed templating and treat any field reflected into an email as a template sink.

### 2026-09-11 — SSTI via First Name field rendered in emails (Glovo) — n/a
- Source: [HackerOne #1104349](https://hackerone.com/reports/1104349)
- Type: SSTI (Server-Side Template Injection) — stored / second-order
- Summary: The First Name value supplied at signup was interpolated into server-side email templates (welcome/promo mail), so template syntax placed in that field was evaluated server-side when the email was generated.
- Technique / pattern: Seed a template probe (`${7*7}` / `{{7*7}}`) into a persisted profile field you suspect feeds a templated email, then trigger the email flow and inspect the received message for evaluation (49) — a second-order SSTI reachable only through the email pipeline.
- Takeaway: Profile fields that flow into templated emails are an overlooked SSTI sink; never render user input as part of a template — pass it as data.

### 2026-09-09 — SSTI via Smarty template → RCE (Unikrn)
- Source: [HackerOne #164224](https://hackerone.com/reports/164224)
- Type: SSTI (Server-Side Template Injection) → Remote Code Execution
- Summary: User-controlled input was rendered through the **Smarty** template engine; template syntax in the input was evaluated server-side, escalating from expression evaluation to **RCE**.
- Technique / pattern: Inject a template probe (`{7*7}` / `{$smarty.version}` / `{math}`); confirmed evaluation → use engine internals (`{php}`, `Smarty_Internal`, filter/static calls) to reach code execution.
- Takeaway: Never build templates from user input. Detect with polyglot probes per engine; SSTI often jumps straight to RCE (highest-impact injection).
