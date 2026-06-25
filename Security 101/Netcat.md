 Netcat (nc)
 
### Overview
 
**Netcat** (`nc`) is a general-purpose networking utility for creating and inspecting TCP and UDP connections. It can initiate connections, listen for incoming ones, transfer data, and scan for open ports — across both IPv4 and IPv6.
 
```
nc [-46DdhklnrStUuvz] [-i interval] [-p source_port] [-s source_ip_address]
   [-T ToS] [-w timeout] [-X proxy_protocol] [-x proxy_address[:port]]
   [hostname] [port[s]]
```
 
> Often described as the **"Swiss Army knife" of networking** — its flexibility makes it useful for legitimate diagnostics, scripting, and testing, while the same capabilities are frequently relevant in penetration testing and CTF contexts.
 
### Why Netcat Over Telnet?
 
| Netcat | Telnet |
|---|---|
| Scripts cleanly | Less script-friendly |
| Separates errors to **stderr** | Mixes errors into **stdout** |
| Supports TCP **and** UDP | TCP only |
 
### Common Use Cases
 
- Simple TCP proxies
- Shell-script-based HTTP clients/servers
- Network daemon (service) testing
- A SOCKS/HTTPS `ProxyCommand` for `ssh`
- Port scanning
- Basic file transfer between hosts
- Setting up simple listeners for connection testing
---
 
### Option Reference
 
| Option | Description |
|---|---|
| `-4` | Force IPv4 only |
| `-6` | Force IPv6 only |
| `-D` | Enable debugging on the socket |
| `-d` | Do not attempt to read from stdin |
| `-h` | Print help |
| `-i interval` | Delay (in seconds) between lines sent/received, and between connections to multiple ports |
| `-k` | Keep listening for further connections after the current one ends (**requires `-l`**) |
| `-l` | Listen for an incoming connection rather than initiating one (cannot combine with `-p`, `-s`, or `-z`; ignores `-w`) |
| `-n` | Skip DNS/service name resolution — use raw IPs and port numbers only |
| `-p source_port` | Specify the source port to use |
| `-r` | Choose source/destination ports randomly instead of sequentially |
| `-s source_ip_address` | Specify the source interface IP to send from (cannot combine with `-l`) |
| `-T ToS` | Set IP Type of Service (`lowdelay`, `throughput`, `reliability`, or a hex value) |
| `-t` | Send RFC 854 `DON'T`/`WON'T` responses — enables scripting Telnet sessions |
| `-U` | Use UNIX-domain sockets |
| `-u` | Use UDP instead of the default TCP |
| `-v` | Verbose output |
| `-w timeout` | Close the connection if idle for more than `timeout` seconds (no effect with `-l`; default is no timeout) |
| `-X proxy_protocol` | Proxy protocol to use: `4` (SOCKSv4), `5` (SOCKSv5, default), or `connect` (HTTPS proxy) |
| `-x proxy_address[:port]` | Connect via a proxy at the given address/port (defaults: `1080` for SOCKS, `3128` for HTTPS) |
| `-z` | Scan only — check for listening services without sending data (cannot combine with `-l`) |
 
---
 
### Practical Command Patterns
 
#### Listening for a Connection
 
```bash
nc -lvp 4444
```
| Flag | Effect |
|---|---|
| `-l` | Listen mode |
| `-v` | Verbose output |
| `-p 4444` | Bind to local port `4444` |
 
#### Connecting to a Remote Host
 
```bash
nc -v 192.168.1.10 80
```
> Initiates a TCP connection to port `80` on `192.168.1.10`, with verbose output.
 
#### Banner Grabbing / Service Testing
 
```bash
nc -nv 192.168.1.10 22
```
> Connects without DNS lookups (`-n`), often used to capture a service banner (e.g., SSH version string) for reconnaissance.
 
#### Port Scanning
 
```bash
nc -zv 192.168.1.10 1-1000
```
| Flag | Effect |
|---|---|
| `-z` | Scan-only mode (no data sent) |
| `-v` | Verbose output |
| `1-1000` | Port range to scan |
 
#### UDP Mode
 
```bash
nc -u -v 192.168.1.10 53
```
> Connects using UDP (`-u`) instead of the TCP default — relevant for services like DNS.
 
#### Persistent Listener (Multiple Connections)
 
```bash
nc -lkvp 4444
```
> `-k` keeps the listener active to accept new connections after the current one closes (requires `-l`).
 
#### Connecting Through a Proxy
 
```bash
nc -X connect -x 192.168.1.5:3128 example.com 443
```
> Routes the connection through an HTTPS proxy (`-X connect`) located at `192.168.1.5:3128`.
 
---
 
### Quick Reference
 
| Goal | Command Pattern |
|---|---|
| Listen on a port | `nc -lvp <port>` |
| Connect to a host/port | `nc -v <host> <port>` |
| Scan a port range | `nc -zv <host> <port_range>` |
| Use UDP | `nc -u <host> <port>` |
| Skip DNS resolution | `nc -n <host> <port>` |
| Keep listener alive after disconnect | `nc -lkvp <port>` |
| Set a connection timeout | `nc -w <seconds> <host> <port>` |
| Connect via SOCKS5 proxy | `nc -X 5 -x <proxy>:<port> <host> <port>` |
 
---
