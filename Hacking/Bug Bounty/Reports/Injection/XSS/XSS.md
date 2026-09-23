---
tags: [hacking, bug-bounty, injection, xss, reports]
type: log
---

# XSS

## Up
- [[Injection]]

Disclosed **Cross-Site Scripting** reports — reflected, stored, and DOM-based. See [[Pattern]] → Injection (XSS).

## Reports

### 2026-09-22 — Stored XSS in /admin/products and /admin/collections rich-text HTML editor (Shopify) — $5,300
- Source: [HackerOne #1147433](https://hackerone.com/reports/1147433)
- Type: Stored XSS (rich-text editor HTML mode)
- Summary: The product and collection description editors offered an HTML view whose contents were saved with insufficient sanitisation, so a payload such as an `img` tag with an `onerror` handler persisted and executed for anyone later opening that product or collection in the admin panel.
- Technique / pattern: Editors that expose a "view HTML / source" mode bypass the WYSIWYG's own escaping — paste raw markup there rather than typing it in the visual editor, and then check every surface that re-renders the stored field (list views, previews, storefront, exports).
- Takeaway: Sanitisation must happen server-side on save and again on render; an admin-facing stored XSS is high value because the viewers are privileged sessions.

### 2026-09-22 — Stored XSS via client-side template injection in the account address form (WordPress / WooCommerce) — n/a (bounty awarded, amount not shown)
- Source: [HackerOne #250837](https://hackerone.com/reports/250837)
- Type: Stored XSS through template-expression injection in a client-side binding
- Summary: The name field on `/my-account/edit-address/` stored an AngularJS-style template expression — `{{constructor.constructor('alert(1)')()}}` — which the account page later evaluated, running script in the victim's session. Alone it looked like self-XSS, but combined with registering an account under someone else's (unverified) email, the victim inherited the stored payload after reclaiming the account by password reset.
- Technique / pattern: Probe persisted profile fields with `{{7*7}}` as well as with tags: where a client-side framework binds user data into the DOM, expression evaluation is the sink and the classic filter-defeating trick is reaching `constructor.constructor` to build a function. Escalate "self-XSS" by finding a flow that hands the poisoned account to a real user (pre-registration + account recovery).
- Takeaway: Template expressions are an output context of their own — escape them, and never bind untrusted data into a live templating scope. Self-XSS becomes real XSS whenever account ownership can transfer.
### 2026-09-22 — Stored XSS on inventory-retrieve.php (Revive Adserver) — n/a
- Source: [HackerOne #3399809](https://hackerone.com/reports/3399809)
- Type: Stored XSS (CVE-2025-52667)
- Summary: The campaign `Name` field was stored without output encoding and rendered raw by `inventory-retrieve.php` and `campaign-edit.php`, executing JavaScript for any admin viewing those pages.
- Technique / pattern: Created a campaign whose name breaks out of an HTML attribute (e.g. `"><img src=x onerror=alert(document.domain)>`), saved it, then opened `inventory-retrieve.php?clientid=1` to trigger it.
- Takeaway: Low-privilege name/label fields viewed later by admins are classic stored-XSS sinks; test every render location, not just the input form.

### 2026-09-22 — Reflected XSS via URL path in archive endpoint (NASA VDP) — n/a
- Source: [Bugcrowd #655092e1](https://bugcrowd.com/disclosures/655092e1-8200-4089-a42f-3803edfdeadd/reflected-xss-via-url-path-in-archive-endpoint)
- Type: Reflected XSS (path-based), CSP-mitigated
- Summary: On `seabass.gsfc.nasa.gov`, anything appended after the `/archive/` path segment was reflected into the page unescaped; a strict CSP blocked actual script execution.
- Technique / pattern: Appended HTML/attribute markup to the URL path after `/archive/` and confirmed it was echoed unencoded into the DOM, demonstrating the injection even though CSP prevented execution.
- Takeaway: Path segments are user input too (check directory-listing and 404-style pages that echo the path); CSP is defense-in-depth, not a substitute for output encoding.

### 2026-09-21 — DOM-based XSS via Cesium Sandcastle `#c=` URL fragment (NASA VDP)
- Source: [Bugcrowd #6765826c](https://bugcrowd.com/disclosures/6765826c-df24-47cd-afa4-c158bde0e4b6/dom-based-cross-site-scripting-through-the-publicly-exposed-cesium-sandcastle-shared-code-feature)
- Type: DOM-based XSS (reflected via URL fragment)
- Summary: The Cesium Sandcastle shared-code feature took attacker JavaScript from the `#c=` URL fragment, decoded/decompressed it, and executed it same-origin on `gpm.nasa.gov` with no sandboxing.
- Technique / pattern: Craft a link whose fragment carries encoded/compressed JS in `#c=`; opening it makes the page decode and `eval` the payload in the trusted origin — no server round-trip, so the fragment never hits server logs or WAF.
- Takeaway: 'Share this code' features that execute fragment content are a client-side sink — run shared code in a sandboxed iframe/worker on a throwaway origin, never eval it same-origin.

### 2026-09-21 — Stored XSS via SVG upload in chat.line.biz (LINE / LY Corporation) — $100
- Source: [HackerOne #3008878](https://hackerone.com/reports/3008878)
- Type: Stored XSS (SVG file upload)
- Summary: The chat management interface accepted SVG uploads containing embedded JavaScript and served them inline, so scripts ran in another user's browser when the file was viewed.
- Technique / pattern: Upload an SVG with an inline `<script>`/event handler through the chat file feature; when a staff user opens it in the management UI it renders inline from a trusted origin and the script executes.
- Takeaway: SVGs are active content — strip scripts server-side, or serve uploads from a sandboxed origin with `Content-Disposition: attachment` / restrictive CSP so they never render inline.


### 2026-09-21 — Stored XSS via `javascript:` URI in hyperlink embedding feature (NASA VDP) — n/a
- Source: [Bugcrowd disclosure 4e648494](https://bugcrowd.com/disclosures/4e648494-aad1-4a50-a959-a4484776e502/stored-cross-site-scripting-xss-in-hyperlink-embedding-feature)
- Type: XSS (stored, javascript: URI)
- Summary: The link-embedding feature built an anchor as `<a href="{{link_here}}">` but did not block the `javascript:` scheme, so a payload of `javascript:prompt(document.domain);` executed when the stored link was clicked.
- Technique / pattern: Enter `javascript:prompt(document.domain);` in the link field; the value is stored and reflected into the `href` unsanitized, running script on click.
- Takeaway: Sanitizing HTML tags is not enough — URL attributes must be scheme-allowlisted (http/https/mailto only), or `javascript:`/`data:` URIs turn a link field into stored XSS.

### 2026-09-21 — Stored XSS via post title on Autodesk Forums enables low-priv to admin exploitation (Autodesk) — n/a
- Source: [HackerOne #2974307](https://hackerone.com/reports/2974307)
- Type: XSS (stored)
- Summary: The forum post-title field on forums.autodesk.com stored unescaped JavaScript that executed when the post was viewed, including by privileged users, allowing actions in their context.
- Technique / pattern: Inject a script payload into the post title; it is stored and executes in every viewer's browser, so a non-privileged author's payload runs with a viewing admin/moderator's session.
- Takeaway: Title/name fields are frequently trusted more than body fields — stored XSS in a widely-rendered field becomes privilege escalation when higher-privileged users view the content.

### 2026-09-21 — Double stored XSS in Federalist admin panel via Custom Domain field (GSA Bounty) — n/a
- Source: [HackerOne #245172](https://hackerone.com/reports/245172)
- Type: XSS (stored, pseudo-protocol)
- Summary: The Custom Domain field on the Federalist admin settings page stored javascript: pseudo-protocol payloads that fired on both the settings view and the published-sites page when an admin interacted with interface controls.
- Technique / pattern: Enter javascript:alert(document.domain) as a domain value; it is stored and later executed via clickable elements, hitting other admins on two separate pages.
- Takeaway: Fields that expect URLs must reject javascript:/data: schemes — validating 'is it a domain?' is not the same as blocking pseudo-protocol payloads that later become href/click sinks.


### 2026-09-21 — Stored XSS in file upload leads to privilege escalation and full workspace takeover (Dust) — n/a
- Source: [HackerOne #3115705](https://hackerone.com/reports/3115705)
- Type: Stored XSS via file upload (content-type confusion)
- Summary: An attacker uploaded a malicious HTML file disguised with an image extension into a Dust conversation; when an authenticated member (especially an admin) opened it, the embedded JavaScript ran in their session, enabling self-promotion to admin and full workspace compromise. Disclosed 2025-05-02.
- Technique / pattern: Upload a polyglot/mislabeled file, then confirm the server serves attacker-controlled HTML inline from a trusted origin so the browser renders and executes it rather than treating it as an inert image.
- Takeaway: Never serve user-uploaded files inline from a trusted origin without enforcing Content-Type, Content-Disposition: attachment, and a restrictive CSP; extension checks alone do not stop HTML/JS execution.

### 2026-09-21 — Second-Order XSS via javascript protocol in MCP Server Portal Apps leads to ATO (Cloudflare) — n/a
- Source: [HackerOne #3316910](https://hackerone.com/reports/3316910)
- Type: XSS - second-order / stored via OAuth redirect_uri (javascript: scheme)
- Summary: Cloudflare's MCP server portals did not sanitize OAuth redirect_uri; an attacker could register a client with a javascript: redirect_uri and then send an authenticated victim to /authorize, where the script executed in the victim's session, potentially leading to account takeover.
- Technique / pattern: Two stages: store the payload through OAuth dynamic client registration (returns a client_id), then trigger it through the authorize flow in the victim's logged-in context. Fixed by upgrading workers-oauth-provider.
- Takeaway: Any OAuth/OIDC server with dynamic client registration (common with MCP) must restrict redirect_uri to http(s) allow-listed values at registration AND at redirect time.

### 2026-09-21 — Stored XSS on TikTok's backend leads to the leakage of highly sensitive administrator data (TikTok) — n/a
- Source: [HackerOne #3037447](https://hackerone.com/reports/3037447)
- Type: XSS - blind stored XSS in internal back-office tooling
- Summary: A payload submitted through a public partner application/contact form executed when TikTok employees viewed the submissions in an internal analytics tool, exposing session tokens, admin JWTs, PII and internal paths.
- Technique / pattern: Blind XSS: plant callback payloads in public-facing forms whose data is later rendered in staff-only dashboards, and wait for the out-of-band callback to reveal where and in whose context it fired.
- Takeaway: Data from public forms often lands in internal tools that assume trusted input - output-encode everywhere, including admin/analytics UIs, and treat blind XSS as high impact.

### 2026-09-20 — Stored XSS via Kroki diagram blocks in GitLab markdown (GitLab) — $13,950
- Source: [HackerOne #1731349](https://hackerone.com/reports/1731349)
- Type: XSS (stored; selector/attribute-read mismatch in a markdown renderer)
- Summary: With Kroki enabled, GitLab selected diagram nodes with `pre[lang="<type>"] > code` **or** `pre > code[lang="<type>"]`, but then resolved the diagram type from `node.parent['lang']` with a fallback. Because the node that matched and the node whose attribute was read could differ, a crafted `pre` block injected arbitrary attributes into the generated `img` tag, giving stored XSS to everyone who viewed the page.
- Technique / pattern: Look for *disagreement between two passes over the same tree*. Renderers commonly select nodes with one predicate and then read attributes with another (parent vs child, first-match vs last-match, with vs without a fallback); wherever the two can point at different nodes, the validated value and the used value are not the same value. Build inputs that satisfy one arm of an `or` while carrying the payload on the other arm, and treat any attribute that flows into generated HTML — not just text content — as a sink.
- Takeaway: Select and read from the same node, and encode attributes at construction. An `or` in a selector paired with a fallback in the lookup is a bypass waiting to be found.

### 2026-09-20 — Client-side template injection leading to XSS (Mars) — n/a
- Source: [HackerOne #2234564](https://hackerone.com/reports/2234564)
- Type: XSS (client-side template injection in a JS template framework)
- Summary: User-supplied input was embedded into a page rendered by a client-side template framework, so a template expression provided by the attacker was evaluated by the framework during rendering and executed as script in the victim's browser.
- Technique / pattern: CSTI fires where classic XSS filters do not look, because the payload carries no angle brackets. Probe every reflected parameter with the framework's own arithmetic markers (`{{7*7}}`, `${7*7}`, `<%= 7*7 %>`) and watch for `49` in the rendered DOM; the interpolation delimiter identifies the framework, and from there the known sandbox-escape chain for that version gives script execution. Fingerprint the framework from the bundle first so the marker matches its syntax.
- Takeaway: A client-side template is an interpreter in the browser. Bind user data to the template rather than concatenating it into the template source, and keep frameworks past their sandbox-escape-prone versions.

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
- Summary: HTML injection surviving in GitLab's syntax_highlight_filter markdown pipeline let an attacker inject a `<base>` tag into notes, wiki pages and issue descriptions; that repointed relative script URLs at an attacker-controlled host, defeating the nonce-based CSP and executing script for every viewer.
- Technique / pattern: When a previous fix removed a particular XSS sink but left the underlying HTML injection, look for tags that change document semantics rather than tags that execute directly - `<base>` rewrites the resolution of every relative URL on the page. Then enumerate which script files the page expects to load and host matching paths on the attacker origin: those requests inherit the page's nonce, so the CSP is satisfied by construction.
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
