## HTTP Methods — Deep Dive on GET Requests

---

## 1. HTTP — A Stateless Protocol

### What Does "Stateless" Mean?
Each HTTP request is processed **independently** — the server does **not** remember previous requests.

```
Request 1 ──► Server (no memory of this after responding)
Request 2 ──► Server (treats this as brand new, unrelated to Request 1)
```

### How Websites Add "Statefulness"
Since HTTP itself has no memory, websites use **sessions** to simulate it:

```
[Login Request] ──► Server creates Session ID ──► Stored in Cookie/Token
                                                          │
[Future Requests] ──► Cookie/Token sent each time ◄──────┘
                            │
                Server recognises you without re-login
```

> Without this, you'd have to **re-enter your login credentials** on every single page request!

---
---

## 2. HTTP Methods (Commands)

According to the official HTTP specifications (**RFC documents**), there are **9 core methods**:

```
GET      POST     PUT
DELETE   PATCH    HEAD
OPTIONS  CONNECT  TRACE
```

> In HTTP terminology, these are called **"methods"**, not "commands."

---
---

## 3. The GET Method

### What is GET?
Used to **retrieve a resource** from a web server.

```
GET https://tryhackme.com/index.php
```
> This retrieves the **homepage** of TryHackMe.

---

### How a GET Request Happens (Behind the Scenes)

```
[You type URL] ──► [Browser constructs HTTP request] ──► [Web Server]
                                                                │
                                                                ▼
                                                    [Server processes request]
                                                                │
                                                                ▼
[Browser receives & renders] ◄──── [Response: Status Code + Data] 
```

You never type the raw HTTP request yourself — your **browser builds it automatically** using the URL and other standard fields.

---
---

## 4. Inspecting a GET Request (Practical)

### Tools Used
**Firefox Developer Tools** → `F12` or Right-click → **Inspect** → **Network tab**

---

### Key Fields You'll See

| Field | Meaning |
|---|---|
| **Scheme** | Protocol used — HTTP or HTTPS |
| **Host** | The server/domain being requested |
| **Filename** | The specific file requested (e.g. `/` = `index.html`) |
| **Address** | The IP address hosting the website |
| **Status** | Outcome of the request (e.g. `200 OK` = success) |

---

### Example Walkthrough
```
URL Requested:  http://httpdemo.local:8080
       │
       ▼
Scheme:    HTTP
Host:      httpdemo.local
Filename:  /  (→ index.html)
Address:   127.0.0.1  (locally hosted)
Status:    200 OK ✅
```

---
---

## 5. Anatomy of an HTTP Response

```
┌─────────────────────────────┐
│      RESPONSE HEADER        │  ← Metadata about the response
├─────────────────────────────┤
│       RESPONSE BODY         │  ← The actual requested content (HTML, etc.)
└─────────────────────────────┘
```

| Part | Contains |
|---|---|
| **Response Header** | Metadata (status, content type, server info, etc.) |
| **Response Body** | The actual content — e.g. the HTML of the webpage |


---
---

