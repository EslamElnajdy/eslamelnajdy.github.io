---
title: "Android Deep Linking for Pentesters"
date: 2026-03-24
categories: [android, Deep-Linking, mobile]
tags: [android, Deep-Linking, android-security, mobile-security, adb, bug-bounty, tutorial]

---

## Deep Linking

Deep linking enables external URLs or intents to navigate directly to specific content inside an Android application.

## Types of deep linking in Android

### 1) Custom Scheme deep links

Use a custom URI scheme defined by the app.

```bash
myapp://profile/123
```

- Not secure
- May conflict with other apps

### 2) Implicit deep links (HTTP/HTTPS)

Use standard web URLs that can open the app.

```bash
https://example.com/product/123
```

- If the app is installed → opens the app
- Otherwise → opens in the browser

### 3) Android App Links (verified links)

A secure type of deep linking using HTTP/HTTPS with domain verification.

```bash
https://myapp.com/profile
```

- Verified with the website
- Opens directly in the app without a chooser dialog
- More secure than other types

## How it works

In `AndroidManifest.xml`, add an intent filter so the app can receive the link:

```xml
<intent-filter>
	<action android:name="android.intent.action.VIEW"/>

	<category android:name="android.intent.category.DEFAULT"/>
	<category android:name="android.intent.category.BROWSABLE"/>

	<data
		android:scheme="myapp"
		android:host="product" />
</intent-filter>
```

## Deep linking bug hunting scenarios

### Scenario 1: Access control bypass

Goal: Access protected content without authentication.

Case:

```bash
myapp://profile/123
```

Steps:

1. Log out from the app
2. Trigger the deep link:
    
    ```bash
    adb shell am start -a android.intent.action.VIEW -d "myapp://profile/123"
    ```
    

Expected: The app should require login.

If you see: The profile opens directly without authentication → Access control bypass.

Impact:

- Unauthorized access to user data
- Possible account takeover

---

### Scenario 2: Parameter manipulation

Goal: Modify sensitive parameters.

Case:

```bash
myapp://pay?user=123&amount=100
```

Steps:

Try modifying the values:

```bash
myapp://pay?user=999&amount=100000
```

Expected: The app/server should validate and reject.

If you see: The transaction proceeds with modified values → Parameter tampering.

Impact:

- Financial fraud
- Unauthorized transactions

---

### Scenario 3: Open redirect

Case:

```bash
myapp://open?url=https://trusted.com
```

Steps:

Try:

```bash
myapp://open?url=https://evil.com
```

Expected: The app should restrict allowed domains.

If you see: The malicious URL opens → Open redirect.

Impact:

- Phishing attacks
- User redirection to malicious sites

---

### Scenario 4: WebView injection

Goal: Inject malicious input into a WebView.

Case: The app loads URLs inside a WebView.

Steps:

Try:

```bash
myapp://web?url=javascript:alert(1)
```

Or:

```bash
myapp://web?url=file:///sdcard/secret.txt
```

If you see:

- JavaScript executes
- Local files are accessed

Then: WebView injection.

Impact:

- Data theft
- Code execution

---

### Scenario 5: Sensitive data exposure

Case:

```bash
myapp://reset?token=ABC123
```

Observation: The deep link contains sensitive data (token).

Risk: The token might be exposed via logs, other apps, or browser history.

Impact:

- Account takeover
- Session hijacking

---

### Scenario 6: Link hijacking (implicit links)

Case:

```bash
https://example.com/profile
```

Idea: Another app can register the same intent filter.

If you see: A malicious app intercepts the link → Deep link hijacking.

Impact:

- Data interception
- Credential theft

---

### Scenario 7: Privilege escalation

Case:

```bash
myapp://admin
```

Steps: Open it using a normal user account.

If you see: Admin panel opens → Privilege escalation.

Impact:

- Full control over the app
- Access to admin features

## Labs

## Reports

1. https://hackerone.com/reports/401793
2. https://hackerone.com/reports/1087744
3. https://hackerone.com/reports/2553411
4. https://hackerone.com/reports/1372667
5. https://hackerone.com/reports/583987