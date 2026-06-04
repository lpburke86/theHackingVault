Liam —  
Here it is.  
The **full, detailed, dense, guided, narrative-driven challenge workbook** for all 7 days of your networking crash course.

This is not a worksheet.  
This is not a list of tasks.  
This is a **training manual** — a structured set of challenges designed to _force_ the student to think, observe, break, fix, reason, and internalize the networking concepts you’ve taught them.

Every day includes:

- **Core Challenges** (must complete)
- **Deep‑Dive Challenges** (for mastery)
- **Wireshark Challenges** (to build packet intuition)
- **Bug Bounty Challenges** (to connect theory → exploitation)
- **Reflection Prompts** (to build mental models)
- **Expected Outcomes** (what they should know by the end)

This is the workbook you give someone when you want them to _earn_ their understanding.

Let’s begin.

---

# 🧨 **NETWORKING CRASH COURSE — 7‑DAY CHALLENGE WORKBOOK**

### _A complete, guided, hands‑on workbook for mastering networking fundamentals for bug bounty hunting._

---

# 🧱 **DAY 0 — LAB SETUP CHALLENGES**

### _“Build your world before you learn to navigate it.”_

---

## **Core Challenges**

### **0.1 — Build the 3‑Node Network**

Construct the topology:

```
Kali → pfSense → Ubuntu
```

**Requirements:**

- Kali: `192.168.10.10/24`, gateway `192.168.10.1`
- pfSense LAN: `192.168.10.1/24`
- pfSense WAN: `192.168.20.1/24`
- Ubuntu: `192.168.20.10/24`, gateway `192.168.20.1`

**Success Criteria:**  
Kali can ping Ubuntu.

---

### **0.2 — Enable Packet Capture on All Links**

Start captures on:

- Kali ↔ pfSense
- pfSense ↔ Ubuntu

**Success Criteria:**  
You can see ARP, ICMP, and MAC addresses in Wireshark.

---

## **Deep‑Dive Challenges**

### **0.3 — Break the Network**

Delete Ubuntu’s default route:

```
ip route del default
```

Observe:

- What fails?
- What still works?
- Why?

Then fix it.

---

### **0.4 — Change Subnets**

Reassign Ubuntu to:

```
192.168.50.10/24
```

Fix routing until the network works again.

---

## **Reflection Prompts**

- What surprised you about how fragile routing is?
- What did you learn about how machines “know” where to send packets?
- What did you learn from breaking the network?

---

# 🎯 **Expected Outcome**

You understand how to build, break, and fix a real network.

---

# 🧨 **DAY 1 — PACKET FLOW & THE LIFE OF A PACKET**

### _“See the network as a living system.”_

---

## **Core Challenges**

### **1.1 — Capture the ARP Dance**

From Kali:

```
ping 192.168.20.10
```

In Wireshark:

- Filter: `arp`
- Identify:
    - ARP request
    - ARP reply
    - MAC addresses

**Write down:**  
What is ARP doing? Why does it matter?

---

### **1.2 — Trace the ICMP Echo Path**

Filter:

```
icmp
```

Identify:

- Echo request
- Echo reply
- TTL values
- Source/destination IPs

---

## **Deep‑Dive Challenges**

### **1.3 — Manipulate ARP Cache**

On Kali:

```
ip neigh flush all
```

Ping again.

Observe:

- ARP broadcast
- ARP reply
- Cache repopulation

---

### **1.4 — Change MAC Address**

On Kali:

```
ip link set dev eth0 address 00:11:22:33:44:55
```

Observe ARP behavior.

---

## **Wireshark Challenges**

- Identify the exact moment the ARP cache is repopulated.
- Identify the MAC address of pfSense.
- Identify the MAC address of Ubuntu.
- Identify the TTL of ICMP replies.

---

## **Bug Bounty Challenges**

- Explain how ARP spoofing enables MITM.
- Explain how ARP poisoning leads to credential theft.
- Explain why ARP is a trust vulnerability.

---

## **Reflection Prompts**

- What does ARP reveal about trust on a LAN?
- Why is packet flow the foundation of hacking?

---

# 🎯 **Expected Outcome**

You understand ARP, ICMP, MAC addresses, and packet flow at a visceral level.

---

# 🧨 **DAY 2 — SUBNETTING, ROUTING, NAT, FIREWALLS**

### _“Understand the geometry of networks.”_

---

## **Core Challenges**

### **2.1 — Add a Second Subnet**

Add:

- pfSense OPT1: `192.168.30.1/24`
- Ubuntu2: `192.168.30.10/24`

Fix routing until Kali → Ubuntu2 works.

---

### **2.2 — Create Firewall Rules**

On pfSense:

- Allow HTTP
- Block SSH
- Allow ICMP

Test from Kali.

---

## **Deep‑Dive Challenges**

### **2.3 — NAT Behavior**

From Kali:

```
curl http://192.168.20.10
```

Capture on pfSense WAN.

Identify:

- source IP
- translated IP
- NAT behavior

---

### **2.4 — Break Firewall Rules**

Create a rule that accidentally blocks all traffic.

Fix it.

---

## **Wireshark Challenges**

- Identify blocked SSH attempts.
- Identify NAT‑translated packets.
- Identify routing failures.

---

## **Bug Bounty Challenges**

- Explain how segmentation failures lead to internal access.
- Explain how NAT enables SSRF escalation.
- Explain how firewall misordering leads to vulnerabilities.

---

## **Reflection Prompts**

- What did you learn about segmentation?
- Why is NAT an illusion?

---

# 🎯 **Expected Outcome**

You understand routing, NAT, segmentation, and firewall logic.

---

# 🧨 **DAY 3 — DNS, HTTP, HTTPS, TLS, PROXIES**

### _“Learn the languages machines speak.”_

---

## **Core Challenges**

### **3.1 — Build a DNS Zone**

Create:

```
test.local → 192.168.20.10
```

Query with:

```
dig test.local @192.168.20.10
```

---

### **3.2 — Install Nginx**

Visit:

```
http://192.168.20.10
```

Capture HTTP traffic.

---

### **3.3 — Add TLS**

Generate cert → enable HTTPS.

Capture TLS handshake.

---

## **Deep‑Dive Challenges**

### **3.4 — Modify DNS TTL**

Set TTL to 1 second.

Observe caching behavior.

---

### **3.5 — Proxy Interception**

Configure Burp Suite.

Intercept:

- cookies
- headers
- redirects

---

## **Wireshark Challenges**

- Identify ClientHello
- Identify ServerHello
- Identify certificate
- Identify HTTP headers

---

## **Bug Bounty Challenges**

- Explain how DNS rebinding works.
- Explain how Host header injection works.
- Explain how CORS misconfigurations happen.

---

## **Reflection Prompts**

- What surprised you about DNS trust?
- What did you learn about TLS identity?

---

# 🎯 **Expected Outcome**

You understand DNS, HTTP, HTTPS, TLS, and proxies deeply.

---

# 🧨 **DAY 4 — WIRESHARK DEEP DIVE**

### _“See the network like an X‑ray.”_

---

## **Core Challenges**

### **4.1 — Capture HTTP Conversation**

Use:

```
curl http://192.168.20.10
```

Follow TCP stream.

---

### **4.2 — Capture HTTPS Handshake**

Visit:

```
https://192.168.20.10
```

Identify:

- ClientHello
- ServerHello
- Certificate

---

## **Deep‑Dive Challenges**

### **4.3 — Capture SQL Injection Payload**

```
curl "http://192.168.20.10/?id=1' OR '1'='1"
```

Find payload in Wireshark.

---

### **4.4 — Capture XSS Payload**

```
curl "http://192.168.20.10/?q=<script>alert(1)</script>"
```

---

## **Wireshark Challenges**

- Identify retransmissions
- Identify SYN scans
- Identify ARP poisoning attempts

---

## **Bug Bounty Challenges**

- Explain how Wireshark reveals vulnerabilities.
- Explain how to detect brute force attacks.
- Explain how to detect MITM.

---

## **Reflection Prompts**

- What did you learn about reading packets?
- What patterns did you start to recognize?

---

# 🎯 **Expected Outcome**

You can read packets like sentences.

---

# 🧨 **DAY 5 — ENUMERATION & FINGERPRINTING**

### _“Make machines talk.”_

---

## **Core Challenges**

### **5.1 — Basic Scan**

```
nmap 192.168.20.10
```

---

### **5.2 — Version Detection**

```
nmap -sV 192.168.20.10
```

---

### **5.3 — OS Detection**

```
nmap -O 192.168.20.10
```

---

## **Deep‑Dive Challenges**

### **5.4 — Full Scan**

```
nmap -A 192.168.20.10
```

---

### **5.5 — NSE Scripts**

```
nmap --script http-title 192.168.20.10
```

---

## **Wireshark Challenges**

- Identify SYN packets
- Identify version probes
- Identify OS fingerprinting behavior

---

## **Bug Bounty Challenges**

- Explain how enumeration reveals attack surfaces.
- Explain how version detection leads to CVE discovery.

---

## **Reflection Prompts**

- What did you learn about service behavior?
- What surprised you about fingerprinting?

---

# 🎯 **Expected Outcome**

You can enumerate any machine.

---

# 🧨 **DAY 6 — MITM, PROXYING, TUNNELING**

### _“Control the path packets travel.”_

---

## **Core Challenges**

### **6.1 — ARP Spoofing**

```
ettercap -T -M arp:remote /victim/ /gateway/
```

Capture traffic.

---

### **6.2 — SSH SOCKS Proxy**

```
ssh -D 9050 user@192.168.20.10
```

Route browser through SOCKS5.

---

## **Deep‑Dive Challenges**

### **6.3 — DNS Spoofing**

Modify DNS zone → redirect traffic.

---

### **6.4 — Tunnel Through Firewall**

Use SSH tunnel to access blocked ports.

---

## **Wireshark Challenges**

- Identify ARP poisoning
- Identify MITM traffic
- Identify tunneled traffic

---

## **Bug Bounty Challenges**

- Explain how SSRF → internal pivoting
- Explain how tunnels bypass segmentation

---

## **Reflection Prompts**

- What did you learn about trust boundaries?
- What did you learn about traffic manipulation?

---

# 🎯 **Expected Outcome**

You can manipulate, intercept, and reroute traffic.

---

# 🧨 **DAY 7 — BUG BOUNTY NETWORKING SCENARIOS**

### _“Turn knowledge into exploitation.”_

---

## **Core Challenges**

### **7.1 — Simulate SSRF**

```
curl http://localhost:80
curl http://127.0.0.1:80
curl http://192.168.20.10:80
```

---

### **7.2 — Internal Port Scanning**

```
curl http://192.168.20.10:22
```

---

## **Deep‑Dive Challenges**

### **7.3 — DNS Rebinding Simulation**

Alternate DNS responses.

Observe browser behavior.

---

### **7.4 — Cloud Metadata Simulation**

Host fake metadata endpoint:

```
/latest/meta-data/
```

Access via SSRF.

---

## **Wireshark Challenges**

- Identify SSRF traffic
- Identify internal scanning
- Identify DNS rebinding behavior

---

## **Bug Bounty Challenges**

- Explain how SSRF → RCE
- Explain how DNS rebinding → internal access
- Explain how Host header injection → cache poisoning

---

## **Reflection Prompts**

- What did you learn about real-world vulnerabilities?
- How does networking knowledge change how you see bug bounty?

---

# 🎯 **Expected Outcome**

You can apply networking knowledge directly to bug bounty exploitation.

---

# 🏁 **FINAL NOTE**

This workbook is not just practice.  
It is **transformation**.

Anyone who completes these challenges will:

- understand networking deeply
- think like an attacker
- reason about systems
- see vulnerabilities before they’re obvious
- be ready for your 10‑week offensive security course

If you want, I can also generate:

- A **final exam**
- A **grading rubric**
- A **student version** and **instructor version**
- A **print‑ready PDF layout**
- A **visual map** of all challenges

Just tell me what you want next.