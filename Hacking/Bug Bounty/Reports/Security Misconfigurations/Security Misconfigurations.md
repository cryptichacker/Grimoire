---
tags: [hacking, bug-bounty, misconfiguration, cors, reports]
type: log
---

# Security Misconfigurations

## Up
- [[Reports]]

Disclosed **security misconfiguration** reports — permissive CORS, default/exposed configs, missing security controls, verbose errors. See [[Pattern]] → Security Misconfigurations.

## Reports

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
