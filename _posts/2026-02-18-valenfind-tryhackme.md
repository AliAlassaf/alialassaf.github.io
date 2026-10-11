---
layout: post
title: "ValenFind (TryHackMe): From LFI to the Admin Database"
date: 2026-02-18
tags: [tryhackme, web, lfi, information-disclosure]
canonical_url: https://medium.com/@youalialassaf/valenfind-tryhackme-a8001e85d5e8
excerpt: "A Medium web room where a layout parameter leads to Local File Inclusion, the app source code, a hardcoded admin token, and a hidden database export endpoint."
---

*Originally published on [Medium](https://medium.com/@youalialassaf/valenfind-tryhackme-a8001e85d5e8). The flag is redacted here in line with TryHackMe's guidance.*

| Field | Value |
|---|---|
| Machine | ValenFind |
| Difficulty | Medium |
| Vulnerability | LFI and Information Disclosure |

## 1. Initial attempts (SQLi and XSS)

I started by testing the login page for SQL Injection with standard payloads such as `' OR 1=1--`, but the server returned "Invalid credentials," so the attempt failed.

![Login page rejecting a SQL injection payload](/assets/images/valenfind/01-sqli-login.webp)

Next I registered a user to test the internal pages. In the profile settings I tried stored XSS payloads in the Real Name, Phone, and Address fields. The application sanitized the input, so nothing interesting happened.

![Profile form with script payloads in the fields](/assets/images/valenfind/02-xss-profile.webp)

## 2. Discovery: Local File Inclusion

While browsing I noticed an API request that looked interesting:

```
GET /api/fetch_layout?layout=theme_classic.html
```

The `layout` parameter looked like it might read a file from disk. I intercepted the request in Burp and changed it to a path traversal payload pointing at `/etc/passwd`. The server returned the file contents.

![Burp showing the etc-passwd file returned by the layout parameter](/assets/images/valenfind/03-lfi-etc-passwd.webp)

I then tried to read the SSH private key for the `ubuntu` user. That failed, but the error message leaked the application's full path:

```
Error loading theme layout: [Errno 2] No such file or directory:
'/opt/Valenfind/templates/components/<traversal>/home/ubuntu/.ssh/id_rsa'
```

So the app lives in `/opt/Valenfind/`. To confirm the main file I read the process command line through `/proc/self/cmdline`, which pointed at `/opt/Valenfind/app.py`.

## 3. Reading the source code

Using the same LFI I read `app.py`. Inside I found two important things:

1. A hardcoded admin token.
2. A hidden endpoint: `/api/admin/export_db`.

I also saw that the developer blocked any path ending in `.db`, so I could not read the database file directly through the LFI.

## 4. Exploitation

The `.db` block only covered direct file reads, not the export endpoint. I called the hidden endpoint with the admin token from the source code:

```
curl -H "X-Valentine-Token: <admin-token-from-source>" \
     http://TARGET:5000/api/admin/export_db -o cupid.db
```

Then I opened the database and listed the users:

```
sqlite3 cupid.db
SELECT * FROM users;
```

The admin user's row contained the flag.

![Database dump with the flag redacted](/assets/images/valenfind/04-database-dump.webp)

> Flag: `THM{ redacted }`

## What I took away

- An error message is information disclosure: the failed SSH-key read is what handed me the app's real path.
- One blocklist is not a control. Blocking `.db` reads did nothing once the export endpoint plus a hardcoded token were in the source.
- Chaining low-noise findings (LFI, then source review, then a hidden endpoint) is what turned "read a file" into full database access.
