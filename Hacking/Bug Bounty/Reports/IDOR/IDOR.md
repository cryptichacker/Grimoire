---
tags: [hacking, bug-bounty, idor, reports]
type: log
---

# IDOR

## Up
- [[Reports]]

Disclosed **Insecure Direct Object Reference** reports. Core idea: an object identifier (id, uuid, filename, account number) in a request is honored **without checking the session owns that object**. See [[Pattern]] → IDOR.

## Reports

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
