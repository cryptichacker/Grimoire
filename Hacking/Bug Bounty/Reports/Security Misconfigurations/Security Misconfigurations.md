---
tags: [hacking, bug-bounty, misconfiguration, cors, reports]
type: log
---

# Security Misconfigurations

## Up
- [[Reports]]

Disclosed **security misconfiguration** reports — permissive CORS, default/exposed configs, missing security controls, verbose errors. See [[Pattern]] → Security Misconfigurations.

## Reports

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
