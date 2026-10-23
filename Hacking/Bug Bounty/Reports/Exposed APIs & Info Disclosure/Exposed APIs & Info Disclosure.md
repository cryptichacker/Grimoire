---
tags: [hacking, bug-bounty, info-disclosure, api, secrets, reports]
type: log
---

# Exposed APIs & Info Disclosure

## Up
- [[Reports]]

Disclosed **exposed API / sensitive information disclosure** reports — leaked API keys/secrets, PII returned by APIs, exposed config/debug endpoints, open `.git`/backups. See [[Pattern]] → Exposed APIs & Info Disclosure.

## Reports

### 2026-10-23 — Valid API keys recoverable from archived snapshots of API endpoints (Planet Labs) — n/a
- Source: [HackerOne #1639011](https://hackerone.com/reports/1639011)
- Type: Credential exposure via third-party web archives
- Summary: The reporter found sensitive API keys exposed through archived copies of Planet's API endpoints held by the Wayback Machine, and stated that some of the recovered keys were still valid and exploitable. The exposure came from historical snapshots cached by the Internet Archive's crawler rather than from the live site.
- Technique / pattern: Query `web.archive.org`'s CDX index for every URL ever recorded under a target domain, then grep the archived responses for key material and token patterns. Secrets that were removed from the live site months ago persist indefinitely in archives, so the live-site clean bill of health proves nothing.
- Takeaway: Rotation, not deletion, is the only remediation for a leaked credential. Removing a secret from the current response leaves it readable in third-party archives forever — and archives are an unauthenticated, in-scope-adjacent source worth checking on every engagement.

### 2026-10-23 — Flickr API key committed to a public GitHub repository (Mozilla) — n/a (out of scope, no bounty)
- Source: [HackerOne #1992261](https://hackerone.com/reports/1992261)
- Type: Hardcoded credential in a public source repository
- Summary: A researcher found a Flickr API key in a public Mozilla GitHub repository. Mozilla assessed that the key only retrieved public Flickr data, judged it to pose no security risk, revoked it anyway, resolved the report as out of scope for a bounty, and said similar reports would be closed as informative.
- Technique / pattern: Scanning an organization's public repositories and full commit history for key material is cheap and high-volume, but the value of a hit depends entirely on what the key authorizes. Before reporting, determine the key's scope by exercising a read-only call and establishing whether it reaches non-public data, a paid quota, or a writable resource.
- Takeaway: "A key was leaked" is not by itself a vulnerability. Triage turns on demonstrated capability — a credential limited to already-public data is informative, so spend the effort on scope evidence rather than on volume of findings.

### 2026-10-23 — Public exposure of personnel and stakeholder PII in documents on NASA subdomains (NASA Vulnerability Disclosure Program) — n/a
- Source: [Bugcrowd 7d10efbc](https://bugcrowd.com/disclosures/7d10efbc-a774-4b42-9f7d-cb76dfc19127/public-exposure-of-pii-of-nasa-meeting-registrations)
- Type: Sensitive information disclosure via publicly reachable documents (P4, resolved; disclosed 25 Sep 2025)
- Summary: The reporter identified multiple publicly accessible documents on NASA subdomains containing personally identifiable information of staff, collaborators and stakeholders, including names, email addresses, phone numbers and addresses. The files were not access-controlled, and the reporter states the work used only publicly available archival resources and manual inspection.
- Technique / pattern: Document-hunting across subdomains — search-engine `filetype:` queries, archived URL indexes, and open directory listings — surfaces attachments that were published for one small audience and never restricted. No intrusive testing is required, which keeps the activity comfortably inside most VDP rules of engagement.
- Takeaway: PII exposure findings are won by coverage and careful handling, not by exploitation. Report the locations and the categories of data, quantify the number of affected individuals, and never redistribute the files or the records themselves.

### 2026-10-23 — Sensitive information including contact data and infrastructure detail in public PDFs (NASA Vulnerability Disclosure Program) — n/a
- Source: [Bugcrowd d3bb7067](https://bugcrowd.com/disclosures/d3bb7067-4d21-4708-82ce-85c7169eb9e7/sensitive-information-disclosure)
- Type: Sensitive information disclosure in published documents (P3, informational; disclosed 18 Nov 2024)
- Summary: The reporter found publicly reachable PDF files exposing a combination of personal data — phone numbers, names and addresses — alongside a detailed diagram describing how a firewall was configured. The program rated it P3 and recorded it as an accepted business risk.
- Technique / pattern: Treat published documents as a reconnaissance corpus rather than only a privacy problem. Architecture diagrams, runbooks and network drawings inside PDFs name internal hostnames, product versions and security-control placement — material that shapes every later test against that estate.
- Takeaway: The P3 rating here reflects the architecture detail, not the contact data. When a document leak includes infrastructure design, argue the finding as attack-surface disclosure that enables targeting, which is consistently rated above a plain PII listing.

### 2026-10-22 — Endpoint revealing whether another researcher was invited to the same private program (Bugcrowd) — $900
- Source: [Bugcrowd 66b2fe17](https://bugcrowd.com/disclosures/66b2fe17-fd5d-4eb5-a3df-7a4f13b9b6ca/endpoint-is-showing-whether-the-researcher-has-been-in-particular-private-program-or-not-leads-to-sensitive-info-disclosure)
- Type: Exposed API / information disclosure (membership oracle) — P3
- Summary: An endpoint on Bugcrowd's own platform could be used to confirm whether a given researcher had been invited to a particular private program, turning a confidential membership list into something queryable from outside.
- Technique / pattern: Hunt for oracles rather than dumps — find a request whose response differs measurably depending on a fact you should not know, such as a lookup or autocomplete that only resolves for principals inside a scope. Repeating the query across identifiers reconstructs the hidden set one answer at a time.
- Takeaway: Confidentiality can be broken without any record ever being returned; a reliable yes/no is enough to rebuild a private roster. Endpoints that merely resolve identifiers need the same authorization as endpoints that return data, and differential responses, error-message variance and timing all count as disclosure.

### 2026-10-22 — Browser extension's geolocation spoofing disabled by a crafted page message (ExpressVPN) — $1,250
- Source: [Bugcrowd a3ece17b](https://bugcrowd.com/disclosures/a3ece17b-5da7-452d-a6dc-04398700c71d/expressvpn-browser-extension-html5-geolocation-spoof-bypass-client-physical-location-leak)
- Type: Sensitive data exposure / client location leak — P2
- Summary: The ExpressVPN browser extension spoofs HTML5 geolocation so sites see the VPN exit location. Its `gps.js` content script listened for messages while checking only that the event originated from the same window, so a page could send a crafted message that switched the spoofing off and exposed the user's real coordinates.
- Technique / pattern: Read the extension's content scripts and audit each `message` listener for sender validation — a same-window origin check does not authenticate the sender, because the page under test shares that window. Reachable through a site the user grants location permission to, or through an XSS on a site they already trust.
- Takeaway: Content scripts run alongside the page, so any `postMessage` handler they expose is attacker-reachable and must validate the sender, not just the origin. For privacy tooling the severity argument is about who relies on the guarantee — this report framed impact around users for whom a location leak is a physical-safety matter.

### 2026-10-22 — WordPress REST API exposing administrator account details to anonymous users (Yoti) — n/a
- Source: [HackerOne #727870](https://hackerone.com/reports/727870)
- Type: Exposed API / information disclosure (user enumeration)
- Summary: The WordPress install behind Yoti's marketing site left a REST API endpoint reachable without authentication that returned administrator account information, usernames included, to any anonymous requester.
- Technique / pattern: On a WordPress target, request the users collection of the REST API directly instead of scraping author pages — the route is enabled by default, is not linked from the site, and returns the account list as JSON. Harvested admin usernames then feed credential stuffing and targeted phishing.
- Takeaway: Default-on API surfaces of a CMS are part of the attack surface even when the real application lives elsewhere; a marketing subdomain shares the brand's trust without sharing its security review. Enumeration findings read strongest when reported as the first stage of a named follow-on attack rather than as a standalone leak.

### 2026-10-22 — Debug endpoint returning environment variables including AWS S3 credentials (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1720278](https://hackerone.com/reports/1720278)
- Type: Sensitive information disclosure (debug / introspection endpoint)
- Summary: An endpoint at `/api/getEnvVars` on a DoD host returned the process environment. AWS S3 credentials, `PATH` values, and internal IP addresses and ports were all included in the response.
- Technique / pattern: Brute-force the API namespace with diagnostic verbs and nouns — `env`, `getEnvVars`, `config`, `debug`, `health`, `actuator`, `info` — rather than only the business endpoints a crawler can see. Introspection routes are written for a developer, carry no authorization, and are rarely stripped from the production build.
- Takeaway: Environment-variable dumps are among the highest-yield single responses in an engagement, since they hand over cloud credentials and internal network layout at once. Any route that reflects server state back to the caller is a secret-disclosure sink, and the internal hostnames and ports are worth as much as the keys for the next stage.

### 2026-10-21 — New GraphQL team_profile field exposes restricted report metadata (HackerOne) — n/a
- Source: [HackerOne #389600](https://hackerone.com/reports/389600)
- Type: Exposed API — GraphQL field-level information disclosure
- Summary: The reporter noticed a newly added `team_profile` field on the GraphQL schema and queried it to retrieve data that should have been restricted, including the latest report marked serious and the reports a team had received over the previous three months.
- Technique / pattern: Diff the GraphQL schema over time. Introspection (or watching the fields the front end starts requesting after a release) surfaces newly added types and fields, and a new field is the most likely place for an authorization check to be missing because the resolver is newer than the policy layer around it. Query the field directly as a low-privilege or unrelated user rather than through the UI that normally gates it.
- Takeaway: In GraphQL, authorization lives per resolver, so schema additions are the recurring regression point. A habit of re-running introspection after each deploy turns release notes into a finding pipeline.

### 2026-10-21 — Hidden profile feedback still retrievable through a single GraphQL POST (HackerOne) — n/a
- Source: [HackerOne #1264725](https://hackerone.com/reports/1264725)
- Type: Exposed API — client-side-only privacy control
- Summary: After the reporter unchecked the "Show this blurb on my profile" option for their feedback settings, the feedback was no longer displayed in the UI but could still be retrieved with a single POST request to the `/graphql` endpoint. The privacy preference suppressed rendering without filtering the data at the resolver.
- Technique / pattern: For every privacy or visibility toggle, flip it off and then re-issue the underlying API call directly instead of reloading the page. Capture the request while the content is visible, toggle the setting, and replay — identical data in the response proves the control is presentational only. Profile, feedback, activity and "who can see this" settings are the highest-yield places to try it.
- Takeaway: A visibility setting that is enforced in the client is not enforced. Every user-facing privacy control should be tested by replaying the API request it is supposed to govern, which is a repeatable check rather than a lucky find.

### 2026-10-21 — Path traversal on a NASA host (NASA Vulnerability Disclosure Program) — n/a (P1)
- Source: [Bugcrowd #c209da9f](https://bugcrowd.com/disclosures/c209da9f-651f-4efc-a61d-1c12c1379f95/path-traversal)
- Type: Exposed files — path traversal
- Summary: A path traversal reported to the NASA VDP, rated P1 and resolved, disclosed 25 Oct 2024. The reporter chose a restricted disclosure and explicitly withheld the technical details, so the public page records the class, the priority and the outcome only.
- Technique / pattern: Traversal still earns top priority when it reaches server-side files, so the surfaces worth testing are the ones that take a filename rather than a URL: download and export handlers, template or locale selectors, archive extraction, and image or document resizers. Test encoded and double-encoded separators, absolute paths, and null or extension-suffix tricks, and judge impact by what the reachable file contains — configuration, credentials, source, or keys.
- Takeaway: A P1 traversal on a public-sector estate is a reminder that file-path parameters deserve the same scrutiny as database parameters. It also illustrates restricted disclosure: the finding is public, the exploit is not, which is a legitimate choice for a sensitive target.

### 2026-10-20 — Server fully exposed with database credentials (Essity) — n/a
- Source: [HackerOne #3584078](https://hackerone.com/reports/3584078)
- Type: Sensitive information disclosure (directory listing exposing credentials)
- Summary: A directory listing on an Essity server exposed its contents, including database credentials. Triaged Critical, classified `Information Exposure Through Directory Listing`; the program redacted the hostnames in the public report.
- Technique / pattern: Directory indexing remains one of the highest-yield checks — request the bare directory for every path you discover, and when a listing appears, read the config, backup and environment files it reveals instead of stopping at the listing. The listing is where the finding starts, not where it ends.
- Takeaway: Disable autoindex, keep configuration and backup files out of web roots, and treat any credential that has sat in a public listing as compromised and rotate it.

### 2026-10-20 — Sensitive information disclosure (CVE-2025-61138) in Qlik Sense Dev Hub (US EPA VDP) — n/a (P3)
- Source: [Bugcrowd df96dba0](https://bugcrowd.com/disclosures/df96dba0-779d-49fc-84f0-0f353490b6aa/sensitive-information-disclosure-cve-2025-61138-in-qlik-sense-dev-hub)
- Type: Sensitive information disclosure (known CVE in a third-party product)
- Summary: A Qlik Sense Dev Hub instance inside EPA scope was affected by CVE-2025-61138 and disclosed sensitive information. Resolved, rated P3.
- Technique / pattern: Product-and-version reconnaissance — fingerprint the off-the-shelf platforms in scope (BI tools, wikis, CI dashboards), map each to recently published CVEs, and verify the advisory's non-destructive indicator against the deployed version. Almost no custom exploitation is needed, and coverage is wide because these products hide in the long tail of a large estate.
- Takeaway: Inventory third-party web applications per host and patch them on the vendor's clock; a current CVE feed scoped to the products actually deployed is a dependable source of valid findings on either side of the fence.

### 2026-10-20 — Public exposure of staging STAC API, test datasets and Keycloak configuration on the NASA VEDA platform (NASA VDP) — n/a (P5, informational)
- Source: [Bugcrowd 06a4cc5b](https://bugcrowd.com/disclosures/06a4cc5b-ce68-49a7-a262-aca70316ba5b/medium-public-exposure-of-staging-stac-api-test-datasets-keycloak-configuration-on-nasa-veda-platform)
- Type: Exposed API / configuration disclosure (unauthenticated staging environment)
- Summary: A publicly reachable staging environment in the NASA VEDA ecosystem served STAC API endpoints, test datasets and authentication configuration metadata without authentication, enabling unauthenticated reconnaissance of staging systems and internal architecture. No data modification or privileged access was demonstrated; NASA closed it as expected behaviour at P5.
- Technique / pattern: Enumerate environment-prefixed hostnames (`staging-`, `dev-`, `test-`) for each production service, then pull the well-known descriptor endpoints — an API's own catalog or OpenAPI document and the identity provider's `/.well-known/openid-configuration` — which between them map endpoints, realms and client ids without a single authenticated call.
- Takeaway: Staging deployments inherit production's architecture, so place them behind authentication or network restriction. Note also that a reconnaissance-only finding is frequently accepted as risk, so chain it to a concrete impact before reporting.

### 2026-10-20 — Internal IP address and verbose server version disclosure via 404 error pages (NASA VDP, globe.gov) — n/a (P5)
- Source: [Bugcrowd ab3fd857](https://bugcrowd.com/disclosures/ab3fd857-030a-417c-9a34-62e2d6aba64b/internal-ip-address-and-verbose-server-version-disclosure-via-404-error-pages)
- Type: Sensitive information disclosure (verbose error / client-side config object)
- Summary: Invalid or modified API requests to `globe.gov` returned a verbose error response whose `Liferay.ThemeDisplay` JavaScript object revealed an internal private IP address along with detailed Apache Tomcat and Liferay module versions. Resolved, rated P5 informational.
- Technique / pattern: Deliberately malform API requests and read the error page as data — including the JavaScript configuration objects a CMS injects client-side (`Liferay.ThemeDisplay` and equivalents). Those objects often hold internal hostnames, paths and exact module versions the rendered page never shows.
- Takeaway: Strip internal infrastructure detail from client-side config objects and disable verbose error output in production; the version strings these leak are what turn a later CVE sweep into a precise hit.

### 2026-10-18 — Blind SSRF escalated to unauthenticated arbitrary file read in NASA MMGIS via inline GDAL VRT (NASA VDP) — n/a (P1)
- Source: [Bugcrowd #4712dd0d](https://bugcrowd.com/disclosures/4712dd0d-e913-412d-98e6-6a5de00fa5d2/escalated-f8f98208-efd6-4dfd-984c-2d8fae11ac19-informative-report-from-blind-ssrf-to-arbitrary-file-read)
- Type: Exposed API / information disclosure — unvalidated path into a library's dataset opener, escalated from blind SSRF
- Summary: Three MMGIS endpoints on a NASA JPL host passed a client-supplied path straight into GDAL's `gdal.Open()` with no authentication or validation. Because GDAL accepts inline VRT (Virtual Raster) XML in place of a filename, and a `VRTRawRasterBand` can map a local file's raw bytes onto pixel values, an attacker could use `POST /api/utils/getprofile` — which returns pixel values in its response — to read arbitrary local files as an anonymous user. Rated P1 and fixed.
- Technique / pattern: The escalation is the lesson: a finding first triaged as informative blind SSRF became a critical read primitive once the researcher looked at what the *library* accepts rather than what the endpoint documents. Data-processing libraries routinely treat their filename argument as a mini-language — GDAL VRT and `/vsicurl/`, ImageMagick MSL and MVG, FFmpeg concat and HLS playlists, XML external entities, `libxslt` document functions, `pandas.read_*` URLs. Pair one of those with an endpoint that returns processed values to the caller and the blind primitive becomes an oracle. The hunt: identify the processing library and version, read its documentation for descriptor or indirection formats, then find the response field that carries data outward.
- Takeaway: Never pass user input into a library's open or load entry point; resolve it to a server-side identifier and map that to a path. Disable the library's indirection features explicitly — for GDAL, restrict the allowed drivers and `GDAL_DISABLE_READDIR_ON_OPEN` — and authenticate utility endpoints. When triaging, re-examine "informative" blind findings for a response field that can be turned into a read channel.

### 2026-10-18 — Publicly accessible conference attendee directory exposes extensive personnel contact data (NASA VDP) — n/a (P5)
- Source: [Bugcrowd #68f5c5dd](https://bugcrowd.com/disclosures/68f5c5dd-ea6c-440c-a6d3-03580982d0a0/publicly-accessible-project-documentation-exposes-extensive-project-personnel-contact-information)
- Type: Information disclosure — legacy document aggregating personal and organizational contact details, served without authentication
- Summary: A project-documentation file reachable without authentication contained a conference attendee directory listing participants' names, organizational affiliations, mailing addresses, phone numbers, fax numbers and e-mail addresses. Reported to NASA's VDP, rated P5, accepted and fixed.
- Technique / pattern: Aggregated contact lists are worth more than their severity rating suggests, because they are the raw material for targeted phishing and support-desk impersonation against the organization whose staff they name. Find them by searching indexed and unindexed document paths for the formats these artefacts take — attendee and participant lists, org charts, distribution lists, meeting minutes, proposal cover sheets, review-panel rosters — as `.pdf`, `.xlsx`, `.docx` and `.csv` under conference, project, programme and archive directories. Directory listings, sitemaps and web archives locate files the site no longer links.
- Takeaway: Rate an exposure by what it enables downstream, not only by whether each field is individually sensitive: a roster is a ready-made target list. Give document repositories a retention and review cycle, unpublish event material once the event is over, and classify documents containing personal data at upload rather than auditing them years later.

### 2026-10-18 — Sensitive, proprietary and predecisional documents served publicly from NASA.gov domains (NASA VDP) — n/a (P5)
- Source: [Bugcrowd #ea4f630b](https://bugcrowd.com/disclosures/ea4f630b-f23a-4bc2-b3ac-51d3e684077a/public-disclosure-of-sensitive-proprietary-and-predecisional-documents-on-nasa-gov-domains)
- Type: Information disclosure — internal documents reachable on public web domains
- Summary: Documents described as sensitive, proprietary and predecisional were found publicly reachable on NASA.gov domains. The disclosure is brief and does not enumerate the files; the report was accepted and fixed, rated P5.
- Technique / pattern: Markings such as "predecisional", "proprietary", "source selection sensitive", "pre-decisional draft", "FOUO" and "internal use only" are the useful pivot, because the documents label themselves. Search-engine operators scoped to the estate, plus content-grep over crawled documents, surface them quickly, and the same markings let the report argue sensitivity from the document's own classification rather than the researcher's judgement of the contents. Handle carefully: cite the marking, the path and the title, keep copies out of the report, and do not redistribute.
- Takeaway: Classification markings inside a file are the cheapest signal for both attacker and defender. Build the marking list into automated scanning of what the public estate actually serves, and gate document publication on a classification check rather than on the author's choice of upload directory.

### 2026-10-18 — Publicly editable OPAG collaboration spreadsheet linked from science.nasa.gov (NASA VDP) — n/a (P5)
- Source: [Bugcrowd #18f49357](https://bugcrowd.com/disclosures/18f49357-f70c-4f60-b2d1-e43a0dcddf2f/data-integrity-risk-pii-exposure-publicly-editable-opag-collaboration-document-google-sheets-via-science-nasa-gov)
- Type: Information disclosure / data-integrity risk — third-party collaboration document shared with anyone-can-edit permissions
- Summary: A Google Sheets collaboration document for the OPAG group, linked from `science.nasa.gov`, was configured so that anyone with the link could edit it, raising both a personal-data exposure and a data-integrity concern. Reported to NASA's VDP, rated P5 and accepted as a business risk.
- Technique / pattern: Externally hosted documents linked from an in-scope site are a surface the target's own hardening does not cover. Extract every link to a document-sharing provider from the estate's pages, then record for each file whether it is view-only or editable, whether comment or suggestion mode is open, whether revision history is visible, and whether the content names people. Checking *write* access is the step usually skipped, and an editable document linked from an authoritative page is also a content-injection channel, since visitors trust what the official page points to. Confirm the permission from the sharing metadata rather than by modifying anyone's data.
- Takeaway: Shared-document permissions belong in the same review as application authorization. Default collaboration links to view-only, keep personal data out of link-shared files entirely, and audit an external document's permissions as part of publishing the page that links to it.

### 2026-10-16 — Private subforum content broadcast in full through the BuddyPress activity REST API (WordPress / bbPress) — n/a
- Source: [HackerOne #4053546](https://hackerone.com/reports/4053546)
- Type: Exposed API / information disclosure — unauthenticated leak of private content
- Summary: bbPress's BuddyPress integration computed the `hide_sitewide` flag for new topic and reply activity by checking only the posting forum's own status, never its ancestors. A publish-status subforum nested under a private or hidden parent therefore had its topics broadcast in full — title, forum name, permalink and raw body — to the public BuddyPress activity stream, even though the topic's own permalink correctly returned 404 to the same viewer.
- Technique / pattern: Two views of one object disagreed about its visibility. The researcher confirmed the inconsistency directly: posting into the nested subforum, verifying the permalink 404s anonymously, then reading the content back unauthenticated from `GET /wp-json/buddypress/v1/activity`. Hierarchical permissions that are evaluated per-node rather than over the ancestor chain are the recurring root cause.
- Takeaway: When content lives in a tree, the effective visibility is the most restrictive value along the whole path to the root — compute it that way, once. And always re-check "private" content through every secondary surface: REST collections, activity feeds, search indexes, sitemaps and RSS, which commonly bypass the canonical route's access control.

### 2026-10-16 — Ticket-trick attack on an unverified support-portal e-mail grants access to an organization's workspaces (Rockstar Games) — n/a
- Source: [HackerOne #1534465](https://hackerone.com/reports/1534465)
- Type: Information disclosure / account takeover via support-desk e-mail ingestion ("ticket trick")
- Summary: A support portal did not verify ownership of an e-mail address before granting access, allowing the classic ticket-trick attack. By registering on the portal with an address at the organization's support domain, the attacker could read inbound mail delivered to that address as support tickets — including magic login links requested for the organization's accounts on third-party sites — and then sign in as the organization.
- Technique / pattern: The support desk is an unintended mail reader. The attacker signs up with an address the help desk itself receives at, logs into the portal dashboard, then triggers a login-link or password-reset flow for that address on an external SaaS product. The resulting e-mail becomes a ticket visible in the attacker's dashboard, converting "can receive mail at a domain address" into authentication at every site that trusts e-mail possession.
- Takeaway: Any system that both ingests e-mail and lets users self-register must verify address ownership before showing message contents, and should refuse to create accounts on its own ingest domains. Conversely, e-mail possession is a weak authenticator — magic links and resets for high-value accounts need a second factor.

### 2026-10-16 — Unauthenticated Gravity Forms REST route on nasa.gov exposes internal form metadata (NASA VDP) — n/a
- Source: [Bugcrowd #623ea694](https://bugcrowd.com/disclosures/623ea694-3569-4725-ab60-cfdc48c19427/unauthenticated-nasa-gravity-forms-api-endpoint-exposes-sensitive-internal-metadata)
- Type: Exposed API / information disclosure — missing authentication on a custom REST route (P5, informational)
- Summary: A custom, unauthenticated REST API endpoint on nasa.gov exposed internal metadata from NASA's Gravity Forms installation, disclosing labels and IDs for several internal forms covering HR processes, grants and mission-specific workflows such as Artemis II. No submissions or personal data were exposed, but the internal process names and form IDs provide reconnaissance value and social-engineering material. Root cause was missing authentication and permission callbacks on a custom route.
- Technique / pattern: On WordPress targets, enumerate `/wp-json/` and walk the route index for custom namespaces added by plugins or site code; those are where `permission_callback` is most often left as `__return_true`. Form-builder plugins are especially productive because their metadata routes enumerate every form on the site, including unlinked internal ones, giving a map of workflows that are not reachable from the public navigation.
- Takeaway: Every custom REST route needs an explicit permission callback — a framework that defaults to open turns "we just expose metadata" into a published inventory of internal processes. Metadata-only leaks still deserve reporting: form and endpoint names are the reconnaissance input for targeted phishing.

### 2026-10-16 — `developers.json` in a public NASA-related repository exposes privileged users' names and corporate e-mails (NASA VDP) — n/a
- Source: [Bugcrowd #c1753cc6](https://bugcrowd.com/disclosures/c1753cc6-7193-4792-b99a-9e4a3ee6d0a4/public-exposure-of-sensitive-information-of-highly-privileged-users-in-the-developers-json-file)
- Type: Information disclosure — PII in a committed configuration file (P3)
- Summary: A public NASA-related GitHub repository contained a `developers.json` file listing personally identifiable information for several developers, including full names and corporate e-mail addresses. The exposure of a roster of highly privileged users opened the door to social engineering, spear phishing and sender spoofing.
- Technique / pattern: Source-repository reconnaissance aimed at config and manifest files rather than secrets: `developers.json`, `CODEOWNERS`, `AUTHORS`, `.mailmap`, `package.json` maintainer blocks, CI variable files and commit author metadata. These are rarely scrubbed because they are not credentials, yet together they map who holds privileged access — the prerequisite for a credible phishing or help-desk impersonation attempt against the people who can grant it.
- Takeaway: Treat the identities of privileged staff as sensitive, not merely as contact information. Keep contributor rosters out of published artifacts (or reduce them to role aliases), and audit repositories for files naming individuals alongside their access. A leak with no credential in it can still be the first step of a full compromise.

### 2026-10-15 — NASA-hosted training PDF exposes hardcoded credentials and insecure credential distribution (NASA VDP) — n/a (P5)
- Source: [Bugcrowd #4d8f1660](https://bugcrowd.com/disclosures/4d8f1660-3fb1-4f96-8607-d35b96341d68/nasa-hosted-training-pdf-exposes-hardcoded-credentials-and-insecure-credential-distribution)
- Type: Credential exposure in published documentation
- Summary: A recently uploaded training PDF on a public NASA subdomain contained a live username and password together with staff email addresses; the report also flagged the practice of distributing demo credentials inside published documents.
- Technique / pattern: Watch a target's document directories and "recently uploaded" listings, then extract PDF/DOCX text and grep it for credential keywords such as `password`, `username`, `login` and `api key` — training and onboarding material is written for internal use and reviewed least.
- Takeaway: Treat published documentation as code for secret-scanning purposes; demo credentials must be disposable and scoped, and files need a sanitisation pass before release.

### 2026-10-15 — Full path disclosure and verbose error messages on webdisks.jpl.nasa.gov (NASA VDP) — n/a (P5)
- Source: [Bugcrowd #aaf069ae](https://bugcrowd.com/disclosures/aaf069ae-4bde-4b3e-8647-225b0bff1b92/full-path-disclosure-and-verbose-error-messages-on-webdisks-jpl-nasa-gov)
- Type: Information disclosure through verbose errors (full path disclosure)
- Summary: The application failed to handle unexpected input gracefully and emitted verbose PHP errors that printed internal server directory paths, which an attacker could use to map the host's filesystem layout.
- Technique / pattern: Feed each parameter a type the code cannot handle (an array where a string is expected, an empty value, oversized input) and read the resulting error output; full paths recovered this way feed later local-file-inclusion, log-poisoning and arbitrary-file-write attempts.
- Takeaway: Low-severity verbose errors are reconnaissance fuel for higher-severity file-path bugs — turn off error display in production and return generic messages.

### 2026-10-15 — CVE-2026-86409: stale SigV4 headers sent unintended (curl) — n/a
- Source: [HackerOne #3973135](https://hackerone.com/reports/3973135)
- Type: Credential leakage through an incomplete option reset
- Summary: Setting `CURLOPT_AWS_SIGV4` to `NULL` returned success but left AWS SigV4 authentication enabled, because the handler in `lib/setopt.c` cleared the parameter string without removing the `CURLAUTH_AWS_SIGV4` flag from the handle's auth settings. Later `curl_easy_perform()` calls on the same handle therefore still sent an `AWS4-HMAC-SHA256` `Authorization` header, including to a URL set after the reset.
- Technique / pattern: For each option that configures a credential, check that clearing it also clears the derived state (auth-method flags, cached headers), then re-run the same handle against a different host and watch whether the credential still goes out.
- Takeaway: Reusable client handles need a complete reset path; a partially cleared credential setting is how secrets end up sent to an unintended origin.

### 2026-10-15 — Exposing HackerOne users' personally identifiable information by abusing a sandbox with swag reward enabled (HackerOne) — n/a
- Source: [HackerOne #357576](https://hackerone.com/reports/357576)
- Type: Sensitive information disclosure (PII surfaced by a reward workflow)
- Summary: With swag rewards enabled on a sandbox program, a hacker's shipping details were sent to the program owner and shown on the program's swag page, so a sandbox owner could reward swag for a test report from an invited team member and read that member's name, address and phone number.
- Technique / pattern: Exercise a platform's own test or sandbox features with every optional reward type switched on, then compare the data each role can see against what the feature actually needs; shipping, invoicing and payout flows are where PII crosses role boundaries.
- Takeaway: Sandbox and demo modes inherit production data paths — PII collected for fulfilment should never be surfaced to a role that only needs to confirm the reward.

### 2026-10-12 — Unauthenticated disclosure of draft/private/pending post titles and IDs via Secure Custom Fields nopriv AJAX (WordPress) — n/a
- Source: [HackerOne #3893632](https://hackerone.com/reports/3893632)
- Type: Information disclosure (unauthenticated AJAX)
- Summary: Draft, private and pending post titles and ids could be disclosed to unauthenticated users through a Secure Custom Fields `nopriv` AJAX field handler.
- Technique / pattern: A `wp_ajax_nopriv_` action was registered for a field-fetching endpoint without a capability check, so logged-out requests returned metadata for non-published posts.
- Takeaway: `nopriv` AJAX handlers run for anonymous users by design — gate them with capability/nonce checks, or do not register them for data that is not public.

### 2026-10-12 — Unauthenticated disclosure of private and hidden bbPress forum content via the WordPress core REST single-item endpoint (WordPress) — n/a
- Source: [HackerOne #3801107](https://hackerone.com/reports/3801107)
- Type: Information disclosure / improper access control (REST API)
- Summary: Private and hidden bbPress forum topics and replies — stored with public post status — could be read by unauthenticated users via the WordPress core REST single-item endpoint, because forum visibility was enforced only through query filters.
- Technique / pattern: The plugin registered post types with REST exposure while relying on list-query filters for access control; a direct single-item REST request by sequential post id bypassed those filters and returned full restricted content.
- Takeaway: Access control must live on the object, not only on list queries — a single-item fetch by id will bypass filters applied only to listing endpoints.

### 2026-10-12 — Live password-reset token reaches Datadog RUM, and the same session records the account it unlocks (HackerOne) — $200
- Source: [HackerOne #4071670](https://hackerone.com/reports/4071670)
- Type: Information disclosure — secret captured by telemetry (RUM)
- Summary: A live, single-use password-reset token was captured by Datadog Real User Monitoring because it travelled in the URL query string, and the same RUM session also recorded the account it unlocked; sampling was 100% with no URL sanitization.
- Technique / pattern: Sensitive tokens placed in a URL are swept up by front-end monitoring SDKs; a prior fix had redacted one telemetry sink, but an alternate RUM sink remained uncovered.
- Takeaway: Keep secrets out of URLs entirely, and apply redaction across every telemetry/monitoring sink — patching one analytics pipeline does not cover the others.

### 2026-10-12 — Leaked MySQL and SMTP credentials exposed on GitHub (program undisclosed) — n/a
- Source: [Bugcrowd #a2d35d68](https://bugcrowd.com/disclosures/a2d35d68-f10c-440a-8cee-bc1a6fdb3343/leaked-credentials-mysql-and-smpt-credentials-exposed-on-github)
- Type: Sensitive information disclosure — leaked credentials in public repo
- Summary: Live MySQL and SMTP credentials were exposed in a public GitHub repository, handing out usable database and mail credentials.
- Technique / pattern: Secrets were committed to a public repo (config/source) and were discoverable via code search; the credentials were still valid and reusable.
- Takeaway: Scan repos for secrets before and after pushing, rotate anything committed, and keep credentials in a secrets manager — public commit history is permanently searchable.

### 2026-10-09 — Sensitive user information disclosure via the userUuid parameter on an RPC endpoint (Uber) — bounty awarded (amount not public)
- Source: [HackerOne #542340](https://hackerone.com/reports/542340)
- Type: Exposed API / sensitive information disclosure (authorization bypass)
- Summary: The `getConsentScreenDetails` RPC on `bonjour.uber.com/marketplace/_rpc` accepted a `userUuid` POST parameter and returned that user's personal data along with their mobile auth token, which could then be used to call Uber's mobile APIs as that rider or driver. Rated High (CVSS 8.5); submitted 2019-04-19 and disclosed 2019-09-09.
- Technique / pattern: The researcher probed an internal-style RPC gateway where each call names a method and its parameters, and found a method that trusted a caller-supplied user identifier instead of the session. Only a low-privilege account was needed and no victim interaction; supplying another user's UUID returned their record. The leaked field was not merely PII but a usable credential, escalating disclosure into full account access.
- Takeaway: RPC and `_rpc`-style gateways multiply attack surface because every method is a separate authorization decision and they are rarely enumerated by scanners. Always read a disclosure response for tokens, not just for PII — a leaked auth token turns a read bug into account takeover.

### 2026-10-09 — Full access to the inDrive Jira panel via an exposed API token (inDrive) — bounty awarded (amount not public)
- Source: [HackerOne #1785145](https://hackerone.com/reports/1785145)
- Type: Exposed secret in public source code / information disclosure
- Summary: A public GitHub repository belonging to inDrive contained a source file with a hardcoded Jira API token (plus a second token in a commented-out line). Used as HTTP Basic auth against inDrive's Atlassian instance, it returned issue data and granted access to projects, issues, comments and accounts. Rated Critical; submitted 2022-11-27 and disclosed 2023-06-28.
- Technique / pattern: Organisation-scoped source reconnaissance: identify repositories plausibly owned by the target, then grep committed files for credential patterns, including commented-out and historical values that developers assume are inert. The researcher verified the token's live validity with a single authenticated request — the site normally redirects to an Atlassian OAuth login, so the token bypassed the access control entirely.
- Takeaway: A secret in a public repository is only a finding once you prove it still authenticates, so always demonstrate impact with one minimal authenticated request. Commented-out credentials are as live as uncommented ones, and internal trackers hold the roadmap, infrastructure detail and PII that make them high-value targets.

### 2026-10-09 — Path traversal in a Tomcat server via proxy path-normalisation mismatch (LY Corporation) — n/a
- Source: [HackerOne #988877](https://hackerone.com/reports/988877)
- Type: Information disclosure via path traversal (proxy/app-server normalisation mismatch)
- Summary: A misconfiguration between the reverse proxy and the Tomcat application server let `..;/` in the URL path reach restricted paths, including an administrator page, and return their resources. Weakness recorded as Information Disclosure, no bounty; submitted 2020-09-23 and disclosed 2023-03-28. The flaw arose while the team was correcting an incorrect path in proxy settings.
- Technique / pattern: The two tiers disagree about what a path segment means. The proxy does not recognise `..;/` as traversal, so its path-based access rules do not match and it forwards the request unchanged; Tomcat then treats `;` as the start of a path parameter and resolves the segment as a directory traversal, landing outside the intended prefix. Probing `..;/` (and relatives such as `/.;/` or encoded separators) against proxy-protected prefixes tests for this disagreement.
- Takeaway: Access control enforced on a URL prefix at one tier is only as strong as the agreement between tiers on how that URL parses. Wherever nginx or another proxy guards a path while Tomcat serves it, test the normalisation gap — and note that a routine proxy-config edit is exactly when the gap gets introduced.

### 2026-10-09 — Internal network scanning through SSRF in the NASA WorldWind API (NASA VDP) — n/a
- Source: [Bugcrowd #6296825c](https://bugcrowd.com/disclosures/6296825c-7c02-4771-80d8-9d1496331981/internal-scan-through-ssrf-in-nasa-worldwind-api)
- Type: Exposed API / SSRF-driven internal information disclosure
- Summary: The WorldWind WMS `GetMap` endpoint fetched and processed the URL supplied in its `SLD` parameter without validation, letting an attacker force outbound requests to internal addresses such as `127.0.0.1` as well as external hosts. Rated P3 and marked accepted and fixed; disclosed 2025-12-17.
- Technique / pattern: The researcher used a standards-defined parameter as the SSRF primitive: OGC WMS allows an `SLD` styling document to be referenced by URL, so a conforming request is also a server-side fetch. Information came back through error messages rather than response bodies — MapServer's errors revealed which ports were open, which internal services were reachable, how far interaction with them went, and whether given file paths existed, which is enough for internal reconnaissance and enumeration.
- Takeaway: Standardised APIs document their own SSRF sinks; read the specification for any parameter that takes a URL or external reference. When the fetched content is not echoed back, differential error messages are the oracle — a blind SSRF still yields a port scanner and a path-existence check.

### 2026-10-09 — GraphQL team object exposes private-program participant counts and details (HackerOne) — n/a
- Source: [HackerOne #380317](https://hackerone.com/reports/380317)
- Type: Excessive data exposure via GraphQL
- Summary: After a release that added user ordering, the GraphQL API began returning the number of participants in **private** programs to users who had never been invited, and the reporter noted the participants' details were reachable the same way.
- Technique / pattern: Pull the GraphQL schema (introspection, or the shipped `.graphql`/persisted-query files when introspection is off) and diff it after each deploy. Then request every field on a sensitive type — here `team` — as an unauthorized user. Aggregate fields such as counts, `totalCount`, `edges` and connection metadata are often left unguarded even when the underlying list is protected.
- Takeaway: GraphQL authorization has to be per-field, not per-endpoint. New features that add sorting, counting or pagination to an existing type silently widen what that type will answer, so re-test old types after unrelated releases.

### 2026-10-09 — OrderListInitial GraphQL operation leaks order details to Customer-only staff (Shopify) — n/a
- Source: [HackerOne #882412](https://hackerone.com/reports/882412)
- Type: Excessive data exposure / broken function-level authorization in GraphQL
- Summary: The `OrderListInitial` operation on `POST /admin/internal/web/graphql/core` returned more order information than a staff account holding only the "Customer" permission was supposed to see.
- Technique / pattern: Create a staff account with the narrowest permission set the product offers, drive the admin UI, and capture the internal GraphQL operations it issues — then replay each operation from the low-privilege session and diff the response against the admin's. Internal/undocumented admin GraphQL endpoints (note the `internal` path segment) are where permission scoping most often diverges from the public API.
- Takeaway: Granular staff-permission models need the permission checked on every field the resolver returns, not on the operation name. Low-privilege-account replay against internal admin APIs is one of the highest-yield tests on any multi-role SaaS product.

### 2026-10-09 — Bearer token exposed on a NASA subdomain leading to account takeover (NASA VDP (Bugcrowd)) — n/a (P3)
- Source: [Bugcrowd #5c6b25ac](https://bugcrowd.com/disclosures/5c6b25ac-4bed-4a28-b5f1-2eac292378b8/bearer-token-disclosed-in-a-nasa-subdomain-leads-to-account-takeover)
- Type: Sensitive credential disclosure → account takeover
- Summary: Researcher `yatanim` found a bearer token exposed on a NASA subdomain and showed it could be used to take over a user account. Accepted and fixed, rated P3; disclosed 2024-11-13. The public page records the impact but not the discovery method.
- Technique / pattern: Hunt for tokens in the places applications accidentally publish them: front-end bundles and source maps, `Authorization` values hard-coded in JavaScript, API responses that echo tokens, error pages and stack traces, archived pages, and build/config artefacts left in the webroot. Then verify the token is actually live by calling a benign authenticated endpoint such as a self/whoami route — enough to prove impact without touching third-party data.
- Takeaway: A leaked bearer token is a credential, so treat discovery and reporting with the same care: prove validity with the minimum possible call, never pivot into another person's data, and report immediately. Defenders should keep tokens out of anything served to the browser and support fast revocation.

### 2026-10-09 — Unrestricted Google Docs exposing NASA documentation (NASA VDP (Bugcrowd)) — n/a (P3)
- Source: [Bugcrowd #c2e88322](https://bugcrowd.com/disclosures/c2e88322-057a-4ac5-99a2-06cae386f0a7/google-docs-no-restriction-vulnerability-in-nasa-documentation)
- Type: Broken access control on third-party document hosting / information disclosure
- Summary: Researcher `0xSaikat` reported NASA documentation hosted in Google Docs with no access restriction, making the documents readable by anyone holding or discovering the link. Accepted and fixed, rated P3; disclosed 2024-11-04.
- Technique / pattern: Third-party document hosting is part of the attack surface: search for `docs.google.com`, `drive.google.com`, Sharepoint, Notion, Airtable and Confluence links that reference the target, both in search-engine indexes and in links embedded on the target's own pages, then check whether "anyone with the link" or "anyone on the internet" sharing is set.
- Takeaway: Data leaves the perimeter through SaaS sharing defaults far more often than through application bugs. Inventory externally-hosted documents and enforce domain-restricted sharing — "unlisted link" is not an access control.

### 2026-10-07 — Email disclosure via dummy report invitations (HackerOne) — n/a
- Source: [HackerOne #2032716](https://hackerone.com/reports/2032716)
- Type: Information disclosure — PII via a collaboration feature
- Summary: The report-collaboration invite flow revealed the private email addresses of invited users: an attacker could file a throwaway report, add target users as collaborators, and read back email addresses those users had chosen to keep private.
- Technique / pattern: The leak was in the response of a legitimate feature, not in a broken endpoint. Inviting a user put them in a pending state and the pending-invite representation carried their email, so the attacker-controlled report became a lookup oracle mapping usernames to private emails.
- Takeaway: Features that let you reference other users by handle — invites, mentions, sharing, transfers, team add — are PII oracles whenever their response echoes back more of the referenced user's record than the UI displays. Read the raw JSON of such responses rather than the rendered page.

### 2026-10-07 — Exposed .git/config file (BlockDev Sp. Z o.o) — n/a
- Source: [HackerOne #686805](https://hackerone.com/reports/686805)
- Type: Information disclosure — exposed version-control metadata
- Summary: A WordPress theme directory on a production blog served its version-control metadata over HTTP: `https://blog.makerdao.com/wp-content/themes/makerDAO/.git/config` was directly retrievable, exposing repository details.
- Technique / pattern: The deploy copied the working tree, `.git` directory included, and the web server had no rule blocking dot-directories. The check is a direct request for `.git/config` under each application root — and under *nested* roots like a theme or plugin folder, which is where this one was, since scanners often only test the site root.
- Takeaway: `.git/config` is the cheap confirmation; the real risk is that `.git/HEAD`, the index and the object store can allow reconstruction of the source, including secrets in history. Test dot-paths (`.git`, `.svn`, `.env`, `.DS_Store`) at every directory level of a CMS install, not just at `/`.

### 2026-10-07 — Source code disclosure of SQL database (NASA VDP) — n/a (P4)
- Source: [Bugcrowd #0f426f40](https://bugcrowd.com/disclosures/0f426f40-1b22-404a-87c6-516a9f8b31e4/source-code-disclosure-of-sql-database)
- Type: Information disclosure — exposed source / database code
- Summary: Source code relating to a SQL database was exposed on a NASA web asset. The finding was accepted and remediated. (The public disclosure records the class and resolution but not the specific path.)
- Technique / pattern: Source disclosure on a web asset typically comes from one of a small set of causes: a file extension the server does not map to a handler, an editor or backup suffix (`.php~`, `.bak`, `.old`, `.inc`, `.txt` appended), or a path served as a static asset. Enumerating those suffixes against known script paths is the usual discovery route.
- Takeaway: Exposed database-layer source is valuable twice over: it may contain connection credentials directly, and it reveals the query construction and schema, which turns later injection testing from blind guessing into targeted work. Always grep recovered source for credentials and for unparameterised query building.

### 2026-10-07 — Private information exposed through GraphQL search aggregates (HackerOne) — n/a
- Source: [HackerOne #1838329](https://hackerone.com/reports/1838329)
- Type: Information disclosure via GraphQL aggregation arguments
- Summary: The `search` and `opportunities_search` root fields accepted an `aggs` argument whose returned buckets were not filtered by authorization, so a query such as `opportunities_search(query:{}, aggs:{results:{terms:{field:"handle"}}})` returned bucket keys for private program handles even while `me` resolved to null.
- Technique / pattern: Aggregations, facets, counts and sort orders are a read channel distinct from the records themselves — once a search or GraphQL API exposes a `terms`, `aggs` or `facet` parameter, enumerate it against every field that could hold private values, because the per-record authorization filter usually runs on hits and not on the aggregation pipeline.
- Takeaway: Apply the same authorization scope to aggregation queries as to record reads, and restrict which fields may be aggregated at all.

### 2026-10-07 — H1514: access to non-public store data by pivoting with GraphQL queries (Shopify) — $1,500
- Source: [HackerOne #423388](https://hackerone.com/reports/423388)
- Type: Broken function-level authorization in a GraphQL admin API
- Summary: A collaborator account holding only Apps permission could use the GraphiQL app to read data from permission areas it did not hold — store locations and addresses via `locations`, inventory via `inventoryLevel(inventoryItemId)`, marketing spend via `marketingActivities`, and publication records including an app's `apiKey` — with the API returning results rather than access-denied.
- Technique / pattern: Take the thinnest possible permission set, then walk the schema field by field against the vendor's own permission documentation and record which root queries answer anyway; a GraphQL explorer shipped as a first-party app is the ideal client, and a silent result instead of an authorization error is the signal that a resolver has no permission check.
- Takeaway: Authorization in GraphQL belongs in every resolver, mapped to the documented permission scopes — and a low-privilege role is the right starting point for a schema audit.

### 2026-10-07 — NASA FTP credentials exposed in a publicly hosted CORAL mission PDF (NASA VDP) — n/a (P3, letter of appreciation)
- Source: [Bugcrowd #d1847e21](https://bugcrowd.com/disclosures/d1847e21-df10-4001-99ef-9ba9a3459141/public-exposure-of-nasa-ftp-credentials-in-coral-document-pdf-hosted-on-google-docs)
- Type: Sensitive information disclosure — plaintext credentials in public documentation
- Summary: A publicly reachable PDF for NASA's CORAL mission, hosted on Google Docs, contained FTP URLs with embedded user and password pairs in plaintext referencing the `avng.jpl.nasa.gov` domain; NASA removed public access, revoked the credentials and audited for unauthorized use.
- Technique / pattern: Credential hunting is not only a code-and-bucket exercise — passive OSINT across document hosts (Google Docs and Drive, slide decks, conference proceedings, research portals) for a target's mission and project names regularly surfaces `ftp://user:pass@host` style URLs in operational guides written for internal readers; report the finding without authenticating to the service.
- Takeaway: Treat documentation as a secret store nobody rotates, and keep credentials out of URLs, where they survive in cached and shared copies long after the document is withdrawn.

### 2026-10-07 — Sensitive PII reachable through an unprotected route on nightsky.jpl.nasa.gov (NASA VDP) — n/a (P3)
- Source: [Bugcrowd #f5a1b2b0](https://bugcrowd.com/disclosures/f5a1b2b0-5aaa-4afd-8f83-abcf4bf4bd2c/pii-disclosure-exposure-of-phone-numbers)
- Type: Broken access control leading to PII disclosure
- Summary: Fuzzing hidden directories on a NASA subdomain, combined with search-engine reconnaissance, surfaced a route that served documents containing phone numbers, home addresses and handwritten signatures instead of returning an unauthorized response.
- Technique / pattern: Pair content discovery (directory and file fuzzing with a wordlist tuned to the stack) with search-engine and archive reconnaissance for the same host — indexed filenames reveal which naming conventions to extend, and an unauthenticated `200` on a path the interface never links is the finding; stop at proof rather than collecting the PII in bulk.
- Takeaway: Authorization must be enforced on the document route itself — the absence of a link is not an access control, because unlinked paths are discoverable and indexable.

### 2026-10-07 — Confidential third-party document publicly retrievable on a NASA domain (NASA VDP) — n/a (P5, informational)
- Source: [Bugcrowd #c0e34a88](https://bugcrowd.com/disclosures/c0e34a88-8e9c-4c02-8342-221c08d7d67c/public-access-to-confidential-document-on-nasa-website)
- Type: Information disclosure — a confidentially marked file served without authentication
- Summary: A file marked "Confidential" by a third party, covering proprietary carbon-dioxide-removal technology, was retrievable from NASA's domain with no authentication; the program noted it may have originated in public conference distribution but updated the site to remove it.
- Technique / pattern: Treat a document's own markings as the evidence — crawl a target's file paths for `pdf`, `pptx` and `xlsx` assets, extract their text, and grep for "confidential", "proprietary", "internal use only" and distribution statements, which turns a pile of public files into a ranked list; impact rests on whether the content was ever meant to be public, which a program may dispute.
- Takeaway: Hosting a third party's confidential material is still a disclosure issue even when the host does not own it — review uploaded assets for classification markings before publishing them.

### 2026-10-06 — Internal meeting chat logs published on a public NASA site (NASA VDP) — n/a
- Source: [Bugcrowd #9f1283c7](https://bugcrowd.com/disclosures/9f1283c7-4d47-44b7-8e81-b6e96515ab52/sensitive-information-disclosure-nasa-meeting-internal-chat-logs-exposed)
- Type: Sensitive information disclosure (P5, accepted business risk)
- Summary: A file holding the internal chat log of an October 2020 Mars Exploration Program Analysis Group meeting — discussion between officials, presenters and external participants — was reachable without authentication on a public NASA host, and was removed after the report.
- Technique / pattern: Found passively with no exploitation: enumerate the document-bearing paths of a program's public sites (meeting, agenda, minutes, archive and presentation directories) and look for artifacts generated as a by-product of the event rather than published deliberately — chat transcripts, attendee lists, raw question-and-answer exports.
- Takeaway: Event platforms emit side artifacts nobody reviews for publication, so the by-products of a workflow need the same review as its intended output.

### 2026-10-06 — Public exposure of NASA personnel contact information on a JPL server (NASA VDP) — n/a
- Source: [Bugcrowd #0329715c](https://bugcrowd.com/disclosures/0329715c-e5f3-4e6f-a78e-63ba2e8df286/public-exposure-of-nasa-personnel-contact-information-on-jpl-server)
- Type: PII / information disclosure (P5, accepted business risk)
- Summary: An unauthenticated file on a JPL server listed staff full names, email addresses and phone numbers; NASA assessed the material as intended for public release, so it closed as an accepted business risk.
- Technique / pattern: Passive reconnaissance of a host's static files for roster-shaped data — directory, contacts, participants, distribution list. The reporting lesson is that aggregated contact data is rated by whether it was *meant* to be public, so check the organisation's own public directory first and argue impact from the aggregation and its social-engineering value rather than from the presence of a name.
- Takeaway: Published-by-design data is not a finding; a credible PII report needs a field that is not already public, or a volume and structure that makes targeting materially easier.

### 2026-10-06 — Directory listing exposes space-weather data files on cdaw.gsfc.nasa.gov (NASA VDP) — n/a
- Source: [Bugcrowd #13ee821c](https://bugcrowd.com/disclosures/13ee821c-1083-4acd-b4fc-057f03120300/information-disclosure-https-cdaw-gsfc-nasa-gov)
- Type: Information disclosure / directory listing (P5, accepted business risk)
- Summary: A directory index on the CDAW host listed files describing space-weather events; NASA's response confirmed the listing is the intended delivery mechanism for researchers and the public, and the report closed informational.
- Technique / pattern: A calibration case for directory-index findings — before reporting, read what the index actually serves and whether the host exists to distribute data. Scientific, mirror and contributor hosts publish indexes on purpose, so the finding only exists when the listed files were not meant to be public.
- Takeaway: Rate a directory listing by the sensitivity of what it indexes rather than by the presence of the index; sampling one file answers the question before the report is written.

### 2026-10-06 — Publicly accessible NASA internal shipping document (NASA VDP) — n/a
- Source: [Bugcrowd #c51e2180](https://bugcrowd.com/disclosures/c51e2180-2a9e-46a7-9ded-ecf8930cec36/publicly-accessible-nasa-internal-shipping-document-exposing-sensitive-information)
- Type: Sensitive information disclosure (P3)
- Summary: An internal shipping document describing NASA logistics was reachable from the public internet without authentication, disclosing operational detail about internal shipping processes.
- Technique / pattern: Logistics and procurement artifacts — shipping manifests, packing lists, bills of lading, asset inventories — carry names, internal addresses, site codes and hardware descriptions, and are typically dropped into a web root for a one-off transfer and never removed. Search a host by document convention and filetype for these operational file classes, not only for credentials.
- Takeaway: Operational paperwork is sensitive data, and one-off file transfers need an expiry and a review step, because a web root is permanent storage.

### 2026-10-05 — 1.7M password hashes and no-auth-required access tokens exposed (Wemo) — n/a
- Source: [Bugcrowd #01e09745](https://bugcrowd.com/disclosures/01e09745-59b7-4aea-a7ce-8a0b39837e6b/1-7m-password-hashes-no-auth-required-access-tokens)
- Type: Exposed API / mass sensitive data disclosure through unauthenticated endpoints
- Summary: Several Wemo API endpoints required no authentication and returned, variously, every user's username and unsalted SHA256 password hash (base64-encoded), authentication tokens for all users, and live Salesforce backend tokens — enough for direct account and device takeover. Rated P1; recorded as accepted business risk.
- Technique / pattern: On IoT and mobile backends, enumerate API routes from the app's own traffic and strip the auth header from each to find the ones that answer anonymously; then read the response schema for credential material (hashes, tokens, keys) rather than only obvious PII. Unsalted hashes and long-lived tokens turn a single leaky endpoint into full compromise.
- Takeaway: Every API route needs its own authentication and authorization; one unauthenticated endpoint returning credential material is catastrophic, and salting plus short-lived tokens limit the blast radius when one does leak.

### 2026-10-05 — Sensitive data exposure: restricted NASA document publicly accessible (NASA) — n/a
- Source: [Bugcrowd #b564c052](https://bugcrowd.com/disclosures/b564c052-0e21-448e-9c38-0da86e3b9810/sensitive-data-exposure-restricted-nasa-document-publicly-accessible)
- Type: Sensitive information disclosure / missing access control on a document
- Summary: A restricted NASA document — International Space Station Heat Rejection Subsystem specifications meant only for contractors and authorised US-government personnel — was reachable publicly with no authentication. Rated P1; NASA removed public access.
- Technique / pattern: Documents carrying distribution markings ("restricted", "ITAR", "for official use only") are high-value even when found through passive means such as search-engine indexing or directory guessing; the finding's severity comes from the content's classification, not from exploit complexity. Search for marking strings and predictable document paths on government and contractor hosts.
- Takeaway: Access control has to be enforced at the object itself; a sensitive file placed at a web-reachable path with no auth check is disclosed to anyone who finds the URL, and marked content makes that a serious exposure.

### 2026-10-05 — Leak of usernames from a private website via browser local storage (NASA) — n/a
- Source: [Bugcrowd #0bfcbdb2](https://bugcrowd.com/disclosures/0bfcbdb2-1faf-4714-97cf-597d17d58ca2/leak-of-usernames-from-a-private-website)
- Type: Information disclosure through client-side storage and client-reachable endpoints
- Summary: A site stored data in browser local storage and exposed endpoints from which the usernames of private, non-public users could be read. Rated P3, accepted and fixed.
- Technique / pattern: Read the application's client-side JavaScript and inspect `localStorage`/`sessionStorage` and the endpoints the front end calls; data placed there is fully attacker-readable, and front-end code often references internal or private-user routes the server does not actually restrict.
- Takeaway: The client is an untrusted environment — anything in local storage or referenced in front-end code is visible to the user. Keep private identifiers out of the client and enforce authorization on every endpoint the JavaScript can reach.

### 2026-10-01 — Akismet API keys are exposed by authentication method (Automattic) — n/a (bounty awarded, amount not disclosed)
- Source: [HackerOne #1736846](https://hackerone.com/reports/1736846)
- Type: Sensitive credential exposure through transport placement
- Summary: Akismet clients authenticated by putting the API key into the request hostname in the form `api-key.rest.akismet.com`, so the key travelled in plaintext DNS queries and could be logged by any resolver or network observer; Automattic changed the default to send the key in the request body.
- Technique / pattern: Trace where a secret appears in every layer of a request — hostname, path, query string, `Referer`, server logs — rather than only the body; a key placed in a subdomain label is resolvable metadata that TLS does not protect.
- Takeaway: Credentials must never sit in a DNS-visible position; when reviewing an integration, check the transport placement of the secret, because "it is HTTPS" only covers what is inside the TLS tunnel.

### 2026-10-01 — Git repository found (Grab) — n/a (bounty awarded, amount not disclosed)
- Source: [HackerOne #248693](https://hackerone.com/reports/248693)
- Type: Exposed `.git` directory leading to source and credential disclosure
- Summary: A WordPress theme directory on a Grab host served its Git metadata at `/wp-content/themes/.git/config`, revealing the private GitHub remote and allowing retrieval of tracked files including Ansible inventories, `host_vars` for development and production, and an SSH private key at `ansible/roles/common/files/id_rsa`.
- Technique / pattern: Request `.git/config` on each web root and on theme and plugin paths; when it resolves, read the tracked-file list and pull the high-value paths directly (`host_vars`, `provision.yml`, key material) instead of only noting that the directory is exposed.
- Takeaway: Deploy by export rather than by checking out a working copy — and escalate an exposed `.git` finding by enumerating what it actually contains, because infrastructure-as-code trees routinely carry keys and production variables.

### 2026-10-01 — Exposed NASA/JPL Credentials and Emails via Public Pastebin Leak (NASA VDP) — n/a
- Source: [Bugcrowd disclosure e4c2e5e0](https://bugcrowd.com/disclosures/e4c2e5e0-b90e-4d60-ae82-bc6c3d88b34e/exposed-nasa-jpl-credentials-and-emails-via-public-pastebin-leak)
- Type: Credential exposure on a third-party paste service
- Summary: Internal NASA and JPL email addresses, hashed passwords and internal environment markers — consistent with a misconfigured MediaWiki instance or its backups — were published on Pastebin and discoverable through search engines without authentication; the paste was removed after verification.
- Technique / pattern: Search public paste and gist services for organization-specific markers (mail domains, internal hostnames, configuration key names) rather than scanning only the target's own infrastructure, because the leak often sits on a platform the organization does not monitor.
- Takeaway: The attack surface includes places the organization does not own — periodic paste-site and code-sharing sweeps belong in a recon routine, and what turns up there is in scope as credential exposure.

### 2026-10-01 — Systemic PII Disclosure (Personal Emails) via CKAN API Endpoints (NASA VDP) — n/a (P5)
- Source: [Bugcrowd disclosure 67d9b0ae](https://bugcrowd.com/disclosures/67d9b0ae-995c-4cf1-a548-b9ef6310709c/systemic-pii-disclosure-personal-emails-via-ckan-api-endpoints)
- Type: Information disclosure via an over-returning catalog API
- Summary: NASA's open-data portal exposed the `maintainer_email` field through its CKAN API across many datasets, returning individual employee and contractor addresses instead of a generic support contact; NASA accepted it as a data-redaction improvement.
- Technique / pattern: Query the platform's package or dataset listing API and aggregate a single field across every record (`maintainer_email`, `author_email`, nested contact objects) — a value that looks harmless per record becomes a harvestable directory at scale, and documenting that breadth is what establishes impact.
- Takeaway: Open-data and catalog APIs leak people as a side effect of metadata; read the full field set the API returns rather than what the web UI chooses to display.

### 2026-09-30 — Alert Subscribers API Returns Subscriber Emails to Low-Privileged Users (SingleStore) — n/a
- Source: [HackerOne #3353000](https://hackerone.com/reports/3353000)
- Type: Exposed API / information disclosure through missing role check (triaged as privilege escalation)
- Summary: In SingleStore Helios, users holding a low-privileged role could call the Alert Subscribers API endpoint and retrieve the email addresses and alert-severity preferences of every notification subscriber, despite having no authorization to view that page. Resolved by restricting the endpoint.
- Technique / pattern: Notification, alerting and subscription settings are administrative screens that the UI hides by role while the backing endpoint checks only that the caller is authenticated. Enumerate the calls a privileged account's admin screens make, then replay each one from the lowest role the product offers — the response body is the finding, since subscriber lists pair verified corporate addresses with an interest signal that is directly usable for targeted phishing.
- Takeaway: Hiding an admin screen is not authorizing its API. Rate a disclosure by what the leaked fields enable rather than by their apparent sensitivity: an email list scoped to "people who receive production alerts" is a social-engineering target list, not just contact data.


### 2026-09-30 — Publicly Accessible CDN Endpoint Exposing XML Object Metadata Including ETag (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #3346375](https://hackerone.com/reports/3346375)
- Type: Information disclosure — object listing and metadata exposure
- Summary: A CDN URL returned a raw XML listing of stored objects including `Key`, `LastModified`, `Size`, `StorageClass` and `ETag` for each one. The files themselves remained access-controlled, but filenames, timestamps, sizes and content hashes were world-readable.
- Technique / pattern: Request the bucket or CDN root rather than a known object path and look for an S3-style XML listing. `ETag` is the interesting field: a single-part S3 upload sets it to the object's MD5, so an exposed value lets an attacker confirm that a file matches a known copy elsewhere, spot duplicates and versions, and correlate content across systems and past leaks — a multipart upload produces a non-MD5 marker instead.
- Takeaway: Object listing is a separate permission from object read, so deny it explicitly. Treat exposed filenames, timestamps and hashes as reconnaissance and fingerprinting material even when downloads themselves are blocked.

### 2026-09-30 — Unauthenticated Metrics Endpoint Exposes Internal Grafana and Infrastructure Data (NASA VDP) — n/a
- Source: [Bugcrowd #a2dbc42c](https://bugcrowd.com/disclosures/a2dbc42c-f4e5-4af4-97e0-0ce9c49f14be/unauthenticated-metrics-endpoint-exposes-sensitive-internal-grafana-nasa-infrastructure-data)
- Type: Exposed API / information disclosure — observability endpoint (P4)
- Summary: A publicly reachable Prometheus-style metrics endpoint exposed internal Grafana and InfluxDB telemetry: user, admin and organization counts, authentication activity statistics, Grafana build details (branch, revision, edition, version, Go version), plugin usage, database connection details, and host architecture and resource metrics. Accepted and resolved.
- Technique / pattern: Sweep every discovered host for the standard observability paths — `/metrics`, `/debug/vars`, `/actuator/prometheus`, `/-/metrics` — because exposition-format text is rarely reviewed for sensitivity. The payoff is twofold: the build and version labels give an exact fingerprint for selecting known CVEs, and the gauge names themselves enumerate users, admins and organizations without touching any application endpoint.
- Takeaway: Metrics and health endpoints are production attack surface. Bind them to an internal interface or require authentication, and keep build, version and plugin labels out of anything internet-facing.

### 2026-09-30 — Unauthenticated Public Download of an Internal Mobile Device Inventory XLSX (NASA VDP) — n/a
- Source: [Bugcrowd #cec21217](https://bugcrowd.com/disclosures/cec21217-102d-41e8-a14e-50779f002334/unauthenticated-public-download-of-nasa-internal-mobile-device-inventory-xlsx-exposing-pii-and-operational-asset-data)
- Type: Information disclosure — unauthenticated document download (P3)
- Summary: An internal mobile-device inventory spreadsheet containing PII alongside operational asset data was downloadable from a NASA host without authentication. Reported through the VDP and resolved after coordinated disclosure.
- Technique / pattern: Office documents on an organization's web hosts are found by crawler- and search-driven discovery rather than by application testing: filetype-scoped search queries, directory listings, `sitemap.xml`, and cached index entries regularly surface inventories, rosters and asset registers that were uploaded but never meant to be linked. Search-engine caches also keep them reachable after the link is removed.
- Takeaway: Treat any spreadsheet or PDF sitting on a web host as published data. An inventory that pairs staff identities with asset identifiers is high-value for follow-on targeting, so it needs authentication or removal plus a cache-eviction request, not merely an unlinked path.


### 2026-09-30 — Sensitive and PII-bearing documents indexed by search engines on globe.gov (NASA VDP) — n/a
- Source: [Bugcrowd #a47a5fe7](https://bugcrowd.com/disclosures/a47a5fe7-0c03-4d77-9a3f-fed114dcffd0/exposure-of-sensitive-information-that-contain-sensitive-markings-and-pii-information-via-search-engine-indexing)
- Type: Sensitive information disclosure via search-engine indexing
- Summary: The `/documents/` path on the globe.gov domain was not excluded from indexing, so applications containing personally identifiable information and files carrying restricted handling markings were retrievable through ordinary search queries.
- Technique / pattern: Run site-scoped queries against a document directory (`site:target inurl:/documents/ filetype:pdf`) before probing the application itself — indexed content needs no exploitation and evidences the exposure with a link the triager can open.
- Takeaway: Access control, not `robots.txt`, is what keeps documents private; indexable directories should also carry `X-Robots-Tag: noindex`, and anything already indexed must be treated as copied.

### 2026-09-30 — Confidential PDF with employee PII found via Google dorking (Department of the Interior VDP) — n/a
- Source: [Bugcrowd #ab094307](https://bugcrowd.com/disclosures/ab094307-76cf-4235-876f-ee34b306ed30/google-dork-leaded-information-confidential-in-pdf)
- Type: Sensitive data exposure / disclosure of secrets (P3)
- Summary: A PDF marked confidential and containing employee names, signatures, dates and security-contract details was publicly reachable at an ordinary web URL and surfaced by an indexed search, requiring no authentication or technical exploitation.
- Technique / pattern: Combine a domain filter with sensitivity keywords and a file-type filter — for example `site:example.gov ("confidential" OR "employee only" OR "internal") filetype:pdf` — so the documents' own handling markings do the triage work for you.
- Takeaway: Files that label themselves confidential are the highest-signal OSINT target; publishing workflows need a gate that reads classification markings before a document reaches a public path.

### 2026-09-30 — Unauthenticated access to SWEHB admin pages leaking internal scheduling and PII (NASA VDP) — n/a
- Source: [Bugcrowd #f425b701](https://bugcrowd.com/disclosures/f425b701-318e-4fd6-ba9e-4710130a8ca7/unauthenticated-access-to-nasa-swehb-admin-pages)
- Type: Broken access control / information disclosure
- Summary: Admin pages on the SWE handbook domain were viewable without authentication and disclosed internal scheduling information together with a lead developer's personal data.
- Technique / pattern: The pages were located with a plain indexed search (`site:nasa.gov "blog"`) rather than by brute force — the crawler has often already walked an admin surface that a wordlist would miss, so search the index before running a directory scan.
- Takeaway: Admin routes need a server-side authentication check and a `noindex` policy; being unlinked from the main navigation is not access control.

### 2026-09-30 — NASA/USGS personnel contact details exposed in public Earthdata metadata (NASA VDP) — n/a
- Source: [Bugcrowd #a5cd0ef0](https://bugcrowd.com/disclosures/a5cd0ef0-84ad-44db-a5cf-ceb9ae1c5d40/critical-personal-data-leak-exposed-sensitive-contact-information-of-nasa-usgs-personnel)
- Type: Sensitive information disclosure in API / catalogue metadata
- Summary: Publicly accessible metadata on NASA's Earthdata platform carried personnel contact information — phone numbers and email addresses — with no access restriction or protection.
- Technique / pattern: Pull the raw records a data-catalogue API returns rather than the rendered page, and grep the JSON or XML for contact, owner, maintainer and point-of-contact fields. Catalogue metadata is routinely published wholesale because it is thought of as description rather than data.
- Takeaway: Metadata is part of the API response and needs the same PII review as the primary data — strip or generalise contact fields before publication.

### 2026-09-29 — Publicly accessible `.git` directory on a staging host (X / xAI) — n/a
- Source: [HackerOne #218465](https://hackerone.com/reports/218465)
- Type: Sensitive information disclosure — exposed version-control metadata
- Summary: A staging engineering host served its repository metadata over the web; requesting `/.git/logs/refs/heads/master` returned commit history, and with the object store reachable the source tree and anything committed alongside it could be reconstructed.
- Technique / pattern: Probe `/.git/HEAD` and `/.git/logs/refs/heads/<branch>` on every host, staging included — a valid response usually means the object store is fetchable and the working tree can be rebuilt offline from it.
- Takeaway: Deploying by `git pull` leaves the metadata inside the web root by default, and staging hosts (less monitored, same codebase) are where that survives longest.

### 2026-09-29 — Slack and Google API tokens recovered from an Electron app published on GitHub (Grab) — n/a
- Source: [HackerOne #397527](https://hackerone.com/reports/397527)
- Type: Sensitive information disclosure — secrets embedded in a distributed binary
- Summary: A public GitHub repository's releases page hosted an Electron application; downloading and reverse-engineering the package revealed a Slack access token and a Google API token, the former reaching the organisation's Slack channels.
- Technique / pattern: Do not stop at a repository's source tree — pull the release artefacts too. Electron apps bundle their application JavaScript in an `app.asar` archive that unpacks with standard tooling, and any secret compiled into a desktop build ships to everyone who installs it.
- Takeaway: Anything embedded in a client users download is public by construction, so release assets deserve the same secret scanning as commits.

### 2026-09-29 — LDAP credentials committed in a public repository's testing dotenv file (Acronis) — n/a
- Source: [HackerOne #1004412](https://hackerone.com/reports/1004412)
- Type: Sensitive information disclosure — credentials in version control
- Summary: A `.env.testing` file in a public GitHub repository carried LDAP directory settings including the domain, base distinguished name and administrative credentials. The reporter advised confirming whether the credentials were live, removing the file and rotating the passwords; the submission was ultimately self-closed.
- Technique / pattern: Search public code for dotenv variants beyond the plain `.env` — `.env.testing`, `.env.example`, `.env.local` — since they are excluded by `.gitignore` patterns far less often, and LDAP keys such as `LDAP_BASE_DN` or `LDAP_USERNAME` make distinctive search terms.
- Takeaway: A committed credential is only a finding when it is both current and in scope; establish both before reporting or the report closes without impact.

### 2026-09-29 — Directory listing exposing server contents (NASA VDP) — n/a
- Source: [Bugcrowd #85c40ad7](https://bugcrowd.com/disclosures/85c40ad7-66f7-45c6-9ac8-79af052b7b19/information-exposure-through-directory-listing-vulnerability)
- Type: Sensitive information disclosure — directory listing (P3, resolved)
- Summary: A web server was configured to render directory indexes instead of an index page, exposing the file layout and the files themselves to unauthenticated visitors. The program accepted and resolved it.
- Technique / pattern: Request the parent path of any asset URL already in hand — `/assets/`, `/uploads/`, `/backup/` — and repeat one level up; autoindex enabled on a single directory usually means the whole virtual host inherits the setting.
- Takeaway: Directory listings turn every unlinked file on a host into a discoverable one, which is why they rate above the informational bucket when real files sit behind them.

### 2026-09-29 — Directory listing exposing Drupal installation and development files (Department of the Interior) — n/a
- Source: [Bugcrowd #df47543c](https://bugcrowd.com/disclosures/df47543c-b6e9-4e6d-a5b0-b1d5c94ab9c9/directory-listing-exposes-drupel-installation-and-development-files)
- Type: Sensitive information disclosure — exposed installation and development files (P3, resolved)
- Summary: Directory enumeration found publicly browsable Drupal installation and development files, revealing platform version detail and leftover artefacts that should never have shipped to production.
- Technique / pattern: For a known CMS, request its standard install and metadata paths — `/CHANGELOG.txt`, `/install.php`, `/core/`, `/sites/default/files/` — since they pin the exact version and frequently sit in a directory that lists.
- Takeaway: Leftover installer and development files are both a version oracle and a deployment-hygiene signal; where they survive, other pre-production artefacts usually do too.

### 2026-09-29 — Sensitive NASA material surfaced through search-engine indexing (NASA VDP) — n/a
- Source: [Bugcrowd #714c0fd7](https://bugcrowd.com/disclosures/714c0fd7-439e-4a1d-9729-7368f0a6b321/critical-information-disclosure-vulnerability-via-google-dorking-on-nasa-servers)
- Type: Sensitive information disclosure — search-engine indexing (P5, accepted risk)
- Summary: Search operators surfaced restricted-looking material indexed from NASA servers, but the program could not reproduce the access — triage received a `404` — and closed the report as an accepted business risk.
- Technique / pattern: Targeted dorking with `site:` plus file-type and path operators finds content that was never linked yet remained crawlable; capture the live response at the moment of discovery, because an indexed copy outlives the file.
- Takeaway: A cached search result is not proof of current exposure — evidence has to be captured against the live host or the report dies at triage.

### 2026-09-28 — Unauthenticated disclosure of draft and private posts via Secure Custom Fields nopriv AJAX handlers (WordPress) — n/a
- Source: [HackerOne #3893632](https://hackerone.com/reports/3893632)
- Type: Exposed API / improper access control
- Summary: Secure Custom Fields 6.9.2 registered `wp_ajax_nopriv_` AJAX query handlers for its `post_object`, `relationship` and `page_link` field types. Those handlers ran a `WP_Query` whose `post_status` defaulted to `any` with no capability or `perm` filtering, so they returned draft, pending, private and future posts. When such a field appeared on a public `acf_form()`, the required per-field nonce was printed into the page HTML, and because logged-out WordPress nonces are shared across all anonymous visitors, any unauthenticated caller could reuse it to enumerate non-public post titles and IDs, including by keyword search.
- Technique / pattern: Grep a plugin for `wp_ajax_nopriv_` to list its unauthenticated entry points, then follow each handler to its query and check whether `post_status` and a capability test are actually set. The nonce is not an authorization boundary here: logged-out nonces are identical for every anonymous visitor, so harvesting one from public HTML satisfies the check.
- Takeaway: A nonce proves intent, not identity — unauthenticated handlers still need an explicit capability check and an explicit public `post_status`, never the framework default.

### 2026-09-28 — Desktop client leaks credentials to an attacker-chosen directDownloadUrl (Nextcloud) — $250
- Source: [HackerOne #3400143](https://hackerone.com/reports/3400143)
- Type: Sensitive information disclosure / insufficiently protected credentials
- Summary: The Nextcloud desktop client honoured the server-supplied `directDownloadUrl` without validating its origin and without setting `DontAddCredentialsAttribute`, so `HttpCredentialsAccessManager` attached the user's HTTP Basic `Authorization` header to whatever host the server named. A malicious or compromised server could point that field at its own domain and collect the credentials.
- Technique / pattern: In a client-server protocol, list every field where the server dictates a URL the client will fetch, then check whether credentials, cookies or tokens ride along cross-origin. The vulnerable shape is a credential-injecting HTTP stack combined with a URL or redirect field that was never origin-checked — the same pattern as a bearer token surviving a cross-host redirect.
- Takeaway: Clients must treat server-supplied URLs as untrusted: compare the origin against the configured account host and suppress credentials for anything else.

### 2026-09-28 — Public NLSP sidemenu API exposes internal usernames, roles and ACL metadata (NASA VDP) — n/a
- Source: [Bugcrowd #b2c00b8c](https://bugcrowd.com/disclosures/b2c00b8c-2a5e-4d3c-87e6-41f153f3644c/public-api-exposes-internal-nasa-usernames-and-acl-metadata-on-nlsp-sidemenu-endpoint)
- Type: Exposed API / sensitive information disclosure
- Summary: A public endpoint used by the LSDA/NLSP web interface to build user context returned verbose metadata without authentication, including internal account identifiers, NASA staff usernames, access control lists and internal role mappings — enough to enumerate valid usernames and map the platform's role and group structure.
- Technique / pattern: The API calls a single-page app makes on load are often assumed to be internal; read them out of the JavaScript bundle or the network tab and replay each one with no session cookie. Context and menu endpoints are especially productive because they exist to describe the user and their permissions, so they over-return by design.
- Takeaway: A UI-bootstrapping endpoint should authenticate and return only what the current session needs — leaked usernames and role maps are the reconnaissance input for phishing and credential attacks.

### 2026-09-28 — Exposed Next.js build artifacts disclose internal structure and configuration (NASA VDP) — n/a
- Source: [Bugcrowd #62588ff6](https://bugcrowd.com/disclosures/62588ff6-5ae0-444c-9b3b-bd40339c2033/exposed-next-js-build-artifacts-sensitive-configuration-disclosure)
- Type: Sensitive information disclosure / security misconfiguration
- Summary: Build-related Next.js artifacts and configuration files were publicly reachable in a production deployment, revealing internal application structure and configuration details that are not meant to be served.
- Technique / pattern: On a Next.js target, request the build manifests and route lists under `/_next/` — `build-manifest.json`, `routes-manifest.json` and the `_buildManifest.js` and `_ssgManifest.js` pair — to recover the full client route table, then pull the referenced chunks and any source maps for configuration and non-public endpoints. The inlined `__NEXT_DATA__` blob is worth reading on every page too.
- Takeaway: Ship production builds without source maps and block build-metadata paths at the edge; a route manifest hands over the application map that would otherwise take a long crawl to assemble.

### 2026-09-28 — Public Google Maps API key on www.globe.gov allows billable Geocoding API calls (NASA VDP) — n/a
- Source: [Bugcrowd #88f21ab6](https://bugcrowd.com/disclosures/88f21ab6-d1e0-453c-992a-1661d1be50a9/public-google-maps-api-key-on-www-globe-gov-allows-geocoding-api-calls-billable-use)
- Type: Exposed API key (unrestricted client-side key)
- Summary: A Google Maps API key was present in the homepage source with no API or HTTP-referrer restrictions, so it could be used for Geocoding API calls from anywhere, risking unexpected billing, quota exhaustion and abuse. Accepted as informational (P5, accepted business risk).
- Technique / pattern: Client-side Google, Algolia, Stripe and Firebase keys are *meant* to ship in the page, so the finding is never the exposure but the missing restriction. Confirm with a single benign request from an unrelated origin and check the key's API allow-list and referrer restriction rather than reporting the string itself.
- Takeaway: Report a browser key only when you can show it is unrestricted and what it can actually do — and expect a low or informational rating when the only impact is billable quota; restrict such keys by referrer and by API.

### 2026-09-28 — Cross-user PHI/PII leakage via prompt injection in ChatGPT (OpenAI) — $2,000
- Source: [Bugcrowd #5d10a6d3](https://bugcrowd.com/disclosures/5d10a6d3-734e-401d-8871-b141de740a46/urgent-critical-data-breach-cross-user-phi-pii-leakage-via-prompt-injection-non-malicious-discovery)
- Type: Sensitive information disclosure (cross-tenant context isolation failure in an LLM product)
- Summary: A researcher reported health and personal data belonging to another user surfacing inside their own ChatGPT session — a context-isolation failure rather than a conventional API authorization bug. The submission was reopened, validated and awarded P1 ($2,000).
- Technique / pattern: The write-up's real lesson is the triage problem it names: a model can *hallucinate* plausible personal data, so a genuine leak and a fabrication look identical from outside. The evidence that separates them is what to collect — verbatim structured artifacts (record numbers, timestamps, internal identifiers), reproducibility in fresh sessions, and details that could not have been inferred from the prompt.
- Takeaway: When reporting an AI-product data-exposure bug the burden is proving the output came from real stored data; gather corroborating identifiers before filing, and never go hunting for a third party's records to strengthen the case.

### 2026-09-28 — Mail contact autocomplete bypasses administrator-configured user-enumeration restrictions (Nextcloud) — $250
- Source: [HackerOne #3617729](https://hackerone.com/reports/3617729)
- Type: Information disclosure / user enumeration (privacy-setting bypass)
- Summary: An authenticated user could enumerate other users' id, display name and email address through the mail app's contact autocomplete even where the instance restricted enumeration to same-group or exact-match, because the handler only checked `core.shareapi_allow_share_dialog_user_enumeration` and never applied the stricter share-enumeration settings.
- Technique / pattern: Privacy and enumeration settings are enforced per code path, not globally. Inventory every surface that searches users — share dialog, mention picker, mail composer autocomplete, calendar attendee search, admin user list, mobile endpoints — and re-run the same partial-prefix query against each with the restriction enabled.
- Takeaway: A setting is only as strong as its least-careful consumer: when an app exposes the same directory through several features each one needs the full policy check, and a secondary app's controller is where it is usually missing.

### 2026-09-28 — Sensitive information exposed via an unauthenticated panelist export endpoint (Mars) — n/a
- Source: [HackerOne #3376598](https://hackerone.com/reports/3376598)
- Type: Exposed API / Sensitive information disclosure (missing authentication on an export route)
- Summary: An `/export_panelists_to_xlsx` endpoint lacked authentication and authorization and returned panelist email addresses and telephone numbers. Classified CWE-312 with a CVSS of 6.1 (medium), reproduced, validated and remediated.
- Technique / pattern: Export and report-generation routes (`/export*`, `/download*`, `*_to_xlsx`, `*_to_csv`, `/report/generate`) are built for internal or admin use and are frequently mounted outside the middleware guarding the main API. Harvest their paths from JS bundles and admin UI code, then request each one with credentials stripped.
- Takeaway: Every route that serializes a whole table into a file deserves its own authorization check — a bulk export converts a minor gap into a complete dataset in one request.


### 2026-09-27 — Leaked ServiceNow credentials usable because API logins skipped MFA (Tesla) — $5,000
- Source: [Bugcrowd #41991d9c](https://bugcrowd.com/disclosures/41991d9c-3866-40e7-97c2-f33b812565d7/disclosed-servicenow-credentials-leading-to-data-leak)
- Type: Exposed credentials / MFA enforced inconsistently between the UI and the API
- Summary: Production ServiceNow credentials were found through GitHub reconnaissance; MFA was required for interactive sign-in but not for API authentication, so the credentials alone were enough to read all Incidents, Problems, Changes and Configuration Items from the instance. Tesla removed the credentials and extended MFA to production and development API environments.
- Technique / pattern: Treat a found credential as unusable only after testing every authentication surface the product exposes — the REST or SOAP API, a mobile endpoint, a legacy basic-auth path. MFA is routinely attached to the web login while the API keeps password auth alive for integrations, and that gap is exactly where a leaked secret regains its full value.
- Takeaway: Scan your own public repositories and their history for secrets, and make MFA — or key-based auth — a property of the identity rather than of one login surface, so no channel accepts a bare password.

### 2026-09-27 — NASA-owned PDF shared publicly with editor rights (NASA VDP) — n/a
- Source: [Bugcrowd #ed7b7d63](https://bugcrowd.com/disclosures/ed7b7d63-bec4-426f-8ea0-5222cf22be40/nasa-user-owned-pdf-publicly-exposed-with-full-edit-rights-risk-of-deletion-and-pii-disclosure)
- Type: Exposed document / over-permissive sharing configuration
- Summary: A NASA-owned PDF was reachable without authentication and shared at editor level, so anyone holding the link could read the personal data it contained, change the document's collaborator list, or delete the file permanently. The researcher found several further files carrying the same misconfiguration.
- Technique / pattern: When a document link turns out to be public, check the *permission level* rather than stopping at read access — open the sharing dialog, look for edit or collaborator controls, and test whether the pattern repeats across neighbouring files from the same uploader or folder. "Anyone with the link" combined with editor rights converts disclosure into destruction.
- Takeaway: Default document sharing to view-only and to named accounts, and audit link-shared files specifically for write permissions, since one wrong dropdown makes a document deletable by the internet.

### 2026-09-27 — Unauthenticated PII leak through profile version history (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1627962](https://hackerone.com/reports/1627962)
- Type: Exposed API / unauthenticated information disclosure
- Summary: Rendering different versions of a published profile triggered a background request to an unprotected API endpoint that returned user identifiers and names for every version — and, for verified profiles or those pending verification, their email addresses as well, all with no authentication required.
- Technique / pattern: Watch the network panel while using a public feature, then replay each backing request on its own with cookies and auth headers stripped. Version-history, revision and audit endpoints are prime targets because they are written for internal review and tend to return the union of fields across all versions rather than the narrower projection the public view is allowed to show.
- Takeaway: Authorize each API endpoint independently of the page that calls it, and make history endpoints return the same field projection as the public view.

### 2026-09-27 — Unauthenticated access to a "For Official Use Only" NASA document (NASA VDP) — n/a
- Source: [Bugcrowd #411165dd](https://bugcrowd.com/disclosures/411165dd-8c0a-4ae7-b731-6f685d862e9d/critical-identity-and-communication-data-exposed-in-unprotected-nasa-hangar-demolition-doc-vulnerability)
- Type: Exposed document / missing access control on a sensitive internal file
- Summary: An internal NASA hangar-demolition document marked "For Official Use Only – Not for Public Release" was retrievable with no authentication, exposing names, email addresses, phone numbers, postal addresses and agency affiliations of staff and external partners, along with internal correspondence and signatures. Rated P3 and resolved.
- Technique / pattern: Search for the handling markings themselves — phrases like "For Official Use Only", "Not for Public Release" or "Internal Use Only" — across a target's document hosts and the search-engine index. A file that declares its own sensitivity supplies self-labelling evidence of impact, which makes the finding fast to triage and hard to dispute.
- Takeaway: Sensitivity markings are labels, not access controls; files carrying them need authentication and an index exclusion, and the owner's own periodic search should be what finds them exposed.

### 2026-09-27 — Asana S3 attachments leaked via a server-signed arbitrary `s3_path` plus OSINT (Asana) — $2,500
- Source: [Bugcrowd #5399313d](https://bugcrowd.com/disclosures/5399313d-b86d-4965-9c32-dbf77660d4ea/user-data-from-the-asana-s3-bucket-can-be-leaked-via-presigned-url-and-osint)
- Type: Exposed API / server-side signing of a client-supplied storage path
- Summary: A websocket mutation to `/s3_attachment_create` accepted an `s3_path` field without validating it, so the server would generate a valid presigned URL for any object path the client named. Combined with S3 keys harvested from public sources, that gave read access to other users' attachments.
- Technique / pattern: Look for the shape "client names the storage location, server signs it" in upload and attachment flows — presigned-URL issuance endpoints, `create_attachment`, `direct_upload`, `get_signed_url`. The server is the trust anchor for the signature, so an unvalidated path parameter turns it into a signing oracle for the whole bucket; websocket and RPC mutations are a good hunting ground because they often sit outside the REST validation layer.
- Takeaway: Derive the storage key server-side from the authenticated session and the target record; never sign a path that arrived in the request, and scope presigned URLs to a single object the caller is authorized for.

### 2026-09-27 — Hardcoded credentials in client-side JavaScript open the Lucidworks Fusion admin panel (U.S. EPA VDP) — n/a
- Source: [Bugcrowd #d900e24e](https://bugcrowd.com/disclosures/d900e24e-af98-48d1-ac5f-74c1c2ec5ad5/public-user-login-credentials-leaked-for-lucidworks-fusion-admin-panel)
- Type: Exposed API / secrets shipped in client-side code
- Summary: A cleartext credential pair sat in `app_v2_1.js` and a Base64-encoded administrative pair in `setup.js`, giving access to the Lucidworks Fusion admin interface at `search.epa.gov/admin`. From that session the researcher reached dashboards, DevOps panels and configuration interfaces exposing internal system information and service logs, and also demonstrated prototype pollution against notification objects held in session storage.
- Technique / pattern: Pull and beautify every script a search or admin frontend loads, then grep for credential-shaped strings *and* for Base64 blobs, because encoding is not a control. Setup and bootstrap scripts are the most productive files since they configure the backend rather than render the UI; rate the finding by walking the administrative routes the recovered session actually opens.
- Takeaway: Anything shipped to the browser is public — backend credentials belong on the server behind an authenticating proxy, and Base64 in a bundle should be read as plaintext.

### 2026-09-27 — Unauthenticated PII exposure through an ECFS prototype API directory (FCC VDP) — n/a
- Source: [Bugcrowd #1f0205ac](https://bugcrowd.com/disclosures/1f0205ac-3eb1-45c8-a777-a27b10e66bd5/sensitive-data-exposure-user-personal-details)
- Type: Exposed API / unauthenticated data exposure on a non-production host
- Summary: The `/filings/` path on the host `ecfsapi-prototype.fcc.gov` returned filer records including email and physical addresses with no authentication at all. It was triaged P1 and has been resolved.
- Technique / pattern: Enumerate the environment variants of every API hostname in scope — `-prototype`, `-dev`, `-staging`, `-uat`, `api2`, numbered versions — because these carry production data far more often than they carry production controls, and they are usually absent from the WAF and monitoring covering the main host. Certificate-transparency logs and DNS brute-forcing both surface them.
- Takeaway: Non-production hostnames must not hold production records, and each deployment needs its own authentication rather than inheriting the main host's perimeter.

### 2026-09-27 — PII of all site users exposed through faulty access control on an endpoint (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #904659](https://hackerone.com/reports/904659)
- Type: Exposed API / broken access control on a user-listing endpoint
- Summary: A redacted DoD site let any registered user reach an endpoint listing all site users and, by selecting a username, view their personally identifiable information — the check on that endpoint did not distinguish "logged in" from "allowed to see other users".
- Technique / pattern: The cheapest privilege boundary to test is "any account at all versus everyone else's data": register, then hunt for directory, listing, search, org-chart and member-picker endpoints and see what a plain user may enumerate. Authorization written as an authentication check is the usual cause, and it is invisible to an unauthenticated scan.
- Takeaway: Authenticated does not mean authorized — listing endpoints need an explicit role or relationship check, and a self-registrable account is the right baseline for testing them.

### 2026-09-26 — Unauthenticated JSON API endpoint exposing internal user IDs, names and corporate emails (Mars) — n/a (Medium 4.6)
- Source: [HackerOne #3360293](https://hackerone.com/reports/3360293)
- Type: Exposed API / information disclosure (CWE-200)
- Summary: An unauthenticated JSON API endpoint returned sensitive account data — internal UUIDs, display names, corporate email addresses, and even administrative role flags — with no authentication, enabling user enumeration and targeted attacks. Fixed by adding authentication and minimizing the returned fields.
- Technique / pattern: During recon, hit JSON/REST endpoints directly with no session and read the raw body: endpoints built for an authenticated front end frequently answer anonymous callers and over-return internal fields (UUIDs, roles, emails) the UI never shows. Role flags in the payload are the escalation signal — they turn a directory leak into a target list of privileged accounts.
- Takeaway: Two controls were missing at once — authentication and data minimization. Require auth on every data endpoint and serialize only the fields the caller is authorized to see; internal identifiers and role metadata should never appear in an anonymously reachable response.

### 2026-09-26 — Information disclosure in API endpoint /users (U.S. Dept Of Defense) — n/a (Low, Resolved)
- Source: [HackerOne #3027405](https://hackerone.com/reports/3027405)
- Type: Exposed API / information disclosure
- Summary: A `/users` API endpoint returned user information that should not have been exposed; the report was triaged, resolved, and later publicly disclosed (rated Low).
- Technique / pattern: Enumerate REST collection endpoints (`/users`, `/accounts`, `/members`) and inspect the full JSON body, not just the rendered UI: APIs routinely over-return fields (internal IDs, emails, roles, flags) that the front end hides. Compare authenticated vs unauthenticated responses and trim query params to see what the server discloses by default.
- Takeaway: Practice response minimization on APIs; serialize only the fields a caller is authorized to see rather than the whole record. UI-side filtering is not access control, and a verbose collection endpoint is a standing information-disclosure surface.

### 2026-09-26 — Critical information disclosure via search path leak (Department of Interior VDP) — n/a (P5, Informational)
- Source: [Bugcrowd 810206c6](https://bugcrowd.com/disclosures/810206c6-f547-48b9-9fc6-64da9933b029/critical-information-disclosure-at-https-www-doi-gov)
- Type: Sensitive data exposure / path disclosure
- Summary: A search parameter on `doi.gov` disclosed internal file paths (for example a `pdsimage2.wr.usgs.gov/data/` location) pointing at further resources. It was classified P5 / Informational as disclosure of already-public information.
- Technique / pattern: Feed crafted queries into search and lookup features and read the responses for absolute paths, host names, and directory structures leaking from error text or result metadata. Path disclosure is useful mainly as recon that feeds a stronger bug (LFI, directory traversal, access-control test) rather than as an end in itself.
- Takeaway: Path and host disclosure alone usually rates Informational; its value is as a stepping stone. Chain the leaked path to a demonstrable read of non-public data before claiming "critical," and defensively return generic errors that never echo server-side paths.

### 2026-09-26 — Open redirect via redirect-uri leaking verification PII through Referer (Persona) — n/a (P5, Informational)
- Source: [Bugcrowd eaa43961](https://bugcrowd.com/disclosures/eaa43961-ac08-481a-8196-26552516dd77/open-redirect-using-redirect-uri-parameter-in-the-url-leads-to-mass-information-disclosure-of-all-sensitive-information-collected-during-verification)
- Type: Information disclosure via open redirect (GET-based)
- Summary: Persona's identity-verification flow let an attacker override the post-inquiry destination with a `redirect-uri` parameter that was not validated, so when a user finished verification and was redirected to an attacker-controlled site, sensitive data (SSN, government ID, email) traveled to that site in the `Referer` header and URL. Closed as Informational.
- Technique / pattern: On any flow that carries sensitive values in the URL or query string, test whether the redirect target is attacker-controllable, then observe what leaks to the third-party origin via `Referer` and forwarded parameters. The leak is a composition bug: open redirect plus sensitive data in the URL plus a permissive referrer policy.
- Takeaway: Never put sensitive identifiers in URLs, validate redirect targets against an allowlist, and set `Referrer-Policy: no-referrer` (or `same-origin`) on pages that handle PII, so a redirect cannot exfiltrate data through the browser.

### 2026-09-25 — Unauthenticated testing endpoint of the notify_push component exposes internal IP addresses (Nextcloud) — n/a
- Source: [HackerOne #3513471](https://hackerone.com/reports/3513471)
- Type: Sensitive information disclosure via an unauthenticated diagnostic endpoint
- Summary: A self-test endpoint belonging to the `notify_push` component answered without authentication and returned internal IP addressing, handing an unauthenticated caller a view of the deployment's private network layout.
- Technique / pattern: Sidecar and helper daemons — push/notify services, metrics exporters, health shims, queue workers — ship self-test and diagnostic routes that are assumed to be reachable only from inside the cluster, and stay open when the component is deployed alongside the public app. Enumerate the routes of each auxiliary component separately from the main application's route map.
- Takeaway: Every companion service needs its own authentication and network policy; a diagnostic route is production attack surface. Bind self-test endpoints to loopback or require a shared secret, and never echo internal addressing to an unauthenticated caller.

### 2026-09-25 — Email enumeration via the identity-verification step on password-protected shares (Nextcloud) — n/a
- Source: [HackerOne #3507273](https://hackerone.com/reports/3507273)
- Type: Information disclosure — response-differential (oracle) enumeration
- Summary: When a password was requested for an email-protected share, the application returned a different message depending on whether the submitted address was the share's intended recipient, so anyone holding a share token could confirm who the share was addressed to.
- Technique / pattern: The researcher submitted a known-wrong address and a candidate address to the same flow and diffed the responses. Any branch that words its reply differently for a valid subject is an enumeration oracle, and the differential can be in wording, status code, redirect target, response length or timing rather than in an explicit error.
- Takeaway: Identity-verification and password-request flows must return a single indistinguishable response for every input. Compare responses byte-for-byte, including headers and timing, when testing — and when designing, generate the reply before the lookup decides anything.

### 2026-09-25 — Source code and internal infrastructure disclosure via a publicly exposed GitLab repository on git.smce.nasa.gov (NASA Vulnerability Disclosure Program) — n/a (P2)
- Source: [Bugcrowd b080842d](https://bugcrowd.com/disclosures/b080842d-1a0c-4eaa-9c91-989a01c48a54/source-code-and-internal-infrastructure-disclosure-via-publicly-exposed-gitlab-repository-on-git-smce-nasa-gov)
- Type: Sensitive information disclosure — self-hosted VCS project left public
- Summary: A repository on a self-hosted GitLab instance was readable without authentication, disclosing internal source code along with infrastructure and architecture details.
- Technique / pattern: Self-hosted GitLab, Gitea and Gerrit instances on organization subdomains expose a project-explore listing that enumerates every repository whose visibility is public or internal; the researcher browsed that listing rather than guessing paths. The same instances also serve raw file, blob and pipeline-artifact routes that can remain reachable after the web UI is locked down.
- Takeaway: Audit visibility on every project of a self-hosted VCS, not just the instance's sign-in page, and re-check after migrations — default-public project settings are the usual cause. Treat a public repository listing on an internal-looking subdomain as a finding in itself.

### 2026-09-25 — Systemic API misconfiguration in CKAN harvesting exposes internal secrets, an infrastructure map and PII (NASA Vulnerability Disclosure Program) — 7 points (P5)
- Source: [Bugcrowd 2309545f](https://bugcrowd.com/disclosures/2309545f-8d7f-4c94-90cf-caef26a09dff/systemic-api-misconfiguration-in-ckan-harvesting-exposes-internal-secrets-infrastructure-map-and-pii)
- Type: Exposed API / sensitive information disclosure through data-portal harvesting configuration
- Summary: The harvesting API of a CKAN open-data portal returned harvest source configuration to callers who should not see it, disclosing internal secrets, a map of connected infrastructure and personal information. The program accepted the risk rather than treating it as impactful, so it was closed as informational.
- Technique / pattern: Data-portal and ETL platforms store the credentials and endpoints of every upstream system inside the objects that describe a harvest or sync job. Those config objects are often served by the same public read API as the datasets themselves, so listing harvest sources yields an inventory of internal hosts plus whatever secrets the connectors need.
- Takeaway: Wherever a platform ingests from elsewhere, the ingestion configuration is a credential store — keep it behind a separate authorization boundary from the published data. When reviewing any open-data or integration API, enumerate the job/connector/source objects, not just the content ones.

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
