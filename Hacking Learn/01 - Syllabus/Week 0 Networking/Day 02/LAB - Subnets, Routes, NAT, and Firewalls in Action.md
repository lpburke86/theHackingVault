Absolutely, Liam.  
Here is **Day 2 — LAB 1**, written in the **exact same style** as your Day 1 labs:

- **Long, flowing, narrative paragraphs**
- **Deep theory woven into every step**
- **Code blocks only where they represent real, observable actions**
- **No bullet‑point lists except the TOC**
- **Full walkthrough with “what” and “why” explanations**
- **Artifact creation, timestamps, captures, routing tables, NAT tables, firewall rules**
- **The same density and tone as Labs 1–4**

This is the **complete, integrated, expanded Day 2 Lab**, ready for your vault.

---

# **🧪 DAY 2 — LAB - Subnets, Routes, NAT, and Firewalls in Action (Complete Deep Dive)**

---

# 🧭 **Table of Contents**

- [[# 1. Lab overview — shaping the network rather than observing it]]
- [[# 2. Lab goals — what this lab teaches you]]
- [[# 3. Environment and mental model — building a multi‑subnet world]]
- [[# 4. Walkthrough part 1 — creating a second subnet and giving it life 🌱]]
- [[# 5. Walkthrough part 2 — teaching pfSense how to reach new territory 🧭]]
- [[# 6. Walkthrough part 3 — creating firewall policy and enforcing borders 🔥]]
- [[# 7. Walkthrough part 4 — testing segmentation from Kali 🧪]]
- [[# 8. Walkthrough part 5 — capturing blocked packets and watching the firewall think 👁️]]
- [[# 9. Walkthrough part 6 — observing routing tables, NAT tables, and firewall state 🧠]]
- [[# 10. Artifact strategy — manifests, timestamps, and narrative structure]]
- [[# 11. Reflection prompts — internalizing network geometry]]

---

# **1. Lab overview — shaping the network rather than observing it**

Day 1 taught you how to observe a packet. Day 2 teaches you how to **shape the world that packet moves through**. This lab is where you stop being a passive observer and start becoming a network architect. You will carve a new subnet into existence, connect it to the existing network, teach pfSense how to route between them, create firewall rules that allow some traffic and block others, and then watch the consequences ripple through the system.

This lab is not about typing commands. It is about **reshaping the geometry of the network** and then watching how packets respond to the new terrain. You will see how a subnet boundary changes reachability, how a routing table changes movement, how NAT changes identity, and how a firewall rule changes destiny.

[[#🧭 Table of Contents]]

---

# **2. Lab goals — what this lab teaches you**

By the end of this lab, you should be able to create a subnet, attach it to a router, configure routing, apply firewall rules, and verify segmentation using real traffic. You should be able to explain why a packet succeeds or fails, not in vague terms but in precise, structural language. You should be able to read a routing table and understand the worldview of a machine. You should be able to read a NAT table and understand how identities are rewritten. You should be able to read firewall logs and understand how policy is enforced.

This lab teaches you how to **shape the map**, not just read it.

[[#🧭 Table of Contents]]

---

# **3. Environment and mental model — building a multi‑subnet world**

Before you begin, you must build a mental model of the world you are about to create. You already have a subnet — the 192.168.20.0/24 network where Kali and Ubuntu live. pfSense sits at 192.168.20.1, acting as the gateway. This is your first neighborhood.

You will now create a second neighborhood — the 192.168.30.0/24 subnet — and attach it to pfSense. This new subnet will contain a new Ubuntu machine. pfSense will become the gateway for both subnets, and you will teach it how to route between them.

You begin by creating a directory structure for Day 2’s lab:

```bash
mkdir -p ~/day2_lab/{pcaps,scans,notes,screenshots,artifacts}
date -u > ~/day2_lab/scans/lab2_start_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

This timestamp marks the moment the network begins to change.

[[#🧭 Table of Contents]]

---

# **4. Walkthrough part 1 — creating a second subnet and giving it life 🌱**

You start by creating a new Ubuntu VM in GNS3. This machine will live in the new subnet. You assign it the IP address `192.168.30.10/24` and set its gateway to `192.168.30.1`. This gateway does not exist yet, but it will.

You then add a new interface to pfSense. In GNS3, you connect a new link from pfSense to the new Ubuntu VM. In pfSense, you navigate to Interfaces → Assignments and add the new interface. You name it OPT1. You enable it and assign it the IP address `192.168.30.1/24`.

At this moment, the new subnet exists. It is a newborn territory — a land with one inhabitant and one gateway. But it is isolated. pfSense knows it exists, but nothing else does.

You verify the new Ubuntu machine’s configuration:

```bash
ip addr
ip route
```

You should see that its default route points to `192.168.30.1`. This is the first sign of life.

[[#🧭 Table of Contents]]

---

# **5. Walkthrough part 2 — teaching pfSense how to reach new territory 🧭**

pfSense now has two interfaces: one in the 20.x subnet and one in the 30.x subnet. But pfSense does not yet know how to route traffic between them. You must teach it.

In pfSense, you navigate to System → Routing → Static Routes. You add a route for the `192.168.30.0/24` network with a gateway of `192.168.30.1`. This tells pfSense that any packet destined for the 30.x subnet should be sent out the OPT1 interface.

You verify pfSense’s routing table:

```bash
netstat -rn
```

You should see entries for both subnets. pfSense now understands the geometry of the world.

[[#🧭 Table of Contents]]

---

# **6. Walkthrough part 3 — creating firewall policy and enforcing borders 🔥**

A network without policy is chaos. Now that pfSense knows how to route between subnets, you must decide what is allowed and what is forbidden.

In pfSense, you navigate to Firewall → Rules → OPT1. You create a rule that allows HTTP traffic on port 80. You create a rule that blocks SSH traffic on port 22. You create a rule that allows ICMP.

These rules are not arbitrary. They are the first expression of **policy**. They are the first borders you draw. They are the first constraints you impose on the movement of packets.

You verify the rules:

```bash
pfctl -sr
```

You should see the rules listed in the order you created them. This order matters. Firewalls process rules from top to bottom. A misplaced rule can change the fate of a packet.

[[#🧭 Table of Contents]]

---

# **7. Walkthrough part 4 — testing segmentation from Kali 🧪**

Now you test the geometry you have created. From Kali, you attempt to reach the new Ubuntu machine in the 30.x subnet.

You begin with ICMP:

```bash
ping -c 1 192.168.30.10
```

If your routing and firewall rules are correct, the ping should succeed. This proves that pfSense can route between subnets and that ICMP is allowed.

You then test HTTP:

```bash
curl http://192.168.30.10
```

If the new Ubuntu machine is running a web server, you should see a response. If it is not, you will see a connection but no content. Either way, the connection should succeed because port 80 is allowed.

You then test SSH:

```bash
ssh 192.168.30.10
```

This should fail. The firewall rule blocking port 22 should silently drop the SYN packet. Kali will hang for a moment and then report a timeout. This is segmentation in action.

[[#🧭 Table of Contents]]

---

# **8. Walkthrough part 5 — capturing blocked packets and watching the firewall think 👁️**

Now you observe the firewall enforcing policy. In pfSense, you navigate to Diagnostics → Packet Capture. You set the interface to OPT1 and the filter to:

```
host 192.168.30.10 and port 22
```

You start the capture. You return to Kali and attempt SSH again:

```bash
ssh 192.168.30.10
```

You stop the capture and download the pcap. You open it in Wireshark. You see the SYN packet from Kali. You see nothing else. No SYN/ACK. No RST. No ICMP error. The firewall silently drops the packet.

This is what “blocked” looks like. It is not dramatic. It is not noisy. It is absence. It is silence. It is a packet that dies without a trace.

[[#🧭 Table of Contents]]

---

# **9. Walkthrough part 6 — observing routing tables, NAT tables, and firewall state 🧠**

Now you examine the internal state of the network. On pfSense, you inspect the routing table:

```bash
netstat -rn
```

You see entries for both subnets. You see the default route. You see the geometry of the network expressed as a table.

You inspect the NAT table:

```bash
pfctl -sn
```

You see translations for outbound traffic. You see how internal identities are rewritten when packets leave the network.

You inspect the firewall state table:

```bash
pfctl -ss
```

You see active connections. You see which packets are allowed to live and which are denied.

This is the network’s memory. This is how pfSense understands the world.

[[#🧭 Table of Contents]]

---

# **10. Artifact strategy — manifests, timestamps, and narrative structure**

You create a manifest file:

```bash
nano ~/day2_lab/notes/manifest_day2_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

Inside, you record every action, every timestamp, every pcap, every command output, every firewall rule, every routing change. This manifest becomes the skeleton of your narrative.

You package the lab:

```bash
zip -r ~/day2_lab/artifacts/day2_lab_package_$(date -u +"%Y%m%dT%H%M%SZ").zip ~/day2_lab/
```

This archive is your evidence. It is your proof. It is your story.

[[#🧭 Table of Contents]]

---

# **11. Reflection prompts — internalizing network geometry**

You write your reflections in:

```bash
nano ~/day2_lab/notes/reflections_day2_$(date -u +"%Y%m%dT%H%M%SZ").md
```

You answer questions like:

What does segmentation feel like now that you have created it?  
How does routing shape the movement of packets?  
How does NAT distort identity?  
How does a firewall enforce policy?  
How does geometry become security?

These reflections turn commands into intuition.

---

If you want, we can now move to **Day 3**, or we can begin **Day 2 — Lab 2** if you want multiple labs per day.