# 🧨 DAY 3 — DNS, HTTP, HTTPS, TLS, and Proxies
Day 3 is where networking stops being “packets moving through pipes” and becomes **language**.

If Day 1 taught you how packets _live_,  
and Day 2 taught you how packets _travel_,  
then Day 3 teaches you **how packets _mean_**.

Because today is about the protocols that make the internet _human-readable_:

- DNS — the internet’s phonebook
- HTTP — the language of the web
- HTTPS — trust, encryption, identity
- TLS — the handshake that makes the modern world possible
- Proxies — the middlemen, the translators, the liars

This is the day where you learn how the web actually works — not the “browser version,” but the **wire version**.

### _2 hours_

### _“Today you learn the languages machines speak when they want to talk like humans.”_

---

# I. **Why Day 3 Matters**

You can’t do bug bounty hunting without understanding:

- how a browser finds a website
- how a server identifies itself
- how encryption works
- how certificates work
- how proxies rewrite traffic
- how DNS lies can break everything
- how HTTP headers can be abused
- how HTTPS can be intercepted
- how TLS can be downgraded
- how redirects work
- how cookies travel
- how sessions persist

Every single web vulnerability you will ever exploit lives in this layer.

This is the layer where:

- SQL injection
- XSS
- CSRF
- SSRF
- CORS
- Host header injection
- Cache poisoning
- Cookie hijacking
- JWT tampering
- API abuse

…all happen.

If you don’t understand Day 3, you don’t understand the web.

---

# II. **DNS — The Internet’s Phonebook (and Its Weakest Link)**

DNS is the system that turns:

```
google.com
```

into:

```
142.250.190.78
```

It is:

- old
- unauthenticated
- cacheable
- spoofable
- redirectable
- hijackable

DNS is the **weakest link** in the entire internet.

---

## **How DNS Works (The Real Version)**

When you type `example.com` into your browser:

1. Your machine checks its local DNS cache
2. If not found → it asks your DNS resolver
3. The resolver asks the root servers
4. The root servers point to the TLD servers
5. The TLD servers point to the authoritative servers
6. The authoritative servers return the IP
7. The resolver caches it
8. Your machine caches it
9. Your browser connects to the IP

This entire process happens in milliseconds.

---

## **Why DNS Matters for Hacking**

Because DNS is:

- spoofable
- cache‑poisonable
- redirectable
- injectable
- observable
- abusable

DNS is the foundation of:

- SSRF
- DNS rebinding
- subdomain takeover
- CNAME misconfigurations
- wildcard hijacking
- internal host discovery
- cloud metadata access
- phishing
- exfiltration channels

DNS is the **first trust boundary** in the web.

---

# III. **HTTP — The Language of the Web**

HTTP is not a protocol.  
HTTP is a **conversation**.

A browser says:

```
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: */*
```

The server replies:

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 5123
```

This is the entire web.

---

## **HTTP Is Stateless**

Every request is independent.

This is why cookies exist.  
This is why sessions exist.  
This is why authentication is hard.  
This is why CSRF exists.  
This is why JWTs exist.  
This is why session fixation exists.

HTTP is simple.  
But simplicity breeds vulnerability.

---

## **HTTP Headers Are Power**

Headers control:

- caching
- cookies
- redirects
- content types
- authentication
- CORS
- proxies
- compression
- security policies

Headers are where:

- cache poisoning
- host header injection
- CORS misconfigurations
- cookie hijacking
- SSRF
- request smuggling

…all begin.

---

# IV. **HTTPS — The Illusion of Safety**

HTTPS is not encryption.  
HTTPS is **identity + encryption + integrity**.

It answers three questions:

1. **Who are you?** (certificate)
2. **Can anyone read this?** (encryption)
3. **Can anyone change this?** (integrity)

HTTPS is a handshake, not a magic spell.

---

# V. **TLS — The Handshake That Runs the World**

TLS is the protocol that makes HTTPS possible.

The handshake looks like this:

1. ClientHello
2. ServerHello
3. Certificate
4. Key exchange
5. Encrypted communication

This is where:

- certificate validation happens
- cipher negotiation happens
- downgrade attacks happen
- MITM attacks fail (or succeed)
- trust is established

TLS is the **border between trust and chaos**.

---

# VI. **Proxies — The Middlemen of the Internet**

A proxy is a machine that sits between you and the server.

There are many types:

- forward proxies
- reverse proxies
- transparent proxies
- caching proxies
- load balancers
- WAFs
- API gateways

Proxies rewrite:

- headers
- URLs
- cookies
- hostnames
- paths
- responses

Proxies are where:

- SSRF becomes internal access
- request smuggling happens
- cache poisoning happens
- host header injection happens
- CORS misconfigurations happen

Proxies are the **choke points** of the web.

---

# 🧪 **LAB — DNS, HTTP, HTTPS, TLS, and Proxying in Action**

This is where Day 3 becomes real.

---

## **1. Install DNS Server on Ubuntu**

```
sudo apt install bind9
```

Create zone file:

`/etc/bind/db.test.local`

Add:

```
test.local. IN A 192.168.20.10
```

Reload:

```
sudo systemctl restart bind9
```

---

## **2. Query DNS from Kali**

```
dig test.local @192.168.20.10
```

Observe:

- query
- response
- TTL
- authoritative section

You are watching DNS happen.

---

## **3. Install Nginx on Ubuntu**

```
sudo apt install nginx
```

Visit:

```
http://192.168.20.10
```

Capture in Wireshark.

Observe:

- GET request
- Host header
- response headers

---

## **4. Add TLS**

Generate cert:

```
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -nodes
```

Configure Nginx for HTTPS.

Visit:

```
https://192.168.20.10
```

Capture TLS handshake.

Observe:

- ClientHello
- ServerHello
- Certificate

You are watching trust being negotiated.

---

## **5. Configure Burp Suite as Proxy**

In Firefox:

```
127.0.0.1:8080
```

Intercept HTTP and HTTPS.

Observe:

- headers
- cookies
- redirects
- parameters

You are watching the web speak.

---

# 🎯 **What You Should Feel in Your Bones by the End of Day 3**

- DNS is the first trust boundary
- HTTP is a conversation
- HTTPS is identity + encryption
- TLS is the handshake that makes trust possible
- Proxies rewrite reality
- Every web vulnerability is built on these layers
- You can now read traffic like a language
- You can now see the web as a system, not a website

You are no longer browsing the web.  
You are _listening_ to it.

---

If you want, I’ll continue with **Day 4** in the same style — deep, narrative, conceptual, practical, and hacker‑focused.