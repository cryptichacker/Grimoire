---
tags: [hacking, bug-bounty, idor, reports]
type: log
---

# IDOR

## Up
- [[Reports]]

Disclosed **Insecure Direct Object Reference** reports. Core idea: an object identifier (id, uuid, filename, account number) in a request is honored **without checking the session owns that object**. See [[Pattern]] → IDOR.

## Reports

### 2026-10-26 - IDOR in Push Server management exposes any user's FTP/sFTP credentials (U.S. Dept Of Defense) - n/a
- Source: [HackerOne #228383](https://hackerone.com/reports/228383)
- Type: IDOR / missing object ownership check
- Summary: A DoD application's Push Server feature did not verify that the referenced server belonged to the requesting account, so an authenticated user could read the FTP and sFTP credentials stored against any other user's server.
- Technique / pattern: Register a resource of your own (here a push/transfer server), capture the request that reads its configuration, then substitute or increment the server identifier. Credential-bearing configuration objects - FTP targets, SMTP relays, webhook secrets - are high-value because the config-read endpoint often returns the stored secret in cleartext.
- Takeaway: Any object that stores credentials on a user's behalf must be authorized on read, not only on create. Enumerable connection identifiers plus a config-read endpoint is a direct path from a low-privilege account to third-party credentials.

### 2026-10-26 - IDOR lets an authenticated user delete records owned by other accounts (Mars) - n/a
- Source: [HackerOne #1947376](https://hackerone.com/reports/1947376)
- Type: IDOR / missing authorization on a destructive action
- Summary: On a Mars-operated site the delete action did not check whether the target record belonged to the requesting user, so any authenticated user could delete another user's records. The report notes the authorization mechanism failed to verify the requester was permitted to reach the protected resource.
- Technique / pattern: Create your own record, capture the delete request, then replay it with another account's record identifier. Destructive verbs - a `DELETE` request, or a `POST` to a `.../delete` path - are tested less often than read endpoints and frequently skip the ownership check entirely.
- Takeaway: Test the full CRUD surface of every object and treat delete as its own authorization boundary. An application that correctly hides other users' records from listings may still accept a delete for them.

### 2026-10-26 - IDOR allowing one user to read another user's data (Semrush) - n/a
- Source: [HackerOne #1464168](https://hackerone.com/reports/1464168)
- Type: IDOR / cross-account data read
- Summary: Semrush disclosed an insecure direct object reference that allowed reading data belonging to another user. The published summary states a hotfix shipped quickly and that an investigation found no evidence of exploitation; the public page does not include reproduction steps.
- Technique / pattern: The generic shape is to take an identifier that appears in a request for your own data, substitute another account's value, and diff the responses for records you should not see. SaaS analytics products expose many per-account objects - projects, reports, scheduled exports - and each one is a separate authorization decision.
- Takeaway: Worth logging even when details are withheld: a fast hotfix plus an access-log review of whether anyone else exploited it is the expected vendor handling for a cross-account read.

### 2026-10-23 — IDOR allowing modification of other users' order renewal period (Hostinger) — n/a
- Source: [HackerOne #837349](https://hackerone.com/reports/837349)
- Type: IDOR / missing object ownership check
- Summary: An order identifier in Hostinger's order-management flow was honoured without verifying that the authenticated session owned that order, so a user could alter the renewal period of other customers' orders. The fix added validation that the requesting user owns the order being changed.
- Technique / pattern: Exercise a settings-change action on an object you legitimately own, capture the request, then resubmit it with another account's order identifier. Billing and subscription objects are good targets because the mutation endpoint is usually separate from the read endpoint that does enforce ownership.
- Takeaway: Ownership checks are frequently implemented on the page that *renders* an object and forgotten on the endpoint that *mutates* it. Test read and write paths for the same object separately; never assume one implies the other.

### 2026-10-23 — IDOR via responder_user_id lets an attacker act on users in another organization (Automattic) — n/a
- Source: [HackerOne #725569](https://hackerone.com/reports/725569)
- Type: IDOR / missing authorization on a cross-tenant action
- Summary: A request in Automattic's Happy Tools platform lacked authorization checks, so changing the `responder_user_id` parameter let an attacker approve or decline an "afk" request belonging to users in a different organization.
- Technique / pattern: Create two tenants (organizations) you control, perform the action as a legitimate member of the first, then swap the user-reference parameter in the request body for an identifier belonging to the second tenant. A single parameter such as `responder_user_id` that names *whose* record is acted on is a classic cross-tenant boundary to probe.
- Takeaway: In multi-tenant products the authorization question is two-layered: does the caller have the right role, and does the target object belong to the caller's tenant? A role check alone passes happily while the tenant boundary is crossed.

### 2026-10-23 — Unprivileged staff could change a store's customer-facing email via an undocumented GraphQL mutation (Shopify) — n/a
- Source: [HackerOne #980511](https://hackerone.com/reports/980511)
- Type: Broken access control via an undocumented GraphQL mutation
- Summary: A staff member with no permissions could change the customer-facing sender email address of a store they had no access to. The reporter identified `emailSenderConfigurationUpdate` as an undocumented GraphQL mutation that applied the change without a permission check.
- Technique / pattern: Enumerate the GraphQL schema for mutations that the web UI never calls, then invoke them from a deliberately under-privileged account. Undocumented or internal-only mutations often predate the permission framework that guards the documented ones.
- Takeaway: A GraphQL endpoint's real attack surface is its schema, not its UI. Any mutation reachable from a low-privilege session needs its own authorization check — and the ones nobody documented are the ones nobody reviewed.

### 2026-10-22 — Non-supermarket products accepted into a supermarket-only cart (Skroutz) — n/a
- Source: [Bugcrowd f9f73dcb](https://bugcrowd.com/disclosures/f9f73dcb-2d3a-4074-a1a1-b04e53db135c/exploiting-supermarket-section-for-non-supermarket-products)
- Type: Broken object-level validation / business-logic flaw (P4)
- Summary: Skroutz's cart did not verify that items added through the supermarket flow actually belonged to the supermarket catalogue, so ordinary products could be pushed into supermarket orders. Affected orders had to be cancelled and refunded manually.
- Technique / pattern: Take an identifier the UI only ever populates from one restricted set — here a product reference inside the supermarket add-to-cart request — and submit a reference drawn from a different set. The server trusted the client to send only category-appropriate objects.
- Takeaway: Wherever one flow is a filtered view of a larger object space, the filter usually lives in the front end. The check worth hunting is not only "do I own this object" but "is this object of the kind this endpoint is allowed to accept"; low-severity on paper, it still generates real operational cost.

### 2026-10-22 — Missing role check lets a lower admin demote a global admin (Abnormal Security) — n/a
- Source: [Bugcrowd b2406123](https://bugcrowd.com/disclosures/b2406123-c02d-47cf-bcf1-8af57e1de526/no-rbac-validation-on-api-requests-user-management)
- Type: IDOR / broken function-level authorization (P3)
- Summary: The user-management endpoint `/v1.0/rbac/users_v2/<USER_ID>/` accepted attribute changes, roles and permissions included, without checking that the caller outranked the target. A Portal Tenant Admin could therefore downgrade a Portal Global Admin.
- Technique / pattern: Enumerate the admin API's own role-management calls, then test each direction of the privilege lattice separately with a crafted `PUT`. Escalation upward was blocked but the downward path was not — the reporter probed both rather than assuming one validation covered the whole endpoint.
- Takeaway: RBAC is relational, so test it as a matrix rather than a toggle. "Can I raise my own privileges" and "can I lower someone else's" are different checks in the code and frequently only one exists; destructive downward abuse alone is enough to lock an organisation out of its own tenant.

### 2026-10-22 — Customer records readable by changing the id in a mini-program profile API (Shopify) — n/a
- Source: [HackerOne #975047](https://hackerone.com/reports/975047)
- Type: IDOR (horizontal, authenticated)
- Summary: The Shopify guide mini-program on WeChat loaded the signed-in user's profile from an API path of the form `/api/sp/customer/<id>`. Substituting another customer's identifier returned that customer's information.
- Technique / pattern: Proxy the mini-program client rather than only the web app, open the least interesting screen that loads personal data (the edit-profile view), and inspect what the client sends — the identifier sat in the path, so the test was simply to replace it. The authorization decision was made from the path segment instead of the session.
- Takeaway: Companion surfaces — mini-programs, mobile apps, partner SDKs — often call a different API tier than the audited web front end and inherit none of its access-control review. Intercepting the client the vendor expects nobody to intercept remains one of the cheapest ways to find an IDOR.

### 2026-10-22 — Fare manipulation on another customer's ride via unauthenticated trip endpoints (Bykea) — n/a
- Source: [HackerOne #2861888](https://hackerone.com/reports/2861888)
- Type: IDOR (unauthenticated, horizontal) / business-logic flaw
- Summary: Two Bykea ride endpoints accepted a trip identifier with no authentication, letting a third party raise the fare on a ride belonging to another customer. The inflated bid then surfaced on the driver's side of the trip.
- Technique / pattern: Chain two unauthenticated endpoints that share an object identifier — `GET /v1/config?trip_id=XXX` returned a hash derived from the trip, and that hash was replayed into a forged `PUT /v1/bidding` request. The hash was treated as proof of authorization even though anyone who knew the `trip_id` could obtain one.
- Takeaway: A server-generated token is not an authorization check if an unauthenticated endpoint will hand it to anybody who asks. When a request carries a derived value, test whether that value is obtainable without a session — a signed parameter that an open endpoint will sign on demand is just an object identifier with extra steps.

### 2026-10-21 — Unauthenticated IDOR enabling mass account takeover on a DoD-controlled host (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #685338](https://hackerone.com/reports/685338)
- Type: IDOR (unauthenticated, horizontal)
- Summary: While sweeping a network range the reporter found a virtual host inside a DoD-controlled ASN whose account pages were addressed by a plain numeric identifier. Incrementing that identifier returned other users' accounts with no session check, across a population the reporter estimated at roughly 320,000 accounts.
- Technique / pattern: Asset discovery first (ASN and IP-range sweep for a device/service fingerprint, then virtual-host enumeration), then the classic numeric-identifier walk on the account endpoint — request a record you do not own and see whether the server authorizes on the identifier alone. Unauthenticated reach makes the walk trivially scriptable.
- Takeaway: Scope is defined by ownership, not by domain name: a host on a third-party domain sitting in the target's ASN can still be in scope. And an IDOR on an unauthenticated endpoint scales to the whole table, so impact should be argued from the size of the enumerable id space.

### 2026-10-21 — Unauthenticated profile-management page exposes name, email and EDIPI (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #2858876](https://hackerone.com/reports/2858876)
- Type: IDOR / missing authentication on a management function
- Summary: A profile-management page was reachable without logging in and rendered a user's name, email address and 10-digit EDIPI (the DoD personnel identifier). The same page carried an update function, so the exposure was not read-only.
- Technique / pattern: Request a management or settings page directly rather than through the UI flow, with no session cookie attached. Where the page still renders, check whether its write actions (the update or save handler) are equally unauthenticated — a missing gate on the view usually means a missing gate on the mutation behind it.
- Takeaway: Test the write side of every unauthenticated read you find. A government personnel identifier like an EDIPI raises severity well above "an email address leaked", because it is a stable cross-system key.

### 2026-10-21 — IDOR in a CSV export endpoint exposes other users' quiz results (U.S. General Services Administration) — n/a
- Source: [HackerOne #1118638](https://hackerone.com/reports/1118638)
- Type: IDOR (horizontal, authenticated) via a path-segment identifier
- Summary: The report endpoint `/reports/quizzes-taken-by-user.csv/USERID` carried the target user's identifier as a path segment and returned that user's quiz results without checking the requester was authorized for them. Changing the identifier let any user download another user's results.
- Technique / pattern: Drive the feature normally — open the report view and click Download CSV — then intercept the request in a proxy and vary only the identifier. Export and download handlers are worth testing separately from the screens they sit behind: the UI gates who sees the button, while the file route is often a thin wrapper with its own missing authorization check. Note the identifier lives in the *path*, not a query parameter, so parameter-only fuzzing misses it entirely.
- Takeaway: Every report, export and download route needs the same ownership check as the page that links to it, and the id to tamper with is frequently a path segment or a filename rather than a `?param`.
### 2026-10-21 — IDOR chained with CSRF and XSS on a TikTok SMB subdomain leading to account takeover (TikTok) — n/a
- Source: [HackerOne #1404612](https://hackerone.com/reports/1404612)
- Type: IDOR chained with CSRF and XSS
- Summary: A TikTok SMB subdomain had an IDOR caused by a missing authorization check that let an attacker modify another user's details, alongside a CSRF weakness and an XSS. The reporter chained the issues toward account takeover.
- Technique / pattern: Treat a write-capable IDOR as a chain primitive rather than a standalone finding: if the vulnerable mutation can change an identity-bearing field and a CSRF gap lets it be triggered from an attacker page, the pair upgrades from "edit someone's profile" to takeover. Secondary subdomains for business or SMB tooling are a productive hunting ground because they often reuse the main app's session but not its authorization middleware.
- Takeaway: Report the chain, not the parts. An IDOR that writes plus a missing anti-CSRF token plus script execution in origin is an account-takeover narrative, and that framing is what moves severity.

### 2026-10-20 — User can delete other users' Personal Access Tokens on Mozilla Pontoon (Mozilla) — n/a
- Source: [HackerOne #3325582](https://hackerone.com/reports/3325582)
- Type: IDOR / broken object-level authorization on a destructive route
- Summary: Pontoon's token-deletion route accepted any token id and deleted it without checking the token belonged to the requesting user, so any authenticated user could destroy another user's personal access tokens. Triaged Low, classified `Improper Access Control - Generic`.
- Technique / pattern: Find a destructive, state-changing route that takes a bare numeric id in the path (`/delete-token/<token_id>/`), then replay it from a second account using an id owned by the first. Deletion paths are routinely missed in authorization reviews because testing concentrates on read endpoints.
- Takeaway: Authorization has to be re-asserted per verb, not per resource — a delete handler must resolve the object and confirm `object.owner == current_user` before acting.

### 2026-10-20 — Arbitrary board preference injection via the Deck config API (Nextcloud) — $200
- Source: [HackerOne #3599383](https://hackerone.com/reports/3599383)
- Type: IDOR / broken object-level authorization on a settings endpoint
- Summary: Nextcloud Deck's board configuration API let a user write preference keys against board identifiers they held no rights over, so board settings could be changed on boards owned by other users. Triaged Medium, classified `Improper Access Control - Generic`, $200 bounty.
- Technique / pattern: Preference and config endpoints are a soft spot — they usually take an object id plus a free-form `key`/`value` pair, and the permission check is often only wired into the main CRUD routes. Capture a legitimate preference write, swap in a board id from another account, and see whether the key is accepted.
- Takeaway: Treat settings writes as first-class mutations of the parent object: run the same permission check the parent's CRUD routes use, and allowlist which keys may be written.

### 2026-10-20 — Hidden comments disclosed on the media section of hub.vroid.com (pixiv) — $500
- Source: [HackerOne #2541962](https://hackerone.com/reports/2541962)
- Type: IDOR (disclosure of hidden/moderated objects)
- Summary: Comments hidden from VRoid Hub's media section were still resolvable through their own object lookup, so their content could be read by requesting them directly. Triaged Medium, classified `Insecure Direct Object Reference (IDOR)`, $500 bounty.
- Technique / pattern: When a UI "hides" content — moderation, soft delete, a visibility flag — check whether the filter lives only in the rendering layer. Pull the same record through its own id or an adjacent API route and diff what comes back against what the page shows.
- Takeaway: Visibility state belongs in the data-access layer, not the view: a hidden or soft-deleted record should fail to resolve for an unauthorized caller rather than merely be omitted from a list.

### 2026-10-20 — Valid share tokens allow access to the share owner's temporary upload files (Nextcloud) — n/a
- Source: [HackerOne #3483708](https://hackerone.com/reports/3483708)
- Type: IDOR / capability-token scope too broad
- Summary: Holding a valid share token for one resource was enough to reach temporary upload files belonging to the share owner, extending the token's reach well beyond the item it was issued for. Triaged Medium, classified `Improper Access Control - Generic`.
- Technique / pattern: Treat a capability token (share link, signed URL, upload token) as a scope to probe rather than a key to one object: once you hold a legitimate token, walk sibling paths and staging/temporary locations behind the same handler and record which ones the token still authorizes.
- Takeaway: Bind every capability token to the exact object and operation it was issued for, and put temporary/staging storage behind its own authorization check instead of letting it inherit the parent share's.

### 2026-10-18 — Arbitrary read of another user's private repository without authorization (GitHub) — $10,000
- Source: [HackerOne #3124517](https://hackerone.com/reports/3124517)
- Type: IDOR / improper access control — cross-account private-repository read (High, Resolved)
- Summary: An improper access control flaw in GitHub Enterprise Server allowed a user to read the contents of another user's private repository without holding any authorization on it. Reported through the GitHub Bug Bounty program, resolved, and awarded $10,000.
- Technique / pattern: On a platform whose entire value rests on repository visibility, the authorization check is dense and well-tested on the obvious routes — so findings of this size live on the indirect ones: content APIs addressed by blob, tree or commit object identifier; raw and archive download paths; fork, network and compare views that legitimately span repositories; and caches or mirrors that answer before the permission layer. The method is to enumerate every route that can name a repository or an object inside one, then request each as a principal with no relationship to the target rather than as a downgraded collaborator.
- Takeaway: Object-level authorization must be enforced at the data-access layer that every route shares, not re-implemented per controller, because a single unguarded path defeats the whole visibility model. When auditing a permission-heavy product, inventory routes by the object they can address rather than by the feature they belong to.

### 2026-10-18 — Unauthenticated access to other users' bug-report attachments via a sequential attachment id (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #3259610](https://hackerone.com/reports/3259610)
- Type: IDOR / broken access control — unauthenticated object read on an administrative route (High, Resolved)
- Summary: A DoD application served bug-report attachments from `/BugReport/Admin/Attachment/{id}` without authenticating the requester or checking ownership, so anyone could iterate the identifier and download other users' attachments.
- Technique / pattern: Two things make this shape common. First, attachment and file-serving routes are frequently implemented as thin streaming handlers that were never wired into the application's authorization middleware, because the developer reasoned the identifier was unguessable. Second, a path segment such as `Admin` often reflects which *UI* exposes the route rather than any server-side role check — so always request admin-named paths anonymously rather than assuming they are gated. The probe is to find one legitimate attachment identifier, then replay the request with no cookies or authorization header and walk the identifier space.
- Takeaway: File download and attachment endpoints need the same object-level authorization as the records they belong to, enforced on the server for every request. Sequential identifiers are not a secret, a path name is not a permission, and "only the admin UI links here" is not access control.

### 2026-10-18 — WebAuthn device entry updated by public key instead of by owner (Nextcloud) — n/a
- Source: [HackerOne #3360354](https://hackerone.com/reports/3360354)
- Type: IDOR — record resolved by an attacker-supplied credential identifier (CVE-2025-66558, Low 3.1)
- Summary: Nextcloud's WebAuthn device management located the stored credential record using the submitted public key rather than scoping the lookup to the authenticated owner, so a registered device entry could be updated based on the key value supplied in the request.
- Technique / pattern: A classic "wrong lookup key" IDOR, but on authentication metadata rather than business data. Whenever a handler resolves a row by something the client sends — a public key, a device fingerprint, a token value, an external identifier — instead of by the pair `(owner, id)`, the owner check is structurally absent. Security-settings screens (MFA devices, API tokens, recovery methods, trusted browsers) are high-value, because a write there changes who can log in.
- Takeaway: Load records with the owning principal as part of the query, not as a check afterwards, and never treat a client-supplied credential value as a primary key. Give self-service security-settings routes the same object-level authorization review as the core data model.

### 2026-10-18 — Public Collectives allowed non-members to create pages (Nextcloud) — n/a
- Source: [HackerOne #3533697](https://hackerone.com/reports/3533697)
- Type: IDOR / missing access control on a write endpoint reached through a read-only sharing mode
- Summary: Nextcloud's Collectives app exposed collectives publicly for reading, but the page-creation endpoint did not require membership, so a non-member who could view a public collective could also add pages to it.
- Technique / pattern: "Public" almost always means public *read*, and the bug is the write handler that inherited the same visibility gate. Enumerate the full verb set behind any publicly viewable object — create, rename, move, delete, comment, attach — and send each as an unauthenticated or non-member principal. The open view route is also what makes the object identifiers discoverable in the first place.
- Takeaway: Read visibility and write capability need separate authorization decisions on separate checks. When a feature gains a "public" or "anyone with the link" mode, re-audit every mutating route in that feature rather than assuming the new mode only widened reads.

### 2026-10-16 — Improper authorization lets a project writer edit issue/PR metadata via Memex bulk update (GitHub) — n/a
- Source: [HackerOne #3527788](https://hackerone.com/reports/3527788)
- Type: IDOR / broken object-level authorization (CVE-2026-3306)
- Summary: A user with only **read** access to a repository but **write** access to a GitHub Project could change issue and pull-request metadata through the project's bulk-update path. When an item was added to a project it already belonged to, column value updates were applied without re-checking the actor's repository write permission.
- Technique / pattern: Look for a second surface that writes the same underlying object through a different permission model. Here the project board was the alternate path to repository objects; the "add item that already exists" branch skipped the repo-level authorization that the normal issue edit route enforced. The researcher then showed real impact by setting labels that GitHub Actions workflows gate on, turning a metadata write into a CI control primitive.
- Takeaway: Authorization must be enforced on the **target object**, not on the container the request came through. Audit every alternate write path (bulk endpoints, board/project views, import flows) and the "already exists / upsert" branches, which are where the permission check is most often skipped.

### 2026-10-16 — Any logged-in user can unapprove approved translations in Mozilla Pontoon (Mozilla) — n/a
- Source: [HackerOne #3020021](https://hackerone.com/reports/3020021)
- Type: IDOR / vertical privilege escalation (missing authorization)
- Summary: Pontoon restricted the "unapprove translation" action to privileged users and translation owners in the UI, but the `POST /translations/unapprove/` endpoint itself performed no role check. Any authenticated user could unapprove any approved translation by supplying its translation ID.
- Technique / pattern: The researcher noted a UI-level restriction, then hunted for the object identifier that the restricted action needs. Replaying the page revealed a `/get-history/` request whose response contains the approved translation's numeric ID; that ID was then replayed into the privileged endpoint with the attacker's own session cookie and CSRF token. UI-hidden does not mean server-enforced.
- Takeaway: Any action a UI hides behind a role must repeat that role check server-side on the handler itself. Sibling read endpoints that hand out internal object IDs (history, audit, metadata routes) are the standard source of the identifier needed to exercise a missing check.

### 2026-10-16 — IDOR in NASA PROSAMS exposes PII for 1,700+ application users (NASA VDP) — n/a
- Source: [Bugcrowd #5c2ca864](https://bugcrowd.com/disclosures/5c2ca864-dc69-43b1-bdd4-c61c81a08b09/idor-that-allows-disclosing-username-email-pin-firstname-lastname-uei-firmname-address-phonenumbers-etc-of-prosams-application-users)
- Type: IDOR / broken access control (P2)
- Summary: An Insecure Direct Object Reference in NASA's PROSAMS (Proposal Submissions and Awards Management System) staging application allowed unauthorized retrieval of records belonging to other users and firms, including usernames, e-mail addresses, PINs, first/last names, UEI and firm names, addresses and phone numbers for more than 1,700 users.
- Technique / pattern: A classic record-identifier walk against a proposal/award management workflow: change the user or firm reference in the request and the application returns the other tenant's full profile, including admin-level entries. The researcher scoped the impact by counting distinct records reachable, which is what moved it from "one record leaks" to a mass-disclosure finding.
- Takeaway: Grant-, proposal- and award-management portals hold dense PII keyed by sequential or guessable organization identifiers, so they are high-yield IDOR targets. Always quantify how many records an identifier walk reaches — scale is what sets severity. Note the finding was on staging, so confirm which environment is in scope before testing.

### 2026-10-16 — Explicit-media scan endpoints return message objects from channels the attacker cannot read — $2,000
- Source: [Bugcrowd #9e02d0a1](https://bugcrowd.com/disclosures/9e02d0a1-b625-4b11-966e-757e8fc43cca/permission-bypass-in-explicit-media-endpoints-leading-to-private-message-leakage)
- Type: IDOR / broken access control — permission bypass (P2)
- Summary: A chat platform's explicit-media scanning endpoint accepted a message ID plus channel ID and returned the corresponding message object without enforcing the caller's `VIEW_CHANNEL` and `READ_MESSAGE_HISTORY` permissions. An attacker could read contents of messages carrying an embed or media attachment from channels, DMs and threads they had no access to.
- Technique / pattern: Two-stage chain. First, a legitimate listing call (`GET /guilds/:guildId/channels`) leaks each channel's `last_message_id`, which updates live as users post — that supplies the object identifiers. Second, `PATCH /messages/explicit-media` with a body of `{"messages": [{"message_id": "...", "channel_id": "..."}]}` makes the server emit `MESSAGE_UPDATE` gateway events containing the full message objects back to the attacker. The response channel was the realtime gateway, not the HTTP reply, which is why the bypass went unnoticed.
- Takeaway: Safety/moderation and content-scanning endpoints are frequently built outside the main permission middleware and are a reliable place to find authorization gaps. Also check whether a result is delivered over a side channel such as a websocket or gateway event — needing to know an ID is not a mitigation when another endpoint hands those IDs out.

### 2026-10-15 — Unauthenticated Create, Read, and Delete of Any User's Data + Email Relay on JPL Hurricane Watch (NASA VDP) — n/a (P1)
- Source: [Bugcrowd #eb823c31](https://bugcrowd.com/disclosures/eb823c31-5e78-4bf9-9b2a-4c339918e879/unauthenticated-create-read-and-delete-of-any-user-s-data-email-relay-on-jpl-hurricane-watch)
- Type: Broken access control / unauthenticated IDOR
- Summary: A NASA/JPL production web application's API let anyone create, read and delete any user's saved data by supplying only a name as the record identifier — no session, cookie or token required — and the same service also exposed an unauthenticated email-relay endpoint.
- Technique / pattern: Inventory the app's API calls, then replay every state-changing request with all authentication material stripped; where a human-readable field (a name) is the only identifier, substitute other users' values to confirm cross-user read, write and delete.
- Takeaway: Any API that keys records on a guessable, user-supplied identifier should be re-tested with no credentials at all and for every verb (`GET`, `POST`, `DELETE`), not just with a second user's session.

### 2026-10-15 — View user voice activity status without sharing mutual server / channel permission (program undisclosed) — $750
- Source: [Bugcrowd #67b12410](https://bugcrowd.com/disclosures/67b12410-5e35-46bb-8b33-f311c84fa794/view-user-voice-activity-status-without-sharing-mutual-server-channel-permission)
- Type: Broken access control / presence leak over the realtime gateway
- Summary: A user in a voice call while in a "hang" presence state had their live voice activity and custom voice-channel status text exposed to accounts that shared no mutual server or channel with them. The data arrived in `PRESENCE_UPDATE` events on the gateway WebSocket at `wss://gateway.discord.gg/`.
- Technique / pattern: Attach to the application's realtime gateway WebSocket, log every pushed event, and diff the fields actually received against what the permission model allows the UI to show; authorization enforced on REST routes is often missing on the push channel.
- Takeaway: WebSocket/event-stream payloads are a separate authorization surface — compare what a client receives on the wire with the permission model instead of trusting the rendered UI.

### 2026-10-15 — An organization admin can create an attacker-controlled organization owner via the gem transfer flow (RubyGems) — n/a
- Source: [HackerOne #3938927](https://hackerone.com/reports/3938927)
- Type: Improper access control / privilege escalation
- Summary: RubyGems.org blocked organization admins from granting the owner role through the normal membership-invitation flow, but the gem transfer flow carried no such restriction, so an admin could make an attacker-controlled account an organization owner. That new owner could then remove the legitimate owners and members along with the publishing rights their memberships granted.
- Technique / pattern: Enumerate every code path that can assign a role rather than only the obvious one; when a permission rule lives in a single controller (invitations) instead of the model or policy layer, a secondary flow (transfer, import, bulk edit) often writes the same field unchecked.
- Takeaway: Authorization for role assignment belongs at the model/policy layer; when one entry point is hardened, look for the alternate flow that reaches the same privileged write.

### 2026-10-15 — IDOR while uploading attachments at a redacted endpoint (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1196976](https://hackerone.com/reports/1196976)
- Type: IDOR on a sub-resource (attachment upload), chained with a race condition
- Summary: An attacker could attach files to another user's item whenever that item had no attachment of its own yet, by supplying the victim's object identifier in the upload request; combined with a race condition the same trick also worked against freshly created items.
- Technique / pattern: Upload to your own object, capture the request, then replay it with another user's object id. Where the server only checks ownership once an attachment already exists, the empty-state window is the bug, and racing the creation step widens it.
- Takeaway: Authorization on sub-resources must be checked on every write rather than inferred from whether the parent already has children — empty and newly created states deserve their own test cases.

### 2026-10-12 — Unauthenticated project discovery enables cross-session modification of marketplace projects (Creoline) — n/a
- Source: [Bugcrowd #81921e6f](https://bugcrowd.com/disclosures/81921e6f-7696-45f8-9ace-f9b8ba283623/unauthenticated-project-discovery-enables-cross-session-modification-of-marketplace-projects)
- Type: IDOR / broken access control (unauthenticated object discovery)
- Summary: Unauthenticated enumeration of marketplace project identifiers let an attacker discover and then modify projects across other users' sessions on `marketplace.creoline.com`.
- Technique / pattern: Project objects were discoverable without authentication, and the modification endpoint relied on knowledge of the object id rather than a per-session ownership check, enabling cross-session edits.
- Takeaway: Treating an object id as a secret is not access control — discoverable ids plus a missing ownership check on the write path give cross-tenant modification.

### 2026-10-12 — Unauthenticated API allows reading, writing and deleting any user's private chat history (program undisclosed) — n/a
- Source: [HackerOne #4020767](https://hackerone.com/reports/4020767)
- Type: Unauthenticated IDOR / broken access control (API)
- Summary: An unauthenticated API allowed reading, writing and deleting any user's private chat history by referencing conversation identifiers directly, with no authentication or ownership check on the endpoint.
- Technique / pattern: The chat API exposed CRUD operations keyed on a predictable object id and never enforced a session or ownership check, so iterating ids exposed and modified arbitrary users' conversations.
- Takeaway: Every state-changing API route needs an explicit authentication and ownership gate; `unauthenticated` plus `object id in the request` is the classic recipe for mass data exposure and tampering.

### 2026-10-12 — Files drop share links for E2EE folders allow dropping into other folders of the share owner (Nextcloud) — n/a
- Source: [HackerOne #3304830](https://hackerone.com/reports/3304830)
- Type: IDOR / broken access control (file-drop share)
- Summary: A `file drop` share link for an end-to-end-encrypted folder could be abused to drop files into other folders belonging to the share owner, not only the intended drop target (CVE-2026-45159).
- Technique / pattern: The upload handler trusted a client-supplied target path/identifier from the drop share without confirming it resolved to the single folder the share was scoped to, allowing writes to sibling folders of the owner.
- Takeaway: Share links must bind every write to the exact resource they were issued for; validate the resolved destination server-side rather than trusting a path parameter sent by the uploader.

### 2026-10-12 — Missing ownership validation allows cross-manager tracker-campaign linking (Revive Adserver) — n/a
- Source: [HackerOne #3780709](https://hackerone.com/reports/3780709)
- Type: IDOR / Broken Object-Level Authorization (missing ownership validation)
- Summary: A low-privileged manager could link their own trackers to campaigns owned by other managers on the same Revive Adserver instance because the reverse linking operation in `tracker-campaigns.php` did not re-check object ownership (incomplete fix for CVE-2026-34913; assigned CVE-2026-50739).
- Technique / pattern: The original CVE fix validated ownership on one direction of the campaign-tracker link but not the inverse operation; the attacker replays the link request against another manager's campaign id to create cross-owner relationships.
- Takeaway: When patching a missing-authorization bug, audit every endpoint that performs the inverse or related operation — a fix applied to one direction of a relationship often leaves the mirror action unguarded.

### 2026-10-09 — Add labels to arbitrary issues and PRs via project column permission gap (GitHub) — bounty awarded (amount not public)
- Source: [HackerOne #3527771](https://hackerone.com/reports/3527771)
- Type: IDOR / improper authorization
- Summary: An improper authorization flaw in GitHub Enterprise Server (tracked as CVE-2026-3306, severity Medium) let a user with only read access to a repository but write access to a project change issue and pull request metadata through that project. Disclosed 2026-03-19.
- Technique / pattern: The researcher chained two different permission scopes: project write plus repository read. When an item already present in a project was added to it again, the handler applied the column-value update without re-checking that the actor held repository write permission, so project-scoped writes became a side channel for editing repository-scoped metadata such as labels.
- Takeaway: Where one object (a project) can write fields that belong to another object (a repo issue), authorization must be re-evaluated against the target object on every write path, including the idempotent "add an item that already exists" branch that developers assume is a no-op.

### 2026-10-09 — IDOR on the www.acronis.com API leaks private business user information (Acronis) — $100
- Source: [HackerOne #1182465](https://hackerone.com/reports/1182465)
- Type: IDOR (read) / predictable identifier
- Summary: A lead-record API endpoint on `www.acronis.com` did not verify that the requester was authorized to read the record it was asked for, exposing other users' company name, name and phone number. Rated Medium; submitted 2021-05-02 and disclosed 2021-08-31.
- Technique / pattern: The request carried an identifier plus a token, which at a glance looked like adequate entropy. The researcher fetched their own record first, then compared the token across accounts and found that only its last five digits varied and that they were a sequential integer — so the token was guessable despite its length, and the endpoint had no server-side ownership check behind it.
- Takeaway: Diff a "random" token across two of your own accounts before trusting it. Partial entropy is the trap: a long opaque string whose varying substring is a counter is enumerable, and the fix is both a real authorization check and a fully random identifier.

### 2026-10-09 — IDOR deletes skill assessment results and earned badges of any user (LinkedIn) — bounty awarded (amount not public)
- Source: [HackerOne #1592587](https://hackerone.com/reports/1592587)
- Type: IDOR (horizontal, destructive)
- Summary: The skill-assessment API's `DELETE` endpoint for attempt reports identified the target by a profile identifier plus a numeric skill ID and never confirmed the requester owned that data, so an authenticated user could wipe any member's assessment results and badges. Rated Medium (CVSS 5.3); submitted 2022-06-06 and disclosed 2022-10-05.
- Technique / pattern: The researcher completed an assessment to reach the legitimate results page where a delete control exists, intercepted that delete request, swapped in a victim's profile identifier (exposed in the page source of public profiles), then used Burp Intruder to enumerate the small numeric skill ID space. Replaying the request removed the victim's badge for the chosen skill.
- Takeaway: Reaching a feature's happy path first is what reveals the authenticated request worth tampering with. Mixed identifier strength is a classic tell: one component is a long public identifier and the other a short enumerable integer, and the endpoint checks neither.

### 2026-10-09 — IDOR in locid parameter exposes other accounts' profile locations (Yelp) — n/a
- Source: [HackerOne #966949](https://hackerone.com/reports/966949)
- Type: IDOR (read) / business logic
- Summary: The `locid` parameter on `GET /profile_location/add_or_edit` was not checked for ownership, so substituting another account's location ID returned that profile location. Yelp closed it as Informative (Low) on the grounds that a returned location was not tied back to a specific user; submitted 2020-08-25 and disclosed 2020-09-02.
- Technique / pattern: The researcher logged into their own account, opened the profile-location edit flow, intercepted the request in Burp and replaced `locid` with other values, confirming the server returned the record without an ownership check. They argued the random-looking IDs were still brute-forceable because no rate limiting or anti-enumeration control was present, and that relying on their unpredictability is security by obscurity.
- Takeaway: A missing ownership check is only half the report; triage weighs whether the leaked object can actually be attributed to a person. Pair the access-control finding with evidence of identifiability or enumerability, or expect an informative close.

### 2026-10-09 — Notes can be created on another user's private personal snippet (GitLab) — n/a
- Source: [HackerOne #1751258](https://hackerone.com/reports/1751258)
- Type: IDOR / broken object-level authorization
- Summary: An attacker could post a note (comment) on a victim's **private** personal snippet. Creating the note leaked the private snippet's title onto the attacker's own activity feed, and because the attacker owned the note they could then edit or delete it using the `id` the server returned in the create response.
- Technique / pattern: Create your own snippet to learn the note-creation request shape, then replay it substituting the victim's private snippet identifier. The create response is itself an oracle: it returns the new object's `id` plus parent metadata (here the snippet title), so a write primitive doubles as a read primitive.
- Takeaway: Authorization must be checked on the *parent* object, not just the child being created. Also treat the response body of a successful write as leaked data — echoed parent fields such as titles and names are a common side channel.

### 2026-10-09 — Unprivileged user can enumerate private group names via the subgroup parent_id (GitLab) — n/a
- Source: [HackerOne #215384](https://hackerone.com/reports/215384)
- Type: IDOR / information disclosure via object reference
- Summary: In GitLab 9.0's Subgroups feature, an unprivileged user could change the `parent_id` value when creating or inspecting a subgroup and the application would reveal the **name** of private groups the user had no access to.
- Technique / pattern: The researcher stood up a clean GitLab 9.0 CE instance, created a private group as root, then drove the new Subgroups UI as a low-privilege user while mutating `parent_id`. Running your own instance of the target software lets you diff privileged vs. unprivileged responses with known ground truth.
- Takeaway: Every new relational feature adds new foreign-key parameters (`parent_id`, `group_id`, `namespace_id`) and each one is a fresh authorization boundary. Error messages and validation responses that name the referenced object are still disclosure.

### 2026-10-09 — IDOR in Stocky app lets a merchant change another merchant's Low Stock settings (Shopify) — n/a
- Source: [HackerOne #853130](https://hackerone.com/reports/853130)
- Type: IDOR (write) / cross-tenant broken access control
- Summary: The Stocky app's Low Stock report stored each user's visible-column preferences against an identifier that was not bound to the session. A malicious merchant could modify the column settings of a different merchant's Stocky Low Stock variant settings. Rated Low severity.
- Technique / pattern: Classic two-account cross-tenant test: set up accounts A and B, each with its own store and Stocky installed, capture the settings-save request as A, then replay it with B's object reference. Preference and personalization endpoints are frequently missed because they look non-sensitive.
- Takeaway: Test third-party/embedded apps as their own trust boundary — a tenancy check enforced by the host platform is not automatically enforced by an app bolted onto it. Low-impact-looking write IDORs on settings are still the cheapest way to prove a missing tenancy check.

### 2026-10-09 — IDOR in Shop app exposes checkout details and shipping addresses by iterating checkout IDs (Shopify) — n/a
- Source: [HackerOne #1064869](https://hackerone.com/reports/1064869)
- Type: IDOR / sensitive data exposure
- Summary: An IDOR in Shopify's Shop app let an attacker iterate over checkout identifiers and read checkout details including shipping addresses. Shopify scoped the impact to checkouts created through the Shop app for shops owned by Shopify staff, and shipped a fix.
- Technique / pattern: Enumerate sequential or weakly-random order/checkout identifiers against the mobile app's API rather than the web storefront — companion mobile apps often talk to a separate API surface whose authorization rules lag the primary one. Proxy the app, capture one legitimate checkout read, then walk the identifier.
- Takeaway: A resource identifier that is guessable *and* an endpoint that only checks authentication gives you full-table read. Checkout, order and invoice objects should be addressed by unguessable tokens AND gated per-session.

### 2026-10-07 — Insecure Direct Object References allowing unauthorized comment access (Vimeo) — n/a
- Source: [HackerOne #52181](https://hackerone.com/reports/52181)
- Type: IDOR — information disclosure
- Summary: Comments attached to private Vimeo videos could be retrieved by anyone who knew the comment's numeric identifier. The comment-fetch endpoint resolved a `comment_id` without re-checking the privacy of the parent video.
- Technique / pattern: Classic sequential-ID probe on a *child* object. The researcher noted the parent video's privacy was enforced, but the comment resource hung off its own predictable integer (`comment_id=1301116`), so incrementing or guessing that value returned content the video ACL was supposed to protect. Built on a previously reported access-control gap in the same area.
- Takeaway: Authorization has to be re-evaluated on every nested resource, not just the top-level one. When auditing, enumerate the child objects (comments, attachments, revisions, thumbnails) that reference a protected parent — they frequently carry their own ID space and their own missing check.

### 2026-10-07 — Delete anyone's Spotlight content (Snapchat) — n/a
- Source: [HackerOne #1819832](https://hackerone.com/reports/1819832)
- Type: IDOR — unauthorized state change (delete)
- Summary: The Spotlight post-deletion flow accepted a post identifier belonging to another user, letting an attacker remove arbitrary users' Spotlight videos.
- Technique / pattern: The researcher opened their own post list at `my.snapchat.com/myposts`, intercepted the delete request in a proxy, and substituted another account's post identifier. The server performed the delete without verifying that the requesting session owned the referenced post.
- Takeaway: Destructive IDORs are easy to miss because testers tend to probe read endpoints. Always replay the delete/archive/unpublish call with a foreign object ID — and treat a 200 on someone else's object as critical, since it is unrecoverable for the victim.

### 2026-10-07 — IDOR to update folder name of other user (Trint Ltd) — n/a
- Source: [HackerOne #587687](https://hackerone.com/reports/587687)
- Type: IDOR — unauthorized state change (write) via GraphQL
- Summary: An authenticated user could rename folders owned by other accounts by supplying the victim's project identifier to the GraphQL API.
- Technique / pattern: The mutation at `graphql2.trint.com` took a `projectId` argument and applied the rename without checking ownership. The researcher logged in as user B and replayed user A's mutation with A's `projectId`, which succeeded.
- Takeaway: GraphQL concentrates many operations behind one POST endpoint, so a single missing ownership check is easy to overlook in review. When testing GraphQL, enumerate mutations separately from queries and replay each one cross-account with only the ID argument swapped.

### 2026-10-07 — IDOR on the NASA Technical Reports Server (NTRS) file download API (NASA VDP) — n/a (P3, accepted risk)
- Source: [Bugcrowd #becb15bd](https://bugcrowd.com/disclosures/becb15bd-039e-4403-b585-1b8ab536d8c5/expose)
- Type: IDOR — unauthenticated document access
- Summary: The NTRS download API served PDF documents to unauthenticated callers based on a predictable citation identifier, with no access validation on the file-download path. NASA confirmed the affected documents had in fact been declassified and approved for public release, and accepted the finding as business risk.
- Technique / pattern: The download route follows the shape `https://ntrs.nasa.gov/api/citations/<citationID>/downloads/<fileID>.pdf`. Because both identifiers are sequential and the API performed no authorization check, walking the `<citationID>` range returned documents that the site's own UI did not link — a pure enumeration of a document store through its API rather than its front end.
- Takeaway: A document portal's API surface and its web UI often enforce different rules; the UI hiding a record is not an access control. Also note the disposition here: predictable-ID enumeration is a real finding, but impact depends entirely on whether the enumerated data was actually meant to be restricted — verify that before claiming severity.

### 2026-10-07 — Horizontal privilege escalation via the phone number in an offers URL (MTN Group) — n/a
- Source: [HackerOne #2319586](https://hackerone.com/reports/2319586)
- Type: IDOR / broken access control after an OTP-gated login
- Summary: After completing the OTP login, the offers dashboard was served from `https://mtn.ng/offers/list?phone=<number>`, and substituting any other subscriber's number into the `phone` parameter returned that subscriber's dashboard with no further authentication.
- Technique / pattern: Complete the intended OTP flow once, then examine what the post-authentication URL actually keys on — when identity lives in a query parameter rather than in the session, the OTP only proved ownership of *a* number, not of the one being read; a nationally enumerable identifier such as a phone number makes the id space free.
- Takeaway: Derive the subject of every request from the session, and treat phone numbers, emails and account numbers as public input rather than as proof of identity.

### 2026-10-07 — Unauthorized deletion of any pending system-access request (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1493007](https://hackerone.com/reports/1493007)
- Type: IDOR / missing authorization on a destructive workflow endpoint
- Summary: Users requesting access to a DoD information system received an identifier for their submitted request, and the endpoint that deletes access requests honored that identifier from an unauthorized caller, letting anyone destroy legitimate pending requests and disrupt the approval workflow.
- Technique / pattern: Enumerate a workflow object's whole lifecycle — submit, review, approve, reject, delete — and call the delete handler directly instead of through the administrator interface that normally invokes it; the identifier handed back to the submitter at creation time is usually sufficient, and this was the third finding in a chain against the same application, so re-testing a previously patched area paid off.
- Takeaway: Destructive workflow endpoints need their own check tied to the reviewer role, and an id disclosed to the requester must never double as the authority to act on it.

### 2026-10-07 — Critical IDOR: delete any rule of any organization remotely (Veris) — n/a
- Source: [HackerOne #120126](https://hackerone.com/reports/120126)
- Type: IDOR / missing ownership check on a DELETE route
- Summary: A tenant-scoped rule object could be destroyed from any account by changing the rule id in the `DELETE` request, with no check that the caller's organization owned it; the same reporter found the identical flaw on members, groups, venues and terminals in the same product.
- Technique / pattern: Once one object type in a multi-tenant product falls to an id swap on `DELETE`, treat it as a product-wide authorization gap and sweep every sibling object type the same way — a fix applied to one controller rarely reaches routes that merely share the pattern.
- Takeaway: Report the pattern, not the instance: enumerate sibling object types after the first IDOR, and enforce tenant ownership in a shared authorization layer instead of per controller.

### 2026-10-06 — IDOR allows a user to access report details via the reference endpoint (HackerOne) — n/a
- Source: [HackerOne #3862641](https://hackerone.com/reports/3862641)
- Type: IDOR / information disclosure in an authorization-failure response
- Summary: The `/reports/<id>/reference.json` endpoint correctly answered HTTP 422 with "You do not have the appropriate access", yet the error envelope still carried report information in its `context.subject` field, letting VDP-signup accounts read details of reports they had no access to.
- Technique / pattern: Request a non-owned object id on a sub-resource of the report route from a low-privilege account type, then read the *error body* field by field instead of stopping at the status code — the guard rejected the request, but the serializer had already populated the object into the error context.
- Takeaway: An authorization failure must not describe what it refused; build error payloads before the object is loaded, and diff error bodies across object ids when testing access control.

### 2026-10-06 — Attacker can create malicious child epics in a victim's epic hierarchy (GitLab) — n/a
- Source: [HackerOne #1892200](https://hackerone.com/reports/1892200)
- Type: IDOR / broken object-level authorization on a create route
- Summary: Through GitLab's REST epics API a user could create epics inside their *own* group that referenced another group's epic as `parent_id`, attaching attacker-controlled children into a victim's epic tree without holding any membership of the victim's group.
- Technique / pattern: On any creation endpoint, test each foreign-key field in the body (`parent_id`, `source_id`, `target_id`) separately from the container named in the path — authorization commonly covers only the group or project you POST to, while the referenced object is accepted on trust; the victim identifiers here were obtainable without membership.
- Takeaway: Authorize every object referenced in a request body, not only the container it is created in; forming a relationship needs a permission check on both ends.

### 2026-10-06 — Boards leak private label names and descriptions (GitLab) — n/a
- Source: [HackerOne #162147](https://hackerone.com/reports/162147)
- Type: IDOR / cross-project information disclosure via a relationship object
- Summary: In GitLab's then-new issue boards feature a user could create a board list bound to a label id belonging to a private project, and the rendered list returned that label's name and description.
- Technique / pattern: Freshly shipped features lag the platform's authorization conventions — create a relationship object (board list, pin, link, attachment) using an identifier from a resource you cannot read, then harvest the attributes the response renders back for the linked object.
- Takeaway: New features need their own authorization audit, and the response of a "link these two objects" action is a read channel for the object being linked.

### 2026-10-06 — Add and access labels of any private project or group via the boards API (GitLab) — n/a
- Source: [HackerOne #439729](https://hackerone.com/reports/439729)
- Type: IDOR / missing authorization on a write route
- Summary: A non-member could send a `PUT` to `/<namespace>/<project>/boards/<board_id>.json` to add private group or project labels to a board and then read them back, even though the private project blocked non-member access throughout the interface.
- Technique / pattern: Replay the interface's own JSON write request directly instead of driving the rendered page — the board-update route authorized the board but not the label ids supplied alongside it, so a write primitive became a cross-project read.
- Takeaway: Routes exposing the same capability need identical checks; after a read endpoint is confirmed hardened, test its write variant.

### 2026-10-05 — IDOR lets a malicious user reveal the unpinned achievement badges of any Reddit user (Reddit) — n/a
- Source: [HackerOne #2618486](https://hackerone.com/reports/2618486)
- Type: IDOR / horizontal access control on a preview route
- Summary: Reddit's badge share-preview route returned a badge image for any username plus badge id, so achievements a user had deliberately unpinned in order to hide them stayed enumerable by anyone. Rated low, no bounty.
- Technique / pattern: The route `https://share.redd.it/preview/user/<username>/achievement/<badge-id>` took both the target identity and a small incrementing badge id, and a valid image versus a `Not Found` response is a boolean oracle — walking the badge id range for a username rebuilds the hidden badge set from a throwaway account.
- Takeaway: A privacy toggle that only removes an object from a profile listing is not authorization. Every route that can resolve the object, preview and image endpoints on a separate share domain included, must re-check the visibility flag, and a `404`-versus-`200` difference is itself a disclosure channel.

### 2026-10-05 — Insecure Direct Object Reference - unauthorized access to videos of a channel whose privacy is set to Private (Vimeo) — bounty awarded (amount undisclosed)
- Source: [HackerOne #45960](https://hackerone.com/reports/45960)
- Type: IDOR / private-content exposure through a secondary renderer
- Summary: Vimeo's embeddable montage widget builder rendered the videos of whatever channel identifier it was handed without consulting that channel's privacy setting, so videos in channels limited to moderators and invited people were viewable by anyone. Older (2015) but canonical.
- Technique / pattern: The builder at `/tools/widget/montage` accepted a `badge_channel` parameter; the channel page itself returned an access-denied message, yet the same id in the widget preview returned the private video list. The general move is to inventory the other surfaces that render a resource — embed builders, oEmbed, RSS, print views, OG-tag scrapers, export renderers — and replay the identifier at each.
- Takeaway: Authorization implemented in the primary view does not travel to auxiliary renderers of the same object; treat every feature that accepts a resource id as its own endpoint with its own check.

### 2026-10-05 — Mass account takeover at app.taxjar.com with no user interaction (Stripe / TaxJar) — bounty awarded (amount undisclosed)
- Source: [HackerOne #1581240](https://hackerone.com/reports/1581240)
- Type: IDOR on an invitation flow leading to account takeover
- Summary: TaxJar's delete-invitation request resolved a pending invite by its secret invitation token rather than by an identifier scoped to the caller's organization, and the flow exposed that token, so pending invitations belonging to other organizations could be claimed without any user interaction. The fix was to look the invite up by its invitation id and validate that the id belongs to the requesting user's organization.
- Technique / pattern: Treat an invitation as an object with a full lifecycle — create, list, resend, accept, revoke, delete — and record for each step what it uses as its lookup key and what it returns. Where a one-time secret doubles as the primary key, any route that accepts or echoes it becomes a credential-disclosure route, and the recovered token grants exactly the privileges the invite was meant to confer.
- Takeaway: Never key an object lookup on the secret that authorizes it. Resolve by a non-sensitive identifier, scope that identifier to the caller's tenant, and keep one-time tokens out of response bodies.

### 2026-10-05 — Embedded submission form UUIDs enumerable through the GraphQL node interface, exposing private program details (HackerOne) — n/a
- Source: [HackerOne #447930](https://hackerone.com/reports/447930)
- Type: IDOR via a GraphQL global-node resolver (opaque id decoding)
- Summary: Embedded submission forms were addressed by unguessable UUIDs, but the GraphQL `node` interface accepted base64 global ids that wrapped an auto-incrementing primary key, so an unauthenticated user could walk the integer keys, read each form's UUID and use it to pull private programs' policy, handle, resolved-bug count and bounty table. Older (2019) classic.
- Technique / pattern: Base64-decode a global node id to expose its `Type:123` structure, increment the integer, re-encode, and query the type-agnostic resolver; then feed each recovered UUID into the public parameter that consumes it, here `embedded_submission_form_uuid`. Always check whether a schema exposes a global `node` lookup beside its authorized per-type queries.
- Takeaway: A random UUID is a lookup key, not an access control. A global node resolver re-exposes every object by sequential primary key, so authorization has to live in the type's own resolver rather than in identifier entropy.

### 2026-10-01 — IDOR on ads.tiktok.com Allows Arbitrary Product Addition (TikTok) — n/a
- Source: [HackerOne #2848610](https://hackerone.com/reports/2848610)
- Type: IDOR / broken object-level authorization
- Summary: An API endpoint on TikTok's advertising platform did not verify that the caller owned the target product catalog, so an attacker could add arbitrary products into another advertiser's catalog.
- Technique / pattern: Exercise the advertiser-facing catalog API as a normal user, capture the request, then swap the owning catalog or account identifier in the body for another tenant's value and confirm the write lands on the victim's object.
- Takeaway: Write endpoints in multi-tenant advertising and commerce consoles need the same object-ownership check as read endpoints — "may this account write to this catalog" must be re-derived server-side from the session, never taken from the request.

### 2026-10-01 — India - An Insecure Direct Object Reference (IDOR) (Starbucks) — n/a
- Source: [HackerOne #701160](https://hackerone.com/reports/701160)
- Type: IDOR / horizontal privilege escalation
- Summary: The Starbucks India card service endpoint `https://card.starbucks.in/StarbucksMSRModule/starbucksGetCardData.do` accepted a `cardId` parameter without checking that the card belonged to the authenticated user, exposing other customers' card records and balances.
- Technique / pattern: Log in as a legitimate customer, capture the card-details call, and iterate the `cardId` value; a `200` carrying a different cardholder's balance confirms the missing authorization check.
- Takeaway: Legacy regional sub-applications (`.do` servlets, country-specific portals) often sit outside the main platform's authorization layer — enumerate them separately instead of assuming the flagship app's controls apply.

### 2026-10-01 — [IDOR] Modify other users' reminders (Lark Technologies) — n/a
- Source: [HackerOne #946323](https://hackerone.com/reports/946323)
- Type: IDOR / unauthorized write
- Summary: Larksuite's reminder feature accepted a `reminderId` in a POST body without verifying ownership, letting any authenticated user modify reminders belonging to other users.
- Technique / pattern: Create your own object first to learn the request shape, then replay the update call with a second account's object id; a successful mutation rather than a `403` shows the write path skips the ownership check even when the read path enforces it.
- Takeaway: Audit each verb separately — collaboration suites frequently guard `GET` on an object while leaving the `POST`/`PATCH` handler for the same id unprotected.

### 2026-10-01 — Secret Board Name Exposure (Pinterest) — 40 points (P1)
- Source: [Bugcrowd disclosure 0849f218](https://bugcrowd.com/disclosures/0849f218-4132-4396-ba93-943861f07955/secret-board-name-exposure)
- Type: IDOR / private-object metadata leak
- Summary: A researcher recovered the name of a confidential ("secret") Pinterest board belonging to a test account by addressing the board object directly, during a 2018 Pinterest challenge that invited hackers to find the hidden board.
- Technique / pattern: Treat a "private" flag as display-layer state — request the object by its identifier through a secondary surface (API, embed, share or preview route) and check whether metadata such as a name or title comes back even when the object body is withheld.
- Takeaway: Privacy has to be enforced field by field, not just on the main payload; a leaked title is still an access-control failure, so secret-object handling is worth retesting on every route that can reference the object.

### 2026-09-30 — Cross-User Lock/Unlock via Absolute DAV Path (Nextcloud) — n/a
- Source: [HackerOne #3301553](https://hackerone.com/reports/3301553)
- Type: IDOR / broken object-level authorization (cross-user write)
- Summary: In the Nextcloud `files_lock` app any authenticated user could lock or unlock files belonging to other users by addressing the victim's absolute WebDAV path, blocking the owner's writes; the response also handed the lock token back to the unauthorized caller, letting them remove other users' client locks.
- Technique / pattern: The DAV plugin resolved the target with `getFileFromAbsoluteUri()` on the raw request URI and never compared the user-ID segment in that path against the authenticated session user, so swapping that segment in a `LOCK` / `UNLOCK` request carrying the `X-User-Lock` header reached another account's files. Reachable on default deployments with `files_lock` enabled.
- Takeaway: A user-ID segment inside a path is an object reference and needs the same ownership check as a numeric `id`. Whenever a handler resolves an object from a full URI instead of from the session's own scope, every segment of that URI is attacker-controlled input.

### 2026-09-30 — Delete any folder for any user within the organization (SingleStore) — n/a
- Source: [HackerOne #3353035](https://hackerone.com/reports/3353035)
- Type: IDOR (unauthorized delete, CWE-639)
- Summary: A low-privileged SingleStore user could delete notebook folders owned by other users in the same organization by changing the folder identifier in a `DELETE` request to the notebooks contents API on `backend.singlestore.com`.
- Technique / pattern: The route `DELETE /public/notebooks/api/contents/<id>/_internal-s2-stage/<folder_id>/<folder_name>/` acted on the supplied identifiers without verifying folder ownership. The report was rated CVSS Low (3.8) only because the attack needs knowledge of two UUIDs, i.e. the mitigating factor was guessability rather than any authorization check.
- Takeaway: UUIDs raise attack complexity; they are not authorization. Destructive verbs deserve an explicit ownership check, and triage should keep "hard to guess" and "properly authorized" as separate findings.

### 2026-09-30 — Unauthorized Access to Cross Tenant Data - 1 (Atlassian / Confluence Cloud) — $1,200
- Source: [Bugcrowd #940fd364](https://bugcrowd.com/disclosures/940fd364-b42f-4c9a-9597-985cb2d83f2c/unauthorized-access-to-cross-tenant-data-1)
- Type: IDOR / cross-tenant information disclosure
- Summary: A cross-tenant information disclosure issue in Confluence Cloud allowed a user to reach data belonging to a different tenant. Atlassian's own summary classifies it as a cross-tenant information disclosure vulnerability; it was resolved at P3 with a $1,200 reward.
- Technique / pattern: In multi-tenant SaaS the tenant boundary is enforced in application code rather than by separate datastores, so any request carrying a site, tenant, cloud-id or workspace identifier is worth replaying with a second organization's identifier to see whether the server re-derives the tenant from the session or simply trusts the request.
- Takeaway: Cross-tenant authorization is a distinct layer from per-object authorization. Test it with two independent tenants you control, not with two users inside one tenant.

### 2026-09-30 — Unlisted NASA Task Book Projects and PDFs Accessible via Predictable TASKID (NASA VDP) — n/a
- Source: [Bugcrowd #6748232d](https://bugcrowd.com/disclosures/6748232d-3c95-4ed1-99b4-2470788558bd/unlisted-nasa-task-book-projects-pdfs-accessible-via-predictable-taskid-idor-style-enumeration-unauthenticated)
- Type: IDOR-style enumeration (unauthenticated)
- Summary: Incrementing the `TASKID` parameter on the NASA Task Book application returned project records and their PDFs that were not reachable through the public search interface, with no authentication required.
- Technique / pattern: Sequential integer identifiers were walked across the full range and each response diffed against what the public search UI actually listed; the records that appeared only in the direct-fetch responses were "unlisted" rather than access-controlled.
- Takeaway: Exclusion from a listing, a search index or a sitemap is a discoverability measure, not an access control. Unlisted, draft and embargoed states must be enforced at the record-fetch endpoint itself.


### 2026-09-30 — Uncontrolled access to a sensitive Department of Defense dashboard (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #992618](https://hackerone.com/reports/992618)
- Type: Improper access control (generic) — High severity
- Summary: A sensitive Department of Defense dashboard was reachable without the access control that was meant to gate it, exposing its contents to users who should not have been able to load it.
- Technique / pattern: Request a privileged view directly by URL from an unprivileged or unauthenticated session instead of navigating to it. Dashboards and reporting views are frequently wired up after the main application and inherit no authorization filter, because the only thing hiding them is the absence of a link.
- Takeaway: Every view, not just every API, needs an explicit server-side authorization check; obscurity of the route is not a control.

### 2026-09-30 — IDOR leading to leakage of usernames on a chat-linked DoD domain (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1093908](https://hackerone.com/reports/1093908)
- Type: IDOR / information disclosure
- Summary: An IDOR on a domain connected to an encrypted chat application let the reporter, testing from their own account, retrieve usernames belonging to other accounts; the exposure was limited to usernames.
- Technique / pattern: Chase the satellite domains around a primary application — the chat, support, directory and notification hosts that share its identity model — and test their object references separately. The reporter first had to establish that the domain was in scope, which is itself part of the workflow on large programs.
- Takeaway: A username-only leak is still a finding: it is the enumeration input for credential stuffing and phishing, so report it rather than discarding it as low value, and rate it by what it enables downstream.


### 2026-09-30 — IDOR in Content Outline Builder GraphQL request reveals other users' information (Semrush) — n/a
- Source: [HackerOne #1770858](https://hackerone.com/reports/1770858)
- Type: IDOR (GraphQL user-ID tampering)
- Summary: In the Content Outline Builder product, changing the user ID inside a GraphQL request returned additional information about other users; the vendor's review found no evidence of unauthorized exploitation.
- Technique / pattern: Intercept the request a single feature makes, swap only the user-ID variable for another valid ID and diff the response. Newer product surfaces bolted onto an existing GraphQL gateway inherit the schema but not always the authorization checks.
- Takeaway: A user identifier in a request body is attacker-controlled — resolve the acting user from the session and ignore any ID the client supplies.

### 2026-09-30 — Insecure Direct Object Reference allowing modification of content and database parameters (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #207099](https://hackerone.com/reports/207099)
- Type: IDOR (write-side)
- Summary: A DoD website exposed direct object references that could be manipulated to modify web content and certain database parameters rather than merely read them.
- Technique / pattern: Classic object-reference tampering aimed at state-changing requests — identify the object handle a form or API submits, substitute a reference the account does not own, and check whether the write is accepted.
- Takeaway: Cover write and delete paths when testing for IDOR; read-only enumeration understates impact, and write-side IDOR is consistently rated higher.

### 2026-09-29 — CRITICAL Insecure Direct Object Reference in the rewards signup flow (Yelp) — n/a
- Source: [HackerOne #358143](https://hackerone.com/reports/358143)
- Type: IDOR
- Summary: The `/rewards/signup` flow accepted an attacker-controlled reference to a payment-card object, letting the reporter associate a card that had been added and later deregistered with their own account. Primary account numbers stayed unreadable, so the impact was the unauthorised association itself.
- Technique / pattern: Walk a multi-step enrolment wizard and record every identifier the client hands back to the server between steps, then replay the final step with an identifier belonging to a different object and see whether the server re-checks ownership there or only at step one.
- Takeaway: Multi-step flows commonly validate ownership on the first request and trust the client's identifier afterwards — the last call in a wizard is usually the weakest link.

### 2026-09-29 — Workspace member bypasses board-deletion restriction by moving boards via the API (Trello / Atlassian) — $1,200
- Source: [Bugcrowd #36bcd5a4](https://bugcrowd.com/disclosures/36bcd5a4-4dfb-46f4-9675-f07910eefd2d/privilege-escalation-vulnerability-a-normal-workspace-member-can-bypass-the-restriction-of-board-deletetion-permision-set-by-workspace-admins-to-delete-any-board-in-the-workspace)
- Type: IDOR / broken access control — privilege escalation (P3, CVSS 6.5)
- Summary: Admins could forbid ordinary workspace members from deleting boards, but the restriction lived only in the UI; a direct `PUT /1/boards/<boardId>` naming the attacker's own organisation moved any board out of the workspace, after which that member could delete it outright.
- Technique / pattern: When the interface greys out or hides an action, find the API call the enabled version of that action would make and issue it directly with the object IDs already visible to you — here only the board ID and the attacker's own workspace ID were needed.
- Takeaway: A permission that exists only as a disabled button is not a permission; every restricted action needs the same check on the endpoint that performs it.

### 2026-09-29 — Board members run owner-only Butler automation buttons via a direct API call (Trello / Atlassian) — $300
- Source: [Bugcrowd #1418142b](https://bugcrowd.com/disclosures/1418142b-8b82-4ad6-9c9d-ce9ff1baf3ea/privilege-escalation-vulnerability-other-users-of-the-board-can-run-local-automation-buttons-for-owner-which-is-not-visible-to-them)
- Type: IDOR / broken access control — hidden-function invocation (P4)
- Summary: Butler automation buttons set to personal were hidden from other board members but remained executable: a request to the `powerup-run-command` API with the caller's own token plus the owner's `cmd_id`, `uid` and `board_id` ran the owner's automation.
- Technique / pattern: Where an identifier is predictable — `cmd_id` was the owner ID plus an incrementing counter — and the remaining parameters are already visible to every member, enumerate it and replay the command endpoint under your own session.
- Takeaway: Hiding a control from the interface is not authorisation, and sequential per-owner command IDs make the hidden objects trivially enumerable.

### 2026-09-29 — Privilege escalation to any user group via an unverified group-change endpoint (NASA — GLOBE) — n/a
- Source: [Bugcrowd #2fd3398f](https://bugcrowd.com/disclosures/2fd3398f-7b58-4f37-88a1-ba920c141a5f/critical-privilege-escalation-to-any-user-group-account-status-to-regional-office-representative)
- Type: Broken access control — mass assignment / privilege escalation (P1)
- Summary: A user-management endpoint on the GLOBE platform did no adequate server-side verification of group changes, so a low-privileged account could set itself to Regional Office Representative and other privileged groups, gaining create, modify and delete rights over events, news and resources.
- Technique / pattern: Submit the profile or membership update request with the role or group field changed to a higher-privileged value, then check whether the server re-derives the caller's authority or simply persists whatever arrived.
- Takeaway: Role and group fields must be server-derived; if a self-service profile update can carry them, self-service becomes self-promotion.

### 2026-09-28 — IDOR in Circles getMembership() exposes any user's membership in any circle (Nextcloud) — $200
- Source: [HackerOne #3484601](https://hackerone.com/reports/3484601)
- Type: IDOR
- Summary: `MembershipService::getMembership()` in the Nextcloud Circles app looked up membership records using only the supplied `circleId` and `singleId`, with no check that the caller was allowed to see that pair. Any authenticated user could read membership levels, inheritance paths and circle relationships for any user and any circle (CVE-2026-68493).
- Technique / pattern: Trace a read endpoint (here the `link()` action in `LocalController`) down through the service layer to the repository call and look for the authorization check in between; when the service forwards caller-supplied identifiers straight into the query builder, the object reference is unprotected. Two opaque IDs are not a control, since `singleId` values leak in-band elsewhere in the app.
- Takeaway: Authorization belongs beside the lookup in the service layer, not in whichever controller happens to call it, and a `detailed` flag that fans out to related objects widens the same missing check.

### 2026-09-28 — Shared smart albums resolve the viewer's source-folder config against the owner's files (Nextcloud) — n/a
- Source: [HackerOne #3506873](https://hackerone.com/reports/3506873)
- Type: IDOR / broken access control
- Summary: When a user opened a filter-based smart album shared by someone else, `FiltersManager` built the search conditions from the *viewer's* `photosSourceFolders` setting instead of the album owner's, so a viewer who configured broader paths had the album's filters run across folders the owner never intended to expose (CVE-2026-82985).
- Technique / pattern: On any shared or delegated resource, ask whose configuration is read at evaluation time. Set your own preference to something deliberately wide, open the victim-owned shared object, and check whether the result set grows: a config value that is attacker-controlled but applied to another user's data is the bug.
- Takeaway: Scope for a shared object must be resolved from the owner's stored settings at share time; reading the requester's preferences quietly promotes a user setting into an access-control parameter.

### 2026-09-28 — Unauthenticated IDOR on a self-hosted GitLab users API leaks SSH key metadata and internal hostnames (NASA VDP) — n/a
- Source: [Bugcrowd #713064c8](https://bugcrowd.com/disclosures/713064c8-35c7-41ac-b6b3-443c1c9daeaa/unauthenticated-idor-on-users-api-leading-to-information-disclosure-of-internal-hostnames-and-pii)
- Type: IDOR
- Summary: NASA's private self-hosted GitLab at `gitlab.smce.nasa.gov` correctly returned 403 on the bare users API, but a filter-parameter variant of the same endpoint answered unauthenticated requests and enumerated every account. Feeding the recovered user IDs into a second unauthenticated endpoint returned SSH key metadata whose key titles carried internal workstation hostnames, data-centre FQDNs and contractor email addresses.
- Technique / pattern: A 403 on the collection route is not a verdict on the whole API: retry with filter, search and pagination parameters and with per-ID routes, because the check is often attached to one handler rather than to the resource. Then chain, letting the IDs from step one become the object references for step two.
- Takeaway: Test every parameterised form of an endpoint that returned 403, and treat free-text fields such as SSH key titles as disclosure surfaces in their own right.

### 2026-09-28 — Broken function-level authorization in a project funding API exposes internal staff PII (NASA VDP) — n/a
- Source: [Bugcrowd #003aefd2](https://bugcrowd.com/disclosures/003aefd2-dd3a-41a2-9e3a-448ba90403d1/broken-function-level-authorization-allows-standard-users-to-access-internal-funding-manager-pii-via-project-funding-api)
- Type: IDOR / broken function-level authorization
- Summary: An API endpoint in NASA's Request Management System that supplied funding metadata during the project-creation workflow did not enforce function-level authorization, and its response was far more verbose than the interface displayed. Any standard authenticated user could read internal funding managers' personally identifiable information.
- Technique / pattern: Compare what the front end renders against the raw JSON the backend returns for the same workflow step; the gap between the two is where over-fetching lives. Then replay that call from a low-privilege account to see whether the role check exists only in the UI.
- Takeaway: Data minimisation and function-level authorization are separate controls and both were missing here — trimming the response is not a fix while the endpoint still answers unprivileged callers.

### 2026-09-28 — Mass PII Disclosure (Emails/Phones) via IDOR on Headless API (NASA VDP) — n/a
- Source: [Bugcrowd #ab0208c1](https://bugcrowd.com/disclosures/ab0208c1-df67-47ac-a671-990ce55996d8/mass-pii-disclosure-emails-phones-via-insecure-direct-object-reference-idor-on-headless-api)
- Type: IDOR / Broken Object-Level Authorization
- Summary: A headless (decoupled front end) API on a NASA asset served user records by direct object reference, so an unauthorized caller could pull email addresses and phone numbers at scale. Rated P3 and resolved.
- Technique / pattern: Headless/JAMstack sites move data access to a separate content API that the browser calls directly — enumerate the object ids in those XHR calls rather than in the rendered page, because the delivery API is frequently deployed without the CMS's own permission model.
- Takeaway: When a site is “headless”, treat its content/delivery API as a distinct authorization surface: the front end's access rules do not travel with it.

### 2026-09-28 — Broken Access Control: Unauthenticated Mass Data Extraction and Arbitrary Item Deletion (NASA VDP — NASA Trek) — n/a
- Source: [Bugcrowd #ba2df973](https://bugcrowd.com/disclosures/ba2df973-58fc-48be-9fb2-8205cca4e7c0/broken-access-control-unauthenticated-mass-data-extraction-and-arbitrary-item-deletion)
- Type: IDOR / Broken Access Control (unauthenticated)
- Summary: Internal API endpoints on the NASA Trek platform were missing authentication and authorization, so internal object identifiers could be enumerated and the underlying items both read in bulk and deleted. Rated P2, resolved.
- Technique / pattern: Improper input handling combined with exposed HTTP methods on “internal” routes: enumerate the ids, then walk the verb set (`GET`, `POST`, `PUT`, `DELETE`) on each route — read access and destructive access are separate findings, and the delete path is usually the one nobody guarded.
- Takeaway: “Internal” endpoints reachable from the internet more often have no authorization at all than weak authorization; test the destructive verbs too, carefully and only against records you created.

### 2026-09-28 — API-only delegated admin can enumerate all Team Folders and grant access to arbitrary groups (Nextcloud) — n/a
- Source: [HackerOne #3674940](https://hackerone.com/reports/3674940)
- Type: IDOR / Privilege escalation (broken object-level authorization)
- Summary: A delegated admin restricted to `API/REST only` could call `POST /index.php/apps/groupfolders/folders/<id>/groups` for any Team Folder id and add a group they controlled, because the handler never checked folder-level authorization.
- Technique / pattern: Pair a *restricted* administrative role with a *grant-access* endpoint keyed on an auto-increment id: the ids are predictable, so the attack is enumerate-then-mass-assign rather than a single object tamper. Hunt routes that create a permission (`/groups`, `/shares`, `/members`, `/acl`) rather than routes that read data.
- Takeaway: Authorization on a permission-granting route must be evaluated against the target object, not the caller's global admin flag — predictable ids turn one missed check into organization-wide access.

### 2026-09-28 — Air Force candidate PII and recruitment chat logs accessible via BAC/IDOR on a DoD Salesforce asset (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #2968391](https://hackerone.com/reports/2968391)
- Type: IDOR / Improper Access Control (Salesforce record-level)
- Summary: Record-level configuration on the `Document` object of a DoD Salesforce instance let an attacker retrieve a very large number of Air Force recruiting records — full names, addresses, phone numbers, emails and recruiter chat transcripts containing medical, drug-use, criminal and academic detail. Rated critical.
- Technique / pattern: Salesforce-backed portals expose record ids that are enumerable whenever object- and record-level permissions are misset; light id fuzzing plus a response-size oracle established the scale without needing to pull the data.
- Takeaway: On Salesforce/Experience Cloud assets audit the standard objects (`Document`, `Attachment`, `ContentDocument`) separately — sharing rules, not application code, are the access control, and one misconfigured object can expose an entire program's records.


### 2026-09-27 — IDOR leaking PII via the `VendorId` field of a save request (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1690044](https://hackerone.com/reports/1690044)
- Type: IDOR / object-level authorization missing on a write endpoint
- Summary: A `POST` to `Vendor/Company/Contacts/SavePOC` carried a `VendorId` field that the server trusted without checking that the session owned that vendor, so swapping it to another id returned that vendor's point-of-contact personal data.
- Technique / pattern: Intercept a save/update request that already contains an owning-entity id in its body, change only that id, and replay. Write endpoints get skipped in authorization testing because testers concentrate on `GET` reads, yet a `Save*` handler often echoes the target record back in its response — which turns a write primitive into a read oracle.
- Takeaway: Any request naming an entity id in its body must re-derive ownership from the session; a body field is attacker-controlled input, never a trust boundary.

### 2026-09-27 — Team object in GraphQL disclosed a private comment via a guessable node id (HackerOne) — $2,500
- Source: [HackerOne #978143](https://hackerone.com/reports/978143)
- Type: IDOR / broken object-level authorization through GraphQL global node ids
- Summary: An unauthenticated `POST` to `/graphql` querying a `SurveyRatingItem` node — addressed as `gid://hackerone/SurveyRatingItem/<id>` — returned survey rating data including the `private_comment` field that should have been restricted to the team.
- Technique / pattern: GraphQL global ids are usually plain `gid://<app>/<Type>/<numericId>` strings or their `base64` form, so once a type name is known the id space is enumerable. Query the node interface directly for every type you can name, unauthenticated first, and diff which fields come back against what the UI shows.
- Takeaway: A global node id is an address, not a capability; the node resolver itself needs an authorization check for every type it can return.

### 2026-09-27 — Invited admin re-enables a disabled input to demote the organization owner (Stripo Inc) — n/a
- Source: [HackerOne #751299](https://hackerone.com/reports/751299)
- Type: IDOR / broken function-level authorization behind a client-side-only control
- Summary: An admin invited into an organization could edit the owner's role by using browser devtools to remove the `disabled` attribute from the role selector and sending the resulting `PUT` request, demoting the owner to admin and locking them out of their own organization permanently.
- Technique / pattern: Treat every greyed-out control as a live endpoint — drop the attribute, submit, and see whether the server accepts it. Role-change handlers are a recurring blind spot precisely because the UI already prevents the illegal transition, so no server-side rule was ever written for it.
- Takeaway: A `disabled` field is a hint to the user, not an authorization control; role transitions need an explicit server-side rule about who may change whom, including who may change the owner.

### 2026-09-27 — IDOR lets an attacker create and verify accounts for addresses they do not own (WakaTime) — n/a
- Source: [HackerOne #244636](https://hackerone.com/reports/244636)
- Type: IDOR / verification request not bound to the requesting account
- Summary: The email-confirmation request body `{"email":"<target>"}` was not tied to the session that sent it, so an attacker could request a verification link for an arbitrary address, receive the link themselves, and stand up a controlled account for an email they never owned.
- Technique / pattern: Find the request that triggers a confirmation mail, swap the address in the body, and trace where the token lands. When the token is generated for the value in the request but delivered to the session's own inbox, the binding between "who asked" and "what got proved" is broken — the same check to run against invite, re-send and address-change flows.
- Takeaway: A verification token must be bound to both the requesting session and the address being proved, and delivered only to that address.

### 2026-09-27 — Improper Authorization Leads to Vertical Privilege Escalation (Lovable VDP) — n/a
- Source: [HackerOne #3371448](https://hackerone.com/reports/3371448)
- Type: IDOR / broken function-level authorization (BFLA)
- Summary: A workspace member holding only the Editor role could call the admin-only endpoint `/workspaces/<WORKSPACE_ID>/tool-preferences/ai_gateway/enable`, because the route enforced no server-side role check. A non-admin could therefore flip a workspace-wide administrative setting.
- Technique / pattern: Log in with the lowest role a tenant offers, then replay every administrative request you can discover — from the admin UI, JS bundles or API docs — using that low-privilege session. The tell is a route whose path already scopes the tenant correctly, so object-level authorization looks fine, while the *function* behind it is only hidden in the UI.
- Takeaway: Scoping a route to a workspace id is not authorization; every privileged capability needs its own server-side role check, and "the button is not rendered for Editors" is not one.

### 2026-09-27 — Improper access control on LinkedIn Pages: a downgraded admin keeps super-admin actions (LinkedIn) — n/a
- Source: [HackerOne #1587246](https://hackerone.com/reports/1587246)
- Type: IDOR / stale privilege after a role change
- Summary: A user granted `super admin` on a LinkedIn Page kept the ability to publish as super admin after their role was edited down to `analyst`, because the already-open session's privileges were never re-evaluated against the new role.
- Technique / pattern: Take a role you are about to lose, open the privileged view, have the role downgraded *without closing the session*, then replay the privileged action unchanged. Authorization computed at grant time — or cached in session/view state — leaves a window that an ordinary UI reload would hide.
- Takeaway: Re-derive permissions from current server-side state on every request, and invalidate or re-authorize open sessions whenever a role is changed or removed.

### 2026-09-27 — View another user's profile by swapping the `UID2` cookie (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1004745](https://hackerone.com/reports/1004745)
- Type: IDOR / identity taken from a client-supplied cookie
- Summary: A profile page decided whose record to display from a `UID2` cookie value rather than from the authenticated session, so an authenticated attacker who rewrote that cookie to another user's id read that user's profile.
- Technique / pattern: Treat the cookie jar as a parameter surface — after logging in, diff every cookie for anything resembling an identity (`UID`, `userid`, `acct`, an encoded email) and increment or swap it. A cookie is attacker-controlled input exactly like a query parameter, but it is often consumed by an older code path that predates the session layer.
- Takeaway: Identity must be derived server-side from the session token alone; never read the subject of a request from a cookie, header or body field the client can rewrite.

### 2026-09-27 — Sensei LMS below 4.5.2: arbitrary private-message sending via IDOR, CVE-2022-2080 (Automattic) — n/a
- Source: [HackerOne #1592596](https://hackerone.com/reports/1592596)
- Type: IDOR / missing sender validation on a write
- Summary: The Sensei LMS WordPress plugin never verified that the sender of a private message was the teacher or the original participant of that conversation, so any authenticated user could inject messages into arbitrary private threads — though not read the replies.
- Technique / pattern: On messaging, comment and thread features, test the *write* side separately from the read side, because participation checks are frequently implemented only where a conversation is rendered. Supply another conversation's id in the send request from an unrelated account; a write-only IDOR still enables impersonation and harassment even when reads stay protected.
- Takeaway: Every message-send path must re-check that the session is a participant in that specific thread, and a partial IDOR (write without read) is still a reportable authorization failure.

### 2026-09-26 — IDOR protection bypass by changing HTTP method in IBM Your Learning endpoint (IBM) — n/a
- Source: [HackerOne #2456603](https://hackerone.com/reports/2456603)
- Type: IDOR / broken object-level authorization
- Summary: An IBM Your Learning endpoint enforced object-level authorization on one HTTP verb but not on others, so re-issuing the same request with a different method bypassed the IDOR protection and reached another user's object. IBM confirmed and remediated it.
- Technique / pattern: After an object-reference tamper is blocked, replay the identical request with a different verb (`GET` to `POST`, `POST` to `PUT`/`PATCH`, plus `HEAD` and override headers such as `X-HTTP-Method-Override`). Authorization filters are often wired per route+verb, so an unmapped verb falls through to a handler with no check.
- Takeaway: A "fixed" IDOR is only fixed for the verb that was tested; always re-test every method the route accepts, because access control bound to route+verb pairs leaves gaps.

### 2026-09-26 — Remove every user, admin and owner from their teams on developers.mtn.com via IDOR + information disclosure (MTN Group) — n/a (Critical 9-10)
- Source: [HackerOne #1448550](https://hackerone.com/reports/1448550)
- Type: IDOR / missing object-level access control
- Summary: The team member-removal request on `developers.mtn.com` trusted attacker-supplied `user_id` and `team_id` values with no ownership check, so any authenticated user could evict arbitrary users (including team owners and admins) from teams they had no access to. The response also leaked the victim's username and the team name.
- Technique / pattern: Create three accounts to establish a victim relationship you are deliberately outside of, intercept the legitimate "remove member" call in a proxy, then swap both identifiers for the victim pair and replay. Because both IDs were short sequential integers (4 digits), the request could be driven through an intruder-style sweep over the full `user_id` x `team_id` space to affect every team at once.
- Takeaway: Object-level checks must cover both the target object and the container it lives in; short sequential IDs turn a single-victim IDOR into a platform-wide denial of service, and a response that echoes names converts it into a PII leak as well.

### 2026-09-26 — Insecure Direct Object Reference in report participant removal (HackerOne) — $500
- Source: [HackerOne #46397](https://hackerone.com/reports/46397)
- Type: IDOR on a destructive DELETE endpoint
- Summary: The program-portal call that removes an external participant, `DELETE /reports/<reportId>/external_users/<userId>`, did not verify that `<userId>` was actually a participant on that report, so changing the value generated a genuine "you were removed" notification email to any user on the platform.
- Technique / pattern: Look for identifiers embedded in the URL path of destructive verbs rather than in the body; path parameters are frequently used to load the object but not to authorize it. The impact here is the side effect (a platform-sent email) rather than data read, so mass abuse is proven by the notification, not by a response body.
- Takeaway: IDOR impact is not limited to reading data; write and delete endpoints can be abused to send trusted, legitimate-looking notifications from the target's own infrastructure. Check that the referenced object belongs to the parent resource, not just that it exists.

### 2026-09-26 — Broken Access Control on NASA SPDF host (NASA Vulnerability Disclosure Program) — n/a (Informational)
- Source: [Bugcrowd f0e3ceec](https://bugcrowd.com/disclosures/f0e3ceec-b3a8-416f-90c2-759a45f78a67/broken-access-control)
- Type: Broken access control / unauthorized resource access
- Summary: A researcher reported that restricted directories and resources on a NASA SPDF host could be reached without proper authorization, exposing research documents and operational logs. NASA closed it as Informational (accepted risk).
- Technique / pattern: Enumerate object identifiers programmatically against a status route (the report used a script sweeping campaign IDs against a `/status` endpoint) and compare responses across locales, then walk the directory structure toward administrative and user-management paths. Scripted enumeration plus response-diffing is what separates a guess from evidence.
- Takeaway: Enumeration evidence is necessary but not sufficient; this closed as Informational because the write-up asserted broad impact (CSRF chaining, admin panel reach) without demonstrating it. Prove the single strongest reproducible consequence rather than listing hypothetical ones.

### 2026-09-25 — Autotranslate DDP method exposes private messages without authentication or room access check (Rocket.Chat) — n/a
- Source: [HackerOne #3734326](https://hackerone.com/reports/3734326)
- Type: IDOR / broken object-level authorization (BOLA)
- Summary: The Meteor DDP method `autoTranslate.translateMessage` accepted a client-supplied message object and passed it to `translateMessage()` without checking `Meteor.userId()` or verifying room membership, so any connected user could read message bodies from rooms they were not a member of — including private channels, DMs and end-to-end-encrypted rooms.
- Technique / pattern: The equivalent REST route had already been fixed, so the researcher looked for the *other* transports that reach the same server function and replayed the call over DDP supplying only a target message id. The method returned the message content, confirming the authorization check lived in the route rather than in the shared handler.
- Takeaway: Patching one transport does not patch a capability. Inventory every protocol surface that reaches the same handler — REST, DDP/WebSocket, RPC, GraphQL — and enforce the ownership check inside the handler, not per route.

### 2026-09-25 — Hidden and restricted tags can be mutated through synonym ID paths without per-tag authorization (Discourse) — n/a
- Source: [HackerOne #3689633](https://hackerone.com/reports/3689633)
- Type: IDOR / missing per-element authorization in a bulk operation
- Summary: `TagsController#create_synonyms` authorized only the route's target tag with `guardian.ensure_can_edit_tag!(@tag)`, then processed attacker-supplied synonym tag ids from the request body without re-checking edit permission or visibility for each one, letting a user fold hidden or restricted tags into a tag they legitimately control.
- Technique / pattern: Reading the open-source controller showed a single guard standing in front of a loop over ids taken from the payload. The researcher then sent one tag id they owned together with one restricted tag id and watched the second be accepted.
- Takeaway: In bulk and relationship-forming endpoints the guard is usually written once against the named container. Authorize every element of an id array or id list, not just the primary object named in the path.

### 2026-09-25 — Cross-tenant data access: IDOR and information disclosure in the Jira custom field metadata endpoint (Atlassian) — 10 points (P3)
- Source: [Bugcrowd e73a6c59](https://bugcrowd.com/disclosures/e73a6c59-737f-48b1-9202-dae9aa49092d/cross-tenant-data-access-idor-and-information-disclosure-in-jira-custom-field-metadata-endpoint)
- Type: IDOR / cross-tenant broken access control
- Summary: A Jira endpoint that serves custom field metadata did not scope its authorization check to the caller's own tenant, so manipulating the identifiers in the request returned field configuration data belonging to other customer organizations.
- Technique / pattern: In a multi-tenant SaaS product, the researcher took an ordinary authenticated request from their own site and swapped the object identifiers for ones belonging to a different tenant. Configuration and metadata endpoints are a good place to try this because they are often treated as low-sensitivity plumbing and skipped when tenant scoping is added to the main data APIs.
- Takeaway: In multi-tenant apps every query must be scoped by tenant at the data layer, not merely checked for "is this caller authenticated". Metadata, schema and configuration endpoints deserve the same tenant filter as the records themselves.

### 2026-09-25 — Authenticated BFLA in the DIRS Critical Need of Help API exposes all submitted requests and PII (FCC Vulnerability Disclosure Program) — n/a (P1)
- Source: [Bugcrowd 4f63c6af](https://bugcrowd.com/disclosures/4f63c6af-68cf-4d58-a1c9-911fb8a202ad/authenticated-bfla-in-dirs-critical-need-of-help-api-allows-unauthorized-access-to-all-submitted-requests-and-pii)
- Type: Broken function-level authorization (BFLA) / IDOR
- Summary: An authenticated endpoint in the FCC DIRS platform was intended only for *submitting* emergency assistance requests, but it also answered `GET` and returned every historical submission — bulk PII, emergency details and operational metadata belonging to other organizations and individuals.
- Technique / pattern: Verb enumeration against a write-only route: the researcher took an endpoint the UI only ever issues `POST` to and tried `GET` on the same path with a low-privileged account. The read path had no function-level authorization because the designers never expected it to be reachable.
- Takeaway: Authorize per method, not per path. Any route the UI uses in one direction should be tested in every other verb, and a "submit" endpoint that silently supports a list operation is a bulk-disclosure bug.

### 2026-09-24 — IDOR in a GraphQL campaign mutation allows deleting any campaign by id (HackerOne) — n/a
- Source: [HackerOne #1969141](https://hackerone.com/reports/1969141)
- Type: IDOR — unauthorized object deletion via a GraphQL mutation
- Summary: A `POST /graphql` mutation accepted a `campaign_id` the caller did not own and carried out the deletion, so any campaign could be destroyed by supplying its identifier.
- Technique / pattern: Enumerate mutations from the application's JavaScript bundles, then replay each destructive one with an object id taken from a second account. Deletion mutations are routinely missed by authorization test suites precisely because they are awkward to test non-destructively.
- Takeaway: Authorization has to be enforced inside every resolver, not only on the queries that read an object; treat destructive GraphQL mutations as a first-class IDOR sink.

### 2026-09-24 — IDOR at checkout lets an attacker pay with another user's stored credit card (Yelp) — n/a
- Source: [HackerOne #391092](https://hackerone.com/reports/391092)
- Type: IDOR — cross-account use of a stored payment instrument
- Summary: The Grubhub-backed ordering flow exposed a `/checkout/transaction_platform` request in which the reference to a saved payment method could be swapped for another user's, letting the attacker place and pay for orders on someone else's card without ever seeing the card data.
- Technique / pattern: On any checkout that references a saved payment method by an opaque token or id, capture the final submit request from account A and replay it from account B carrying A's payment reference. Tokenization hides the card number but does not by itself bind the token to its owner.
- Takeaway: A payment token is an object reference like any other — the server must verify at charge time that it belongs to the authenticated purchaser, not merely that it is well formed.

### 2026-09-24 — Unauthorized access to confidential data via an admin-only endpoint (NASA VDP) — n/a (P4)
- Source: [Bugcrowd eb75437c](https://bugcrowd.com/disclosures/eb75437c-841c-44e5-a5b0-2c3613a48309/unauthorized-access-to-confidential-data-via-admin-endpoint)
- Type: Improper access control — role check missing on a privileged endpoint
- Summary: An endpoint intended only for the "Photo Approvers - Admins" group returned its sensitive contents to any authenticated user; the route required a session but never checked group membership.
- Technique / pattern: Authentication was present, so the endpoint did not look unprotected — the gap was the missing *role* check. Harvest admin-only routes from JavaScript bundles, sitemaps and role documentation, then request each one with a low-privilege session and compare against the `403` you expect.
- Takeaway: "Requires login" and "requires the right role" are two different controls; test every privileged route with an ordinary account rather than assuming a login wall implies authorization.

### 2026-09-24 — Privilege escalation allows deleting any account in an organization, including the main admin (Indeed) — 10 points (P3)
- Source: [Bugcrowd 96eefd21](https://bugcrowd.com/disclosures/96eefd21-7376-4fd6-b6e0-56168f5ef1fd/privilege-escalation-to-delete-main-admin-user-account-indeed-com)
- Type: Broken access control — vertical escalation on a user-management action
- Summary: On `account.indeed.com` a lower-privileged organization member could invoke the member-removal action against any account in the organization, up to and including the owning administrator.
- Technique / pattern: In multi-tenant team settings, enumerate the member-management actions (invite, change role, remove) and run each from the lowest role against a higher-privileged member id. Removal endpoints are often gated only in the interface, which hides the button but leaves the route open.
- Takeaway: Destructive team-management routes need a server-side rule that the actor outranks the target; hiding a control in the UI is not authorization.

### 2026-09-24 — IDOR / Broken Access Control on /users/{id} exposes other users' PII and password hash (NASA VDP) — n/a (P3)
- Source: [Bugcrowd fdb0f70d](https://bugcrowd.com/disclosures/fdb0f70d-1cb2-4709-ad64-78f078d8e2be/idor-broken-access-control-users-id-exposes-other-users-pii-and-password-hash-password_)
- Type: IDOR — missing ownership check plus an over-permissive serializer
- Summary: An authenticated user could read other users' profile records by modifying the identifier on a `/users/<id>` API endpoint, and the response additionally leaked sensitive attributes the UI never displays, including the stored password-hash field (`password_`).
- Technique / pattern: Two defects in one response — the missing object-level authorization check, and a serializer returning the whole ORM row instead of a public projection. After confirming cross-user access, diff the JSON against what the front end actually renders: internal columns (hashes, tokens, `is_admin`, internal hostnames) are routinely still in the payload because nobody reads the raw response.
- Takeaway: Rate an IDOR by the *fields* in the body, not merely by the fact of cross-user access. A leaked password hash turns a routine read into offline cracking and credential-stuffing material.

### 2026-09-24 — Authenticated IDOR in DIRS Users API allows access to other users' profiles and PII (FCC VDP) — n/a (P1)
- Source: [Bugcrowd 479095d0](https://bugcrowd.com/disclosures/479095d0-e0f9-4b8d-bd64-feda95bb6b17/authenticated-idor-in-dirs-users-api-allows-access-to-other-users-profiles-and-pii-production)
- Type: IDOR / Broken Object Level Authorization
- Summary: In the FCC DIRS production environment an authenticated API endpoint returned any user's full profile and associated company data when a predictable numeric `userid` in the request path was changed, exposing PII and organizational metadata on a system used by telecommunications providers.
- Technique / pattern: Log in with the lowest privilege the application will grant, find the route that renders *your own* profile (`/users/<userid>`), then walk the numeric identifier in the path. A profile route that answers for neighbouring ids also yields free user enumeration, which lifts the finding from disclosure to a targeted-phishing and social-engineering primitive.
- Takeaway: "Authenticated" is not "authorized". An identifier in a path segment needs the same ownership check as one in a query string, and enumerability compounds the impact far past the single record you first read.

### 2026-09-24 — IDOR Exposes PII of Tens of Thousands of Users and Supervisors (U.S. Dept Of Defense) — n/a (High)
- Source: [HackerOne #2967032](https://hackerone.com/reports/2967032)
- Type: IDOR — sequential numeric identifier (CWE-639)
- Summary: A system-access-request (SAAR) workflow honored a `saarnId` URL parameter without checking that the session owned the record, so decrementing the value returned other users' submissions — address, full name, email, phone, date of birth, supervisor, DoD ID number, clearance level and Controlled Unclassified Information — across tens of thousands of people.
- Technique / pattern: Found while using the application exactly as intended: submit one legitimate request, note the identifier handed back in the URL, then decrement it. Sequential integers on a submission workflow are the highest-yield IDOR shape there is, because every lower value is guaranteed to be a real record rather than a guess.
- Takeaway: Onboarding and request-submission workflows accumulate the richest PII an organization holds and are usually built outside the main product's authorization conventions. Test the identifier returned by your own submission before anything else on the target.

### 2026-09-24 — Insecure Direct Object Reference (IDOR) allows creating folders (SingleStore) — n/a (Low)
- Source: [HackerOne #3353057](https://hackerone.com/reports/3353057)
- Type: IDOR — write-side / container-level authorization missing
- Summary: A low-privileged authenticated user could create folders and files inside other users' workspaces by changing the `clusterID` parameter on a `POST` to `/public/notebooks/api/contents/` on `backend.singlestore.com`; the handler never verified that the session owned the referenced workspace.
- Technique / pattern: Notebook and workspace features expose a filesystem-style API whose container is named in the request. Take a legitimate create request from your own workspace, swap the container identifier for another tenant's, and look for a success rather than a `403`. A write that lands proves the container itself was never authorized — and unlike a read IDOR, nothing in the response gives it away, so the confirmation has to be a follow-up read from the victim side.
- Takeaway: Write-side IDOR on a container identifier is easy to miss because nothing is disclosed. Authorize the container (`clusterID`, `workspaceId`, `projectId`), not only the rows or paths beneath it.

### 2026-09-23 — Create-only role can read Access Role configuration (Akeyless (Bugcrowd)) — n/a (P5, informational)
- Source: [Bugcrowd 5f01f264](https://bugcrowd.com/disclosures/5f01f264-c35f-4323-a246-026b1569cd89/broken-access-control-issue)
- Type: Broken object-level authorization / inconsistent permission model
- Summary: On the Akeyless console an account granted only Create permission on Access Roles could still read those roles' details and configuration — inconsistent, since the same Create-only grant correctly blocked reads of Secrets, Keys and Auth Methods.
- Technique / pattern: Built by hand from a permission matrix: bind an Access Role granting Create (and nothing else) across every resource type, authenticate with those restricted credentials, then attempt a read on each resource type and compare which ones answer.
- Takeaway: When one resource type enforces a permission correctly, use it as the oracle for the others. Differential testing across resource types surfaces the single handler that skipped the check.

### 2026-09-23 — Improper authorization on the App Search credentials API exposes all API keys (Elastic) — bounty awarded (amount undisclosed)
- Source: [HackerOne #1218680](https://hackerone.com/reports/1218680)
- Type: Broken object/function-level authorization
- Summary: A user holding only the Dev role with access limited to specific engines could still call `/api/as/v1/credentials/` directly and read every API key in the deployment, including private keys with read/write access to all engines.
- Technique / pattern: The engine restriction was enforced in the UI and on engine-scoped routes but not on the account-wide credentials route. A down-scoped role calling the collection endpoint directly, rather than through the product UI, surfaces the gap.
- Takeaway: Drive a role-by-endpoint permission matrix from the API, not the UI. Scope checks enforced on data routes are frequently absent on credential, billing and settings routes, and exposed keys turn a read bug into privilege escalation.

### 2026-09-23 — IDOR when editing blogs lets any user rewrite another user's site details (Automattic (IntenseDebate)) — bounty awarded (amount undisclosed)
- Source: [HackerOne #974222](https://hackerone.com/reports/974222)
- Type: IDOR / missing ownership check on write
- Summary: The profile editor on IntenseDebate saved website/blog records keyed on a hidden form field, `hidBlogID`, without verifying that the submitting account owned that blog, so any authenticated user could overwrite another user's stored blog and website information.
- Technique / pattern: The victim's own blog identifier already appeared in page source as `radMainSite`, supplying a valid target id without brute force. Pairing that identifier leak with the write endpoint that consumes the same id demonstrates impact.
- Takeaway: Hidden form fields are user input. When an object id is rendered into a page somewhere on the site, check whether a write endpoint consumes the same id without an ownership check — the leak provides the target, the missing check provides the impact.

### 2026-09-23 — IDOR in cashier transaction history exposes another customer's email and phone (Unikrn) — n/a
- Source: [HackerOne #1966006](https://hackerone.com/reports/1966006)
- Type: IDOR / broken object-level authorization
- Summary: The transaction-history flow on the Unikrn cashier host tied returned records to a client-supplied object reference rather than to the authenticated session, so a user could retrieve another customer's cashier data including email address and phone number.
- Technique / pattern: The identifier was carried in the session-handshake request that runs before `cashier/transaction-history`, so the missing ownership check sat in a setup call rather than the final data call. The bug only reproduced mid-handshake, which is why triage first saw it as non-reproducible.
- Takeaway: In a multi-step flow, an identifier passed during an early setup call is as security-sensitive as one in the final data call. Review authorization on every request in a sequence, and document the exact request order when a finding depends on it.

### 2026-09-23 — IDOR for changing privacy settings on TikTok Now (TikTok) — n/a
- Source: [HackerOne #1733627](https://hackerone.com/reports/1733627)
- Type: IDOR / unauthorized write to another user's privacy settings
- Summary: An endpoint in the TikTok Now Android app accepted another user's object reference without an ownership check, letting any user change the "Who Can View" privacy setting on somebody else's Memory.
- Technique / pattern: Proxy the mobile app, capture the settings-update call for your own content, and replay it with another user's content id; privacy toggles are attractive targets because a successful write is verified simply by viewing the victim's content afterwards.
- Takeaway: Privacy and visibility controls are themselves objects needing authorization — an IDOR here does not read data directly, it silently unlocks it for everyone else.

### 2026-09-23 — IDOR on Add services into victims account (Opera Public Bug Bounty) — n/a (P5)
- Source: [Bugcrowd #46a394f4](https://bugcrowd.com/disclosures/46a394f4-4efc-4694-864f-a5d1b74e17fc/idor_4-on-add-services-into-victims-account)
- Type: IDOR / unauthorized write to another account
- Summary: The service-provisioning flow accepted a publisher account reference without checking ownership, letting an authenticated attacker add services into any publisher's account. Disclosed 2021-09-23.
- Technique / pattern: Enumerate the write-side endpoints of a feature, not just the read-side ones, and swap the account reference in the add/create request; the researcher found this as one of a numbered series, which is the sign of sweeping one parameter across a whole feature.
- Takeaway: When one IDOR turns up in an application, test the same identifier across every sibling action — the missing check is usually a pattern across the controller, not a one-off.

### 2026-09-23 — IDOR - remove users from community groups (Atlassian — community.atlassian.com) — n/a (P3, 10 points)
- Source: [Bugcrowd #2fce1eaa](https://bugcrowd.com/disclosures/2fce1eaa-b482-4279-a7c8-8ca89cdad472/idor-remove-users-from-community-groups)
- Type: IDOR / broken function-level authorization
- Summary: Group management on the Atlassian community site did not verify that the caller was entitled to modify the group, so any user could remove any other member from a community group by supplying their identifier. Disclosed 2023-07-21.
- Technique / pattern: Perform the legitimate action on an object you own, capture the removal request, then replay it substituting another member's id; a destructive IDOR is confirmed by the state change rather than by the response body.
- Takeaway: Membership and moderation endpoints deserve the same object-level checks as data reads — a "remove member" call with no authorization check is a denial-of-service primitive against a whole community.

### 2026-09-23 — Privilege Escalation of Publisher Account due to IDOR (Opera Public Bug Bounty) — n/a (P2, 20 points)
- Source: [Bugcrowd #d54dbd31](https://bugcrowd.com/disclosures/d54dbd31-5996-4018-acb6-9792f1c09d67/privilege-escalation-of-publisher-account-due-to-idor)
- Type: IDOR leading to privilege escalation
- Summary: An object reference in the publisher account area was not bound to the caller's permission level, letting a low-privilege user raise their own permissions. Disclosed 2022-03-23.
- Technique / pattern: Compare the requests a high-privilege role issues against what a low-privilege account can reach, then replay the privileged call with the low-privilege session and the attacker's own object id.
- Takeaway: IDOR is not only about reading other people's data — when the referenced object controls roles or entitlements, the same missing check becomes vertical privilege escalation.

### 2026-09-22 — Blind enumeration of private card names via sort oracle and ID discovery (Trello) — 5 points (P4)
- Source: [Bugcrowd #0ecb51a3](https://bugcrowd.com/disclosures/0ecb51a3-2064-4f9d-aa19-aa7b6ae21812/blind-enumeration-of-private-card-names-via-sort-oracle-and-id-discovery)
- Type: IDOR / broken object-level authorization via side channel
- Summary: Cards the tester could not read still took part in the board's "Sort by card name alphabetically" ordering, so their hidden titles could be recovered one character at a time; a second flaw let card IDs be discovered because unauthenticated requests answered "unauthorized" for real cards and "not found" for fake ones.
- Technique / pattern: Insert probe cards with chosen names, trigger the alphabetical sort, and read the probe's resulting position as a comparison oracle (binary search over the hidden title); batch the probes with GraphQL aliasing (~50 renames per request via `updateCardName`) and harvest IDs through the REST `/batch` endpoint, which accepted about 350 sub-requests despite a documented limit of 10 and had no rate limit.
- Takeaway: Anything that *orders, counts or compares* objects you cannot read is a disclosure channel — the authorization check must cover sorting and search, not just direct reads; and documented batch limits mean nothing until they are enforced server-side.

### 2026-09-22 — IDOR on /user-teams-management exposing internal user and school organization IDs (NASA VDP) — n/a (P5)
- Source: [Bugcrowd #871d4c4e](https://bugcrowd.com/disclosures/871d4c4e-ec99-437c-9df3-8f9425a82a44/insecure-direct-object-reference-idor-on-user-teams-management-exposing-internal-user-ids-and-school-organization-ids)
- Type: IDOR / missing ownership check on a management endpoint
- Summary: An authenticated user could change a user identifier parameter on the `/user-teams-management` endpoint and receive limited data belonging to other accounts — internal user IDs, usernames and school organization IDs. The program accepted it as a business risk because the exposed fields were not sensitive enough to rate higher.
- Technique / pattern: Standard two-account replay against an administrative/management route, then a judgement call on impact: the identifiers harvested here are mainly useful as *input* to a further IDOR rather than as a finding on their own.
- Takeaway: A confirmed authorization gap is not automatically a high-severity bug — grade it by what the returned fields let you do next, and say so in the report instead of overclaiming.

### 2026-09-22 — Insecure direct object reference gives access to other users' and groups' DMs (X / Twitter) — n/a (bounty awarded, amount not shown)
- Source: [HackerOne #53858](https://hackerone.com/reports/53858)
- Type: IDOR on a state-changing endpoint (former-member access)
- Summary: On `mobile.twitter.com`, the group-DM deletion route `/a/messages/<dm_id>/delete` did not verify that the requester was still a participant, so someone who had once been in a group conversation could keep reaching its messages after leaving by calling the endpoint with the known DM ID.
- Technique / pattern: Classic *former-privilege* testing — join a shared object, record its ID, leave, then replay the endpoints that still accept that ID. The mobile web surface was checked separately from the main site, and the destructive route (`delete`) carried weaker checks than the read route.
- Takeaway: Authorization must be evaluated at request time against current membership, not at the time access was first granted; and mobile/legacy front-ends need the same object-level checks as the flagship app.

### 2026-09-22 — IDOR on tagged people lets any user be tagged on videos they do not own (TikTok) — n/a (bounty awarded, amount not shown)
- Source: [HackerOne #1555376](https://hackerone.com/reports/1555376)
- Type: IDOR / write-side missing authorization (CVSS 6.5, medium)
- Summary: On `*.tiktokv.com` the tagging feature accepted a target user reference without checking the relationship between the caller, the video and the tagged account, so an attacker could tag arbitrary users on videos that were not theirs.
- Technique / pattern: Look at features that *link two objects* (tag, mention, attach, invite) — the server often authorizes only the caller's right to the action, not their right to each object named in the request. The tagged user's public ID is all the attacker needs.
- Takeaway: Write-side IDORs on social features are reputational attacks: every object referenced in a linking request needs its own ownership/consent check.
### 2026-09-22 — Private circle can be added to another circle via API despite visibility restriction (Nextcloud) — $150
- Source: [HackerOne #3511998](https://hackerone.com/reports/3511998)
- Type: IDOR / missing authorization check in API (Circles app, CVE-2026-45155)
- Summary: In Nextcloud's Circles app, a low-privileged user could add a private circle to another circle through the API, bypassing its visibility restriction; the web UI enforced the check but the API endpoint did not.
- Technique / pattern: Skipped the UI and called the circle-membership API directly, supplying the ID of a private circle the user should not be able to reference; the server accepted the object ID without checking the caller's permissions on it.
- Takeaway: UI-only access rules are not access rules. Replay every UI action as a raw API call and swap in IDs of private/hidden/other users' objects.

### 2026-09-22 — IDOR to make someone attend or leave an event (LinkedIn) — n/a
- Source: [HackerOne #1734639](https://hackerone.com/reports/1734639)
- Type: IDOR on a state-changing API
- Summary: The `voyagerScheduledcontentDashViewerStates` API changed event attendance for whichever user was named in the `fsd_profile` parameter, letting an attacker add users to events or remove real attendees and block them from rejoining.
- Technique / pattern: The victim's `fsd_profile` ID is readable from any public profile's page source; substituting it into the attendance `POST` request worked because the server never tied the ID to the session.
- Takeaway: Client-supplied identifiers on state-changing requests must be validated against the session; publicly visible IDs make such IDORs free to exploit at scale.

### 2026-09-22 — Stored XSS chained with IDOR in LabCAS Saved Searches (NASA VDP) — n/a
- Source: [Bugcrowd #bf88b0d6](https://bugcrowd.com/disclosures/bf88b0d6-61fd-4cc9-8a04-84e36e71c8a6/high-stored-xss-chained-with-idor-at-https-edrn-labcas-jpl-nasa-gov-labcas-ui-s-index-html-search)
- Type: IDOR chained with stored XSS
- Summary: The Saved Searches feature posted to `/ksdb/save_labcas_search_input/` with a client-supplied `userid` that was not checked against the logged-in user, so entries could be written into another account; the `profile_name` field was also rendered unsanitized.
- Technique / pattern: Noticed the owner reference (`userid`) came from the request body rather than the session, allowing writes under another user's identity, and combined it with an unescaped stored field so the planted entry would carry script to the victim.
- Takeaway: Derive ownership for writes from the server-side session, never from a body `userid`; an IDOR that plants content in another account turns self-XSS into stored XSS against the victim.

### 2026-09-22 — Broken access control leads to account takeover (Federal Communications Commission VDP) — n/a
- Source: [Bugcrowd #d66b0aa9](https://bugcrowd.com/disclosures/d66b0aa9-5df4-414a-81bc-21e63bfa8fd7/broken-access-control-leads-to-account-takeover)
- Type: IDOR / broken access control (P1)
- Summary: An IDOR combined with authentication weaknesses exposed other users' confidential data and allowed resetting another account's PIN and password, i.e. full account takeover.
- Technique / pattern: Classic two-account testing: replaying one account's requests with the other account's `<user_id>`/object reference; the account-recovery actions (PIN/password reset) accepted the foreign reference.
- Takeaway: Prioritise IDOR testing on credential-reset and recovery actions: an IDOR there is immediately an account takeover.

### 2026-09-21 — Unauthenticated UUID-based IDOR exposes internal files (NASA VDP)
- Source: [Bugcrowd #085b5b66](https://bugcrowd.com/disclosures/085b5b66-83fb-4580-beb2-5b592c260418/unauthenticated-access-to-internal-files-via-direct-object-reference-uuid-based-leading-to-sensitive-data-exposure)
- Type: IDOR / broken object-level authorization
- Summary: An endpoint served internal files keyed only on a UUID, with no authentication or ownership check, letting anyone who obtained a `uuid` read sensitive internal files.
- Technique / pattern: Directly reference `uuid`-based objects on the endpoint without a session; the server returns the file because it never verifies the caller owns it. UUID unguessability was treated as the only control.
- Takeaway: A UUID is not an authorization boundary — every object-serving route must re-check ownership against the session, not rely on identifier entropy.

### 2026-09-21 — IDOR to view any user's order and profile data via `/api/v1/users/` (WakaTime)
- Source: [HackerOne #2524562](https://hackerone.com/reports/2524562)
- Type: IDOR / broken object-level authorization
- Summary: The `/api/v1/users/current` endpoint accepted any username or user id in place of `current`, returning that user's profile PII (location, socials, timestamps) without an authorization check.
- Technique / pattern: Call `/api/v1/users/current`, then swap `current` for another username or id (e.g. `@victim`); the API returns the victim's full profile. Classic self-vs-other identifier substitution.
- Takeaway: Endpoints that accept a self-referential alias like `current`/`me` must reject arbitrary substituted identifiers — resolve identity from the session, never from the path.

### 2026-09-21 — IDOR allows deleting any user's support tickets on ads.tiktok.com (TikTok)
- Source: [HackerOne #1475520](https://hackerone.com/reports/1475520)
- Type: IDOR / broken object-level authorization (destructive)
- Summary: The ticket-deletion flow authorized on a `draft_order_id` parameter without verifying the requester owned the ticket, allowing deletion of other users' tickets (CVSS 7.5, availability impact).
- Technique / pattern: Manipulate the `draft_order_id` in the delete request to reference another user's ticket; no ownership check runs before the deletion is processed. Destructive verb + attacker-controlled id.
- Takeaway: Destructive actions (DELETE) are where missing ownership checks hurt most — re-derive the target's owner from the session before mutating, especially on `*_id` parameters.

### 2026-09-21 — Critical IDOR — delete any venue of any organization (Veris)
- Source: [HackerOne #120123](https://hackerone.com/reports/120123)
- Type: IDOR / broken object-level authorization (destructive)
- Summary: A DELETE venue request honored a venue id belonging to any organization, letting an authenticated attacker remotely delete venues across organizational boundaries.
- Technique / pattern: Intercept the venue-delete request and change the venue id to a target org's venue; the server deletes it without checking the requester belongs to that organization. Same methodology as sibling member/group IDORs on the app.
- Takeaway: Cross-tenant authZ must prove 'admin of THIS object', not merely 'authenticated somewhere' — a repeated IDOR pattern across sibling endpoints signals a systemic missing check.


### 2026-09-21 — IDOR grants access to private resumes via `/pdf/` path (Indeed) — $250
- Source: [Bugcrowd disclosure ab4df6da](https://bugcrowd.com/disclosures/ab4df6da-de87-46a4-9558-15b59bb40018/idor-access-to-private-resumes)
- Type: IDOR (P3)
- Summary: An endpoint meant for public resume downloads could be reached for private resumes by appending `/pdf/` to the URL and enumerating resume ids, bypassing the visibility control.
- Technique / pattern: Take the public-resume download route, insert the `/pdf/` variant plus a target resume id; the alternate path skipped the private/public check that the primary route enforced.
- Takeaway: Alternate representations of the same object (`/pdf/`, `.json`, print/export views) are a separate code path — the authorization check must live on every one, not just the canonical URL.

### 2026-09-21 — Unauthorized team/org creation via IDOR on `userId` (NASA VDP) — n/a
- Source: [Bugcrowd disclosure d8ed3e0d](https://bugcrowd.com/disclosures/d8ed3e0d-923b-49a8-8a46-767d81fcd5c6/unauthorized-team-and-organization-creation-via-insecure-direct-object-reference-idor)
- Type: IDOR (write-side, P3)
- Summary: The Globe team/organization creation POST did not validate the `userId` in the body against the session user, so teams and orgs could be created under any user's identity (ids were numeric and sequential, e.g. `123231405`).
- Technique / pattern: Capture the create-team/org POST, swap `userId` for another sequential id, and the object is created as that victim user — a write IDOR keyed on a client-supplied identity field.
- Takeaway: Identity for a write must come from the session, never a body parameter; sequential numeric user ids make the abuse trivially scriptable.

### 2026-09-21 — IDOR discloses PII of 3,200+ PROSAMS users (NASA VDP) — n/a
- Source: [Bugcrowd disclosure 68f4566e](https://bugcrowd.com/disclosures/68f4566e-2358-40c2-92d0-3a5e21023908/idor-that-allows-disclosing-username-email-firstname-lastname-address-phonenumbers-of-prosams-application-users)
- Type: IDOR (P1, mass PII)
- Summary: The PROSAMS application honored object identifiers without an ownership check, exposing usernames, emails, names, phone numbers, addresses, firm names and EIN numbers for over 3,200 users across organizations.
- Technique / pattern: Iterate the user/record identifier on the affected PROSAMS endpoint; the app returned full profile records for arbitrary ids with no cross-org authorization.
- Takeaway: A single missing object-level check on a records endpoint escalates to bulk PII disclosure — treat any id-keyed profile route as a mass-extraction risk and rate-limit plus authorize it.

### 2026-09-21 — IDOR in Team Members API exposes emails and roles of any team (NASA VDP) — n/a
- Source: [Bugcrowd disclosure 1e63488a](https://bugcrowd.com/disclosures/1e63488a-ca15-4ed4-99f3-c0298c72a638/idor-in-team-members-api-exposes-private-emails-and-roles-of-any-team)
- Type: IDOR (broken object-level authZ)
- Summary: On globe.gov team management, changing the `orgId` parameter returned member data — emails and role assignments — for teams the caller was not part of.
- Technique / pattern: Enumerate the `orgId` value on the team-members endpoint while authenticated to an unrelated team; the resolver returned other orgs' member lists with no ownership check.
- Takeaway: A read endpoint keyed on an org/team id must verify the session belongs to that org — exposed emails and roles feed straight into targeted phishing.

### 2026-09-21 — Cross-tenant IDOR in GraphQL AddRulesToPixelEvents — add/update/delete any advertiser's pixel rules (TikTok) — bounty
- Source: [HackerOne #984965](https://hackerone.com/reports/984965)
- Type: IDOR (cross-tenant, GraphQL)
- Summary: The ads-portal GraphQL mutation AddRulesToPixelEvents honored a pixel/event id from the request without checking the caller's tenant owned it, so any advertiser could add, modify or delete the pixel-event rules of any other advertiser.
- Technique / pattern: Authenticated as one advertiser, capture the AddRulesToPixelEvents GraphQL call and swap in another tenant's pixel/event object id; the mutation applies cross-tenant with no object-level authorization.
- Takeaway: Cross-tenant authZ is a distinct check from cross-user — every GraphQL mutation that names an object id must verify the caller's tenant owns it, especially in ads/analytics portals.

### 2026-09-21 — IDOR leaks analytics of any restaurant via GraphQL (Uber) — n/a
- Source: [HackerOne #1116387](https://hackerone.com/reports/1116387)
- Type: IDOR (GraphQL, disclosure)
- Summary: restaurant.uber.com's GraphQL analytics service returned sales/performance data for any restaurant id supplied, without verifying the authenticated user managed that restaurant.
- Technique / pattern: Change the restaurant identifier in the analytics GraphQL query while logged in as an unrelated user and read another restaurant's business metrics; the resolver skipped ownership checks.
- Takeaway: Read-only GraphQL resolvers leak just as badly as write ones — every resolver that accepts an entity id needs its own authorization, not just the mutations.

### 2026-09-21 — Any program user can suspend/resume HackerOne Gateway (VPN) via UpdateGatewayProgramStateMutation (HackerOne) — n/a
- Source: [HackerOne #717716](https://hackerone.com/reports/717716)
- Type: IDOR (GraphQL, broken function-level authZ)
- Summary: The UpdateGatewayProgramStateMutation resolved teams through an over-broad 'teams_i_can_see' scope and its interactors skipped authorization, letting any user of a Gateway program toggle vpn_suspended on any team.
- Technique / pattern: Craft a GraphQL POST with a target team's base64-encoded id and the desired vpn_suspended value; the mutation flips VPN state because the scope used for lookup is far wider than the set the user may manage.
- Takeaway: An over-broad lookup scope ('everything I can see') is a common source of IDOR — the query that finds the object must be scoped to what the actor can *manage*, not merely view.

### 2026-09-21 — IDOR on program visibility — reveal/conceal any team member via updateTeamMemberVisibility (HackerOne) — n/a
- Source: [HackerOne #291721](https://hackerone.com/reports/291721)
- Type: IDOR (GraphQL)
- Summary: The /graphql updateTeamMemberVisibility mutation took a base64-encoded team_member_id and changed that member's public security-team affiliation without checking it belonged to the caller.
- Technique / pattern: Decode the base64 team_member_id, substitute a colleague's id, re-encode and replay; the mutation updates any member's visibility with no ownership verification.
- Takeaway: Base64 is encoding, not authorization — decode every opaque id in a mutation and swap it to test object-level access.

### 2026-09-21 — Authentication Bypass / IDOR in subscription-management endpoint exposes billing data (lemlist) — n/a
- Source: [HackerOne #3417162](https://hackerone.com/reports/3417162)
- Type: IDOR (billing/PII disclosure)
- Summary: A subscription-management API endpoint lacked proper access control, so manipulating the customer identifier returned other users' subscription and payment/billing information.
- Technique / pattern: Iterate/swap the customer id parameter on the subscription endpoint; the response returns another account's billing details without valid ownership of that customer record.
- Takeaway: Billing and subscription endpoints are prime IDOR targets — the customer id in the request must be bound to the authenticated session, not trusted as-is.


### 2026-09-21 — IDOR lets a removed member delete messages in a group using the message ID (Rocket.Chat) — n/a
- Source: [HackerOne #2028450](https://hackerone.com/reports/2028450)
- Type: IDOR - stale-membership authorization bypass (delete)
- Summary: A user who had left, been removed, or been banned from a Rocket.Chat channel could still delete their old messages there by sending the message ID to /api/v1/method.call/deleteMessage, which the UI would otherwise block. Rated low; no bounty.
- Technique / pattern: Note a message ID, leave the channel, capture a legitimate delete request from a different channel, swap in the old message ID and replay it - the server deleted it without rechecking current membership.
- Takeaway: Authorization must be judged on the user's current relationship to the parent resource (membership, ban, mute), not only on who created the object; UI-only blocks fall to replayed generic RPC/method-call endpoints.

### 2026-09-21 — IDOR in external status check API leaks data about any status check on the instance (GitLab) — n/a
- Source: [HackerOne #1372216](https://hackerone.com/reports/1372216)
- Type: IDOR - unscoped child ID (cross-project information disclosure)
- Summary: `POST /projects/<id>/merge_requests/<iid>/status_check_responses` did not verify that external_status_check_id belonged to the project in the URL, so changing that ID returned status-check details from any project, including private ones (names, IDs, service URLs, protected branches). Rated medium (CVSS 4.3); bountied, amount not public.
- Technique / pattern: Call the endpoint on a merge request in a controlled project so the project-level check passes, then enumerate sequential external_status_check_id values in the body to pull other projects' objects.
- Takeaway: Authorizing the parent resource in the URL is not enough - every child or foreign-key ID in the body must be confirmed to belong to that parent. Hunt for "authorized container, unscoped child ID".

### 2026-09-21 — IDOR allows an attacker to modify the links of any user (Reddit) — n/a
- Source: [HackerOne #1661113](https://hackerone.com/reports/1661113)
- Type: IDOR - GraphQL mutation writes to another user's objects
- Summary: Reddit's GraphQL API let an attacker edit the social links on any user's profile because the update accepted a link ID without checking the requester owned it. Rated high; bountied but amount not public.
- Technique / pattern: Query the API with a victim's username to obtain that user's social-link IDs, then send the link-update mutation with those IDs plus an attacker-chosen URL/title.
- Takeaway: GraphQL queries often leak other users' internal object IDs that then feed into mutations - enforce ownership on every mutation argument, not just the top-level query.

### 2026-09-21 — Unauthenticated IDOR allows modification of payment customer billing information (Weblate) — n/a
- Source: [HackerOne #3869124](https://hackerone.com/reports/3869124)
- Type: IDOR - unauthenticated write to a UUID-keyed resource
- Summary: Weblate's payment edit endpoint /en/payment/{UUID}/edit/ performed no authentication or ownership check, so anyone holding a payment UUID could change and save the billing details (name, address) tied to that payment. Disclosed 2026-08-30, rated high; program pays no bounties.
- Technique / pattern: Create a payment object, note its UUID, open the edit URL while logged out, submit changed fields, confirm the save persisted.
- Takeaway: A UUID is only an identifier. Every read and write handler, including legacy edit views, needs both an auth check and an ownership check; always retest state-changing endpoints with no session at all.

### 2026-09-21 — [IDOR] Improper Access Control on Embedded Submission Form (HackerOne) — $2,500
- Source: [HackerOne #2483666](https://hackerone.com/reports/2483666)
- Type: IDOR - UUID-as-secret in GraphQL (private/inactive resources)
- Summary: GraphQL queries for embedded submission forms returned sensitive program data (intro text, structured scopes, response-efficiency stats) for private or inactive forms to anyone who supplied the form's UUID.
- Technique / pattern: The researcher harvested historical form UUIDs from archived URLs (waybackurls) for forms that had once been public and were later made private, then queried GraphQL directly with those IDs - the server never re-checked authorization after the resource's visibility changed.
- Takeaway: A UUID is an identifier, not access control. When a resource flips from public to private, old IDs leaked in archives, JS or links still work unless the server re-authorizes every request.

### 2026-09-21 — IDOR vulnerability in unreleased HackerOne Copilot feature (HackerOne) — n/a
- Source: [HackerOne #2218334](https://hackerone.com/reports/2218334)
- Type: IDOR - write/delete via GraphQL mutation (unreleased feature)
- Summary: The DestroyLlmConversation GraphQL mutation lacked an ownership check, so any logged-in user could delete another user's Copilot conversation by passing its llm_conversation_id.
- Technique / pattern: Found by monitoring the platform's JavaScript bundles for operations belonging to not-yet-released features, then replaying the mutation from a second account against the first account's conversation ID.
- Takeaway: Unreleased features shipped in client JS are often reachable before their authorization is finished - enumerate every mutation in the bundle and test each with two accounts.

### 2026-09-21 — IDOR in API applications (able to see any API token, leads to account takeover) (Automattic (Pressable)) — n/a
- Source: [HackerOne #1695454](https://hackerone.com/reports/1695454)
- Type: IDOR - sequential ID, secret leaked in error response -> ATO
- Summary: POSTing to /api/applications with only a changed application[id] (plus the CSRF token) caused the response to render the target application's Client ID and Client Secret, giving full API access including collaborator management and thus account takeover.
- Technique / pattern: While updating their own app the researcher noticed application[id] in the request body, stripped the other parameters so validation failed, and swapped the ID; the error page echoed the full object. Sequential IDs made every application enumerable.
- Takeaway: Look at error and validation responses, not just success paths - minimal-body requests can make the server render a whole object it never authorized you to see.

### 2026-09-21 — Insecure Direct Object Reference (IDOR) Vulnerability in Autodesk User Profile (Autodesk) — n/a
- Source: [HackerOne #2965357](https://hackerone.com/reports/2965357)
- Type: IDOR - write (profile modification by id parameter)
- Summary: Changing the id parameter in a profile-update request allowed an attacker to modify another Autodesk user's profile; the report was resolved and disclosed in February 2025.
- Technique / pattern: Classic parameter tampering: capture your own profile-update request, replace the user identifier with a victim's, and confirm the change lands on the victim account.
- Takeaway: Write endpoints deserve the same object-level checks as reads - bind the target object to the authenticated session server-side instead of trusting a client-supplied id.

### 2026-09-20 — IDOR in the autotranslate.translateMessage endpoint leaks message content from any room (Rocket.Chat) — n/a
- Source: [HackerOne #3713682](https://hackerone.com/reports/3713682)
- Type: IDOR (auxiliary endpoint missing the room-access check)
- Summary: `/api/v1/autotranslate.translateMessage` fetched the target message with `Messages.findOneById(messageId)` and never called the room-access helper `canAccessRoomIdAsync`, so any authenticated user who supplied a message id could read content from private groups, DMs and channels they had no membership in.
- Technique / pattern: Utility endpoints built *around* a resource — translate, preview, summarise, export, render, share-card — are written after the primary read path and routinely skip its authorization helper. On an open-source target, learn the canonical access check by name and grep for every handler that loads the same model without calling it; on a closed target, enumerate the secondary actions the UI offers on an object and replay each one with another room's message id.
- Takeaway: Authorization belongs to the object load, not to one route. Where a helper like `canAccessRoomIdAsync` exists, every handler touching that model must go through it.

### 2026-09-20 — IDOR in the GraphQL deleteProfileImages mutation deletes other users' photos (Autodesk) — n/a
- Source: [HackerOne #2968039](https://hackerone.com/reports/2968039)
- Type: IDOR (destructive GraphQL mutation keyed on a supplied id)
- Summary: The `deleteProfileImages` mutation took an `id` parameter naming whose profile image to remove and did not verify the id belonged to the caller, letting an attacker delete other users' profile photos.
- Technique / pattern: Profile and avatar surfaces are treated as cosmetic and reviewed lightly, yet they expose full CRUD. Introspect the schema for mutations whose name contains delete/remove/reset and whose argument is a bare `id`, then run the two-account test: perform the mutation as A with A's id to learn the shape, then send B's id from A's session. A 200 with no error and the image gone from B's profile is the whole proof.
- Takeaway: A destructive mutation must re-derive the subject from the session, or authorize the supplied id against it. Do not treat low-value objects as low-risk routes.

### 2026-09-20 — Unauthorized reservation cancellation by supplying a reservation id (Yelp) — n/a
- Source: [HackerOne #2944357](https://hackerone.com/reports/2944357)
- Type: IDOR (guest flow authorized purely by knowledge of a reference id)
- Summary: Yelp's reservation feature did not require the user to be logged in, and a reservation could be cancelled by anyone who knew its reservation id — so harvesting or guessing ids allowed cancelling other people's bookings. Yelp stated it was already aware of the issue.
- Technique / pattern: Deliberately unauthenticated flows (guest checkout, reservation management, parcel tracking, "view your quote") replace a session with a reference number, which makes the reference number the credential. Check its entropy and whether it is exposed in confirmation emails, QR codes, printable pages or Referer headers, and check whether a *destructive* verb is reachable with it and nothing else. Read access to a guest booking may be an accepted design trade-off; cancellation almost never is.
- Takeaway: When a flow intentionally has no login, the reference id must be high-entropy and paired with a second factor (email or phone) before any state-changing action.

### 2026-09-20 — IDOR in the FetchMemberships GraphQL operation leaks current team data to former members (Tools for Humanity) — $500
- Source: [HackerOne #2381816](https://hackerone.com/reports/2381816)
- Type: IDOR (authorization evaluated at grant time, not request time)
- Summary: The `FetchMemberships` operation on `POST /api/v1/graphql` did not re-check whether the caller still belonged to the organization, so a person who had been removed from a team could keep reading the names, email addresses, roles and ids of the current members.
- Technique / pattern: Test the *former*-privilege state as a first-class case: join or be granted access, record the exact request the UI makes, have the access revoked, then replay the unchanged request. Membership-listing operations are the highest-value target for this because they return the whole roster in one call, and because offboarding is usually implemented as a UI change plus a row deletion rather than as session invalidation.
- Takeaway: Offboarding must revoke sessions and tokens as well as the membership row, and each request must re-evaluate membership rather than trusting a claim baked in at login.

### 2026-09-19 — IDOR — scheduled-job data of other users' projects via GetNotebookScheduledPaginatedJobs (SingleStore) — n/a
- Source: [HackerOne #3219944](https://hackerone.com/reports/3219944)
- Type: IDOR (broken object-level authorization on a list endpoint)
- Summary: The backend endpoint `GetNotebookScheduledPaginatedJobs` returned scheduled-job information for any project whose `projectID` was supplied. The API never verified that the requesting user had permission on that project, so an authenticated user could read other tenants' scheduled job metadata.
- Technique / pattern: Paginated *list* endpoints are as much an object-level authZ surface as single-object reads: they take a container id (projectID, workspaceId, orgId) and return everything inside it. Capture the call the UI makes for your own project, swap in another project id, and diff. Notebook/scheduling/automation panes are newer surfaces than the core product and frequently authorize the notebook but not the project wrapper around it.
- Takeaway: Authorize the container named in a list query, not just the rows it returns. A pagination endpoint that trusts a supplied `projectID` leaks the whole project.

### 2026-09-19 — IDOR leads to PII leak via numeric user id in the update-profile endpoint (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #2586584](https://hackerone.com/reports/2586584)
- Type: IDOR (sequential numeric id, write-side route used for read)
- Summary: A DoD application's update-profile URL carried a numeric user id. Changing that id returned and operated on other users' profile records, exposing PII such as email addresses across accounts.
- Technique / pattern: The oldest pattern still works on legacy enterprise stacks: find any route where a small integer identifies a person, then walk it. Test the *update* route even when you only want to read — profile-edit pages usually pre-populate the form with the target record, so a GET to the edit URL is a read primitive and the POST is a write primitive, both behind the same missing check.
- Takeaway: Sequential integer user ids plus an edit route is the canonical IDOR. Derive the subject of a profile operation from the session, never from the URL.

### 2026-09-19 — IDOR in a legacy Download.aspx handler leaks user attachments (tax, contract and PII documents) (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1626508](https://hackerone.com/reports/1626508)
- Type: IDOR (file-download handler, direct id reference)
- Summary: An API endpoint backed by a classic `Download.aspx` file handler served attachments by id with no ownership check. The reporter showed it returned other users' documents — bank tax forms, contracts, full addresses, phone numbers and emails.
- Technique / pattern: Legacy server-rendered download handlers (`Download.aspx?id=`, `GetFile.ashx`, `download.jsp?docId=`, `/servlet/FileDownload`) predate the app's modern API authorization layer and are usually wired straight to a storage table. Fingerprint the stack, then grep the HTML and JS for those handler paths rather than only the JSON API; they are rarely covered by the middleware that protects `/api/*`.
- Takeaway: Every path that serves bytes is an object store with its own authorization requirement. Inventory the legacy file handlers a modern front end still links to.

### 2026-09-19 — IDOR and statistics leakage in Orders (X / xAI (MoPub)) — n/a
- Source: [HackerOne #544329](https://hackerone.com/reports/544329)
- Type: IDOR (analytics/stats query endpoint)
- Summary: MoPub's `/web-client/api/orders/stats/query` endpoint accepted order identifiers belonging to other advertisers and returned their private order statistics, disclosing campaign data across accounts.
- Technique / pattern: Analytics and reporting endpoints take a *query object* rather than a path id, which hides the identifier from casual review — it sits in a JSON body as `orderId`, `campaignIds`, `filters`. Record a stats request for your own object, then substitute another tenant's id anywhere it appears in the body. Advertising, billing and BI panes are built by different teams from the CRUD API and routinely re-implement (or omit) the ownership predicate.
- Takeaway: A stats query is a read of the underlying object. Authorize every identifier inside a reporting request body, not just the ones in the URL.

### 2026-09-19 — TaskProcessing callback authorization bypass lets ex-members post as the Assistant Talk Bot (Nextcloud) — n/a
- Source: [HackerOne #3799010](https://hackerone.com/reports/3799010)
- Type: IDOR (broken object-level authorization on an asynchronous callback)
- Summary: An authenticated user could schedule a text-processing task whose callback targeted the Assistant Talk Bot in a conversation they no longer belonged to. The bot posted the result without checking that the scheduling user still had access to the target room, so ex-members could inject messages under the bot's trusted identity.
- Technique / pattern: Look for features that accept a target object id now and act on it later — scheduled jobs, webhooks, callbacks, queued exports, "notify me when done". Schedule the job while you legitimately hold access, then leave the room/project/org before it fires. Authorization is usually evaluated once at scheduling time (or not at all) and never re-checked at execution, and the deferred action runs with a service/bot identity that other members implicitly trust.
- Takeaway: Authorize deferred work at execution time against the requester's *current* access, not at the moment it was queued. A bot or service account acting on a user's behalf must inherit that user's live permissions, not the platform's.

### 2026-09-19 — Access to deactivated LinkedIn company pages via the Competitor Analytics API (LinkedIn) — bounty awarded (amount undisclosed)
- Source: [HackerOne #3604288](https://hackerone.com/reports/3604288)
- Type: IDOR (object lifecycle state not enforced on a secondary API)
- Summary: LinkedIn's Competitor Analytics API returned analytics data for company pages that had been deactivated and should no longer have been reachable. Any authenticated user could request a deactivated page's identifier and receive its data (CVSS 5.3, medium).
- Technique / pattern: Enumerate the lifecycle states an object can hold — active, deactivated, archived, suspended, soft-deleted, expired — and re-request it in each state through *every* endpoint that touches it, not just the primary one. Comparison, analytics, export, search-index and recommendation APIs are built on a different code path from the main read route and routinely miss the "is this object still live" predicate that the primary route enforces.
- Takeaway: Deactivation is an authorization state, and it has to be checked wherever the object is served. Secondary and analytics APIs are a separate enforcement surface — audit them against the main route's checks.

### 2026-09-19 — Missing access control in MigrationFile lets an attacker upload files to any migration (GitHub) — n/a
- Source: [HackerOne #3506183](https://hackerone.com/reports/3506183)
- Type: IDOR (write-side, CVE-2026-1355, GitHub Enterprise Server)
- Summary: GitHub Enterprise Server's repository-migration upload endpoint did not verify that the caller owned the migration identified in the request. An authenticated user could supply another user's migration id and upload content into their export, replacing the archive the victim would later download and restore.
- Technique / pattern: Most IDOR hunting tests reads; this is the write mirror. Find endpoints that *deposit* data into a container identified in the request — migration/export archives, backup slots, import staging areas, batch-upload sessions, avatar/asset slots — and replay them with another account's container id. The impact is not disclosure but content poisoning: the victim later consumes the artifact as trusted, so a missing check upgrades to supply-chain-style code or data injection at restore time.
- Takeaway: Authorize writes to a container as strictly as reads from it. Any artifact a user will later download and trust is an integrity boundary, and unowned writes into it are high severity even when nothing leaks.

### 2026-09-19 — Unauthenticated access to private files on app.fizzy.do via Active Storage URLs (Basecamp) — $100
- Source: [HackerOne #3467641](https://hackerone.com/reports/3467641)
- Type: IDOR (unauthenticated object access via framework blob URLs)
- Summary: Files uploaded to Basecamp's Fizzy application were served through Rails Active Storage URLs that remained fetchable without a session. The researcher uploaded a file while authenticated, took the blob URL out of the page markup, and retrieved it from a logged-out browser and from a different account; PDFs, images, text and HTML all loaded.
- Technique / pattern: Framework blob/attachment URLs (Rails Active Storage, Django storages, S3 pre-signed links, `/rails/active_storage/blobs/...`) are *bearer* URLs by default — possession of the URL is the authorization. Harvest them from page source, API responses and embeds, then replay in a clean unauthenticated session. No guessing is needed, which is exactly why the class is easy to miss: the id looks unguessable, so nobody adds a check.
- Takeaway: An unguessable URL is not an access control. Route private attachments through an authorizing controller, or use short-lived signed URLs scoped to the requesting session, and re-test the raw storage path separately from the API.

### 2026-09-18 — IDOR in the banner deletion endpoint lets one Manager delete another Manager's banners (Revive Adserver) — n/a
- Source: [HackerOne #3401612](https://hackerone.com/reports/3401612)
- Type: IDOR (broken object-level authorization, child object)
- Summary: Revive Adserver's banner-deletion feature authorized the request against the parent campaign but never checked ownership of the specific banner being removed. A Manager-level user could therefore delete banners belonging to another Manager, sabotaging their campaigns.
- Technique / pattern: Two-account testing against a parent/child object model: create objects as Manager A, then as Manager B issue the delete request with A's child (banner) id while keeping a campaign id B legitimately owns. A success response shows the check was performed one level up the hierarchy and never re-applied to the leaf object actually being mutated.
- Takeaway: Authorize the exact object the request mutates. A valid check on the parent (campaign, folder, project) says nothing about whether the caller owns the child.

### 2026-09-18 — Cross-tenant IDOR in GraphQL allows adding a co-founder to another merchant's Atlas application (Stripe) — n/a
- Source: [HackerOne #1066203](https://hackerone.com/reports/1066203)
- Type: IDOR (cross-tenant, GraphQL mutation)
- Summary: A GraphQL mutation on Stripe Atlas allowed someone who had previously held admin rights on a merchant account to add a co-founder to that merchant's Atlas application. The mutation resolved the target application from the supplied identifier without re-confirming the caller's current authorization over it.
- Technique / pattern: Record the privileged GraphQL mutation while holding admin on a tenant, then replay it after the privilege should have lapsed (or from a different tenant context), keeping the original object identifier. Testing the 'former admin' state - not just 'never an admin' - is what exposes checks that were evaluated once at grant time rather than at request time.
- Takeaway: Re-evaluate authorization on every request, for the current session, against the current object. Revoked or historical privilege must not keep a mutation reachable.

### 2026-09-18 — IDOR in saved-projects deletion lets any user delete another user's saved projects (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #800608](https://hackerone.com/reports/800608)
- Type: IDOR (destructive action via GET, sequential id)
- Summary: The saved-searches / saved-projects area exposed a deletion route that took the record id from the URL. Changing the id in a GET request removed saved projects belonging to other users, because the handler never checked that the record was owned by the requesting session.
- Technique / pattern: Enumerate the ids appearing in a user's own 'saved items' view, then replay the delete URL from a second account with an id that account never created. A state-changing action reachable over GET is doubly weak: it is trivially replayable and also CSRF-able from a third-party page.
- Takeaway: Destructive routes must re-derive the owner from the session and should not be reachable via GET; 'saved items' collections are a routine source of sequential, unauthorized-deletable ids.

### 2026-09-18 — IDOR lets a basic organization member view and delete API keys (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #1628012](https://hackerone.com/reports/1628012)
- Type: IDOR (role/permission bypass on credential management)
- Summary: A user holding only basic member permissions inside an organization could create, view and delete API keys - operations that should have been restricted to administrators. The API-key endpoints enforced organization membership but not the role required for key management.
- Technique / pattern: Create an organization, invite a low-privilege second account, and from that account call every administrative endpoint the UI hides rather than only the ones it shows. Credential-management routes (API keys, tokens, webhooks, integrations) are a high-value place to check, because the UI usually hides them and the server often only checks membership.
- Takeaway: Membership is not a role. Credential-management endpoints need an explicit role check, and exposure of existing key material makes the finding an escalation path, not just a permissions nit.

### 2026-09-18 — IDOR on a Nextcloud instance via direct photo URL exposes other users' and deleted photos (Nextcloud) — n/a
- Source: [HackerOne #3518758](https://hackerone.com/reports/3518758)
- Type: IDOR (broken object-level authorization, WebDAV/Photos)
- Summary: A photo uploaded to a Nextcloud album kept a directly addressable DAV URL under `/remote.php/dav/photos/<user>/albums/<album>/<photo>`. A second, unrelated account could fetch that URL and retrieve the image - including after the owner had deleted it - because the endpoint never checked that the requesting session owned the object.
- Technique / pattern: Two-account differential testing on a file-serving path: record the canonical asset URL as account A, delete or unshare the object, then replay the exact URL from account B. A 200 with the image body - rather than a 403/404 - shows the DAV layer is resolving the path without consulting the sharing/ownership model, and that deletion only unlinked the object from the album view.
- Takeaway: Static asset and WebDAV paths are objects too: authorization must be enforced at the file-serving layer, and 'deleted' must mean unreachable, not merely unlisted.

### 2026-09-18 — Broken access control lets advertiser accounts delete trackers they do not own (CVE-2026-21641) (Revive Adserver) — n/a
- Source: [HackerOne #3445710](https://hackerone.com/reports/3445710)
- Type: IDOR (missing ownership check on destructive action)
- Summary: Revive Adserver's tracker-delete.php accepted a clientid parameter and deleted the referenced tracker without verifying that the calling advertiser account owned it, so any user holding the tracker-delete permission could destroy other tenants' trackers.
- Technique / pattern: Enumerate the actions a low-privileged role is legitimately allowed to perform, then re-issue each one against an identifier belonging to a different tenant. Destructive endpoints are frequently the weakest link because developers test them only from the owning account; a permission check ('may this role delete trackers?') is often mistaken for an ownership check ('may this role delete THIS tracker?').
- Takeaway: Role permission and object ownership are two separate gates. Verifying only the first turns every privileged role into a cross-tenant one.

### 2026-09-18 — IDOR: authorization bypass in LockReport GraphQL mutation for public reports (HackerOne) — n/a
- Source: [HackerOne #2139190](https://hackerone.com/reports/2139190)
- Type: IDOR (GraphQL mutation, missing authorization)
- Summary: The LockReport GraphQL mutation accepted any public report's ID and locked it without checking that the caller had rights over that report, letting an outsider shut down comment threads on disclosed reports.
- Technique / pattern: Harvest the mutation names and argument shapes from the GraphQL schema, then call state-changing mutations with an object ID the tester does not own, from a low-privileged or unauthenticated session. GraphQL centralises resolvers, so an authorization check omitted in one resolver is invisible from the UI, which simply never renders the button.
- Takeaway: Every GraphQL resolver needs its own authorization check - the absence of a UI affordance is not an access control.

### 2026-09-18 — Improper access control to messages of the Social app (CVE-2020-8278) (Nextcloud) — n/a
- Source: [HackerOne #921717](https://hackerone.com/reports/921717)
- Type: IDOR (unauthenticated object access via guessable token)
- Summary: The Nextcloud Social app's displayPost handler served /@{username}/{token} without any authentication or authorization check, so an unauthenticated visitor holding or guessing a post token could read any message, including private direct messages.
- Technique / pattern: Read the controller source for the feature under test and grep for absent auth guards and TODO comments - here the ActivityPubController carried a TODO where the access check belonged. Then assess the identifier's entropy: the token derived from a Unix timestamp, narrowing the guess space enough to make enumeration practical.
- Takeaway: An unguessable identifier is not an access control, and a low-entropy one is not even unguessable. Code comments admitting missing checks are a high-yield grep in open-source targets.

### 2026-09-18 — Broken Access Control leads to unauthorized transaction history access (MTN Group) — n/a
- Source: [HackerOne #2746709](https://hackerone.com/reports/2746709)
- Type: IDOR (broken object-level authorization, mobile API)
- Summary: An API endpoint used by the MyMTN NG mobile app applied an insufficient authorization check, so a caller could retrieve another subscriber's recharge/transaction history — recharge dates, balances before and after the transaction, and transaction identifiers.
- Technique / pattern: Proxy the mobile app, inventory the account-scoped endpoints it calls, then replay each one from a second account with the victim's subscriber identifier substituted; a 200 carrying the victim's financial records confirms the endpoint trusts the client-supplied identity.
- Takeaway: Mobile back-ends are the same web API with a different client: an endpoint that is only ever called by the app still needs a server-side ownership check, because the app is not the only thing that can call it.

### 2026-09-18 — Broken Access Control exposes email metadata via the public user API (WakaTime) — n/a
- Source: [HackerOne #3114132](https://hackerone.com/reports/3114132)
- Type: IDOR / excessive data exposure (REST user endpoint)
- Summary: The `/api/v1/users/{username}` endpoint returned email-related metadata such as `is_email_confirmed` and `is_email_public` without an authorization check, exposing per-account attributes the owner had chosen to keep private.
- Technique / pattern: Request a public profile endpoint for an account you do not own and diff the JSON against what the rendered UI shows; fields present in the response but absent from the page are the ones the serializer forgot to gate.
- Takeaway: Authorization failures are not only about whole objects — a single extra field in a serializer leaks state the user deliberately kept private, and boolean metadata is enough to profile and target accounts.

### 2026-09-18 — Access control flaw allowed closing a report as a duplicate of a report in another program (HackerOne) — n/a
- Source: [HackerOne #2516250](https://hackerone.com/reports/2516250)
- Type: IDOR (cross-tenant object reference in a state-changing action)
- Summary: The duplicate-closing flow checked only `can?(:view, original_report)` before accepting the referenced original, so a user could mark a report as a duplicate of a report belonging to a different program or organization.
- Technique / pattern: Find an action that accepts a reference to a second object, then supply an object from another tenant that you can legitimately *see* (a disclosed or shared item) rather than one you own; if the handler only tests visibility, the cross-tenant link is accepted.
- Takeaway: `can view` is not `can use`: an action that links two objects has to authorize the relationship — same tenant, same program, right role — and not merely the reader's visibility of the target.

### 2026-09-18 — IDOR to view other users' order information and IP addresses (BOHEMIA INTERACTIVE a.s.) — n/a
- Source: [HackerOne #287789](https://hackerone.com/reports/287789)
- Type: IDOR (sequential order identifier)
- Summary: An order-details view honored an attacker-supplied order reference without checking that the session owned the order, exposing other customers' order information along with details such as their IP addresses.
- Technique / pattern: Place a real order, note the identifier's format, then walk neighbouring values; order and invoice views are a classic spot because the identifier is short-lived in the UI but permanent server-side.
- Takeaway: Purchase and billing records carry the richest PII in most applications, so they deserve the strictest ownership check — and identifiers that look like internal bookkeeping are exactly the ones developers forget to scope.

### 2026-09-18 — BOLA/IDOR in Out-of-Office API allows any authenticated user to read other users' absence data (Nextcloud) — n/a
- Source: [HackerOne #3382343](https://hackerone.com/reports/3382343)
- Type: IDOR (broken object-level authorization, REST path parameter)
- Summary: The Out-of-Office endpoints `/ocs/v2.php/apps/dav/api/v1/outOfOffice/{userId}` and `.../{userId}/now` honored the `userId` path parameter without validating that the session owned that account, so any authenticated user could read any other user's absence data.
- Technique / pattern: Enumerate the OCS/REST routes an app registers, spot the ones carrying a user identifier in the path, then replay the request from a second low-privileged account with the victim's `userId` substituted — a 200 carrying the victim's data confirms the missing ownership check.
- Takeaway: A user identifier in the path is never authorization: every route accepting one must re-derive the subject from the session or check ownership explicitly, and freshly added app APIs are where the platform's usual check is most often skipped.

### 2026-09-18 — An attacker can submit arbitrary work project requests and access other users' project details (LinkedIn) — n/a
- Source: [HackerOne #2291999](https://hackerone.com/reports/2291999)
- Type: IDOR (broken object-level authorization in a two-sided marketplace flow)
- Summary: LinkedIn's Request Services flow let an attacker submit work-project requests referencing projects belonging to other users, exposing project details that had been sent confidentially to competing service providers.
- Technique / pattern: In two-sided marketplace flows, invoke the request/apply/bid action with an object id belonging to another participant and diff the response for counterparty-only fields — the leaked data is often the business value of the platform itself.
- Takeaway: Multi-party flows add trust boundaries beyond 'is this user logged in': the check must be 'may this actor see this counterparty's object', and leaked bid/project data is a business-logic impact, not merely an info leak.

### 2026-09-18 — IDOR on the DELETE /comments/ endpoint (RGhost) — n/a
- Source: [HackerOne #861849](https://hackerone.com/reports/861849)
- Type: IDOR (destructive verb, sequential integer identifiers)
- Summary: The comment delete/edit endpoint acted on a sequential numeric comment id without verifying the requester authored the comment, and returned the affected comment's content in the response.
- Technique / pattern: Note the sequential id space, script a range of ids against the destructive route from a second account, and use the comment body echoed back in the response as the oracle that confirms unauthorized access.
- Takeaway: Sequential ids + a destructive verb + a response that echoes the object is the classic IDOR trifecta — authorize DELETE/PATCH as rigorously as GET, and do not return the object you just acted on.

### 2026-09-18 — IDOR in 3d.cs.money — save, edit and delete another user's builds (CS Money) — n/a
- Source: [HackerOne #990878](https://hackerone.com/reports/990878)
- Type: IDOR (public third-party identifier used as the authorization key)
- Summary: The 3D builder keyed operations on the user's Steam ID — a publicly discoverable identifier — so supplying a victim's Steam ID allowed saving, editing and deleting that victim's builds without any grant on the account.
- Technique / pattern: Identify which identifier the application trusts, confirm it is publicly obtainable (here via Steam ID lookup services), then reproduce with two accounts — attacker acting on the victim's id — to prove writes and deletes land on the victim's objects.
- Takeaway: An identifier sourced from a third-party platform is public input, never proof of identity; bind every action to the server-side session rather than to an id the requester supplies.

### 2026-09-17 — IDOR - Delete all Licenses and certifications from users account using CreateOrUpdateHackerCertification GraphQL query (HackerOne) — $12,500
- Source: [HackerOne #2122671](https://hackerone.com/reports/2122671)
- Type: IDOR (broken object-level authorization, GraphQL mutation)
- Summary: A GraphQL mutation used to create/update hacker certifications acted on a supplied object identifier without verifying the session owned that record, allowing an attacker to wipe any user's licenses and certifications.
- Technique / pattern: Enumerate GraphQL mutations (introspection or captured client traffic), then replay a write/destroy mutation with an object id belonging to a second account and diff the result — destructive mutations first, since impact is immediate.
- Takeaway: GraphQL mutations need the same object-level authorization as REST routes; a mutation that can destroy data is the highest-value place to test for a missing ownership check.

### 2026-09-17 — IDOR: Account Deletion via Session Misbinding – Attacker Can Delete Victim Account (Mozilla) — n/a
- Source: [HackerOne #3154983](https://hackerone.com/reports/3154983)
- Type: IDOR (session/object misbinding on a destructive action)
- Summary: The account-deletion flow acted on an identifier taken from the request instead of the identity bound to the authenticated session, so one user could delete another user's account.
- Technique / pattern: For every destructive action, compare the subject identifier in the request against the session's own identity, then substitute a second account's value to see whether the server re-derives the subject or simply trusts the parameter.
- Takeaway: The subject of a destructive action must be derived server-side from the session; any user-controllable id in a delete flow is an authorization bug waiting to be found.

### 2026-09-17 — Cross-repository IDOR in /settings/security_analysis/bypass_reviewers allows unauthorized delegated bypass reviewer modification (GitHub) — n/a
- Source: [HackerOne #3560256](https://hackerone.com/reports/3560256)
- Type: IDOR (cross-tenant / cross-repository authorization)
- Summary: A repository security-settings endpoint accepted identifiers belonging to a different repository, letting a user change delegated bypass reviewers on repositories they did not administer.
- Technique / pattern: Hold admin rights on one tenant (repo/org/shop), capture its privileged settings request, then swap in identifiers owned by a second tenant — this tests whether the check is scoped to the object or merely to 'is an admin somewhere'.
- Takeaway: Test authorization across tenants, not just authenticated vs anonymous: 'admin of some repository' must never be allowed to imply 'admin of this repository'.

### 2026-09-17 — IDOR Exposes All Machine Learning Models (GitLab) — $1,160
- Source: [HackerOne #2528293](https://hackerone.com/reports/2528293)
- Type: IDOR (broken object-level authorization)
- Summary: An endpoint in GitLab's machine-learning model registry returned model records without checking project membership, exposing models belonging to projects the requester had no access to.
- Technique / pattern: Prioritize recently shipped feature areas, enumerate their sequential object ids from an unrelated account, and confirm whether records from other projects come back.
- Takeaway: New features often lag behind a platform's mature authorization conventions — recently added endpoints are the highest-yield place to hunt for IDOR.

### 2026-09-17 — IDOR on in-app hardcoded zombie endpoint (Bykea) — n/a
- Source: [HackerOne #3085742](https://hackerone.com/reports/3085742)
- Type: IDOR (broken object-level authorization)
- Summary: A deprecated ("zombie") API endpoint hardcoded in the Bykea Android app was still live and honored trip/driver ids without an ownership check, exposing driver details tied to other users' trips.
- Technique / pattern: Reverse-engineered the Android app and reviewed its code to locate unused/legacy endpoints, then replayed requests with other ids to read data belonging to other users.
- Takeaway: Decommissioned-but-reachable endpoints are prime IDOR targets — unpack mobile apps to find "zombie" routes, and truly retire old APIs (not just hide them) with server-side authorization.

### 2026-09-17 — IDOR leads to view other users' demographic info (U.S. Dept Of Defense) — n/a
- Source: [HackerOne #2586641](https://hackerone.com/reports/2586641)
- Type: IDOR (broken object-level authorization)
- Summary: Several endpoints under the shared base path /JOINOnline/Board/QuestionCard/ returned other users' demographic/PII when the object id was changed, with no check that the requester owned the record.
- Technique / pattern: Enumerated the id across a family of endpoints sharing one base path, swapping ids to retrieve records belonging to other accounts.
- Takeaway: When one endpoint is IDOR-vulnerable, test its whole sibling family — related routes under the same base path usually share the same missing authorization check.

### 2026-09-17 — IDOR on GraphQL BillingDocumentDownload / BillDetails exposes other shops' billing data (Shopify) — n/a
- Source: [hackerone #2207248](https://hackerone.com/reports/2207248)
- Type: IDOR (Insecure Direct Object Reference)
- Summary: The GraphQL `BillingDocumentDownload` and `BillDetails` queries accepted a `BillingInvoice` object id that was not scoped to the caller's shop, letting a staff user read another shop's billing invoices and details.
- Technique / pattern: Replace/enumerate the `BillingInvoice` node id inside the GraphQL query as a second (unrelated) shop's staff user and observe cross-tenant billing data returned with no ownership check.
- Takeaway: GraphQL node ids need per-object, per-tenant authorization exactly like REST — a node id accepted in a query is never proof the caller's shop owns it.

### 2026-09-17 — IDOR in user-management/invite endpoint discloses other users' account data (Automattic) — n/a
- Source: [hackerone #915114](https://hackerone.com/reports/915114)
- Type: IDOR (Insecure Direct Object Reference)
- Summary: The Crowdsignal invite/edit-user endpoint (`app.crowdsignal.com/users/invite-user.php?id=`) honored a user-supplied numeric `id` without an authorization check, exposing another user's account data such as their email address.
- Technique / pattern: Change the numeric `id` in the invite-user request to another user's value and read the details the response returns without any ownership/permission check.
- Takeaway: Admin/user-management endpoints must bind the target record to the caller's permissions; a numeric id in the query string is not authorization.

### 2026-09-16 — IDOR in User Profile allowing view/replace of another user's photo (Autodesk) — n/a
- Source: [hackerone #2962056](https://hackerone.com/reports/2962056)
- Type: IDOR (Insecure Direct Object Reference)
- Summary: The User Profile feature exposed a numeric `id` parameter that was not bound to the authenticated user, letting an attacker view and modify another user's profile photograph.
- Technique / pattern: Intercept the profile-photo request, substitute another user's `id`/object identifier, and confirm the server performs the action without an ownership check.
- Takeaway: Object references on profile/media endpoints must be authorized against the current session — a client-supplied ID is never proof of ownership.

### 2026-09-16 — IDOR exposing an internal object reference (HackerOne) — n/a
- Source: [hackerone #2633771](https://hackerone.com/reports/2633771)
- Type: IDOR (Insecure Direct Object Reference)
- Summary: An endpoint exposed a direct reference to an internal implementation object (a real backend record identifier), letting a user reach a resource by manipulating that reference.
- Technique / pattern: Identify a parameter mirroring a backend storage identifier/format, then substitute or enumerate values to reach objects belonging to other users or contexts.
- Takeaway: Don't surface raw internal identifiers; enforce per-request access control and prefer indirect/opaque references mapped server-side.

### 2026-09-15 — IDOR on Delete Email address features (Mozilla) — n/a
- Source: [HackerOne #2382484](https://hackerone.com/reports/2382484)
- Type: IDOR (broken object-level authorization)
- Summary: Mozilla Monitor let any user delete secondary email addresses belonging to other accounts by supplying the target email-address ID; the delete operation was not scoped to the requesting user.
- Technique / pattern: Trigger the "delete secondary email" action on your own account, capture the request, then swap the email-address ID for one belonging to another account — the server acted on the ID without an ownership check.
- Takeaway: Destructive actions keyed by an object ID must verify the session owns that object server-side; deletion endpoints are a high-impact IDOR sink.

### 2026-09-15 — IDOR allows deleting any user's featured images (LinkedIn) — n/a
- Source: [HackerOne #1608735](https://hackerone.com/reports/1608735)
- Type: IDOR (broken object-level authorization, DELETE)
- Summary: The endpoint that removes a profile's "featured" images honored the supplied image/object identifier without verifying ownership, letting an authenticated attacker delete featured images belonging to any LinkedIn user.
- Technique / pattern: While deleting one's own featured image, capture the delete request and swap the object id for another user's; the server performs the destructive action with no owner check. Classic two-account, test-the-DELETE-verb IDOR.
- Takeaway: Enforce object-level authorization on destructive verbs (DELETE/PATCH), not just reads — every id in a state-changing request must be checked against the session owner.

### 2026-09-15 — [IDOR] Authenticated user can access other sites' build data (GSA / Federalist) — n/a
- Source: [HackerOne #245872](https://hackerone.com/reports/245872)
- Type: IDOR (broken object-level authorization)
- Summary: In 18F's Federalist static-site platform, an authenticated user could reference site/build objects they did not own by manipulating the identifier, exposing another organization's site data; the fix landed in Federalist PR #1061.
- Technique / pattern: After authenticating, enumerate or modify the numeric site/build id in requests; the backend returned or acted on records without checking the requesting user's membership on that site.
- Takeaway: Multi-tenant apps must scope every object lookup to the caller's org/membership; sequential site/build ids make a missing check trivially exploitable.

### 2026-09-14 — IDOR to account takeover on a DoD system (U.S. Dept of Defense) — n/a
- Source: [HackerOne #969223](https://hackerone.com/reports/969223)
- Type: IDOR (broken object-level authorization)
- Summary: A Department of Defense web system honored an account/object identifier from the request without confirming the session owned it, letting an attacker reach and take over another user's account (researcher write-up: naglinagli.github.io/DoD_IDOR).
- Technique / pattern: Classic two-account IDOR — authenticate as A, capture an account-scoped request, substitute B's identifier, and confirm cross-account access; government VDPs often triage such write/access IDORs quickly.
- Takeaway: Every account-scoped endpoint must verify ownership server-side; an identifier present in the request is never proof of authorization.

### 2026-09-14 — Full account take-over via recovery-email manipulation (U.S. Dept of Defense) — n/a
- Source: [HackerOne #847452](https://hackerone.com/reports/847452)
- Type: IDOR (broken object-level authorization)
- Summary: An account-update POST accepted attacker-controlled identity fields (userName, originalEmail, Email, RecoveryEmail), letting an attacker set a victim's recovery email and fully take over the account on a DoD social network.
- Technique / pattern: Inspect account-update POST bodies for identity fields (email, recovery email, username); repoint them at a victim, then drive the password-reset/recovery flow to the attacker-controlled address.
- Takeaway: Bind sensitive-field updates — especially recovery email — to the authenticated session and require re-verification; never trust identity fields supplied in the request body.


### 2026-09-13 — IDOR to account takeover on POST to profile endpoint (Mars) — n/a
- Source: [HackerOne #2132183](https://hackerone.com/reports/2132183)
- Type: IDOR (broken object-level authorization)
- Summary: A profile-related POST endpoint honored a user identifier taken from the request without verifying the session owned it, letting an attacker overwrite another user's account data and take it over.
- Technique / pattern: Two-account testing — capture the authenticated POST as user A, substitute user B's identifier, and confirm B's record changes; write-verb IDOR (POST/PUT) is usually higher-impact than read-only IDOR.
- Takeaway: Enforce object-level authorization on every state-changing request, not just reads; never trust an id supplied in the request body.

### 2026-09-13 — IDOR leads to account takeover without user interaction (MTN Group) — n/a
- Source: [HackerOne #1272478](https://hackerone.com/reports/1272478)
- Type: IDOR (broken object-level authorization)
- Summary: On an MTN business subdomain a logged-in user could update another user's profile fields (e.g. address) by manipulating the identifier in an update request, giving zero-interaction account takeover.
- Technique / pattern: Register two verified accounts, perform a profile update as A, replay the request substituting B's id, and diff responses to confirm cross-account writes.
- Takeaway: Bind every update to the authenticated session's own object id server-side; multi-tenant subdomains need the same authZ checks as the main app.

### 2026-09-12 — IDOR in Bugs overview via hackathons filter parameter (HackerOne) — n/a
- Source: [HackerOne #663431](https://hackerone.com/reports/663431)
- Type: IDOR
- Summary: The /bugs overview accepted a `hackathons[]` filter parameter; supplying arbitrary hackathon IDs let a program member scope the view to hackathons their program was not part of.
- Technique / pattern: Tampered a client-supplied filter parameter (`hackathons[]=<id>`) that was trusted server-side without an ownership/scope check, enumerating IDs to reach out-of-scope objects.
- Takeaway: Filter and query parameters are object references too — enforce authorization on every ID the client can supply, not just on the primary resource in the path.

### 2026-09-12 — Critical IDOR allows deleting any organization member (Veris) — n/a
- Source: [HackerOne #120115](https://hackerone.com/reports/120115)
- Type: IDOR
- Summary: A DELETE request to the org-member management API (`/api/v1/org-member/.../<memberID>/`) removed the referenced member with no check that the caller was authorized over that member or organization.
- Technique / pattern: Captured the member-removal request and swapped the member ID to values belonging to other organizations, performing the destructive action entirely from the attacker's own account.
- Takeaway: State-changing endpoints (especially DELETE) need object-level authorization; an authenticated session is not proof the caller owns the target object.

### 2026-09-11 — IDOR on Feedback Review (HackerOne) — n/a
- Source: [HackerOne #262661](https://hackerone.com/reports/262661)
- Type: IDOR / broken object-level authorization (disclosure + unauthorized write)
- Summary: HackerOne's feedback-review feature let programs post publicly visible feedback on hackers who never engaged with them, and let hackers read private feedback meant to stay confidential.
- Technique / pattern: Replay the feedback endpoint with object ids you shouldn't be able to act on — reading others' private feedback and creating feedback bound to arbitrary hacker/program ids — because the reference was trusted without a relationship check.
- Takeaway: Object-level checks must cover BOTH read and create/update, and confirm a legitimate relationship exists between the actor and the referenced object.

### 2026-09-11 — IDOR causing deletion/closure of any user account (Ubiquiti) — n/a
- Source: [HackerOne #156537](https://hackerone.com/reports/156537)
- Type: IDOR / broken object-level authorization (destructive)
- Summary: On community.ubnt.com an account close/delete action honored a client-supplied victim user id, letting an attacker close any user's account and wipe all of their data.
- Technique / pattern: Capture the account-close/delete request, replace your own (predictable/sequential) user id with a victim's, and replay; the server performs the destructive action without verifying ownership. Always test destructive verbs, not just reads.
- Takeaway: Enforce authorization on every state-changing action — especially destructive ones; sequential ids turn one IDOR into mass account deletion.

### 2026-09-11 — IDOR when editing email leads to Account Takeover on Atavist (Automattic) — n/a
- Source: [HackerOne #950881](https://hackerone.com/reports/950881)
- Type: IDOR / broken object-level authorization → ATO
- Summary: The email-change flow on Atavist honored a client-supplied user id without checking ownership, so an attacker could set any user's email address; because ids are sequential, this automated into mass account takeover.
- Technique / pattern: Intercept the "change email" request, swap the user id to a victim's (sequential) id, submit — the attacker-controlled email is bound to the victim account, then trigger password reset to finish the takeover.
- Takeaway: Identity-change actions (email/phone) must bind to the authenticated session, never a request parameter; sequential ids turn one IDOR into bulk ATO.

### 2026-09-11 — IDOR to view order information of other users (Affirm) — n/a
- Source: [HackerOne #1323406](https://hackerone.com/reports/1323406)
- Type: IDOR / broken object-level authorization (disclosure)
- Summary: An order-information endpoint returned other users' order details when the request referenced an order/object id the caller did not own.
- Technique / pattern: Capture a "view order" API call, replace the order id with another user's value and replay; with no per-object authorization the victim's order data is returned — confirmed by two-account diffing.
- Takeaway: Read endpoints on order/transaction objects need server-side ownership checks; a valid session authenticates the caller but does not authorize the object.

### 2026-09-10 — Broken access control (IDOR) in booking detail & bids (Bykea) — n/a
- Source: [HackerOne #2374730](https://hackerone.com/reports/2374730)
- Type: IDOR / broken access control (API, disclosure)
- Summary: Booking/bid API endpoints returned another customer's booking details, bid information and bid config when the request referenced that user's id/token, exposing other users' data.
- Technique / pattern: Capture a "view booking/bids" API call, swap the user id (and access token) to a victim's, and replay — the endpoint returns the object without checking that the session owns it.
- Takeaway: Read endpoints on booking/ride objects need per-object authorization, not just a valid token; two-account diffing on every id-bearing API call surfaces these.

### 2026-09-10 — IDOR in /bugs.json exposes private reports (HackerOne) — n/a
- Source: [HackerOne #2487889](https://hackerone.com/reports/2487889)
- Type: IDOR / broken object-level authorization
- Summary: A POST to the `/bugs.json` endpoint with an organization id and a text query returned private report data that the caller should not have been able to read.
- Technique / pattern: Probe JSON/search endpoints with an org/object id you don't own; if results aren't scoped to your session, private records leak. Vary the id and query params.
- Takeaway: Search/list endpoints must filter by the authenticated principal server-side; trusting a client-supplied org id is an IDOR.

### 2026-09-09 — IDOR to add secondary users via businessmanage API (PayPal) — $10,500
- Source: [HackerOne #415081](https://hackerone.com/reports/415081)
- Type: IDOR / broken object-level authorization (API)
- Summary: The `businessmanage/users` API accepted an account identifier that wasn't tied to the caller's session, letting an attacker **add secondary users to another account**.
- Technique / pattern: Capture the "add user" request, swap the account/object id to a victim's, replay — the endpoint acts on the supplied id without an ownership/authZ check.
- Takeaway: State-changing API actions must authorize the target object against the session, not just authenticate the caller. High impact because it grants account control.

### 2026-09-09 — IDOR to expire other users' sessions (Shopify) — $1,000
- Source: [HackerOne #56511](https://hackerone.com/reports/56511)
- Type: IDOR (session management action)
- Summary: A session-management endpoint let a user **invalidate/expire another user's sessions** by referencing the victim's identifier.
- Technique / pattern: Enumerate/guess the target id on an action endpoint; the action runs without verifying the session owns that target → cross-user effect (DoS / forced logout).
- Takeaway: Even "minor" actions (logout, expire) need per-object authorization; IDOR isn't only about reading data.
