---
tags: [hacking, bug-bounty, info-disclosure, api, secrets, reports]
type: log
---

# Exposed APIs & Info Disclosure

## Up
- [[Reports]]

Disclosed **exposed API / sensitive information disclosure** reports — leaked API keys/secrets, PII returned by APIs, exposed config/debug endpoints, open `.git`/backups. See [[Pattern]] → Exposed APIs & Info Disclosure.

## Reports

### 2026-09-24 — One-click data exfiltration via the rovoChatPrompt URL parameter in Confluence Rovo (Atlassian) — $6,000 (P2)
- Source: [Bugcrowd bf1922fb](https://bugcrowd.com/disclosures/bf1922fb-99d0-4d3b-b419-1728720d29ec/one-click-data-exfiltration-via-rovochatprompt-url-parameter-confluence-rovo)
- Type: Prompt injection via a URL parameter — sensitive information disclosure through an AI assistant
- Summary: A link carrying attacker-written instructions in the `rovoChatPrompt` parameter was executed by Rovo as though the victim had typed it. The prompt had the assistant gather data the victim could access, then fetch an "image" from an attacker-controlled host with that data embedded in the URL path — exfiltrating Confluence pages, internal documents and API keys on a single click.
- Technique / pattern: Any parameter that pre-fills an AI assistant's input is an injection sink, and the assistant runs with the victim's privileges. Outbound rendering — images, link previews, webhooks — is the exfiltration channel. Guardrails were bypassed by indirection ("help me identify the bird from the image") rather than by stating the goal outright.
- Takeaway: Treat a URL-supplied prompt as untrusted input that must never auto-execute, and constrain what an assistant may fetch outbound; a model that can both read private data and request an arbitrary URL is a complete exfiltration primitive.

### 2026-09-24 — Hidden VPN help page exposed by tampering with a Pulse Connect Secure URL parameter (Department of the Interior VDP) — n/a (P5)
- Source: [Bugcrowd a4e04494](https://bugcrowd.com/disclosures/a4e04494-3100-4c52-98b7-4f7a7e087266/information-disclosure-via-url-tampering)
- Type: Information disclosure — an undocumented view reachable by changing a page-selector parameter
- Summary: Changing `p=no_cert` to `p=help` on a Pulse Connect Secure portal's `welcome.cgi` endpoint rendered a built-in help page disclosing the organisation's username format, links to other VPN endpoints and a support contact address.
- Technique / pattern: Parameters that select which view to render are enumerable — collect the values a product's documentation or source defines and try each one. The payoff is rarely a session; it is the reconnaissance that makes password spraying and social engineering work, since a known username format converts an email list into a credential list.
- Takeaway: Vendor appliances ship views the operator never chose to publish; audit what each parameter value renders after deployment, and treat username format as sensitive rather than cosmetic.

### 2026-09-24 — Jira admin API token hardcoded in a script shared in a public Slack channel (Mozilla) — n/a
- Source: [HackerOne #2467999](https://hackerone.com/reports/2467999)
- Type: Credential disclosure — secret leaked through a chat attachment
- Summary: A script posted in a publicly joinable Slack channel contained a hardcoded Jira admin API token that granted administrative access to Mozilla's Jira. The key was revoked and the script deleted from the channel.
- Technique / pattern: Secrets leak wherever code is shared informally, not only in repositories — public chat channels, issue attachments, support tickets, CI logs and pastebins. For programs whose scope permits it, searching public community spaces for uploaded scripts and configuration snippets is as productive as scanning git history.
- Takeaway: Anywhere that accepts file uploads is a secret store nobody is scanning; keep tokens out of scripts entirely, and point automated secret detection at chat uploads rather than only at commits.

### 2026-09-24 — Publicly accessible WordPress debug.log discloses server paths and plugin detail (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #3318295](https://hackerone.com/reports/3318295)
- Type: Information disclosure — framework debug log served over HTTP
- Summary: A WordPress installation left its `debug.log` readable under the content directory; the PHP warnings and deprecation notices inside disclosed absolute server paths along with the names and versions of installed plugins.
- Technique / pattern: Request the framework's conventional log location directly (`wp-content/debug.log` and its siblings) rather than waiting for a linked index. The content is the value: plugin names and versions map straight onto known CVEs, and absolute paths supply the prefix that local file inclusion and log-poisoning chains need.
- Takeaway: A debug log is an unauthenticated inventory of the stack; disable `WP_DEBUG_LOG` in production, and where logging must stay on, write it outside the web root and block the path at the server.

### 2026-09-24 — Exposed credentials in a public .env file on a NASA git repository (NASA VDP) — n/a (P3)
- Source: [Bugcrowd a9ba7ee3](https://bugcrowd.com/disclosures/a9ba7ee3-bb0e-4945-9834-34171056a680/exposed-credentials-in-public-env-file-on-nasa-git-repository)
- Type: Sensitive information disclosure — secrets committed to a public repository
- Summary: A `.env` file containing credentials was publicly readable on a NASA git repository. The researcher found it through search-engine dorking, with no interaction with a live application at all.
- Technique / pattern: Dorking is still the cheapest first pass over a large scope, because the index has already done the crawling and therefore surfaces files nothing links to any more. Combine a `site:` restriction with the artifact name and a distinctive key — a `site:`/`ext:env` pair, or a search for `DB_PASSWORD`, `SECRET_KEY` or `AWS_ACCESS_KEY_ID` alongside the organization's domains and repository hosts. Report the exposure and the need for rotation; do not authenticate with a discovered credential, since possession is the finding and use is generally out of scope.
- Takeaway: `.env`, `config.json`, `settings.py` and backup files keep leaking through search indexes long after the link is gone. The remediation is rotation, not deletion — a committed secret survives in git history and in caches.

### 2026-09-24 — Unauthenticated restricted file read and out-of-tree file disclosure via GetVectorFile.php (NASA VDP) — n/a (P1)
- Source: [Bugcrowd 72e039d9](https://bugcrowd.com/disclosures/72e039d9-7e6c-4548-b1c3-4e278e573c6e/unauthenticated-restricted-file-read-and-out-of-tree-file-disclosure-via-getvectorfile-php)
- Type: Information disclosure — arbitrary file read through a legacy download handler
- Summary: `GetVectorFile.php` served files from outside the intended application scope to unauthenticated requesters, including protected application files and host operating-system information. It was validated with non-destructive proof-of-concept requests and resolved by the program.
- Technique / pattern: Legacy server-rendered file handlers (`GetVectorFile.php`, `download.jsp`, `GetFile.ashx`, `Download.aspx`) are wired straight to the filesystem and sit outside whatever middleware guards the modern `/api/*` routes. Find them by grepping HTML and JavaScript for handler paths rather than crawling the JSON API, then test the filename parameter for traversal sequences and for absolute paths. Prove impact with an application config file — which usually holds credentials — plus one harmless OS file, and stop there.
- Takeaway: File-serving endpoints are an object store with no authorization layer in front of them. Inventory them separately from the API, and rate the finding by what the readable files contain: an app-config read is normally a credential compromise, not a disclosure nit.

### 2026-09-24 — Unauthenticated account registration + email verification bypass + PII leak of 2,480 records (Essity) — n/a (Medium)
- Source: [HackerOne #3737516](https://hackerone.com/reports/3737516)
- Type: Exposed API / information disclosure through an authentication-gate bypass chain
- Summary: A portal intended only for authorized employees and partners exposed a public registration API. An unauthenticated attacker could register an account, bypass the email-verification gate by running the password-reset flow against the still-pending account, log in as an "active" user, and then reach protected API endpoints returning 2,480 employee records (names, emails, profile URLs) — defeating both the SAML SSO requirement and the verification gate.
- Technique / pattern: Three links, each trivial on its own. The CSRF token needed to register was served from an unauthenticated endpoint (`/actions/blitz/csrf/json`), so that "protection" was self-service. The password-reset flow set the account active as a side effect — reset and verification are two routes to the same state flag, and hardening one rarely hardens the other. Once any session existed, the data APIs authorized on "is logged in" rather than on role or SSO provenance. Run the reset flow against accounts in every lifecycle state: pending, unverified, disabled, deleted.
- Takeaway: SSO in front of a portal means nothing while a legacy local-registration path still exists behind it. Enumerate every way an account can reach the "active" state, and check whether the data APIs distinguish an SSO-provisioned identity from any authenticated session at all.

### 2026-09-24 — Unauthenticated API allows reading, writing to and deleting any user's private chat history (Essity) — n/a (Critical)
- Source: [HackerOne #4020767](https://hackerone.com/reports/4020767)
- Type: Exposed API — authentication enforced on one route, absent on every sibling
- Summary: An internal Microsoft Teams chatbot backed by a retrieval-augmented knowledge base and Azure OpenAI also exposed an HTTP API with no authentication on any of its data routes. The Bot Framework webhook `/api/messages` correctly returned `401 Missing Authorization header from Bot Service`, but `/api/sessions` (`GET`), `/api/sessions/<session_id>` (`DELETE`), `/api/sessions/<session_id>/history` (`GET`), `/api/chat` (`POST`), `/api/feedback` and `/api/health` were all open — letting an unauthenticated attacker on the internet list every conversation, read any conversation's full message history, create conversations and drive the LLM backend, and delete any conversation. The same API was exposed on the development instance.
- Technique / pattern: Enumerate the whole route table, not the route the product is documented around. The entire API surface here was seven routes and exactly one was protected — a shape that recurs whenever authentication arrives with a framework integration (a bot webhook, an OAuth callback, a payment webhook) instead of being applied as middleware across the app. Note too that the only control on `/api/chat` was an Azure Application Gateway WAF rule blocking JSON bodies containing the key `session_id`, and a WAF rule is not authentication.
- Takeaway: Where one route's auth is supplied by an SDK rather than by the application, assume the neighbouring routes have none. Then look for the sibling development or staging host: it almost always shares the deployment and rarely gains the fix first.

### 2026-09-23 — Directory listing on an unauthenticated extranet path leaks source and internal docs (NASA (Bugcrowd)) — n/a (P3)
- Source: [Bugcrowd ff4614f2](https://bugcrowd.com/disclosures/ff4614f2-3260-4653-ab10-16fccbb6bb58/login-bypass-leads-to-internal-information-disclosure)
- Type: Information disclosure / broken access control + directory listing
- Summary: An extranet path intended to be restricted was reachable without authentication and had directory listing enabled, exposing proprietary source code and internal collaboration documents to anyone who requested the path.
- Technique / pattern: The pattern is a supposedly private path served with autoindex on and no auth gate; content discovery against the directory reveals the enumerable file tree and its downloadable contents.
- Takeaway: "Private" network zones and extranet paths still need enforced authentication and directory listing disabled. An unauthenticated, browsable directory turns a broken access-control issue into bulk source and document exposure.

### 2026-09-23 — Exposed `.old` backup file discloses internal structure (NASA (Bugcrowd)) — n/a (P5, informational)
- Source: [Bugcrowd 0c52ff12](https://bugcrowd.com/disclosures/0c52ff12-28ba-4257-95d6-eae2ba2ec97d/sensitive-information-disclosure)
- Type: Information disclosure / exposed backup artifact
- Summary: A NASA SOHO subdomain served a leftover `index.html.old` backup file at a predictable path, revealing internal page structure and directory references useful for reconnaissance (no credentials or PII were exposed).
- Technique / pattern: Detected by requesting common backup/temporary suffixes (`.old`, `.bak`, `.swp`, `~`) alongside known page paths — a routine content-discovery check for development artifacts left in production.
- Takeaway: Backup and editor-temporary files are a persistent low-effort information leak. Add backup-suffix probing to content discovery, and keep such artifacts out of web-served directories.

### 2026-09-23 — Internal report attachments retrievable through the "Export as .zip" feature (HackerOne) — $12,500
- Source: [HackerOne #186230](https://hackerone.com/reports/186230)
- Type: Information disclosure / access-control gap in export
- Summary: A newly shipped "Export as .zip" feature bundled attachments that should have stayed internal, and an attachment removed from a disclosed report's thread still appeared inside the exported archive. Root-cause work also flagged that inline attachments could be reached by guessing attachment identifiers.
- Technique / pattern: The pattern is an export/download path re-deriving its contents without re-applying the visibility rules of the primary view. Comparing what the public report shows against what its export contains reveals the gap.
- Takeaway: Export, print and download features must reapply the same authorization and redaction as the primary view, and a removed item must be gone from every representation. Re-test data-visibility bugs against secondary output formats.

### 2026-09-23 — Report metadata inferable via timing side channel on JSON endpoints (HackerOne) — n/a
- Source: [HackerOne #350432](https://hackerone.com/reports/350432)
- Type: Information disclosure / timing side channel
- Summary: Chaining incremental report IDs, distinguishable HTTP status codes for authenticated vs unauthenticated requests, and response-timing measurement against unauthenticated JSON endpoints created a theoretical path to infer counts of reports matching given criteria. HackerOne judged it not reliably exploitable and later moved the endpoints behind authenticated GraphQL.
- Technique / pattern: The building blocks are generic: sequential identifiers leak volume between two known IDs, differing status/error responses reveal auth state, and browser timing APIs measure response size differences on endpoints that lack authentication.
- Takeaway: Predictable sequential IDs plus unauthenticated JSON endpoints leak aggregate information even when individual records are protected. Prefer non-sequential identifiers and require authentication on data endpoints.

### 2026-09-23 — Information Disclosure or 403 Bypass via an archived copy of a protected directory (NASA VDP) — n/a (P3)
- Source: [Bugcrowd #b781d060](https://bugcrowd.com/disclosures/b781d060-ee5c-4ef6-963f-b174d089c8ba/information-disclosure-or-403-bypass)
- Type: Access control bypass / information disclosure
- Summary: Direct requests to an endpoint returned `403 Forbidden`, but an archive of the same directory was downloadable without restriction, so extracting it yielded the protected contents. Disclosed 2025-01-21; resolved by removing the archive.
- Technique / pattern: When a path returns `403`, look for a parallel representation of the same data — a `.zip` or `.tar.gz` sibling, a backup copy, a cached or mirrored version — since access control is usually applied per route rather than per resource.
- Takeaway: A `403` protects one route, not the underlying data; always check whether the same content is reachable through an archive, export or alternate path before writing the endpoint off.

### 2026-09-23 — Exposed .svn Metadata Leads to Information Disclosure and Unauthenticated File Access (NASA VDP) — n/a (P2)
- Source: [Bugcrowd #2b53813a](https://bugcrowd.com/disclosures/2b53813a-7264-46ed-9df9-ae2215c8353d/exposed-svn-metadata-leads-to-information-disclosure-and-unauthenticated-file-access)
- Type: Exposed version-control metadata / unauthenticated file access
- Summary: A Subversion metadata directory was served publicly on a NASA portal; the `.svn/entries` file disclosed repository paths, the full file structure, timestamps and MD5 checksums, and the referenced files could then be fetched without authentication. Disclosed 2025-10-31.
- Technique / pattern: Request version-control metadata paths such as `.svn/entries`, `.git/config` and `.git/HEAD` on every host, then use the recovered file list as a map for retrieving individual files directly.
- Takeaway: Leftover VCS metadata turns a blind directory into an index — the finding is not the metadata file itself but the unauthenticated reads it makes possible.

### 2026-09-23 — IDOR in Report CSV export discloses IDs of Custom Field Attributes of Programs (HackerOne) — n/a
- Source: [HackerOne #510759](https://hackerone.com/reports/510759)
- Type: Exposed API / information disclosure via bulk export
- Summary: The `POST /reports/export` endpoint accepted arbitrary `report_ids[]` values without checking team ownership, and the generated CSV header leaked the custom-field attribute ids belonging to other teams.
- Technique / pattern: Point export, report-generation and bulk-download endpoints at identifiers you do not own, and read the metadata of the result — headers, column names and filenames — not just the rows, which may legitimately come back empty.
- Takeaway: Export endpoints are a frequently unguarded parallel path to data the UI protects, and generated headers can disclose internal identifiers even when the body does not.

### 2026-09-23 — User object in GraphQL exposes private program information (HackerOne) — n/a
- Source: [HackerOne #350964](https://hackerone.com/reports/350964)
- Type: Exposed API / GraphQL information disclosure
- Summary: A GraphQL query reachable by anyone holding admin or member access to a single sandbox team revealed whether external programs existed in the directory, including programs configured as private.
- Technique / pattern: From the lowest-privilege account that can reach the schema, walk related objects outward from the user node and look for fields that answer existence questions; leaking "this program exists" is a disclosure even when no attributes are returned.
- Takeaway: GraphQL's object graph lets a low-privilege node reach high-privilege neighbours — authorization has to live on each field and edge, and existence itself is sometimes the confidential fact.

### 2026-09-22 — Unauthenticated disclosure of organizer email addresses via The Events Calendar REST API, CVE-2025-9808 (NASA VDP) — n/a (P5)
- Source: [Bugcrowd #08b14f25](https://bugcrowd.com/disclosures/08b14f25-72b5-4d36-83a3-5aeb1e3f11cc/unauthenticated-disclosure-of-nasa-organizer-email-addresses-via-the-events-calendar-rest-api-cve-2025-9808)
- Type: Over-exposed plugin REST endpoint (information disclosure)
- Summary: The Events Calendar WordPress plugin (through 6.15.2) served organizer records — including email addresses and contact details — from its REST routes without authentication on a NASA subdomain, giving a ready-made list for targeted phishing.
- Technique / pattern: Fingerprint the CMS and its plugins, then query the framework's own API namespace (`/wp-json/...`) for each plugin's collections; plugin authors frequently expose "public" post types whose meta fields carry contact data the site owner never intended to publish. Check the plugin version against known CVEs before writing it up.
- Takeaway: A CMS plugin extends your public API surface — inventory the REST routes each one registers and confirm which fields they serialise to anonymous callers.

### 2026-09-22 — Unauthenticated access to MMGIS webhooks (NASA VDP) — n/a (P2)
- Source: [Bugcrowd #c20138e9](https://bugcrowd.com/disclosures/c20138e9-28af-423f-a92b-0a2a1f85b3b2/unauthenticated-access-to-mmgis-webhooks)
- Type: Admin API missing authentication
- Summary: The webhook create/list/update endpoints of MMGIS, intended for administrators, answered requests carrying no token, cookie or auth header, so anyone could read webhook configuration and point or alter webhooks — enabling data exfiltration or tampering with downstream automation.
- Technique / pattern: Strip all credentials and replay every administrative route you can enumerate (from the project's public source, docs or JS bundle) — integration-management routes such as webhooks, API keys and OAuth apps are frequently added after the auth middleware was written and miss it. Open-source products make this a source-reading exercise rather than guesswork.
- Takeaway: Webhook configuration is credential-grade: it names where your data goes. Authentication and authorization belong on every route by default, enforced centrally rather than per-handler.

### 2026-09-22 — CVE-2026-3783: bearer token leaked on cross-host redirect when netrc is used (curl) — n/a
- Source: [HackerOne #3583983](https://hackerone.com/reports/3583983)
- Type: Credential leakage across a trust boundary (information exposure through sent data)
- Summary: With `--oauth2-bearer` combined with `--netrc` (a `default` entry), curl's netrc path skipped the host check that normally strips credentials on redirect, so following a redirect (`-L`) to another host sent the OAuth2 bearer token to that host in `lib/http.c`.
- Technique / pattern: For client-side credential handling, map every source of authentication (flag, config file, environment, keychain) and test each one against the redirect path — the guard is usually implemented once, on the most obvious source, and a second source bypasses it. A wildcard `default` entry in a credentials file is the classic amplifier.
- Takeaway: Credential scoping must be enforced at the point the header is written, not where the credential was configured; when reviewing HTTP clients, redirects are where secrets escape their intended host.

### 2026-09-22 — Email address of any user disclosed by the Report Invitation GraphQL type (HackerOne) — $1,000 bonus
- Source: [HackerOne #792927](https://hackerone.com/reports/792927)
- Type: GraphQL authorization gap returning PII in a mutation payload
- Summary: Calling the `addReportParticipant` mutation with a known username returned an invitation object whose `email` field held that user's real address, so any user's email could be resolved from their username. The cause was that REST-era ACLs were not carried over when the invitation object was migrated to GraphQL.
- Technique / pattern: Do not only introspect queries — enumerate *mutations* and inspect what their return payloads serialise; the object echoed back after a write is a read surface that often escapes the authorization annotations applied to query resolvers. Migrated objects are the highest-yield place to look.
- Takeaway: Every resolver, including the payload types of mutations, needs its own field-level authorization; architecture migrations silently drop the checks the old layer enforced.
### 2026-09-22 — GraphQL query "namespace" leaks data (GitLab) — bounty paid (amount not shown)
- Source: [HackerOne #614355](https://hackerone.com/reports/614355)
- Type: Over-exposed GraphQL resolver / privacy-setting bypass (unauthenticated)
- Summary: GitLab's GraphQL `namespace(fullPath:)` query returned details of private user profiles (including their project lists) and secret groups/subgroups — descriptions, visibility and project metadata — to unauthenticated callers, while the REST API correctly returned 404 for the same namespaces.
- Technique / pattern: Sent an unauthenticated POST to `/api/graphql` with a `namespace(fullPath: ...)` query naming a user with "private profile" enabled or a secret group, and compared the response with the equivalent REST call; the GraphQL resolver did not apply the visibility check the REST layer did.
- Takeaway: Treat GraphQL and REST as two separate implementations of the same access rules — for every privacy toggle or 404-protected object, ask the GraphQL API the same question unauthenticated and diff the answers.

### 2026-09-22 — Publicly accessible endpoint exposing internal user identifiers and emails (Mars) — n/a
- Source: [HackerOne #3360293](https://hackerone.com/reports/3360293)
- Type: Unauthenticated JSON API exposing PII (CWE-200)
- Summary: An unauthenticated JSON endpoint on a Mars subdomain returned internal user UUIDs, names, corporate emails and admin role details; fixed by adding auth and trimming fields.
- Technique / pattern: Found during wildcard-scope recon and requested with no session, returning bulk user records usable for enumeration and targeting admins.
- Takeaway: Request every discovered API endpoint without credentials; user directories and role data should never be public.

### 2026-09-22 — Exposed Zotero API key allows unauthorized write access to group library (NASA VDP) — n/a
- Source: [Bugcrowd #36c73c74](https://bugcrowd.com/disclosures/36c73c74-7822-4407-8172-73229b00242b/exposed-zotero-api-key-allows-unauthorized-write-access-to-group-library)
- Type: Exposed third-party credential (P3)
- Summary: A Zotero API key committed to a public file in the GES-DISC Reference Management System had read/write scope on a Zotero group library.
- Technique / pattern: Located the key in public source and confirmed (non-destructively) it was live and write-capable against the Zotero API.
- Takeaway: Assess leaked keys by scope: a write-capable third-party key enables tampering with data the org relies on; use least-privilege keys and secret scanning.

### 2026-09-22 — Hardcoded Scopus API key in public NASA GitHub repository (NASA VDP) — n/a
- Source: [Bugcrowd #9355808f](https://bugcrowd.com/disclosures/9355808f-9f75-4631-95fb-b590c0219004/hardcoded-api-key-found-in-public-nasa-github-repository)
- Type: Hardcoded third-party API key (P3)
- Summary: A hardcoded Elsevier Scopus API key in the public repo `podaac_tools_and_services` allowed anyone to access licensed research data under NASA's license.
- Technique / pattern: Reviewed the organisation's public repositories for embedded secrets and assessed the key by the service and scope it granted.
- Takeaway: Leaked keys to paid/licensed third-party data services are impactful (licence abuse); scan org repos and git history, revoke and rotate committed keys.

### 2026-09-21 — Public admin config file exposes DES password hashes and internal config (NASA VDP)
- Source: [Bugcrowd #e2794469](https://bugcrowd.com/disclosures/e2794469-0dec-4c81-a5ca-916a663f35c5/publicly-accessible-administrative-configuration-file-exposes-authentication-hashes-and-internal-configuration)
- Type: Sensitive information disclosure (exposed config file)
- Summary: An administrative configuration file was reachable without authentication, exposing server paths, CGI endpoints, and Unix DES-style password hashes suitable for offline cracking.
- Technique / pattern: Request the unprotected admin config file directly; parse it for internal endpoints and credential hashes, which were then recovered offline during testing.
- Takeaway: Config/admin files must never be web-servable — enforce authZ and block known config paths; DES-crypt hashes are trivially crackable, so exposure is effectively credential disclosure.

### 2026-09-21 — Unauthenticated WordPress REST endpoint leaks unpublished/embargoed content (NASA VDP)
- Source: [Bugcrowd #c25fa845](https://bugcrowd.com/disclosures/c25fa845-a833-4df7-b706-ece0eab5bda0/unauthenticated-disclosure-of-unpublished-embargoed)
- Type: Exposed API / sensitive information disclosure
- Summary: A custom WordPress REST route, `/wp-json/nasa-external-content/v1/unpublished-posts`, served without authentication, returned 3,939 unpublished/non-public IDs across 27 content types.
- Technique / pattern: Send one unauthenticated GET to the custom `wp-json` route; it dumps draft IDs, content types, and modification timestamps — revealing the editorial pipeline and timing of upcoming announcements.
- Takeaway: Custom REST routes need explicit `permission_callback` gating — the default is public, and 'unpublished' content is exactly what must not be enumerable unauthenticated.

### 2026-09-21 — MQTT wildcard subscription leaks all pinboard UUIDs (Opera) — $800
- Source: [Bugcrowd #585d9b4e](https://bugcrowd.com/disclosures/585d9b4e-c301-4b68-8e55-1e21139d97bc/unauthenticated-mqtt-wildcard-board-leaks-all-pinboard-uuids-to-unauthorized-users)
- Type: Exposed API / broken access control (AWS IoT / Cognito)
- Summary: A public Cognito Identity Pool handed out temporary AWS credentials that allowed subscribing to a wildcard MQTT topic on AWS IoT Core, passively harvesting every pinboard UUID as users interacted — and the UUID was the sole access token.
- Technique / pattern: Pull temp creds from the exposed Cognito Identity Pool, subscribe to the wildcard topic `board/#`, and collect board UUIDs as they flow by; then open each board directly since the UUID is the only gate.
- Takeaway: Wildcard topic subscriptions plus unauthenticated identity pools break tenant isolation — scope IoT policies to exact per-user topics and never treat a leaked UUID as authorization.

### 2026-09-21 — Single-request GraphQL DoS via circular introspection query (Sorare)
- Source: [HackerOne #2048725](https://hackerone.com/reports/2048725)
- Type: Exposed API / GraphQL (no query depth limit)
- Summary: `api.sorare.com/graphql` enforced no query depth limit, so one recursive `__schema` introspection query generated >3.7MB of duplicated data and 5-7s backend delays — an unauthenticated single-request DoS.
- Technique / pattern: Send a POST to `/graphql` nesting `__schema.types.fields.type.fields` recursively; each layer multiplies response size and time, so a single request degrades the backend without DDoS-scale traffic.
- Takeaway: Disable introspection in production and enforce query depth/complexity and cost limits — an unbounded schema graph turns one request into a denial of service.

### 2026-09-21 — Hardcoded API keys / tokens in Android APK (Zenly) — $750
- Source: [HackerOne #753868](https://hackerone.com/reports/753868)
- Type: Sensitive information disclosure (secrets in mobile app)
- Summary: The Android app shipped hardcoded, overly permissive API keys and auth tokens in cleartext, extractable by decompiling the APK and usable to call backend APIs as the app.
- Technique / pattern: Decompile the APK with standard tooling, grep the smali/resources for key/token formats, and extract credentials that grant unauthorized backend access.
- Takeaway: Never embed backend secrets in a mobile client — anything shipped in an APK is public; use short-lived server-issued tokens and scope keys tightly.

### 2026-09-21 — PII disclosure — removed team members' personal emails viewable by unprivileged staff (Shopify) — $500
- Source: [HackerOne #415622](https://hackerone.com/reports/415622)
- Type: Sensitive information disclosure (broken function-level authZ)
- Summary: On the Partner Dashboard, a staff member with no permissions could open `/{PartnerTeam_ID}/memberships/removed` and see past members' names, personal emails, and removal dates.
- Technique / pattern: As an unprivileged invited staff user, browse directly to the removed-members endpoint; it renders PII with no permission check, and even reflects former members' later profile edits.
- Takeaway: Function-level authorization must cover every sub-page, including 'removed/archived' views — permissionless roles should see nothing sensitive, and retained ex-member PII widens the blast radius.


### 2026-09-21 — Restricted consent forms with PII exposed via search-engine indexing on globe.gov (NASA VDP) — n/a
- Source: [Bugcrowd disclosure 2ac5bb50](https://bugcrowd.com/disclosures/2ac5bb50-77f9-4249-9c2d-32c69d32c54d/unauthorized-access-to-piis-and-signatures-through-search-engine-indexing-of-restricted-documents)
- Type: Exposed APIs / Info Disclosure (indexed PII, P3)
- Summary: Restricted directories on globe.gov holding media-consent forms were indexed by search engines, so PII — names, addresses, emails and guardian/minor signatures — was publicly reachable without authentication despite direct access requiring a login.
- Technique / pattern: Use search-engine dorks to pull the indexed document URLs and open them directly; the auth control on the directory did not stop the crawler-cached links.
- Takeaway: Authentication on the directory is not enough if crawlers already indexed the files — add `robots.txt`/`X-Robots-Tag noindex`, remove cached results, and gate documents behind auth at the file level.

### 2026-09-21 — Legacy directory listing exposes a CV with full PII (NASA VDP) — n/a
- Source: [Bugcrowd disclosure 2560ef0c](https://bugcrowd.com/disclosures/2560ef0c-bc4d-47ae-882d-fd28d665d2aa/publicly-accessible-legacy-directory-exposes-curriculum-vitae-containing-personal-information-pii)
- Type: Exposed APIs / Info Disclosure (PII, P3)
- Summary: A legacy documentation subdomain had directory listing enabled, exposing a PDF curriculum vitae containing an individual's name, photo, home address, email and phone numbers.
- Technique / pattern: Enumerate the indexed legacy directory (not linked from normal navigation) and download the exposed PDF containing the PII.
- Takeaway: Old, forgotten subdomains with indexing on are a recurring PII leak — inventory legacy assets and disable directory listing everywhere, not just on the primary site.

### 2026-09-21 — Public `swagger.yaml` reveals internal MEDITOR API endpoints and auth flows (NASA VDP) — n/a
- Source: [Bugcrowd disclosure 316abfc6](https://bugcrowd.com/disclosures/316abfc6-33cc-41c6-b4f5-e1eabe3e2b5d/public-exposure-of-nasa-meditor-api-specification-revealing-internal-endpoints-and-authentication-mechanisms)
- Type: Exposed APIs / Info Disclosure (API spec, P3)
- Summary: A publicly reachable `swagger.yaml` for the MEDITOR API exposed internal endpoints, OAuth2 flows, CSRF-token retrieval and file-upload endpoints, aiding reconnaissance even though calls still needed credentials.
- Technique / pattern: Fetch the unauthenticated `swagger.yaml`/OpenAPI file and map the full internal API surface — hidden endpoints, parameters and auth mechanics — to plan targeted follow-on testing.
- Takeaway: OpenAPI/Swagger specs are a roadmap for attackers — keep them behind auth and block indexing; a leaked spec turns blind API testing into a guided one.

### 2026-09-21 — OAuth `client_id`/`client_secret` exposed in front-end JavaScript (NASA VDP) — n/a
- Source: [Bugcrowd disclosure 60690beb](https://bugcrowd.com/disclosures/60690beb-4006-43df-b3e8-296492718793/source-map-disclosure-in-public-api-endpoint-on-globe-gov)
- Type: Exposed APIs / Info Disclosure (secrets in client, P5)
- Summary: On globe.gov the OAuth token URL, `client_id` and `client_secret` were stored in JavaScript variables and readable via browser DevTools, with scopes fetched from the API and shown in the UI.
- Technique / pattern: Open DevTools / the JS bundle and read the OAuth `client_id` and `client_secret` directly; the confidential client is effectively public.
- Takeaway: A `client_secret` in browser JS is not a secret — public OAuth clients must use PKCE with no secret, and any credential shipped to the client should be treated as disclosed.

### 2026-09-21 — Exposed API endpoint leaks usernames and user keys without auth (NASA VDP) — n/a
- Source: [Bugcrowd disclosure a655d2f1](https://bugcrowd.com/disclosures/a655d2f1-67d3-4e26-a477-46ef86b5fb8b/exposed-api-endpoints-leading-to-disclosure-of-user-information)
- Type: Exposed API / Info Disclosure (P3)
- Summary: A public Confluence-style API at `https://rmakp.nasa.gov/rest/api/content/16121857` returned usernames, user keys and display names with no access control and no rate limiting.
- Technique / pattern: Request the `rest/api/content/<id>` endpoint unauthenticated and iterate content ids; the responses expose user identifiers usable for enumeration and credential-stuffing target lists.
- Takeaway: Product REST APIs (`/rest/api/...`) often ship open by default — require auth and rate-limit them, since exposed user keys enable social-engineering and brute-force at scale.

### 2026-09-21 — Discoverability restriction bypass enables user ID enumeration by phone/email (X (Twitter)) — $5,040
- Source: [HackerOne #1439026](https://hackerone.com/reports/1439026)
- Type: Info Disclosure (enumeration / privacy bypass)
- Summary: Submitting a phone number or email through the Android login flow returned the associated user id even when the target had disabled discoverability, allowing mass phone/email-to-account enumeration.
- Technique / pattern: Feed phone numbers or emails into the Android login/discovery endpoint; it resolves them to user ids regardless of the victim's privacy setting, enabling bulk identity-to-account databases.
- Takeaway: Privacy toggles must be enforced on every code path — an alternate client (mobile login) endpoint frequently ignores a restriction the main flow honors.

### 2026-09-21 — Unauthenticated bulk download of candidate resumes/CVs via Salesforce Aura (U.S. Dept of Defense) — n/a
- Source: [HackerOne #2623715](https://hackerone.com/reports/2623715)
- Type: Info Disclosure (Salesforce Aura / BAC)
- Summary: A Salesforce Experience Cloud registration page let an Aura API payload query ContentDocument records, yielding file ids that fed direct-download URLs for thousands of confidential resumes and transcripts.
- Technique / pattern: Send crafted Aura getItems/ContentDocument queries to enumerate file ids, then request each via the file-download URL; access control on the documents was missing.
- Takeaway: Salesforce Experience/Community sites expose Aura endpoints (ContentDocument, record queries) that routinely leak data — always probe /aura for object and file enumeration on Salesforce-backed portals.

### 2026-09-21 — PII of tour visitors leaked via misconfigured Salesforce record permissions (U.S. Dept of Defense) — n/a
- Source: [HackerOne #2294930](https://hackerone.com/reports/2294930)
- Type: Info Disclosure (Salesforce Aura / object permissions)
- Summary: Salesforce Aura endpoints served Contact/Account/AccountContactRelation records to any registered portal user, exposing hundreds of thousands of names, emails and phone numbers via sequential-id enumeration.
- Technique / pattern: Register as a standard user, capture an authenticated Aura POST, then modify parameters to enumerate sequential Salesforce record ids; object-level permissions failed to restrict the records returned.
- Takeaway: Object-/record-level (FLS + sharing) permissions are the real control on Salesforce — a valid portal login should not read arbitrary Contact/Account records via Aura, so test record enumeration explicitly.


### 2026-09-21 — Exposed .git directory on a test/canary API host (Kubernetes) — n/a
- Source: [HackerOne #970520](https://hackerone.com/reports/970520)
- Type: Exposed .git directory (source disclosure)
- Summary: A Kubernetes test/canary API host returned 403 for /.git itself but served subpaths such as .git/logs/HEAD and .git/config, letting an attacker recover source code and git history. Disclosed 2021-01-07; closed as duplicate, no bounty.
- Technique / pattern: When the directory root is forbidden, request known files inside .git directly - the step git-dumper-style tools automate to rebuild the repo.
- Takeaway: A 403 on a directory does not mean its files are blocked - deny the whole /.git/* pattern, keep VCS metadata out of web roots, and include test/canary hosts in scans.

### 2026-09-21 — GraphQL introspection enabled, leaks schema and user-enumeration operation (On) — n/a
- Source: [HackerOne #1132803](https://hackerone.com/reports/1132803)
- Type: GraphQL introspection enabled (schema disclosure)
- Summary: A production GraphQL endpoint answered introspection queries from anyone, exposing types, fields, queries and mutations including a userExists operation usable to check whether an email is registered. Disclosed 2021-05-09; resolved, no bounty.
- Technique / pattern: Send a standard __schema introspection query, map the returned operations, and look for sensitive or weakly protected ones such as user enumeration.
- Takeaway: Disable introspection in production or restrict it to authorized users, and - more importantly - enforce authorization on every resolver, since a leaked schema only speeds up finding unprotected operations.

### 2026-09-21 — GitHub API key for BrewTestBot publicly exposed in CI logs (Homebrew) — n/a
- Source: [HackerOne #388740](https://hackerone.com/reports/388740)
- Type: Leaked API token in exposed CI/build logs
- Summary: jenkins.brew.sh was publicly reachable and its build logs printed a HOMEBREW_GITHUB_API_TOKEN for the BrewTestBot user, which could authenticate to the homebrew-core repo. Disclosed 2018-08-11; no bounty.
- Technique / pattern: Browse the public CI server's build output, pull the secret from the printed environment variables, and prove it works with a harmless GitHub API call while making no destructive changes.
- Takeaway: CI dashboards and logs are high-value recon - mask secrets in build output, keep CI private, and scope bot tokens minimally (a repo deploy key over a broad personal token).

### 2026-09-21 — Public GitHub repos of managed triage team leak private report details (HackerOne) — n/a
- Source: [HackerOne #2937622](https://hackerone.com/reports/2937622)
- Type: Sensitive data in public source repositories
- Summary: Shared GitHub accounts run by HackerOne's managed triage team had 44 public repos whose exploit scripts, PoCs, commit history and Actions workflows revealed details of at least 9 private reports on H1-managed programs, some still active. Disclosed 2025-05-31, rated medium; bountied, amount not shown.
- Technique / pattern: OSINT on org-linked GitHub accounts, then a manual read of each repo's contents, commit history and CI workflow files to find leftover PoC code and its targets.
- Takeaway: Treat staff and shared service-account repos as attack surface - scratch PoCs, CI configs and git history leak confidential data, so audit repo visibility and default new repos to private.

### 2026-09-21 — Possible PII Disclosure via Advanced Vetting Process (HackerOne) — n/a
- Source: [HackerOne #2421796](https://hackerone.com/reports/2421796)
- Type: Excessive data exposure - unauthorized CSV export of PII
- Summary: A terms-acceptance export endpoint returned a CSV of researcher PII (names, usernames, addresses, countries, signature dates) to users not authorized for the relevant programs.
- Technique / pattern: Requested the terms_acceptance_data.csv export endpoint directly with a plain GET, bypassing the UI that normally gated it.
- Takeaway: Export/download endpoints (.csv/.json/.xlsx) often skip the authorization the UI enforces - request them directly from an unprivileged account.

### 2026-09-21 — API key (api.semrush.com) leak in JS-file (Semrush) — n/a
- Source: [HackerOne #1218754](https://hackerone.com/reports/1218754)
- Type: Leaked API token in JavaScript bundle
- Summary: A not-found page referenced JS files belonging to Semrush's internal interface; one publicly accessible file contained an internal API token that granted access to internal system statistics.
- Technique / pattern: Requested a nonexistent route, read the returned HTML for referenced script files, then downloaded those bundles and grepped them for tokens.
- Takeaway: Error and 404 pages can reference different (internal) bundles than the main app - collect JS from every page variant, not just the landing page.

### 2026-09-21 — PHP Info Exposing Secrets at https://radio.mtn.bj/info (MTN Group) — n/a
- Source: [HackerOne #1049402](https://hackerone.com/reports/1049402)
- Type: Verbose info page (phpinfo) leaking secrets
- Summary: A public /info phpinfo page exposed the Laravel APP_KEY, database credentials and SMTP authentication details; the researcher confirmed impact by sending a test email with the leaked SMTP credentials.
- Technique / pattern: Probed common diagnostic paths (/info, /phpinfo.php), parsed the environment variables shown, and validated the SMTP credential safely.
- Takeaway: Remove phpinfo/diagnostic pages from production and rotate any secrets they showed; a leaked Laravel APP_KEY can also enable cookie forgery/deserialization attacks.

### 2026-09-20 — Information disclosure via Logback configuration injection in the GoCD agent (GoCD) — n/a
- Source: [HackerOne #3509632](https://hackerone.com/reports/3509632)
- Type: Information disclosure (external logging config loaded in preference to built-in defaults)
- Summary: The GoCD agent's Logback setup supports property substitution and loads a custom configuration from a config directory that does not exist by default in the installation path. An attacker able to create that missing directory could place an `agent-launcher-logback.xml` there, and the agent would load it over its defaults, causing sensitive data to be written out through the logging framework.
- Technique / pattern: Treat "a config path the application searches but that ships empty" as an injection point. For any agent or daemon, read the documented configuration search order, list which of those paths exist on a default install, and check the permissions on their parent directories — a world-writable parent makes a *missing* directory attacker-creatable, which beats needing write access to an existing config. Logging frameworks are the highest-value target of this shape because their config languages support substitution, file appenders and remote appenders.
- Takeaway: Create and lock down every directory in a configuration search path at install time, and ignore configuration from locations the installer does not own.

### 2026-09-20 — Customer PII exposed in plaintext SQL statements written to ORDER_ERROR_LOG (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #3242830](https://hackerone.com/reports/3242830)
- Type: Information disclosure (error handler logs full database statements including PII)
- Summary: When a database write failed, the application logged the complete INSERT statement, so the `ORDER_ERROR_LOG` file held customers' personally identifiable information in plaintext, readable by anyone who could reach the log.
- Technique / pattern: Error logs are a second copy of the database with none of its access controls. Fuzz for log files by convention rather than by link — `*_ERROR_LOG`, `debug.log`, `error_log`, `app.log`, dated variants — at the web root and one directory up, and check the archive and directory-listing variants too. The higher-value question during triage is *what the handler writes*: a handler that logs the failing statement rather than the failing operation converts every validation error into a PII disclosure, so a single reachable log file can hold years of records.
- Takeaway: Log the operation and an identifier, never the statement or its bound values, and keep logs outside the document root with the data classification the underlying table carries.

### 2026-09-20 — Email address embedded in a confirmation-link token and preserved publicly by the Wayback Machine (Omise) — n/a
- Source: [HackerOne #3210022](https://hackerone.com/reports/3210022)
- Type: Information disclosure (PII encoded in a URL token, archived by a public crawler)
- Summary: An email-confirmation link on the Omise dashboard carried the user's email address directly inside the token visible in the URL. Because those URLs were crawled and archived by the Internet Archive, the addresses became permanently retrievable from public historical snapshots.
- Technique / pattern: Public archives are a persistence layer for anything that ever reached a URL. Query the Wayback CDX API and similar indexes for a target's hosts, filter for paths that look like confirmation, invite, reset, unsubscribe or share links, and decode the token — base64, hex and JWT payloads frequently carry an email or user id in the clear. This also converts short-lived leaks into permanent ones, so a rotated secret or an expired link is still worth decoding for the PII it embeds.
- Takeaway: Tokens must be opaque random identifiers looked up server-side, never encodings of the data they identify — and treat any URL the browser can reach as permanently public, because archives keep it after the link dies.

### 2026-09-20 — Server memory contents leaked into uploaded files via the file-upload endpoint (Bykea) — n/a
- Source: [HackerOne #3228011](https://hackerone.com/reports/3228011)
- Type: Information disclosure (uninitialised buffer contents written into stored objects)
- Summary: Bykea's `/talos/api/v1/files/upload` endpoint buffered uploads on the server before transferring them to Amazon S3, and a misconfiguration caused chunks of server memory to be included in some of the stored files, exposing server-side data to anyone who could retrieve them. The report was triaged as critical.
- Technique / pattern: This is the "heartbleed-shaped" upload bug: a buffer sized from a declared length but only partially filled leaves the remainder holding whatever was previously in memory. Probe it by uploading small files while varying the declared size — a `Content-Length` larger than the body, a truncated multipart part, a chunked request cut short — then download the stored object and compare its byte length to what was sent. Inspect the trailing bytes with `xxd` and grep for header fragments, tokens and other requests' content; repeat the upload to see whether the tail changes between attempts, which distinguishes leaked memory from static padding.
- Takeaway: Always zero or explicitly size upload buffers, and verify the stored object byte-for-byte against what was received. Compare sent and stored lengths as a standard check on any upload pipeline.

### 2026-09-19 — GraphQL query discloses email addresses hidden on public profiles (GitLab) — n/a
- Source: [HackerOne #985124](https://hackerone.com/reports/985124)
- Type: Information disclosure (GraphQL field not covered by the profile's privacy setting)
- Summary: A GraphQL query returned the email addresses of users whose profiles were configured not to display them, so a privacy control enforced in the profile page did not hold on the API.
- Technique / pattern: Privacy toggles are typically implemented in the view that owns them and not on the underlying field. For every 'hide my X' setting, enable it, then request the same object through GraphQL — root query, nested resolver, and via a connection from another object — and diff against the rendered page. Reaching a user through someone else's object (a project's members, an issue's author, a group's list) frequently bypasses the check that the direct profile query honours.
- Takeaway: A privacy setting is an authorization rule on a field. Enforce it where the field is resolved, and test every path that can reach the object.

### 2026-09-19 — UpdatePhabricatorIntegration mutation's base_url leaks the stored Phabricator Conduit API token (HackerOne) — n/a
- Source: [HackerOne #1161141](https://hackerone.com/reports/1161141)
- Type: Information disclosure (integration config rewrite → stored secret exfiltrated)
- Summary: The `base_url` argument of the `UpdatePhabricatorIntegration` GraphQL mutation could be changed so that the already-stored Conduit API token was sent to a different destination, disclosing the secret. HackerOne rated it high risk, high impact and highly exploitable.
- Technique / pattern: Integration settings are a two-field pair: a destination and a credential. Where the UI masks the credential but lets the destination be edited, changing the destination alone makes the server replay the stored secret to you — no read of the secret field is required. Enumerate every integration (issue trackers, chat, SIEM, webhooks, SSO metadata URLs), point the destination at a listener you control, and trigger a sync or test action.
- Takeaway: Re-editing a destination must invalidate the credential bound to it. Never replay a stored secret to a newly supplied URL without re-authenticating the integration.

### 2026-09-19 — GitHub OAuth clientID and clientSecret still valid in repository git history (Kubernetes) — n/a
- Source: [HackerOne #796139](https://hackerone.com/reports/796139)
- Type: Information disclosure (secret in version-control history)
- Summary: OAuth application credentials committed to a Kubernetes repository had been removed from the current tree long before, but remained present in the git history and were still valid and usable.
- Technique / pattern: Cloning and scanning history is cheap and the deletion commit is a signpost, not a fix: `git log -p`, `git rev-list --all` with gitleaks or trufflehog, and the GitHub Events API for force-pushed or deleted refs. The reusable move is to search for the *removal* — a commit that deletes a config line is a precise pointer to the secret's earlier location. Validate liveness with the provider's own metadata endpoint, never by exercising the credential's privileges.
- Takeaway: Rotation, not deletion, is the remediation. A secret ever committed to a public repository must be treated as disclosed from the moment of the push.

### 2026-09-19 — API key embedded in a JavaScript file — closed as intentional public Algolia search key (1Password) — n/a
- Source: [HackerOne #2923061](https://hackerone.com/reports/2923061)
- Type: Information disclosure (not a vulnerability; key was public by design)
- Summary: A key found in a JavaScript file on 1Password's public developer documentation site was reported as an exposed API key. 1Password explained it was an intentionally embedded, search-only Algolia key with strictly limited permissions, granting no access to sensitive data or privileged actions — expected behaviour rather than a flaw.
- Technique / pattern: Classify a key before reporting it. Several key families are designed to ship in client code: Algolia search-only keys, Stripe publishable `pk_` keys, Firebase web config, Google Maps browser keys, Sentry public DSNs. The test is capability, not secrecy — read the vendor's documentation for that key type, then establish what the specific key can actually do (scope or permissions endpoint where one exists, otherwise a single benign read) and report only if it exceeds its intended scope, for instance an Algolia admin key where a search key belongs.
- Takeaway: 'A key in JavaScript' is a finding only when the key's permissions exceed what a browser should hold. Demonstrating capability is the difference between a valid report and noise.

### 2026-09-19 — API token sent to a URL dictated by an untrusted project .weblate file (Weblate) — n/a
- Source: [HackerOne #3825141](https://hackerone.com/reports/3825141)
- Type: Sensitive information disclosure (credential sent to attacker-chosen origin; CVSS 6.3)
- Summary: The Weblate CLI (`wlc`) discovers config by walking the working directory and its parents for `.weblate`/`.weblate.ini`/`weblate.ini`. The API URL comes from that possibly-untrusted file, but the token from `WLC_KEY` was not origin-bound: `get_url_key()` returned the token for *any* URL. A malicious `.weblate` (via a pull request, a cloned repo, or a planted ancestor directory) pointed the URL at an attacker server, and any `wlc` command in CI with `WLC_KEY` set exfiltrated the token.
- Technique / pattern: Look for the pattern "secret from a trusted source (env var, keychain, CI secret) + destination from an untrusted source (a repo file, a redirect, a config discovered by directory walk)". Any tool that auto-discovers configuration by climbing parent directories will read a file an attacker dropped in a checked-out repo, and if the credential is not scoped to a specific origin it travels to whatever endpoint that file names. CI pipelines that run `push`/`pull` on public projects with a secret exported are the highest-value trigger.
- Takeaway: Bind every credential to the exact origin it was issued for and refuse to send it anywhere else. Configuration discovered from the filesystem is untrusted input, especially in tools that walk parent directories.

### 2026-09-19 — Exposed Google Maps API key in 8x8.vc/index.js allowing paid-service abuse (8x8) — $500
- Source: [HackerOne #3250315](https://hackerone.com/reports/3250315)
- Type: Sensitive information disclosure (client-shipped key without restrictions; CVSS 6.1)
- Summary: A Google Maps API key was embedded in `index.js` on the 8x8.vc domain. Client-side placement was intentional, but the key carried no HTTP-referrer or API restrictions, so a third party could reuse it and run up billing against paid Maps services (medium).
- Technique / pattern: Client-shipped keys (`AIza…` for Google, and analogous Maps/geocoding/analytics keys) are not secrets and cannot be hidden — the finding is not "a key is exposed" but "an exposed key lacks the restrictions that make exposure safe". Extract the key from JS bundles, then test whether it is constrained: call the API from an unlisted referrer/origin and with an unintended API enabled. A key that answers from anywhere is abusable for quota/billing exhaustion; a properly referrer-and-API-restricted key returns an error. Report the missing restriction, with the specific unauthorized call that succeeded, not merely the key's presence.
- Takeaway: For keys that must ship to the client, security comes from HTTP-referrer/origin, API and quota restrictions, not from concealment. Demonstrate impact by making an unauthorized call succeed.

### 2026-09-19 — Information leakage via a clicked link in a GitHub repository, enabling fingerprinting (GitHub) — $4,000
- Source: [HackerOne #2505761](https://hackerone.com/reports/2505761)
- Type: Sensitive information disclosure (user metadata leak, CVE-2024-9539, GitHub Enterprise Server)
- Summary: An attacker could upload a crafted SVG asset and social-engineer a victim into opening the hosted asset URL; the request leaked metadata about the clicking user that could be used to build a convincing targeted phishing page. Fixed in GHES 3.14.2/3.13.5/3.12.10/3.11.16.
- Technique / pattern: User-hosted assets served from the application's own domain (SVGs, attachments, avatars, CDN-proxied images) turn any link the victim opens into a same-origin request that can carry or reveal identifying context. Assess what an attacker-controlled asset URL learns about whoever fetches it — request headers, redirect behavior, per-user tokens in the path, or timing/response differences that distinguish an authenticated victim from an anonymous one. SVG is the recurring vehicle because it is treated as an image yet can reference external resources and script contexts.
- Takeaway: Serve user-supplied assets from a separate sandbox origin and strip identifying context from asset fetches. "Just an image link" can be a fingerprinting oracle when it is same-origin and authenticated.

### 2026-09-19 — Microsoft x-apikey exposed in Mozilla CI public logs (Mozilla) — $200
- Source: [HackerOne #3243860](https://hackerone.com/reports/3243860)
- Type: Sensitive information disclosure (secret in public CI artifacts; CVSS 5.3)
- Summary: A Microsoft telemetry API key appeared in publicly accessible Mozilla CI logs, captured inside `mitmproxy.log` artifacts from automated Firefox testing that recorded HTTP POSTs to Microsoft's telemetry endpoint. Mozilla moved the mitmproxy artifacts to internal storage.
- Technique / pattern: Public CI/CD logs and test artifacts are a first-class secret source, and *proxy capture files* are the sharp edge — when tests run traffic through mitmproxy/Charles/HAR capture, the recorded requests contain every header and token that crossed the wire, including third-party keys the project never intended to publish. Enumerate a project's public CI (task logs, job artifacts, uploaded `.har`/`.log`/`.pcap`, screenshots) and grep for key formats and `Authorization`/`x-apikey` headers. The leaking secret often belongs to a *third party* the project integrates with, which is still a valid, rewardable disclosure.
- Takeaway: Keep raw traffic captures out of public CI artifacts, and scrub headers from any log that ships publicly. Scan your own public build logs for third-party credentials, not just your own.

### 2026-09-18 — Exposed .git/config on foundation01.mariadb.org (MariaDB) — n/a
- Source: [HackerOne #1176174](https://hackerone.com/reports/1176174)
- Type: Information disclosure (exposed version-control metadata)
- Summary: The host foundation01.mariadb.org served its .git/config file over HTTP, revealing repository configuration and indicating that version-control metadata had been deployed to the web root - a path that can lead to source-code exposure.
- Technique / pattern: Include version-control and build metadata in every content-discovery sweep: /.git/config, /.git/HEAD, /.svn/entries, /.hg, along with .env, .DS_Store and editor backups. Reading .git/config is enough to demonstrate the exposure; whether the object store is also retrievable determines the severity, and that should be checked carefully and reported rather than mass-downloaded.
- Takeaway: Deploy build artifacts, not working copies. A .git directory under the web root turns a static host into a source-code and secret-history disclosure.

### 2026-09-18 — GraphQL introspection enabled on a Storefront GraphQL endpoint (Shopify) — n/a
- Source: [HackerOne #2886723](https://hackerone.com/reports/2886723)
- Type: Information disclosure (GraphQL introspection on a production endpoint)
- Summary: A Shopify storefront GraphQL endpoint answered introspection queries from unauthenticated callers, returning the complete schema including query types, fields and mutation operations, and so exposing the full API structure to anyone.
- Technique / pattern: Send the standard __schema introspection query to every GraphQL endpoint found in JS bundles and mobile traffic. Where it answers, the value is the map it provides: diff the live schema against the published documentation and enumerate the mutations and fields the UI never calls, as those are the operations least likely to have been authorization-reviewed.
- Takeaway: Introspection on production is a recon multiplier rather than a breach on its own. Disable it externally, and treat every undocumented operation it reveals as needing its own authorization audit.

### 2026-09-18 — Unauthenticated GraphQL access on the Enjin Platform (Enjin) — n/a
- Source: [HackerOne #3452015](https://hackerone.com/reports/3452015)
- Type: Exposed API (GraphQL operations reachable without authentication)
- Summary: Certain GraphQL queries on the Enjin Platform schema could be executed with no authentication. The reporter traced the cause to the open-source platform-core repository, which let the team locate and fix the unguarded code quickly.
- Technique / pattern: Against a product whose code is public, pair black-box probing with source review: after finding an operation that answers unauthenticated, locate its resolver in the repository and check which middleware, directive or policy the authorized operations carry and it does not. Naming the missing guard in the report turns a symptom into a precise, quickly actionable fix.
- Takeaway: In GraphQL, authorization is per-resolver, so one unannotated resolver is a hole in an otherwise protected schema. With an open-source target, cite the code path - it speeds triage and proves the root cause.

### 2026-09-18 — Team object disclosed via GraphQL despite account access restrictions (HackerOne) — n/a
- Source: [HackerOne #342978](https://hackerone.com/reports/342978)
- Type: Information disclosure (GraphQL object-level authorization)
- Summary: Team-related objects that should have been restricted were retrievable through the GraphQL API, with the access controls available on accounts - such as two-factor authentication and IP whitelisting - not enforced at the API layer for that object.
- Technique / pattern: Request restricted objects through GraphQL by their global id and through nested resolvers rather than only through the routes the UI uses. Account-level protections such as IP allowlisting and 2FA are frequently applied at the web session layer, so reaching the same object by a different path shows whether the restriction is a property of the object or only of one entry point.
- Takeaway: Access restrictions must be enforced where the data is resolved. If a protection only exists on the UI path, the API is an unrestricted second door to the same objects.

### 2026-09-18 — Proxy credentials leak over redirect to a different proxy (CVE-2026-6253) (curl) — n/a
- Source: [HackerOne #3669637](https://hackerone.com/reports/3669637)
- Type: Sensitive information disclosure (credential leak across a trust boundary)
- Summary: When a request redirected from HTTP to HTTPS and libcurl re-selected the proxy accordingly, credentials configured for the first proxy were retained and sent to the second proxy's CONNECT request, exposing reusable proxy secrets to a different administrative domain.
- Technique / pattern: Model credential scope as a trust boundary and then look for state that survives a transition across it: protocol switches, redirects and connection reuse are where a client re-decides which endpoint to talk to but often forgets to re-decide which secret applies. A test harness of two proxies plus a redirecting server makes the leak directly observable.
- Takeaway: Secrets must be scoped to the exact endpoint they were configured for, and re-evaluated whenever the destination changes - including changes driven by a remote server's redirect.

### 2026-09-18 — Information disclosure through an exposed .env file via directory listing (AWS VDP) — n/a
- Source: [HackerOne #2784712](https://hackerone.com/reports/2784712)
- Type: Sensitive information disclosure (exposed environment/config file)
- Summary: A publicly reachable endpoint served an application .env file through directory listing, exposing database credentials and mail service authentication details.
- Technique / pattern: Combine directory-listing detection with a wordlist of deployment artefacts (.env, .env.bak, config.json, docker-compose.yml). Directory listing turns a blind guessing exercise into enumeration: once the index renders, the sensitive filename no longer has to be guessed, so check for listing before brute-forcing paths.
- Takeaway: Environment files belong outside the web root, and directory listing should be disabled by default - together they turn one deployment slip into a full credential set.

### 2026-09-18 — PII leakage through links shared in HackerOne reports (HackerOne) — n/a
- Source: [HackerOne #1256371](https://hackerone.com/reports/1256371)
- Type: Sensitive information disclosure (PII via parameter manipulation on a fulfilment link)
- Summary: A swag distribution link published inside a disclosed report could be altered by adding a parameter, returning other researchers' personal data including postal addresses and phone numbers.
- Technique / pattern: Treat URLs embedded in public artefacts - disclosed reports, screenshots, support threads - as attack surface, then test the linked third-party workflow for parameter-level access control. Fulfilment, survey and shipping tools are rarely in the main threat model yet routinely hold PII.
- Takeaway: Disclosure and redaction processes must cover links, not just text, and any third-party form handling personal data needs the same authorization scrutiny as first-party endpoints.

### 2026-09-18 — Exposed .git repository reveals source code and database credentials (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1629822](https://hackerone.com/reports/1629822)
- Type: Sensitive information disclosure (exposed .git directory)
- Summary: A live server exposed its .git metadata directory, allowing reconstruction of the application's source - classes, models and controllers - along with a config.json holding database username and password.
- Technique / pattern: Probe for /.git/config and /.git/HEAD on each host; a valid response means the object store is retrievable and the working tree can be rebuilt offline. Reading the recovered source then converts a single misconfiguration into a map of the application's routes, authentication logic and embedded secrets.
- Takeaway: Deploying from a git checkout ships the whole repository history. Block dotfile directories at the web server and keep credentials in environment configuration that is never committed.

### 2026-09-18 — Private program metadata disclosed by enumerating GraphQL node IDs (HackerOne) — n/a
- Source: [HackerOne #1618347](https://hackerone.com/reports/1618347)
- Type: Information disclosure (GraphQL node/global-ID enumeration)
- Summary: A GraphQL endpoint resolved `PolicyPageAssetGroup` nodes by ID without checking the caller's access to the owning program, so enumerating IDs exposed private program names, scope details and associated private report titles.
- Technique / pattern: Collect the global-ID scheme a GraphQL API uses, then request nodes by ID directly instead of through the queries the UI issues; node resolvers are frequently added without the authorization the surrounding query already performs.
- Takeaway: In GraphQL, authorization belongs on the type/resolver, not on the entry-point query — any object reachable by a global ID is effectively a public endpoint until it checks the viewer.

### 2026-09-18 — Secure-schema filter bypass exposes private data through the `or` operator (HackerOne) — n/a
- Source: [HackerOne #645299](https://hackerone.com/reports/645299)
- Type: Information disclosure (GraphQL filter/secure-schema bypass)
- Summary: The secure schema restricting GraphQL `where` filters could be circumvented by nesting conditions under the `or` operator, letting a caller filter on and infer data the schema was meant to keep out of reach.
- Technique / pattern: Treat rich filter arguments as an oracle: combine allowed predicates with boolean operators (`or`, `not`, nested groups) and read the result count or ordering — even without the field being returned, a filterable field is a readable field one bit at a time.
- Takeaway: Any attribute you allow in a query filter is disclosed, whatever the field-level permissions say; authorize the filter input itself, including everything reachable through logical operators.

### 2026-09-18 — Inadequate redaction exposes sensitive references via GraphQL (HackerOne) — n/a
- Source: [HackerOne #2357012](https://hackerone.com/reports/2357012)
- Type: Information disclosure (redaction applied at the view layer only)
- Summary: The redaction feature that conceals data such as JIRA references hid it in the rendered interface, but the underlying values were still retrievable through GraphQL requests.
- Technique / pattern: Whenever the UI masks, truncates or redacts something, request the same object through the API and compare: masking implemented in the presentation layer leaves the original value in the response payload.
- Takeaway: Redaction has to happen where the data is served, not where it is displayed — if the value still travels to the client, it is disclosed regardless of how the page renders it.

### 2026-09-18 — Hardcoded third-party API key and secret in the mobile application package (Reddit) — n/a
- Source: [HackerOne #1241116](https://hackerone.com/reports/1241116)
- Type: Information disclosure (secrets embedded in a shipped client)
- Summary: Twitter consumer key and consumer secret values were shipped inside the application package, recoverable from `res/values/strings.xml` in the decompiled resources, exposing credentials intended only for the developer.
- Technique / pattern: Decompile the published app and grep the resource tables and string pools — not just the code — for `*_key`, `*_secret`, `client_id` and base64-looking constants; resource XML is where build-time configuration usually lands.
- Takeaway: Anything shipped to a client is public: third-party secrets belong on a server that proxies the call, and any key found in a binary should be treated as compromised and rotated rather than merely removed.

### 2026-09-18 — Google API key leaked to public across multiple hosts (FetLife) — n/a
- Source: [HackerOne #1065041](https://hackerone.com/reports/1065041)
- Type: Sensitive information disclosure (unrestricted third-party API key)
- Summary: Several FetLife hosts exposed a Google API key that the researcher confirmed was usable against the Geocoding API — evidence it was unrestricted and billable to the owner.
- Technique / pattern: Collect `AIza…` keys from responses across all subdomains, then test each against specific Google APIs to establish which are enabled and unrestricted — this is what turns 'a key is visible' into demonstrated impact.
- Takeaway: Client-shipped Google keys are unavoidable, so the real control is API and referrer/IP restriction; a key that answers an unrelated API call is a live billing and quota liability.

### 2026-09-18 — Attachment disclosure via the report summary feature (HackerOne) — n/a
- Source: [HackerOne #2442008](https://hackerone.com/reports/2442008)
- Type: Sensitive information disclosure (IDOR on attachment identifiers at render time)
- Summary: The report-summary editing feature resolved attacker-supplied attachment identifiers without an ownership check, returning files belonging to other users' reports.
- Technique / pattern: Wherever a feature composes content by attachment or file id — summaries, previews, exports, embeds — substitute ids harvested from another context and check whether the renderer resolves them without re-authorizing.
- Takeaway: File and attachment ids are objects in their own right: authorize them at render time, not only at upload or at the parent record, because composition features are exactly where that check gets skipped.

### 2026-09-18 — Information disclosure in the /skills endpoint during staged rollout (HackerOne) — n/a
- Source: [HackerOne #188719](https://hackerone.com/reports/188719)
- Type: Sensitive information disclosure (authorization gap behind a feature flag)
- Summary: During the staged rollout of the skill sets feature, the `/skills` endpoint exposed data to roughly 20 early-access users who should not have been able to see it; it was identified and resolved the same day.
- Technique / pattern: Target features in limited beta or staged rollout: enumerate the endpoints the new UI calls and check whether their authorization matches the feature flag that gates the interface.
- Takeaway: Feature flags hide UI, not data — every endpoint behind a staged rollout needs its own authorization check, and new features deserve an early re-audit.

### 2026-09-17 — API access to Phabricator via leaked certificate in git repo (Uber) — $39,999
- Source: [HackerOne #591813](https://hackerone.com/reports/591813)
- Type: Sensitive information disclosure (credential leaked through source control)
- Summary: A client certificate committed to a git repository granted API access to Uber's internal Phabricator instance and the code-review data held there.
- Technique / pattern: Mine repository history rather than only the current tree for credential material (certificates, private keys, tokens), then validate the recovered material against the internal endpoint it belongs to, minimally and in scope.
- Takeaway: Removing a secret from HEAD does not remove it from history — rotate on exposure, and scan full history including forks, mirrors and CI caches.

### 2026-09-17 — OTP code Leaked in the API Response (MTN Group) — n/a
- Source: [HackerOne #2635315](https://hackerone.com/reports/2635315)
- Type: Sensitive information disclosure (over-exposed API response)
- Summary: The API that triggers a one-time passcode returned the OTP value inside its own response body, so anyone who could request a code for an account could simply read it.
- Technique / pattern: For every authentication step, compare what the UI renders against the full raw API response, looking for fields the client never displays.
- Takeaway: Read raw responses rather than rendered pages — backends routinely return more than the front end shows, and a leaked OTP collapses the entire second factor.

### 2026-09-17 — Partner PII via Unauthenticated API Endpoint (Starbucks) — n/a
- Source: [HackerOne #659248](https://hackerone.com/reports/659248)
- Type: Exposed API / PII disclosure
- Summary: An API endpoint served partner (employee) personal data without requiring authentication, exposing PII to anyone who knew or could guess the route.
- Technique / pattern: Harvest API routes from JavaScript bundles, mobile app traffic and vendor documentation, then replay each request with all cookies, tokens and auth headers stripped to find routes that never had their own check.
- Takeaway: Every endpoint needs its own authorization check — re-issue discovered requests unauthenticated instead of assuming a gateway enforces it.

### 2026-09-17 — Undocumented fileCopy GraphQL API (Shopify) — $2,000
- Source: [HackerOne #981472](https://hackerone.com/reports/981472)
- Type: Exposed API (undocumented GraphQL operation)
- Summary: An undocumented GraphQL mutation, fileCopy, was reachable on the public API and permitted file operations that were never intended to be exposed to merchants and third-party apps.
- Technique / pattern: Run GraphQL introspection, or diff the live schema against the published documentation, to list operations that exist but are undocumented — then test each for missing authorization.
- Takeaway: The schema, not the documentation, defines a GraphQL attack surface: introspect and diff, because undocumented usually also means unreviewed.

### 2026-09-17 — /reports/:id.json endpoint leaking reporter's sensitive data (HackerOne) — n/a
- Source: [HackerOne #3000510](https://hackerone.com/reports/3000510)
- Type: Exposed API / Information Disclosure (format-variant over-serialization)
- Summary: The .json representation of a disclosed report returned far more than the HTML view — reporter email, OTP backup codes, phone number, GraphQL secret token and more — an over-scoped serializer on the format variant.
- Technique / pattern: Appended .json to a disclosed report URL and inspected the response, finding sensitive fields the rendered page never exposed.
- Takeaway: Alternate response formats (.json/.xml/API variants) often bypass view-layer field filtering — audit serializers per format and scope them to what the caller may see.

### 2026-09-17 — Private list members disclosure via GraphQL (X / xAI) — n/a
- Source: [HackerOne #885539](https://hackerone.com/reports/885539)
- Type: Exposed API / Information Disclosure (timing + broken rate-limit)
- Summary: A GraphQL endpoint let an attacker reconstruct the members of private lists by chaining a timing attack with broken rate limiting.
- Technique / pattern: Located GraphQL endpoints (`api.twitter.com/graphql/<hash>/<name>`) and used response-timing differences plus missing rate limits to enumerate private list membership.
- Takeaway: Authorization must not leak through side channels — timing differences and absent rate limits can reconstruct data the API refuses to return directly; use constant-time checks and throttling.

### 2026-09-17 — Unauthenticated API endpoint (/users) discloses PII (U.S. Dept of Defense) — n/a
- Source: [hackerone #3027405](https://hackerone.com/reports/3027405)
- Type: Sensitive Information Disclosure (over-scoped / unauthenticated API)
- Summary: An API endpoint returning user records (identifiers, names, email addresses, roles, and auth-related fields) was reachable without authentication, leaking personal data to anyone.
- Technique / pattern: Discover the user-listing API path during recon and call it unauthenticated, observing full PII returned with no access control.
- Takeaway: Data-returning API endpoints must enforce authentication and object/field-level authorization; "internal" or undocumented endpoints are still externally reachable and must be gated.

### 2026-09-17 — Open AWS S3 bucket leaks all user-uploaded chat images (Zomato) — n/a
- Source: [hackerone #507097](https://hackerone.com/reports/507097)
- Type: Sensitive Information Disclosure (open cloud storage)
- Summary: A publicly accessible S3 bucket exposed every image uploaded to Zomato Chat, readable by anyone without credentials.
- Technique / pattern: Identify the bucket, enumerate objects via the AWS CLI (e.g. by year/month prefixes), and fetch files directly from the bucket's public URL structure.
- Takeaway: Cloud storage holding user content must be private with least-privilege ACLs/policies; audit buckets for public list/read and never rely on obscure object paths for protection.

### 2026-09-16 — GraphQL introspection query leaks sensitive schema/data (HackerOne) — n/a
- Source: [hackerone #291531](https://hackerone.com/reports/291531)
- Type: Exposed API / Sensitive Information Disclosure (GraphQL introspection)
- Summary: A GraphQL endpoint left introspection enabled, letting anyone query the full schema and discover types, fields, and operations that were not meant to be publicly exposed.
- Technique / pattern: Send a standard `__schema` / `__type` introspection query to map hidden queries, mutations, and fields, then target the newly discovered operations.
- Takeaway: Disable (or restrict) introspection in production and never rely on schema obscurity — enforce authorization on every field and operation.

### 2026-09-16 — Confidential user/program data queryable via GraphQL endpoint (HackerOne) — n/a
- Source: [hackerone #489146](https://hackerone.com/reports/489146)
- Type: Exposed API / Sensitive Information Disclosure (GraphQL authorization bypass)
- Summary: After HackerOne migrated to a class-based GraphQL backend, missing permission checks let researchers query confidential user data and limited program/report metadata through the GraphQL endpoint.
- Technique / pattern: After a backend/framework migration, re-test GraphQL nodes and fields for missing object-level authorization; newly wired resolvers often ship without the old permission gates.
- Takeaway: Enforce field- and object-level authorization on every resolver, and re-audit access control after any backend refactor or migration.

### 2026-09-15 — User API key leaked (WakaTime) — n/a
- Source: [HackerOne #3098717](https://hackerone.com/reports/3098717)
- Type: Sensitive information disclosure (leaked API key)
- Summary: A WakaTime user's API key was exposed in an older/legacy URL encountered while testing; the leaked key was still valid and successfully authenticated requests to a restricted endpoint.
- Technique / pattern: Inspect legacy/deprecated URLs, embeds, and cached responses for live credentials, then confirm impact by using the key against protected API endpoints.
- Takeaway: Treat any credential that ever appeared in a URL as compromised — rotate leaked keys immediately, keep secrets out of URLs, and scope API keys to least privilege.

### 2026-09-15 — Leaking of sensitive information in public GitHub repo (Liberapay) — n/a
- Source: [HackerOne #837733](https://hackerone.com/reports/837733)
- Type: Sensitive information disclosure (secrets in source control)
- Summary: Liberapay's public GitHub repository contained committed secrets — AWS credentials plus multiple OAuth client secrets/API keys (Twitch, Facebook, Twitter, GitHub, Bitbucket, Google, etc.).
- Technique / pattern: Scan a target's public repositories (and their git history) with a secret-detection tool such as gitleaks to surface inadvertently committed keys and tokens.
- Takeaway: Never commit secrets to source control; scan repos and history in CI, and rotate any exposed credential — git history preserves secrets even after deletion.

### 2026-09-15 — User information disclosed via unauthenticated API endpoint (GSA) — n/a
- Source: [HackerOne #1218461](https://hackerone.com/reports/1218461)
- Type: Sensitive Information Disclosure (missing authentication)
- Summary: A "system accounts" API endpoint lacked authentication, exposing user data including email addresses and organization/integration details (e.g. submitted IP addresses).
- Technique / pattern: Enumerate API paths and call them without credentials; an endpoint intended for internal/authenticated use returned records to any caller, leaking PII directly.
- Takeaway: Every data-returning API needs an authorization check; "internal" endpoints reachable externally must not assume the caller is trusted.

### 2026-09-15 — GitHub access token disclosed via Nginx off-by-slash exposing .git (Adobe) — n/a
- Source: [HackerOne #1386547](https://hackerone.com/reports/1386547)
- Type: Sensitive Information Disclosure (source/secret exposure via misconfig)
- Summary: An Nginx "off-by-slash" alias misconfiguration allowed path traversal into the application's `.git` directory, whose config exposed a GitHub access token usable to retrieve source code.
- Technique / pattern: Where an `alias` location lacks a trailing slash (`location /assets { alias /path/assets/; }`), request `/assets../.git/config` to traverse to the parent and read exposed `.git` files, then extract credentials/tokens.
- Takeaway: Ensure Nginx `alias` locations end with a trailing slash (or use `root`), block dotfiles like `.git`, and rotate any token that touched a public path.

### 2026-09-14 — Non-revoked API key disclosure (Stripo Inc) — n/a
- Source: [HackerOne #1047125](https://hackerone.com/reports/1047125)
- Type: Sensitive information disclosure — leaked live API key
- Summary: An API key exposed publicly was found still active (not revoked), so the leaked credential remained usable rather than harmless.
- Technique / pattern: When a secret is found (repos, JS bundles, history), safely validate whether it is still live before reporting — a non-revoked key has real impact; a rotated one does not.
- Takeaway: Rotate/revoke secrets immediately on exposure and add secret scanning to CI; the impact of a leak hinges on whether the key is still valid.

### 2026-09-14 — Google/Firebase API key leaked in client-side JavaScript (Clario) — n/a
- Source: [HackerOne #1066410](https://hackerone.com/reports/1066410)
- Type: Sensitive information disclosure — hardcoded API key
- Summary: A Google API key was hardcoded in a JavaScript file and used against the Firebase Dynamic Links endpoint, exposing the credential to anyone reading the client bundle.
- Technique / pattern: Grep JS bundles for key formats (AIza... for Google) and identify the service they call; gauge real impact by checking the key's enabled APIs and restrictions rather than assuming exposure equals compromise.
- Takeaway: Keep privileged keys off the client, apply API/referrer restrictions to any key that must ship client-side, and rotate leaked keys.


### 2026-09-13 — Sensitive AWS keys leaked in public GitHub repo (AWS VDP) — n/a
- Source: [HackerOne #3017105](https://hackerone.com/reports/3017105)
- Type: Sensitive information disclosure (leaked secret)
- Summary: An AWS Access Key and Secret Key were committed to a public GitHub repository, exposing credentials that could be abused against AWS services.
- Technique / pattern: Grep public repos and org members' repos for key formats (e.g. AKIA...), including commit history and un-scrubbed files; validate scope carefully without abusing the credentials. Tools: gitleaks/trufflehog.
- Takeaway: Keep secrets out of source control (secret managers, pre-commit scanning); rotate any key that ever touched a repo, since history preserves it.

### 2026-09-13 — Sensitive data leaked via developer JS file (Reddit) — n/a
- Source: [HackerOne #2353237](https://hackerone.com/reports/2353237)
- Type: Sensitive information disclosure (client-side secret exposure)
- Summary: A JavaScript file intended for internal developers was publicly reachable and leaked sensitive data including API keys and other credentials.
- Technique / pattern: Enumerate and read all served JS bundles, source maps, and "internal"/debug scripts; developer-only assets left on the public origin routinely embed secrets and internal endpoints.
- Takeaway: Never ship secrets in client-delivered JavaScript; gate internal tooling behind auth and keep it off public origins.

### 2026-09-12 — API keys hardcoded in a public GitHub repository (Rocket.Chat) — n/a
- Source: [HackerOne #766346](https://hackerone.com/reports/766346)
- Type: Sensitive information disclosure (hardcoded secrets)
- Summary: A public GitHub repository belonging to the program contained hardcoded API keys and a `google-services.json` file exposing sensitive credentials.
- Technique / pattern: Searched the org's public repositories for committed secrets (keys, service-account/config files) and confirmed the exposed values.
- Takeaway: Keep secrets out of source control, scan repos and history with secret-detection tooling, and rotate any key that has ever been committed.

### 2026-09-12 — Public and secret API key leaked in client-side JavaScript (Top Echelon Software) — n/a
- Source: [HackerOne #1051029](https://hackerone.com/reports/1051029)
- Type: Sensitive information disclosure (exposed API key)
- Summary: The job board site's search functionality shipped both a public and a secret API key inside a publicly accessible JavaScript file.
- Technique / pattern: Reviewed loaded JS bundles for embedded credentials and extracted the secret key directly from client-side source.
- Takeaway: Never embed secret keys in front-end code; the client should carry only public/scoped tokens, with sensitive operations proxied server-side.

### 2026-09-11 — Private API keys leaked (Reddit) — n/a
- Source: [HackerOne #1762927](https://hackerone.com/reports/1762927)
- Type: Sensitive information disclosure — leaked API keys / credentials
- Summary: Valid private API keys were exposed within Reddit's application surface; disclosure of such keys can grant unauthorized access to any systems that rely on them for authentication.
- Technique / pattern: Harvest secrets from reachable surfaces (JS bundles, configs, responses), then validate whether an exposed key is live and in-scope — without abusing it — to demonstrate impact.
- Takeaway: Treat any key reachable by clients as compromised; scan for secret patterns, confirm validity before reporting, and rotate/scope exposed keys.

### 2026-09-11 — PII exposed via dashboard widgets (U.S. Dept of Defense) — n/a
- Source: [HackerOne #819591](https://hackerone.com/reports/819591)
- Type: Sensitive information disclosure / improper access control
- Summary: A dashboard let a user add "widgets" that returned large amounts of data including PII (full names, emails, physical addresses, phone numbers) and diagnostic info, and even allowed changes to certain catalogs.
- Technique / pattern: Exercise every configurable/optional feature (widgets, add-ons, report builders) and observe whether it returns more than the current user should see; over-scoped data-fetching components frequently leak other users' PII.
- Takeaway: Data-returning UI components must apply the same per-user authorization as core APIs; "add a widget/report" features are a common over-scoped disclosure vector.

### 2026-09-11 — Public and secret API key leaked via JavaScript file (Stripo Inc) — n/a
- Source: [HackerOne #983331](https://hackerone.com/reports/983331)
- Type: Sensitive information disclosure — secret in client-side JS
- Summary: Both public and secret API keys were exposed in a publicly accessible bundled JavaScript file on a staging host (staging.empleio.stripo.email).
- Technique / pattern: Download the site's JS bundles/source maps and grep for key patterns; a secret key shipped to the client is usable directly against the API. Don't forget staging/subdomain hosts.
- Takeaway: Secret keys must never reach the client — scan JS bundles (and non-prod hosts) for leaked credentials and rotate any exposed key immediately.

### 2026-09-11 — AppLovin API key hardcoded in a public GitHub repository (X / xAI) — n/a
- Source: [HackerOne #674774](https://hackerone.com/reports/674774)
- Type: Sensitive information disclosure — hardcoded secret in source control
- Summary: An AppLovin SDK/API key was hardcoded in the source of a public GitHub project (mopub-android-mediation), exposing the credential.
- Technique / pattern: Search public GitHub repos, forks and commit history for hardcoded key patterns (gitleaks/trufflehog); validate the key's scope without abusing it before reporting.
- Takeaway: Keep secrets out of source control; run secret scanning over repos AND their history, and rotate leaked keys — deleting the line does not remove it from git history.

### 2026-09-10 — Insecurely issued api.data.gov API keys (GSA) — n/a
- Source: [HackerOne #266449](https://hackerone.com/reports/266449)
- Type: Sensitive info disclosure / insecure API-key issuance
- Summary: The api.data.gov key-signup mechanism let a remote user request valid working API keys for arbitrary email addresses without secure verification, handing out usable credentials.
- Technique / pattern: Probe key/registration endpoints — submit signup requests for emails you don't control and check whether a working key is returned or derivable without email verification or a secure channel.
- Takeaway: API-key issuance must verify the requester owns the email and deliver keys out-of-band; unauthenticated key minting is a disclosure/abuse vector.

### 2026-09-10 — Public and secret API keys leaked (Omise) — n/a
- Source: [HackerOne #508024](https://hackerone.com/reports/508024)
- Type: Secret/credential disclosure
- Summary: An Omise account's public and secret API keys were exposed; the secret key enables privileged operations (customer creation, payments, data retrieval), so its leak allowed unauthorized transactions and data access.
- Technique / pattern: Hunt keys in client assets, JS and embed snippets; distinguish public (safe, client-side) from secret keys by prefix/scope, and validate a secret key's impact without abusing it.
- Takeaway: Never expose secret/live keys client-side; scope keys tightly, rotate on leak, and treat a leaked secret key as critical (full payment/API access).

### 2026-09-09 — Disclose any user's private email through API (HackerOne)
- Source: [HackerOne #196655](https://hackerone.com/reports/196655)
- Type: Sensitive information disclosure (API / broken object-level authZ)
- Summary: An API endpoint returned **any user's private email** when queried with their identifier — PII exposed without scoping the response to the requester.
- Technique / pattern: Probe API responses (REST/GraphQL) for fields beyond your own scope; swap ids; inspect GraphQL introspection + nested resolvers that over-return data.
- Takeaway: APIs must scope responses to the authenticated principal; "disclosure" IDOR on PII is valid, high-signal impact. Overlaps with [[IDOR]].

### 2026-09-09 — JumpCloud API key leaked via exposed resource (Starbucks)
- Source: [HackerOne #716292](https://hackerone.com/reports/716292)
- Type: Secret/credential disclosure
- Summary: A **JumpCloud API key** was exposed via a publicly reachable resource, handing over privileged API access.
- Technique / pattern: Hunt secrets in JS bundles, source maps, open directories, `.git`, S3/buckets, CI logs, and error pages; grep for key formats; validate the key's scope safely.
- Takeaway: One leaked key can equal full service access — treat exposed secrets as critical and report without abusing them. Use trufflehog/gitleaks-style hunting on in-scope assets. See [[Wordlists]]/[[OSINT]].
