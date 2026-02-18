---
title: "XSS with HttpOnly Cookies — What’s the Real Impact?"
date: 2025-12-31
categories: [web, tutorials, XSS]
tags: [XSS, tutorials, web-security]
---
---

If you have an **XSS vulnerability** in your target but the session cookie is set with:`httpOnly=true` 
This means the browser **prevents JavaScript from accessing the session cookie**, which **mitigates session hijacking** via `document.cookie`.
However, **the XSS vulnerability is still exploitable** and can lead to serious impact.

---

## Scenario 1: Action Abuse

Even though the attacker cannot steal the session cookie, the injected JavaScript still executes **in the context of the victim’s authenticated session**.

This allows an attacker to perform **authenticated actions on behalf of the victim**, such as:

- Changing the email address
- Updating account settings
- Deleting data
- Performing sensitive state-changing actions

### Example PoC

```jsx
fetch("/change-email", {
  method: "POST",
  body: "email=attacker@mail.com",
  credentials: "include"
});
```

The browser automatically includes the victim’s session cookies, so the request is processed as if it was sent by the victim.

---

## Scenario 2: CSRF-Like Attack

Even if a **CSRF token** is implemented, XSS can often bypass it.

Since the attacker’s JavaScript runs in the same origin, it can:

- Read the CSRF token from the DOM or JavaScript variables
- Send a valid request including the token

This effectively turns the XSS into a **CSRF bypass**, allowing unauthorized state-changing requests.

---

## Scenario 3: Sensitive Data Exfiltration (Non-Cookie)

Even if the session cookie is protected with `HttpOnly=true`, there is often **other sensitive data that is not stored in cookies**.

Many applications store important information in client-side locations such as:

- User email
- User ID
- Access tokens
- API responses
- Feature flags or internal identifiers

These values are commonly stored in:

- `localStorage`
- `sessionStorage`
- JavaScript variables
- Embedded HTML or JSON responses

Since XSS executes in the context of the victim’s browser, an attacker can **read and exfiltrate this data**, even when cookies are not accessible.

---

## Scenario 4: UI Redressing & In-Page Phishing

Since the injected JavaScript executes **within the trusted origin of the target application**, an attacker can fully manipulate the page’s DOM.

This allows the attacker to:

- Modify existing UI elements
- Inject fake forms or dialogs
- Display convincing prompts requesting sensitive information such as:
    - Account passwords
    - One-Time Passwords (OTP)
    - Credit card details

Because the attack occurs on a **legitimate and trusted domain**, users are far more likely to trust the content and interact with it.

From the user’s perspective, everything appears normal and legitimate.