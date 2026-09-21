---
tags: [hacking, bug-bounty, injection, sqli, reports]
type: log
---

# SQLi

## Up
- [[Injection]]

Disclosed **SQL injection** reports — error/boolean/time-based, UNION, blind, and second-order. See [[Pattern]] → Injection (SQLi).

## Reports

### 2026-09-21 — SQL injection extracts Starbucks enterprise accounting/payroll DB (Starbucks) — bounty
- Source: [HackerOne #531051](https://hackerone.com/reports/531051)
- Type: SQLi (time-based, XML-encoded bypass)
- Summary: A Microsoft Dynamics AX web service accepted XML HTTP requests to an accounting endpoint; XML entity encoding of the apostrophe (&apos;) slipped past the app's quote-escaping, enabling time-based SQLi against MSSQL 2012.
- Technique / pattern: Encode the injection's single quote as the XML entity &apos; so it survives the app's escaping and reaches the query, then confirm/extract with time-based payloads across ~1M accounting records.
- Takeaway: When input arrives as XML/JSON, escape filters that assume raw characters can be defeated by an equivalent entity/encoding — test encoded forms of quotes against every serialization the endpoint accepts.

### 2026-09-21 — SQL injection in GraphQL endpoint via embedded_submission_form_uuid (HackerOne) — n/a
- Source: [HackerOne #435066](https://hackerone.com/reports/435066)
- Type: SQLi (GraphQL, PostgreSQL SET SESSION)
- Summary: The /graphql endpoint interpolated the unsanitized embedded_submission_form_uuid parameter into a PostgreSQL 'SET SESSION' statement, allowing injection confirmed with pg_sleep().
- Technique / pattern: Inject into embedded_submission_form_uuid in the GraphQL request and confirm with a time-based pg_sleep() payload; the value reached SQL through a session-config statement rather than a normal query.
- Takeaway: SET SESSION / configuration statements are overlooked SQL sinks — any user value placed into session setup interpolation is injectable even outside conventional SELECT/WHERE clauses.


### 2026-09-21 — Time-based SQL injection in search parameter (U.S. Department of State) — n/a
- Source: [HackerOne #1878584](https://hackerone.com/reports/1878584)
- Type: SQL injection - time-based blind
- Summary: A GET parameter in the site's search functionality was vulnerable to time-based blind SQL injection, confirmed with SQLMap, allowing table enumeration and potential extraction of sensitive data. Disclosed 2023-04-20.
- Technique / pattern: Probe a search parameter with time-delay payloads, confirm conditional response timing, then automate enumeration with SQLMap to prove impact without dumping live data.
- Takeaway: Search/filter parameters remain a top source of SQLi - use parameterized queries and ensure no user input concatenates into SQL, since blind time-based injection is exploitable even with no visible output.

### 2026-09-21 — Potential SQL Injection when annotating FilteredRelation on PostgreSQL (Django) — n/a
- Source: [HackerOne #3417967](https://hackerone.com/reports/3417967)
- Type: SQLi - ORM alias injection via PostgreSQL dollar-quoting (CVE-2025-57833 / CVE-2025-59681)
- Summary: Django's FORBIDDEN_ALIAS_PATTERN, which rejects unsafe annotation aliases, did not block '$'; on PostgreSQL a user-controlled FilteredRelation annotation key could use $tag$...$tag$ dollar-quoting to inject SQL (e.g. pg_read_file).
- Technique / pattern: The alias is emitted several times in the generated query; dollar-quotes in the payload swallow the SQL between those copies so the injected subquery executes. Fixed by adding '$' to the forbidden pattern.
- Takeaway: Deny-list regexes on identifiers must cover every quoting form of every supported dialect (PG $$, MySQL backticks...). Passing user input as ORM kwargs/alias names (e.g. **request.GET into annotate) defeats ORM protection.

### 2026-09-20 — Blind SQL injection via the clientid parameter in zone-include.php (Revive Adserver) — n/a
- Source: [HackerOne #3653196](https://hackerone.com/reports/3653196)
- Type: SQLi (blind, post-authentication, low-privilege admin-panel script)
- Summary: In Revive Adserver 6.0.6 and earlier, the `zone-include.php` script did not sanitise user input, letting an authenticated low-privileged user inject SQL through the `clientid` parameter and query the database blindly. The fix added validation for every parameter the script processes.
- Technique / pattern: In a self-hosted product, the interesting surface is the *authenticated, low-privilege* admin panel rather than the public site: dozens of small maintenance scripts take an id, and each one was written by a different hand. Get a basic account, crawl every panel page as that role, collect each `id`-shaped parameter, and run the boolean pair before any time-based payload. Reproduce against the vendor's own release so the report cites a version range rather than a single host.
- Takeaway: "Requires a login" is not a mitigation when the lowest role in the product can reach the parameter. Parameterize in the small scripts too — they are where the audit stops looking.

### 2026-09-19 — SQL injection in a DoD web application parameter (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1015406](https://hackerone.com/reports/1015406)
- Type: SQLi (classic parameter injection)
- Summary: A DoD application passed a user-supplied request parameter into an SQL statement without separating it from the query, allowing an attacker to alter the statement and reach data the application was not meant to expose.
- Technique / pattern: Government and enterprise portals commonly front a database directly through server-rendered pages. Enumerate every parameter that plausibly reaches a WHERE, ORDER BY or LIKE clause, send the boolean pair (`' AND 1=1--` vs `' AND 1=2--`) and compare response length and status before escalating to a time-based oracle. Confirm with the minimum proof the program accepts — a version string or a boolean differential — and stop there.
- Takeaway: Parameterize every query. On legacy stacks the finding is usually in the least-glamorous search or filter field, not the flagship feature.

### 2026-09-19 — SQL injection on docs.atavist.com (Automattic) — n/a
- Source: [HackerOne #1039315](https://hackerone.com/reports/1039315)
- Type: SQLi (injection on a documentation/secondary subdomain)
- Summary: A parameter on Automattic's `docs.atavist.com` documentation host was injectable, exposing the backing database of a site that sat outside the main product's hardened code path.
- Technique / pattern: Documentation, marketing, status, careers and legacy-product subdomains run on separate stacks — often an old CMS or a bespoke PHP app — maintained by a different team and excluded from the main product's security review and WAF coverage. After enumerating subdomains, sort them by how unlike the flagship app they look, and test those first; the parameter surface is small but the code is frequently a decade older.
- Takeaway: Attack surface follows the org chart. A secondary subdomain inside the same scope is often the weakest code the program owns.

### 2026-09-19 — SQL injection in the column type parameter of Nextcloud Tables (Nextcloud) — n/a
- Source: [HackerOne #3462991](https://hackerone.com/reports/3462991)
- Type: SQL injection (identifier/type parameter, CVE-2026-45545)
- Summary: The Nextcloud Tables app built database statements from a user-supplied column *type* parameter without sanitizing it, letting a low-privileged authenticated user execute arbitrary SQL and read or write data outside their scope (CVSS 8.2, high).
- Technique / pattern: Continue the identifier-injection hunt beyond column names and aliases into *type* and *schema* parameters. Anywhere an app lets a user define structure rather than supply data — create-a-table, add-a-column, change-a-field-type, dynamic report builders, custom-field designers — the value ends up concatenated into DDL/DML because placeholders cannot bind identifiers or types. Probe these with a valid type plus an appended clause rather than the usual `'` and watch for 500s or timing shifts.
- Takeaway: Parameterization cannot protect identifiers or types. Validate those against a fixed allow-list of known-good values; anything user-defined about schema is a SQL sink by construction.

### 2026-09-19 — WooCommerce SQL injection in WC_Report_Coupon_Usage (Automattic) — n/a
- Source: [HackerOne #3198980](https://hackerone.com/reports/3198980)
- Type: SQL injection (blind/time-based, authenticated reporting endpoint)
- Summary: In WooCommerce 9.9.3 the `coupon_codes` GET parameter of the coupon-usage report reached `get_order_report_data()` insufficiently escaped. The researcher hit `/wp-admin/admin.php?page=wc-reports&tab=orders&report=coupon_usage` with `')+union+select+1,sleep(10)--+-` and confirmed the flaw from the delayed response.
- Technique / pattern: The code *did* call `sanitize_text_field()` — a string-cleaning helper, not a SQL escaper. Treat the presence of any sanitizer as a prompt to check which threat it addresses: functions that strip tags, trim whitespace or normalize encoding leave quotes and SQL syntax intact. Reporting and analytics screens are the richest hunting ground here, because they assemble ad-hoc aggregate queries from filter parameters rather than going through the ORM, and they sit behind a low-privilege role that many testers never exercise.
- Takeaway: A sanitizer is not parameterization. Grep for query construction that interpolates a variable which passed through an XSS-oriented cleaner, and always test low-privilege reporting roles rather than only admin and anonymous.

### 2026-09-18 — Blind SQL injection in a public-facing parameter (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #2737595](https://hackerone.com/reports/2737595)
- Type: SQL injection (blind - boolean/time-based)
- Summary: A request parameter was concatenated into a database query, letting an attacker manipulate the query through injected input. The endpoint returned no query results directly, so extraction relied on inferring data from boolean and time-based response differences.
- Technique / pattern: When the response body never echoes query output, fall back to an oracle: paired boolean payloads (AND 1=1 vs AND 1=2) to detect a content difference, or a sleep primitive to detect a timing difference. Once one parameter reliably flips the oracle, the rest of the schema is recoverable a bit at a time; confirm manually before automating.
- Takeaway: A silent response is not a safe response. Parameterize queries everywhere - blind SQLi is fully exploitable and is found by measuring behaviour, not by reading output.

### 2026-09-18 — SQL injection via JSON 'name' parameter in a subtitle search API (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #3257171](https://hackerone.com/reports/3257171)
- Type: SQL injection (JSON request body parameter)
- Summary: A POST endpoint at /er2_mrrp/api/v3/hft/subtitle/ took a JSON body of the form {"filters":{"name":"..."}} and concatenated the name value into a backend SQL query, allowing arbitrary query manipulation.
- Technique / pattern: Do not stop at query-string and form parameters: feed JSON body fields to the same injection tests. Save the raw request and point an automated tester at the specific JSON field (sqlmap's request-file mode with the parameter marked) so the tool preserves content-type and body structure rather than flattening it into a form post.
- Takeaway: Parameters reachable only through a JSON API get tested less and parameterised less. Treat every deserialised field as untrusted input to the query builder.

### 2026-09-18 — SQL injection in the Structure plugin's delete_channels() function (CVE-2025-59473) (ExpressionEngine) — n/a
- Source: [HackerOne #3249794](https://hackerone.com/reports/3249794)
- Type: SQL injection (admin-authenticated, second-order privilege escalation)
- Summary: The Structure plugin for ExpressionEngine passed the channel_ids parameter straight into a SQL query inside delete_channels() with no sanitisation, so a lower-privileged administrator could run arbitrary database operations beyond their assigned rights.
- Technique / pattern: Audit plugin and extension code rather than only core: third-party add-ons reuse the CMS's database layer but often skip its binding helpers. Grep for query-building calls that interpolate variables, then trace which roles can reach the enclosing action - an injection behind an admin gate still matters when the panel has multiple privilege tiers.
- Takeaway: 'Requires admin access' is a severity reducer, not a dismissal. In multi-tier admin panels, injection is a privilege-escalation primitive between tiers.

### 2026-09-18 — SQL injection in CVE Discovery search (HackerOne) — n/a
- Source: [HackerOne #1893800](https://hackerone.com/reports/1893800)
- Type: SQL injection (blind/boolean-based, unauthenticated)
- Summary: The CVE Discovery search feature split search terms on whitespace and inserted them into a query without further sanitisation, letting an unauthenticated visitor inject SQL that reached the analytics database holding report, team and asset data.
- Technique / pattern: When a search box splits input on whitespace, use inline comment sequences (/**/) as the space character so the payload survives tokenisation. Confirm blind injection with a boolean pair: a condition that is always true returns the normal result set, and the same payload with an always-false condition returns nothing - the differential proves query control without extracting data.
- Takeaway: Input transformation before the query - splitting, trimming, normalising - is not sanitisation, and the boolean-differential test proves injection without ever touching the data.

### 2026-09-18 — SQL injection on intensedebate.com (Automattic) — n/a
- Source: [HackerOne #1069561](https://hackerone.com/reports/1069561)
- Type: SQL injection (GET parameter, error/union based)
- Summary: The `acctid` parameter of `/js/importStatus.php` on intensedebate.com was concatenated into a query unsanitised; the reporter confirmed injection and enumerated the backing databases (including `heartbeat` and `id_comments`).
- Technique / pattern: Enumerate the small helper scripts a site loads from `/js/` and similar support paths — they are often older, outside the main framework, and take numeric identifiers; test each numeric parameter for injection rather than only the ones visible in forms.
- Takeaway: Legacy side-channel endpoints that the main application still loads are a persistent SQLi surface: the framework's parameter binding protects the routes it owns, not the standalone scripts left beside them.

### 2026-09-18 — Unauthenticated SQL injection in ImpressCMS leading to remote code execution (ImpressCMS) — n/a
- Source: [HackerOne #1081145](https://hackerone.com/reports/1081145)
- Type: SQL injection (unauthenticated, chained to RCE)
- Summary: An unauthenticated SQL injection in ImpressCMS was chained through to remote code execution, with the researcher publishing a full technical write-up of the injection and the escalation path.
- Technique / pattern: Audit an open-source CMS's pre-authentication request handlers for input that reaches a query, confirm injection, then escalate from SQL read/write primitives to code execution through the application's own file and configuration write paths.
- Takeaway: Pre-auth code paths are the highest-value audit target in open-source applications, and SQLi in a CMS rarely stops at data disclosure — it usually reaches code execution via the app's own writable surfaces.

### 2026-09-18 — Blind SQL injection on id.indrive.com (inDrive) — n/a
- Source: [HackerOne #2051931](https://hackerone.com/reports/2051931)
- Type: SQL injection (blind)
- Summary: A parameter on inDrive's identity domain `id.indrive.com` was injectable with no data returned directly in the response, so the vulnerability was confirmed and demonstrated blind.
- Technique / pattern: With no visible output, build a boolean or time-based oracle (paired true/false conditions, sleep payloads) to confirm injection, then infer data one condition at a time; automate only after the parameter is confirmed.
- Takeaway: Identity and auth subdomains (`id.`, `sso.`, `account.`) are prime SQLi targets and often sit outside the main application's hardening — and the absence of visible output is not the absence of injection.

### 2026-09-17 — SQL Injection Detection Bypass in AWS WAF Managed Rules (AWSManagedRulesSQLiRuleSet) (AWS VDP) — n/a
- Source: [HackerOne #3591725](https://hackerone.com/reports/3591725)
- Type: SQL injection (WAF detection bypass)
- Summary: The AWS managed SQLi ruleset could be evaded so that payloads matching known injection patterns passed through to the backing application without triggering the rule.
- Technique / pattern: Systematically mutate confirmed SQLi payloads — comment insertion, alternate encodings, whitespace and keyword substitution, and moving the payload between request parts — and measure which variants the managed ruleset fails to flag.
- Takeaway: A WAF is a speed bump, not a fix: test detection bypasses explicitly, and never accept managed rules as a substitute for parameterized queries.

### 2026-09-17 — SQL Injection when using FilteredRelation (Django) — n/a
- Source: [HackerOne #3292573](https://hackerone.com/reports/3292573)
- Type: SQL injection (ORM / framework-level)
- Summary: Django's ORM allowed untrusted input to reach generated SQL through the FilteredRelation construct, so an application passing user input into that API could be injected despite 'using the ORM'.
- Technique / pattern: Audit ORM code paths that build SQL fragments from names, keys or aliases rather than bound parameters — identifier positions are where ORMs historically leak injection, unlike value positions.
- Takeaway: ORMs only protect value positions; column, relation and alias arguments are frequently interpolated, so keep user input out of them and track framework CVEs.

### 2026-09-17 — CVE-2024-42005: SQL injection in QuerySet.values()/values_list() (Internet Bug Bounty) — n/a
- Source: [HackerOne #2646493](https://hackerone.com/reports/2646493)
- Type: SQL Injection (ORM / column-alias injection)
- Summary: Django's QuerySet.values() and values_list() were injectable on models with a JSONField — attacker-controlled JSON object keys passed as arguments were used to build column aliases, letting crafted keys break out into SQL.
- Technique / pattern: Supplied malicious JSON key names to values()/values_list() on a JSONField so the keys were interpolated into the query's column-alias SQL rather than parameterized.
- Takeaway: Injection isn't only raw SQL value strings — ORM method arguments (field names, aliases) can reach the query as identifiers unescaped; keep untrusted input out of identifiers and patch framework CVEs promptly.

### 2026-09-17 — Boolean-based blind SQL injection via the User-Agent header (U.S. Dept of Defense) — n/a
- Source: [hackerone #2599826](https://hackerone.com/reports/2599826)
- Type: SQL Injection (boolean-based blind, header-borne)
- Summary: A `.mil` endpoint passed the `User-Agent` request header into a SQL query unsanitized, allowing boolean-based blind SQL injection from a header most inputs validation overlooks.
- Technique / pattern: Inject boolean conditions into the `User-Agent` header and infer true/false from differing server responses to extract data blind — fuzz headers, not only URL/body parameters.
- Takeaway: Every request-controlled value — including headers like `User-Agent`/`X-Forwarded-For` that often reach logging/analytics queries — must be parameterized; attacker control is not limited to visible params.

### 2026-09-16 — SQL injection on admin.acronis.host development web service (Acronis) — n/a
- Source: [hackerone #923020](https://hackerone.com/reports/923020)
- Type: SQLi (SQL Injection)
- Summary: A SQL injection flaw existed on the `admin.acronis.host` development service; Acronis confirmed the affected dev system held no sensitive or real user data.
- Technique / pattern: Probe parameters on forgotten dev/admin subdomains with SQLi payloads (error-based / boolean / time-based) — non-production hosts often run with weaker input handling and stale code.
- Takeaway: Development and admin subdomains are in-scope attack surface; parameterize every query and keep dev hosts off the public internet.

### 2026-09-15 — SQL injection on a Sony website (Sony) — swag
- Source: [HackerOne #2256032](https://hackerone.com/reports/2256032)
- Type: SQL injection (error-based)
- Summary: A Sony website was vulnerable to error-based SQL injection; the researcher demonstrated impact by extracting database metadata (such as database names) with SQLMap.
- Technique / pattern: Find a parameter that reflects database errors, confirm injection manually, then use SQLMap to enumerate DBMS structure (databases/tables) as proof of exploitability.
- Takeaway: Error-based SQLi is still common on large estates; parameterize queries and suppress verbose DB errors, which both leak data and accelerate exploitation.

### 2026-09-15 — [www.zomato.com] Blind SQL injection via res_id (Zomato / Eternal) — n/a
- Source: [HackerOne #838855](https://hackerone.com/reports/838855)
- Type: SQL Injection (blind / time-based)
- Summary: The `res_id` parameter of the `/php/geto2banner` endpoint on www.zomato.com was concatenated into a backend SQL query, allowing blind SQL injection and inference-based data extraction.
- Technique / pattern: A non-obvious internal/AJAX endpoint parameter was tested with boolean and time-based payloads; induced delays confirmed a blind oracle, extractable value-by-value (or via sqlmap once confirmed).
- Takeaway: Fuzz internal/AJAX endpoint parameters, not just the visible UI; any id concatenated into SQL is a sink — use parameterized queries.

### 2026-09-14 — SQL injection via User-Agent header on labs.data.gov (GSA) — n/a
- Source: [HackerOne #297478](https://hackerone.com/reports/297478)
- Type: SQL injection (SQLi) — header-borne, blind/time-based
- Summary: The /dashboard/datagov/csv_to_json endpoint concatenated the incoming User-Agent header into a SQL query, making injection reachable purely through a request header.
- Technique / pattern: Fuzz headers (not only params) with time-based payloads — arithmetic and SLEEP/pg_sleep probes confirm blind SQLi without exfiltrating data, keeping the PoC in-scope and non-destructive.
- Takeaway: Any logged or stored request value — User-Agent, Referer, X-Forwarded-For — can reach a query sink; parameterize queries and treat all headers as untrusted input.


### 2026-09-12 — Union-based SQL injection in IntenseDebate comment history (Automattic) — n/a
- Source: [HackerOne #1046084](https://hackerone.com/reports/1046084)
- Type: SQL injection (union-based)
- Summary: The IntenseDebate endpoint `/commenthistory/<siteId>` was injectable through the site-ID path value, allowing UNION-based extraction of database contents after registering a site to obtain a valid ID.
- Technique / pattern: Injected SQL via a numeric path segment (not a query string), used UNION SELECT to align columns and exfiltrate data; a legitimate account/site was created first to reach the vulnerable authenticated feature.
- Takeaway: Path segments are user input too — parameterize every query and never assume values that look like internal IDs are safe.

### 2026-09-11 — www.drivegrab.com SQL injection (Grab) — n/a
- Source: [HackerOne #273946](https://hackerone.com/reports/273946)
- Type: SQL injection (third-party WordPress plugin)
- Summary: A SQL injection in the Formidable Forms plugin on the WordPress-based www.drivegrab.com granted read access to the WordPress database.
- Technique / pattern: Fingerprint the CMS and its plugins/versions, then fuzz a vulnerable plugin parameter with SQLi payloads (boolean/UNION) to read DB contents; verify with a minimal PoC before reporting.
- Takeaway: Third-party CMS plugins are a frequent SQLi sink — enumerate installed plugins and test their parameters, not just first-party code.

### 2026-09-10 — Blind SQL injection via User-Agent header (U.S. Dept of Defense) — n/a
- Source: [HackerOne #2597543](https://hackerone.com/reports/2597543)
- Type: SQL injection (blind, via HTTP header)
- Summary: A backend processed the `User-Agent` header into a SQL query unsafely, allowing blind SQL injection through that header and access to backend database data.
- Technique / pattern: Fuzz non-body inputs — `User-Agent`, `X-Forwarded-For`, `Referer` — with boolean/time payloads (e.g. `' AND SLEEP(5)--`); a response-time/behaviour delta confirms a header-borne sink, then automate with sqlmap targeting that header.
- Takeaway: Every request component that reaches a query is an injection surface, headers included; parameterize regardless of where the value came from.
