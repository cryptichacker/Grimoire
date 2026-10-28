---
tags: [hacking, bug-bounty, misconfiguration, cors, reports]
type: log
---

# Security Misconfigurations

## Up
- [[Reports]]

Disclosed **security misconfiguration** reports — permissive CORS, default/exposed configs, missing security controls, verbose errors. See [[Pattern]] → Security Misconfigurations.

## Reports

### 2026-10-28 — CloudFront misconfiguration makes rubygems.org serve and cache content from an unclaimed S3 bucket (Internet Bug Bounty) — n/a
- Source: [HackerOne #2262939](https://hackerone.com/reports/2262939)
- Type: Cloud/CDN misconfiguration (dangling origin takeover)
- Summary: A CloudFront distribution for rubygems.org pointed at the S3 bucket `index.rubygems.org`, which no longer existed. Claiming that bucket name let an outsider serve arbitrary content — with an attacker-chosen content type — from a path on the real `rubygems.org` origin, and CloudFront then cached it.
- Technique / pattern: The reporter noticed `https://rubygems.org/names` returning an S3 `NoSuchBucket` error naming the missing bucket. Registering that bucket name produced a `TemporaryRedirect` error that revealed the region the distribution expected, and re-creating it in the correct region made the CDN serve the attacker's file. The error bodies themselves were the oracle for both the bucket name and the region.
- Takeaway: Treat CDN and proxy error bodies as reconnaissance: `NoSuchBucket` names a claimable origin, and a region-mismatch error tells you where to claim it. Audit every CDN origin for names that are no longer owned, and remember CDN caching turns a brief takeover into a persistent one.

### 2026-10-28 — Premium plugin archives with embedded license keys publicly readable in the uploads directory (Essity) — n/a
- Source: [HackerOne #3839889](https://hackerone.com/reports/3839889)
- Type: Security misconfiguration / sensitive file exposure
- Summary: The WordPress media library exposed 29 ZIP archives of paid plugins (Slider Revolution, ACF Pro, Wordfence Premium and others), downloadable without authentication and indexed by the REST API. The archives contained full plugin source and embedded license/purchase verification tokens.
- Technique / pattern: Enumerate the media library through the REST API rather than guessing paths: `/wp-json/wp/v2/media?per_page=100&media_type=application` lists non-image uploads, including archives whose filenames carry random suffixes that would never be found by brute force. Each `source_url` was then fetched unauthenticated and grepped for `purchase_code`, `license`, `api_key` and `activation`.
- Takeaway: An unguessable filename is not access control when an API enumerates it for you. Check `/wp-json/wp/v2/media` (and equivalent CMS listing APIs) for archives, backups and configuration files, and keep distributable packages out of web-served upload directories.

### 2026-10-28 — Vercel subdomain takeover via dangling CNAME records on retired marketing subdomains (Settle VDP) — n/a
- Source: [Bugcrowd af0ecc4f](https://bugcrowd.com/disclosures/af0ecc4f-c7aa-4a72-9653-f5051e79e214/vercel-subdomain-takeover-capitalcalc-settle-com-embed-settle-com)
- Type: Subdomain takeover (dangling DNS)
- Summary: Two retired marketing subdomains still had CNAME records pointing at a Vercel deployment that had been deleted or left unclaimed. Claiming the orphaned project would have let an attacker host content on those names, obtain valid TLS certificates for them, and run convincing credential-phishing against users and staff. The records were removed within a day.
- Technique / pattern: Resolve every subdomain and compare the CNAME target against what the hosting provider actually serves; a provider-specific 404 or "project not found" page on a live CNAME marks a claimable name. The impact is driven by what the name confers rather than by code: a valid certificate under the real domain plus first-party cookie scope.
- Takeaway: Decommissioning a site means deleting the DNS record, not just the deployment. Inventory CNAMEs pointing at PaaS providers continuously, since the window opens the moment a project is deleted on the provider side.

### 2026-10-28 — Apache Flink Dashboard exposed to the internet with no authentication (NASA VDP) — n/a
- Source: [Bugcrowd 5f083144](https://bugcrowd.com/disclosures/5f083144-038f-455a-9985-ad4eea855913/broken-acces-control-leads-to-unauthenticated-access-to-apache-flink-dashboard)
- Type: Security misconfiguration (unauthenticated admin interface)
- Summary: An Apache Flink Dashboard was reachable from the public internet with no authentication mechanism, letting any remote user monitor cluster activity, read internal job configurations and pull sensitive operational metadata. Rated P1 and since fixed.
- Technique / pattern: Fingerprint data-platform and orchestration consoles that ship with authentication disabled by default — Flink, Spark, Dagster, Airflow, Kubernetes dashboards, Elasticsearch and similar — on their well-known ports and paths. Job configuration views on these consoles routinely carry connection strings, bucket names and internal hostnames, and several expose a job-submission path that turns read access into code execution.
- Takeaway: Big-data and workflow components are built to trust their network, so they are secure only as long as that network is private. Treat any such console answering from the internet as a critical finding, and check whether its submit/upload path extends the exposure to RCE.

### 2026-10-26 - Administrator password hardcoded in client-side JavaScript on an Uber vendor site (Uber) - n/a
- Source: [HackerOne #344566](https://hackerone.com/reports/344566)
- Type: Misconfiguration / secret embedded in client-side code
- Summary: `desafio5estrelas.com`, a vendor-built and vendor-managed site Uber used in Portugal to help low-rated drivers improve their ratings, exposed an administrator login password in its JavaScript source. Anyone reading the source could sign into the admin panel, which exposed in-app survey data and user IDs.
- Technique / pattern: Pull every script a page loads and grep for credential-shaped strings - `password`, `passwd`, `token`, `secret`, basic-auth blobs. Vendor-run marketing and campaign microsites are the richest ground: they carry the brand's name and users' trust while sitting outside the main security review.
- Takeaway: Enumerate third-party and campaign domains as part of the target's attack surface. "Managed by a vendor" predicts weaker controls, not stronger ones, and client-side JavaScript is not a place to keep a password.

### 2026-10-26 - Administrative interface exposed to the public internet on a government host (U.S. Dept Of Defense) - n/a
- Source: [HackerOne #223948](https://hackerone.com/reports/223948)
- Type: Misconfiguration / administrative interface reachable from the internet
- Summary: A researcher found an administrative interface reachable from the public internet on an `nsa.gov` host. The disclosed report is heavily redacted and documents the exposure itself rather than any exploitation of it.
- Technique / pattern: Content discovery against known admin paths, combined with certificate-transparency and DNS enumeration to surface hosts that were never meant to be public. On large estates the finding is usually the reachability - an admin console that should sit behind a VPN or an IP allowlist answering from the open internet.
- Takeaway: Exposure alone is a valid report; you do not need to authenticate to show impact. Record the host, the path and the fact that it responds, and stop there rather than attempting access.

### 2026-10-26 - Open redirect via `redirecturl` on a SAP Fiori logoff endpoint (Village Roadshow) - n/a (P4)
- Source: [Bugcrowd #8ad32ee1](https://bugcrowd.com/disclosures/8ad32ee1-85a6-4810-9a1a-33be69c5a921/open-redirect-on-https-fiori-vrl-com-au)
- Type: Unvalidated redirect (CWE-601), GET-based
- Summary: The logoff endpoint on `fiori.vrl.com.au` accepted a `redirecturl` parameter and sent the user to whatever external address it named, letting an attacker bounce victims off a trusted host onto a malicious site. Submitted February 2025, disclosed June 2025, resolved.
- Technique / pattern: Logout and logoff routes are a standard open-redirect hotspot, because the post-logout destination is deliberately configurable. Test every parameter whose name contains `url`, `redirect`, `return`, `next`, `continue` or `dest` with an absolute external URL, a protocol-relative `//evil.example`, and a value that merely prefixes the allowed host.
- Takeaway: Redirect destinations belong on an allowlist of relative paths or known hosts. Framework-provided logoff endpoints ship this parameter by default, so check the vendor route even when the application's own code looks clean.

### 2026-10-26 - Open redirect on a NASA asset (NASA VDP) - n/a (P4)
- Source: [Bugcrowd #53007ed9](https://bugcrowd.com/disclosures/53007ed9-d4b4-4b28-98ee-1ce5c2a613ad/open-redirect-on-nasa)
- Type: Unvalidated redirect
- Summary: An open redirect on a NASA asset, reported December 2023, accepted and fixed at P4. The public page records only the classification and the outcome - the reporter's description is a single word, so no endpoint or payload is documented.
- Technique / pattern: Nothing new technically; logged as a data point that even P4 unvalidated redirects on a large government estate get accepted and fixed, which makes them a reasonable opening finding when mapping a wide VDP scope.
- Takeaway: A disclosed report with no writeup is still evidence of what a program accepts. When budgeting time on a VDP, the distribution of resolved low-severity classes tells you where effort is likely to be rewarded.

### 2026-10-23 — Exposed Slinky admin panel allowing modification of trusted URL redirects (Shopify) — n/a
- Source: [HackerOne #808762](https://hackerone.com/reports/808762)
- Type: Exposed administrative interface / missing authentication
- Summary: The reporter noticed a Shopify Slinky host that had been returning 404 start returning 200, and found an unauthenticated admin panel that allowed creating and modifying URL redirects. The reported impact was the ability to alter redirects that users would treat as trusted Shopify links.
- Technique / pattern: Keep a monitored list of in-scope hosts and alert on HTTP status changes over time. A host flipping from 404 to 200 marks a newly deployed or newly misrouted service, and newly deployed admin interfaces are the ones most likely to still be missing their auth layer.
- Takeaway: Recon is a time series, not a snapshot. The finding here was not a clever payload but noticing a status-code change overnight — and the impact argument was redirect trust on a first-party domain, not the panel itself.

### 2026-10-23 — Unauthenticated access to an administrative panel and all its functions (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1394910](https://hackerone.com/reports/1394910)
- Type: Missing authentication on an administrative interface
- Summary: An administrative panel belonging to a management system was reachable without logging in, and the reporter stated that its functions were usable too, allowing files to be uploaded, deleted or modified. The host and the product name are redacted on the public page.
- Technique / pattern: Reaching an admin panel's login-free landing page is only half the finding. Enumerate the panel's own action endpoints and confirm at least one performs a privileged operation unauthenticated — many panels render without a session but still reject the actions behind them.
- Takeaway: Severity hinges on whether the exposure is cosmetic (a visible dashboard) or functional (an action that executes). Demonstrate one benign functional call, and note an upload capability explicitly, since file upload on an unauthenticated panel is a direct path to code execution.

### 2026-10-23 — Header-based open redirect on the identity-verification settings endpoint (Bugcrowd) — n/a
- Source: [Bugcrowd 8a036dce](https://bugcrowd.com/disclosures/8a036dce-d5ee-406d-a35f-a91df1cc57d1/open-redirect-on-https-bugcrowd-com-settings-identity_verification)
- Type: Unvalidated redirect / open redirect, header-based (P5, informational; disclosed 16 Apr 2020)
- Summary: The reporter stated that `POST /settings/identity_verification` redirected the user to an arbitrary external site based on the request's `Referer` header, demonstrating it by intercepting the request and substituting an external domain. The proof-of-concept request also carried an `X-Forwarded-Host` header. It was triaged P5 and treated as an accepted business risk.
- Technique / pattern: When no `url`/`next`/`redirect` parameter is present, test whether the redirect target is derived from a request *header* instead — `Referer`, `X-Forwarded-Host`, `Host`. Header-driven redirects are easy to miss because parameter-focused scanners never look there.
- Takeaway: A redirect sourced from `Referer` is usually rated low precisely because the attacker cannot set that header in a victim's browser cross-site. Check deliverability before reporting: if the only way to set the header is an intercepting proxy on your own traffic, it is self-inflicted.

### 2026-10-23 — Server-side request forgery in an image-import feature (engagement undisclosed) — n/a
- Source: [Bugcrowd 3b725bac](https://bugcrowd.com/disclosures/3b725bac-d2b8-49eb-842d-f73c32631cab/ssrf)
- Type: Server-Side Request Forgery (P4, informational; disclosed 23 Nov 2023)
- Summary: The program's summary records server-side request forgery in an image-import function: a feature that fetches a user-supplied URL server-side could be pointed at destinations of the submitter's choosing. It was rated P4 and accepted as a business risk; the engagement name is withheld on the public page.
- Technique / pattern: Any "import from URL" feature — avatars, image import, link previews, webhooks — is an SSRF candidate because the server is designed to fetch an address you control. Severity then depends on what the fetch can reach: cloud instance metadata and internal-only services turn a P4 into a critical.
- Takeaway: Finding the SSRF is the easy half. The rating is decided by the follow-on evidence — a response from an internal host or metadata endpoint — so a report that only proves an outbound callback to your own listener lands as low severity.

### 2026-10-22 — Outdated WordPress plugin carrying a known Contributor-level stored XSS (NASA VDP) — n/a
- Source: [Bugcrowd b8cacace](https://bugcrowd.com/disclosures/b8cacace-a46d-455b-9718-d351a1f98de2/wordpress-plugin-vulnerability)
- Type: Security misconfiguration (outdated third-party component) — P5, informational
- Summary: The report flagged Popup Maker below version 1.20.5, which carries a known stored XSS reachable by any Contributor-or-above user through its `popupID` parameter. The program verified the site was already running the fixed 1.20.5 and closed the finding as informational.
- Technique / pattern: Fingerprint the CMS, enumerate installed plugin and theme versions from their own readme and asset paths, then map each version against public advisories to find components below a known-vulnerable boundary. The work is in establishing the exact version, since the advisory supplies the impact.
- Takeaway: Version fingerprinting is cheap, but the claim has to be verified against the live install before reporting — a scanner-grade "plugin X below Y" finding on a site already running Y is noise. The credible version of this report proves the vulnerable code path is actually reachable on the target.

### 2026-10-22 — Exposed credentials for a Jira account with full Admin permissions (ISC2) — n/a
- Source: [Bugcrowd 43b31ad4](https://bugcrowd.com/disclosures/43b31ad4-c1b1-458f-95fb-e81ed0455014/found-username-and-password-of-your-account-jira-with-full-permission-admin)
- Type: Security misconfiguration (exposed credentials / administrative access) — P1
- Summary: A researcher recovered a working username and password for an ISC2 Jira account carrying full Admin permissions. It was triaged P1, accepted and fixed; the public report does not record where the credentials were found.
- Technique / pattern: Credential hunting against internal collaboration tooling rather than the flagship product — Jira, Confluence, CI dashboards and wikis are frequently stood up outside the asset inventory, keep weak or shared logins, and hold the tickets and attachments that describe the rest of the estate.
- Takeaway: Valid credentials to an admin-level issue tracker are usually a P1 on their own, because the tracker's contents map the organisation's infrastructure and its unfixed vulnerabilities. Internal SaaS belongs inside the scope sweep, not treated as out of band.

### 2026-10-22 — Dangling S3 bucket name referenced by a CI dependency script (Reddit) — n/a
- Source: [HackerOne #1285598](https://hackerone.com/reports/1285598)
- Type: Security misconfiguration (S3 bucket takeover / dependency-source hijack)
- Summary: A dependency-installation script in Reddit's `rpan-studio` repository pulled build inputs from an S3 bucket named `obs-nightly` in `us-west-2` that no longer existed. Because bucket names are globally allocated and reusable once released, anyone could register the name and control what the build downloaded.
- Technique / pattern: Read the CI and dependency scripts in a target's public repositories for every external fetch, then check whether each named bucket, package or domain is still owned by the target. An unresolved name is a claimable name, and creating the bucket in the stated region is the whole proof of concept.
- Takeaway: Supply-chain takeover starts in source control, not on the web app — grep public repos for hardcoded bucket, registry and host names and resolve each one. The impact argument is build-time code execution for whoever claims the name, which is why these are worth more than the "it is just a 404" framing suggests.

### 2026-10-22 — AWS credentials served in a publicly readable config.json (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1704035](https://hackerone.com/reports/1704035)
- Type: Security misconfiguration (exposed configuration file / hardcoded cloud credentials)
- Summary: A `config.json` file holding sensitive AWS material was reachable over HTTP from the web root of a DoD host. The proof of concept was nothing more than requesting the file by its path.
- Technique / pattern: Content discovery against the deployed application directory for build-time configuration artefacts — `config.json`, `.env`, `appsettings.json`, `settings.py` and friends — which get bundled by the deploy pipeline and then served as static files because the web server has no rule excluding them.
- Takeaway: Secrets in a config file are only as protected as the directory they sit in, and front-end build steps routinely copy server config into the public bundle. Any file the application reads at startup should be verified as unreachable from outside, and credentials found this way should be treated as potentially live rather than assumed stale.

### 2026-10-21 — Stealing users' OAuth tokens via an unvalidated redirect_uri on a .gov login flow (GSA Bounty) — n/a
- Source: [HackerOne #665651](https://hackerone.com/reports/665651)
- Type: Security misconfiguration — OAuth open redirect / token leak
- Summary: A crafted link to the cloud.gov login OAuth authorize endpoint used `response_type=token` with `redirect_uri` pointing at an attacker-controlled external domain. After the user selected a .gov account to sign in with, the authorization flow delivered the access token to that external site.
- Technique / pattern: On any authorize endpoint, test `redirect_uri` against the registered allowlist: a fully external host, a subdomain of an allowed host, an open-redirect chain on an allowed host, and path or fragment variations. The implicit flow (`response_type=token`) raises impact because the credential is returned in the URL fragment, so a single navigation hands it over with no code-exchange step to intercept.
- Takeaway: Redirect-URI validation must be an exact match against registered values, not a prefix or substring check. Where a provider still supports the implicit flow, a weak allowlist is a direct credential leak rather than a mere redirect nuisance.

### 2026-10-21 — Open redirect on an 8x8 subdomain via IP-plus-path URL confusion (8x8) — n/a
- Source: [HackerOne #1637571](https://hackerone.com/reports/1637571)
- Type: Security misconfiguration — open redirect
- Summary: The reporter found that a URL built from a raw IP address followed by a path ending in a lookalike suffix caused the application to redirect users to a hostname that merely appeared to sit under the target's domain — the demonstration landed on `https://www.8x8.com.example.com`. The issue was fixed.
- Technique / pattern: Attack the host-matching logic rather than the redirect parameter alone. Suffix-confusion payloads (`target.com.attacker.com`), raw-IP forms, and path segments crafted to be parsed as authority are the standard set, because validators that use "contains" or "endswith" on an unparsed string accept all of them.
- Takeaway: Any redirect allowlist must parse the URL and compare the resolved host, never pattern-match the raw string. A domain that is a prefix of the attacker's host is the single most common bypass of a naive check.

### 2026-10-21 — HTML injection in Confluence redirects searches to an arbitrary page (Atlassian) — 2 points (P3)
- Source: [Bugcrowd #f91f6ee5](https://bugcrowd.com/disclosures/f91f6ee5-9400-468d-a75a-509c9eec67ba/critical-html-injection-redirect-csrf)
- Type: Security misconfiguration — HTML injection leading to forced redirect
- Summary: Disclosed 21 Jul 2023 and resolved. Injected markup placed in a Confluence document body caused a user who searched for certain keywords to be redirected to an attacker-chosen URL. The reporter listed running JavaScript on the destination page, using the redirect as a CSRF delivery vehicle, and sending users to a fake login page to capture credentials as the downstream impacts.
- Technique / pattern: Where a wiki or CMS sanitizer strips `<script>` but still permits layout or meta-level markup inside stored content, test the markup that redirects without scripting — meta refresh, base-tag rewriting, and framing constructs. The search-results renderer is a good place to look because it re-renders fragments of stored documents through a different code path than the page view.
- Takeaway: HTML injection is not a weaker XSS; a sanitizer that only blocks script tags can still be driven into navigation control. Test every secondary renderer (search snippets, previews, exports, notification emails) separately from the primary page view.

### 2026-10-21 — Blind SSRF on gofundme.com (GoFundMe) — n/a (P4)
- Source: [Bugcrowd #05a760c6](https://bugcrowd.com/disclosures/05a760c6-21c5-45eb-80b4-6760dc71fed1/blind-ssrf-on-gofundme-com)
- Type: Security misconfiguration — blind server-side request forgery
- Summary: A blind SSRF was reported against gofundme.com, triaged P4, and resolved in a product release before disclosure on 26 Mar 2021. The public page publishes only the summary and timeline, so no endpoint or payload is given.
- Technique / pattern: Blind SSRF is confirmed by out-of-band interaction rather than by response content: point any server-fetched URL field (avatar or image import, webhook, link preview, PDF or thumbnail generator, feed importer) at a collaborator host and watch for the DNS or HTTP callback. Once a callback lands, the escalation questions are whether redirects are followed, whether internal addresses and cloud metadata endpoints are reachable, and which schemes besides HTTP are honored.
- Takeaway: Keep an out-of-band listener in the default workflow so blind classes get caught at all — and when a program resolves quietly, the disclosure may be summary-only, which is still worth logging as a surface reminder even without technical detail.

### 2026-10-21 — Clickjacking in a NASA web application (NASA Vulnerability Disclosure Program) — n/a (P5, Informational)
- Source: [Bugcrowd #36b7176e](https://bugcrowd.com/disclosures/36b7176e-2fbd-4e6d-a2d1-917e91c8bbc6/vulnerability-report-clickjacking-in-nasa-application)
- Type: Security misconfiguration — missing framing protection (clickjacking)
- Summary: Disclosed 15 Jan 2025 against the NASA VDP. NASA's published note is simply that the site was updated to resolve the finding; the report was classified Informational, meaning an accepted business risk. The public page carries no technical detail.
- Technique / pattern: Framing checks are cheap and belong in a first-pass sweep of any new host: look for a missing `X-Frame-Options` header and the absence of a `frame-ancestors` directive in `Content-Security-Policy`, then confirm by actually loading the page in an iframe. Severity depends entirely on what a framed click can accomplish, so the research effort goes into finding a one-click state-changing action behind it.
- Takeaway: Clickjacking with no sensitive single-click action attached is routinely closed as informational. Log the header gap, but only write it up when a specific consequential action can be driven through the frame.

### 2026-10-20 — HTTP request smuggling via unsanitized hop-by-hop headers (Cloudflare Public Bug Bounty) — $1,500
- Source: [HackerOne #3622877](https://hackerone.com/reports/3622877)
- Type: Security misconfiguration / HTTP request smuggling (proxy header handling)
- Summary: Hop-by-hop headers were forwarded rather than stripped at the proxy boundary, so a request could be interpreted differently by the front end and the origin, enabling request smuggling. Triaged High, classified `Improper Access Control - Generic`, $1,500 bounty.
- Technique / pattern: At any proxy or CDN boundary, test which connection-scoped headers survive the hop — `Connection`, `Transfer-Encoding`, `Keep-Alive`, `Upgrade`, and anything a request nominates inside `Connection:`. A header the front end should have consumed but passes on instead leaves two parsers disagreeing about where one request ends.
- Takeaway: A proxy must normalise and strip hop-by-hop headers, including the ones listed dynamically in `Connection`, before forwarding; smuggling comes from parser disagreement, so the fix is one canonical interpretation at the edge.

### 2026-10-20 — Host header injection on a bia.gov host (Bureau of Indian Affairs VDP) — n/a (P5, informational)
- Source: [Bugcrowd 4abc50f7](https://bugcrowd.com/disclosures/4abc50f7-2400-475f-b126-822b9c19cea3/host-header-injection)
- Type: Security misconfiguration (unvalidated `Host` header)
- Summary: A `*.bia.gov` host accepted an attacker-supplied `Host` header and let it influence application behaviour. Triaged P5 and closed as informational — an accepted business risk for this target.
- Technique / pattern: Send the request with a `Host` header pointing at your own domain (and separately with `X-Forwarded-Host`), then look for where that value resurfaces: absolute URLs in the body, cache keys, redirects and — most valuably — password-reset links. What moves this class off P5 is a demonstrated consequence, not the reflection itself.
- Takeaway: Validate `Host` against an allowlist of expected names and build absolute URLs from configuration rather than from the request. Worth noting how triage reasons here: informational without impact, high once it poisons a reset link or a cache entry.

### 2026-10-20 — Identity theft via broken link hijacking on a NASA astronaut news release page (NASA VDP) — n/a (P4)
- Source: [Bugcrowd 98a08457](https://bugcrowd.com/disclosures/98a08457-4f9d-4d27-b7e2-daaa975a1a67/identity-theft-via-broken-link-hijacking-on-nasa-astronaut-news-release-page)
- Type: Security misconfiguration (broken link hijacking / dangling social handle)
- Summary: A NASA news release page linked to an official social media handle that had since been released, so anyone could register that handle and inherit the trust carried by a link from nasa.gov. Resolved, rated P4.
- Technique / pattern: Crawl the target's pages for outbound social and third-party links, then check each destination for a 404 or an available handle. The exploit is a registration rather than a request, which is why these turn up with a link checker plus a scripted availability lookup instead of a scanner.
- Takeaway: Outbound links are part of the attack surface — audit them on a schedule, remove references to retired accounts, and retain ownership of handles that official pages still point at.

### 2026-10-20 — Public Google Drive linked in a NASA ARSET training PDF allowed unauthenticated file modification and deletion (NASA VDP) — n/a (P4)
- Source: [Bugcrowd 24753086](https://bugcrowd.com/disclosures/24753086-60a4-4597-825d-6613faa35306/public-google-drive-linked-in-nasa-arset-training-pdf-allows-unauthenticated-users-to-modify-and-delete-files)
- Type: Security misconfiguration (third-party storage sharing scope)
- Summary: A training PDF published on a NASA site linked to a Google Drive location shared with edit rather than view rights, so anyone holding the link could change or delete the hosted files. NASA remediated by removing the link from the site; rated P4.
- Technique / pattern: Treat published documents as a link source — pull the PDFs, decks and spreadsheets a target hosts, extract every embedded URL, then check the sharing scope of each cloud-storage destination. "Anyone with the link can edit" is the finding, and it is invisible from the HTML alone.
- Takeaway: Links inside published documents inherit the publisher's authority, so review the permission scope of every externally shared folder and default to view-only for anything referenced publicly.

### 2026-10-18 — Unauthenticated go-httpbin instances on an AWS domain enable cookie tossing and open redirect (AWS VDP) — n/a
- Source: [HackerOne #3966599](https://hackerone.com/reports/3966599)
- Type: Security misconfiguration — a request-reflection test service exposed unauthenticated on a trusted parent domain
- Summary: Unauthenticated instances of the `go-httpbin` request-reflection service were found running on hosts under `aws.dev`. Because the service sets arbitrary cookies and issues arbitrary redirects on request, its presence on a shared parent domain produced cookie tossing against `*.aws.dev` siblings plus an open redirect on an AWS-owned origin. Rated Medium and resolved.
- Technique / pattern: The bug is not in the tool, it is in where the tool lives. `httpbin`-style echo services, request bins and debug reflectors expose "set this cookie", "redirect to this URL" and "echo these headers" as features; placed on a subdomain whose siblings share cookie scope, those features become attacks, because a cookie written for the parent domain is sent to every sibling and can overwrite or fixate session, CSRF or preference cookies there. During recon, fingerprint every subdomain for known echo and debug software, then ask which sibling origins share cookie scope and which clients trust the parent domain by hostname.
- Takeaway: Keep test, debug and reflection services off any domain that hosts real sessions — give them an isolated apex rather than a subdomain. Use `__Host-` prefixed cookies so siblings cannot write them, and never treat "this URL is on our domain" as a trust decision, because one reflector or open redirect anywhere on the estate invalidates every such check.

### 2026-10-18 — Content spoofing via broken link hijacking of the ENERGYSTARHomes handle (US EPA VDP) — n/a (P4)
- Source: [Bugcrowd #598e8a2c](https://bugcrowd.com/disclosures/598e8a2c-6c6b-4fe7-9b76-8eb06283cfd5/content-spoofing-via-broken-link-hijacking-energystarhomes)
- Type: Security misconfiguration — dangling outbound social-media reference enabling impersonation
- Summary: `www.energystar.gov` linked to the social handle `@ENERGYSTARHomes`, which was unclaimed. Anyone could register the handle and publish content that an official government page vouches for by linking to it. Reported to the EPA VDP, rated P4, accepted and fixed.
- Technique / pattern: Crawl the target's own pages for outbound links to platforms where identifiers are first-come-first-served — social handles, app-store listings, package names, link shorteners, community channels — then resolve each and flag any that return not-found, say the account does not exist, or are otherwise still registrable. Government and large-institution estates are productive because programme microsites are created per campaign and outlive the team that ran them, so the handle is renamed or retired while the page keeps pointing at the old name.
- Takeaway: Outbound links are part of the attack surface, and the impersonation inherits the authority of the page that links to it. Treat the inventory of referenced third-party identifiers as an asset list, reserve the handles you cite, and add a periodic link-health scan that flags unresolvable destinations rather than only broken internal links.

### 2026-10-18 — Dangling social-media link lets the NASAOSBP handle be claimed and impersonated (NASA VDP) — n/a (P4)
- Source: [Bugcrowd #f6341af5](https://bugcrowd.com/disclosures/f6341af5-c9da-4e29-893f-8d66c4d18907/dangling-social-media-link-takeover-broken-link-hijack-on-nasaosbp)
- Type: Security misconfiguration — abandoned social handle referenced from official assets
- Summary: NASA Office of Small Business Programs assets referenced the `NASAOSBP` Twitter/X handle, which was unclaimed. The researcher demonstrated impact by registering the handle, showing that an attacker could publish under what appears to be an official NASA channel. Rated P4 and remediated.
- Technique / pattern: The distinguishing detail from the generic case is the proof step: for a dangling *outbound* reference the demonstration is claiming the identifier, which is why scope and restraint matter — claim it, show control, release it, and publish nothing from it. Prioritise handles referenced from pages that confer authority: press, procurement, recruiting, security-contact pages.
- Takeaway: An organization's identity perimeter includes names it does not host. Keep a register of every third-party identifier cited anywhere on the estate, keep those registrations alive even for retired programmes, and release accounts only in the same change that removes the references.

### 2026-10-18 — Expired official handle referenced from the NASA Earth Matters blog enables impersonation (NASA VDP) — n/a (P4)
- Source: [Bugcrowd #081a696d](https://bugcrowd.com/disclosures/081a696d-d1d2-4d7d-acb7-29b313c9aa9b/impersonation-via-broken-link-hijacking-on-nasa-earth-matters-blog-page)
- Type: Security misconfiguration — broken link hijacking on a content/blog surface
- Summary: A NASA Earth Matters blog page referenced an expired official social-media handle. Because the handle had lapsed it could be registered by anyone and used to impersonate NASA from a link an official blog still carried. Rated P4, validated and patched.
- Technique / pattern: Blogs, newsrooms and archived article pages are the densest source of this issue and the least likely to be re-reviewed, because posts are published once and then treated as immutable history. Scrape the full archive rather than the current navigation — sitemaps, pagination and tag listings reach posts the menus no longer link — and resolve every outbound identifier found in old posts.
- Takeaway: Content that is never edited again still carries live trust. Include archived and paginated content in link-health scanning, and when retiring an account, search the estate's historical content for references to it as part of decommissioning.

### 2026-10-18 — macOS screen-share permission dialog overlaps ChatGPT Atlas's full-screen notification, enabling UI spoofing (OpenAI) — $200 (P4)
- Source: [Bugcrowd #44e37fb3](https://bugcrowd.com/disclosures/44e37fb3-3a52-4d7d-8429-c3c78b77f514/atlas-browser-fullscreen-notification-overlap-by-screen-share-permission-dialog-leading-to-ui-spoofing-and-unauthorized-screen-capture-in-chatgpt-atlas-for-macos)
- Type: Security misconfiguration / UI redressing — a security indicator obscured by a native permission prompt
- Summary: In ChatGPT Atlas for macOS, a page in full-screen mode could trigger the operating system's screen-recording permission request, and that native dialog could overlap or obscure the browser's own full-screen notification. A security-sensitive permission request could therefore be made to look as though it belonged to a different, benign action. Rated P4, rewarded $200, accepted and fixed.
- Technique / pattern: The target is the *indicator*, not the permission. Full-screen notifications, address-bar origin chips, "a site is using your camera" badges and download prompts are the user's only evidence of what is happening, so anything that covers, delays or races them converts an informed click into a blind one. The probe is systematic: enter full-screen from page content, then fire every prompt-producing API — screen capture, camera, microphone, notifications, geolocation, external-protocol open, file picker — and observe whether the indicator stays visible, legible and on top. Timing matters as much as geometry, since an indicator that fades on a timer is as good as covered.
- Takeaway: Treat security indicators as a surface with their own threat model in any embedded browser or desktop client: they must render above page and dialog content, persist for as long as the state they describe is active, and suppress the sensitive action when they cannot be shown.

### 2026-10-16 — Case-sensitive scheme check on `app.hey.com/proxy_image` turns into an open redirect that defeats the HEY desktop origin allowlist (Basecamp) — n/a
- Source: [HackerOne #3972760](https://hackerone.com/reports/3972760)
- Type: Security misconfiguration — open redirect / improper URL parsing bypassing a client-side origin allowlist
- Summary: `app.hey.com/proxy_image` compared the submitted URL's scheme case-sensitively. A lowercase `https://` target was signed and proxied correctly, but `HTTPS://` was not recognised and the endpoint issued a 302 straight to the raw URL — an unauthenticated open redirect. Because the HEY desktop app used "is this URL on `app.hey.com`?" as its test for trusted internal content, an emailed link could open attacker content inside a chrome-less HEY `BrowserWindow`.
- Technique / pattern: RFC 3986 makes URL schemes case-insensitive, so `HTTPS://` is a valid URL rather than a malformed edge case; the filter was still correctly rejecting `javascript:`, which proved the gap was the comparison, not a missing denylist. The researcher then chained the primitive: a host-based trust decision in the desktop client was satisfied by a URL that genuinely lived on the trusted host but did not stay there.
- Takeaway: Normalise scheme and host (case-fold, resolve) before any security comparison, and test allowlists with case variants, trailing dots and encoded forms. More broadly, never decide "is this our own content" from the hostname of the *initial* URL — an open redirect on a trusted origin converts every host-based allowlist in every client into a content-injection vector.

### 2026-10-16 — Unsafe nginx regex in `/colibri-relay-ws/` turns docker-jitsi-meet into an open proxy (8x8 / Jitsi) — n/a
- Source: [HackerOne #3889473](https://hackerone.com/reports/3889473)
- Type: Security misconfiguration — overly permissive nginx location regex feeding `proxy_pass`
- Summary: In `docker-jitsi-meet`, the nginx location for `/colibri-relay-ws/` used the pattern `[a-zA-Z0-9-\._]+` to capture a relay target and interpolated it into a `proxy_pass` directive. With `ENABLE_OCTO` enabled, that character class accepted arbitrary domain names and IP addresses, so unauthenticated requests could be proxied to attacker-specified destinations. The deprecated location and relay websocket proxy config were removed in the fix.
- Technique / pattern: The finding comes from reading deployment configuration rather than the application: grep an infrastructure repo for `proxy_pass` directives that contain a captured variable, then check whether the capturing regex is actually constrained to known hosts. A class permitting letters, digits, dots and hyphens matches any hostname, so the "validation" is cosmetic. Feature flags are part of the attack surface — the path only existed when `ENABLE_OCTO` was on.
- Takeaway: Audit reverse-proxy configs as source code. A variable in `proxy_pass` must resolve against an explicit map or upstream allow-list, never a permissive character class, otherwise the host becomes an SSRF/open-proxy pivot into the internal network. Deprecated-but-still-shipped routes are worth checking first.

### 2026-10-16 — Exposed AWS Cognito pool identifiers on a NASA login page allow unauthenticated user creation (NASA VDP) — n/a
- Source: [Bugcrowd #cbe0edab](https://bugcrowd.com/disclosures/cbe0edab-82a9-46ae-9ed3-f3a0b30d1cca/unauthenticated-user-creation-in-aws-cognito-user-pool)
- Type: Security misconfiguration — client-side secret exposure plus permissive Cognito self-sign-up (P3)
- Summary: The page at `loginapp.prosams.nasa.gov/login-legal-notices` exposed AWS Cognito configuration values, including `UserPoolId` and `UserPoolWebClientId`. Although the web interface offered no registration, those identifiers were enough to call the Cognito API directly and successfully create a new user in the production user pool.
- Technique / pattern: Read the login page's client-side configuration (inline JS, bundled config objects, `/env.json`-style endpoints) for identity-provider identifiers, then test the provider's API directly rather than the app's UI — here `aws cognito-idp sign-up --client-id <clientId> --username <email> --password <pw>`. Absence of a registration form in the SPA says nothing about whether the pool has self-sign-up enabled.
- Takeaway: Cognito pool and client IDs are not secrets, so the real control is the pool's own configuration: disable self-service sign-up when registration is meant to be admin-only, and restrict the app client's allowed auth flows. Whenever an app front-end reveals a managed identity service's identifiers, test that service's API surface directly for flows the UI never exposes.

### 2026-10-16 — CSRF token accepted as the upload-permission token enables unauthenticated writes to a NASA production GCS bucket (NASA VDP) — n/a
- Source: [Bugcrowd #817dab3c](https://bugcrowd.com/disclosures/817dab3c-ec4a-45b6-b855-c8d44b18d6af/broken-access-control-csrf-token-accepted-as-upload-permission-token-unauthenticated-file-upload-to-nasa-production-google-cloud-storage-adapt-fellowship-portal)
- Type: Security misconfiguration / broken access control — two distinct tokens sharing one signer (P4)
- Summary: The ADAPT fellowship portal let anonymous users obtain a CSRF token from `POST /upload/refresh-csrf-token`. The upload endpoint `POST /upload/generate-resume-url` required both `csrf_token` and `upload_token`, but both were validated by the same signer and `upload_token` was not bound to any form submission. Submitting the CSRF token in both fields returned a valid signed PUT URL for the production bucket, allowing unauthenticated file writes into NASA's HR pipeline.
- Technique / pattern: When a request carries two different tokens, try swapping one for the other. A shared HMAC signer with no type, audience or purpose claim makes any token it issued interchangeable, so the anonymous-by-design CSRF token satisfies the gate meant to prove a completed application. The researcher confirmed end to end by PUTting a PDF to the returned signed URL and getting a 200.
- Takeaway: Tokens must be bound to their purpose — distinct keys, or a signed `purpose`/`aud` claim plus the specific resource and submission they authorize. Treat every endpoint that mints a cloud-storage pre-signed URL as a privileged operation and verify what actually gates it, since the signed URL carries the service's own authority.

### 2026-10-15 — Unauthenticated SSRF in NASA Trek addManifest allows internal network access from the Trek server (NASA VDP) — n/a (P2)
- Source: [Bugcrowd #652ddcf8](https://bugcrowd.com/disclosures/652ddcf8-f52a-44f5-8a72-2e4a61f7c9ce/unauthenticated-ssrf-in-nasa-trek-addmanifest-allows-internal-network-access-from-the-trek-server)
- Type: Server-side request forgery (unauthenticated endpoint, no egress controls)
- Summary: The outreach endpoint `/moon/TrekServices/ws/outreach/eq/addManifest` accepted a caller-supplied URL in the `manifestURL` parameter and fetched and parsed it with no authentication, allowlist or egress restriction (CVSS 3.1 base 8.6), so an external requester could make the server reach its own loopback interface and an internal-only backend host. The reporter limited testing to proving reachability.
- Technique / pattern: Grep API documentation and front-end JavaScript for parameters whose names end in `URL`, `uri`, `src`, `callback` or `manifest`, then point them at loopback and internal hostnames and read the outcome from response differences and timing.
- Takeaway: A fetch-a-URL feature needs authentication plus a destination allowlist and a network egress policy; an "internal only" backend is reachable the moment one public service will proxy to it.

### 2026-10-15 — Insecure Google Drive permissions allow complete deletion of NASA SEPVAL 2023 event data (NASA VDP) — n/a (P3)
- Source: [Bugcrowd #5e76bccc](https://bugcrowd.com/disclosures/5e76bccc-1df1-4403-8532-b289f19659b0/insecure-google-drive-permissions-allow-complete-deletion-of-nasa-sepval-2023-event-data)
- Type: Third-party sharing misconfiguration (integrity and availability loss)
- Summary: Google Drive files for a NASA event were shared with permissions broader than read-only, so anyone holding the link could act beyond intended read access — including deleting the event documents — without authenticating.
- Technique / pattern: Follow links from the in-scope site out to SaaS document hosts and check the effective permission rather than only whether the file opens; a link-shared file that grants editor access is an unauthenticated write primitive.
- Takeaway: Link-sharing defaults are part of the attack surface — review every "anyone with the link" grant for the role attached, because integrity and availability loss need no credential at all.

### 2026-10-15 — Subdomain takeover in help.tictail.com pointing to Zendesk, a Shopify acquisition (Shopify) — n/a
- Source: [HackerOne #869605](https://hackerone.com/reports/869605)
- Type: Subdomain takeover (dangling CNAME to a SaaS provider)
- Summary: The subdomain `help.tictail.com`, belonging to a Shopify acquisition, still had a CNAME pointing at `tictail.zendesk.com` after the Zendesk account was gone, and the reporter confirmed during Zendesk signup that the target name was free to claim.
- Technique / pattern: Enumerate DNS records of acquired brands, resolve each CNAME, and flag any that resolve to a SaaS provider's host; confirm by checking whether the provider still has that subdomain name available, without claiming production assets.
- Takeaway: Acquisitions leave orphaned DNS behind long after the SaaS subscriptions lapse — decommissioning has to remove the DNS record, not just the vendor account.

### 2026-10-15 — Subdomain takeover via an insecure CloudFront distribution on cdn.grab.com (Grab) — $1,000
- Source: [HackerOne #352869](https://hackerone.com/reports/352869)
- Type: Subdomain takeover (unclaimed CDN alternate domain name)
- Summary: `cdn.grab.com` had a CNAME to a CloudFront hostname that was not registered as an alternate domain name in Grab's own CloudFront account, so the reporter could claim the name in their own distribution and serve a proof-of-concept page from the Grab subdomain. The first recommended fix was simply removing the CNAME record.
- Technique / pattern: For CNAMEs that end in a cloud CDN domain, test whether the alternate-domain claim is actually held by the owner — an unclaimed alias on a CDN lets any account bind the name to its own distribution.
- Takeaway: CDN aliases are claimed per-distribution, so a CNAME pointing at a CDN with no matching alias registration is a takeover waiting to happen.

### 2026-10-12 — LiteLLM proxy credential exposed through WorldWind configuration (NASA VDP) — n/a
- Source: [Bugcrowd #c83e7c95](https://bugcrowd.com/disclosures/c83e7c95-3fb9-45c6-9e22-2464e84aba3a/litellm-proxy-credential-exposed-through-worldwind-configuration)
- Type: Security misconfiguration — credential exposed in client-reachable config
- Summary: A live LiteLLM proxy credential was exposed through a publicly reachable NASA WorldWind configuration surface, handing out a usable secret for the LLM proxy.
- Technique / pattern: A configuration served to clients embedded a live proxy credential; simply reading the config yielded the key directly.
- Takeaway: Secrets must never ship in client-reachable configuration; treat any config endpoint as public and keep credentials server-side only.

### 2026-10-12 — Sensitive information disclosure (CVE-2025-61138) in Qlik Sense Dev Hub (EPA VDP) — n/a
- Source: [Bugcrowd #7e6fce2c](https://bugcrowd.com/disclosures/7e6fce2c-4cae-42cc-b402-eae4284a791c/sensitive-information-disclosure-cve-2025-61138-in-qlik-sense-dev-hub)
- Type: Security misconfiguration — exposed dev interface (known CVE)
- Summary: A Qlik Sense Dev Hub instance under `*.epa.gov` was publicly reachable and affected by CVE-2025-61138, leaking sensitive information that should not have been exposed.
- Technique / pattern: A development/management hub was left internet-facing on a version matching a disclosed CVE, so unauthenticated requests returned configuration/information details.
- Takeaway: Keep dev/admin hubs off the public internet and patched; an exposed `Dev Hub` is both an information leak and an entry point when it matches a public CVE.

### 2026-10-12 — Content spoofing via broken link hijacking on www.nasa.gov (NASA VDP) — n/a
- Source: [Bugcrowd #91812b4b](https://bugcrowd.com/disclosures/91812b4b-29a0-4558-950a-1e914d4e73ef/content-spoofing-via-broken-link-hijacking-on-www-nasa-gov)
- Type: Security misconfiguration — broken link hijacking / content spoofing
- Summary: A live page on `www.nasa.gov` linked to an external/abandoned resource whose destination could be claimed by an attacker, allowing content spoofing under the trust of the official page.
- Technique / pattern: An outbound link or embedded resource pointed to an unregistered/expired external handle; claiming it lets the attacker place arbitrary content reached from the trusted page.
- Takeaway: Periodically audit outbound links and embedded third-party resources; abandoned destinations become impersonation and spoofing channels that borrow your domain's trust.

### 2026-10-12 — Confirmed S3 bucket takeover of ASF Cumulus production buckets (NASA VDP) — n/a
- Source: [Bugcrowd #78b51a29](https://bugcrowd.com/disclosures/78b51a29-4aa4-4dad-9d2a-e5356c482f4a/confirmed-s3-bucket-takeover-of-asf-cumulus-production-buckets-asf-cumulus-prod-browse-asf-cumulus-prod-opera-product)
- Type: Security misconfiguration — dangling S3 bucket takeover
- Summary: Production ASF Cumulus S3 buckets referenced by NASA infrastructure (e.g. `asf-cumulus-prod-browse`) were unclaimed and re-registerable, so an attacker could claim the names and serve content from buckets the application trusted.
- Technique / pattern: The application pointed at S3 bucket names that no longer existed in the owner's account; because bucket names are globally unique, an attacker claims the free name and controls whatever the site loads from it.
- Takeaway: Remove or re-point application references to decommissioned cloud storage; globally-unique bucket names left dangling are claimable by anyone.

### 2026-10-09 — Config files with VPN pre-shared key and other credentials exposed by directory indexing (Tesla) — $10,000
- Source: [Bugcrowd #bdc4fb5a](https://bugcrowd.com/disclosures/bdc4fb5a-ac99-4c3e-866f-37b3f58a0fa3/config-files-with-vpn-pre-shared-key-and-other-credentials-in-them)
- Type: Security misconfiguration (directory indexing enabled) leading to credential exposure
- Summary: An nginx server on a Tesla subdomain had directory indexing enabled, exposing a browsable directory containing a zip archive of RADIUS passwords, network-equipment administrator passwords, store Wi-Fi passwords, the IKE pre-shared key for corporate VPN access, and real-estate financial data. Rated P1 and resolved; disclosed 2021-02-26.
- Technique / pattern: Entirely passive: enumerate subdomains, request directory paths rather than only files, and read what the index page lists. No authentication bypass and no payload — the web server volunteered its own file listing, and the archive was downloadable anonymously.
- Takeaway: Directory indexing turns every forgotten upload into a public one. When reviewing a host, request bare directory paths (not just known filenames) and treat archive extensions as the highest-value hit; one misconfigured directive produced a P1 and a $10,000 payout.

### 2026-10-09 — Wildcard allowlist bypass leads to full SSRF in Jira Data Center (Atlassian) — n/a
- Source: [Bugcrowd #8ceda9d9](https://bugcrowd.com/disclosures/8ceda9d9-7f87-4621-84f8-76b9262bbd96/bypass-of-wildcard-allowlist-which-leads-to-full-ssrf)
- Type: Security misconfiguration / SSRF (allowlist parsing flaw)
- Summary: In Jira 8.20.6 the gadget proxy endpoint `/plugins/servlet/gadgets/makeRequest` forwarded requests with arbitrary HTTP methods and custom headers, and the configured wildcard allowlist entry could be satisfied by placing the allowlisted string in a query parameter instead of the host. Rated P4 (VRT: Broken Access Control > SSRF > Internal High Impact); submitted 2022-05-18 and disclosed 2025-07-14.
- Technique / pattern: The allowlist was a wildcard expression over a domain the researcher controlled, documented as matching "one or more characters". A direct request to an out-of-band catcher returned 403, but a URL of the shape `https://<attacker-domain>?r=<allowlisted-host>` passed the check and returned 200 OK with the response body and headers visible — a full-read SSRF. The researcher suggested this bypassed the fix for CVE-2019-8451 and noted Confluence might share the flaw.
- Takeaway: Allowlist checks that substring-match a URL rather than parsing it and comparing the authority are bypassable by putting the trusted value in the query string, path or userinfo. Always test the 403-versus-200 difference, and check whether a past CVE's patch validates the host or merely searches the URL.

### 2026-10-09 — Content-Security-Policy script-src includes unsafe-inline and unsafe-eval (Asana) — n/a
- Source: [Bugcrowd #e7da9ecf](https://bugcrowd.com/disclosures/e7da9ecf-e470-4f75-8495-a29c36b1888a/csp-parameter-script-src-includes-unsafe-inline)
- Type: Security misconfiguration (weak security headers)
- Summary: The `Content-Security-Policy` response header on `asana.com` listed `'unsafe-inline'` and `'unsafe-eval'` in its `script-src` directive alongside an allowlist of third-party domains, weakening CSP as a defence against injected script. Rated P5 and accepted as a business risk; submitted 2021-04-25 and disclosed 2021-05-19.
- Technique / pattern: A pure header review, reproducible in three steps: issue a request to the root URL and capture the response headers, locate the `Content-Security-Policy` header, and read the `script-src` directive for unsafe keywords and overly broad host sources. Full response headers plus a screenshot were attached as evidence.
- Takeaway: Header hygiene findings are almost always P5 or informational on their own — the control is weakened, not broken, and no exploit is shown. Their value is as a multiplier: record the weak directive, then look for an injection point it would otherwise have contained, and report the chain.

### 2026-10-09 — Authentication bypass and Apache Tomcat misconfiguration on a pilot host (8x8) — n/a
- Source: [HackerOne #1364022](https://hackerone.com/reports/1364022)
- Type: Security misconfiguration (exposed management interfaces)
- Summary: A single host in 8x8's pilot environment exposed the Apache Tomcat `/admin` and `/manager` endpoints; access was subsequently restricted. Rated Medium with no bounty; submitted 2021-10-08 and disclosed 2021-11-04.
- Technique / pattern: Enumerate well-known administrative paths on every host found, not just the primary application. Tomcat's `/manager` and `/admin` contexts are default-shipped management surfaces whose presence indicates the application server is directly reachable rather than fronted by a proxy that strips them; pilot, staging and pre-production hosts routinely skip the hardening the production fleet receives.
- Takeaway: Non-production hosts in scope are often the weakest asset a program owns. A short wordlist of default management paths against every discovered host costs almost nothing and finds what deployment checklists missed.

### 2026-10-09 — Misconfigured S3 bucket found via JavaScript references exposes a "blocked" docs folder (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #998981](https://hackerone.com/reports/998981)
- Type: Cloud storage misconfiguration (public S3 bucket)
- Summary: A DoD site appeared to block access to a documents folder at the web tier, but the folder's contents were actually hosted in an Amazon S3 bucket that was readable by anyone. The researcher located the bucket by reading the site's JavaScript files.
- Technique / pattern: Front-end JavaScript is the cheapest source-code disclosure you get for free: pull every script and source map, and grep for bucket names, `s3.amazonaws.com`/`storage.googleapis.com` hosts, CDN origins and API base paths. Then test the storage origin directly — a 403 from the application does not mean the backing store is private.
- Takeaway: Access control enforced only at the application or CDN layer is bypassed by addressing the origin store directly. Enumerate the asset origins behind every "forbidden" path, and on the defensive side make buckets private and serve through signed URLs.

### 2026-10-09 — CORS misconfiguration reflects arbitrary Origin with credentials allowed (Clario) — n/a
- Source: [HackerOne #731472](https://hackerone.com/reports/731472)
- Type: CORS misconfiguration
- Summary: A GET request carrying an arbitrary `Origin` header came back with that same origin echoed in `Access-Control-Allow-Origin` together with `Access-Control-Allow-Credentials: true` — the combination that lets any attacker-controlled site read authenticated responses cross-origin. Reported against a Kromtech/Clario translate host.
- Technique / pattern: One-line test on every API host: resend a request with `Origin: https://evil.example` and inspect the response. Reflected origin plus `Allow-Credentials: true` is exploitable; a wildcard `*` with credentials is rejected by browsers, and a reflected-but-credential-less origin is usually low impact. Also try null origin, subdomain and suffix/prefix tricks (`target.com.evil.com`, `eviltarget.com`) against sloppy allow-list regexes.
- Takeaway: Never build `Access-Control-Allow-Origin` by reflecting the request header. Keep a static allow-list, and when credentials are permitted treat the allow-list as a security control with the same scrutiny as authentication.

### 2026-10-09 — Internal RFC1918 address exposed through a public DNS record (NASA VDP (Bugcrowd)) — n/a (P5, informational)
- Source: [Bugcrowd #33889902](https://bugcrowd.com/disclosures/33889902-f5bd-4346-93c0-d66c82a74506/internal-ip-disclosure-via-public-dns-record-blue-guest-hq-nasa-gov)
- Type: Information-leaking DNS misconfiguration
- Summary: A public DNS record for `blue.guest.hq.nasa.gov` resolved to the private address `192.168.50.4`, publishing internal addressing to anyone who queried it. The researcher argued it aids network mapping and could support DNS rebinding if chained. NASA closed it as an accepted business risk (site decommissioned); disclosed 2025-10-02.
- Technique / pattern: Bulk-resolve every subdomain you enumerate and filter the answers for RFC1918 / loopback / link-local ranges. These records are also leads: a hostname that resolves internally often has a sibling that resolves externally, and a public name pointing at `127.0.0.1` or a private range is the precondition for DNS-rebinding and SSRF-allow-list bypass work.
- Takeaway: Keep internal zones on internal resolvers. Treat this as a reconnaissance finding rather than a standalone bug — report it with the chain it enables, or expect P5/informational.

### 2026-10-09 — Directory listing enabled on maps.nccs.nasa.gov (NASA VDP (Bugcrowd)) — n/a (P5, informational)
- Source: [Bugcrowd #db8806c9](https://bugcrowd.com/disclosures/db8806c9-cdf1-4d3a-87f9-6fabe5bd1e47/information-exposure-through-directory-listing-vulnerability-on-https-maps-nccs-nasa-gov)
- Type: Security misconfiguration — directory indexing
- Summary: The web server on `maps.nccs.nasa.gov` returned auto-generated directory indexes, letting anyone browse the file listing of the affected paths. Triaged P5 / accepted business risk; disclosed 2024-08-30.
- Technique / pattern: Request every discovered directory with a trailing slash and compare against the configured index document; `Index of /` in the body is the tell. The value is in what the listing then reveals — backups, archives, `.sql`/`.bak`/`.zip` dumps, credentials in config files — so always triage the contents before writing it up.
- Takeaway: Directory indexing on its own is routinely closed as informational. It only becomes a real finding when the listing itself exposes something sensitive, so lead the report with the file you found, not with the listing.

### 2026-10-07 — S3 bucket takeover on learn2.khanacademy.org (Khan Academy) — n/a
- Source: [HackerOne #1777077](https://hackerone.com/reports/1777077)
- Type: Dangling DNS record / subdomain takeover
- Summary: The subdomain `learn2.khanacademy.org` pointed at Amazon S3, but no bucket with the matching name existed — so anyone could register that bucket and serve arbitrary content from a khanacademy.org hostname.
- Technique / pattern: Standard dangling-record hunt: resolve every subdomain, keep the ones whose target is a cloud service, and check whether the backing resource still exists. Here the tell was the S3 error response for an unregistered bucket, visible by simply loading `http://learn2.khanacademy.org`.
- Takeaway: Decommissioning an app means removing its DNS record, not just its bucket — and the two are usually owned by different teams, which is why this class persists. Content served from a real subdomain inherits the organisation's trust and, depending on cookie scope, its session reach.

### 2026-10-07 — Open redirect on a NASA Keycloak staging instance (NASA VDP) — n/a (P4)
- Source: [Bugcrowd #d69f5c9e](https://bugcrowd.com/disclosures/d69f5c9e-d5e4-40bb-94f8-bc6cd1d14bcf/open-redirect-vulnerability-occurring-at-https-keycloak-shared-services-staging-appdat-jsc-nasa-gov)
- Type: Open redirect on an identity provider
- Summary: A NASA-hosted Keycloak authentication instance in a staging environment accepted an attacker-controlled redirect target, sending authenticated users to an external site.
- Technique / pattern: The target was found by enumerating subdomains and noticing a staging identity provider exposed to the internet. On an IdP, the redirect parameter (the post-login return URL) is the natural place to test, since a permissive allowlist there is both common and more impactful than a redirect on a content page.
- Takeaway: Open redirects matter most on login and SSO endpoints: the redirect fires at the moment the user has just authenticated, which makes credential-phishing and token-leak chains plausible. Staging instances of identity infrastructure are worth enumerating specifically — they are frequently configured with looser allowlists than production.

### 2026-10-07 — Clickjacking exposing sensitive information through iframes (StackPath) — n/a (P4, 5 points)
- Source: [Bugcrowd #812ca668](https://bugcrowd.com/disclosures/812ca668-217a-4b4d-aed1-8d92c3f26846/clickjacking-exposing-sensitive-information-through-iframes)
- Type: Missing framing protection (clickjacking / UI redressing)
- Summary: Pages containing sensitive information could be embedded in a third-party iframe because the application sent no framing restrictions, enabling UI-redressing attacks.
- Technique / pattern: The check is a one-liner: load the page inside an attacker-hosted iframe and see whether it renders. It renders when neither `X-Frame-Options` nor a CSP `frame-ancestors` directive is present on the response.
- Takeaway: Framing protection is a response-header property, so it has to be verified per route — a header set at the edge for the main app often misses legacy paths, error pages and embedded widgets. Impact is driven by what the framed page can do in one click, so prioritise pages with a destructive or state-changing control.

### 2026-10-07 — Directory listing exposure (NASA VDP) — n/a (P5, accepted risk)
- Source: [Bugcrowd #0e4d1410](https://bugcrowd.com/disclosures/0e4d1410-da5a-4aff-b0ad-3e402199ac9b/directory-listing)
- Type: Directory listing enabled
- Summary: A NASA web path had automatic directory indexing enabled, exposing the contents of a directory to anyone who requested it. NASA accepted the finding as informational business risk.
- Technique / pattern: Found by requesting directory paths rather than files — trimming a known file's URL back to its parent path and checking whether the server returns an index instead of a 403 or 404. Cheap to test across many hosts once a subdomain list exists.
- Takeaway: Directory listing is rarely the finding in itself; its value is as a discovery primitive, because the index reveals backup files, config files and older versions you would otherwise have to guess. Chase what the listing exposes before reporting the listing alone — that is usually what decides severity.

### 2026-10-07 — WordPress takeover via a reachable setup-config.php pointed at an attacker's database (U.S. Dept Of Defense) — $1,000
- Source: [HackerOne #1626205](https://hackerone.com/reports/1626205)
- Type: Misconfiguration — exposed installer leading to administrative takeover
- Summary: A leftover WordPress tree exposed `/old/wp-admin/setup-config.php`, the installer page that collects database details; because the installer accepts any database server it can reach, the reporter supplied a free externally hosted MySQL instance of their own, completed installation, and obtained WordPress administrator access on the `.edu` host without ever knowing a target credential.
- Technique / pattern: Enumerate leftover and staging paths (`/old/`, `/new/`, `/backup/`, `/wp/`) for installer and setup endpoints of any stack — WordPress `setup-config.php`, framework install routes, database admin setup scripts — and remember that a credential prompt is not a gate when the credentials may point outward; CMS admin then means plugin or theme upload and code execution.
- Takeaway: An unfinished or re-exposed installer is a full compromise rather than an information leak — block setup routes after installation and delete abandoned copies of the application instead of leaving them unlinked.

### 2026-10-07 — Link hijacking via an encoded slash yields an open redirect on accounts.yoyogames.com (Opera) — $100
- Source: [Bugcrowd #73a1a65b](https://bugcrowd.com/disclosures/73a1a65b-7c95-4b0a-9791-e0f9a48a0fe7/link-hijacking-leads-to-open-redirection-on-accounts-yoyogames-com)
- Type: Misconfiguration — unvalidated redirect through URL normalization (GET-based, P4)
- Summary: Requesting a crafted path such as `https://accounts.yoyogames.com//%2fevil.com/test/` caused the page's logout navigation link to be rendered with an attacker-supplied external host in its `href`, so opening that link in a new tab sent the user to the attacker's domain.
- Technique / pattern: Attack the link-*building* code rather than a redirect parameter — when a template constructs navigation hrefs from the current request path, a leading `//` plus `%2f` makes a relative link resolve as protocol-relative and absolute; inspect rendered anchors for your injected host, and note the exploitation quirk that a plain click did not redirect while opening in a new tab did.
- Takeaway: Build links from server-side route definitions rather than from the incoming path, and normalize or reject encoded separators before any URL is echoed into markup.

### 2026-10-07 — Open redirect through the browser extension's Network Lock page (ExpressVPN) — n/a (5 points, P4)
- Source: [Bugcrowd #62ce59a6](https://bugcrowd.com/disclosures/62ce59a6-9016-4efd-b52b-f0e6cf686775/open-redirect-via-chrome-extension)
- Type: Misconfiguration — unvalidated redirect inside an extension page
- Summary: The ExpressVPN Chrome extension's Network Lock page could be driven to redirect to an arbitrary external page by manipulating the extension-scoped URL of the form `chrome://<browser-extension-id>/..`; the vendor rated it low because the unusual URL shape makes convincing social engineering unlikely, and fixed it in extension version 4.9.6.4116.
- Technique / pattern: Extension pages are their own origin with their own routing and routinely skip the validation applied to website redirects — enumerate `web_accessible_resources` and internal pages from the manifest, then test each page's URL parameters for redirect and injection sinks.
- Takeaway: Validate redirect targets inside extension pages as well, and expect triage to weigh how plausible the delivery URL looks rather than only whether the redirect works.

### 2026-10-07 — TLSv1.3 disabled on caps.gsfc.nasa.gov (NASA VDP) — n/a (P5, informational)
- Source: [Bugcrowd #3367a84c](https://bugcrowd.com/disclosures/3367a84c-0d94-45a3-9120-a270e247b004/ssl-tls-configuration-issue-on-caps-gsfc-nasa-gov)
- Type: Misconfiguration — transport security configuration
- Summary: The host negotiated only TLSv1.2 with TLSv1.3 disabled; ciphers were secure and the certificate valid, so the program closed it as informational, noting the certificate had been updated and met current NASA requirements.
- Technique / pattern: Protocol-version and cipher scans are cheap reconnaissance worth running, but a configuration that is merely not the newest has no attack narrative — pair any TLS finding with a concrete consequence (a downgrade, a genuinely weak cipher, mixed content, an expired chain) before submitting it.
- Takeaway: "Not best practice" is not a vulnerability; calibrate reports to demonstrable impact and keep raw scanner output out of submissions unless you can exploit it.

### 2026-10-06 — AEM misconfiguration exposes the content repository via JSON render selectors (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1939272](https://hackerone.com/reports/1939272)
- Type: Security misconfiguration / information disclosure
- Summary: Appending `/.1.json` to certain URLs on an Adobe Experience Manager site returned a JSON rendering of the underlying repository node, disclosing every template and file in the web root together with system usernames.
- Technique / pattern: Sling-based stacks such as AEM re-render any node in alternate formats chosen by a URL selector, so suffixes like `/.1.json` or `/content.3.json` serve content the page template filtered out, with the integer controlling traversal depth. Fingerprint the CMS first, then append that stack's own selectors to ordinary page paths before concluding the site exposes no API.
- Takeaway: Disable or restrict default render selectors and depth parameters in production — a URL suffix that swaps the serializer is an undocumented API.

### 2026-10-06 — Directory listing on a publicly accessible service (Nextcloud) — n/a
- Source: [HackerOne #2311179](https://hackerone.com/reports/2311179)
- Type: Security misconfiguration (directory listing)
- Summary: A web server in the program's estate was configured to display a directory's file list to anonymous visitors; the reporter flagged the listing and deliberately did not enumerate files or check tokens because they could not confirm the host was in scope.
- Technique / pattern: Indexing is found by requesting the parent path of any known asset — `/assets/`, `/uploads/`, `/static/` — and watching for an index page instead of a `403`. The discipline worth copying is scope-first reporting: describe the configuration and stop, rather than reading what the listing exposes.
- Takeaway: `autoindex off` / `Options -Indexes` belongs in the baseline config, and where scope is uncertain, report the misconfiguration without exploring the data behind it.

### 2026-10-06 — Unauthenticated access to an InterScan VirusWall admin interface (NASA VDP) — n/a
- Source: [Bugcrowd #702a130f](https://bugcrowd.com/disclosures/702a130f-5f34-4fd5-bfe4-beefe8813db7/unauthenticated-access-to-interscan-viruswall-admin-interface)
- Type: Security misconfiguration / exposed administrative interface (P5, accepted business risk)
- Summary: Administrative endpoints of a Trend Micro InterScan VirusWall appliance, among them `/admin.cgi` and `/interscan/`, answered without credentials, putting the gateway's security configuration within reach of anonymous visitors; the endpoints were subsequently secured.
- Technique / pattern: Security appliances — mail and web gateways, antivirus relays, load balancers, printers — publish fixed vendor-documented admin paths. Fingerprint the appliance from page titles, asset paths and CGI names, then walk its documented management routes; these hosts are usually missing from the application team's inventory and sit outside the web app's auth middleware.
- Takeaway: Appliance management planes belong on an internal network or behind authentication; keep non-application hosts in the recon inventory and rate them by the controls their console can change.

### 2026-10-06 — Clickjacking on NASA's AHED portal login (NASA VDP) — n/a
- Source: [Bugcrowd #fa505134](https://bugcrowd.com/disclosures/fa505134-406d-46d0-9e2f-77cb9ff8ee50/clickjacking-vulnerability-on-nasa-s-ahed-portal-login)
- Type: Security misconfiguration (missing anti-framing controls, P5)
- Summary: The AHED portal's login page could be loaded in an invisible iframe on an attacker-controlled site, enabling UI-redress attacks against the authentication form; it was remediated by adding `X-Frame-Options` protection.
- Technique / pattern: Test framing per sensitive page rather than per host — request each login, consent, payment and delete-confirmation page and check for `X-Frame-Options` or CSP `frame-ancestors` in the *response headers*, then build a minimal iframe proof of concept. Impact rests entirely on what a tricked click accomplishes.
- Takeaway: Send `frame-ancestors 'self'` on every authentication and state-changing page; a framable login only matters when one click yields a security-relevant outcome.

### 2026-10-05 — Security misconfiguration: weak passwords accepted (Semrush) — n/a
- Source: [HackerOne #285482](https://hackerone.com/reports/285482)
- Type: Security misconfiguration / weak password policy
- Summary: Semrush's registration accepted trivial passwords such as `123`, `12345` and `abc12345`, leaving accounts exposed to credential stuffing and brute force. Classified under violation of secure design principles and closed as informative. Older but a clean teaching case.
- Technique / pattern: Test the password policy as its own check — submit a single digit, a short dictionary word and a common password at signup and at change-password — and separately verify whether login is rate-limited, since a weak policy plus no throttling is what turns the finding into practical account compromise.
- Takeaway: Minimum-strength enforcement, a breached-password check and login rate limiting are three independent controls; the absence of any one is a real weakness even when each alone is rated low.

### 2026-10-05 — Exposed debug.log file at a NASA WordPress subdomain reveals internal data (NASA) — n/a
- Source: [Bugcrowd #b3138422](https://bugcrowd.com/disclosures/b3138422-f649-4b97-9b2e-7fa572cb0b84/high-exposed-debug-log-file-at-nasa-wordpress-subdomain-reveals-sensitive-internal-data)
- Type: Security misconfiguration / debug artifact left world-readable
- Summary: A WordPress subdomain left its `debug.log` publicly fetchable, exposing internal paths and operational detail useful for further attacks. NASA corrected the exposure and recorded it as an accepted business risk.
- Technique / pattern: WordPress writes `WP_DEBUG` output to `wp-content/debug.log`, a predictable path, so request it directly across hosts; generalise to other convention-named artifacts (`error_log`, `.env`, `phpinfo.php`, backup and dated log variants) at the web root and one level up.
- Takeaway: Debug logging belongs off in production and, where kept, outside the document root. Error output is an uncontrolled copy of stack traces, paths and sometimes credentials, so a readable log at a guessable path is a reconnaissance gift.

### 2026-10-05 — Authorized drivers can disable remote monitoring (Tesla) — n/a
- Source: [Bugcrowd #391e1551](https://bugcrowd.com/disclosures/391e1551-2bdb-4d0d-8333-188b2a7344a1/authorized-drivers-can-disable-remote-monitoring)
- Type: Security misconfiguration / over-privileged guest role
- Summary: A guest granted Mobile Access to a Tesla vehicle held permissions beyond their role: they could revoke access for all users including the owner and override security features such as Valet Mode, PIN to Drive and Glovebox PIN. Rated P3; Tesla fixed it with server-side checks restricting these operations to the owner. 
- Technique / pattern: When a product has owner, delegate and guest tiers, enumerate every state-changing action a lower tier can reach and confirm each is re-checked server-side against the acting role. Interfaces that merely hide a control leave the underlying operation callable, so exercise the API directly from the low-privilege account.
- Takeaway: Each sensitive operation needs its own server-side role check; a shared permission surface that only the UI differentiates lets a delegated role perform owner-only actions.

### 2026-10-05 — Multiple privilege escalations using a low-privileged user (Pinterest) — n/a
- Source: [Bugcrowd #e0e71bd1](https://bugcrowd.com/disclosures/e0e71bd1-789b-4304-be0e-a9bb4e23338e/multiple-privilege-escalations-using-a-low-privileged-user)
- Type: Security misconfiguration / broken access control (vertical privilege escalation)
- Summary: A low-privileged account could bypass restrictions and view data it was not authorised to see on a Pinterest domain. Accepted and remediated.
- Technique / pattern: From a deliberately minimal account, walk the admin and elevated-role routes the UI hides and replay them with the low-privilege session. Where several such routes fall through, the application is checking authentication but not role, so collect the whole set rather than reporting a single instance.
- Takeaway: Authorization must be enforced per action against the caller's actual role; hiding privileged routes in the interface is not a control when the endpoints still answer a basic account.

### 2026-10-01 — CORS Misconfiguration on trust.yelp.com (Yelp) — n/a
- Source: [HackerOne #1716286](https://hackerone.com/reports/1716286)
- Type: CORS misconfiguration
- Summary: `trust.yelp.com` served a permissive cross-origin policy together with `Access-Control-Allow-Credentials: true`, so content on an attacker-controlled origin could make credentialed two-way requests to the host and read the responses.
- Technique / pattern: Replay requests with an `Origin` header you control and watch whether it is reflected into `Access-Control-Allow-Origin` while `Access-Control-Allow-Credentials` stays `true`; also test the `null` origin and prefix, suffix and subdomain matches before claiming impact.
- Takeaway: `Access-Control-Allow-Credentials: true` is only safe behind a strict allow-list — reflecting the request `Origin` turns every authenticated endpoint on the host into a cross-origin read primitive.

### 2026-10-01 — 403 Bypass Leading to Exposed WordPress Authentication Endpoint on NASA Science Domain (NASA VDP) — n/a (P5)
- Source: [Bugcrowd disclosure add0c75e](https://bugcrowd.com/disclosures/add0c75e-8e55-4bf7-bcfc-991b482742ef/403-bypass-leading-to-exposed-wordpress-authentication-endpoint-on-nasa-science-domain)
- Type: WAF / access-control bypass exposing an administrative login surface
- Summary: A WAF rule meant to block the WordPress administration area of a NASA science domain could be bypassed, leaving the WordPress authentication endpoint reachable from the internet.
- Technique / pattern: When a path returns `403`, retry it with path and encoding variants — a trailing slash, an added `;` segment, case changes, double-encoded characters, doubled `//`, a differently placed query string — because edge rules often match the literal string rather than the normalized path the origin resolves.
- Takeaway: A blocking rule at the edge is not an access control; a login surface should be restricted at the origin with a network allow-list or an authentication gateway, since edge pattern matches are routinely bypassable.

### 2026-10-01 — Publicly Accessible Version Control Log For NASA's Vulcan CFD (NASA VDP) — n/a
- Source: [Bugcrowd disclosure b0fe6ea0](https://bugcrowd.com/disclosures/b0fe6ea0-aac0-4d04-9e6f-b4b923626c50/publicly-accessible-version-control-log-for-nasa-s-vulcan-cfd)
- Type: Security misconfiguration / development metadata exposure
- Summary: An unauthenticated repository log for NASA's Vulcan CFD project exposed commit messages, internal development notes, usernames and software change descriptions; it was later placed behind authentication, with permissions corrected and the history sanitized.
- Technique / pattern: Hunt for version-control byproducts that are not the repository itself — exported commit logs, `ChangeLog` files, build manifests and CI job output — which are often published as plain static files and missed by scanners that only look for `.git/`.
- Takeaway: Commit metadata is reconnaissance material: it names developers, internal hosts and the shape of the codebase, so treat exported history as sensitive even where no source is served.

### 2026-10-01 — Improper Access Control: Authenticated Resource Exposed via Wayback Machine Archive (NASA VDP) — n/a (P5)
- Source: [Bugcrowd disclosure b4233894](https://bugcrowd.com/disclosures/b4233894-756a-43cf-a4ef-8ce484278b90/improper-access-control-authenticated-resource-exposed-via-wayback-machine-archive-nasa-sheets-docs)
- Type: Security misconfiguration / third-party document sharing
- Summary: NASA Google Docs and Sheets that were meant to require authentication were reachable without it, with the links recoverable from Internet Archive snapshots of NASA pages; the documents were subsequently locked down.
- Technique / pattern: Mine archive sources — the Wayback Machine, its CDX index, search-engine caches — for links that have since been removed from the live site, then open each one anonymously, because link-sharing permissions on third-party document platforms usually outlive the page that referenced them.
- Takeaway: Removing a link does not revoke access; "secret URL" sharing on SaaS document platforms has to be re-permissioned, and archive mining is a cheap way to find what a site used to publish.

### 2026-09-30 — Sandbox User Can Inject a Rogue CA into the OS Trust Store via a Sudo-Allowed Script (AWS VDP) — n/a
- Source: [HackerOne #3633146](https://hackerone.com/reports/3633146)
- Type: Security misconfiguration (over-broad passwordless sudo rule on a script reading a writable directory)
- Summary: In the Bedrock AgentCore Code Interpreter sandbox, the unprivileged `genesis1ptools` user held passwordless sudo on `/opt/amazon/genesis1p-tools/bin/deploy-certificates.sh`, a script that reads certificates from the sandbox-writable `/tmp/certificates/` and installs them as root into the system trust anchors. Any authenticated user with Code Interpreter access could therefore have a rogue CA trusted system-wide and forge certificates for arbitrary domains, including internal services.
- Technique / pattern: Enumerate `sudo -l` inside any sandbox or CI container and, for each permitted command, trace where that program reads its *inputs* from. A sudo rule is only as tight as the least-trusted path the target script touches, so a root helper reading a world-writable directory, a config file under the user's control or a `PATH`-resolved binary is a privilege boundary with a hole in it. The proof here was generating a CA, dropping it in `/tmp/certificates/`, invoking the allowed script, and confirming the anchor after `update-ca-trust`.
- Takeaway: Audit sudoers entries by data flow, not by the command name. A script granted root must validate and own its input location; and in a sandbox, the OS trust store is a security control that unprivileged code must not be able to extend.


### 2026-09-30 — Unauthenticated ClickHouse UI on an AWS a2z.com Host: Database Access and SSRF (AWS VDP) — n/a
- Source: [HackerOne #3809407](https://hackerone.com/reports/3809407)
- Type: Security misconfiguration — internet-exposed data-plane console with authentication disabled (triaged as Authentication Bypass)
- Summary: A ClickHouse web interface running on an internal-style Amazon `a2z.com` subdomain was reachable from the internet with no authentication, giving database access and, per the report, a server-side request forgery primitive reachable from the query interface.
- Technique / pattern: The path to it is subdomain enumeration across a large corporate apex followed by probing each discovered host for the default ports and paths of data-plane and analytics consoles — ClickHouse, Flink, Grafana, Kibana, Jupyter, Airflow — which ship with authentication off and are frequently published by a reverse proxy that forwards everything. A query console that can read remote URLs or table functions is also an SSRF sink, so enumerate what the console itself can reach, not just the data it holds.
- Takeaway: A deployment whose only protection is an unguessable hostname fails the first time certificate-transparency logs are read. Authentication belongs on the service, and query consoles need their outbound capabilities restricted as well as their inbound access.

### 2026-09-30 — Backdooring the Next Release of Codex Through GitHub Action Pipeline Exploit Chaining (OpenAI) — $1,000
- Source: [Bugcrowd #3c0e5972](https://bugcrowd.com/disclosures/3c0e5972-74ad-46a3-ab56-169db3d4c66f/backdooring-the-next-release-of-codex-through-github-action-pipeline-exploit-chaining)
- Type: CI/CD security misconfiguration chained into a supply-chain compromise (P1)
- Summary: A full CI/CD compromise chain against the `openai/codex` repository began with a prompt injection in the `issue-labeler.yml` workflow — any user could embed attacker instructions in a GitHub issue and obtain command execution inside the Codex agent sandbox — and ended with a persistent backdoored workflow able to publish releases under OpenAI's identity, against a package seeing roughly 10M weekly npm downloads.
- Technique / pattern: From the agent sandbox the researchers escaped `workspace-write` confinement by abusing .NET CLR debug pipes left exposed in `/tmp/`, read runner process memory to leak `ACTIONS_RUNTIME_TOKEN` and `GITHUB_TOKEN`, used those credentials to poison the GitHub Actions cache, pivoted execution into the more privileged `rust-ci.yml` workflow to gain `contents: write`, and planted a persistent workflow holding `id-token: write`.
- Takeaway: A workflow that feeds untrusted issue or PR text to an LLM is an injection sink with the runner's privileges behind it. Keep untrusted-input workflows on least-privilege tokens and isolate them from build and release workflows, and treat the Actions cache as a writable trust boundary between jobs rather than as inert storage.

### 2026-09-30 — HTTP Verb Tampering Leads to Authorization Bypass on a Protected Archive Directory (NASA VDP) — n/a
- Source: [Bugcrowd #b5a435ec](https://bugcrowd.com/disclosures/b5a435ec-c13c-4208-81f7-64431b1b7a4e/http-verb-tampering-leads-to-authorization-bypass-on-archive-exist-team-directory)
- Type: Security misconfiguration — method-scoped access control / broken access control (P2)
- Summary: A protected directory `/archive/exist/team/` on a NASA web application returned `401 Unauthorized` to a `GET` request but served its restricted contents — RFI responses, internal team communications and technical data — when the identical path was requested with `POST`. Accepted and resolved.
- Technique / pattern: Server-level authorization rules that name specific methods leave every other verb unprotected; the classic instance is an Apache `<Limit GET>` or `<Limit GET POST>` block, or an IIS/servlet security constraint listing `http-method` elements. Replaying a `401` or `403` response with `POST`, `HEAD`, `PUT` or an arbitrary verb is a one-request check that costs nothing.
- Takeaway: Access control must deny by default across all HTTP methods rather than allow-listing the ones that were considered. Whenever a path answers `401` or `403`, re-send it with other verbs before concluding it is protected.


### 2026-09-30 — Publicly accessible phpinfo() exposes detailed server configuration (NASA VDP) — n/a
- Source: [Bugcrowd #41e4cef6](https://bugcrowd.com/disclosures/41e4cef6-b0bf-4a9c-9a5d-9a34747ad7c4/publicly-accessible-phpinfo-exposes-detailed-server-configuration)
- Type: Security misconfiguration / information exposure
- Summary: A PHP information page sat at an easily guessable URL and leaked the exact PHP version, internal file paths, internal IP addresses, loaded extensions such as `pdo_mysql` and `mongodb`, disabled-function settings and SSL certificate data naming internal JPL subdomains.
- Technique / pattern: Probe every in-scope host for leftover diagnostic pages by name (`phpinfo.php`, `info.php`, `test.php`, `server-status`); they are absent from sitemaps and navigation but present in any reasonable wordlist.
- Takeaway: Diagnostic pages belong to development only — ship them disabled and assert their absence in CI, because a version-plus-path leak turns generic public exploits into targeted ones.

### 2026-09-30 — OAuth misconfiguration: third-party auth cookies not invalidated after logout (Opera) — $150
- Source: [Bugcrowd #6ff09a42](https://bugcrowd.com/disclosures/6ff09a42-c27f-4d5d-9e7c-8eaafe8eee70/oauth-misconfiguration-found-on-https-wemedia-opera-com)
- Type: OAuth / session misconfiguration
- Summary: On the Opera Wemedia portal, third-party authentication cookies were not properly invalidated on logout from either the application or the authorisation server, so a later user of the same device could re-enter the victim's account.
- Technique / pattern: Log out, then replay the retained third-party auth cookie and re-run the provider's `authorize` step; if the identity provider still holds a session it silently re-issues a token with no credential prompt, which is what makes the "account squatting" outcome possible.
- Takeaway: Logout must clear the relying-party session and trigger provider-side logout, or at minimum force re-authentication — otherwise logging out is cosmetic on any shared device.

### 2026-09-30 — Exposed credentials on the FCFOPS site allow access to a NASA facility portal (NASA VDP) — n/a
- Source: [Bugcrowd #351f28d8](https://bugcrowd.com/disclosures/351f28d8-007f-4b51-9ce8-ca6ee9b84e2e/unauthorized-access-to-nasa-fluid-combustion-facility-portal-https-fcfops-grc-nasa-gov)
- Type: Security misconfiguration / exposed credentials (rated P1)
- Summary: Credentials published on the Fluid & Combustion Facility operations site permitted unauthorized access to the portal; the remediation was to remove the offending credentials from the site.
- Technique / pattern: Read the target's own static content — help pages, onboarding guides, training PDFs, sample instructions — for working logins before attacking the login form. Published documentation is an under-searched credential store and needs no exploitation at all.
- Takeaway: Treat anything published as public permanently: credentials exposed in documentation must be rotated, not merely deleted from the page.

### 2026-09-30 — Admin panel exposed without authentication (Shopify) — n/a
- Source: [HackerOne #1417288](https://hackerone.com/reports/1417288)
- Type: Security misconfiguration / missing authentication for a critical function
- Summary: An administrative panel was reachable at a public URL with no authentication at all, so anyone who located it could edit and destroy data.
- Technique / pattern: Enumerate subdomains first, then sweep common admin paths on each one. Forgotten staging deployments and vendor-installed panels regularly leave authentication to a network control that was never actually applied.
- Takeaway: Authentication belongs in the application, not in an assumption about who can reach the host — inventory every deployed panel and assert a login check on each.

### 2026-09-29 — Internal debug page and unauthenticated gRPC service reachable from the Starlink network (SpaceX) — $4,800
- Source: [Bugcrowd #35b94f7c](https://bugcrowd.com/disclosures/35b94f7c-75a6-469e-ab6c-7f9649c05595/spacex-debug-page-accessible-when-using-starlink)
- Type: Security misconfiguration — internal service exposed to customers (P2, resolved)
- Summary: While standing up new ground infrastructure, SpaceX exposed unauthenticated services externally; a subscriber could reach an internal debug page and, more seriously, an unauthenticated gRPC endpoint.
- Technique / pattern: Enumerate from inside the provider's own network — `traceroute` from a subscriber connection reveals internal hop addresses, and probing those addresses from that same vantage point finds services firewalled from the public internet but not from customers.
- Takeaway: "Internal" is a position on the network, not a property of a service, and newly deployed infrastructure is the likeliest place to find controls that have not been applied yet.

### 2026-09-29 — Publicly listable Amazon S3 bucket (NASA VDP) — n/a
- Source: [Bugcrowd #067b785f](https://bugcrowd.com/disclosures/067b785f-a7a3-41ef-9261-176b824f1d6b/amazon-s3-bucket-misconfiguration)
- Type: Security misconfiguration — cloud storage ACL (P5, informational)
- Summary: An S3 bucket permitted public listing and retrieval of `.tlf` files. The program judged the content intended to be public and closed the report as informational, while the reporter argued such files can still leak layout, metadata or configuration detail depending on their use.
- Technique / pattern: Enumerate bucket names from page source, DNS records and JavaScript bundles, then issue a list request to test the ACL — but the impact has to be argued from the contents, not from listability alone.
- Takeaway: An open bucket is only a finding when what sits inside it matters, so open and read a sample before writing up or expect an informational close.

### 2026-09-29 — Anonymous FTP login exposing directory contents on a NASA subdomain (NASA VDP) — n/a
- Source: [Bugcrowd #fb4e1846](https://bugcrowd.com/disclosures/fb4e1846-6f51-405f-a73e-8a6c83bc0c90/nasa-ftp-vulnerable-anonymous)
- Type: Security misconfiguration — anonymous service access (P5, informational)
- Summary: An `nmap` scan of the host backing a NASA science subdomain showed FTP open, and the anonymous account accepted a login, allowing directory browsing and file download with no authentication.
- Technique / pattern: Resolve in-scope hostnames to addresses and scan beyond ports 80 and 443 — legacy protocols such as FTP, SMB and rsync are rarely covered by web-focused reviews, and anonymous credentials are the first thing to try against them.
- Takeaway: Non-HTTP services on in-scope hosts are routinely under-tested, but as with open buckets the report stands or falls on whether the exposed files are actually sensitive.

### 2026-09-29 — Tomcat manager reachable with default credentials (JetBlue) — n/a
- Source: [HackerOne #1267174](https://hackerone.com/reports/1267174)
- Type: Security misconfiguration — default credentials on an admin interface
- Summary: A JetBlue host ran Apache Tomcat 6.0.35 with the manager application exposed, and the vendor default account pair `tomcat` / `tomcat` still worked, granting administrative access to the application server.
- Technique / pattern: Fingerprint the server banner and version, then request the well-known admin paths such as `/manager/html` and `/host-manager/html` and try the vendor default pairs before anything else — an outdated version string is a strong signal the install was never hardened.
- Takeaway: Tomcat manager access is a deploy-a-WAR primitive, so default credentials there are effectively remote code execution; confirm impact within scope rather than deploying anything.

### 2026-09-28 — Retired GitHub username takeover from a link in a public AWS repository (AWS VDP) — n/a
- Source: [HackerOne #3478646](https://hackerone.com/reports/3478646)
- Type: Security misconfiguration / dangling reference takeover
- Summary: A public AWS repository linked to a GitHub account whose username had been retired and left unclaimed. The researcher registered the username and published a repository at the same path, so the official link began serving attacker-controlled content.
- Technique / pattern: Crawl a target's repositories, docs and READMEs for outbound links and check which return 404; a dead link pointing at a platform that allows re-registration of the identifier is a takeover. This is the same dangling-reference class as subdomain takeover, applied to usernames, package names and social handles.
- Takeaway: Outbound links are part of the attack surface — audit them for dead destinations, because a trusted domain pointing at a re-registrable identifier lends the attacker its credibility.

### 2026-09-28 — Kiro IDE writes authentication tokens world-readable at mode 0644 (AWS VDP) — n/a
- Source: [HackerOne #3630605](https://hackerone.com/reports/3630605)
- Type: Security misconfiguration / incorrect default permissions
- Summary: Kiro IDE 0.11.107 wrote its access and refresh tokens to `~/.aws/sso/cache/kiro-auth-token.json` with mode `0644`, so any local user or process could read bearer tokens granting full CodeWhisperer and Q Developer API access (CVE-2026-11931). The AWS CLI stores its tokens in the same directory at `0600`, and Kiro itself used the macOS Keychain for other secrets.
- Technique / pattern: For desktop and CLI tooling, enumerate where credentials land on disk and check the mode — `ls -l` across the application's cache and config directories, compared against a sibling tool that does it correctly. A same-vendor inconsistency is the strongest argument available in the report.
- Takeaway: Token files need mode `0600` and, where the platform offers one, the system keystore; inconsistent handling inside a single product usually means one code path missed the umask or an explicit `chmod`.

### 2026-09-28 — Unauthenticated Apache Flink dashboard exposes pipelines and allows job cancellation (NASA VDP) — n/a
- Source: [Bugcrowd #caaf6992](https://bugcrowd.com/disclosures/caaf6992-f58f-46ce-8b34-cb256859804e/unauthenticated-apache-flink-dashboard-access)
- Type: Security misconfiguration
- Summary: An Apache Flink instance at `main.aws-ultra-swift.smce.nasa.gov` served its web dashboard with no authentication challenge. The Job Manager and Task Manager views exposed internal hostnames, software versions, library classpaths, configuration parameters and operational logs, and any visitor could cancel or stop live data streams.
- Technique / pattern: Cluster and pipeline consoles — Flink, Spark, Airflow, Kubernetes dashboards, Jenkins — frequently ship with authentication off and then get published by a reverse proxy that forwards everything. Fingerprint the framework from its static assets, walk its documented admin routes, and record both the read exposure and the state-changing controls on offer.
- Takeaway: Treat an unauthenticated admin console as more than information disclosure: it usually carries destructive actions, so report the reachable controls alongside the metadata leak.

### 2026-09-28 — Default credentials give administrative access to Teamwork Cloud (NASA VDP) — n/a
- Source: [Bugcrowd #6cea28c5](https://bugcrowd.com/disclosures/6cea28c5-b3a9-45a8-8617-826ba0649279/default-credentials-for-teamwork-cloud)
- Type: Security misconfiguration / default credentials
- Summary: An internal Teamwork Cloud deployment was reachable using the product's shipped default credentials, which granted administrative access. Bugcrowd rated it P1; the public disclosure is a short summary only.
- Technique / pattern: For any identified commercial product, look up the vendor's documented default account and try that before anything else — deployment guides and container images name them, and installations that were never hardened keep them. Confirm the privilege level actually reached, since admin access is what moves this from a low finding to a critical one.
- Takeaway: Default credentials remain a critical finding on internal applications; provisioning should force a credential change before the service accepts its first login.

### 2026-09-28 — External SMTP submission allows unauthenticated email delivery to internal NASA recipients (NASA VDP) — n/a
- Source: [Bugcrowd #350eedae](https://bugcrowd.com/disclosures/350eedae-27d6-4d94-a1aa-bec8fc480c69/external-smtp-submission-allows-unauthenticated-email-delivery-to-internal-nasa-domain-recipients-with-potential-for-spoofed-sender-identity-display)
- Type: Security misconfiguration (mail infrastructure / SMTP submission)
- Summary: An external SMTP service accepted unauthenticated submissions addressed to internal NASA domain recipients without validating sender identity at the submission layer, letting externally originated messages enter the internal mail flow with a potentially spoofed sender display. Rated P3, resolved.
- Technique / pattern: Mail infrastructure belongs in the attack surface map — enumerate MX and submission hosts, then test whether the relay accepts a message for an internal recipient with no authentication and how much of the sender display name it will carry. Spoofed mail landing in an internal inbox bypasses the “external sender” banner staff are trained to look for.
- Takeaway: Check the mail path as well as the web path: SPF, DKIM and DMARC on the domain do not help when a relay in front of the internal mail flow accepts unauthenticated submission for internal recipients.

### 2026-09-28 — Broken Link Hijacking (impersonation) on ntrs.nasa.gov via an abandoned Facebook URL (NASA VDP) — n/a
- Source: [Bugcrowd #cd65bae3](https://bugcrowd.com/disclosures/cd65bae3-c385-4964-af8f-a902da35051b/broken-link-hijacking-impersonation-on-ntrs-nasa-gov-via-abandoned-facebook-url)
- Type: Security misconfiguration (broken link hijacking / impersonation)
- Summary: A report published on `ntrs.nasa.gov` linked to a NASA Facebook page that no longer existed, so the handle could be claimed by anyone and used to impersonate NASA under the authority of the linking page. Rated P4, resolved.
- Technique / pattern: Crawl the target's own pages for outbound links to social handles, app-store listings, community channels and short links, then check each destination's claim status. Unlike a dangling CNAME the DNS is perfectly healthy, so only link-level checking surfaces it.
- Takeaway: Outbound links are an inventory item — one dead social link on a `.gov` research host hands an impersonator the domain's credibility for free.

### 2026-09-28 — Unauthenticated file upload with CORS wildcard and no rate limiting (Essity) — n/a
- Source: [HackerOne #3765476](https://hackerone.com/reports/3765476)
- Type: Security misconfiguration (missing authentication plus permissive CORS on an upload route)
- Summary: `POST /Umbraco/Api/ContactApi/SaveAttachment` on two production hosts accepted file uploads with no authentication, no CAPTCHA and no rate limiting, and answered with `access-control-allow-origin: *`, so any website on the internet could silently push files — including PDFs with embedded JavaScript — into the backend storage.
- Technique / pattern: Three weak controls compose into one strong finding: missing auth makes the route reachable, wildcard CORS makes it drivable from any origin with no user interaction, and the missing rate limit turns it into storage/cost abuse and a malware-hosting primitive. Test upload endpoints for all three, then check the stored file's URL and served content type.
- Takeaway: Report composed misconfigurations as a single chain with the combined impact — an anonymous upload that a third-party page can drive is materially worse than an anonymous upload alone.

### 2026-09-28 — Bypass of an open-redirect fix on lovable.dev via path traversal in the redirect parameter (Lovable VDP) — n/a
- Source: [HackerOne #3599248](https://hackerone.com/reports/3599248)
- Type: Security misconfiguration / Open redirect (incomplete fix)
- Summary: A previously patched open redirect remained exploitable: the fix blocked backslash payloads (`/\` and `/%5C`) but not `/..//google.com`, which the server normalized to `//google.com` and then treated as a protocol-relative URL, redirecting authenticated users off-site after login via `https://lovable.dev/auth/post-login?redirect=...`.
- Technique / pattern: Always retest a patched redirect. Feed the validator values that *normalize* into a different URL than they appear to be — `/..//host`, `/%2f%2fhost`, `/\/host`, `https:/host`, `//host%2f@target` — because a denylist patch removes only the exact string that was reported.
- Takeaway: Redirect validation must parse the final URL and compare its host against an allow-list; a denylist built from the last report's payload is an invitation to send the next variant.


### 2026-09-27 — Subdomain takeover on `s3.shopify.com` via an unclaimed S3 bucket (Shopify) — $500
- Source: [HackerOne #207576](https://hackerone.com/reports/207576)
- Type: Security misconfiguration / dangling CNAME to an unregistered S3 bucket
- Summary: `s3.shopify.com` pointed by CNAME at Amazon S3 while no bucket of that name existed, so anyone could register the bucket and serve arbitrary content — HTML for stored XSS, or a phishing page — from a trusted Shopify subdomain.
- Technique / pattern: Enumerate subdomains, resolve each CNAME, and flag any whose provider answers with a "no such bucket" or "not found" fingerprint instead of a real site. The claim is first-come, so a defensive researcher registers the bucket themselves to hold the name while the vendor corrects DNS.
- Takeaway: DNS records outlive the resources they point at — retire the CNAME together with the bucket, and audit for dangling delegations continuously rather than only at deprovisioning time.

### 2026-09-27 — Subdomain takeover of a DoD host via an unclaimed Amazon S3 bucket (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #918946](https://hackerone.com/reports/918946)
- Type: Security misconfiguration / dangling CNAME to an unregistered S3 bucket
- Summary: A DoD subdomain still resolved through a CNAME to an S3 bucket in `us-east-1` that no longer existed; the researcher registered the same bucket name and served their own `index.html` plus an XSS proof of concept from the government domain.
- Technique / pattern: The same dangling-delegation pattern as any cloud takeover, with one wrinkle worth remembering — bucket names are global while the endpoint host is regional, so match the region encoded in the original CNAME before concluding a name is unclaimable.
- Takeaway: Content served from a trusted domain inherits that domain's trust for cookies, CSP allowlists and the user's own judgment, so a takeover on a government or brand subdomain is a phishing platform, not merely a defacement.

### 2026-09-27 — Base tag hijacking via Host header injection (NASA VDP) — n/a
- Source: [Bugcrowd #fd5b1c75](https://bugcrowd.com/disclosures/fd5b1c75-54a1-4db9-8618-a4e26c6b8147/base-tag-hijacking-via-host-header-injection)
- Type: Security misconfiguration / Host header reflected into the `<base href>` tag
- Summary: The server built the page's `<base>` tag from the incoming Host header, so an injected host produced `<base href="https://evil.com">` and every relative reference on the page — `/js/main.js`, `/img/logo.png` — resolved to the attacker's origin. The program rated it P5, since it requires Host header injection and affects only client-side loading.
- Technique / pattern: Send a modified `Host` or `X-Forwarded-Host` and grep the whole response body, not just the `Location` header, for the value you supplied: absolute links, `<base>`, canonical tags, password-reset URLs and script `src` attributes are all built from it. `<base>` is the highest-leverage sink of the set because it silently repoints every relative reference at once.
- Takeaway: Derive absolute URLs and `<base>` from server-side configuration, never from a client-controlled header, and validate Host against an allowlist at the edge.

### 2026-09-27 — Open redirection through the `X-Forwarded-Host` header (NASA VDP) — n/a
- Source: [Bugcrowd #b00f800d](https://bugcrowd.com/disclosures/b00f800d-63bf-4206-a2ea-83ff9563c571/host-header-injection-via-x-forwarded-host-lead-to-open-redirection)
- Type: Security misconfiguration / Host header injection producing an open redirect
- Summary: The application trusted `X-FORWARDED-HOST` when constructing redirect targets, so an attacker-supplied host sent users off-site — usable to front a phishing page or to sidestep controls keyed to the original domain. The program accepted it as an informational business risk.
- Technique / pattern: Proxy-forwarding headers (`X-Forwarded-Host`, `X-Forwarded-Proto`, `X-Original-URL`, `X-Rewrite-URL`) are trusted by many frameworks on the assumption that a reverse proxy sets them; when the app is reachable without that proxy, or the proxy forwards what it received, the client sets them instead. Test each one against redirect handling, emailed links and absolute-URL generation.
- Takeaway: Only the edge proxy may set forwarding headers, and it must overwrite rather than append the client's value; everything downstream should treat them as untrusted until that guarantee is actually enforced.

### 2026-09-27 — Takeover of hackerone.engineering during a GitHub Pages CNAME-hold release (HackerOne) — n/a
- Source: [HackerOne #2085260](https://hackerone.com/reports/2085260)
- Type: Security misconfiguration / subdomain takeover through a provider-side claim window
- Summary: For roughly ten minutes GitHub released its hold on the `hackerone.engineering` custom domain while HackerOne's DNS still pointed at GitHub Pages, and a researcher claimed the hostname in their own repository. HackerOne remediated by enabling GitHub's Domain Verification for the host and moving the Pages domain to a verified state.
- Technique / pattern: The takeover window does not open only when *you* delete a resource — it also opens when the *provider* stops reserving the name, after a plan change, a repository transfer or rename, a verification lapse or an account migration. Re-check every dangling-CNAME candidate repeatedly rather than once, because a host that answers "claimed by someone else" today can be claimable for minutes tomorrow.
- Takeaway: Pair every custom-domain CNAME with the provider's domain-verification feature so the name stays reserved to you even when the underlying site or plan changes.

### 2026-09-27 — Unprotected admin panel on app.lemlist.com reachable by any ordinary user (lemlist) — n/a
- Source: [HackerOne #937921](https://hackerone.com/reports/937921)
- Type: Security misconfiguration / missing authorization on an administrative surface
- Summary: Administrative endpoints on `app.lemlist.com`, discovered by reading the application's JavaScript bundles, carried no role check, so any ordinary authenticated user could load the admin panel.
- Technique / pattern: Treat the frontend bundle as the application's route table — pull every path, route definition and feature-flag name out of the JS, then request the admin-looking ones from a plain free account. Single-page applications ship the admin routes to every visitor because one bundle serves all roles, and the server-side check is often assumed to be implied by the UI not rendering the link.
- Takeaway: Admin routes must be authorized server-side and independently of the client bundle; extract paths from JavaScript rather than only crawling what the UI links to.

### 2026-09-27 — Subdomain takeover of a Bosch host via a dangling Azure Traffic Manager CNAME (Bosch) — 20 points
- Source: [Bugcrowd #9ae60aa2](https://bugcrowd.com/disclosures/9ae60aa2-fb4c-4ae9-a4ad-49952de29125/subdomain-takeover)
- Type: Security misconfiguration / subdomain takeover (dangling CNAME)
- Summary: `sidaccounts.bosch.com` kept a CNAME to `sidaccounts.trafficmanager.net` after the underlying Azure Traffic Manager profile was abandoned; the researcher registered that Traffic Manager name themselves and so controlled what was served on the Bosch subdomain.
- Technique / pattern: A classic worth keeping for its provider specifics — Traffic Manager profile names are globally unique and free to claim, so a resolution chain of host to `*.trafficmanager.net` ending in NXDOMAIN or a default page is a takeover signature. Enumerate subdomains, resolve each CNAME, and bucket the targets by cloud provider so each vendor's claim test can be applied in turn.
- Takeaway: Remove DNS records together with, or before, the cloud resource they point at, and reconcile DNS against live cloud inventory as a recurring task rather than only at teardown.

### 2026-09-27 — Public Amazon S3 bucket behind a NASA JPL host exposes stored files (NASA VDP) — n/a
- Source: [Bugcrowd #b4053dc4](https://bugcrowd.com/disclosures/b4053dc4-2ad1-48f7-a1a6-308dc63ec549/amazon-s3-bucket-misconfiguration)
- Type: Security misconfiguration / world-readable cloud storage
- Summary: The bucket behind `descanso.jpl.nasa.gov` was reachable at its S3 endpoint with no access restriction, letting anyone who knew the URL list and download the HTML, PDF and image files it held. NASA accepted it as an informational business risk, as no PII was exposed, and removed the bucket.
- Technique / pattern: Derive candidate bucket names mechanically from hostnames already in scope, since the `<host>.s3.amazonaws.com` convention means a site's own domain name is frequently the bucket name, then test list and read access. Note how this one was triaged: with no sensitive content a public bucket alone is informational, which is why enumerating the bucket's full verb set and the sensitivity of its objects is what decides severity.
- Takeaway: Bucket naming that mirrors public hostnames makes cloud storage trivially discoverable; rate an exposure by what the objects are and which verbs are permitted, not by the bucket being public.

### 2026-09-26 — Magic-link login token reusable across browsers (lemlist) — n/a (High 8.8)
- Source: [HackerOne #1486327](https://hackerone.com/reports/1486327)
- Type: Security misconfiguration / broken passwordless-auth token handling
- Summary: The email "magic link" used to log in to `app.lemlist.com` was not single-use and was not bound to the requesting session, so the same link worked repeatedly and in any browser or incognito window, letting anyone who obtained it log into the victim's account.
- Technique / pattern: For any passwordless or "click to log in" flow, request one link and then replay it: use it a second time, in a fresh browser, from a different IP, and after the stated expiry. Each success is a separate defect (no single-use, no session binding, no expiry, no IP pinning). Capture the link once and script the reuse to prove it deterministically.
- Takeaway: Magic links are bearer credentials and need the same hygiene as password-reset tokens: high entropy, short expiry, one-time use, and invalidation of the token the moment it is consumed.

### 2026-09-26 — CORS misconfiguration allowing credentialed cross-origin reads (U.S. Dept Of Defense) — n/a (High 7-8.9)
- Source: [HackerOne #470298](https://hackerone.com/reports/470298)
- Type: Security misconfiguration / CORS
- Summary: A DoD JSON API reflected an arbitrary `Origin` into `Access-Control-Allow-Origin` while also returning `Access-Control-Allow-Credentials: true`, so an attacker page could issue a credentialed `XMLHttpRequest` and read the authenticated response, bypassing CSRF protections for state-changing actions.
- Technique / pattern: Send a request with `Origin: exploit.com` and inspect the response headers; a reflected origin combined with `Access-Control-Allow-Credentials: true` is the whole proof. Demonstrate impact with a hosted page doing `xhr.withCredentials = true` against a sensitive endpoint. Because a credentialed cross-origin read can retrieve CSRF tokens, a CORS read frequently escalates into CSRF writes.
- Technique / pattern: Do not stop at exact-origin reflection; also test the `null` origin and substring/suffix trust bugs (an origin merely containing the allowed host).
- Takeaway: Reflecting `Origin` with credentials enabled is equivalent to allowing any site to act as the logged-in user. Use a strict allowlist of trusted origins and never combine wildcard/reflected origins with credentialed responses.

### 2026-09-26 — Missing SPF enabling email spoofing from a gov domain (GSA Bounty) — n/a
- Source: [HackerOne #263508](https://hackerone.com/reports/263508)
- Type: Server security misconfiguration / mail (missing SPF)
- Summary: The email domain lacked an enforcing SPF policy, so mail could be sent that appeared to originate from an address at the organization's own domain and was delivered to victims' inboxes.
- Technique / pattern: Check the domain's `TXT` records for SPF (and, by extension, DKIM/DMARC), then send a test message spoofing a first-party sender and confirm inbox (not spam) delivery. The report's own trajectory shows the caveat: without DMARC-enforcement evidence and a concrete phishing scenario, mail-auth findings are routinely rated low or N/A.
- Takeaway: Missing SPF/DKIM/DMARC enables convincing phishing from the target's own name, but severity hinges on demonstrating enforcement gaps and delivery. Publish `SPF -all`, sign with DKIM, and set a `DMARC` reject policy.

### 2026-09-26 — Missing X-Frame-Options / clickjacking on video player (20 Minuten) — n/a (P5, Informational)
- Source: [Bugcrowd be140cd9](https://bugcrowd.com/disclosures/be140cd9-c640-4bdd-a34f-74e71669fadd/lack-of-security-headers)
- Type: Server security misconfiguration / missing security headers
- Summary: `https://videoplayer.20min.ch` returned responses without an `X-Frame-Options` (or equivalent frame-ancestors) header, so the page could be framed by a malicious site for UI-redressing (clickjacking). Closed as P5 / Informational.
- Technique / pattern: Check response headers for framing controls and build a minimal PoC page that embeds the target in an `iframe`; if it renders, framing is allowed. Severity depends entirely on whether a framed, sensitive, click-driven action exists, so a bare header-absence report on a static or non-interactive page is expected to close as Informational.
- Takeaway: Missing security headers are only worth reporting when tied to a concrete exploited action. Defensively, set `Content-Security-Policy: frame-ancestors` (and legacy `X-Frame-Options`) on any page with state-changing UI.

### 2026-09-25 — Internet-exposed internal LibreChat instance with open self-registration grants access to internal models (AWS Vulnerability Disclosure Program) — n/a
- Source: [HackerOne #3287396](https://hackerone.com/reports/3287396)
- Type: Security misconfiguration — self-registration enabled on an internal tool exposed to the internet
- Summary: An internal LibreChat deployment was reachable from the internet with local self-registration left on for non-corporate email domains, so anyone could create an account and use the UI to query internal and proprietary foundation models.
- Technique / pattern: Internal tooling is routinely assumed to be protected because corporate SSO is "expected", while the application's own signup path stays enabled beside it. The researcher registered with an outside address and reached the model list. The recurring pattern: an internal-looking hostname, a login page that also offers "sign up", and no domain allowlist behind it.
- Takeaway: On every internal tool you find exposed, check the registration policy separately from the login page — disable local signup, enforce a domain allowlist, and put the deployment behind network controls rather than relying on obscurity of the hostname.

### 2026-09-25 — Exposed forward proxy allows access to internal domains via the Host header (Reddit) — n/a
- Source: [HackerOne #2967634](https://hackerone.com/reports/2967634)
- Type: Security misconfiguration — open forward proxy reaching internal-only hosts
- Summary: A proxy listening on a high port was reachable from the internet and would forward requests based on the supplied `Host` header, so setting that header to an internal domain returned employee-only content from behind the perimeter.
- Technique / pattern: The researcher found the internal domain first — certificate-transparency data via Censys showed hostnames that only make sense internally — then sent a request through the open proxy with that hostname in the `Host` header. An open proxy turns the `Host` header into a name resolver for the internal network, giving SSRF-like reach without needing a vulnerable application feature.
- Takeaway: Port-scan your own edge for proxy software on non-standard ports and confirm it refuses hosts outside an explicit allowlist. Pair certificate-transparency mining for internal-looking names with any proxy you find, and remember that internal hostnames are public information.

### 2026-09-25 — Host header injection leads to cookie domain manipulation on sealevel.nasa.gov (NASA Vulnerability Disclosure Program) — 5 points (P5)
- Source: [Bugcrowd fd00af00](https://bugcrowd.com/disclosures/fd00af00-5511-4516-bfe7-c4ae38c52cd4/re-host-header-injection-leads-to-cookie-domain-manipulation-on-sealevel-nasa-gov)
- Type: Security misconfiguration — request host reflected into cookie scope
- Summary: The application derived cookie attributes from the incoming request host, so injecting a crafted `Host` header influenced the `Domain` on which cookies were set, widening their scope beyond the intended host.
- Technique / pattern: Host-header injection is usually hunted for cache poisoning and password-reset link poisoning; this report shows a third sink — the `Domain` attribute of `Set-Cookie`. Send the request with a modified `Host` (and with `X-Forwarded-Host`) and read the `Set-Cookie` response header rather than looking at the page body.
- Takeaway: Derive cookie domain and any absolute URL from server-side configuration, never from the request host, and have the web server reject requests whose `Host` is not in an explicit allowlist. When testing host-header injection, inspect response headers for cookie scope alongside redirects and cached content.

### 2026-09-25 — Broken link hijacking: abandoned X/Twitter handle still linked from a live nasa.gov page (NASA Vulnerability Disclosure Program) — n/a (P4)
- Source: [Bugcrowd 71f7148f](https://bugcrowd.com/disclosures/71f7148f-919f-44c1-b8a4-28e1cf284993/broken-link-hijacking-abandoned-x-twitter-handle-nasa_finesse-still-linked-from-a-live-nasa-gov-page)
- Type: Security misconfiguration — dangling outbound link enabling impersonation
- Summary: A live NASA page about the FINESSE/BASALT programs linked to an official-looking X/Twitter handle that had been abandoned and was no longer registered; the researcher claimed the handle as proof of concept, showing that anyone could have registered it and spoken as NASA from a link the agency itself published.
- Technique / pattern: Crawl the target's own pages for outbound links to social handles, app-store listings, package names, community channels and short links, then check each destination for unclaimed or deleted status and whether it is re-registrable. The value comes from the *linking* page's authority, so prioritize links on official program and press pages.
- Takeaway: Treat outbound links as an asset inventory with an expiry date — audit them on a schedule and remove or update the ones whose destinations have been abandoned, since a link from an official page lends the attacker's account the organization's credibility.

### 2026-09-24 — Django Debug Panel exposed without access control on a non-production Relay host (Mozilla) — n/a
- Source: [HackerOne #2078707](https://hackerone.com/reports/2078707)
- Type: Security misconfiguration — development debug tooling reachable from the internet
- Summary: The Django Debug Panel was left enabled and unauthenticated on `dev.fxprivaterelay.nonprod.cloudops.mozgcp.net`, exposing request internals, settings and database query detail that a debugging interface surfaces by design.
- Technique / pattern: Enumerate `nonprod`, `dev` and `stage` hostname components from certificate transparency logs and passive DNS, then probe the framework's known debug paths. Development deployments inherit the production domain's trust but rarely its access controls.
- Takeaway: Debug tooling is a production-grade information leak wherever it is reachable; gate it on an environment flag that cannot be true on an internet-facing host rather than on a hostname convention.

### 2026-09-24 — Nginx misconfiguration allows downloading PHP source directly, including SAML SSO configuration (GSA Bounty) — n/a
- Source: [HackerOne #268382](https://hackerone.com/reports/268382)
- Type: Security misconfiguration — server returns script files as static content
- Summary: An Nginx `location` rule failed to hand some `.php` paths to the PHP handler, so the raw source was returned instead of executed; the files retrievable this way included SAML single sign-on configuration.
- Technique / pattern: Where a server mixes static and interpreted content, probe the cases the handler rule misses — alternate extensions and casing, a trailing dot or slash, and files under directories that have their own `location` block. Source returned as text is the signal.
- Takeaway: Source disclosure hands over the secrets embedded in configuration; make the interpreter mapping deny-by-default so an unmatched path is refused rather than served as a file.

### 2026-09-24 — Bulk directory listing exposure across multiple NASA subdomains (NASA VDP) — n/a (Informational)
- Source: [Bugcrowd fb4354a1](https://bugcrowd.com/disclosures/fb4354a1-3cd8-4395-a619-2ae39b986c5f/bulk-directory-listing-exposure-on-multiple-nasa-subdomains)
- Type: Security misconfiguration — autoindex enabled on public web roots
- Summary: Several NASA hosts, among them `neo.gsfc.nasa.gov/archive` and `maps.nccs.nasa.gov/download`, returned browsable directory indexes revealing file layout and unlinked files. The program accepted the risk, as those archives are published deliberately for contributors and mirrors.
- Technique / pattern: Sweep a subdomain list for index pages rather than treating one hit as the finding, then judge each by what the listing actually reveals — backups, configuration or unlinked data versus an intentional mirror. Reporting the index alone, with no sensitive file behind it, is what pushes these to informational.
- Takeaway: Directory listing is a severity multiplier, not a vulnerability in itself; the report lands only when the index leads to a file that was never meant to be public.

### 2026-09-24 — Directory listing on a SATCORPS web server exposes PHP source across multiple directories (NASA VDP) — n/a (P5)
- Source: [Bugcrowd 26e3e95b](https://bugcrowd.com/disclosures/26e3e95b-a1af-4baf-afc0-8b05de8abef5/source-code-disclosure)
- Type: Security misconfiguration — directory listing leading to source disclosure
- Summary: A misconfigured directory setting on a NASA SATCORPS server made PHP source files browsable and readable across several directories. NASA rated the finding informational, citing its open-source development posture.
- Technique / pattern: Chain the two findings instead of reporting the index on its own — enumerate the listed directories, fetch the source returned as text, and grep it for credentials, internal hostnames and database connection strings. That grep is what separates an informational listing from a real disclosure.
- Takeaway: Whether exposed source matters depends on what is in it; an organisation that publishes its code openly has already accepted the exposure, so lead with the secret you found rather than with the listing.

### 2026-09-24 — Broken access control: publicly editable NASA research documents holding active mission data (NASA VDP) — n/a (P3)
- Source: [Bugcrowd efe6dc8e](https://bugcrowd.com/disclosures/efe6dc8e-7364-42c0-8467-2371d610d82a/vulnerability-report-broken-access-control-ongoing-project-publicly-editable-google-sheet-owned-by-senior-nasa-researcher)
- Type: Security misconfiguration — third-party document sharing left at "anyone with the link can edit"
- Summary: Five interconnected documents owned by a senior NASA researcher were shared with *edit* permission to anyone holding the link. They contained active operational project-management data for 12+ Earth Science missions, daily task assignments for 6+ researchers and Gantt charts of mission timelines, so any unauthenticated visitor could modify, delete or silently corrupt live planning data. The link was discovered on a public NASA forum.
- Technique / pattern: Attack surface includes the SaaS documents an organization links to, not only the hosts it runs. Harvest Google Docs/Sheets, Airtable, Notion and Figma URLs from public forums, mailing lists, slide decks, PDFs and cached pages within scope, then check each one's permission level while signed out — and check *write* access specifically, since read-only sharing is common and edit access is the actual finding.
- Takeaway: The impact here is integrity and availability, not just disclosure, and silent modification of operational data is far harder to detect than a leak. Name the write capability explicitly when reporting: "publicly readable" and "publicly editable" are different severities.

### 2026-09-24 — Unauthorized access to CI/CD infrastructure and project secrets via exposed GitLab Runner token (NASA VDP) — n/a (P1)
- Source: [Bugcrowd cc46ad29](https://bugcrowd.com/disclosures/cc46ad29-f297-4847-abcd-9f5da5a85621/unauthorized-access-to-ci-cd-infrastructure-and-project-secrets-via-compromised-gitlab-runner-token)
- Type: Security misconfiguration — leaked CI runner registration credential
- Summary: A publicly exposed GitLab Runner registration token let an external researcher register and authenticate an unauthorized runner inside NASA's SMCE CI/CD infrastructure. A runner that joins the fleet can be assigned jobs carrying CI/CD secrets and can influence build output, which is a supply-chain impact; further exploitation was intentionally avoided.
- Technique / pattern: CI registration tokens are long-lived bearer credentials, and they are routinely committed to repositories, pasted into runbooks and left inside `.gitlab-ci.yml` or provisioning scripts. Grep exposed repositories, CI configuration and documentation for runner-registration commands and token prefixes. The proof of impact is the registration itself — a runner that successfully joins is authorized to receive jobs and the variables attached to them — so there is no need to actually harvest a secret to demonstrate the boundary is open.
- Takeaway: Treat CI registration tokens like deploy keys, not like configuration. Demonstrate impact by joining the fleet and stopping there: it proves the trust boundary while keeping the test non-destructive.

### 2026-09-24 — NASA LSDA/NLSP data platform allows unauthenticated write to production Elasticsearch cluster (NASA VDP) — n/a (P2)
- Source: [Bugcrowd 2a6e8df3](https://bugcrowd.com/disclosures/2a6e8df3-c772-4349-9940-00a9b4ad417f/nasa-lsda-nlsp-data-platform-allows-unauthenticated-write-to-production-elasticsearch-cluster)
- Type: Security misconfiguration — authorization middleware not reached on part of the ingestion path
- Summary: A NASA science data platform exposed data-ingestion API routes that accepted writes from unauthenticated users. The access-control layer correctly classified anonymous requesters as having no write permission and enforced that on some models — an anonymous write to one returned `403` with an explicit authorization denial — but other models on the same ingestion path were written without the check ever running. Requests that failed did so on *field validation* rather than authentication, confirming there was no auth boundary in front of the write handler. The platform's schemas were also readable unauthenticated, supplying the field names needed to build a valid write.
- Technique / pattern: Use the sibling endpoint that correctly denies you as the oracle for the ones that do not. Enumerate every model or resource on the same path and compare *error shapes*, not status codes alone: a `403 authorization denied` means the guard ran, whereas a `400 missing required field` means the request already reached the handler. The researcher confirmed non-destructively with clearly marked test records containing no real data, then reported them for cleanup.
- Takeaway: Differential error analysis across sibling resources isolates the one handler that skipped the middleware. A validation error is a stronger signal than a success: it proves you are past authentication even when the write itself fails.

### 2026-09-24 — CORS misconfiguration / broken access control on admin.myndr.net (Myndr) — n/a (CVSS 7.5)
- Source: [HackerOne #3930102](https://hackerone.com/reports/3930102)
- Type: Security misconfiguration — credentialed CORS reflecting any subdomain origin
- Summary: `admin.myndr.net` reflected any `*.myndr.net` origin into `access-control-allow-origin` together with `access-control-allow-credentials: true`, so any page hosted on a Myndr subdomain could read authenticated admin-panel responses — including CSRF nonces, session data and admin functionality — and chain that into full admin account takeover. Reported endpoints included `/`, `/auth/login-admin`, `/auth/login-admin-new-password` and `/cp/postcode`.
- Technique / pattern: Send `Origin: https://evil.myndr.net` on an unauthenticated request and read the response headers; a reflected origin plus `access-control-allow-credentials: true` and `vary: Origin` is the entire proof. The escalation is the half that matters: once a credentialed cross-origin read works, the CSRF nonce that was the last control on state-changing admin routes becomes readable, so a CORS read turns into a CSRF write.
- Takeaway: Wildcard-subdomain trust collapses the origin boundary onto the weakest subdomain an attacker can obtain — a takeover, a user-content host, a forgotten staging box. Before rating a reflected-subdomain CORS finding informational, ask what a readable admin response hands you next.

### 2026-09-23 — Subdomain takeover of an abandoned subdomain pointing to Google infrastructure (Ubiquiti (HackerOne)) — n/a
- Source: [HackerOne #181665](https://hackerone.com/reports/181665)
- Type: Security misconfiguration / dangling DNS → subdomain takeover
- Summary: `moderator.ubnt.com` still pointed (via `ghs.google.com`) to a Google-hosted service that had never been claimed, so an attacker could register the subdomain through Google's normal onboarding and serve content from a Ubiquiti hostname.
- Technique / pattern: Found by resolving the subdomain, seeing it aliased to a shared hosting endpoint (`ghs.google.com`) with no active tenant, and confirming the hosting provider would let a new account claim that hostname.
- Takeaway: A `CNAME` to a third-party SaaS/hosting endpoint with no active account behind it is takeover-prone. Verify the target service actually has a claimed, live tenant, and remove stale aliases.

### 2026-09-23 — Subdomain takeover via dangling DNS to a decommissioned EC2 instance (8x8 (HackerOne)) — n/a
- Source: [HackerOne #1101877](https://hackerone.com/reports/1101877)
- Type: Security misconfiguration / dangling DNS → subdomain takeover
- Summary: An EC2 instance was retired but its DNS record was left pointing at the released address, leaving an 8x8 subdomain claimable by whoever next obtained that infrastructure.
- Technique / pattern: Standard dangling-DNS review: enumerate subdomains and flag records that resolve to cloud infrastructure no longer owned by the target (a released elastic IP or an unclaimed service endpoint).
- Takeaway: Decommissioning must remove or update DNS as part of teardown. Continuously monitor for records pointing at cloud resources you no longer control — they are directly claimable.

### 2026-09-23 — Grafana admin access via default credentials (Kistler (Bugcrowd)) — n/a (P1, 40 points)
- Source: [Bugcrowd f810da90](https://bugcrowd.com/disclosures/f810da90-2aff-4970-b6b9-09a471e1b805/grafana-admin-login-via-default-credentials)
- Type: Security misconfiguration / default credentials
- Summary: A public-facing Grafana instance still used its default administrator login, granting full admin access to the monitoring and dashboard platform.
- Technique / pattern: Third-party dashboard and monitoring software (Grafana, Kibana and similar) frequently ships with a well-known default admin login that survives deployment; checking that default against an exposed instance is the whole test.
- Takeaway: Fingerprint exposed third-party software and check each product's documented default login. Monitoring stacks are often stood up quickly and left with defaults, yet expose infrastructure detail and sometimes data-source credentials.

### 2026-09-23 — Unauthorised admin access via unchanged default password (Tesla (Bugcrowd)) — n/a (P2, 20 points)
- Source: [Bugcrowd 82bb3923](https://bugcrowd.com/disclosures/82bb3923-4097-4a64-a5f7-d5f6e59f1b6d/unauthorised-admin-access-due-to-default-password)
- Type: Security misconfiguration / default credentials
- Summary: An administrative panel was reachable and still protected only by its shipped default password, letting anyone who found the panel log in with admin privileges.
- Technique / pattern: The finding is the pairing of an exposed admin interface with an unchanged vendor default login — located by discovering the admin path, then trying the product's documented default credentials.
- Takeaway: Default credentials on an internet-reachable admin panel remain one of the highest-impact, lowest-effort findings. Inventory admin interfaces and confirm shipped defaults were rotated before go-live.

### 2026-09-23 — Source code leakage due to exposed sourcemap (Atlassian — Bitbucket Cloud) — $200 (P4)
- Source: [Bugcrowd #8cc82a04](https://bugcrowd.com/disclosures/8cc82a04-b1b2-452a-9e68-b332e4fc7794/source-code-leakage-due-to-exposed-sourcemap)
- Type: Security misconfiguration / source code disclosure
- Summary: Production JavaScript on `bitbucket.org` shipped alongside its source maps, letting anyone reconstruct the original front-end source tree straight from the browser. Disclosed 2021-01-22; accepted by Atlassian as a known business risk.
- Technique / pattern: Open developer tools against the target's production bundles and check the Sources tree for mapped original files, or request the `.map` file appended to each bundle URL, then dump the recovered tree.
- Takeaway: Source maps are a build-config leftover rather than a vulnerability in themselves — but the recovered source is where hidden endpoints, feature flags and client-side secrets are found, so treat it as recon input.

### 2026-09-23 — Public Apache Server Status Endpoint Causes Sensitive Data Exposure of Internal System Information (NASA VDP) — n/a (P4)
- Source: [Bugcrowd #7c9313ff](https://bugcrowd.com/disclosures/7c9313ff-516f-4f8f-820f-eb693843f2b2/public-apache-server-status-endpoint-causes-sensitive-data-exposure-of-internal-system-information)
- Type: Security misconfiguration / server hardening
- Summary: An Apache diagnostic status page was reachable without authentication or IP restriction on a NASA subdomain, exposing server software version, platform details, uptime and load metrics, and worker process state. Disclosed 2026-04-16.
- Technique / pattern: Passive reconnaissance plus non-intrusive probing of well-known diagnostic paths such as `/server-status` on each host; the finding is a configuration oversight, so no exploitation beyond loading the page was needed.
- Takeaway: Diagnostic endpoints that ship enabled by default are a reliable low-effort check on every host — their value to an attacker is the precise version fingerprint they hand over for targeting known CVEs.

### 2026-09-23 — Subdomain takeover via a DNS record pointing to a deleted S3 bucket (Affirm) — n/a
- Source: [HackerOne #1297689](https://hackerone.com/reports/1297689)
- Type: Security misconfiguration / subdomain takeover (dangling DNS)
- Summary: A subdomain still carried a DNS record pointing at an AWS S3 bucket that no longer existed; the researcher claimed the bucket name and served their own content from the company domain.
- Technique / pattern: Resolve every known subdomain, flag the ones whose targets return cloud-provider "no such bucket" errors, then register the referenced resource and host a proof file — often enough to also obtain a valid TLS certificate for the name.
- Takeaway: Decommissioning a cloud resource must include removing the DNS record; a dangling `CNAME` hands an attacker a trusted origin for phishing, cookie theft and certificate issuance.

### 2026-09-23 — S3 Bucket Takeover: brave-apt (Brave Software) — n/a
- Source: [HackerOne #1791558](https://hackerone.com/reports/1791558)
- Type: Security misconfiguration / cloud storage takeover (supply chain)
- Summary: An S3 bucket at `https://s3-us-west-2.amazonaws.com/brave-apt/`, used in the Linux installation path for the Brave browser, was left unclaimed and openly reachable, so an attacker who registered the bucket name could serve content into that distribution flow.
- Technique / pattern: Enumerate the program's infrastructure for bucket references in install scripts and documentation, request each bucket URL directly, and check whether the name resolves to an unowned bucket that can be re-registered.
- Takeaway: Buckets referenced by installers and package repositories are supply-chain assets — an unclaimed one is not an information leak, it is code execution on every machine that installs from it.

### 2026-09-22 — Broken link hijacking (impersonation) on jpl.nasa.gov via unregistered Facebook URL (NASA VDP) — n/a (P4)
- Source: [Bugcrowd #47e4872e](https://bugcrowd.com/disclosures/47e4872e-88bd-4dd2-b943-1b30f4d5b4ad/broken-link-hijacking-impersonation-on-jpl-nasa-gov-via-unregistered-facebook-url)
- Type: Broken link hijacking / dangling third-party identity
- Summary: A JPL news article linked to an official Solar System Exploration Facebook page using a misspelled URL, so the handle it pointed at was unregistered; anyone could claim it and receive traffic that users believe is NASA's official account.
- Technique / pattern: Crawl the target's pages for outbound links to social platforms, app stores, package registries and short-link services, then check each destination for "profile not found" / "account unavailable" responses and whether the identifier is still claimable. Typos in hand-written links are a common source of these.
- Takeaway: Trust is inherited from the linking page, so an abandoned or misspelled external handle is an impersonation primitive — treat outbound-link inventory as part of dangling-asset triage alongside CNAMEs.

### 2026-09-22 — Clickjacking on cdn.sit.earthdata.nasa.gov (NASA VDP) — n/a (P5, informational)
- Source: [Bugcrowd #058fc473](https://bugcrowd.com/disclosures/058fc473-2bab-4bb8-86c9-22e64ef1d665/clickjacking-vulnerability-cdn-sit-earthdata-nasa-gov)
- Type: Missing framing protection (`X-Frame-Options` / CSP `frame-ancestors`)
- Summary: A staging CDN host could be embedded in an attacker-controlled frame because it sent no anti-framing header; the program accepted it as an informational business risk, since the framed content supported no sensitive state-changing action.
- Technique / pattern: Framing checks are cheap recon (`curl -I` for `X-Frame-Options` / `Content-Security-Policy`), but severity comes from what a framed click *does* — look for one-click destructive or privilege-granting actions behind the frame before reporting, and check staging hostnames (`sit.`, `uat.`, `dev.`) separately from production.
- Takeaway: A missing header is a finding only when a click inside the frame has consequences; otherwise expect informational triage, and spend the effort on chaining it instead.

### 2026-09-22 — Amazon S3 bucket misconfiguration allows arbitrary uploads via exposed credentials (BCM Messenger) — n/a (bounty awarded, amount not shown)
- Source: [HackerOne #764243](https://hackerone.com/reports/764243)
- Type: Cloud storage misconfiguration + credentials exposed by an API
- Summary: The messenger's attachment API returned AWS access keys and pre-signed upload policies to any client, and the bucket accepted the resulting uploads without further checks; since account creation needed no verification, anyone could mint accounts and use the company's bucket as free (and abusable) file hosting.
- Technique / pattern: For mobile targets, bypass SSL pinning (Frida) to read the real API traffic, then inspect responses for credential material — access keys, pre-signed URLs, upload policies — and test what the returned credential actually permits (`put`, `list`, `delete`) rather than assuming it is scoped.
- Takeaway: Never hand long-lived cloud credentials to a client; issue narrowly scoped, short-lived pre-signed URLs per object, and remember that unverified signup turns any client-side credential into a public one.

### 2026-09-22 — S3 ACL misconfiguration exposes all stored media to any AWS user (Legal Robot) — n/a
- Source: [HackerOne #189023](https://hackerone.com/reports/189023)
- Type: Cloud storage misconfiguration (over-permissive bucket ACL)
- Summary: The company's S3 bucket granted access broadly enough that any account holding AWS credentials could list and copy every object — images, PDFs and video, including the asset used on the homepage.
- Technique / pattern: Find the bucket from asset URLs in page source, then test it *authenticated* as an arbitrary AWS user, not just anonymously: `aws s3 ls` and a single-object `cp` prove read access without touching data. An `AuthenticatedUsers` grant looks closed to an unauthenticated probe and open to everyone with a free AWS account.
- Takeaway: Test buckets with your own AWS identity as well as anonymously, and enumerate the verb set carefully — read proves the leak; avoid delete/overwrite tests on live assets.
### 2026-09-22 — Unauthenticated phpinfo() files could lead to ability file read (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1794884](https://hackerone.com/reports/1794884)
- Type: Debug/diagnostic file left in production (exposed `phpinfo()`)
- Summary: A leftover `info.php` calling `phpinfo()` was reachable without authentication on a DoD host, exposing OS details, PHP build and loaded extensions, configuration directives and environment variables; resolved by removing the file.
- Technique / pattern: Found through directory/content discovery for common diagnostic filenames (`info.php`, `phpinfo.php`, `test.php`); the page itself is the finding, and its paths, `open_basedir`/`disable_functions` values and env vars are recon that makes later file-read or RCE attempts precise.
- Takeaway: Include diagnostic filenames in every content-discovery wordlist, and when you hit one, report it with what it concretely reveals (secrets in env vars, absolute paths, disabled protections) rather than as "version disclosure".

### 2026-09-22 — Exposed wp-config.php file (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #3252302](https://hackerone.com/reports/3252302)
- Type: Server misconfiguration exposing config file
- Summary: A WordPress `wp-config.php` was directly readable on an in-scope DoD host, leaking MySQL and AWS credentials and secret keys.
- Technique / pattern: Requested the config file's path directly and received its raw contents instead of PHP execution / a denial.
- Takeaway: Keep `wp-config.php` and its backup/editor copies (e.g. `wp-config.php.bak`, `wp-config.php~`) in recon wordlists; defenders should block config files at the web server and rotate exposed secrets.

### 2026-09-22 — Several internal applications have open CORS (Tesla) — n/a (20 points)
- Source: [Bugcrowd #0e3f3821](https://bugcrowd.com/disclosures/0e3f3821-1d26-466b-8599-7cc2f206f4d9/several-internal-applications-have-open-cors-allowing-external-folks-to-access-the-content)
- Type: CORS misconfiguration (P2)
- Summary: About ten internal Tesla apps (e.g. `payment-gateway.teslamotors.com`, `location.teslamotors.com`) had permissive CORS, letting an attacker-controlled origin read their responses from a browser on the internal network.
- Technique / pattern: Used a typosquat domain to get an internal-network browser to load attacker JS, then showed permissive `Access-Control-Allow-Origin` handling let that origin read the internal services' responses.
- Takeaway: Internal apps are reachable via employees' browsers; a permissive CORS policy on an intranet service turns any visited attacker page into an internal data reader.

### 2026-09-22 — Subdomain takeover on worldcup.starlink.com (SpaceX / Starlink) — n/a (5 points)
- Source: [Bugcrowd #af807529](https://bugcrowd.com/disclosures/af807529-67e4-4e48-8933-bbe135639a7e/subdomain-takeover-on-worldcup-starlink-com)
- Type: Subdomain takeover (P4)
- Summary: The Starlink-owned `worldcup.starlink.com` could be claimed by an attacker; the disclosure does not name the third-party service involved.
- Technique / pattern: General method: find a dangling DNS record (usually a CNAME) pointing to a deprovisioned third-party resource and register that resource name on the provider to serve content on the company's subdomain.
- Takeaway: Short-lived campaign/event subdomains are prime takeover candidates; decommissioning a resource must include removing its DNS record.

### 2026-09-21 — Silent webcam activation via Loom extension web-accessible resource (Atlassian) — $800
- Source: [Bugcrowd #6c751112](https://bugcrowd.com/disclosures/6c751112-6cd1-4a9a-90f1-d502c164ea94/unauthorized-silent-webcam-activation-via-loom-chrome-extension-web-accessible-resources)
- Type: Security misconfiguration (extension web_accessible_resources)
- Summary: Loom's Chrome extension (v5.5.173) exposed `bubble.html` as a web-accessible resource to all origins; any site could iframe it and, because camera permission persisted, silently activate the webcam.
- Technique / pattern: Embed the extension's `bubble.html` in an iframe from an attacker page; with `web_accessible_resources` declared for `all_urls` and no origin check before camera init, the cached permission triggers a live feed.
- Takeaway: Declare extension `web_accessible_resources` for the narrowest origins possible and verify the embedding origin before touching sensitive APIs — persistent permissions plus open iframing equals silent abuse.

### 2026-09-21 — Swagger UI injection via `configUrl` parameter (U.S. Dept of Defense)
- Source: [HackerOne #3124103](https://hackerone.com/reports/3124103)
- Type: Security misconfiguration / resource injection (Swagger UI)
- Summary: A hosted Swagger UI loaded its configuration from a user-controlled `configUrl` parameter, letting an attacker point it at an arbitrary JSON spec and manipulate UI behavior / reach XSS.
- Technique / pattern: Supply `?configUrl=` referencing an attacker-hosted spec; Swagger UI fetches and renders it, so malicious spec content drives the page — a resource-injection sink from an unvalidated URL parameter.
- Takeaway: Never let Swagger/OpenAPI UIs take their spec location from user input — pin `configUrl`/`url` server-side and validate any externally loaded document.


### 2026-09-21 — Directory listing exposes sensitive log files (U.S. Department of Labor) — n/a
- Source: [Bugcrowd disclosure eac8d6fd](https://bugcrowd.com/disclosures/eac8d6fd-4786-4664-89e7-23102e03a9f0/sensitive-log-files-exposed-to-public-through-directory-listing)
- Type: Security Misconfiguration (directory listing, P2)
- Summary: Directory indexing was left enabled, making log files that should have been private publicly browsable.
- Technique / pattern: Browse the directory with indexing on, enumerate and download the exposed log files directly from the listing.
- Takeaway: Disable auto-indexing (`Options -Indexes` / `autoindex off;`) — an open directory turns any stray log or backup file into a data leak.

### 2026-09-21 — Exposed `.git/` directory leaks source and config on NASA subdomain (NASA VDP) — n/a
- Source: [Bugcrowd disclosure 976f4f72](https://bugcrowd.com/disclosures/976f4f72-88b9-43e5-a6c5-b6cb79f1ce84/exposed-git-directory)
- Type: Security Misconfiguration (source exposure, P3)
- Summary: A NASA subdomain served its `.git/` directory publicly, exposing internal source code and configuration files.
- Technique / pattern: Request `/.git/` on the host and pull the repository contents (git-dumper-style) to recover source and any secrets committed to config.
- Takeaway: Deploying from a git working tree without blocking `.git/` at the web server hands out full source and history — deny the path and deploy build artifacts only.

### 2026-09-21 — Subdomain takeover of rtncf-rci.ral.r4.fws.gov via lapsed .org domain (U.S. Fish & Wildlife) — n/a
- Source: [Bugcrowd disclosure 629b48ac](https://bugcrowd.com/disclosures/629b48ac-85c6-4af2-8702-a0996c874df0/subdomain-takeover-for-rtncf-rci-ral-r4-fws-gov)
- Type: Security Misconfiguration (subdomain takeover, P3)
- Summary: A government subdomain held a CNAME to `rtncf-rci.ncusfws.org`, an external domain that had lapsed and become purchasable.
- Technique / pattern: Detect the dangling CNAME target, register the abandoned `.org` for ~$7, and serve controlled content — proving takeover of the `.gov` subdomain.
- Takeaway: Dangling CNAMEs point at ordinary registrable domains as often as at cloud services — add a domain-registrability check to takeover triage, not just cloud-provider fingerprints.

### 2026-09-21 — Subdomain takeover of annualmeeting2022.globe.gov via dangling Meteor CNAME (NASA VDP) — n/a
- Source: [Bugcrowd disclosure 1fdbc7d6](https://bugcrowd.com/disclosures/1fdbc7d6-5a1a-45dc-a710-abbfff353cb8/subdomain-takeover-on-annualmeeting2022-globe-gov)
- Type: Security Misconfiguration (subdomain takeover, P3)
- Summary: `annualmeeting2022.globe.gov` still pointed at `us-east-1.galaxy-ingress.meteor.com`, a Meteor hosting endpoint no longer in use, allowing a takeover.
- Technique / pattern: Confirm the dangling CNAME to the Meteor Galaxy ingress, register a Meteor account and deploy an app on that hostname to serve attacker content from the trusted NASA subdomain.
- Takeaway: After an event/campaign ends, its subdomain's CNAME often outlives the hosting account — decommission DNS records with the service, and audit CNAMEs pointing at PaaS ingress hostnames.

### 2026-09-21 — S3 bucket writeable by any authenticated AWS user (HackerOne) — bounty
- Source: [HackerOne #128088](https://hackerone.com/reports/128088)
- Type: Misconfiguration (S3 ACL — AuthenticatedUsers)
- Summary: A HackerOne S3 bucket granted write to the AWS 'AuthenticatedUsers' group, meaning any AWS account (not just HackerOne's) could upload objects into it.
- Technique / pattern: Enumerate buckets named similarly to a known one (hackerone-attachments) and attempt a write via AWS CLI; the AuthenticatedUsers ACL accepts uploads from any AWS principal, enabling malicious file placement.
- Takeaway: 'AuthenticatedUsers' in S3 means every AWS user on earth, not your users — an anonymous probe looks closed while authenticated writes succeed, so always test with valid AWS creds.

### 2026-09-21 — Subdomain takeover at info.hacker.one via dangling Unbounce CNAME (HackerOne) — n/a
- Source: [HackerOne #202767](https://hackerone.com/reports/202767)
- Type: Misconfiguration (subdomain takeover)
- Summary: info.hacker.one had a DNS CNAME to app.unbounce.com; a flaw in Unbounce's domain-claim flow let the researcher claim the subdomain and host arbitrary content.
- Technique / pattern: Identify the dangling CNAME to a third-party SaaS, then exercise that provider's (weak) domain-claim process to serve attacker content on the trusted subdomain, enabling phishing/credential theft.
- Takeaway: Fingerprint the SaaS behind every dangling CNAME and test its specific claim flow — takeover often hinges on the provider's verification weakness, not just an unclaimed record.

### 2026-09-21 — AWS subdomain takeover via dangling CNAME to an unclaimed S3 bucket (U.S. Dept of Defense) — n/a
- Source: [HackerOne #1329792](https://hackerone.com/reports/1329792)
- Type: Misconfiguration (subdomain takeover, S3)
- Summary: A www subdomain's CNAME pointed at an unclaimed S3 bucket; registering that bucket allowed takeover and PoC content on the DoD subdomain.
- Technique / pattern: Detect the dangling DNS record, create the S3 bucket matching the CNAME target, and host a PoC; the browser trusts the original subdomain, enabling cookie theft, CORS/CSP bypass, SSRF pivots and phishing.
- Takeaway: The takeover window opens when the resource is deleted but the DNS record survives — decommissioning order matters; remove the DNS record before (or with) the backing resource.

### 2026-09-21 — Open S3 bucket accessible/listable by any user (omise-cdn-2) (Omise) — $100
- Source: [HackerOne #1474017](https://hackerone.com/reports/1474017)
- Type: Misconfiguration (open S3 bucket)
- Summary: The omise-cdn-2 bucket (served at cdn2.omise.co) allowed unauthenticated listing, reading and downloading of all contents through a browser.
- Technique / pattern: Browse the bucket URL directly to list and pull every object; permissive bucket policy exposes all files and, with write, risks full bucket takeover.
- Takeaway: Enumerate a bucket's full verb set (list, read, then a harmless write marker) rather than assuming read-only — public read plus write can escalate to takeover.


### 2026-09-21 — Exploiting misconfigured CORS to steal user information (Rockstar Games) — $500
- Source: [HackerOne #317391](https://hackerone.com/reports/317391)
- Type: CORS misconfiguration
- Summary: A misconfigured CORS policy on a gateway behind Rockstar's Support site let a third-party origin read users' details (email addresses, account IDs) via credentialed cross-origin requests; it was fixed by removing the leaking gateway.
- Technique / pattern: Find an authenticated endpoint returning user data with permissive CORS headers, then host a PoC page on an attacker origin that makes credentialed cross-origin requests and reads the response.
- Takeaway: Endpoints returning user data must never allow arbitrary or reflected origins together with credentials; keep a strict origin allowlist and remove legacy gateways that carry old loose CORS rules.

### 2026-09-21 — Web cache poisoning at www.acronis.com (Acronis) — n/a
- Source: [HackerOne #1010858](https://hackerone.com/reports/1010858)
- Type: Web cache poisoning (unkeyed headers and parameters)
- Summary: The cache on www.acronis.com ignored certain query parameters and the x-forwarded-port and x-forwarded-url headers when building its key even though they changed the response, allowing poisoned responses to be cached and served to normal users (DoS and delivery of reflected XSS). Bountied, amount not public.
- Technique / pattern: Send requests with non-standard headers/parameters to find inputs that change the response without changing the cache key, then get a tampered response cached so later requests for the same URL receive it.
- Takeaway: Test X-Forwarded-Port, X-Forwarded-URL, X-Original-URL and ignored query parameters as poisoning inputs; normalize or drop them at the CDN, or don't cache pages that reflect them.

### 2026-09-21 — Subdomain takeover of brand.zen.ly (Zenly) — $750
- Source: [HackerOne #1474784](https://hackerone.com/reports/1474784)
- Type: Subdomain takeover (dangling CNAME to third-party SaaS)
- Summary: brand.zen.ly had a CNAME to a Brandpad service no longer in use that returned a "Not Found" page, so anyone could register with Brandpad and claim the subdomain for phishing, malware, XSS or cookie theft.
- Technique / pattern: Enumerate subdomains, resolve their CNAMEs, match a third-party provider's "unclaimed"/"Not Found" fingerprint, then register the name on that provider to claim it.
- Takeaway: When a SaaS integration is retired, remove its DNS record in the same change; regularly audit CNAMEs pointing to external services and flag any returning a provider's "no such site" page.

### 2026-09-21 — Defacement of catalog.data.gov via web cache poisoning to stored DOM XSS (GSA Bounty) — $750
- Source: [HackerOne #303730](https://hackerone.com/reports/303730)
- Type: Web cache poisoning (unkeyed X-Forwarded-Host)
- Summary: catalog.data.gov trusted the unkeyed X-Forwarded-Host header to fill data-site-root/data-locale-root attributes; client JS then fetched JSON from that host and inserted it unescaped, and CloudFront cached the poisoned page and served it to others, giving stored DOM XSS and defacement.
- Technique / pattern: Find a request header that changes the response but is not in the cache key, point X-Forwarded-Host at an attacker-controlled JSON endpoint, and repeat until the CDN caches the poisoned page.
- Takeaway: Any header that changes the response must be in the cache key or stripped at the edge; do not build resource URLs from X-Forwarded-* headers unless the fronting proxy sets and overwrites them.

### 2026-09-21 — Possible Subdomain Takeover For Inbound Emails (Smule) — n/a
- Source: [HackerOne #2567048](https://hackerone.com/reports/2567048)
- Type: Subdomain takeover - dangling CNAME to email provider (SendGrid)
- Summary: email.smule.com had a CNAME pointing at SendGrid, but the subdomain was not claimed on SendGrid, so an attacker could register it and potentially receive inbound email for that host.
- Technique / pattern: Enumerated DNS CNAMEs pointing at third-party SaaS, then checked whether the provider showed the host as unclaimed (404) and registrable.
- Takeaway: Subdomain takeover is not just web hosting - dangling records to email/inbound-parse providers can let attackers intercept mail. Remove DNS records when deprovisioning SaaS.

### 2026-09-21 — Cache poisoning Denial of Service affecting assets.gitlab-static.net (GitLab) — n/a
- Source: [HackerOne #1160407](https://hackerone.com/reports/1160407)
- Type: Web cache poisoning - unkeyed X-HTTP-Method-Override header (DoS)
- Summary: GitLab's static-asset CDN honored x-http-method-override, but the cache did not key on it; a GET with x-http-method-override: HEAD produced an empty body that was cached and served to every user of that asset.
- Technique / pattern: Probed unkeyed headers with cache-busters, found that the method-override header changed the backend response without changing the cache key, and confirmed the empty poisoned response was served for normal GETs.
- Takeaway: Every header that changes the response must be in the cache key or stripped at the edge; method-override headers are a quiet way to cache empty responses on critical assets.

### 2026-09-21 — Memory Dump and Env Disclosure via Spring Boot Actuator (Stripo) — n/a
- Source: [HackerOne #1019367](https://hackerone.com/reports/1019367)
- Type: Exposed management endpoints - Spring Boot Actuator (/heapdump, /env)
- Summary: Spring Boot Actuator endpoints were reachable under a non-root path prefix, exposing /env and a ~110 MB heap dump containing source code, private cryptographic keys and internal application data.
- Technique / pattern: Fuzzed for /actuator/* not only at the web root but beneath application sub-paths, then downloaded /heapdump and mined it for secrets.
- Takeaway: Actuator endpoints must be disabled or authenticated in production; when hunting, fuzz /actuator under every discovered base path, and treat heapdumps as credential goldmines.

### 2026-09-21 — Information disclosure via enabled Django Debug Mode (MTN Group) — n/a
- Source: [HackerOne #2201370](https://hackerone.com/reports/2201370)
- Type: Debug mode enabled in production (Django DEBUG=True)
- Summary: An MTN application ran with DEBUG=True, leaking stack traces, configuration and the URL/endpoint map; from that the researcher found open user registration, user/email enumeration and a DNS-records endpoint that could expose origin IPs.
- Technique / pattern: Triggered errors (e.g. nonexistent routes) to render the Django debug page, harvested the listed URL patterns, then probed the disclosed endpoints for further issues.
- Takeaway: Debug pages hand attackers a complete route map - one misconfiguration seeds a chain of follow-on bugs. Always check 404 debug pages for URLconf listings.

### 2026-09-20 — Subdomain takeover in GitLab Pages through the unverified-custom-domain grace period (GitLab) — n/a
- Source: [HackerOne #2523654](https://hackerone.com/reports/2523654)
- Type: Security misconfiguration (service serves a custom domain before verifying ownership)
- Summary: GitLab Pages kept serving a custom domain for roughly seven days before disabling it when domain verification was never completed, so an attacker could attach a dangling domain to their own Pages project and serve content from it during that window. Testing indicated GitLab-owned hostnames such as `docs-dev.gitlab.com` were affected.
- Technique / pattern: A verification requirement that is enforced *eventually* rather than *before first byte* is a takeover window, not a control. For each PaaS that supports custom domains, read its documented verification flow and ask three questions: does it serve before verification, how long is the grace period, and is the DNS record required to exist at claim time. A provider with a grace period turns "dangling CNAME" into "dangling CNAME for seven days at a time", which survives ordinary reconciliation scans that run weekly.
- Takeaway: Verify ownership before serving, not after. When auditing, record the provider's grace-period behaviour alongside its claim flow — it decides whether a stale record is exploitable.

### 2026-09-20 — Potential subdomain takeover on an ibm.com hostname (IBM) — n/a
- Source: [HackerOne #3592387](https://hackerone.com/reports/3592387)
- Type: Security misconfiguration (dangling DNS record on a high-trust apex domain)
- Summary: A researcher identified an IBM hostname whose DNS pointed at a third-party resource that was no longer claimed, leaving the subdomain takeover-able. IBM analysed the report and remediated it.
- Technique / pattern: On an organisation with decades of DNS history, the yield comes from breadth rather than cleverness — pull hostnames from certificate transparency logs, passive DNS and the program's own published asset list, resolve all of them, and triage by the *shape* of the failure (NXDOMAIN on the CNAME target, a provider's unclaimed-host error page, a SERVFAIL on delegated NS records). Report the condition with the resolution chain as evidence rather than claiming the resource, which keeps the finding in scope.
- Takeaway: Brand trust is the impact: content served from a hostname under a major apex domain inherits that domain's credibility for phishing and cookie scoping. Reconcile DNS against live inventory continuously, not at decommission time only.

### 2026-09-20 — Subdomain takeover on a .mil hostname pointing at a lapsed registrable domain (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #2499178](https://hackerone.com/reports/2499178)
- Type: Security misconfiguration (dangling CNAME into a domain that had expired and was purchasable)
- Summary: A `.mil` subdomain resolved to a domain that had lapsed and was available for registration, so anyone able to buy that domain could serve content from the government hostname. The reporter deliberately did not register it and reported the condition instead, noting the TLD's residency requirement.
- Technique / pattern: Dangling records do not only point at cloud resources — plenty point at ordinary third-party domains (an old agency, vendor or campaign site) that have since expired. Add a registrability check to the takeover triage: for every CNAME target that fails to resolve, query the registrar's availability and WHOIS expiry rather than stopping at "NXDOMAIN, probably a cloud resource". Report the finding from the DNS evidence; buying the domain is neither required to prove it nor always permitted by the program.
- Takeaway: Proof of exploitability does not require exploitation. A resolution chain ending at an available domain is a complete report, and it keeps the researcher clear of registration, cost and legal complications.

### 2026-09-20 — RCE via npm misconfiguration: internal package names resolved from the public registry (PayPal) — $30,000
- Source: [HackerOne #925585](https://hackerone.com/reports/925585)
- Type: Security misconfiguration (dependency confusion / namespace resolution in the package manager)
- Summary: Certain PayPal development projects defaulted to the public npm registry instead of the intended internal one. The researcher published packages on the public registry bearing the names of the missing internal packages and confirmed they were fetched and installed by PayPal's environment — which would have executed attacker-supplied install scripts inside the build.
- Technique / pattern: The bug lives in *name resolution*, not in any single package. Harvest internal package names from published sources — JS bundles, source maps, `package.json` left in artifacts, CI logs, Docker layers, public repos — then check whether each name is unregistered on the public registry. Claiming an unregistered internal name and observing an installation callback proves the misconfiguration; keep the package inert (a DNS or HTTP beacon with no payload), document it in the report and hand it over or unpublish on request. The same shape applies to PyPI, RubyGems, NuGet and Maven.
- Takeaway: A package manager that falls back to the public registry when an internal name is missing is a code-execution path fed by anyone. Scope internal names to a private scope or registry, pin resolution explicitly, and defensively register the internal names publicly.

### 2026-09-19 — Subdomain takeover of d02-1-ag.productioncontroller.starbucks.com via an unclaimed Azure Cloud Service (Starbucks) — n/a
- Source: [HackerOne #661751](https://hackerone.com/reports/661751)
- Type: Security misconfiguration (dangling CNAME → Azure Cloud Service takeover)
- Summary: DNS reconnaissance showed the subdomain resolving to an Azure Cloud Service resource that no longer existed (NXDOMAIN on the target), letting the reporter claim the name in Azure and serve content from a Starbucks hostname.
- Technique / pattern: Resolve every discovered subdomain with `dig` and triage by the *shape* of the failure: NXDOMAIN on a CNAME target means the cloud resource is gone while the DNS record survives. Azure's `cloudapp.net` / `azurewebsites.net` / `trafficmanager.net` names are claimable in any free subscription, so the exploit is an account signup and a resource name, not a technical bypass. Deep infrastructure-style hostnames (`productioncontroller`, `d02-1-ag`) signal automated provisioning, where teardown ordering is most often wrong.
- Takeaway: Delete the DNS record before, or together with, the cloud resource. Reconcile DNS against live cloud inventory after every decommission.

### 2026-09-19 — Subdomain takeover due to a CNAME pointing at an unclaimed CloudFront distribution (GSA Bounty) — n/a
- Source: [HackerOne #317005](https://hackerone.com/reports/317005)
- Type: Security misconfiguration (dangling CNAME → CDN distribution takeover)
- Summary: A GSA subdomain CNAMEd into CloudFront while the distribution's alternate domain name was never claimed, allowing the hostname to be taken over. AWS has since hardened this path so a CloudFront CNAME can no longer be claimed without proving DNS control.
- Technique / pattern: Fingerprint the provider behind each CNAME from the error body it returns, then check whether the specific hostname is registered at that provider. Just as usefully, this report is a reminder to check *when* a technique was mitigated: providers close claim paths over time (CloudFront alternate-domain verification, Azure domain-ownership checks), so re-test on the current platform before writing up a technique learned from an old disclosure.
- Takeaway: Techniques expire. Verify a dangling-CNAME class is still claimable on today's provider before reporting it, and cite the vendor's current behaviour in the write-up.

### 2026-09-19 — Amazon S3 bucket misconfiguration on zomato-share allows list, upload and delete (Eternal (Zomato)) — n/a
- Source: [HackerOne #229690](https://hackerone.com/reports/229690)
- Type: Security misconfiguration (over-permissive S3 bucket policy — write and delete)
- Summary: The `zomato-share` bucket was configured so that an outside party could list its contents, upload new objects and delete existing ones, demonstrated with ordinary AWS CLI commands.
- Technique / pattern: Test buckets for all three verbs, not just read: `s3 ls`, then `s3 cp` a harmless marker file, then remove your own marker. A writable bucket that serves application assets is content poisoning — stored XSS via an uploaded HTML or SVG, a swapped JavaScript bundle, a replaced download — and delete permission is destructive impact on its own, which raises the severity far above 'public bucket'. Touch only objects you created and say so explicitly in the report.
- Takeaway: Enumerate the full permission set of an exposed bucket. Write access to a bucket the application serves from is a supply-chain foothold, not an information leak.

### 2026-09-19 — Open S3 bucket readable and writable by any authenticated AWS user (Ruby) — n/a
- Source: [HackerOne #209223](https://hackerone.com/reports/209223)
- Type: Security misconfiguration (S3 ACL granting AuthenticatedUsers)
- Summary: The bucket used for Ruby's file uploads was reachable with any AWS account's credentials, exposing its object listing (prefixes such as `aix71_ppc/`, `amazon/`) and remaining incompletely fixed after the first remediation attempt.
- Technique / pattern: An anonymous probe is not a full test: an ACL granting `AuthenticatedUsers` denies unauthenticated requests but allows anyone who has signed up for AWS. Always re-run the check with credentials from an unrelated account. Equally important, re-test after the fix lands — partial remediations that close read but leave write, or close the bucket but leave a CloudFront origin open, are common enough to be worth a scheduled re-check.
- Takeaway: `AuthenticatedUsers` means every AWS customer. Verify bucket policy with real credentials, and verify the fix as carefully as the bug.

### 2026-09-19 — Session cookie leakage via a static header field in WebViewerFragment (LinkedIn) — n/a
- Source: [HackerOne #3475626](https://hackerone.com/reports/3475626)
- Type: Misconfiguration (shared mutable state in a mobile WebView, chained to account takeover)
- Summary: A static `CUSTOM_HEADERS` field on the LinkedIn Android app's `WebViewerFragment` retained cookies across successive URL loads instead of being cleared per request. The researcher bypassed the verification WebView's URL validation with a `javascript://` payload disguised as a LinkedIn URL, forced the app down the vulnerable fragment, loaded a legitimate LinkedIn domain to populate the static field, then loaded an attacker host that received the retained authentication cookies (CVSS 8.1).
- Technique / pattern: In mobile clients, look for `static`/singleton state that holds per-request security context — header maps, cookie jars, token caches, `WebView` instances reused across destinations. Decompile and grep for `static` collections in WebView wrappers, then check whether anything clears them between `loadUrl()` calls. Pair that with the app's deep-link and URL-validation surface: validators commonly check for a substring or a host prefix and miss `javascript:` and `data:` schemes, and query/fragment tricks (`#`) can neutralize parameters the validator appended. Also enumerate which fragment or activity handles a given URL shape — apps often have a hardened viewer and a legacy one, and steering the URL toward the legacy handler is half the exploit.
- Takeaway: Per-request security context must not live in static fields. Validate WebView URLs by parsed scheme and exact host against an allow-list, and clear headers and cookies on every navigation.

### 2026-09-19 — Internal access to HackerOne Confluence docs via support-system misconfiguration (HackerOne) — $12,500
- Source: [HackerOne #3113398](https://hackerone.com/reports/3113398)
- Type: Misconfiguration (support workflow bridging the external/internal boundary)
- Summary: A misconfiguration in HackerOne's support system let an external party reach internal Confluence documentation, viewing and modifying limited content that was never meant to leave the organization (CVSS 8.2).
- Technique / pattern: Support desks, ticketing portals and help-center integrations sit deliberately astride the trust boundary — they accept input from anyone and are wired into internal wikis, knowledge bases and SSO. Map that plumbing: which internal systems the help center syncs with, whether article/attachment identifiers from the internal wiki are addressable through the public portal, whether agent-side previews or search indexes are reachable with a customer account, and whether the integration's service account is over-permissioned. The bug is rarely in the wiki; it is in the connector that was configured to see everything.
- Takeaway: Audit the *connectors* between customer-facing tools and internal knowledge systems, and scope their service accounts to exactly the content meant to be public. Treat a help-center integration as an authenticated path into the intranet.

### 2026-09-19 — Double clickjacking on the WakaTime OAuth authorization flow (WakaTime) — n/a
- Source: [HackerOne #3287060](https://hackerone.com/reports/3287060)
- Type: Misconfiguration (violation of secure design — UI redressing against OAuth consent)
- Summary: An attacker page could open a new tab and simultaneously redirect the original tab to `wakatime.com/oauth/authorize`, aligning a decoy button over the consent button. The victim's first click closed the attacker's tab and the second landed on the real authorization button, granting the attacker's OAuth application access to the account and an authorization code exchangeable for a token.
- Technique / pattern: "Double clickjacking" defeats the standard framing defences because the victim's browser is never framing anything — the target is loaded top-level in its own tab, so `X-Frame-Options` and CSP `frame-ancestors` are satisfied and irrelevant. The attack instead exploits the timing between two clicks and a window swap. When assessing a consent, authorization, delete or payment screen, do not stop at "framing is blocked": check whether the confirmation button is actionable immediately on load, without any prior user gesture on that page. The proposed fix in the report is the general one — keep the control inert until real mouse movement or keyboard input is observed on the page itself.
- Takeaway: Frame-busting headers do not cover window-swap UI redressing. High-consequence confirmations need a same-page user-gesture requirement (or a keystroke/second distinct interaction) before the button becomes live.

### 2026-09-19 — Subdomain takeover of addons-preview-cdn.mozilla.net via an unregistered Fastly domain (Mozilla) — $500
- Source: [HackerOne #2706358](https://hackerone.com/reports/2706358)
- Type: Misconfiguration (dangling CNAME → CDN subdomain takeover)
- Summary: `addons-preview-cdn.mozilla.net` carried a CNAME to `addons.allizom.org`, which resolves into Fastly's infrastructure, but Mozilla had never registered that specific hostname inside Fastly. The researcher claimed it in Fastly and served a proof-of-concept file from Mozilla's subdomain.
- Technique / pattern: The dangling record here points at a *CDN* rather than at a deleted origin, which is the variant testers most often skip: the CNAME resolves, the target service answers, and only the vendor-side hostname registration is missing. The signature is a vendor error page ("unknown domain", "Fastly error: unknown domain") on a host whose DNS looks perfectly healthy. Build the check into recon as: resolve every subdomain, fingerprint the service behind each CNAME, and flag any that returns the provider's unclaimed-hostname response — then follow that provider's documented custom-domain flow, which is usually free and requires no exploit.
- Takeaway: A resolving CNAME is not proof the name is claimed at the provider. Reconcile DNS records against live CDN/PaaS configurations as an inventory task, and remove the record whenever the service-side hostname is retired.

### 2026-09-18 — Default credentials (admin/admin) on an internet-facing Kinetic Core System Console (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1938693](https://hackerone.com/reports/1938693)
- Type: Security misconfiguration (default credentials on a management console)
- Summary: The Kinetic Core System Console was reachable from the internet and still accepted the factory credential pair admin/admin, granting administrative access to the platform with no exploitation required.
- Technique / pattern: Fingerprint the product behind each exposed admin console from its login page, favicon, title or asset paths, then consult the vendor's own installation documentation for the shipped default account. This is recon plus reading documentation rather than brute force, and it stays within scope because only the single documented pair is tried.
- Takeaway: Every administrative console on a public address needs its default account changed or removed before exposure; inventory of what is listening is the real control.

### 2026-09-18 — Publicly reachable Spring Boot Admin instance with default credentials (8x8) — n/a
- Source: [HackerOne #954818](https://hackerone.com/reports/954818)
- Type: Security misconfiguration (exposed framework admin UI, default credentials)
- Summary: A Spring Boot Admin instance was publicly accessible with its default credentials still in place, giving unauthenticated outsiders administrative access to the monitoring interface for the applications it managed.
- Technique / pattern: Sweep subdomains and non-standard ports for framework management UIs - Spring Boot Admin, Actuator, Eureka, Consul, Kibana, Jenkins - which are usually deployed on the assumption they are internal. Once reached, a Spring Boot Admin console also fronts the registered applications' Actuator data, so the misconfiguration compounds into environment and secret disclosure.
- Takeaway: Monitoring and management planes are production attack surface. They need network restriction and real credentials, not just the default a quickstart guide left behind.

### 2026-09-18 — Authentication bypass on an admin console using default credentials (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1839012](https://hackerone.com/reports/1839012)
- Type: Security misconfiguration (default administrative credentials)
- Summary: An administrative portal accepted a standard administrative username with its associated default password, granting access to the portal and the data held in it, including stored emails and links.
- Technique / pattern: Treat the login page as recon output: identify the software, then try only the vendor-documented default pair and confirm impact by describing what the session exposes rather than by exercising the data. Documenting the reachable data is what moves such a finding from 'weak password' to a rated access-control failure.
- Takeaway: Default credentials are an authentication bypass, not a hardening suggestion; rate them by the data the resulting session can reach.

### 2026-09-18 — Default admin username and password on an exposed Remedy Single Sign-On server (MTN Group) — n/a
- Source: [HackerOne #1397786](https://hackerone.com/reports/1397786)
- Type: Security misconfiguration (default credentials on an SSO/identity component)
- Summary: A Remedy Single Sign-On server was internet-exposed with its factory administrative credentials unchanged, allowing anyone to authenticate to the administrative interface of the identity system itself.
- Technique / pattern: Prioritize identity infrastructure when sweeping for default credentials - SSO servers, IdPs, directory consoles and password vaults. A default account on the component that issues authentication for everything else is a far larger finding than the same flaw on a leaf application, so map each exposed host to its role before triaging.
- Takeaway: Compromise of the authentication provider is compromise of everything behind it. Identity components belong at the top of any default-credential and exposure audit.

### 2026-09-18 — CORS misconfiguration on the main domain exposes user data via the WordPress REST API (Sifchain) — n/a
- Source: [HackerOne #1188684](https://hackerone.com/reports/1188684)
- Type: Security misconfiguration (reflected CORS origin)
- Summary: The /wp-json/ REST endpoint reflected any supplied Origin header back in Access-Control-Allow-Origin, letting script on an arbitrary site read the API's responses, including registered user IDs, names and login slugs.
- Technique / pattern: Send a request with an obviously foreign Origin header and compare the response headers: an Access-Control-Allow-Origin that mirrors whatever was sent indicates a dynamic reflection rather than an allow-list. Then check whether Access-Control-Allow-Credentials is also true - that pairing is what turns readable public data into readable authenticated data.
- Takeaway: Reflecting the Origin header is equivalent to allowing every origin, and on a CMS it hands out the user enumeration list the platform tries to hide.

### 2026-09-18 — Publicly listable S3 bucket exposing JSON and configuration files (AWS VDP) — n/a
- Source: [HackerOne #3382796](https://hackerone.com/reports/3382796)
- Type: Security misconfiguration (public S3 bucket listing)
- Summary: An S3 bucket allowed unauthenticated ListBucket, returning a ListBucketResult XML index of internal JSON and configuration objects together with their names and modification timestamps.
- Technique / pattern: A single unauthenticated GET to the bucket endpoint distinguishes a locked bucket from a listable one. Report the listing itself rather than harvesting object contents: filenames, paths and timestamps already establish impact by revealing internal endpoints, feature flags and deployment cadence, and stopping there keeps the test within scope.
- Takeaway: Object ACLs and bucket listing are separate permissions - a bucket whose objects are private still leaks architecture if the index is public. Enumerating the listing is enough to prove the finding.

### 2026-09-18 — Subdomain takeover of a host under mozgcp.net (Mozilla) — n/a
- Source: [HackerOne #2143408](https://hackerone.com/reports/2143408)
- Type: Security misconfiguration (dangling DNS / subdomain takeover)
- Summary: A DNS record still pointed at a decommissioned cloud resource under mozgcp.net, so the researcher could claim the backing resource and serve their own content from a Mozilla-controlled hostname.
- Technique / pattern: Enumerate subdomains, resolve each one, and flag records whose CNAME or A target belongs to a cloud provider but returns a provider-specific 'no such resource' page. The claimability check - can the target name be registered in that provider now? - is what separates a real takeover from a cosmetic dangling record; claim it, serve a benign proof file, and stop.
- Takeaway: Subdomain inventory decays with every decommissioned service. DNS records must be retired in the same change as the resource they point to.

### 2026-09-18 — CORS origin validation failure via prefix matching (UPchieve) — n/a
- Source: [HackerOne #1404986](https://hackerone.com/reports/1404986)
- Type: Security misconfiguration (CORS allow-list bypass by prefix match)
- Summary: The server validated the Origin header by prefix rather than exact match while returning Access-Control-Allow-Credentials: true, so an attacker origin such as https://target.example.com.evil.com was accepted and could read authenticated responses.
- Technique / pattern: When an allow-list is present, probe its matching logic instead of concluding it is safe: append the trusted domain as a prefix of an attacker domain (trusted.com.evil.com), embed it as a suffix (eviltrusted.com), and try null and scheme variations. Reflection of any of these alongside allow-credentials proves the check is substring-based.
- Takeaway: CORS origin checks must compare the full origin exactly. Substring matching converts an allow-list into an allow-all for anyone who can register a domain.

### 2026-09-18 — CORS misconfiguration on admin.myndr.net reflecting any subdomain origin with credentials (Myndr) — n/a
- Source: [HackerOne #3930957](https://hackerone.com/reports/3930957)
- Type: Security misconfiguration (CORS: wildcard-subdomain reflection with credentials)
- Summary: The admin panel reflected any `*.myndr.net` origin in `Access-Control-Allow-Origin` while also sending `Access-Control-Allow-Credentials: true`, so script on any subdomain could read authenticated admin responses — including CSRF nonces — and drive admin functionality.
- Technique / pattern: Send an `Origin` of a subdomain you do not control and check whether it is echoed alongside credentialed ACAO; if so, the question becomes whether *any* subdomain can be obtained (takeover, user content, staging host), since trust is delegated to all of them at once.
- Takeaway: Trusting a whole subdomain wildcard with credentials collapses your origin boundary into your weakest subdomain, and a readable CSRF token defeats the one control that was still protecting state-changing admin actions.

### 2026-09-18 — CORS policy on client.amplifi.com and protect.ubnt.com allowed requests from outside the trusted domains (Ubiquiti Inc.) — n/a
- Source: [HackerOne #430249](https://hackerone.com/reports/430249)
- Type: Security misconfiguration (CORS origin-validation flaw)
- Summary: A mistake in the CORS policy meant `client.amplifi.com` and `protect.ubnt.com` accepted cross-origin requests from sites outside the intended `*.ubnt.com` and `*.ui.com` domains, letting an attacker-controlled page read data from or act as a logged-in user.
- Technique / pattern: Test origin validation with near-miss values rather than only a plain foreign origin: prefixes, suffixes, embedded domain strings and unusual schemes reveal that the check is a substring match instead of a parsed-origin comparison.
- Takeaway: Most CORS failures are matching failures, not policy failures — validate the parsed origin against an explicit allowlist, since `endsWith`/`contains` checks are trivially satisfied by a domain the attacker registers.

### 2026-09-18 — Misconfigured AWS S3 bucket exposing internal documents (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1062803](https://hackerone.com/reports/1062803)
- Type: Security misconfiguration (public S3 bucket)
- Summary: An S3 bucket holding production, admin and internal documents was configured to permit unauthenticated listing and retrieval, so anyone reaching the bucket URL could download its contents.
- Technique / pattern: Derive candidate bucket names from the organisation's naming conventions and from hostnames already serving assets, then check anonymous list/read; confirm ownership from the content itself before reporting, since bucket names are easy to guess but attribution is what makes the report actionable.
- Takeaway: Object storage defaults to private but is routinely opened during a migration and never closed again; treat bucket policy as part of the perimeter and audit it the way you audit an exposed host.

### 2026-09-18 — Open S3 bucket accessible by any AWS user (GoCD) — n/a
- Source: [HackerOne #1654145](https://hackerone.com/reports/1654145)
- Type: Security misconfiguration (S3 ACL granting AuthenticatedUsers)
- Summary: A GoCD-associated S3 bucket was reachable without restriction, letting unauthorised parties access its contents — the classic result of an ACL that grants access to any authenticated AWS principal rather than to the account's own roles.
- Technique / pattern: Check both anonymous access and access with an arbitrary AWS account's credentials: `AuthenticatedUsers` grants look closed to an unauthenticated probe but are effectively public, since anyone can create an AWS account.
- Takeaway: 'Authenticated' in an S3 ACL means authenticated to AWS, not to you — a bucket that denies anonymous reads can still be world-readable, so test with credentials before concluding it is locked down.

### 2026-09-18 — DNS misconfiguration — subdomain takeover via a deleted S3 bucket (8x8) — n/a
- Source: [HackerOne #1089502](https://hackerone.com/reports/1089502)
- Type: Security misconfiguration (dangling DNS record / subdomain takeover)
- Summary: An Amazon S3 bucket was deleted while the DNS record pointing at its endpoint was left in place, leaving a dangling reference that an attacker could claim by re-creating the bucket under the same name.
- Technique / pattern: Resolve every subdomain and flag records pointing at cloud endpoints that return a 'no such bucket/app' error — it is deleting the resource, not the DNS entry, that opens the window, so re-scan after every decommission.
- Takeaway: Decommissioning is a security event: remove the DNS record before or together with the resource, and keep an inventory that reconciles DNS records against live cloud resources.

### 2026-09-18 — Subdomain takeover via an unclaimed Netlify target (8x8) — n/a
- Source: [HackerOne #1697402](https://hackerone.com/reports/1697402)
- Type: Security misconfiguration (dangling CNAME to a SaaS host / subdomain takeover)
- Summary: A subdomain pointed at a misconfigured, unclaimed Netlify target, allowing takeover through the hosting provider's own custom-domain claiming flow.
- Technique / pattern: Fingerprint the third-party service behind each CNAME from its error page, then follow that provider's documented custom-domain process — takeover is usually just claiming the domain in a free account, with no exploit involved.
- Takeaway: Every SaaS/PaaS accepting custom domains (Netlify, Heroku, GitHub Pages, Shopify and friends) is a takeover vector; keep a per-provider checklist and re-scan whenever a site is retired.

### 2026-09-18 — Default administrator username and credentials on an exposed application (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1195325](https://hackerone.com/reports/1195325)
- Type: Security misconfiguration (unchanged vendor default credentials)
- Summary: A server application was deployed with the vendor's default 'Administrator for the default organization' account still active, granting full administrative access straight from the login page.
- Technique / pattern: Fingerprint the product from its login page or banner, look up the vendor's documented default account, and try it once — this is a deployment configuration bug, not brute force, and should be tested within program rules.
- Takeaway: Default credentials remain among the highest-impact, lowest-effort findings; hardening must include changing or disabling every vendor-shipped account before the system is exposed.

### 2026-09-18 — Open directory — directory listing enabled on a web server (RATELIMITED) — n/a
- Source: [HackerOne #461242](https://hackerone.com/reports/461242)
- Type: Security misconfiguration (directory listing / autoindex enabled)
- Summary: A misconfigured web server had directory listings enabled, letting anyone browse directory contents and discover files that were never meant to be enumerable.
- Technique / pattern: Strip filenames from known URLs to request the parent paths and watch for auto-generated index pages; combine with a path wordlist to map how much of the tree the listing exposes.
- Takeaway: Disable autoindex by default, restrict unnecessary paths, and place an index file in every directory — a listing turns an unknown location into a free map of the server's contents.

### 2026-09-17 — Exposed Kubernetes API - RCE/Exposed Creds (Snapchat) — $25,000
- Source: [HackerOne #455645](https://hackerone.com/reports/455645)
- Type: Security misconfiguration (exposed orchestration control plane)
- Summary: A Kubernetes API server was reachable from the internet without adequate authentication/authorization, allowing cluster interaction that led to code execution and access to credentials stored in the cluster.
- Technique / pattern: Identify a control-plane API on a public host, confirm with harmless unauthenticated reads (version, namespace/pod listing), then demonstrate impact using ordinary cluster primitives — secret reads and pod exec are part of the same API surface.
- Takeaway: Control-plane APIs must never be publicly reachable; 'read-only' exposure is rarely read-only, because secrets and exec live behind the same endpoint.

### 2026-09-17 — Spring Actuator endpoints available leading to account takeover (LY Corporation) — $5,000
- Source: [HackerOne #862589](https://hackerone.com/reports/862589)
- Type: Security misconfiguration (exposed management/diagnostic endpoints)
- Summary: Spring Boot Actuator endpoints were reachable without authentication, exposing runtime internals that contained material sufficient to take over user accounts.
- Technique / pattern: Fuzz for the standard management paths (/actuator, /actuator/env, /heapdump, /trace, /httptrace), download the exposed diagnostics, and mine them for session tokens, credentials and configuration secrets.
- Takeaway: Framework diagnostics are a credential store in disguise — always probe for management/debug routes, and bind actuator exposure to an internal port with authentication.

### 2026-09-17 — Docker Registry HTTP API v2 exposed without authentication (Semmle) — n/a
- Source: [HackerOne #347296](https://hackerone.com/reports/347296)
- Type: Security misconfiguration (unauthenticated internal service exposure)
- Summary: A Docker registry was reachable on the public internet with no authentication, letting anyone enumerate repositories and pull images containing proprietary code and any secrets baked into their layers.
- Technique / pattern: Probe for the registry version endpoint (/v2/), then walk the catalog and tag-list APIs to enumerate images, and inspect manifests/layers for embedded credentials and build configuration.
- Takeaway: Internal infrastructure services default to open — inventory what actually listens on public addresses, and treat container images as secret-bearing artifacts, not just binaries.

### 2026-09-17 — Subdomain takeover on firefox.com subdomain (Mozilla) — $500
- Source: [HackerOne #2899858](https://hackerone.com/reports/2899858)
- Type: Security misconfiguration (dangling DNS / subdomain takeover)
- Summary: A firefox.com subdomain retained a DNS record pointing at a third-party service whose resource had been released, so an outsider could claim the name and serve content from a highly trusted domain.
- Technique / pattern: Enumerate subdomains, resolve CNAME targets, and look for provider 'no such app / unclaimed' fingerprints, then verify claimability on the provider before reporting.
- Takeaway: DNS records outlive the services they point to — decommissioning must remove the record first, and takeover checks belong in continuous monitoring rather than one-off scans.

### 2026-09-17 — CORS Misconfiguration (Publitas) — n/a
- Source: [HackerOne #2332728](https://hackerone.com/reports/2332728)
- Type: Security Misconfiguration (CORS)
- Summary: An endpoint reflected the request Origin into Access-Control-Allow-Origin together with Access-Control-Allow-Credentials: true, allowing an attacker-controlled origin to make credentialed cross-origin reads of the victim's data.
- Technique / pattern: Sent a request with a spoofed Origin header, saw it reflected in ACAO with credentials allowed, and proved impact with a fetch(..., {credentials:'include'}) PoC from an external origin.
- Takeaway: Never reflect arbitrary Origins with credentials enabled — use a strict allowlist and never return ACAC:true for dynamically reflected origins.

### 2026-09-17 — Subdomain takeover of main domain via acquired asset (Acronis) — n/a
- Source: [HackerOne #1256389](https://hackerone.com/reports/1256389)
- Type: Security Misconfiguration (subdomain takeover)
- Summary: An Acronis-owned subdomain inherited from an acquisition (www.cyberlynx.lu) had a dangling CNAME chain pointing to unclaimed Wix DNS infrastructure, enabling takeover.
- Technique / pattern: DNS reconnaissance traced the CNAME chain to an unclaimed third-party host (Wix); claiming the service under that name would let the researcher serve content on the subdomain.
- Takeaway: Assets gained through mergers/acquisitions are a rich takeover surface — inventory acquired domains and remove dangling CNAMEs that point to unclaimed services.

### 2026-09-17 — CORS misconfiguration on content-paywall API leaks authenticated data (Semrush) — n/a
- Source: [hackerone #769058](https://hackerone.com/reports/769058)
- Type: Security Misconfiguration (CORS)
- Summary: A Semrush API endpoint (`/content-paywall/api/accesslevel`) reflected the request `Origin` and allowed credentials, so a malicious site could read the authenticated victim's response cross-origin.
- Technique / pattern: Send a cross-origin request with a crafted `Origin` header, confirm `Access-Control-Allow-Origin` reflects it with `Access-Control-Allow-Credentials: true`, then read the victim's data from an attacker page via `fetch(..., {credentials:'include'})`.
- Takeaway: Reflecting arbitrary Origins together with credentials is unsafe — allow-list exact trusted origins and never combine a wildcard/reflected origin with `Allow-Credentials`.

### 2026-09-17 — Subdomain takeover via dangling DNS to an unclaimed service (Greenhouse.io) — n/a
- Source: [hackerone #407355](https://hackerone.com/reports/407355)
- Type: Security Misconfiguration (subdomain takeover / dangling DNS)
- Summary: A Greenhouse subdomain had a dangling CNAME pointing to a third-party service whose backing resource had been removed, letting an attacker claim that resource and serve content from the subdomain.
- Technique / pattern: Enumerate subdomains, find a CNAME pointing to an unclaimed provider resource (GitHub Pages/Heroku/etc.), then register that resource to take control of the subdomain.
- Takeaway: Remove DNS records when decommissioning hosted resources; dangling CNAMEs to deleted services allow subdomain takeover and are caught by routine CNAME/DNS auditing.

### 2026-09-16 — CORS misconfiguration leaking sensitive data (U.S. Dept Of Defense) — n/a
- Source: [hackerone #733017](https://hackerone.com/reports/733017)
- Type: Security Misconfiguration (CORS)
- Summary: An endpoint reflected the request `Origin` into `Access-Control-Allow-Origin` while also sending `Access-Control-Allow-Credentials: true`, letting an attacker-controlled site read authenticated responses cross-origin.
- Technique / pattern: Send a crafted/injected `Origin` header and check whether it is reflected back with credentials allowed; a reflected ACAO plus credentials means any origin can read protected data (prove with a `fetch(..., {credentials:'include'})` PoC).
- Takeaway: Never reflect arbitrary origins with credentials enabled — allow-list exact trusted origins and avoid `Allow-Credentials: true` with dynamic origins.

### 2026-09-16 — Insecure S3 bucket exposing .git config and an access token (Mozilla) — n/a
- Source: [hackerone #2383486](https://hackerone.com/reports/2383486)
- Type: Security Misconfiguration (public cloud storage / exposed .git)
- Summary: A public S3 bucket for Mozilla's Infographics project exposed a `.git` directory — including an old (already-invalid) GitHub access token — to anyone.
- Technique / pattern: Enumerate an org's buckets/hosts and probe readable paths like `/.git/config`; a readable `.git` folder can leak source, history, and hard-coded secrets.
- Takeaway: Lock down cloud storage ACLs, never deploy `.git` to public hosts, and rotate any credential that ever touched version control.

### 2026-09-15 — Default credentials lead to Spring Boot Admin dashboard access (8x8) — n/a
- Source: [HackerOne #1417635](https://hackerone.com/reports/1417635)
- Type: Security misconfiguration (default credentials / exposed admin console)
- Summary: An internet-exposed Spring Boot Admin instance was protected only by default credentials, allowing anyone to log in to the admin dashboard.
- Technique / pattern: Locate exposed management/admin panels (Spring Boot Admin/Actuator) and try vendor default credentials; framework admin consoles are frequently deployed without changing defaults.
- Takeaway: Never expose management dashboards with default or shared credentials; restrict them to internal networks and enforce unique, strong authentication.

### 2026-09-15 — Method Enumeration Possible on domain (Weblate) — n/a
- Source: [HackerOne #230648](https://hackerone.com/reports/230648)
- Type: Security misconfiguration (HTTP method enumeration)
- Summary: A Weblate endpoint answered an OPTIONS request by disclosing the HTTP methods it allowed (GET, HEAD, OPTIONS), a server-config weakness (OWASP OTG-CONFIG-006).
- Technique / pattern: Send an OPTIONS request (or read the Allow response header) to enumerate enabled verbs; unnecessary methods widen attack surface and reveal server behavior.
- Takeaway: Restrict allowed HTTP methods to only those required (here GET/HEAD) and avoid advertising verbs, to reduce reconnaissance value.

### 2026-09-15 — CORS misconfiguration on a DoD web asset (U.S. Dept of Defense) — n/a
- Source: [HackerOne #1530581](https://hackerone.com/reports/1530581)
- Type: Security Misconfiguration (CORS)
- Summary: A Department of Defense web application reflected the request `Origin` into `Access-Control-Allow-Origin` with a permissive policy, enabling cross-origin reads of responses from an attacker-controlled site.
- Technique / pattern: Send `Origin: https://evil.example` and check whether it is reflected in `Access-Control-Allow-Origin`; if credentials are allowed, prove impact with a `fetch(..., {credentials:'include'})` PoC.
- Takeaway: Never reflect arbitrary Origins; use a strict allowlist and avoid `Access-Control-Allow-Credentials: true` alongside dynamically reflected origins.

### 2026-09-15 — Subdomain takeover via dangling DNS (MetaMask) — n/a
- Source: [HackerOne #1717626](https://hackerone.com/reports/1717626)
- Type: Security Misconfiguration (subdomain takeover)
- Summary: A MetaMask subdomain pointed (dangling CNAME) at a third-party service that was no longer claimed, letting a researcher register the resource and serve content from the trusted subdomain.
- Technique / pattern: Enumerate subdomains and resolve their CNAMEs; find one aliased to an unclaimed SaaS/cloud endpoint, then claim that resource on the provider to control the subdomain.
- Takeaway: Retire DNS records when decommissioning services; dangling CNAMEs to unclaimed providers let an attacker host content on a trusted origin.

### 2026-09-14 — Exposed .git/config leaking repository contents (curl) — n/a
- Source: [HackerOne #3612891](https://hackerone.com/reports/3612891)
- Type: Security misconfiguration — exposed VCS metadata (.git)
- Summary: A publicly reachable .git/config on the target server exposed repository metadata, from which an attacker can reconstruct source code and hunt for embedded secrets.
- Technique / pattern: During recon, probe for VCS/artifact leftovers (/.git/config, /.git/HEAD, .env, backups); a readable .git directory can be pulled and rebuilt with tools like git-dumper to recover the full tree.
- Takeaway: Never deploy the .git directory to web-served paths; block dotfiles/VCS folders at the web server and scan your own hosts for them.

### 2026-09-14 — Default credentials (admin/admin) on an internet-facing server (Trellix) — n/a
- Source: [HackerOne #2160178](https://hackerone.com/reports/2160178)
- Type: Security misconfiguration — default credentials
- Summary: An internet-facing Trellix host accepted default login credentials (admin/admin), granting unauthorized administrative access.
- Technique / pattern: During asset recon, enumerate exposed admin/login panels and test vendor default credential pairs (admin/admin, admin/password); default creds remain one of the fastest paths to admin on forgotten hosts.
- Takeaway: Force credential changes on first use, remove or disable default accounts, and inventory internet-facing services so no host ships with vendor defaults.


### 2026-09-13 — 2FA bypass via "Sign in with Google" (Shopify) — n/a
- Source: [HackerOne #178293](https://hackerone.com/reports/178293)
- Type: Security misconfiguration (2FA not enforced on SSO path)
- Summary: A user with Google Authenticator 2FA enabled who also activated Google Apps login could sign in through "Sign in with Google" and skip the second factor entirely.
- Technique / pattern: Enumerate every authentication path (password, SSO, API, mobile) and check that each independently enforces 2FA; alternate login providers frequently bypass factors bolted onto the primary flow.
- Takeaway: Enforce MFA as a property of the account/session applied after any authentication method completes — not only on the username/password path.

### 2026-09-13 — Critical server misconfiguration exposes user records (Flickr) — n/a
- Source: [HackerOne #1365738](https://hackerone.com/reports/1365738)
- Type: Security misconfiguration (exposed backend surface)
- Summary: A misconfigured server surface allowed unauthorized retrieval of sensitive user data, including hashed passwords and other database records, risking mass account compromise.
- Technique / pattern: Probe infrastructure inherited from acquisitions/legacy hosting for exposed admin or data endpoints and permissive access; old, distributed backends often lack the main app's controls.
- Takeaway: Inventory and lock down legacy/acquired infrastructure; sensitive datastores must never be reachable without authentication.

### 2026-09-12 — CORS misconfiguration could disclose sensitive information (UPchieve) — n/a
- Source: [HackerOne #1199527](https://hackerone.com/reports/1199527)
- Type: Security misconfiguration (CORS)
- Summary: The API allowed arbitrary origins together with credentials, letting an attacker-controlled site make authenticated cross-origin requests and read the responses.
- Technique / pattern: Sent a rogue `Origin` header, observed it reflected into `Access-Control-Allow-Origin` alongside `Access-Control-Allow-Credentials: true`, then read the cross-origin response from a PoC page using `fetch(..., {credentials:'include'})`.
- Takeaway: Never reflect the Origin header into ACAO while allowing credentials — maintain a strict allowlist of trusted origins.

### 2026-09-12 — Permissive CORS enables unauthenticated WordPress user enumeration (U.S. Dept of Defense) — n/a
- Source: [HackerOne #1092125](https://hackerone.com/reports/1092125)
- Type: Security misconfiguration (CORS)
- Summary: A permissive CORS policy on a WordPress site let any third-party domain perform two-way interaction with the WP REST API, exposing registered users' id, name and login name without authentication.
- Technique / pattern: Combined the overly permissive CORS configuration with the default WordPress `/wp-json/wp/v2/users` endpoint to enumerate account details cross-origin.
- Takeaway: Lock down CORS and restrict/disable the WordPress users REST endpoint; default framework endpoints plus loose CORS leak recon-quality data.

### 2026-09-11 — CORS misconfiguration (substring origin check) -> Account Takeover (X / xAI, niche.co) — n/a
- Source: [HackerOne #426147](https://hackerone.com/reports/426147)
- Type: Security misconfiguration (CORS) — substring/contains origin check -> ATO
- Summary: The app trusted any Origin merely containing "//niche.co", so an attacker origin like https://niche.co.evil.net was reflected and honored; with credentials and multiple methods allowed, this enabled credentialed cross-origin reads leading to account takeover.
- Technique / pattern: Probe a naive `contains("//niche.co")` check with look-alikes that satisfy it (`niche.co.evil.net`); if ACAO reflects with `Access-Control-Allow-Credentials: true`, exfiltrate tokens/authenticated data via `fetch(...,{credentials:'include'})`.
- Takeaway: Never validate Origin with substring/contains logic — match an exact allowlist; permissive CORS + credentials can escalate straight to ATO.

### 2026-09-11 — CORS misconfiguration with credentials (Sifchain) — n/a
- Source: [HackerOne #1194280](https://hackerone.com/reports/1194280)
- Type: Security misconfiguration (CORS) — reflected origin + credentials
- Summary: The site reflected an arbitrary request Origin into Access-Control-Allow-Origin while also sending Access-Control-Allow-Credentials: true, letting a malicious page make authenticated cross-origin requests and read the responses.
- Technique / pattern: Send a request with a foreign `Origin`; if it is echoed into ACAO together with `Allow-Credentials: true`, host a PoC page that issues a credentialed `fetch` and reads the logged-in victim's data.
- Takeaway: `Access-Control-Allow-Origin` must never be the reflected request origin when credentials are allowed — pair an exact allowlist with careful use of the credentials flag.

### 2026-09-11 — CORS Misconfiguration on www.zomato.com (Zomato) — n/a
- Source: [HackerOne #168574](https://hackerone.com/reports/168574)
- Type: Security misconfiguration (CORS) — suffix/endsWith origin bypass
- Summary: A CORS policy meant to allow only *.zomato.com matched the Origin by suffix, so any domain ending in "zomato.com" (e.g. an attacker-registered notzomato.com) was reflected and trusted, enabling credentialed cross-origin reads.
- Technique / pattern: Send Origin headers with a look-alike domain ending in the allowed suffix; if it is reflected into Access-Control-Allow-Origin with credentials allowed, read authenticated responses via fetch(..., {credentials:'include'}).
- Takeaway: Validate Origin against an exact allowlist, not a substring/suffix/regex — endsWith('zomato.com') also matches notzomato.com.

### 2026-09-11 — Subdomain Takeover at creatorforum.roblox.com (Roblox) — n/a
- Source: [HackerOne #264494](https://hackerone.com/reports/264494)
- Type: Security misconfiguration — subdomain takeover (dangling CNAME → Discourse)
- Summary: creatorforum.roblox.com pointed (dangling DNS) to a non-existent Discourse instance, letting an attacker claim that service and serve content under a roblox.com subdomain.
- Technique / pattern: Enumerate subdomains and resolve their CNAME targets, find one pointing to an unclaimed third-party service (Discourse), then register/claim that service to take over the subdomain.
- Takeaway: Remove DNS records when retiring third-party services; dangling CNAMEs to claimable SaaS are takeover-ready and abuse the parent domain's trust.

### 2026-09-10 — Exposed .git folder leads to source disclosure (U.S. Dept of Defense) — n/a
- Source: [HackerOne #1624157](https://hackerone.com/reports/1624157)
- Type: Security misconfiguration (exposed VCS directory)
- Summary: A web root served a publicly accessible `.git/` directory, letting anyone reconstruct the repository — source code, commit history and any secrets committed to it.
- Technique / pattern: Request `/.git/config` or `/.git/HEAD`; if reachable, dump the tree with GitTools `gitdumper` + `extractor` (or git-dumper) and review history for credentials/keys.
- Takeaway: Never deploy the `.git` directory to a web root; block `.git`/`.svn`/backup paths at the server and scan external assets for them.

### 2026-09-10 — Subdomain takeover at info.hacker.one (HackerOne) — $500
- Source: [HackerOne #220002](https://hackerone.com/reports/220002)
- Type: Security misconfiguration (subdomain takeover)
- Summary: A dangling DNS record for `info.hacker.one` pointed at an unclaimed third-party service, letting the researcher claim it and serve content from the subdomain — a bypass of an earlier fix.
- Technique / pattern: Enumerate subdomains, find CNAMEs pointing at services with no active resource (unclaimed SaaS fingerprint, `NoSuchBucket`), then register/claim that resource to take over the host.
- Takeaway: Remove DNS records when decommissioning services; re-test after "fixes" — takeover often recurs on the same dangling record.

### 2026-09-09 — CORS misconfiguration → sensitive info disclosure (Zomato)
- Source: [HackerOne #426165](https://hackerone.com/reports/426165)
- Type: CORS misconfiguration
- Summary: The app **reflected the request `Origin`** into `Access-Control-Allow-Origin` while allowing credentials, letting a malicious origin read authenticated responses cross-origin → sensitive data disclosure.
- Technique / pattern: Send `Origin: https://evil.com`; if the response echoes it in `ACAO` and sets `Access-Control-Allow-Credentials: true`, host a PoC page that `fetch(..., {credentials:'include'})` and exfiltrates the body.
- Takeaway: Never reflect arbitrary origins with credentials; use a strict allow-list. Also test `null` origin and sub-domain/suffix bypasses.

### 2026-09-09 — CORS misconfiguration (Nord Security)
- Source: [HackerOne #758785](https://hackerone.com/reports/758785)
- Type: CORS misconfiguration
- Summary: A permissive CORS policy allowed a controlled origin to make credentialed cross-origin reads of API responses.
- Technique / pattern: Same origin-reflection test as above; verify with a credentialed `fetch` PoC and confirm the sensitive body is readable cross-origin.
- Takeaway: CORS findings need a working credentialed PoC to prove impact; a reflected `ACAO` alone isn't exploitable without `ACAC: true` (or a pre-auth sensitive endpoint).
