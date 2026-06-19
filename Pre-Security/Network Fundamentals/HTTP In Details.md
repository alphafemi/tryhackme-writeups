## HTTP, HTTPS, URLs & Web Fundamentals

---

## 1. HTTP vs HTTPS

| | HTTP | HTTPS |
|---|---|---|
| **Full Name** | HyperText Transfer Protocol | HyperText Transfer Protocol **Secure** |
| **Security** | Unencrypted | Encrypted |
| **Created By** | Tim Berners-Lee (1989–1991) | Secure version of HTTP |
| **Purpose** | Rules for communicating with web servers | Same, but with encryption & server identity verification |

> HTTP is the **language** your browser and web servers use to exchange data — HTML, images, videos, etc.

---
---

## 2. URLs — Uniform Resource Locator

### Anatomy of a URL

```
http://user:password@tryhackme.com:80/view-room?id=1#task3
  │         │              │         │      │        │      │
Scheme    User          Host      Port    Path   Query    Fragment
                                                  String
```

| Component | Purpose | Example |
|---|---|---|
| **Scheme** | Protocol to use | `http`, `https`, `ftp` |
| **User** | Login credentials (if required) | `user:password` |
| **Host** | Domain name or IP address | `tryhackme.com` |
| **Port** | Connection port | `80` (HTTP), `443` (HTTPS) |
| **Path** | Location/file being requested | `/view-room` |
| **Query String** | Extra data sent to the path | `?id=1` |
| **Fragment** | Specific section of the page | `#task3` |

---
---

## 3. Making an HTTP Request

### Minimal Request
```
GET / HTTP/1.1
```

### Full Example Request
```http
GET / HTTP/1.1
Host: tryhackme.com
User-Agent: Mozilla/5.0 Firefox/87.0
Referer: https://tryhackme.com/
```

| Line | Meaning |
|---|---|
| 1 | **GET** method, requesting `/` (homepage), using **HTTP/1.1** |
| 2 | Which website is being requested |
| 3 | Browser software/version being used |
| 4 | The page that linked/referred to this request |
| 5 | Blank line — signals end of request |

---

### Full Example Response
```http
HTTP/1.1 200 OK
Server: nginx/1.15.8
Date: Fri, 09 Apr 2021 13:34:03 GMT
Content-Type: text/html
Content-Length: 98

<html>
<head><title>TryHackMe</title></head>
<body>Welcome To TryHackMe.com</body>
</html>
```

| Line | Meaning |
|---|---|
| 1 | HTTP version + **status code** (200 OK = success) |
| 2 | Web server software/version |
| 3 | Date/time of response |
| 4 | Type of content being returned |
| 5 | Length of the response (confirms nothing is missing) |
| 6 | Blank line — end of headers |
| 7+ | The actual requested content |

---
---

## 4. HTTP Methods

| Method | Purpose |
|---|---|
| **GET** | Retrieve information from the server |
| **POST** | Submit data, potentially creating new records |
| **PUT** | Submit data to **update** existing information |
| **DELETE** | Remove information/records |

---
---

## 5. HTTP Status Codes

### Status Code Ranges

```
100-199 ── Informational Response
200-299 ── ✅ Success
300-399 ── ↪️ Redirection
400-499 ── ❌ Client Error
500-599 ── 💥 Server Error
```

### Common Status Codes

| Code | Meaning |
|---|---|
| **200** | OK — request successful |
| **201** | Created — new resource created |
| **301** | Moved Permanently — page moved for good |
| **302** | Found — temporary redirect |
| **400** | Bad Request — something wrong/missing in request |
| **401** | Not Authorised — need to log in |
| **403** | Forbidden — no permission to view |
| **404** | Page Not Found |
| **405** | Method Not Allowed — wrong HTTP method used |
| **500** | Internal Server Error |
| **503** | Service Unavailable — server overloaded/down |

---
---

## 6. HTTP Headers

### Common Request Headers (Client → Server)

| Header | Purpose |
|---|---|
| **Host** | Specifies which website on a multi-site server |
| **User-Agent** | Identifies browser type/version |
| **Content-Length** | Tells server how much data to expect |
| **Accept-Encoding** | Lists supported compression methods |
| **Cookie** | Sends stored data back to server |

### Common Response Headers (Server → Client)

| Header | Purpose |
|---|---|
| **Set-Cookie** | Tells browser to store data for future requests |
| **Cache-Control** | How long to cache the response |
| **Content-Type** | Type of data returned (HTML, image, video, etc.) |
| **Content-Encoding** | Compression method used |

---
---

## 7. Cookies

### What is a Cookie?
A small piece of data stored on your computer, sent by a server via the **Set-Cookie** header.

### Why Cookies Exist
HTTP is **stateless** — it doesn't remember previous requests. Cookies solve this by letting the server **recognise you** across multiple requests.

```
[Browser] ──── Request ────────────────►  [Server]
[Browser] ◄─── Set-Cookie: session=abc123 [Server]
[Browser] ──── Cookie: session=abc123 ───► [Server]
   (sent on every future request)
```

### Common Uses
- **Authentication** (login sessions)
- **Personal settings**
- **Tracking** previous visits

> Cookie values are usually **tokens** (random secret codes), not plain-text passwords.

### Viewing Cookies
- Open browser **Developer Tools** → **Network tab** → select a request → **Cookies tab**

---
---

