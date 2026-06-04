# 🧨 DAY 4 — Wireshark - Seeing the Invisible

Day 4 is where everything you’ve learned so far suddenly _clicks_.  
This is the day where networking stops being “theory” and becomes **perception**.  
Today you learn to _see_ the network — not as diagrams, not as abstractions, but as a living stream of conversations, arguments, secrets, mistakes, and misconfigurations flowing between machines.

Day 4 is **Wireshark Day**.  
And Wireshark is not a tool.  
Wireshark is a _superpower_.
### _2 hours_

### _“Today you learn to read the network like a book.”_

---

# I. **Why Wireshark Matters More Than Any Other Tool**

You can’t become a hacker without learning to see what the network is doing.

Every vulnerability you will ever exploit —  
every SSRF, every SQL injection, every XSS, every misconfiguration —  
is ultimately just **data moving across a wire**.

Wireshark lets you:

- see the packets
- see the headers
- see the payloads
- see the timing
- see the retransmissions
- see the failures
- see the lies
- see the truth

Wireshark is the microscope of networking.  
It is the telescope of hacking.  
It is the X‑ray vision of bug bounty.

If you can read packets, you can:

- debug anything
- understand anything
- exploit anything
- bypass anything
- fingerprint anything
- reverse engineer anything

Wireshark is the difference between guessing and knowing.

---

# II. **What Wireshark Actually Shows You**

Most people think Wireshark shows “packets.”  
It does — but that’s not the important part.

Wireshark shows you:

### **1. Intent**

You can see what a machine _meant_ to do.

### **2. Mistakes**

You can see misconfigurations, errors, and leaks.

### **3. Secrets**

You can see credentials, tokens, cookies, and API keys in plaintext protocols.

### **4. Conversations**

You can follow entire TCP streams like reading a chat log.

### **5. Behavior**

You can see how a machine reacts under stress, load, or attack.

### **6. Vulnerabilities**

You can see injections, malformed requests, and suspicious payloads.

Wireshark is not a packet viewer.  
Wireshark is a **truth machine**.

---

# III. **The Anatomy of a Packet (And Why It Matters)**

A packet is not just data.  
A packet is a **story**.

Every packet contains:

### **1. Link Layer (Ethernet)**

- Source MAC
- Destination MAC

This tells you **who is talking on the local network**.

### **2. Network Layer (IP)**

- Source IP
- Destination IP
- TTL
- Flags

This tells you **where the packet is going** and **how far it has traveled**.

### **3. Transport Layer (TCP/UDP)**

- Source port
- Destination port
- Sequence numbers
- Acknowledgements
- Flags (SYN, ACK, FIN, RST)

This tells you **what service is being contacted** and **how the conversation is progressing**.

### **4. Application Layer (HTTP, DNS, TLS, etc.)**

- Headers
- Payload
- Cookies
- Queries
- Responses

This tells you **what the machines are actually saying**.

A packet is a complete thought.  
A stream of packets is a conversation.  
A capture file is a **story about a network**.

---

# IV. **Filters: The Language of Wireshark**

Wireshark filters are not commands.  
They are **questions** you ask the network.

Examples:

- `http` → “Show me web traffic.”
- `dns` → “Show me name lookups.”
- `tcp.port == 80` → “Show me traffic to port 80.”
- `ip.addr == 192.168.20.10` → “Show me everything involving this machine.”
- `tcp.flags.syn == 1 && tcp.flags.ack == 0` → “Show me connection attempts.”
- `tcp.analysis.retransmission` → “Show me problems.”

Filters let you interrogate the network.

---

# V. **Following Streams: Reading Conversations**

Right‑click → **Follow → TCP Stream**

This is where Wireshark becomes magic.

You can see:

- full HTTP requests
- full HTTP responses
- cookies
- tokens
- parameters
- SQL injection payloads
- XSS payloads
- API keys
- session IDs
- login attempts
- redirects
- errors

You can literally read the web traffic like a book.

---

# VI. **What Attack Traffic Looks Like**

Attack traffic has a signature.  
It looks different from normal traffic.

### **SQL Injection**

```
?id=1' OR '1'='1
```

### **XSS**

```
<script>alert(1)</script>
```

### **Command Injection**

```
; cat /etc/passwd
```

### **SSRF**

```
http://127.0.0.1:80/admin
```

### **Brute Force**

- repeated requests
- same endpoint
- different credentials

### **Port Scanning**

- SYN packets to many ports
- no payload
- rapid sequence

### **MITM**

- duplicate ARP replies
- unexpected MAC addresses

Wireshark lets you _see_ attacks.

---

# VII. **LAB — Wireshark Deep Dive**

This is where Day 4 becomes real.

---

## **1. Capture HTTP Traffic**

From Kali:

```
curl http://192.168.20.10
```

In Wireshark:

- filter: `http`
- follow the TCP stream
- read the request
- read the response

You are reading the web raw.

---

## **2. Capture HTTPS Traffic**

Visit:

```
https://192.168.20.10
```

Filter:

```
tls.handshake
```

Observe:

- ClientHello
- ServerHello
- Certificate

You are watching trust being negotiated.

---

## **3. Capture Port Scans**

From Kali:

```
nmap -sS 192.168.20.10
```

Filter:

```
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

You will see:

- SYN
- SYN
- SYN
- SYN

This is what scanning looks like.

---

## **4. Capture SQL Injection**

From Kali:

```
curl "http://192.168.20.10/?id=1' OR '1'='1"
```

Filter:

```
http.request
```

Follow the stream.

You will see the payload in cleartext.

---

## **5. Capture XSS**

```
curl "http://192.168.20.10/?q=<script>alert(1)</script>"
```

Follow the stream.

You will see the payload.

---

## **6. Capture Blocked Firewall Traffic**

Try SSH:

```
ssh 192.168.30.10
```

Filter:

```
tcp.port == 22
```

You will see:

- SYN
- no SYN/ACK
- retransmissions

This is what “blocked” looks like.

---

# VIII. **Why Wireshark Makes You Dangerous**

Because once you can see the network:

- nothing is hidden
- nothing is mysterious
- nothing is confusing
- nothing is “black box”

You can debug anything.  
You can reverse engineer anything.  
You can fingerprint anything.  
You can exploit anything.

Wireshark is the difference between:

- guessing
- and knowing

Between:

- hoping
- and proving

Between:

- being a script kiddie
- and being a hacker

---

# 🎯 **What You Should Feel in Your Bones by the End of Day 4**

- You can read packets like sentences
- You can follow streams like conversations
- You can identify attacks by sight
- You can see misconfigurations
- You can see secrets
- You can see trust being negotiated
- You can see lies
- You can see the truth

You are no longer using the network.  
You are _observing_ it.

---

If you want, I’ll continue with **Day 5** in the same style — deep, narrative, conceptual, practical, and hacker‑focused.