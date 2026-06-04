Understood. Below is the **merged, fully expanded Day 3** that combines the previous two responses into one single, complete, ultra‑deep, hands‑on chapter. Nothing from your earlier content was removed; everything has been preserved and amplified. All original commands, configuration snippets, captures, theory, and practical guidance are included — plus additional detail, examples, and forensic steps so you can run the lab, capture artifacts, and reason about every outcome.

I kept the format you asked for: **no bullet lists except the TOC**, long narrative paragraphs, explicit code blocks for every actionable step, full zone and server configs, `dig`/`tcpdump`/`curl`/`openssl`/`tshark` examples, proxy and request‑smuggling notes, TLS inspection, DNSSEC pointers, artifact manifest examples, and investigative heuristics. This is the single authoritative Day 3 you asked for.

---

# 🧨 DAY 3 — DNS, HTTP, HTTPS, TLS, and Proxies (Complete Ultra‑Deep Merge)

### _“Today you learn the languages machines speak when they want to talk like humans.”_

### _Estimated time: 2 hours for the core walkthrough; mastery requires repeated practice_

---

# 🧭 Table of Contents

- [[# 1. Why Day 3 Matters — language meaning and trust]]
- [[# 2. DNS — the internet phonebook and its weakest link 🌐]]
- [[# 3. DNS internals — records zones resolvers and caches with examples]]
- [[# 4. DNS attacks and defenses — poisoning rebinding takeover and DNSSEC]]
- [[# 5. HTTP — the language of the web in full sentences 🗣️]]
- [[# 6. HTTP semantics and state — methods status codes cookies and sessions with commands]]
- [[# 7. HTTP headers — the control plane of the web examples and pitfalls]]
- [[# 8. HTTPS and TLS — identity encryption and integrity 🔒]]
- [[# 9. TLS internals — handshakes key exchange ciphers and pitfalls 🤝]]
- [[# 10. Proxies and intermediaries — rewriting reality 🕵️‍♂️]]
- [[# 11. Lab — build observe and manipulate DNS HTTP HTTPS TLS and proxies 🧪]]
- [[# 12. Packet and trace analysis — what to look for in Wireshark and logs 🔍]]
- [[# 13. Artifact strategy — manifests timestamps and reproducible evidence 📁]]
- [[# 14. Failure modes debugging and investigative heuristics 🧠]]
- [[# 15. Reflection prompts — turning practice into intuition 🎯]]

---

## 1. Why Day 3 Matters — language meaning and trust

Day 3 is the layer where packets stop being raw motion and become **messages**. The protocols you study today are the grammar and vocabulary of the web: DNS names map to addresses, HTTP frames requests and responses, TLS binds identity to keys and encrypts the conversation, and proxies mediate, rewrite, and sometimes betray that conversation. If you can read and manipulate this layer, you can explain why a login works, why a cookie persists, why a redirect sends a user somewhere else, and why a seemingly harmless header can open a door.

This day is crucial because nearly every web vulnerability is an emergent property of how these protocols interact. SQL injection is not just about SQL; it is about how HTTP carries user input to a database. XSS is not just about JavaScript; it is about how responses are rendered and cached. SSRF is not just about making requests; it is about how DNS resolution, proxies, and server‑side HTTP clients behave. TLS is not just encryption; it is the ritual that binds identity to keys and that can be subverted by misconfiguration or compromised trust anchors.

You will not learn this layer by memorizing commands. You will learn it by building, observing, and reasoning about the full stack: name resolution, request formation, header semantics, certificate chains, cipher negotiation, proxy rewriting, and the subtle differences between what a browser does and what a server expects. This chapter gives you the vocabulary, the mental models, and the practical steps to see the web as a system of meaning.

---

## 2. DNS — the internet phonebook and its weakest link 🌐

DNS translates human names into routable addresses and provides other service metadata. It is the first step in almost every network interaction. When you type a domain into a browser, DNS is the mechanism that turns that human label into an IP address the network can route to. DNS is also used for service discovery, mail routing, load balancing, and more. The protocol is simple in concept but complex in practice: it is distributed, hierarchical, and heavily cached.

At its core, DNS is unauthenticated by default. A resolver asks a question and accepts the first plausible answer unless additional protections are in place. That design choice made DNS fast and scalable in the early internet, but it also left it vulnerable to spoofing and cache poisoning. Modern mitigations exist, but the legacy behavior still shapes how attackers operate.

Because DNS is the first step in nearly every connection, it is the first trust boundary. If you control name resolution, you can redirect traffic, intercept credentials, or trick servers into contacting attacker infrastructure. DNS is used by clients, servers, and cloud services; its impact is broad: a poisoned resolver can turn an internal-only hostname into a public endpoint, and a misissued certificate for a DNS name can enable impersonation.

---

## 3. DNS internals — records zones resolvers and caches with examples

DNS zones are text files that define records for a domain. The canonical zone file contains an SOA, NS records, and resource records. The most common records you will encounter are A and AAAA for IPv4 and IPv6 addresses, CNAME for canonical name aliases, MX for mail exchange, TXT for arbitrary text (often used for SPF, DKIM, and verification tokens), SRV for service discovery, and NS for delegations. Each zone is a contiguous range of names managed by authoritative servers. The root zone delegates to TLDs, TLDs delegate to domain owners, and authoritative servers answer for their zones.

Below is a minimal authoritative zone file for `test.local` you can use in a lab. Save it as `/etc/bind/db.test.local` on your Ubuntu authoritative server.

```text
$TTL 86400
@   IN  SOA ns1.test.local. admin.test.local. (
        20260603 ; serial YYYYMMDD
        3600     ; refresh
        1800     ; retry
        604800   ; expire
        86400 )  ; negative cache TTL

    IN  NS  ns1.test.local.
ns1 IN  A   192.168.20.10
test IN A   192.168.20.10
www  IN CNAME test
```

Add the zone to BIND’s configuration by editing `/etc/bind/named.conf.local` and adding:

```text
zone "test.local" {
    type master;
    file "/etc/bind/db.test.local";
};
```

Validate the zone and reload BIND:

```bash
sudo named-checkzone test.local /etc/bind/db.test.local
sudo systemctl restart bind9
sudo systemctl status bind9
```

From a client (Kali), query the authoritative server directly and observe the response:

```bash
dig @192.168.20.10 test.local +noall +answer
dig @192.168.20.10 www.test.local +noall +answer
dig @192.168.20.10 ns1.test.local +noall +authority
```

Use `dig +trace` to follow the full resolution path from the root down:

```bash
dig +trace test.local
```

Resolvers cache answers. To inspect the resolver behavior on a Linux client, check `systemd-resolve --status` or `resolvectl status` depending on your distro. To flush caches:

```bash
sudo systemd-resolve --flush-caches
# or
sudo resolvectl flush-caches
```

DNS transport is usually UDP/53; large responses or DNSSEC use TCP/53. To capture DNS traffic on the network interface:

```bash
sudo tcpdump -i eth0 port 53 -w ~/day3_lab/pcaps/dns_$(date -u +"%Y%m%dT%H%M%SZ").pcap
```

This section shows the concrete files, commands, and captures you need to understand how authoritative servers, resolvers, and caches interact. Keep in mind that TTLs, serial numbers, and SOA parameters control propagation and caching behavior; misconfigurations here are common sources of stale or inconsistent resolution.

---

## 4. DNS attacks and defenses — poisoning rebinding takeover and DNSSEC

DNS cache poisoning is the classic attack: an attacker injects a false answer into a resolver’s cache so that subsequent clients receive the malicious mapping. Historically this was done by predicting query IDs and source ports. Modern resolvers mitigate this with randomization and source port entropy, but weaknesses remain, especially in embedded resolvers or misconfigured servers.

To simulate poisoning in a lab, run a malicious resolver that returns forged answers and point a client at it via `/etc/resolv.conf` or `resolvectl`. Use `dig` to confirm the forged mapping and capture the traffic with `tcpdump`. Example of running a simple malicious responder in a controlled lab is beyond this document’s scope, but the principle is: control the resolver and you control resolution.

DNS rebinding leverages the browser’s trust in a hostname. An attacker registers a domain and serves a short TTL that initially resolves to the attacker’s server. The attacker’s page runs JavaScript in the victim’s browser and then causes the domain to resolve to an internal IP. Because the browser still considers the origin the same hostname, the JavaScript can make requests to internal services, potentially exposing internal APIs or metadata endpoints. Defenses include proper CORS, restricting internal services to non‑routable hostnames, and using DNS pinning mitigations.

Subdomain takeover occurs when a DNS record points to an external service that has been deprovisioned. For example, a CNAME pointing to a cloud storage bucket that has been deleted can be claimed by an attacker, who then serves content for the original subdomain. Prevent this by inventorying DNS records, removing stale delegations, and ensuring that external services are not referenced after deprovisioning.

DNSSEC is the cryptographic defense for DNS. It provides origin authentication and integrity for DNS records by signing zones with public key cryptography. When properly deployed, DNSSEC prevents many forms of spoofing and cache poisoning. However, DNSSEC deployment is uneven, and operational complexity can lead to misconfigurations that break resolution. Signing a zone involves generating keys and adding DNSKEY and RRSIG records; BIND and `dnssec-keygen`/`dnssec-signzone` automate this. Example commands to sign a zone:

```bash
dnssec-keygen -a RSASHA256 -b 2048 -n ZONE test.local
dnssec-signzone -A -3 $(head -c 1000 /dev/urandom | sha1sum | cut -d' ' -f1) -N INCREMENT -o test.local /etc/bind/db.test.local
```

Monitor DNS logs (`/var/log/syslog` or BIND’s logging) for anomalies and use public monitoring to detect unexpected certificate issuance or delegation changes. Use `dig +dnssec` to inspect DNSSEC signatures:

```bash
dig +dnssec test.local @192.168.20.10
```

---

## 5. HTTP — the language of the web in full sentences 🗣️

HTTP is a text protocol that reads like a conversation. A client issues a request line, headers, and an optional body. The server replies with a status line, headers, and an optional body. The request line contains the method, path, and version. Methods encode intent: GET retrieves, POST submits, PUT replaces, DELETE removes, PATCH modifies partially, OPTIONS queries capabilities, and HEAD fetches headers only. The path can include query parameters that carry user input. The Host header disambiguates virtual hosts on the same IP. Headers carry metadata and control behavior. The body carries the payload, which can be form data, JSON, XML, or binary.

To observe HTTP in practice, install Nginx and request a page with `curl` while capturing traffic:

```bash
sudo apt update
sudo apt install -y nginx
sudo systemctl enable --now nginx
echo "Hello from test.local" | sudo tee /var/www/html/index.html

sudo tcpdump -i eth0 tcp port 80 -w ~/day3_lab/pcaps/http_$(date -u +"%Y%m%dT%H%M%SZ").pcap &
echo $! > ~/day3_lab/artifacts/tcpdump_http_pid.txt

curl -v http://192.168.20.10/ --resolve test.local:80:192.168.20.10 | tee ~/day3_lab/scans/curl_http_$(date -u +"%Y%m%dT%H%M%SZ").txt

kill $(cat ~/day3_lab/artifacts/tcpdump_http_pid.txt) || true
```

Open the pcap in Wireshark and follow the TCP stream to see the raw HTTP request and response. The `-v` flag in `curl` prints request and response headers to the terminal for quick inspection. HTTP versions matter: HTTP/1.1 introduced persistent connections and chunked transfer encoding; HTTP/2 introduced multiplexing and header compression; HTTP/3 uses QUIC. Each version changes framing and intermediary behavior and has security and performance implications.

---

## 6. HTTP semantics and state — methods status codes cookies and sessions with commands

HTTP is stateless by design. To create state, servers use cookies and sessions. A server sets a cookie with `Set-Cookie` and the browser sends it back with `Cookie`. Cookie attributes control behavior: `Secure` restricts to HTTPS, `HttpOnly` prevents JavaScript access, `SameSite` controls cross‑site sending, `Domain` and `Path` scope the cookie.

To set a cookie in Nginx for testing, add a header in the server block:

```nginx
add_header Set-Cookie "sessionid=abc123; HttpOnly; Secure; SameSite=Lax";
```

Reload Nginx and request the page with `curl` to see the header:

```bash
curl -v http://test.local/ --resolve test.local:80:192.168.20.10
```

For session testing, create a simple backend that sets and reads a session cookie (Python Flask example):

```python
# app.py
from flask import Flask, session
app = Flask(__name__)
app.secret_key = 'devsecret'

@app.route('/')
def index():
    count = session.get('count', 0) + 1
    session['count'] = count
    return f"Visit count: {count}"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

Run the app and proxy it through Nginx or access directly. Use `curl -c` and `-b` to persist cookies:

```bash
curl -c cookies.txt -v http://192.168.20.10:5000/
curl -b cookies.txt -v http://192.168.20.10:5000/
```

Status codes communicate results: 2xx success, 3xx redirection, 4xx client error, 5xx server error. Use `curl -I` to fetch headers only:

```bash
curl -I http://192.168.20.10/
```

Form encoding matters. `application/x-www-form-urlencoded` is the classic form encoding. `multipart/form-data` is used for file uploads. `application/json` is ubiquitous for APIs. Each encoding has parsing rules and edge cases that can be abused. For example, file upload parsers can be tricked into writing files to unexpected locations, and JSON parsers can be coerced into unexpected types.

Understanding how cookies and sessions are issued, scoped, and transmitted is essential for reasoning about CSRF, session fixation, and cookie theft.

---

## 7. HTTP headers — the control plane of the web examples and pitfalls

Headers are the levers that control caching, content negotiation, authentication, cross‑origin access, and more. They are also the primary attack surface for many web vulnerabilities.

The Host header is used for virtual hosting. If an application uses the Host header to construct links or select behavior, an attacker who controls Host can cause the application to generate links pointing to attacker domains, poison caches keyed by host, or bypass host‑based access controls. Demonstrate Host header manipulation with `curl`:

```bash
curl -v -H "Host: attacker.example" http://192.168.20.10/
```

If the server or proxy trusts Host blindly, it may generate absolute URLs or password reset links using the attacker host.

CORS headers control cross‑origin access. To test CORS behavior, use `curl` to simulate preflight and actual requests:

```bash
curl -v -X OPTIONS http://api.test.local/resource -H "Origin: http://evil.com" -H "Access-Control-Request-Method: GET"
curl -v http://api.test.local/resource -H "Origin: http://evil.com"
```

If `Access-Control-Allow-Origin` is `*` or reflects the origin unsafely, the API may be readable by malicious sites. `Access-Control-Allow-Credentials` combined with permissive origins is particularly dangerous because it allows cookies to be sent and responses to be read.

Cache headers determine how responses are stored by browsers and proxies. `Cache-Control`, `Expires`, `ETag`, and `Vary` are the main controls. If a response containing user‑specific data is cached publicly due to misconfigured headers, other users can receive that data. Cache poisoning attacks exploit proxies that key caches on the wrong set of headers or that do not vary responses correctly. Example Nginx snippet to set caching and vary:

```nginx
add_header Cache-Control "public, max-age=60";
add_header Vary "Accept-Encoding, Cookie";
```

Security headers like `Content-Security-Policy`, `Strict-Transport-Security`, `X-Frame-Options`, and `X-Content-Type-Options` provide defense in depth. Add them in Nginx:

```nginx
add_header Content-Security-Policy "default-src 'self'; script-src 'self'";
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";
add_header X-Frame-Options "DENY";
add_header X-Content-Type-Options "nosniff";
```

Test CSP by injecting script tags and observing whether the browser blocks them. Test HSTS by checking that the browser upgrades HTTP to HTTPS after the header is seen.

Headers are where cache poisoning begins, where host header injection begins, where CORS misconfigurations begin, and where request smuggling can be triggered. If you do not read headers, you are missing half the conversation.

---

## 8. 🔒 HTTPS and TLS — identity encryption and integrity

HTTPS is HTTP over TLS. It provides confidentiality and integrity and ties the connection to an identity via certificates. A certificate binds a public key to a name and is signed by a CA. The browser validates the chain of trust up to a trusted root. Certificates contain SANs, key usage, and validity periods. Misissued certificates, weak keys, or incorrect SANs can break trust or enable impersonation.

Generate a self‑signed certificate for `test.local`:

```bash
sudo mkdir -p /etc/nginx/ssl
openssl req -x509 -newkey rsa:4096 -sha256 -days 365 \
  -nodes -keyout /etc/nginx/ssl/test.local.key -out /etc/nginx/ssl/test.local.crt \
  -subj "/CN=test.local"
```

Configure Nginx to use the cert and prefer TLS 1.3:

```nginx
ssl_protocols TLSv1.3 TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers 'TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256';
ssl_certificate /etc/nginx/ssl/test.local.crt;
ssl_certificate_key /etc/nginx/ssl/test.local.key;
```

Reload Nginx and test with `curl` and `openssl`:

```bash
curl -vk https://test.local/ --resolve test.local:443:192.168.20.10
openssl s_client -connect 192.168.20.10:443 -servername test.local -showcerts
```

`openssl s_client` prints the certificate chain and negotiated cipher. Use `-tls1_2` or `-tls1_3` to force versions for testing:

```bash
openssl s_client -connect 192.168.20.10:443 -tls1_2 -servername test.local
```

Certificate transparency and OCSP are additional layers. Use `openssl` and browser devtools to inspect OCSP stapling and certificate chains. If a certificate is self‑signed, the browser will warn; for lab purposes, import the self‑signed CA into the browser to avoid warnings.

HTTPS is not a guarantee of safety. The padlock indicates a TLS session and a certificate chain that the browser trusts; it does not guarantee the site is honest or that the configuration is secure. Phishing sites can and do use HTTPS.

---

## 9. TLS internals — handshakes key exchange ciphers and pitfalls 🤝

TLS handshakes negotiate protocol version, cipher suite, and keys. In TLS 1.3, the handshake is streamlined and uses ephemeral key exchange by default, providing forward secrecy. In TLS 1.2, RSA key exchange or DHE/ECDHE are common; prefer ECDHE for forward secrecy.

To inspect a TLS handshake in a pcap, capture port 443 traffic:

```bash
sudo tcpdump -i eth0 tcp port 443 -w ~/day3_lab/pcaps/tls_$(date -u +"%Y%m%dT%H%M%SZ").pcap
```

Open the pcap in Wireshark and filter `tls.handshake`. Look for `ClientHello`, `ServerHello`, `Certificate`, `ServerKeyExchange` (if present), `ClientKeyExchange`, and `Finished`. Note the cipher suite and whether `ECDHE` or `RSA` was used. For TLS 1.3, note the `supported_versions` and `key_share` extensions.

Key exchange methods matter. RSA key exchange relies on the server’s private key to decrypt a client‑generated secret. Ephemeral Diffie‑Hellman (DHE or ECDHE) provides forward secrecy: even if the server’s long‑term key is later compromised, past sessions remain secure because ephemeral keys were used. Forward secrecy is a critical property for modern deployments.

Cipher suites combine key exchange, authentication, encryption, and MAC algorithms. AEAD ciphers like AES‑GCM and ChaCha20‑Poly1305 provide both confidentiality and integrity in a single primitive and are preferred. Older constructions that separate encryption and MAC are more error prone.

TLS has many historical pitfalls. Compression combined with TLS enabled CRIME. Predictable IVs and CBC modes enabled BEAST and POODLE. Session resumption and renegotiation introduced complexities that were exploited. Many of these issues are addressed in modern protocol versions, but legacy servers and clients still exist.

Certificate validation is a frequent source of vulnerability. Clients must check the certificate chain, verify the hostname against the SANs, check validity periods, and consider revocation. Many applications implement these checks incorrectly or disable them for convenience, enabling trivial MITM attacks.

Downgrade attacks attempt to force a client and server to use weaker protocol versions or ciphers. TLS includes mechanisms to detect and prevent downgrades, but misconfigurations or legacy fallbacks can still be exploited.

Understanding TLS internals lets you interpret handshakes in packet captures, reason about the security properties of a connection, and identify misconfigurations that reduce the effective security of HTTPS.

---

## 10. 🕵️‍♂️ Proxies and intermediaries — rewriting reality

Proxies are everywhere. Corporate networks use forward proxies to control outbound traffic. CDNs and reverse proxies sit in front of web servers to cache content and absorb load. Load balancers distribute traffic. WAFs inspect and filter requests. API gateways enforce quotas and authentication. Each intermediary can modify requests and responses, add headers, terminate TLS, or re‑encrypt traffic.

When a proxy terminates TLS, it becomes a man in the middle by design. Corporate proxies install a trusted root so that clients accept the proxy’s certificates. This enables inspection but also creates a single point of trust that, if compromised, can be abused. Reverse proxies often use the Host header or SNI to route to backends. If the proxy trusts the Host header from the client, an attacker can manipulate routing or generate links with attacker‑controlled hosts.

Request smuggling is a class of attacks that exploits differences in how frontends and backends parse request boundaries. If a front end uses `Transfer-Encoding: chunked` and the backend uses `Content-Length`, an attacker can craft a request that the front end forwards differently than the backend interprets, allowing the attacker to smuggle a second request inside the first, bypassing security checks, poisoning caches, or hijacking connections. To test smuggling in a lab, craft raw TCP requests with `nc` and observe how each component parses them.

Caching proxies can be poisoned when responses are cached under keys that do not include all relevant request attributes. If a proxy caches a response that includes user‑specific content under a key that does not vary by cookie or authorization header, other users may receive that content. Cache poisoning can be used to serve malicious JavaScript, redirect users, or leak sensitive data.

Proxies also affect logging and forensics. The client IP seen by the backend may be the proxy’s IP unless `X-Forwarded-For` or similar headers are used. If these headers are spoofable, logs can be misleading. Understanding the proxy chain is essential for accurate incident response.

---

## 11. 🧪 Lab — build observe and manipulate DNS HTTP HTTPS TLS and proxies

This lab is intentionally comprehensive. You will build a small environment, instrument it, and exercise the full stack. The goal is not to memorize commands but to create artifacts you can analyze and to internalize the causal chain from name resolution to encrypted application data.

Prepare your workspace and artifact directories:

```bash
mkdir -p ~/day3_lab/{pcaps,scans,notes,screenshots,artifacts}
date -u > ~/day3_lab/scans/lab3_start_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

Step 1 — Install and configure BIND9 on Ubuntu as an authoritative server for test.local. Install and verify the service:

```bash
sudo apt update
sudo apt install -y bind9 bind9utils dnsutils
sudo systemctl enable --now bind9
sudo systemctl status bind9
```

Create `/etc/bind/db.test.local` with the zone file shown earlier and add the zone to `/etc/bind/named.conf.local`. Validate and reload:

```bash
sudo named-checkzone test.local /etc/bind/db.test.local
sudo systemctl restart bind9
```

Capture DNS traffic while querying:

```bash
sudo tcpdump -i eth0 port 53 -w ~/day3_lab/pcaps/dns_query_$(date -u +"%Y%m%dT%H%M%SZ").pcap &
echo $! > ~/day3_lab/artifacts/tcpdump_dns_pid.txt
dig @192.168.20.10 test.local +noall +answer | tee ~/day3_lab/scans/dig_test_local_$(date -u +"%Y%m%dT%H%M%SZ").txt
kill $(cat ~/day3_lab/artifacts/tcpdump_dns_pid.txt) || true
```

Step 2 — Install Nginx and serve a simple page. Capture HTTP traffic and inspect headers:

```bash
sudo apt install -y nginx
sudo systemctl enable --now nginx
echo "Hello from test.local" | sudo tee /var/www/html/index.html
```

From Kali, request the page and capture the HTTP exchange:

```bash
sudo tcpdump -i eth0 tcp port 80 -w ~/day3_lab/pcaps/http_get_$(date -u +"%Y%m%dT%H%M%SZ").pcap &
echo $! > ~/day3_lab/artifacts/tcpdump_http_pid.txt
curl -v http://192.168.20.10/ --resolve test.local:80:192.168.20.10 | tee ~/day3_lab/scans/curl_http_$(date -u +"%Y%m%dT%H%M%SZ").txt
kill $(cat ~/day3_lab/artifacts/tcpdump_http_pid.txt) || true
```

Open the pcap in Wireshark and inspect the GET request, Host header, response headers, and body. Save screenshots of the request and response headers.

Step 3 — Add TLS. Generate a self‑signed certificate for test.local and configure Nginx to serve HTTPS. Use a strong modern configuration: TLS 1.3 preferred, ECDHE key exchange, AEAD ciphers.

```bash
sudo mkdir -p /etc/nginx/ssl
openssl req -x509 -newkey rsa:4096 -sha256 -days 365 -nodes \
  -keyout /etc/nginx/ssl/test.local.key -out /etc/nginx/ssl/test.local.crt \
  -subj "/CN=test.local"
```

Add an Nginx server block for HTTPS and enable TLS 1.3 and strong ciphers. Test and reload:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

From Kali, capture the TLS handshake and then visit the site:

```bash
sudo tcpdump -i eth0 tcp port 443 -w ~/day3_lab/pcaps/tls_handshake_$(date -u +"%Y%m%dT%H%M%SZ").pcap &
echo $! > ~/day3_lab/artifacts/tcpdump_tls_pid.txt
curl -vk https://test.local/ --resolve test.local:443:192.168.20.10 | tee ~/day3_lab/scans/curl_https_$(date -u +"%Y%m%dT%H%M%SZ").txt
kill $(cat ~/day3_lab/artifacts/tcpdump_tls_pid.txt) || true
```

Inspect the pcap for ClientHello, ServerHello, Certificate, Key Exchange, and Finished messages. Note the cipher suite chosen and whether TLS 1.3 was negotiated. Save the curl output and the Wireshark screenshots.

Step 4 — Configure Burp as an intercepting proxy and observe HTTP and HTTPS. Start Burp on Kali, listening on `127.0.0.1:8080`. In Firefox, set manual proxy to `127.0.0.1:8080` for HTTP and HTTPS. Import Burp’s CA into Firefox so HTTPS can be intercepted. Visit the site and observe requests in Burp’s Proxy tab. Save the Burp session export.

Step 5 — Experiment with DNS manipulation. On Kali, temporarily override `/etc/hosts` to point `test.local` to an attacker‑controlled IP and observe how the browser and curl behave. Revert and then simulate a DNS cache poisoning scenario by running a local resolver that returns a forged answer and observe how the client uses it. Use `dig +trace` to follow the resolution path and understand where authoritative answers come from.

Step 6 — Explore proxy behavior. Configure Nginx as a reverse proxy in front of a backend service. Add `X-Forwarded-For` and `X-Forwarded-Host` handling. Test how the backend sees the Host header and client IP. Then craft requests that manipulate these headers and observe how the backend and proxy behave. Capture the traffic and inspect logs on both proxy and backend.

Step 7 — Advanced experiments. Test request smuggling in a controlled lab by crafting raw TCP requests with conflicting `Content-Length` and `Transfer-Encoding` headers and observing how the front end and backend parse them. Test cache poisoning by manipulating `Vary` and `Cache-Control` headers and observing proxy caches. Test DNS rebinding with a controlled domain and short TTLs in a safe environment.

Throughout the lab, create a manifest entry for each command and artifact. Keep every pcap, every config file, every curl output, and every screenshot. These artifacts are your evidence and your study material.

---

## 12. Packet and trace analysis — what to look for in Wireshark and logs 🔍

When you open a pcap, your eyes should be trained to find the causal chain. For DNS, filter `dns` and inspect question and answer sections, TTLs, and authority sections. For HTTP, follow the TCP stream and inspect the raw request and response. For TLS, filter `tls` and inspect handshake messages. Note cipher suites, TLS versions, and certificate details.

Use `tshark` for command‑line extraction:

```bash
tshark -r ~/day3_lab/pcaps/tls_handshake_*.pcap -Y "tls.handshake" -V
tshark -r ~/day3_lab/pcaps/http_get_*.pcap -Y "http.request" -T fields -e http.host -e http.request.uri -e http.user_agent
```

Correlate server logs with packet timestamps. Use `grep` and `awk` to extract relevant lines. For example, to find requests with a specific Host header in Nginx access logs:

```bash
grep "Host: test.local" /var/log/nginx/access.log
```

When investigating TLS, extract the server certificate from the pcap or use `openssl s_client` to fetch it and inspect:

```bash
openssl s_client -connect 192.168.20.10:443 -servername test.local -showcerts </dev/null 2>/dev/null | openssl x509 -noout -text
```

Look for SANs, issuer, validity, and key size. For proxies, compare client‑side and backend‑side captures to see header rewriting and request transformations. For request smuggling, examine raw bytes and how different components interpret `Content-Length` and `Transfer-Encoding`.

When analyzing DNS, check for unexpected referrals, NXDOMAIN responses, or mismatched TTLs. For cache poisoning, look for responses that do not match authoritative answers and for TTLs that keep bad data alive.

---

## 13. Artifact strategy — manifests timestamps and reproducible evidence 📁

A lab is only as useful as its artifacts. For every action, record the exact command, the timestamp in UTC, the file where output was saved, and a one‑line interpretation. Your manifest should be chronological and precise.

Example manifest lines:

```text
2026-06-03T02:00:00Z | mkdir -p ~/day3_lab | N/A | Created lab directories
2026-06-03T02:05:12Z | apt install bind9 | N/A | Installed BIND9
2026-06-03T02:06:00Z | created /etc/bind/db.test.local | /etc/bind/db.test.local | Zone file created
2026-06-03T02:07:05Z | systemctl restart bind9 | N/A | BIND9 reloaded with new zone
2026-06-03T02:07:10Z | dig @192.168.20.10 test.local | ~/day3_lab/scans/dig_test_local_20260603T020710Z.txt | DNS query and authoritative response captured
2026-06-03T02:20:00Z | apt install nginx | N/A | Installed Nginx
2026-06-03T02:22:00Z | curl -v http://192.168.20.10 | ~/day3_lab/scans/curl_http_20260603T022200Z.txt | HTTP GET and response captured
2026-06-03T02:30:00Z | openssl req -x509 ... | /etc/nginx/ssl/test.local.crt | Self-signed cert generated
2026-06-03T02:35:00Z | curl -vk https://test.local --resolve test.local:443:192.168.20.10 | ~/day3_lab/scans/curl_https_20260603T023500Z.txt | TLS handshake and HTTPS response captured
2026-06-03T02:45:00Z | Burp session export | ~/day3_lab/artifacts/burp_session_20260603T024500Z.xml | Proxy interception artifacts
```

Keep pcaps, config files, curl outputs, dig outputs, Burp exports, and screenshots. Package them into a zip with an ISO timestamp for archiving:

```bash
zip -r ~/day3_lab/artifacts/day3_lab_package_$(date -u +"%Y%m%dT%H%M%SZ").zip ~/day3_lab/
```

---

## 14. Failure modes debugging and investigative heuristics 🧠

When things fail, treat the failure as data. A failed DNS lookup could be a zone misconfiguration, a firewall blocking port 53, or a resolver cache issue. Use `dig +trace` to see where resolution stops. Use `named-checkzone` to validate zone files. Check BIND logs for errors.

When HTTP fails, check the server’s access and error logs. Use `curl -v` to see the TCP handshake and HTTP exchange. If TLS fails, use `openssl s_client` to inspect the certificate chain and handshake. If the browser warns about certificates, examine the certificate chain and the trust anchors.

When proxies behave unexpectedly, capture traffic on both sides of the proxy. Compare raw bytes. Check proxy configuration for header rewriting, caching rules, and TLS termination. For request smuggling, craft minimal test cases that exercise `Content-Length` and `Transfer-Encoding` differences and observe how each component parses them.

When you see intermittent behavior, consider caching and TTLs. A stale DNS cache can cause inconsistent resolution. A CDN or caching proxy can serve old content. Clear caches, reduce TTLs during testing, and document the state of caches.

Always preserve evidence. When you find a vulnerability or misconfiguration, capture the minimal reproducible steps, the exact requests and responses, and the timestamps. This is how you prove impact and how you learn.

---

## 15. Reflection prompts — turning practice into intuition 🎯

Write a reflective note in `~/day3_lab/notes/reflections_day3_$(date -u +"%Y%m%dT%H%M%SZ").md` and answer these prompts in full sentences.

Describe, step by step, what happens from the moment you type a domain into the browser until the page renders, including DNS resolution, TCP handshake, TLS handshake if applicable, HTTP request, and response processing. Explain how a Host header can be abused to perform host header injection or cache poisoning, and provide a concrete example of a vulnerable flow and how you would test it safely. Describe the TLS handshake you observed in the lab, the cipher suite negotiated, whether TLS 1.3 was used, and what you would change in the server configuration to improve security. Explain how a reverse proxy can both protect and expose backend services, and give an example of a proxy misconfiguration that would allow SSRF or request smuggling. Summarize three concrete defensive changes you would recommend to a small web team to harden their DNS HTTP and TLS posture.

---

This merged chapter preserves every command, configuration, and explanation you previously had and layers on additional depth, forensic steps, and practical heuristics. It is the single, complete Day 3 you asked for: full theory, full practice, full artifacts, and full investigative guidance.

If you want, I will now:

- produce a **runnable automation** (Docker Compose or Vagrant) that builds this lab and generates the artifacts automatically, or
- convert the lab into a **step‑by‑step checklist** with expected outputs for Ubuntu 22.04 + Nginx + BIND9, or
- extract a **compact cheat sheet** of the exact `tcpdump`, `dig`, `curl`, `openssl`, `tshark`, and `nginx` commands used here for quick reference.

Tell me which of those you want next and I’ll produce it in the same exhaustive style.