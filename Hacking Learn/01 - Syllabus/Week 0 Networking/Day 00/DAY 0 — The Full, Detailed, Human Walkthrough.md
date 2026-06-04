### Day 0 Introduction and Main Goals 🏁

Day 0 is the one day you do not spend learning theory — you spend building the world you’ll use to learn everything else. Today you’ll stop treating networking as invisible “internet stuff” and start treating it as a physical system you can touch, watch, and change: a tiny, real internet inside your laptop with an attacker, a firewall, and a target. The point is not to memorize terms; it’s to make the invisible visible so every later lesson lands on a living, observable system.

**Main goals for Day 0** — create the lab world (GNS3 + GNS3 VM, pfSense, Kali, Ubuntu) and wire them so packets must pass through the firewall; teach identity and direction by assigning static IPs and gateways so each machine knows who it is and where to send mail; make packets visible by enabling packet capture and learning to read ARP, ICMP, and basic TCP flows in Wireshark; see translation and control by enabling NAT and using the pfSense web UI so you can watch address rewriting and firewall decisions in action; prove connectivity and debug with pings, traceroutes, and simple TCP tests; connect theory to practice by always asking which layer changed (link, network, transport, application) and why that matters for attackers and defenders.

---

## DAY 0 — The Full, Detailed, Human Walkthrough (Everything, step by step) 🧱🌐

This is the version where we keep everything you already liked and add the intro you asked for. Read it in chunks. Do one small step at a time. I include exact commands, expected outputs, and the theory behind each action so you always know _what changed_ and _why it matters_.

---

### What you are building and why

You are building a tiny, real internet inside your laptop. It will have three machines and one border between them: Kali (the attacker / recon machine), pfSense (the router + firewall — the guard), and Ubuntu Server (the internal host / target — the house with services). The roads between them are virtual links you draw in GNS3. The whole point of Day 0 is to make invisible network mechanics visible: addressing, ARP, routing, NAT, firewall state, TCP handshakes, and packet capture. Every command you run will be tied to a short theory note explaining **why** it matters and **how** attackers or defenders use it.

---

### 1. The big ideas you need before you click anything

Before we touch the keyboard, hold these mental models in your head.

A packet is a letter. The application (HTTP, SSH) writes the letter. TCP or UDP wraps the letter in an envelope with a port number. IP puts that envelope into a bigger envelope with source and destination IPs. Ethernet wraps that into a physical parcel with MAC addresses so the local mail carrier can deliver it on the same road. This stacking is called **encapsulation**. When you capture traffic with Wireshark you’ll see these layers stacked — that’s your X‑ray view.

An IP address is both an identifier and a locator. The subnet mask tells a host which addresses are “local” and which are remote. If the destination is local, the host asks the local network “who has that IP?” using ARP. If the destination is remote, the host sends the packet to its default gateway — the router — which forwards it toward the destination.

MAC addresses are physical addresses used on a single link. ARP maps IP → MAC. ARP is unauthenticated, which is why ARP spoofing is a real attack.

Routing tables are simple rules: “if the destination matches this prefix, send it out that interface.” The default route (`0.0.0.0/0`) says “if you don’t know where to send it, send it to the gateway.”

NAT rewrites addresses and/or ports. SNAT rewrites the source so internal hosts can share one external IP. DNAT rewrites the destination so external traffic reaches an internal host. NAT hides internal addresses and complicates attribution.

A stateful firewall tracks connection state. When a TCP `SYN` starts, the firewall creates a state entry and allows replies that match that state. This is why a firewall can allow return traffic without an explicit rule for the reply.

TCP uses a three‑way handshake (`SYN`, `SYN‑ACK`, `ACK`) to establish a connection. Flags like `RST` and `FIN` control teardown. Scanners and IDS systems rely on flag behavior to detect or probe services.

DNS maps names to IPs. DNS behavior is central to many web attacks (rebinding, SSRF). In the lab you’ll run DNS queries and see how name resolution affects reachability.

Keep these models in mind. Every time you change a setting, ask: which layer did I change — link, network, transport, or application? That question will guide your debugging and your understanding of attacks and defenses.

---

### 2. Install GNS3 and the GNS3 VM — what you’re actually doing and why 🧩💡

When you install GNS3 you install two cooperating parts: the GUI (the control panel you click) and the GNS3 VM (the engine that runs the virtual devices). The GUI is the cockpit; the VM is the engine room. The GUI tells the VM to create virtual NICs, bridges, and VMs; the VM actually runs the appliances.

**Step by step**

Open your browser, download the GNS3 GUI for your OS, and run the installer. Accept the defaults. When the installer asks about optional components, accept the GNS3 VM helper and the Wireshark integration — they make life easier.

Download the GNS3 VM OVA and import it into VirtualBox or VMware: in VirtualBox choose **File → Import Appliance**, pick the OVA, click **Next**, then **Import**. After import, open the VM settings and give it at least 4 GB RAM and 2 CPU cores (more if your laptop can spare it). Under Network, leave the adapter as NAT for now.

Start the GNS3 VM and let it boot fully. You’ll see a Linux console with a login prompt — that’s normal.

Open the GNS3 GUI, go to **Edit → Preferences → GNS3 VM**, enable it, and point it to the VM you imported. Click **Test** or **Start**. When you see **GNS3 VM: Running** in the GUI, the cockpit and engine are connected. That green status means your GUI can ask the VM to create virtual machines and virtual links.

**Why this matters**

The GUI alone can’t run heavy network appliances. The VM provides CPU, RAM, and the virtualization layer so pfSense, Kali, and Ubuntu behave like real machines. If you skip the VM, things will be slow or fail.

---

### 3. Import pfSense, Kali, and Ubuntu — the devices you’ll use and what each teaches 🛠️🔌

You will import three real systems. Each one is a real OS image that runs inside the GNS3 VM. They behave like the real thing.

**pfSense — the guard.** Download the pfSense ISO. In GNS3 choose **File → New appliance** and import pfSense (or create a VM and attach the ISO). Boot pfSense. The first boot shows a text console and a small menu. That console is normal: pfSense is a network appliance and expects web UI management. The console lets you assign interfaces and do emergency recovery. When pfSense asks which interface is WAN and which is LAN, you’re defining the border between outside and inside.

**Kali — the attacker.** Download the Kali VM image and import it into GNS3. Choose virtio for the NIC type when prompted. Start Kali and open a terminal. Kali is your toolbox: nmap, Wireshark, curl, netcat, Burp Suite, tcpdump. It’s how you look at the network from an attacker’s perspective.

**Ubuntu Server — the target.** Download the Ubuntu Server ISO, create a VM in GNS3, attach the ISO, and install Ubuntu Server. During installation choose the minimal profile and enable OpenSSH. Servers don’t need a desktop; they need stability and services. Enabling SSH gives you a remote door to the machine — a realistic target.

**Why these systems**

pfSense is used in real networks; Kali contains real offensive tools; Ubuntu Server is a realistic target. Running these gives you real behavior — not toy behavior.

---

### 4. Wire the world — drawing the roads and naming doors 🛣️🔗

Place Kali, pfSense, and Ubuntu on the GNS3 canvas. Click the cable icon, click Kali, choose the first interface (it might be named `eth0`, `ens3`, or `vnet0`), then click pfSense and choose its LAN interface. That creates a link — a road — between Kali and pfSense. Repeat to connect pfSense’s WAN interface to Ubuntu’s eth0.

At this point you have three islands and two roads: Kali → pfSense → Ubuntu. Kali is on one side of the guard; Ubuntu is on the other. Any message from Kali to Ubuntu must pass through pfSense.

**Note about interface names**

Inside each VM the NIC may be named `eth0`, `ens3`, `enp0s3`, or `vnet0`. That’s fine. When you connect a cable in GNS3 you’ll be asked which interface to use; pick the first available and keep track of it.

---

### 5. Assign static IPs — teach the machines who they are and where to send mail 🏷️📬

We’ll use two neighborhoods: attacker side `192.168.10.0/24` and internal side `192.168.20.0/24`. Static IPs force you to understand addressing and routing.

On Kali, open a terminal and run:

```bash
sudo ip addr add 192.168.10.10/24 dev eth0
sudo ip route add default via 192.168.10.1
```

On Ubuntu, run:

```bash
sudo ip addr add 192.168.20.10/24 dev eth0
sudo ip route add default via 192.168.20.1
```

On pfSense, set the LAN IP to `192.168.10.1/24` and the WAN IP to `192.168.20.1/24` either via the console menu or the web UI.

**What these commands do**

`ip addr add` assigns an IP and netmask to the interface. `ip route add default via` tells the host where to send packets that are not for the local network — the default gateway.

**Theory**

The `/24` mask means the first 24 bits are the network prefix. Hosts with addresses `192.168.10.x` are local to the LAN; `192.168.20.x` are local to the WAN side. If a host wants to reach an address outside its `/24`, it sends the packet to its gateway.

---

### 6. pfSense web UI and NAT — the guard’s control panel and disguise trick 🛡️🔍

From Kali open a browser and go to `http://192.168.10.1`. The default username is `admin` and the default password is `pfsense`. Change the password immediately.

The pfSense web UI has a top menu and a left sidebar. The items we’ll use are Interfaces, Firewall, and Diagnostics. To enable NAT so Kali can reach Ubuntu, go to Firewall → NAT → Outbound and choose Automatic. Save and apply.

**Why NAT matters**

NAT rewrites the source IP so internal hosts can share one outward identity. When Kali sends a packet to Ubuntu, pfSense can rewrite the source so Ubuntu sees the packet coming from `192.168.20.1`. When Ubuntu replies, pfSense reverses the translation and forwards the reply to Kali. NAT is the guard’s disguise trick and is everywhere on the internet.

**How this connects to hacking**

Attackers use NAT to hide and to pivot. Defenders use NAT logs to trace activity. Seeing NAT in action helps you understand both offense and defense.

---

### 7. Test the roads — ping and interpret the results 📨👂

From Kali:

```bash
ping -c 4 192.168.20.10
```

A successful reply looks like:

```
PING 192.168.20.10 (192.168.20.10) 56(84) bytes of data.
64 bytes from 192.168.20.10: icmp_seq=1 ttl=64 time=1.23 ms
64 bytes from 192.168.20.10: icmp_seq=2 ttl=64 time=1.10 ms
--- 192.168.20.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
```

If you get replies, the packet left Kali, passed the guard, and reached the house. If you don’t, the packet was stopped somewhere — and that’s a lesson.

**If ping fails, check in this order**

On Kali run:

```bash
ip addr show eth0
ip route show
```

Confirm the interface is up and the default route points to `192.168.10.1`. On pfSense check Interfaces and the routing table. On Ubuntu confirm the IP and gateway. Check NAT and firewall rules. Each check tests a hypothesis: interface down, wrong IP, missing route, firewall blocking, NAT misconfigured, or service not running.

**Theory**

`ip route show` reveals the routing table. If the default route is missing, the host won’t send packets to the gateway. `ip addr` shows whether the interface is up and has the expected IP. These are the most common causes of failure.

---

### 8. Put a camera on the road — packet capture and how to read what you see 🔬📸

Right‑click the link between Kali and pfSense in GNS3 and choose Start capture. GNS3 will open Wireshark and show live packets.

**What you’ll see first and why it matters**

ARP requests: a host asks “Who has 192.168.10.1? Tell 192.168.10.10.” ARP is how IP → MAC mapping happens. Because ARP is unauthenticated, it can be spoofed.

ICMP (ping): you’ll see Echo Request and Echo Reply. Expand the IP header and note TTL and checksums. TTL decrements at each hop; traceroute uses TTL to map hops.

TCP handshake: when you open a TCP connection you’ll see `SYN`, `SYN‑ACK`, `ACK`. Expand the TCP header and inspect flags and sequence numbers. Sequence numbers ensure ordered delivery.

NAT translation: capture on both sides of pfSense and you’ll see the source IP change from `192.168.10.10` on the LAN side to `192.168.20.1` on the WAN side. That’s NAT in action.

**Useful Wireshark filters**

To show only ARP: `arp`  
To show only ICMP: `icmp`  
To show traffic between two IPs: `ip.addr == 192.168.10.10 && ip.addr == 192.168.20.10`  
To follow a TCP stream: right‑click a TCP packet → Follow → TCP Stream

**Why packet capture is the microscope**

You can’t fix or understand what you can’t see. Wireshark shows the raw letters moving across the road. Once you can read them, you can reason about why a connection fails or how an attacker might intercept or modify traffic.

---

### 9. Basic services and a first TCP test — netcat and the three‑way handshake ⚙️🔗

On Ubuntu install netcat and start a listener:

```bash
sudo apt update
sudo apt install -y netcat
nc -l -p 8080
```

This opens a TCP listener on port 8080. On Kali connect:

```bash
nc 192.168.20.10 8080
```

Type something on Kali and you’ll see it on Ubuntu’s terminal. In Wireshark you’ll see the TCP three‑way handshake: `SYN`, `SYN‑ACK`, `ACK`. That sequence establishes a TCP connection.

**Theory**

`SYN` means “I want to start a connection.” `SYN‑ACK` means “I hear you and I accept.” `ACK` completes the handshake. Scanners and IDS systems look at these flags to determine whether a port is open or closed.

---

### 10. Useful commands you’ll use again and again 🧰⌨️

Show IP addresses and interfaces:

```bash
ip addr show
```

Show routing table:

```bash
ip route show
```

Test connectivity:

```bash
ping -c 4 192.168.20.10
```

Trace the path packets take:

```bash
traceroute 192.168.20.10
```

Capture packets on a Linux host:

```bash
sudo tcpdump -i eth0 -w capture.pcap
# then open capture.pcap in Wireshark
```

Start a simple HTTP server on Ubuntu:

```bash
python3 -m http.server 8000
```

Fetch the page from Kali:

```bash
curl http://192.168.20.10:8000
```

Each command gives you a different view: addresses, routes, live packets, and application behavior.

---

### 11. Common beginner mistakes and the theory behind the fixes 🔎🧭

If Kali can’t ping pfSense, check Kali’s IP and gateway. If pfSense can’t reach Ubuntu, check pfSense’s WAN IP and Ubuntu’s gateway. If ping works but TCP services fail, check pfSense firewall rules — pfSense may allow ICMP but block TCP ports by default. If Wireshark shows ARP but no replies, the other machine may be down or on a different subnet. If you see packets but no application response, the network is fine but the service might not be listening — confirm with `ss -ltn` or `netstat -tuln`.

**Theory**

Network problems are hypothesis tests. Each command you run confirms or rejects a hypothesis: interface up, IP correct, route present, firewall allowing traffic, NAT translating, service listening. This scientific approach is how you learn networking deeply.

---

### 12. Exercises to cement the learning and what each teaches 🧪📘

Exercise 1 — Follow a single packet end‑to‑end. From Kali, ping Ubuntu while capturing on both links. In Wireshark find the ICMP request on the LAN capture, find the corresponding frame on the WAN capture, and observe NAT translation. This teaches encapsulation and NAT.

Exercise 2 — SYN scan and banner grab. From Kali run:

```bash
nmap -sS -p 22,80,8080 192.168.20.10
curl -v http://192.168.20.10:8000/
```

Watch the SYN, SYN‑ACK, ACK in Wireshark and then the HTTP request/response. This teaches TCP flags, scanning, and application layer behavior.

Exercise 3 — ARP spoof MITM (lab only). On Kali enable IP forwarding and run `arpspoof` to claim the gateway. Capture traffic and observe how packets now flow through Kali. This teaches link‑layer attacks and why ARP is dangerous.

Exercise 4 — SSH reverse tunnel (pivoting). On Ubuntu (after you have a shell) run:

```bash
ssh -R 2222:localhost:22 kaliuser@192.168.10.10
```

From Kali connect to `localhost:2222` to reach Ubuntu’s SSH via the tunnel. This teaches tunneling and pivoting.

Do these one at a time. Each is short and teaches a single concept.

---

### 13. How each concept connects to real attacks and defenses 🎯🛡️

ARP spoofing lets an attacker intercept traffic on a local link. Port scanning and banner grabbing are reconnaissance steps used before exploitation. Pivoting and tunneling let attackers move from one compromised host to others. DNS rebinding and SSRF exploit name resolution and server behavior. Defenders use logs (pfSense firewall logs, NAT translations), IDS signatures, and packet captures to detect anomalies: unexpected ARP replies, unusual port scans, or persistent tunnels.

**Theory to practice**

Every attack technique maps to a layer or mechanism you’ve just built: ARP spoofing targets the link layer, SYN scans target the transport layer, SSRF targets the application layer, and pivoting uses tunnels that operate across layers. Understanding the layer and the mechanism tells you where to look for detection and how to harden systems.

---

### 14. Troubleshooting checklist with the reasoning behind each step

When something fails, follow this chain — each step is a hypothesis test.

First, is the interface up? If `ip addr` shows the interface down, the NIC isn’t connected. No link means no frames. Fix: check GNS3 link and VM NIC.

Second, is the IP correct? Wrong IP or mask makes the host decide incorrectly whether the destination is local or remote. Fix: correct IP/mask.

Third, is the default gateway set? Without a gateway the host won’t send to remote networks. Fix: `ip route add default via <gateway>`.

Fourth, is the firewall blocking the traffic? pfSense rules are ordered; the first matching rule applies. Fix: add a rule or temporarily allow the port.

Fifth, is NAT translating correctly? If NAT is misconfigured replies may not be mapped back. Fix: check NAT rules and logs.

Sixth, is the service running? If the network is fine but the service isn’t listening, connections fail. Fix: start the service and confirm with `ss -ltn`.

Each check narrows the cause. This is how you learn to think like a network engineer and a hacker.

---

### 15. The mental model to keep in your head 🧭

Think of the network as a postal system. IP addresses are mailbox numbers. MAC addresses are the physical house numbers used by the local mail carrier. ARP is the neighbor asking “who lives here?” Routing tables are the postal sorting rules. NAT is the guard who rewrites return addresses. Firewalls are customs officers who check packages. Wireshark is your X‑ray machine that lets you see the package contents and the stamps.

When you can see a packet leave Kali, be rewritten by pfSense, and arrive at Ubuntu, you stop memorizing theory and start understanding cause and effect. That understanding is what makes you effective at both building and breaking networks.

---

If you want, I’ll now expand one of these into a full clickable walkthrough with exact menu names and described screenshots: a pfSense deep dive (change admin password, create firewall rules, NAT mappings, enable logging), a Wireshark tutorial (filters, follow stream, interpret fields), or a first attack lab (nmap scan, interpret results, capture traffic, connect to a service). Tell me which one and I’ll produce it in the same detailed, theory‑rich style with exact commands and expected outputs.