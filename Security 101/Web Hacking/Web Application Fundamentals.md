Web Application Fundamentals & HTTP Security Headers
 
## Overview
 
This document covers the architecture of web applications (front end vs. back end), the anatomy of a URL, the structure of HTTP messages, requests, and responses, and the HTTP security headers used to harden web applications against common attacks such as XSS, MIME-sniffing, and protocol downgrade attacks.
 
---
 
### Web Application Architecture
 
A web application can be thought of as a planet: users (astronauts) interact with its surface, while a complex ecosystem operates beneath.
 
#### Front End (Client-Side)
 
What the user sees and interacts with in the browser:
 
| Technology | Role | Planet Analogy |
|---|---|---|
| **HTML** | Structure and content — tells the browser what to display | DNA of simple organisms |
| **CSS** | Visual styling — colours, layout, typography | DNA elements describing appearance |
| **JavaScript** | Interactivity and dynamic behavior — enables decisions and logic | The brain of an advanced organism |
 
#### Back End (Server-Side)
 
What operates invisibly to support the application:
 
| Component | Role | Planet Analogy |
|---|---|---|
| **Database** | Stores, modifies, and retrieves information | Maps, libraries, filing cabinets |
| **Web/App Server** | Processes requests and serves responses | Roads, vehicles, fuel |
| **WAF** | Filters dangerous requests before they reach the server | Atmosphere filtering harmful radiation |
 
---
 
### URL Anatomy
 
A URL (Uniform Resource Locator) is a structured web address. Each component serves a specific purpose:
 
```
https://user:pass@tryhackme.com:443/api/users?id=123#section
  │      │    │       │           │     │         │      │
Scheme  User Pass    Host        Port  Path     Query  Fragment
```
 
| Component | Example | Purpose | Security Note |
|---|---|---|---|
| **Scheme** | `https://` | Protocol to use (HTTP/HTTPS) | HTTPS encrypts the connection |
| **User** | `user:pass@` | Embedded credentials (rare) | Avoid — exposes credentials in URLs |
| **Host/Domain** | `tryhackme.com` | Which server to reach | Watch for typosquatting in phishing |
| **Port** | `:443` | Which service port | 80=HTTP, 443=HTTPS |
| **Path** | `/api/users` | Resource location on server | Validate to prevent unauthorized access |
| **Query String** | `?id=123` | Parameters passed to the resource | Sanitize — injection attack vector |
| **Fragment** | `#section` | In-page anchor reference | Sanitize — injection attack vector |
 
---
 
### HTTP Message Structure
 
All HTTP communication follows the same basic structure:
 
```
Start Line        ← What kind of message and key metadata
Headers           ← Key-value pairs: instructions and context
                  ← (empty line separates headers from body)
Body              ← The actual data (optional)
```
 
---
 
### HTTP Requests
 
#### Request Line Structure
 
```
METHOD /path HTTP/version
```
 
**Example:** `POST /api/user HTTP/1.1`
 
#### HTTP Methods
 
| Method | Purpose | Security Consideration |
|---|---|---|
| `GET` | Retrieve data (no side effects) | Never put sensitive data in GET params — visible in URLs/logs |
| `POST` | Send data to create/update | Always validate and sanitize input (SQL injection, XSS) |
| `PUT` | Replace/update a resource | Verify authorization before accepting |
| `DELETE` | Remove a resource | Restrict to authorized users only |
| `PATCH` | Partial update of a resource | Validate data to avoid inconsistencies |
| `HEAD` | Like GET but headers only | Safe for metadata checks |
| `OPTIONS` | Discover available methods | Disable if not needed — can expose capabilities |
| `TRACE` | Debug — echoes the request back | Disable in production — security risk |
| `CONNECT` | Create a tunnel (e.g., for HTTPS) | Essential for encrypted tunneling |
 
#### HTTP Versions
 
| Version | Year | Key Features |
|---|---|---|
| HTTP/0.9 | 1991 | GET only |
| HTTP/1.0 | 1996 | Headers, content types, basic caching |
| HTTP/1.1 | 1997 | Persistent connections, chunked transfer, improved caching — still widely used |
| HTTP/2 | 2015 | Multiplexing, header compression, request prioritization |
| HTTP/3 | 2022 | Built on QUIC (UDP-based) — faster, more secure connections |
 
#### Common Request Headers
 
| Header | Example | Purpose |
|---|---|---|
| `Host` | `Host: tryhackme.com` | Identifies the target web server |
| `User-Agent` | `User-Agent: Mozilla/5.0` | Identifies the browser/client |
| `Referer` | `Referer: https://google.com/` | Where the request originated from |
| `Cookie` | `Cookie: session=abc123` | Sends stored session data to the server |
| `Content-Type` | `Content-Type: application/json` | Declares the format of the request body |
 
#### Request Body Formats
 
**URL Encoded** (`application/x-www-form-urlencoded`)
```http
POST /profile HTTP/1.1
Content-Type: application/x-www-form-urlencoded
 
name=Aleksandra&age=27&country=US
```
 
**Form Data** (`multipart/form-data` — used for file uploads)
```http
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----Boundary
 
----Boundary
Content-Disposition: form-data; name="username"
aleksandra
----Boundary
Content-Disposition: form-data; name="profile_pic"; filename="photo.jpg"
Content-Type: image/jpeg
[Binary Data]
----Boundary--
```
 
**JSON** (`application/json`)
```http
POST /api/user HTTP/1.1
Content-Type: application/json
 
{
  "name": "Aleksandra",
  "age": 27,
  "country": "US"
}
```
 
**XML** (`application/xml`)
```http
POST /api/user HTTP/1.1
Content-Type: application/xml
 
<user>
  <name>Aleksandra</name>
  <age>27</age>
  <country>US</country>
</user>
```
 
---
 
### HTTP Responses
 
#### Status Line Structure
 
```
HTTP/version STATUS_CODE Reason Phrase
```
 
**Example:** `HTTP/1.1 200 OK`
 
#### Status Code Ranges
 
| Range | Category | Meaning |
|---|---|---|
| `100–199` | Informational | Request received, continue sending |
| `200–299` | Success | Request processed successfully |
| `300–399` | Redirection | Resource has moved — follow the new location |
| `400–499` | Client Error | Problem with the request itself |
| `500–599` | Server Error | Server failed to process a valid request |
 
#### Common Status Codes
 
| Code | Reason | Meaning |
|---|---|---|
| `100` | Continue | Server ready for rest of request |
| `200` | OK | Success — resource returned |
| `301` | Moved Permanently | Resource at new permanent URL |
| `404` | Not Found | Resource doesn't exist at this URL |
| `500` | Internal Server Error | Server-side failure |
 
#### Common Response Headers
 
| Header | Example | Purpose |
|---|---|---|
| `Date` | `Date: Fri, 23 Aug 2024 10:43:21 GMT` | When the response was generated |
| `Content-Type` | `Content-Type: text/html; charset=utf-8` | Format of the response body |
| `Server` | `Server: nginx` | Server software (often obscured for security) |
| `Set-Cookie` | `Set-Cookie: sessionId=abc123` | Instructs client to store a cookie |
| `Cache-Control` | `Cache-Control: max-age=600` | How long clients may cache this response |
| `Location` | `Location: /index.html` | Target URL for redirections (3xx) |
 
> **Cookie security flags:** Always set `HttpOnly` (blocks JS access) and `Secure` (HTTPS only) on sensitive cookies.
> **Location header:** Validate server-side — unvalidated redirects enable open redirect attacks.
 
---
 
### HTTP Security Headers
 
Security headers are response headers that instruct the browser to enforce specific protections. Use [securityheaders.io](https://securityheaders.io) to audit any site.
 
#### Content-Security-Policy (CSP)
 
Defines which sources (domains) the browser is allowed to load content from — the primary defence against **XSS (Cross-Site Scripting)** attacks.
 
```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.tryhackme.com; style-src 'self'
```
 
| Directive | Value | Meaning |
|---|---|---|
| `default-src` | `'self'` | Default: only load from the same domain |
| `script-src` | `'self' https://cdn.tryhackme.com` | Scripts: same domain + trusted CDN |
| `style-src` | `'self'` | Stylesheets: same domain only |
 
> `'self'` is a special keyword meaning the same origin as the page being served.
 
#### Strict-Transport-Security (HSTS)
 
Forces all connections to the site to use HTTPS — even if the user types `http://`.
 
```http
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```
 
| Directive | Meaning |
|---|---|
| `max-age=63072000` | Enforce HTTPS for 2 years (in seconds) |
| `includeSubDomains` | Also apply to all subdomains |
| `preload` | Allow browser vendors to hardcode this domain as HTTPS-only before first visit |
 
#### X-Content-Type-Options
 
Prevents browsers from **MIME-sniffing** — guessing the content type of a response rather than trusting the `Content-Type` header. Stops certain content injection attacks.
 
```http
X-Content-Type-Options: nosniff
```
 
| Directive | Meaning |
|---|---|
| `nosniff` | Browser must use the declared `Content-Type`, not guess |
 
#### Referrer-Policy
 
Controls how much information the browser sends in the `Referer` header when navigating to another page — protecting user privacy and potentially sensitive URL parameters.
 
```http
Referrer-Policy: strict-origin-when-cross-origin
```
 
| Directive | Behavior |
|---|---|
| `no-referrer` | Never send any referrer information |
| `same-origin` | Send referrer only for same-origin navigations |
| `strict-origin` | Send only the origin (not full URL) when protocol stays the same (HTTPS→HTTPS) |
| `strict-origin-when-cross-origin` | Full URL for same-origin; origin only for cross-origin HTTPS; nothing for HTTP downgrades |
 
---
 
### Security Header Summary
 
| Header | Protects Against |
|---|---|
| `Content-Security-Policy` | XSS, content injection |
| `Strict-Transport-Security` | Downgrade attacks, unencrypted connections |
| `X-Content-Type-Options` | MIME-sniffing attacks |
| `Referrer-Policy` | Referrer information leakage |
 
---
