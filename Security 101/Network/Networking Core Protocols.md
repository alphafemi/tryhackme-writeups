Application Layer Protocols — DNS, HTTP, FTP, SMTP, POP3 & IMAP
 
This section covers the text-based application-layer protocols that power everyday internet use — domain resolution, web browsing, file transfer, and email. Understanding them at the command level reveals what graphical clients abstract away.
 
---
 
### DNS — Domain Name System
 
**Layer:** Application (OSI Layer 7)
**Ports:** UDP 53 (primary), TCP 53 (fallback/zone transfers)
 
DNS resolves human-readable domain names into machine-readable IP addresses — so browsers can locate servers without anyone memorizing numeric addresses.
 
#### Core DNS Record Types
 
| Record | Purpose | Example |
|---|---|---|
| **A** | Maps hostname → IPv4 address | `example.com → 93.184.215.14` |
| **AAAA** | Maps hostname → IPv6 address | `example.com → 2606:2800:21f:...` |
| **CNAME** | Maps domain → another domain | `www.example.com → example.com` |
| **MX** | Identifies mail server(s) for a domain | `example.com → mail.example.com` |
 
> When you type a URL in a browser, an **A record** lookup occurs. When sending email to `user@example.com`, the mail server queries the **MX record**.
 
#### DNS Lookup in Practice
 
```bash
nslookup www.example.com
```
 
```
Server:  127.0.0.53
Address: 127.0.0.53#53
 
Non-authoritative answer:
Name:    www.example.com
Address: 93.184.215.14          (A record — IPv4)
Address: 2606:2800:21f:cb07:... (AAAA record — IPv6)
```
 
#### Packet-Level View (tshark)
 
```
1  192.168.66.89 → 192.168.66.1  DNS  Query A    www.example.com
2  192.168.66.1  → 192.168.66.89 DNS  Response A  93.184.215.14
3  192.168.66.89 → 192.168.66.1  DNS  Query AAAA  www.example.com
4  192.168.66.1  → 192.168.66.89 DNS  Response AAAA 2606:2800:...
```
 
> Modern resolvers typically query **both A and AAAA** records in parallel. DNS responses come from the configured DNS server (often the local router, which relays to an upstream provider).
 
#### WHOIS — Domain Registration Records
 
WHOIS records identify who registered a domain name, including registrant name, address, email, registration date, and expiry.
 
```bash
whois example.com
```
 
```
Domain Name: EXAMPLE.COM
Creation Date: 1993-04-02T00:00:00Z
Registrar: GoDaddy.com, LLC
Registrant Name: Registration Private        ← privacy protection active
Registrant Organization: Domains By Proxy, LLC
```
 
> Domain owners can use **privacy protection services** to mask their personal contact details from public WHOIS lookups. WHOIS is pronounced "who is" — it is not an acronym.
 
---
 
### HTTP & HTTPS — HyperText Transfer Protocol
 
**Layer:** Application (Layer 7)
**Ports:** TCP 80 (HTTP), TCP 443 (HTTPS), TCP 8080/8443 (alternatives)
 
HTTP defines how browsers communicate with web servers. HTTPS adds TLS/SSL encryption.
 
#### HTTP Methods
 
| Method | Purpose |
|---|---|
| `GET` | Retrieve a resource (HTML page, image, etc.) |
| `POST` | Submit new data to the server (forms, file uploads) |
| `PUT` | Create a new resource or overwrite an existing one |
| `DELETE` | Remove a specified resource |
 
#### Manual HTTP via Telnet
 
```bash
telnet 10.112.172.196 80
```
 
```
GET / HTTP/1.1
Host: anything
                        ← blank line terminates the request
HTTP/1.1 200 OK
Content-Type: text/html
Last-Modified: ...
...
```
 
> The blank line after `Host:` is **required by HTTP specification** to signal end of headers. To request a specific file: `GET /file.html HTTP/1.1`.
 
---
 
### FTP — File Transfer Protocol
 
**Layer:** Application (Layer 7)
**Ports:** TCP 21 (control), separate TCP connection for data transfer
 
FTP is optimized for transferring files and can achieve higher throughput than HTTP for file transfer under comparable conditions.
 
#### FTP Commands
 
| Command | Purpose |
|---|---|
| `USER <username>` | Authenticate with username |
| `PASS <password>` | Authenticate with password |
| `RETR <filename>` | Download file from server |
| `STOR <filename>` | Upload file to server |
| `LIST` | List directory contents (server-side equivalent of `ls`) |
 
#### FTP Session Example
 
```bash
ftp 10.112.172.196
```
 
```
Connected to 10.112.172.196.
220 (vsFTPd 3.0.5)
Name: anonymous
331 Please specify the password.
Password:
230 Login successful.
 
ftp> ls
150 Here comes the directory listing.
coffee.txt
flag.txt
tea.txt
226 Directory send OK.
 
ftp> type ascii
200 Switching to ASCII mode.
 
ftp> get coffee.txt
226 Transfer complete.
 
ftp> quit
221 Goodbye.
```
 
> FTP uses **two separate TCP connections**: the **control connection** (port 21) for commands, and a **separate data connection** for each directory listing and file transfer. This is why `ls` and file downloads each create a new connection.
 
> `anonymous` login allows access without a real account — commonly enabled for public file distribution, but a security risk if misconfigured.
 
---
 
### SMTP — Simple Mail Transfer Protocol
 
**Layer:** Application (Layer 7)
**Port:** TCP 25 (default)
 
SMTP defines how mail clients **send** email to mail servers, and how mail servers relay email between each other. Analogous to handing a package to a post office.
 
#### SMTP Commands
 
| Command | Purpose |
|---|---|
| `HELO` / `EHLO` | Initiate SMTP session (EHLO for extended features) |
| `MAIL FROM: <addr>` | Specify sender's email address |
| `RCPT TO: <addr>` | Specify recipient's email address |
| `DATA` | Begin sending message content |
| `.` | Single period on its own line — signals end of message body |
| `QUIT` | End the SMTP session |
 
#### SMTP Session via Telnet
 
```bash
telnet 10.112.172.196 25
```
 
```
220 example.thm ESMTP Exim 4.95
HELO client.thm
250 example.thm Hello client.thm
MAIL FROM: <user@client.thm>
250 OK
RCPT TO: <strategos@server.thm>
250 Accepted
DATA
354 Enter message, ending with "." on a line by itself
From: user@client.thm
To: strategos@server.thm
Subject: Telnet email
 
Hello. I am using telnet to send you an email!
.
250 OK id=1sMrpq-0001Ah-UT
QUIT
221 example.thm closing connection
```
 
> SMTP transmits everything in **plain text** including credentials — use SMTPS (port 465) or STARTTLS (port 587) in production for encryption.
 
---
 
### POP3 — Post Office Protocol v3
 
**Layer:** Application (Layer 7)
**Port:** TCP 110 (default)
 
POP3 defines how mail clients **retrieve** email from a mail server. Analogous to checking your physical letterbox — messages are typically **downloaded and deleted** from the server, suited to single-device access.
 
#### POP3 Commands
 
| Command | Purpose |
|---|---|
| `USER <username>` | Identify the user |
| `PASS <password>` | Authenticate with password |
| `STAT` | Return message count and total mailbox size |
| `LIST` | List all messages and their individual sizes |
| `RETR <n>` | Retrieve (download) message number `n` |
| `DELE <n>` | Mark message `n` for deletion |
| `QUIT` | End session, applying any pending deletions |
 
#### POP3 Session via Telnet
 
```bash
telnet 10.112.172.196 110
```
 
```
+OK Dovecot ready.
USER strategos
+OK
PASS
+OK Logged in.
STAT
+OK 3 1264          ← 3 messages, 1264 bytes total
LIST
+OK 3 messages:
1 407
2 412
3 445
RETR 3
+OK 445 octets
From: user@client.thm
To: strategos@server.thm
Subject: Telnet email
 
Hello. I am using telnet to send you an email!
.
QUIT
+OK Logging out.
```
 
> ⚠️ POP3 transmits credentials and message content in **plain text** — anyone capturing the network traffic can read both the password and email content.
 
---
 
### IMAP — Internet Message Access Protocol
 
**Layer:** Application (Layer 7)
**Port:** TCP 143 (default)
 
IMAP allows mail clients to **synchronize** with a mail server — messages remain on the server and changes (read, moved, deleted) are reflected across all connected devices. The appropriate choice when accessing email from multiple devices.
 
#### POP3 vs. IMAP
 
| Feature | POP3 | IMAP |
|---|---|---|
| Message storage | Downloaded to device, deleted from server | Stays on server |
| Multi-device sync | Not supported | Full synchronization |
| Server storage usage | Minimal (messages removed) | Higher (messages retained) |
| Best for | Single-device access | Multi-device access |
 
#### IMAP Commands
 
| Command | Purpose |
|---|---|
| `LOGIN <user> <pass>` | Authenticate |
| `SELECT <mailbox>` | Open a mailbox folder (e.g., INBOX) |
| `FETCH <n> body[]` | Retrieve message `n` (headers + body) |
| `MOVE <set> <mailbox>` | Move messages to another folder |
| `COPY <set> <mailbox>` | Copy messages to another folder |
| `LOGOUT` | End the IMAP session |
 
> IMAP commands are **prefixed with a client-generated tag** (e.g., `A`, `B`, `C`, `D`) so the server can match responses to their corresponding requests — enabling concurrent command handling.
 
#### IMAP Session via Telnet
 
```bash
telnet 10.10.41.192 143
```
 
```
* OK Dovecot ready.
A LOGIN strategos
A OK Logged in
B SELECT inbox
* 4 EXISTS
B OK Select completed
C FETCH 3 body[]
* 3 FETCH (BODY[] {445}
From: user@client.thm
To: strategos@server.thm
Subject: Telnet email
 
Hello. I am using telnet to send you an email!
)
C OK Fetch completed
D LOGOUT
D OK Logout completed
```
 
---
 
### Protocol Port Reference
 
| Protocol | Full Name | Transport | Default Port | Purpose |
|---|---|---|---|---|
| **Telnet** | Teletype Network | TCP | 23 | Remote terminal (unencrypted) |
| **DNS** | Domain Name System | UDP/TCP | 53 | Domain-to-IP resolution |
| **HTTP** | HyperText Transfer Protocol | TCP | 80 | Web page retrieval |
| **HTTPS** | HTTP Secure | TCP | 443 | Encrypted web traffic |
| **FTP** | File Transfer Protocol | TCP | 21 (control) | File transfer |
| **SMTP** | Simple Mail Transfer Protocol | TCP | 25 | Sending email |
| **POP3** | Post Office Protocol v3 | TCP | 110 | Retrieving email (single device) |
| **IMAP** | Internet Message Access Protocol | TCP | 143 | Retrieving/syncing email (multi-device) |
 
---
 
### Protocol Role Summary
 
```
DNS     ─── Resolve domain names before any connection is made
HTTP    ─── Web browsing (browser ↔ web server)
FTP     ─── File transfer (client ↔ file server)
SMTP    ─── Sending email (mail client → mail server, or server → server)
POP3    ─── Retrieving email (download and remove from server)
IMAP    ─── Retrieving email (keep on server, sync across devices)
```
 
---
