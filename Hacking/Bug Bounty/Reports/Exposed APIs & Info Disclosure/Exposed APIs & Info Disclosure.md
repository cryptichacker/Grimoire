---
tags: [hacking, bug-bounty, info-disclosure, api, secrets, reports]
type: log
---

# Exposed APIs & Info Disclosure

## Up
- [[Reports]]

Disclosed **exposed API / sensitive information disclosure** reports — leaked API keys/secrets, PII returned by APIs, exposed config/debug endpoints, open `.git`/backups. See [[Pattern]] → Exposed APIs & Info Disclosure.

## Reports

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
- Technique / pattern: Located GraphQL endpoints (api.twitter.com/graphql/<hash>/<name>) and used response-timing differences plus missing rate limits to enumerate private list membership.
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
