Liam, buckle up — because **Day 0** is about to become the most important “non‑learning” day of the entire course.  
This is the day where we build the _world_ you’ll be learning in.  
And if we build it right, everything else becomes easier, clearer, and more intuitive.

This isn’t just “install some VMs.”  
This is **building a miniature internet**, one you can break, fix, observe, manipulate, and understand at a level most people never reach.

I’m going to give you:

- **Why** we’re doing each step
- **What it teaches you**
- **How it connects to hacking**
- **How it prepares you for the 10‑week course**
- **The exact steps**
- **The mental model you should walk away with**


### _“Before you learn networking, you’re going to build a network.”_

Most people learn networking backwards.

They start with theory.  
They memorize OSI layers.  
They read about TCP flags.  
They watch videos about routing.

And none of it sticks.

Why?  
Because networking is **not a theoretical discipline**.  
It’s a **physical system**.  
It’s wires and packets and interfaces and routes and tables and rules and flows.

You don’t learn to swim by reading about water.  
You learn by getting in the pool.

So Day 0 is you **building the pool**.

---

# 🧠 **Why We Use GNS3 Instead of Just VMs**

You could technically learn networking with just two VMs and VirtualBox NAT.  
But that teaches you nothing about:

- routing
- NAT
- firewalls
- packet flow
- interfaces
- subnets
- gateways
- ARP
- DHCP
- DNS
- packet capture on links
- how real networks are built

VirtualBox NAT is a black box.  
It hides everything that matters.

GNS3, on the other hand, is:

- visual
- modular
- transparent
- realistic
- used by real network engineers
- used by pentesters to simulate enterprise networks
- capable of capturing packets on any link
- capable of simulating routers, firewalls, switches, and hosts

GNS3 is the **closest thing to a real network** you can build on a laptop.

And once you understand how to build networks, you understand how to break them.

---

# 🧠 **Why pfSense?**

pfSense is:

- a real firewall
- a real router
- a real NAT device
- a real packet filter
- a real VPN endpoint
- a real enterprise‑grade network appliance

It’s not a toy.  
It’s not a simulation.  
It’s the same software used in:

- small businesses
- labs
- homelabs
- some enterprise edge networks

When you understand pfSense, you understand:

- how firewalls think
- how NAT works
- how routing tables work
- how packets are filtered
- how VPNs are terminated
- how port forwarding works
- how attackers bypass rules
- how defenders block attacks

pfSense is your **gateway drug** to real networking.

---

# 🧠 **Why Ubuntu Server?**

Because it’s:

- simple
- predictable
- stable
- easy to configure
- easy to break
- easy to fix
- perfect for hosting:
    - DNS
    - HTTP
    - HTTPS
    - SSH
    - vulnerable services
    - custom scripts
    - internal endpoints for SSRF labs

Ubuntu Server is your **target**.  
It’s the machine you will:

- scan
- enumerate
- exploit
- MITM
- tunnel into
- break
- fix
- observe

It’s the “victim” in your miniature internet.

---

# 🧠 **Why Kali Linux?**

Because Kali is:

- the attacker
- the recon machine
- the packet sniffer
- the scanner
- the proxy
- the MITM box
- the exploitation platform

It contains:

- nmap
- Wireshark
- curl
- Burp Suite
- ettercap
- netcat
- dig
- traceroute
- tcpdump
- proxychains
- metasploit (later)

Kali is your **eyes**, your **hands**, and your **weapons**.

---

# 🧠 **Why This Topology?**

Here’s the network you’re building:

```
Kali ─── pfSense ─── Ubuntu
```

This is the **simplest possible network** that still teaches you:

- routing
- NAT
- firewall rules
- packet flow
- DNS
- HTTP/HTTPS
- scanning
- enumeration
- MITM
- tunneling
- proxying
- SSRF
- DNS rebinding
- internal port scanning
- segmentation
- subnetting

This is the **internet in miniature**.

Kali = your laptop  
pfSense = your home router / corporate firewall  
Ubuntu = the website you’re attacking

Once you understand this, you understand the real world.

---

# 🧠 **Why We Assign Static IPs**

You could use DHCP.  
But DHCP hides the most important part of networking:

> “How does a machine know where to send packets?”

Static IPs force you to understand:

- IP addressing
- subnet masks
- gateways
- routing tables
- interface configuration

This is foundational.

---

# 🧠 **Why We Capture Packets on Day 0**

Because packet captures are the microscope of networking.

You can’t understand:

- ARP
- ICMP
- DNS
- HTTP
- TLS
- routing
- NAT
- scanning
- MITM
- tunneling

…unless you can _see_ the packets.

Wireshark is the **Rosetta Stone** of networking.

You will use it every day.

---

# 🧰 **DAY 0 — Step‑by‑Step Setup**

Now let’s build the lab.

---

# 🔧 **Step 1 — Install GNS3**

Download from:

[https://www.gns3.com/software/download](https://www.gns3.com/software/download)

Install:

- GNS3 GUI
- GNS3 VM
- VirtualBox or VMware

Open GNS3 → Preferences → GNS3 VM → Enable → Select VM

When it works, you’ll see:

```
GNS3 VM: Running
```

This means GNS3 can offload heavy lifting to the VM.

---

# 🔧 **Step 2 — Import pfSense**

Download pfSense CE ISO.  
In GNS3:

- File → New Appliance
- pfSense → Import
- Attach ISO
- Finish

pfSense will boot into a console.  
You’ll configure it later.

---

# 🔧 **Step 3 — Import Kali Linux**

Download Kali VM.  
Import into GNS3.  
Set network adapter to **virtio-net**.

---

# 🔧 **Step 4 — Import Ubuntu Server**

Download Ubuntu Server ISO.  
Install it inside GNS3.  
Minimal install.  
OpenSSH server enabled.

---

# 🔧 **Step 5 — Build the Topology**

Drag:

- Kali
- pfSense
- Ubuntu

Connect:

```
Kali → pfSense LAN
pfSense WAN → Ubuntu
```

This is your internet.

---

# 🔧 **Step 6 — Assign IPs**

### Kali:

```
ip addr add 192.168.10.10/24 dev eth0
ip route add default via 192.168.10.1
```

### pfSense:

LAN: `192.168.10.1/24`  
WAN: `192.168.20.1/24`

### Ubuntu:

```
ip addr add 192.168.20.10/24 dev eth0
ip route add default via 192.168.20.1
```

---

# 🔧 **Step 7 — Enable NAT on pfSense**

pfSense → Firewall → NAT → Outbound → Automatic

This lets Kali reach Ubuntu through pfSense.

---

# 🔧 **Step 8 — Verify Connectivity**

From Kali:

```
ping 192.168.20.10
```

If it works → you’re ready.  
If not → good.  
Troubleshooting is learning.

---

# 🔧 **Step 9 — Enable Packet Capture**

Right‑click any link → Start Capture

This is your microscope.

---

# 🎉 **DAY 0 Outcome**

By the end of Day 0, you have:

- A functioning 3‑node network
- A real router/firewall
- A real server
- An attacker machine
- Packet capture on every link
- A miniature internet you control

You are now ready for **[[🧨 DAY 1 — The Shape of the Internet- OSI, TCP-IP, and the Life of a Packet]]**, where we stop treating networking like a list of facts and start treating it like a living system.

