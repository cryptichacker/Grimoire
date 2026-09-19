---
tags: [hacking, web, portswigger, business-logic]
type: vulnerability-category
source: https://portswigger.net/web-security/logic-flaws
last-verified: 2026-08-25
---

# Business logic vulnerabilities

## Up
- [[PortSwigger]]

## What it is
Business logic vulnerabilities are flaws in the *design and implementation* of an application that let an attacker elicit unintended behaviour by using legitimate functionality in ways the developers didn't anticipate. They're also called application logic flaws.

## How they arise
Developers make **flawed assumptions about how users will interact** with the app — assuming interaction only via a browser, trusting client-side validation, or not foreseeing unusual sequences and inputs. Because assumptions are often undocumented and systems are complex, unexpected states go unhandled.

## Impact
Depends entirely on the affected function: privilege escalation or authentication bypass, direct financial loss/fraud, or reputational damage even where the attacker gains nothing tangible.

## Categories of logic flaws

- **Excessive trust in client-side controls** — the server assumes values validated/limited in the browser (prices, quantities, discounts) can't be tampered with. Intercept and change them.
- **Failing to handle unconventional input** — negative numbers, huge values, integer overflow, or values that break an implicit assumption (e.g. a negative quantity that reduces the total).
- **Making flawed assumptions about user behaviour** — assuming users complete steps in order, don't revisit steps, or that a check done once needn't be repeated.
- **Domain-specific flaws** — abusing discount/loyalty/coupon logic, or workflow rules specific to the business (e.g. stacking promotions).
- **Providing an encryption oracle** — user-controllable input is encrypted and the ciphertext exposed elsewhere, letting the attacker get arbitrary data encrypted/decrypted and forge trusted values.
- **Email address parser discrepancies** — differences in how the app vs a downstream system parse an email address let an attacker register/verify as a domain they don't own (e.g. encoded or sub-addressed forms).

## Prevention
- Never trust client-side validation; re-validate and enforce all assumptions **server-side**.
- Maintain clear design documentation stating every assumption about state and user behaviour.
- Write code that's explicit about the domain rules; avoid relying on tribal knowledge.
- Test the *logic* deliberately — think adversarially about every step, order, and boundary.

## Labs
11 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Excessive trust in client-side controls | Apprentice | Intercept the purchase and change the price the client sent. | |
| High-level logic vulnerability | Apprentice | Send a negative quantity/price to manipulate the order total. | |
| Inconsistent security controls | Apprentice | Register/upgrade to a privileged role using a self-service flow that wasn't meant for you. | |
| Flawed enforcement of business rules | Apprentice | Combine/reuse coupon codes the app assumed couldn't be stacked. | |
| Low-level logic flaw | Practitioner | Exploit integer overflow by ordering enough items to wrap the total. | |
| Inconsistent handling of exceptional input | Practitioner | Overflow/truncate a field (e.g. long email) to land in a privileged domain. | |
| Weak isolation on dual-use endpoint | Practitioner | Omit a parameter on a shared endpoint to skip the current-password check. | |
| Insufficient workflow validation | Practitioner | Skip payment by replaying the order-confirmation step directly. | |
| Authentication bypass via flawed state machine | Practitioner | Skip the role-selection step to default into an admin role. | |
| Infinite money logic flaw | Practitioner | Automate a gift-card/coupon loop that nets positive balance each cycle. | |
| Authentication bypass via encryption oracle | Expert | Use a stay-logged-in cookie as an encryption oracle to forge a valid admin token. | |
