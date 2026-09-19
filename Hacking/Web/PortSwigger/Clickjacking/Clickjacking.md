---
tags: [hacking, web, portswigger, clickjacking]
type: vulnerability-category
source: https://portswigger.net/web-security/clickjacking
last-verified: 2026-08-25
---

# Clickjacking

## Up
- [[PortSwigger]]

## What it is
Clickjacking (UI redressing) tricks a user into clicking actionable content on a hidden page by overlaying it, invisibly, on top of a decoy page. The target site is loaded in a transparent iframe positioned so the victim's clicks land on its buttons while they think they're interacting with the decoy.

## Difference from CSRF
Clickjacking needs a genuine user **click**, and the request goes through the real session in the real domain — so **CSRF tokens don't help**, because the token is present and valid. The defense is *frame* control, not request tokens.

## Constructing a basic attack
Layer a near-invisible target iframe above a decoy with CSS:
```html
<style>
  #target_website { position:relative; width:128px; height:128px; opacity:0.00001; z-index:2; }
  #decoy_website  { position:absolute; width:300px; height:400px; z-index:1; }
</style>
<div id="decoy_website">...decoy content aligning a button under the target...</div>
<iframe id="target_website" src="https://vulnerable-website.com"></iframe>
```
Tune `opacity` (low but non-zero — browsers may flag full transparency), `z-index` (target on top), and position so the target's action sits under the decoy's lure.

## Variations
- **Prefilled forms** — put the values in GET parameters on the iframe `src` so the framed form is pre-populated; overlay only the submit button.
- **Clickjacking → DOM XSS** — frame a URL that carries a DOM-XSS payload so the click triggers script execution.
- **Multistep** — stack several iframes/overlays to walk the victim through a sequence (e.g. add to cart, then confirm).

## Frame busters & bypass
Frame-busting JS checks it isn't framed (e.g. `top === self`). Bypass with the iframe `sandbox` attribute — grant `allow-forms`/`allow-scripts` but omit `allow-top-navigation`, so the buster can't navigate the top window:
```html
<iframe src="https://victim-website.com" sandbox="allow-forms"></iframe>
```

## Prevention
- **`X-Frame-Options`**: `deny`, `sameorigin`, or a specific allow-from (support varies).
- **CSP `frame-ancestors`** (preferred): `Content-Security-Policy: frame-ancestors 'self';` (or `'none'`, or a domain allow-list).
- **`SameSite`** cookies reduce impact by not sending the session in the framed context.
- Use these as layered defense; test carefully.

## Labs
5 labs. Verify exact titles against the live page.

| Lab | Difficulty | General approach | Notes |
|---|---|---|---|
| Basic clickjacking with CSRF token protection | Apprentice | Overlay a decoy over the framed account-delete button. | |
| Clickjacking with form input data prefilled from a URL parameter | Apprentice | Prefill the email via GET param, overlay the submit button. | |
| Clickjacking with a frame buster script | Practitioner | Use `sandbox="allow-forms"` to neutralize the buster. | |
| Exploiting clickjacking vulnerability to trigger DOM-based XSS | Practitioner | Frame a URL with a DOM-XSS payload; align the click. | |
| Multistep clickjacking | Practitioner | Two overlays to complete a two-step action. | |
