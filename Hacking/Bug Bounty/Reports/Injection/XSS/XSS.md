---
tags: [hacking, bug-bounty, injection, xss, reports]
type: log
---

# XSS

## Up
- [[Injection]]

Disclosed **Cross-Site Scripting** reports — reflected, stored, and DOM-based. See [[Pattern]] → Injection (XSS).

## Reports

### 2026-09-19 — Blind stored XSS triggered in the iOS app due to improper input handling (Nextcloud) — n/a
- Source: [HackerOne #575562](https://hackerone.com/reports/575562)
- Type: XSS (blind stored; native mobile client as the output context)
- Summary: A payload stored through Nextcloud's normal input surface was rendered without proper handling by the iOS application, executing in the app's web-backed view. Nothing fired in the web UI, so the issue was only observable on the mobile client.
- Technique / pattern: A native client is a separate output context with its own renderer. Plant blind-XSS callback payloads (an out-of-band beacon rather than `alert`) in every persisted field — display name, file name, share note, comment, calendar title — then open each surface in the iOS app, the Android app and the desktop client in turn. Web-side escaping does not travel with the data, and mobile views built on WebViews or attributed-string renderers routinely interpolate values that the web front end escapes.
- Takeaway: Test every client that renders the same stored data. 'Not exploitable in the browser' says nothing about the app, and blind payloads are the only practical way to cover sinks you cannot see.

### 2026-09-19 — DOM XSS in the fizzy.do import filename preview enables one-click account takeover (Basecamp) — n/a
- Source: [HackerOne #3608199](https://hackerone.com/reports/3608199)
- Type: XSS (DOM-based, filename source; chained to account takeover)
- Summary: The account-import page rendered the selected filename with `innerHTML` instead of `textContent`. A `.zip` whose *name* contained a `<button formaction=...>` element rendered that button inside the already-authenticated import form; one victim click submitted an email-change request with the victim's valid session and CSRF token, and redeeming the confirmation mail at the attacker's address completed a full takeover (CVSS 8.0).
- Technique / pattern: Two reusable ideas. First, the **filename is a DOM XSS source** — file pickers, drag-and-drop zones, upload progress lists and "selected file" previews echo `input.files[0].name` straight into the page, and it is user-controlled before any byte is uploaded. Second, when injection lands *inside an existing authenticated form*, you do not need to forge a request: inject a submit control (`<button formaction>`, `formmethod`, `formtarget`) and let the victim's own click ride the legitimate CSRF token. That sidesteps CSRF defences entirely and works even where script execution is constrained.
- Takeaway: Render filenames as text, never HTML. And when rating an HTML-injection, check what form surrounds the sink — injected submit controls inherit the page's session and anti-CSRF token, turning "HTML injection" into one-click takeover.

### 2026-09-19 — Stored XSS in Rocket.Chat HTML file export, unauthenticated entry via LiveChat (Rocket.Chat) — n/a
- Source: [HackerOne #3779690](https://hackerone.com/reports/3779690)
- Type: XSS (stored, second-order, triggered in an exported artifact)
- Summary: The export-room-messages routine inserted `messageObject.msg` raw into generated HTML with no escaping. An anonymous LiveChat visitor could register and post a message through the public API, and the payload executed when an admin later exported the conversation as HTML and opened it — also affecting the GDPR data-download feature (CVSS 5.4).
- Technique / pattern: Treat **generated artifacts as an output context of their own**: HTML/PDF exports, emailed digests, invoices, GDPR data downloads, printable views and report generators are frequently built by a separate code path that never inherits the live UI's escaping. Plant a payload through the cheapest entry point (here an unauthenticated LiveChat visitor), then trigger the export as the privileged user. The exported file opens from `file://`, so there is no CSP, no origin indicator and no session-bound sandbox — which is precisely what makes the credential-harvesting and exfiltration variants work.
- Takeaway: Escape at every serializer, not just the one the SPA uses. Rate export-triggered XSS by who opens the file — an admin opening a support export from `file://` is a worse context than the app's own origin, not a better one.

### 2026-09-18 — Account takeover via email-only authentication chained with stored XSS exposing PII (U.S. General Services Administration) — n/a
- Source: [HackerOne #1483201](https://hackerone.com/reports/1483201)
- Type: Stored XSS chained with improper authentication
- Summary: A GSA application authenticated users on an email address alone, with no password or secret required. An attacker could log in as any known user and then persist script into the profile fields of that account, with personally identifiable information such as phone numbers exposed as a result.
- Technique / pattern: Chain an access-control weakness with an injection sink rather than reporting either alone: once a weak login gives control of an arbitrary account, every profile field that other users render becomes a stored-XSS vector, and the combined report demonstrates real account takeover and PII exposure instead of a self-XSS.
- Takeaway: Rate a stored-XSS sink by who can reach it. An identifier-only login turns 'self-XSS in a profile field' into attacker-controlled script inside any victim's account.

### 2026-09-18 — Stored XSS in Notes with a CSP bypass for gitlab.com (GitLab) — n/a
- Source: [HackerOne #1481207](https://hackerone.com/reports/1481207)
- Type: Stored XSS (markdown HTML injection chained to a CSP bypass)
- Summary: HTML injection surviving in GitLab's syntax_highlight_filter markdown pipeline let an attacker inject a <base> tag into notes, wiki pages and issue descriptions; that repointed relative script URLs at an attacker-controlled host, defeating the nonce-based CSP and executing script for every viewer.
- Technique / pattern: When a previous fix removed a particular XSS sink but left the underlying HTML injection, look for tags that change document semantics rather than tags that execute directly - <base> rewrites the resolution of every relative URL on the page. Then enumerate which script files the page expects to load and host matching paths on the attacker origin: those requests inherit the page's nonce, so the CSP is satisfied by construction.
- Takeaway: Patching the payload instead of the injection point leaves the bug alive, and a nonce-based CSP protects nothing once an attacker controls the base URL that relative script paths resolve against.

### 2026-09-18 — Reflected XSS chained with a CSRF-able password change for one-click account takeover (TikTok) — n/a
- Source: [HackerOne #968082](https://hackerone.com/reports/968082)
- Type: XSS (reflected) chained with CSRF
- Summary: A URL parameter was reflected without proper sanitisation, giving reflected XSS; separately, the endpoint that sets a password for accounts created through third-party (social) login lacked CSRF protection. Chaining the two turned a single click into a full account takeover.
- Technique / pattern: After finding a reflected sink, look for a state-changing endpoint whose protection assumes a same-origin request; script running in the victim's origin satisfies that assumption, so an unprotected password-set route becomes reachable and upgrades the XSS from session theft to persistent takeover.
- Takeaway: Rate an XSS by what the origin can do, not by the alert box: accounts created via social login often have a weaker password-set path than password accounts, and that path is what turns a reflection into a takeover.

### 2026-09-18 — Stored XSS at linkpop.com via client-side-only URL validation (Shopify) — n/a
- Source: [HackerOne #1441988](https://hackerone.com/reports/1441988)
- Type: XSS (stored; client-side validation bypassed at the request layer)
- Summary: Linkpop validated link URLs only in the browser, so intercepting and modifying the request with a proxy stored an unsanitized payload, which then executed for anyone visiting the page's shareable public link.
- Technique / pattern: When a field looks protected, check whether the rule is enforced client-side only — replay the request through an intercepting proxy with the payload, then confirm persistence by loading the public/share view as a different user.
- Takeaway: Client-side validation is a UX affordance, not a control; sanitize and encode server-side, and prioritize payloads that propagate through share links because they reach the widest victim set.

### 2026-09-18 — Cache poisoning leading to account takeover via XSS (Expedia Group) — n/a
- Source: [HackerOne #1760213](https://hackerone.com/reports/1760213)
- Type: XSS (reflected into a JavaScript response, escalated via web cache poisoning)
- Summary: A `hav` cookie value was reflected unsanitized into a JavaScript variable assignment on an Abritel endpoint that returns JS; the filter stripped double quotes but not `>`, letting the researcher break out of the string context and, through cache poisoning, serve the payload to other users.
- Technique / pattern: Hunt for inputs reflected into responses with a JavaScript content type, test which metacharacters the filter actually removes rather than assuming quotes are enough, then check whether the reflecting input is part of the cache key — an unkeyed input turns a self-only reflection into a stored payload for every cached visitor.
- Takeaway: Partial sanitization is no sanitization: enumerate every metacharacter for the exact output context, and always test cache keying, since an unkeyed reflection escalates severity from self-XSS to mass account takeover.

### 2026-09-17 — Stored XSS on https://paypal.com/signin via cache poisoning (PayPal) — $18,900
- Source: [HackerOne #488147](https://hackerone.com/reports/488147)
- Type: Stored XSS (via web cache poisoning)
- Summary: An unkeyed request input was reflected into the PayPal sign-in page and then cached by the front-end cache, so the injected script was served from cache to every subsequent visitor of the login page.
- Technique / pattern: Identify request components that influence the response but are not part of the cache key (odd headers, query parameters ignored by the cache), inject through one, then re-request as a clean client to confirm the poisoned response is persisted and served to others.
- Takeaway: A reflected XSS in front of a shared cache is effectively stored XSS — always check cache headers and unkeyed inputs before rating a reflection low impact.

### 2026-09-17 — Stored XSS in messages (SideFX) — n/a
- Source: [HackerOne #1669764](https://hackerone.com/reports/1669764)
- Type: Stored XSS
- Summary: A stored XSS payload placed in the messaging feature executed in the victim's browser when they viewed the message, allowing theft of the victim's session id and account hijack.
- Technique / pattern: Posted a message carrying a script payload that persisted and rendered unsanitized to other users; on view it fired and exfiltrated the session id.
- Takeaway: Stored XSS in any user-to-user channel (messages, forums) that renders to other users is high-impact — encode on output and keep session cookies HttpOnly so they can't be scripted out.

### 2026-09-15 — Stored-XSS injected in Wiki page via Banzai pipeline (GitLab) — n/a
- Source: [HackerOne #2257080](https://hackerone.com/reports/2257080)
- Type: Stored XSS (via server-side markdown rendering, CSP bypass)
- Summary: A flaw in GitLab's Banzai markdown reference filter (AbstractReferenceFilter) allowed injecting arbitrary HTML into a rendered Wiki page (via the _sidebar), producing stored XSS that also bypassed CSP.
- Technique / pattern: Abuse a reference-parsing regex that matched only a link prefix, so crafted reference syntax placed in Wiki content was rewritten into attacker-controlled HTML during server-side markdown rendering.
- Takeaway: Server-side markdown/reference processors are a rich stored-XSS surface; regex link parsing must fully validate matches, and output must be sanitized after all transform passes.

### 2026-09-14 — Stored XSS via hacker-facing custom fields (HackerOne) — n/a
- Source: [HackerOne #1173040](https://hackerone.com/reports/1173040)
- Type: Cross-Site Scripting (XSS) — stored
- Summary: The "Hacker Facing Custom Fields" feature stored text-field values that were rendered without adequate output encoding, producing stored XSS that executed in a viewer's browser (demonstrated on IE11).
- Technique / pattern: Plant a script payload into a persisted custom/profile field, then confirm it executes when the field is rendered to another or privileged user; legacy-browser contexts (IE11) can widen exploitable sinks.
- Takeaway: Context-encode every stored value on output, including in configurable/custom fields; do not rely on the input form to sanitize.


### 2026-09-13 — Stored XSS via SVG file upload (Nextcloud) — n/a
- Source: [HackerOne #3357808](https://hackerone.com/reports/3357808)
- Type: Stored XSS (file upload)
- Summary: An uploaded SVG containing embedded script was served inline in the site origin, so opening the stored file executed attacker JavaScript in other users' sessions (tracked as GHSA-qcw2-p26m-9gc5).
- Technique / pattern: Where an image upload accepts SVG, embed a script/event handler in the SVG XML and then load the stored file directly; SVGs are XML documents and run script when rendered inline rather than sanitized or served as attachments.
- Takeaway: Sanitize SVG uploads (strip scripts) or serve user files from a sandbox origin with Content-Disposition: attachment and a restrictive CSP.

### 2026-09-12 — Stored XSS in custom reports feature (X / xAI) — n/a
- Source: [HackerOne #485748](https://hackerone.com/reports/485748)
- Type: Stored XSS
- Summary: A custom "reports" feature stored user-supplied input without sanitization; the injected script executed in the browser of any user who later viewed the report.
- Technique / pattern: Submitted an HTML/JS payload into a report field that was rendered back unescaped on view, giving persistent script execution against other users.
- Takeaway: Encode on output for every stored field rendered in a shared view; stored XSS is high-impact because it fires without the victim taking any unusual action.

### 2026-09-10 — Stored XSS in "Create Groups" (GitLab) — n/a
- Source: [HackerOne #647130](https://hackerone.com/reports/647130)
- Type: Stored XSS
- Summary: A field in GitLab's group-creation flow stored attacker markup that was later rendered without proper output encoding, executing JavaScript when the group was viewed.
- Technique / pattern: Plant a payload in a persisted field (group name/description), then load the page that echoes it; if rendered unescaped in HTML context the script runs for anyone viewing.
- Takeaway: Stored fields shown to other users must be output-encoded per context; group/org name fields are commonly missed sinks.

### 2026-09-09 — Stored XSS in Discounts section (Shopify)
- Source: [HackerOne #618031](https://hackerone.com/reports/618031)
- Type: Stored XSS (privileged/admin context)
- Summary: A discount field value was stored and later rendered **without output encoding** in the admin UI, executing attacker JavaScript in an authenticated context.
- Technique / pattern: Plant a payload in a stored field that's echoed on another page; if it renders unescaped in HTML/JS/attribute context → script execution. Prioritize fields shown to *other/privileged* users.
- Takeaway: Encode on output per context; stored XSS in an admin panel can pivot to session theft / account actions.
