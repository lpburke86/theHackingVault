# 🧨 DAY 2 — Subnetting, Routing, NAT, and Firewalls
Day 2 is where networking stops being “packets moving around” and becomes **geometry**.  
This is the day you learn how networks are _shaped_, how they’re _divided_, how they’re _protected_, and how attackers slip through the cracks.

Day 1 taught you how a packet _lives_.  
Day 2 teaches you how a packet _travels_ — and how defenders try (and often fail) to control that travel.
### _2 hours_

### _“Today you learn how networks are carved, connected, and controlled.”_

---

# **I. Why Day 2 Matters**

If Day 1 was about understanding the _physics_ of networking, Day 2 is about understanding the _geometry_ of it.

Because the internet is not one big flat space.  
It’s a **maze** — a maze carved into:

- subnets
- VLANs
- DMZs
- internal networks
- cloud VPCs
- VPN segments
- NAT boundaries
- firewall zones

Every bug bounty target you will ever touch lives inside one of these shapes.

If you don’t understand how networks are segmented, you can’t understand:

- SSRF impact
- internal port scanning
- cloud metadata access
- VPN pivoting
- firewall bypass
- lateral movement
- exposed services
- misconfigured NAT
- misconfigured ACLs
- misconfigured proxies

Day 2 is the day you learn how to **see the map**.

---

# **II. Subnetting: The Art of Drawing Lines**

Subnetting is not math.  
Subnetting is **cartography**.

It’s drawing borders around groups of machines so they can:

- talk to each other
- be isolated from others
- be routed between
- be protected by firewalls
- be NAT’d
- be segmented

A subnet is simply:

> “A group of IP addresses that live together and share a common gateway.”

That’s it.

But the implications are enormous.

---

## **CIDR Notation (The Only Parts That Matter)**

You don’t need to memorize binary.  
You need to understand **scale**.

- `/24` → 256 IPs → small networks
- `/16` → 65,536 IPs → large networks
- `/30` → 2 usable IPs → point‑to‑point links
- `/32` → 1 IP → a single host

Bug bounty hunters care about:

- `/24` → typical internal subnet
- `/16` → cloud VPCs
- `/32` → firewall rules, VPN routes, ACLs

When you see a `/32` in a firewall rule, it means:

> “This rule applies to exactly one machine.”

When you see a `/0`, it means:

> “This rule applies to the entire internet.”

That difference is the difference between:

- a secure system
- a catastrophic breach

---

## **Why Subnetting Matters for Hacking**

Because segmentation is the only thing standing between:

- a compromised web server
- and the company’s internal network

If segmentation is weak, you get:

- internal SSRF
- internal port scanning
- access to admin panels
- access to databases
- access to cloud metadata
- access to internal APIs
- access to internal dashboards
- access to internal CI/CD systems

Subnetting is the **first line of defense**.  
It is also the **first thing attackers try to break**.

---

# **III. Routing: The Art of Movement**

Routing is the process of deciding:

> “Where does this packet go next?”

A router is not a magical device.  
It is a machine with a table that says:

- “If the destination is in this range, send it here.”
- “If not, send it to the default gateway.”

Routing is **local decisions creating global behavior**.

---

## **The Routing Table (Your New Best Friend)**

On Linux:

```
ip route
```

You will see something like:

```
default via 192.168.10.1 dev eth0
192.168.10.0/24 dev eth0 proto kernel scope link src 192.168.10.10
```

This means:

- “Anything not in my local subnet goes to 192.168.10.1.”
- “Anything in 192.168.10.0/24 stays local.”

Routing is simple.  
But the consequences of misrouting are not.

---

## **Why Routing Matters for Hacking**

Because routing determines:

- what a machine can reach
- what it can’t reach
- what SSRF can reach
- what a VPN can reach
- what a cloud instance can reach
- what a container can reach
- what a compromised server can reach

Routing is the **attack surface map**.

If you understand routing, you understand:

- pivoting
- lateral movement
- SSRF impact
- cloud metadata access
- internal scanning
- VPN segmentation
- proxy bypass

Routing is the **circulatory system** of a network.

---

# **IV. NAT: The Great Illusion**

NAT — Network Address Translation — is the internet’s biggest magic trick.

It lets:

- many internal machines
- share one public IP

It does this by rewriting packets:

- source IP
- source port
- destination IP
- destination port

NAT is not security.  
NAT is **address conservation**.

But NAT _creates_ security side effects:

- internal machines are hidden
- unsolicited inbound traffic is blocked
- outbound traffic is allowed

This is why your home network “feels safe.”

But NAT is also why:

- SSRF can bypass firewalls
- cloud metadata endpoints are reachable
- internal services are exposed
- port forwarding can be misconfigured
- attackers can pivot through NAT

NAT is a **veil**, not a wall.

---

# **V. Firewalls: The Gatekeepers**

Firewalls are not magical.  
They are rule engines.

A firewall rule is simply:

> “If a packet matches these conditions, do this.”

Conditions include:

- source IP
- destination IP
- source port
- destination port
- protocol
- interface
- direction

Actions include:

- allow
- block
- reject
- log

Firewalls are **pattern matchers**.

---

## **The Most Important Rule of Firewalls**

### **Order matters.**

Rules are processed **top to bottom**.

A single misplaced rule can:

- expose a service
- block legitimate traffic
- break routing
- allow attackers in
- prevent defenders from seeing attacks

Firewalls are powerful.  
Firewalls are fragile.

---

## **Why Firewalls Matter for Hacking**

Because firewalls determine:

- what you can reach
- what you can’t
- what SSRF can reach
- what internal services are exposed
- what ports are open
- what ports are filtered
- what ports are silently dropped
- what ports leak information

Firewalls are the **choke points** of a network.

If you understand firewalls, you understand:

- bypassing rules
- exploiting misconfigurations
- pivoting
- tunneling
- port forwarding
- segmentation flaws

Firewalls are the **narrow gates** attackers slip through.

---

# 🧪 **LAB — Subnets, Routes, NAT, and Firewalls in Action**

This is where Day 2 becomes real.

---

## **1. Add a Second Subnet**

In GNS3, add a new Ubuntu VM.

Assign:

```
192.168.30.10/24
Gateway: 192.168.30.1
```

Add a new pfSense interface:

- OPT1 → `192.168.30.1/24`

You now have:

```
Kali → pfSense → Ubuntu (20.x)
                 ↳ Ubuntu2 (30.x)
```

You have created a **segmented network**.

---

## **2. Add Static Routes**

pfSense → System → Routing → Static Routes

Add:

```
Destination: 192.168.30.0/24
Gateway: 192.168.30.1
```

You have taught pfSense how to reach the new subnet.

---

## **3. Add Firewall Rules**

pfSense → Firewall → Rules → OPT1

Allow HTTP:

```
Action: Pass
Protocol: TCP
Port: 80
```

Block SSH:

```
Action: Block
Protocol: TCP
Port: 22
```

Allow ICMP:

```
Action: Pass
Protocol: ICMP
```

You have created **policy**.

---

## **4. Test From Kali**

```
curl http://192.168.30.10
ssh 192.168.30.10
```

HTTP works.  
SSH fails.

This is segmentation in action.

---

## **5. Capture Blocked Packets**

pfSense → Diagnostics → Packet Capture

Filter:

```
host 192.168.30.10 and port 22
```

You will see:

- SYN from Kali
- No SYN/ACK
- Firewall silently drops

This is what “blocked” looks like.

---

# 🎯 **What You Should Feel in Your Bones by the End of Day 2**

- Subnets are borders
- Routing is movement
- NAT is illusion
- Firewalls are gatekeepers
- Segmentation is security
- Mis‑segmentation is vulnerability
- You can now _see_ the shape of a network
- You can now reason about SSRF, pivoting, and internal access

You are no longer looking at networks from the outside.  
You are looking at them from the inside.

---

If you want, I’ll continue with **Day 3** in the same style — deep, narrative, conceptual, practical, and hacker‑focused.