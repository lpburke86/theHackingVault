# DAY 1 — The Shape of the Internet- OSI, TCP-IP, and the Life of a Packet
A _guided understanding_ of how the internet actually works, told in a way that makes you feel like you’re discovering it from the inside.

This is the day where networking stops being a pile of facts and becomes a living system you can reason about, manipulate, and eventually exploit.
### _2 hours_

### _“If you understand how a packet moves, you understand hacking.”_

---

# **I. The Problem With How Networking Is Usually Taught**

Most people are introduced to networking like this:

- “Here are seven OSI layers.”
- “Here are some acronyms.”
- “Here’s a diagram with arrows.”
- “Memorize this for the test.”

And then they wonder why nothing sticks.

Networking is not a vocabulary test.  
Networking is **physics**.  
It’s electrons, radio waves, timing, collisions, buffers, queues, tables, and rules.  
It’s machines trying to talk to each other across a hostile, chaotic environment.

If you want to be a hacker — or even just a competent bug bounty hunter — you need to understand networking the way a mechanic understands an engine:

- What moves
- Why it moves
- What happens when it doesn’t
- What happens when you force it to
- What happens when you lie to it
- What happens when you trick it into trusting you

Today is about building that intuition.

---

# **II. The OSI Model: A Story, Not a List**

The OSI model is usually taught as a rigid seven‑layer stack.  
But the OSI model is not a stack.  
It’s a **story** about how information transforms as it travels.

Let’s tell that story.

---

## **Layer 1 — The Physical Layer**

This is the layer nobody talks about, but everything depends on.

It’s:

- electricity
- voltage
- timing
- radio waves
- fiber optics
- interference
- noise

It’s the reason your Wi‑Fi dies when the microwave is on.  
It’s the reason Ethernet cables have twists.  
It’s the reason your packets don’t teleport — they _propagate_.

You don’t need to be an electrical engineer to hack.  
But you do need to respect that **everything above this layer is built on top of chaos**.

---

## **Layer 2 — The Data Link Layer (MAC + ARP)**

This is where machines learn each other’s names.

Every network interface has a **MAC address** — a hardware identifier.

When Kali wants to talk to pfSense, it doesn’t say:

> “Send this to 192.168.10.1.”

It says:

> “Who has 192.168.10.1? Tell 192.168.10.10.”

This is **ARP** — Address Resolution Protocol.

ARP is:

- simple
- trusting
- unauthenticated
- spoofable
- manipulable

This is why ARP spoofing works.  
This is why MITM attacks work.  
This is why you can poison a network with a single packet.

Layer 2 is where **trust begins**, and therefore where **attacks begin**.

---

## **Layer 3 — The Network Layer (IP + Routing)**

This is where packets decide where to go.

IP addresses are not locations.  
They are **instructions**.

A packet doesn’t know where a server is.  
It only knows:

- “My destination is X.”
- “My next hop is Y.”

Routers don’t know the whole internet.  
They only know:

- “If the destination is in this range, send it here.”
- “If it’s not, send it to the default gateway.”

Routing is **local decisions creating global behavior**.

This is why:

- SSRF works
- internal port scanning works
- cloud metadata attacks work
- VPNs work
- proxies work
- NAT works

Layer 3 is where **movement** happens.

---

## **Layer 4 — The Transport Layer (TCP/UDP)**

This is where conversations happen.

TCP is:

- reliable
- ordered
- connection‑oriented
- polite

UDP is:

- fast
- stateless
- chaotic
- “fire and forget”

Ports live here.

Ports are not physical.  
They are **mailboxes**.

- 80 = HTTP
- 443 = HTTPS
- 22 = SSH
- 53 = DNS
- 3306 = MySQL

When you scan a machine, you’re knocking on its mailboxes.

Layer 4 is where **services** live.

---

## **Layer 7 — The Application Layer**

This is where humans live.

- HTTP
- DNS
- TLS
- APIs
- Cookies
- Sessions
- JSON
- JWT
- HTML
- JavaScript

This is where bug bounty hunters spend most of their time.

But here’s the secret:

> **Every Layer 7 vulnerability is built on top of Layer 3 and Layer 4 behavior.**

If you don’t understand the lower layers, you’re hacking blind.

---

# **III. TCP/IP: The Real Stack**

The OSI model is a teaching tool.  
TCP/IP is the real world.

TCP/IP collapses the OSI model into four layers:

- **Application** → HTTP, DNS, TLS
- **Transport** → TCP/UDP
- **Internet** → IP
- **Link** → Ethernet/Wi‑Fi

This is the stack you will see in:

- Wireshark
- packet captures
- kernel logs
- firewall rules
- routing tables
- NAT tables

This is the stack that matters.

---

# **IV. The Life of a Packet (The Most Important Section Today)**

Let’s walk through the most important event in networking:

### **Kali pings Ubuntu.**

This is the entire internet in miniature.

---

## **Step 1 — Kali checks its ARP cache**

“Do I already know the MAC address of my gateway (192.168.10.1)?”

If yes → use it.  
If no → broadcast:

```
Who has 192.168.10.1? Tell 192.168.10.10
```

This is ARP.  
This is trust.  
This is vulnerability.

---

## **Step 2 — pfSense replies**

“I am 192.168.10.1. My MAC is XX:XX:XX:XX:XX:XX.”

Kali stores this.

---

## **Step 3 — Kali sends ICMP Echo Request**

This is the “ping.”

It contains:

- source IP
- destination IP
- source MAC
- destination MAC
- ICMP payload

---

## **Step 4 — pfSense receives the packet**

pfSense checks:

- “Is this packet allowed?”
- “Where should it go?”

It forwards it to Ubuntu.

---

## **Step 5 — Ubuntu replies**

Ubuntu sends ICMP Echo Reply.

---

## **Step 6 — pfSense routes it back**

pfSense checks:

- “Is this part of an existing connection?”
- “Is this allowed?”

It forwards it to Kali.

---

## **Step 7 — Kali receives the reply**

Ping succeeds.

---

# **V. Why This Matters for Hacking**

Everything you will ever do in bug bounty or offensive security is built on this.

- **SSRF** is just “make the server send packets somewhere else.”
- **DNS rebinding** is “lie about where packets should go.”
- **MITM** is “lie about who you are at Layer 2.”
- **Port scanning** is “knock on Layer 4 mailboxes.”
- **Firewall bypass** is “trick Layer 3 rules.”
- **VPN pivoting** is “change the next hop.”
- **Cloud metadata attacks** are “exploit routing assumptions.”
- **CORS misconfigurations** are “trust the wrong origin.”

If you understand packet flow, you understand the internet.  
If you understand the internet, you understand hacking.

---

# 🧪 **LAB — Seeing the Internet With Your Own Eyes**

This is where Day 1 becomes real.

---

## **1. Start a packet capture on pfSense LAN**

Right‑click → Start Capture

This is your microscope.

---

## **2. From Kali:**

```
ping 192.168.20.10
```

Watch the ARP broadcast.  
Watch the ICMP packets.  
Watch the MAC addresses.  
Watch the routing.

You are watching the internet happen.

---

## **3. Break routing**

On Ubuntu:

```
ip route del default
```

Ping fails.

You have broken the internet.

---

## **4. Fix routing**

```
ip route add default via 192.168.20.1
```

Ping works.

You have repaired the internet.

---

# 🎯 **What You Should Feel in Your Bones by the End of Day 1**

- Networking is not magic.
- Packets are real things.
- ARP is the beginning of trust.
- Routing is the beginning of movement.
- TCP is the beginning of conversation.
- HTTP is the beginning of human meaning.
- Every vulnerability is built on these layers.
- You can now _see_ the internet, not just use it.

You are no longer a user of networks.  
You are a student of them.

---

If you want, I’ll continue with **Day 2** in the same style — deep, narrative, conceptual, practical, and hacker‑focused.