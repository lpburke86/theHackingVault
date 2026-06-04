
## _Everything you need. Nothing you don’t._

### Total Time: **14 hours** (setup time excluded)

---

# 🧰 Tools & Environment

## **Virtual Machines**

- **Kali Linux** (attacker)
- **Ubuntu Server** (target)
- **pfSense CE** (firewall/router)

## **Network Simulator**

### **GNS3 (Free, Open‑Source)**

- Supports virtual routers, switches, firewalls
- Integrates with VMs
- Perfect for packet flow labs
- Runs on Windows/Linux/macOS

You’ll build a **3‑node lab**:

```
Kali ─── pfSense ─── Ubuntu Server
```

You’ll use this to learn:

- Routing
- NAT
- Firewall rules
- Packet capture
- Traffic analysis
- Port scanning
- Service enumeration
- HTTP/HTTPS internals

---

# 🧱 Setup Guide (Not counted in the 15 hours)

## **1. Install GNS3**

- Download from gns3.com
- Install GNS3 + GNS3 VM
- Import the GNS3 VM into VirtualBox or VMware
- Link GNS3 to the VM

## **2. Import VMs**

- Import Kali
- Import Ubuntu Server
- Import pfSense CE

## **3. Build the Topology**

Inside GNS3:

```
[Kali] → [pfSense] → [Ubuntu]
```

Assign networks:

- Kali: `192.168.10.10/24`
- pfSense LAN: `192.168.10.1`
- pfSense WAN: `192.168.20.1`
- Ubuntu: `192.168.20.10/24`

Enable:

- NAT on pfSense
- DHCP off
- Firewall logging on
- Packet capture on all interfaces

---

# 🗓️ **7‑Day Schedule (14 Hours Total)**

Each day = **2 hours**.

---

# **Day 1 — OSI Model, TCP/IP, and Packet Flow**

### Time: 2 hours

## **Theory (45 min)**

- OSI Model (what matters for bug bounty)
- TCP/IP stack
- Ports, protocols, sockets
- How packets move through a network
- ARP, DHCP, DNS, HTTP, TLS
- NAT vs PAT
- Firewalls vs routers vs switches

## **Lab (75 min)**

Inside GNS3:

1. Ping from Kali → Ubuntu
2. Capture packets on pfSense
3. Analyze ARP, ICMP, DNS, TCP handshakes
4. Trace packet flow through interfaces
5. Break routing → observe failure
6. Fix routing → observe recovery

**Skills gained:**  
You understand how packets actually move.

---

# **Day 2 — Subnetting, Routing, NAT, and Firewalls**

### Time: 2 hours

## **Theory (45 min)**

- Subnets, CIDR, masks
- Default gateways
- Static vs dynamic routing
- NAT (source NAT, destination NAT)
- Firewall rule order
- Stateful vs stateless firewalls

## **Lab (75 min)**

1. Add a second subnet
2. Configure pfSense static routes
3. Create firewall rules:
    - Allow HTTP
    - Block SSH
    - Allow ICMP
4. Test from Kali
5. Capture blocked packets

**Skills gained:**  
You understand routing, NAT, and firewall behavior.

---

# **Day 3 — DNS, HTTP, HTTPS, TLS, and Proxies**

### Time: 2 hours

## **Theory (45 min)**

- DNS resolution flow
- A, AAAA, CNAME, MX, TXT
- HTTP request/response lifecycle
- HTTPS handshake
- TLS certificates
- Reverse proxies
- Load balancers

## **Lab (75 min)**

1. Install `bind9` on Ubuntu
2. Create a fake domain: `test.local`
3. Query it from Kali
4. Capture DNS traffic
5. Install Nginx on Ubuntu
6. Add TLS with self‑signed cert
7. Capture TLS handshake

**Skills gained:**  
You understand the protocols bug bounty hunters exploit daily.

---

# **Day 4 — Wireshark Deep Dive**

### Time: 2 hours

## **Theory (30 min)**

- Filters
- Following streams
- Reassembling TCP
- Identifying anomalies
- Detecting injections
- Detecting redirects
- Detecting proxy behavior

## **Lab (90 min)**

1. Capture HTTP traffic
2. Extract credentials
3. Capture HTTPS traffic
4. Identify handshake
5. Capture blocked firewall traffic
6. Capture port scans
7. Capture SQL injection attempts
8. Capture XSS payloads

**Skills gained:**  
You can read traffic like a book.

---

# **Day 5 — Scanning, Enumeration, and Service Fingerprinting**

### Time: 2 hours

## **Theory (30 min)**

- nmap scan types
- Banner grabbing
- Service fingerprinting
- OS detection
- UDP scanning
- Firewall evasion

## **Lab (90 min)**

1. Run full nmap scans
2. Run version detection
3. Run NSE scripts
4. Enumerate SSH, FTP, HTTP, DNS
5. Capture scans in Wireshark
6. Identify scan patterns

**Skills gained:**  
You can enumerate any target like a pro.

---

# **Day 6 — Traffic Manipulation, MITM, and Proxying**

### Time: 2 hours

## **Theory (45 min)**

- ARP spoofing
- DNS spoofing
- Transparent proxies
- Burp Suite proxying
- SSH tunneling
- SOCKS proxies
- VPN routing

## **Lab (75 min)**

1. Use `ettercap` for ARP spoofing
2. Capture MITM traffic
3. Configure Burp Suite as proxy
4. Route Kali browser through Burp
5. Intercept HTTP and HTTPS
6. Create SSH tunnel:  
    `ssh -D 9050 user@ubuntu`
7. Route Firefox through SOCKS5

**Skills gained:**  
You understand how attackers intercept and manipulate traffic.

---

# **Day 7 — Bug Bounty Networking Scenarios**

### Time: 2 hours

## **Theory (30 min)**

- How recon tools use networking
- How SSRF works
- How DNS rebinding works
- How CORS interacts with network layers
- How VPNs affect bug bounty testing
- How cloud networks differ

## **Lab (90 min)**

1. Simulate SSRF using curl
2. Simulate DNS rebinding using custom DNS
3. Simulate internal port scanning
4. Simulate firewall bypass
5. Simulate proxy misconfigurations
6. Capture everything in Wireshark

**Skills gained:**  
You can apply networking knowledge directly to bug bounty exploitation.

---

# 🎓 Final Outcome

After 14 hours, you will:

- Understand the OSI model in a practical way
- Understand TCP/IP deeply
- Understand routing, NAT, firewalls, DNS, HTTP, TLS
- Be fluent in Wireshark
- Be fluent in nmap
- Understand packet flow
- Understand proxies, tunnels, VPNs
- Understand how networking affects bug bounty vulnerabilities
- Have a working GNS3 lab you can extend forever

This is the **minimum viable networking knowledge** for a serious bug bounty hunter.

---

If you want, I can also generate:

### • A printable one‑page version

### • A full Obsidian‑ready syllabus

### • A GNS3 import file for the entire lab

### • A “Networking for Bug Bounty” cheat sheet

### • A companion workbook with exercises

Just tell me what you want next.