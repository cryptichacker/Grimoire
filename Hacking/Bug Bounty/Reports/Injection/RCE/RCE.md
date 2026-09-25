---
tags: [hacking, bug-bounty, injection, rce, reports]
type: log
---

# RCE

## Up
- [[Injection]]

Disclosed **Remote Code Execution** reports reached via injection chains — **OS command injection**, **insecure deserialization**, **malicious file upload**, and SSTI-that-lands-as-pure-RCE. See [[Pattern]] → Injection (RCE).

## Reports

### 2026-09-25 — node --run POSIX positional argument escaping allows shell command injection (Node.js) — n/a
- Source: [HackerOne #3817602](https://hackerone.com/reports/3817602)
- Type: OS command injection via faulty shell argument escaping
- Summary: `node --run <script> -- <args>` appends positional arguments to the package script after escaping each one for the shell, but on POSIX the escaping did not correctly handle single quotes, so an argument containing `'` could close the intended quoted argument and inject further shell syntax into the command line that runs.
- Technique / pattern: Wherever a tool builds a shell string from user input and relies on hand-rolled quoting, the test is a single `'` (and then `'; id; '`) in each interpolated value. The bug class is the escaper's model of the shell diverging from the shell's real parsing rules.
- Takeaway: Do not hand-roll shell escaping; pass argument vectors directly with `execFile`-style APIs instead of composing a command string. When a wrapper must quote, a single-quote character in the input is the canonical probe — and this is the same root cause as JSON-encoded data interpolated into a `shell=True` template.

### 2026-09-24 — CVE-2025-24813: remote code execution and sensitive file access via partial PUT (Internet Bug Bounty — Apache Tomcat) — n/a
- Source: [HackerOne #3031518](https://hackerone.com/reports/3031518)
- Type: Path-handling flaw in partial PUT — arbitrary file write/read escalated to code execution
- Summary: Tomcat's partial PUT implementation derived its temporary file name from the user-supplied path with the separator replaced by `.`, leaving the caller in control of where content landed. With writes enabled on the default servlet and partial PUT supported, an attacker could read security-sensitive files and inject content into them.
- Technique / pattern: Look at how a server derives temporary or staging file names from request-controlled input — "sanitisation" that rewrites separators instead of rejecting the path preserves attacker control over the final name. Then chain the write primitive into an existing deserialization or session-persistence path to reach execution.
- Takeaway: A file-write primitive is only as harmless as the directories it can reach; derive server-side temporary names from a server-generated identifier, never from the client's path.

### 2026-09-24 — Deserialization in Automation Builder via Jint to Newtonsoft serializer coercion (8x8) — $3,000
- Source: [HackerOne #3861550](https://hackerone.com/reports/3861550)
- Type: Insecure deserialization — `TypeNameHandling` reached through a scripting-engine bridge (Critical)
- Summary: `connect.8x8.com`'s Automation Builder exposed Newtonsoft JSON objects directly to the embedded Jint JavaScript engine during server-side template evaluation of an HTTP request step. An authenticated user could abuse Jint's overload resolution to instantiate a `JsonSerializer` and have a crafted JSON response deserialized with `TypeNameHandling` in effect, giving arbitrary type instantiation and code execution on the automation backend.
- Technique / pattern: Low-code and automation builders embed a scripting sandbox (Jint, Jurassic, Nashorn, V8) and then hand it host objects for convenience. The escape is rarely a sandbox bug — it is the exposed object graph. Enumerate which host types the script context can see, look for anything able to construct a serializer, loader, file helper or process wrapper, then use the engine's own overload-resolution rules to reach a constructor the designers never intended. In .NET, any reachable Newtonsoft serializer with `TypeNameHandling` enabled is an arbitrary-type-instantiation primitive.
- Takeaway: A scripting sandbox is only as tight as the objects handed into it. On workflow and automation builders, enumerate the host object graph before hunting for engine bugs — and treat a reachable `JsonSerializer` with `TypeNameHandling` as RCE, not as a configuration nit.

### 2026-09-23 — Unauthenticated RCE in Bitbucket Data Center via Hazelcast deserialization (CVE-2022-26133) (Atlassian Bitbucket (Bugcrowd)) — n/a (P1, 40 points)
- Source: [Bugcrowd ce3d9a93](https://bugcrowd.com/disclosures/ce3d9a93-2168-4785-90b7-47f66a5b8162/rce-in-bitbucket-datacenter-via-hazelcastport)
- Type: Insecure Java deserialization → unauthenticated RCE
- Summary: Bitbucket Data Center's clustering layer exposed a Hazelcast port (default `5701`) whose custom authenticator deserialized attacker-supplied data during the cluster-join handshake, so a remote unauthenticated attacker could achieve code execution (CVE-2022-26133).
- Technique / pattern: The class of issue is untrusted deserialization on an exposed network service: input arriving on the cluster port reached a `readObject` path before authentication. Recognising it starts from noticing a non-HTTP clustering/cache port reachable from outside and confirming the framing it accepts is a serialized-object protocol.
- Takeaway: Clustering, cache and message-bus ports (Hazelcast, RMI and similar) are an under-tested attack surface that is frequently reachable without authentication. Treat any pre-auth `readObject` on network input as critical, and firewall internal cluster ports off the public network.

### 2026-09-23 — RCE on Confluence Data Center via OGNL Injection (Atlassian) — n/a (P1, 40 points)
- Source: [Bugcrowd #f76873aa](https://bugcrowd.com/disclosures/f76873aa-7acc-4f39-b94d-f066317e7c41/rce-on-confluence-data-center-via-ognl-injection)
- Type: Expression-language (OGNL) injection leading to RCE — CVE-2021-26084
- Summary: Confluence Server/Data Center rendered Webwork tags through Velocity such that a value containing `$` was evaluated twice — first by Velocity, then again by Webwork as an OGNL expression — giving unauthenticated remote code execution. Disclosed 2021-10-04.
- Technique / pattern: Supply an OGNL payload in a request parameter such as `sourceTemplateId` that the framework maps onto an action setter, use unicode escapes to slip past the `SafeExpressionUtil` sandbox, and reach `@java.lang.Runtime@getRuntime().exec()`; reachable via `/pages/doenterpagevariables.action`, `/signup.action` and `/users/darkfeatures.action`.
- Takeaway: Double evaluation is the core bug class behind most EL/OGNL RCEs — look for any place a template engine renders a value that a second engine then interprets, and expect expression sandboxes to fall to encoding tricks.

### 2026-09-22 — Command injection in Harmony trajectory-subsetter gives RCE as root (NASA VDP) — n/a (P1, CVSS 8.8)
- Source: [Bugcrowd #31ad3b26](https://bugcrowd.com/disclosures/31ad3b26-beae-4a91-879d-c6640cfe41dc/command-injection-in-harmony-trajectory-subsetter-subset-shape-geojson-gives-any-earthdata-user-remote-code-execution-rce-as-root-on-harmony-earthdata-nasa-gov)
- Type: OS command injection via user-supplied GeoJSON (injection chain → RCE)
- Summary: The subsetter service serialised the user's `subset.shape` GeoJSON with `json.dumps` and interpolated it into a shell string passed to `subprocess.Popen(command, shell=True)`; because `json.dumps` does not escape single quotes, a quote inside any GeoJSON property closed the argument and the rest ran as shell. Any holder of a free Earthdata Login account reached code execution as `uid=0` in the production container.
- Technique / pattern: Trace user-controlled structured data (JSON, GeoJSON, filenames, metadata) into command construction; serialisation is not shell escaping, so look for `shell=True` with formatted strings and test with a bare `'` before any full payload. Fixed upstream in v1.0.11 by passing an argument array with `shell=False`.
- Takeaway: Only an argument vector (`shell=False`, `execve`-style) is safe — quoting a serialised blob into a shell command re-opens injection even when the blob is "valid JSON"; and a free self-service account is the same as unauthenticated for exposure purposes.
### 2026-09-22 — RCE via unsafe raw template rendering in fastify view with EJS (Fastify) — n/a
- Source: [HackerOne #3122019](https://hackerone.com/reports/3122019)
- Type: Template injection to RCE (CWE-94) in `@fastify/view`
- Summary: Passing user-controlled content to `reply.view({ raw: ... })` with the EJS engine sent it straight into EJS `compile()`, so attacker-controlled template syntax executed as server-side Node code.
- Technique / pattern: Identified that the `raw` option compiles its argument as a template rather than treating it as data, then supplied EJS expression syntax that reaches Node's standard library during rendering.
- Takeaway: Any API that accepts a template string (not just variables) is a code-execution sink; audit framework options like `raw` / `renderString` for user-reachable input.

### 2026-09-21 — Unauthenticated RCE via an AI agent's `execute_custom_code` tool (NASA VDP)
- Source: [Bugcrowd #85b61013](https://bugcrowd.com/disclosures/85b61013-5265-4295-ab9f-33ed27754132/unauthenticated-rce-on-hydrology-czdt-smce-nasa-gov-via-the-czdt-flood-agent-s-execute_custom_code-tool)
- Type: RCE (exposed code-execution tool / LLM agent)
- Summary: A public hydrology 'Copilot' agent exposed an `execute_custom_code` tool that ran attacker-supplied Python on the backend with no authentication.
- Technique / pattern: Send unauthenticated requests to the agent endpoint that route into its `execute_custom_code` tool; confirm execution non-destructively with nonce digests and hostname/working-directory checks rather than damaging payloads.
- Takeaway: AI/agent frameworks that wire a code-execution or shell tool into a user-facing endpoint create direct RCE — treat every agent tool as an authenticated, sandboxed, least-privilege capability.


### 2026-09-21 — RCE on beta-partners.tesla.com via SSRS ViewState deserialization (CVE-2020-0618) (Tesla) — 40 pts
- Source: [Bugcrowd disclosure d23e05b1](https://bugcrowd.com/disclosures/d23e05b1-c4cc-440a-a678-d8045468c902/rce-on-https-beta-partners-tesla-com-due-to-cve-2020-0618)
- Type: RCE (insecure deserialization, P1)
- Summary: A SQL Server Reporting Services instance at `https://beta-partners.tesla.com/ReportServer/Pages/ReportViewer.aspx` was vulnerable to CVE-2020-0618, an insecure .NET deserialization flaw allowing arbitrary code execution.
- Technique / pattern: Generate a malicious serialized .NET gadget with `ysoserial.net` and POST it in the `NavigationCorrector$ViewState` parameter to the ReportViewer endpoint; SSRS deserializes it and runs the payload (e.g. PowerShell).
- Takeaway: Fingerprint COTS report/BI servers (SSRS, Cognos) and match the version to known deserialization CVEs — a hidden ViewState/state field is an unauthenticated RCE sink when the patch is missing.

### 2026-09-21 — RCE via insecure deserialization in Telerik UI (U.S. Dept of Defense) — n/a
- Source: [HackerOne #838196](https://hackerone.com/reports/838196)
- Type: RCE via injection chain (arbitrary file upload + insecure deserialization)
- Summary: Telerik Web UI v2016.2.607.40 was exploited by chaining CVE-2017-11317 (weak-key arbitrary file upload) with CVE-2019-18935 (insecure deserialization) for full RCE, demonstrated with a benign 10-second sleep. Disclosed 2020-05-07.
- Technique / pattern: Fingerprint the third-party component and version, then chain a known file-upload primitive to place a malicious mixed-mode assembly with a deserialization gadget that loads and executes it - a known-CVE chain against an outdated dependency.
- Takeaway: Attackers weaponize outdated components - inventory and patch dependencies, and treat any deserialization of attacker-influenced data plus a writable upload path as a critical chain even when each flaw alone looks limited.

### 2026-09-21 — Blind User-Agent SQL injection escalated to blind OS command execution (Sony) — n/a
- Source: [HackerOne #1339430](https://hackerone.com/reports/1339430)
- Type: RCE via SQLi chain (MSSQL xp_cmdshell, OOB DNS)
- Summary: A blind time-based SQLi in the User-Agent header of a Sony login form (MSSQL backend) was escalated to OS command execution by enabling and invoking xp_cmdshell, with output exfiltrated over DNS. Rated CVSS 9.1; disclosed 2022-07-06.
- Technique / pattern: Time-based blind SQLi through a non-obvious point (an HTTP header, not a visible field), then sp_configure to turn on xp_cmdshell and blind command execution with out-of-band DNS exfiltration.
- Takeaway: Injection points hide in headers (User-Agent, Referer, X-Forwarded-For); parameterize everything, run the DB least-privilege, and disable dangerous procedures like xp_cmdshell so a SQLi cannot pivot to full OS RCE.

### 2026-09-21 — Mozilla VPN Clients: RCE via file write and path traversal (Mozilla) — $6,000
- Source: [HackerOne #2995025](https://hackerone.com/reports/2995025)
- Type: RCE - arbitrary file write via path traversal (Windows client)
- Summary: In the inspector live_reload command (developer mode + staging servers), InspectorHotreloader::fetchAndAnnounce() downloaded remote files and built the local path from the unsanitized filename, so ..\ sequences wrote files outside the temp folder on Windows, leading to RCE after visiting a malicious site.
- Technique / pattern: Reviewed the client source for download-and-save paths, found filename concatenation via QString arg() without normalization, and used Windows backslash traversal (not stripped by the URL fileName logic) to plant an executable-reachable file.
- Takeaway: Any 'download then write' feature must canonicalize and confine the destination path; platform-specific separators (\ on Windows) routinely slip past filename sanitization written for '/'.

### 2026-09-21 — Pickle deserialization vulnerability in XComs (Internet Bug Bounty (Apache Airflow)) — n/a
- Source: [HackerOne #2334460](https://hackerone.com/reports/2334460)
- Type: RCE - insecure deserialization (pickle) bypassing a safety flag (CVE-2023-50943)
- Summary: Before Airflow 2.8.1, XCom data could be poisoned so that pickled data was deserialized even with enable_xcom_pickling=False, letting a malicious DAG author run code in other tasks/workers or when UI/API users viewed XCom entries.
- Technique / pattern: Identified an alternate code path that still called pickle deserialization despite the config flag, and used the shared XCom store as the bridge from a low-privilege writer to higher-privilege readers.
- Takeaway: A 'pickling disabled' setting is only as strong as every deserialization path - hunt for legacy/fallback loaders, and treat shared data buses (queues, caches, XCom) as cross-tenant RCE routes.

### 2026-09-20 — RCE in GitLab Workhorse when stripping image metadata with ExifTool (GitLab) — $20,000
- Source: [HackerOne #1154542](https://hackerone.com/reports/1154542)
- Type: RCE (file upload → external binary; type confusion between extension and content)
- Summary: GitLab Workhorse routed uploads whose extension matched `jpg|jpeg|tiff` to ExifTool in order to remove non-whitelisted metadata tags. ExifTool ignores the file extension and determines the format from the file's own content, so a file presented as an image but recognised as a different format was handed to a different parser module and reached a code-execution path on the server (CVE-2021-22205).
- Technique / pattern: The bug class is a *disagreement about file type between the router and the handler*. Map the pipeline an upload traverses — which component decides the type, by what evidence (extension, declared MIME, magic bytes, a library's own sniffing), and which binary eventually parses it — then craft a file that satisfies the gate's evidence and the parser's evidence differently: an image extension over another format's magic bytes, a polyglot, or a format the downstream tool supports but the gate never contemplated. Confirm reachability with an out-of-band DNS or HTTP callback before attempting anything further.
- Takeaway: Whitelisting by extension controls nothing about what the downstream tool decides to parse. Pin the format explicitly when invoking the tool, sandbox media processing, and treat every media-handling dependency as an RCE surface to patch promptly.

### 2026-09-19 — Remote code execution via Java object deserialization in an exposed Oracle PeopleSoft service (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #329376](https://hackerone.com/reports/329376)
- Type: RCE (insecure deserialization, CWE-502, in COTS enterprise software)
- Summary: A DoD web system running Oracle PeopleSoft exposed a `monitor` service that accepted serialized Java objects. Sending crafted objects of particular types allowed arbitrary code execution, and the same path could be used for denial of service.
- Technique / pattern: Commercial off-the-shelf enterprise suites — PeopleSoft, WebLogic, SharePoint, JBoss, Sitecore — ship diagnostic, monitoring and integration servlets that deserialize by design and are frequently left reachable when the product is published to the internet. Fingerprint the product and version from telltale paths, cookies and asset bundles, map that to published CVEs and vendor advisories, then prove reachability with an out-of-band DNS or HTTP callback rather than a shell.
- Takeaway: Patch-gap hunting on COTS software outperforms novel bug hunting on these targets. The first question about any enterprise product on a public address is which of its management endpoints are still listening.

### 2026-09-19 — Unauthenticated RCE in Taskcluster web-server via a GraphQL filter argument (Mozilla) — $12,000
- Source: [HackerOne #3782701](https://hackerone.com/reports/3782701)
- Type: RCE (code injection via a query-library `$where` operator reaching `new Function()`)
- Summary: Taskcluster's `/graphql` endpoint accepted a `filter` argument that was handed to the `sift` library (17.1.3), which compiles `$where` strings into functions with `new Function()`. With `CSP_ENABLED` unset and anonymous requests permitted, an unauthenticated attacker executed JavaScript in the Node process, ran shell commands via `process.getBuiltinModule("child_process").execSync()`, and read `process.env` for Postgres credentials, Taskcluster tokens, OAuth secrets and encryption keys.
- Technique / pattern: Query and filter libraries that mimic MongoDB's operator syntax (`sift`, `mingo`, `json-rules-engine`-style evaluators, and Mongo itself) implement `$where`, `$function` or `$expr` by compiling a *string* into code. Wherever an API exposes a structured filter object to the client, try the engine's scripting operator with a harmless arithmetic probe (`6*7`) and a type check (`typeof process`) before anything else. Read the library version's source to confirm whether the string path is gated by a flag, and check the anonymous role's scopes — a resolver that merely runs is enough, no data access required.
- Takeaway: Never pass client-supplied filter objects to a query library without stripping scripting operators. Treat "filter", "query", "where" and "sort" arguments as code sinks, and confirm the dangerous path is disabled by configuration rather than assuming the default is safe.

### 2026-09-19 — Authenticated Elasticsearch Painless script execution via Query.search.sort_query (HackerOne) — n/a
- Source: [HackerOne #3694007](https://hackerone.com/reports/3694007)
- Type: RCE (script injection into a search backend via a pass-through JSON parameter)
- Summary: The `sort_query` parameter of HackerOne's GraphQL `Query.search` accepted arbitrary Elasticsearch JSON with no validation, so an authenticated user could inject Painless scripts that the backing cluster compiled and executed per document (CVSS 8.8).
- Technique / pattern: A careful, low-blast-radius methodology worth copying. (1) Prove the parameter is parsed, by sending a valid structured sort and observing the ordering change. (2) Prove *compilation*, by alternating deliberately malformed and well-formed scripts and correlating HTTP 500 with compile errors and 200 with success — an error-code oracle establishes a compiler is present without executing anything meaningful. (3) Prove *per-document execution*, by comparing a script returning a constant against one reading internal metadata (`_seq_no`) and showing the resulting orderings diverge. (4) Keep every script confined to the researcher's own documents and to engine metadata. Generalize: any parameter forwarded verbatim to a search engine, rules engine or reporting backend inherits that engine's scripting surface.
- Takeaway: Build search queries server-side from an allow-list of sortable fields; never forward client JSON into the engine. When testing a scripting sink on a live target, use compile-error oracles and metadata reads to demonstrate execution without touching other tenants' data.

### 2026-09-19 — OS command injection in aws-cdk-lib NodejsFunction via the unsanitized OsCommand helper (AWS VDP) — n/a
- Source: [HackerOne #3637898](https://hackerone.com/reports/3637898)
- Type: RCE (OS command injection in build tooling; supply-chain trigger, CWE-78)
- Summary: When a developer listed dependencies in `nodeModules`, AWS CDK read version strings from `package.json` and passed them to `OsCommand.writeJson()`, which built `echo '${data}'` without escaping embedded single quotes. A malicious npm package could set a crafted version string that broke out of the quoting and executed commands inside the Docker bundling container, which bind-mounts the host project directory — so `cdk synth` or `cdk deploy` with a compromised dependency ran attacker code near AWS credentials and source (CVSS 8.6).
- Technique / pattern: Audit the **build and packaging toolchain** as an injection target, not just the running application. Grep bundlers, CI helpers and IaC synthesizers for shell strings assembled with `echo '...'`, `sh -c`, template literals and naive single-quote wrapping. Then ask which of the interpolated values come from *package metadata* rather than from the developer: versions, names, scripts, `files` globs and repository URLs are all attacker-controlled the moment one dependency is malicious or typosquatted. Single-quote wrapping is defeated by a single `'`, so that is the whole payload.
- Takeaway: Build commands as argument arrays (`execFile`/`spawn` with a list), never as shell strings. Data read out of a dependency's manifest is untrusted input, and developer machines and CI runners are the highest-value place for it to execute.

### 2026-09-19 — Apache Solr RCE via Velocity template on an exposed instance (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #822002](https://hackerone.com/reports/822002)
- Type: RCE (template injection against an unauthenticated service — SSTI that is pure RCE)
- Summary: Port scanning found an internet-reachable Apache Solr instance with no authentication holding defense-related data. The researcher abused Solr's Velocity response writer to evaluate template expressions through crafted GET requests, executing `id` and `cat /etc/passwd` as the Solr service account.
- Technique / pattern: The exploit primitive here is a *documented feature* of the product, reachable because the service was exposed and unauthenticated — the same shape as Flink's jar/plan endpoints or an open Jupyter kernel. Method: enumerate non-HTTP-80 services during recon, fingerprint each product and version from its admin UI and response headers, then read the vendor documentation for any feature that evaluates expressions, scripts or templates (Solr's VelocityResponseWriter and `params.resource.loader.enabled`, Elasticsearch scripting, Jenkins script console, Groovy consoles). Confirm with a benign expression before any command.
- Takeaway: Search and data platforms bundle template/scripting engines that assume a trusted network. Rate an exposed instance by the features it ships enabled, not by whether a CVE exists — and keep these services off public addresses with authentication on regardless.

### 2026-09-18 — Remote code execution via insecure deserialization in Telerik UI (CVE-2019-18935 chained with CVE-2017-11317) (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1174185](https://hackerone.com/reports/1174185)
- Type: RCE (insecure deserialization + arbitrary file upload, known CVE chain)
- Summary: An exposed Telerik.Web.UI.WebResource.axd endpoint was affected by two known flaws: a weak/known encryption key permitting arbitrary file upload (CVE-2017-11317) and a JavaScriptSerializer deserialization sink (CVE-2019-18935). Chained, they gave remote code execution on the underlying host.
- Technique / pattern: Fingerprint third-party components from their telltale handler paths (.axd, .ashx, vendor JS bundles, version strings) and match the identified version against published CVEs before hunting for novel bugs. Where a component has both an upload primitive and a deserialization sink, the chain - place a payload on disk, then trigger deserialization that loads it - is the documented path.
- Takeaway: Patch-gap hunting on internet-exposed enterprise components is high yield: the bug is public, and the work is inventory and version fingerprinting rather than exploit development.

### 2026-09-18 — Insecure deserialization leads to RCE on Sitecore (CVE-2025-27218) (Mars) — n/a
- Source: [HackerOne #3090123](https://hackerone.com/reports/3090123)
- Type: RCE via insecure deserialization (.NET BinaryFormatter)
- Summary: A Sitecore instance deserialised attacker-controlled data taken from the ThumbnailsAccessToken header using BinaryFormatter, so a crafted serialized object executed operating-system commands on the server and allowed file read and exfiltration.
- Technique / pattern: Treat custom headers as deserialization sinks, not just bodies and cookies. Fingerprint the platform, look up which formatter that product version uses, then generate a gadget chain for exactly that formatter and confirm execution out-of-band (a DNS or HTTP callback) before attempting anything further - callbacks prove code execution without touching data.
- Takeaway: BinaryFormatter on any attacker-reachable input is remote code execution by design; an out-of-band callback is the safe, sufficient proof.

### 2026-09-18 — Remote code execution via .NET __VSTATE deserialization (8x8) — n/a
- Source: [HackerOne #1391576](https://hackerone.com/reports/1391576)
- Type: RCE via insecure deserialization (.NET LosFormatter / __VSTATE)
- Summary: A third-party community platform integrated into an 8x8 subdomain deserialised the __VSTATE form field with LosFormatter, so a gadget-chain payload submitted through an ordinary web form ran arbitrary code on the server.
- Technique / pattern: Inventory the hidden form fields a page posts back: __VSTATE, __VIEWSTATE and similar ASP.NET state fields carry serialized server objects, and are exploitable whenever MAC validation is absent or the key is known. Build the chain for the specific formatter in use, apply the expected encoding layers (compress, then base64), and validate with an out-of-band DNS callback.
- Takeaway: Acquired and vendor-supplied applications on a subdomain inherit the parent's trust but not its security review - enumerate and fingerprint them separately.

### 2026-09-18 — Unsafe deserialization escalating a SQL injection to remote command execution (Liberapay) — n/a
- Source: [HackerOne #361341](https://hackerone.com/reports/361341)
- Type: Insecure deserialization (escalation of an injection primitive)
- Summary: Liberapay deserialized data that an attacker could influence only by way of a SQL injection; on its own the deserialization was not reachable, but combined with an injection primitive it upgraded database access into remote command execution.
- Technique / pattern: Treat deserialization sinks as impact multipliers: after finding a write primitive (SQLi, cache write, file write), trace which stored values are later deserialized, and report the chain rather than dismissing either half as unexploitable alone.
- Takeaway: Two findings that are each 'not exploitable in isolation' can compose into RCE — the severity question is what an attacker who already holds one primitive can reach, not what an anonymous user can reach.

### 2026-09-18 — Remote code execution through deserialization in the ownBackup marketplace app (ownCloud) — n/a
- Source: [HackerOne #562335](https://hackerone.com/reports/562335)
- Type: Insecure deserialization (third-party app / plugin)
- Summary: The ownBackup application distributed through the ownCloud marketplace contained a deserialization flaw that allowed code execution on the server hosting it, affecting instances where an administrator had installed the app.
- Technique / pattern: Extend the review past the core product to its plugin/marketplace ecosystem: download the published apps, grep for `unserialize`/`pickle`/`readObject` on request-derived data, and check what the platform does to sandbox third-party code (usually nothing).
- Takeaway: An app store attached to a self-hosted platform inherits the platform's trust but rarely its review: a single vulnerable plugin gives the same server-side execution as a flaw in the core, so plugin code belongs in scope.

### 2026-09-18 — Remote code execution in Slack desktop apps via in-app redirect and HTML injection (Slack) — n/a
- Source: [HackerOne #783877](https://hackerone.com/reports/783877)
- Type: RCE (HTML/JavaScript injection inside a web-rendering desktop client, chained with an in-app redirect)
- Summary: Any in-app redirect, open redirect or HTML/JavaScript injection within the Slack desktop client could be chained with a control bypass and a crafted payload to execute arbitrary code on the user's machine, confirmed on desktop versions 4.2 and 4.3.2.
- Technique / pattern: In a desktop application that renders web content, hunt for any injection or redirect primitive reachable inside the trusted app origin, then chain it toward the renderer's privileged bridge — the web bug is the entry point, the client's privileges are the impact.
- Takeaway: Severity depends on where the web content runs: open redirects and HTML injection that would rate 'low' on a website become code execution inside a desktop client, so always test the desktop build separately from the web app.

### 2026-09-17 — Apache Flink RCE via GET jar/plan API Endpoint (Aiven Ltd) — $6,000
- Source: [HackerOne #1418891](https://hackerone.com/reports/1418891)
- Type: RCE via exposed data-platform management API
- Summary: An internet-reachable Apache Flink REST API allowed jar/plan operations without authentication, so its intended job-submission functionality could be used to execute attacker-supplied code on the cluster.
- Technique / pattern: Fingerprint exposed big-data / job-scheduler management interfaces, read the vendor's own REST documentation, and use the documented jar upload / plan / submit endpoints as the execution primitive rather than hunting a memory-safety bug.
- Takeaway: Data-platform components (Flink, Spark, Airflow, Jupyter, Hadoop) ship powerful APIs that are unauthenticated by default — finding one exposed is usually equivalent to finding RCE.

### 2026-09-17 — OS command injection via OpenSSH ProxyCommand/ProxyJump hostname handling (Internet Bug Bounty / OpenSSH) — n/a
- Source: [hackerone #2293731](https://hackerone.com/reports/2293731)
- Type: OS command injection (RCE) — CVE-2023-51385
- Summary: OpenSSH's ssh client expanded a hostname containing shell metacharacters via the `%h` token into the `ProxyCommand`, so a malicious hostname (e.g. supplied by an untrusted config, git submodule URL, or automation) executed arbitrary commands on the client.
- Technique / pattern: Provide a crafted "hostname" carrying shell metacharacters that flows into `%h`/token expansion of a command template, achieving command execution when the connection is initiated — a template/token-expansion sink rather than an obvious `system()` call.
- Takeaway: Data substituted into a command template (`%`-token / string interpolation into a shell) is a command-injection sink; validate/escape seemingly-benign identifiers like hostnames before they reach a shell.

### 2026-09-15 — Java deserialization RCE via JBoss on card.starbucks.in (Starbucks) — n/a
- Source: [HackerOne #221294](https://hackerone.com/reports/221294)
- Type: RCE (insecure Java deserialization)
- Summary: An exposed JBoss service (JBossMQ / invoker) on card.starbucks.in deserialized attacker-controlled Java objects without validation, yielding remote code execution on the server.
- Technique / pattern: Fingerprint exposed enterprise middleware (JBoss JMX/HTTP invoker endpoints), then send a malicious serialized object (ysoserial-style gadget chain) to a `readObject` sink to execute commands.
- Takeaway: Don't expose app-server management/messaging endpoints to the internet; deserialization of untrusted data on legacy middleware is a reliable RCE surface — patch, restrict, and remove default invokers.

### 2026-09-13 — OS command injection in 'rdoc' via crafted filename (Ruby) — n/a
- Source: [HackerOne #1161691](https://hackerone.com/reports/1161691)
- Type: OS command injection
- Summary: RDoc processed a list of files and passed names to Ruby's Kernel#open; a filename beginning with a pipe (|) caused the supposed "file" to be executed as a shell command (CVE-2021-31799).
- Technique / pattern: Ruby open(name) treats a leading "|" as "run this command", so when attacker-controlled filenames reach Kernel#open / IO.read a crafted name like "|touch pwned" yields code execution — a classic path-vs-command sink mismatch.
- Takeaway: Never pass untrusted paths to Kernel#open; use File.open / File.read (which do not honor the pipe) and validate/normalize filenames.

### 2026-09-11 — Unauthenticated RCE via SharePoint CVE-2019-0604 (U.S. Dept of Defense) — n/a
- Source: [HackerOne #534630](https://hackerone.com/reports/534630)
- Type: Insecure deserialization -> RCE (known CVE, unauthenticated)
- Summary: A DoD-owned Microsoft SharePoint server was vulnerable to CVE-2019-0604, an unsafe deserialization of a crafted/encoded parameter that yields unauthenticated remote code / command execution on the server.
- Technique / pattern: Fingerprint the tech stack, match exposed products/versions to known deserialization CVEs, then send the specially crafted parameter to the vulnerable SharePoint endpoint to execute code — patch-gap hunting on internet-facing enterprise software.
- Takeaway: Unpatched enterprise apps (SharePoint, etc.) are high-value RCE targets; enumerate versions and check for known deserialization CVEs before assuming a custom bug.

### 2026-09-11 — Remote code execution on rubygems.org via unsafe deserialization (RubyGems) — n/a
- Source: [HackerOne #274990](https://hackerone.com/reports/274990)
- Type: Insecure deserialization → RCE
- Summary: An unsafe object-deserialization flaw in RubyGems could be escalated to remote code execution on rubygems.org.
- Technique / pattern: Feed crafted serialized (marshalled) data into a sink that deserializes untrusted input; gadget chains present in the deserialized object graph are leveraged to reach code execution.
- Takeaway: Never deserialize untrusted input with native/unsafe deserializers — insecure deserialization is a direct path to RCE, especially on package/registry infrastructure.

_No entries yet — the daily task files OS command injection / deserialization / upload→RCE reports here._
