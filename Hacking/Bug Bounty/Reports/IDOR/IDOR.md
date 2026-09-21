---
tags: [hacking, bug-bounty, idor, reports]
type: log
---

# IDOR

## Up
- [[Reports]]

Disclosed **Insecure Direct Object Reference** reports. Core idea: an object identifier (id, uuid, filename, account number) in a request is honored **without checking the session owns that object**. See [[Pattern]] → IDOR.

## Reports

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
