### _Week 06 — MITM, Proxying, Tunneling, Traffic Manipulation_

**Estimated Time:** 90–120 minutes  
**Difficulty:** Intermediate → Advanced  
**Tools:** Burp Suite Community Edition, ARP spoofing tool (ettercap / arpspoof), Firefox/Chromium, DVWA/Mutillidae, local LAN environment

---

## **1. Purpose of This Lab**

This lab teaches students how to perform a **transparent, invisible man‑in‑the‑middle (MITM)** attack using ARP spoofing combined with Burp Suite as the interception and manipulation engine.

By the end of this lab, students will be able to:

- Perform ARP spoofing to silently position themselves between two hosts
- Route victim traffic through Burp Suite without browser proxy settings
- Intercept and modify HTTP/HTTPS traffic invisibly
- Understand how Burp handles TLS interception during MITM
- Analyze victim traffic in Burp without touching the victim’s configuration
- Validate MITM success using controlled tests

This lab demonstrates how Burp Suite becomes a **network‑level MITM tool**, not just a browser proxy.

---

## **2. Pre‑Lab Requirements**

Students must have:

- Completed Labs 6.1–6.3
- A LAN environment with:
    - Kali attacker
    - Victim machine (Windows or Linux)
    - pfSense or router
- Burp Suite configured with its CA certificate installed **only on the attacker**, not the victim
- IP forwarding enabled on Kali

---

## **3. Lab Tasks**

---

# **Task 1 — Enable IP Forwarding on Kali**

### **Objective**

Allow Kali to forward packets between victim and gateway.

### **Steps**

Enable forwarding:

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Make it persistent (optional):

```
nano /etc/sysctl.conf
net.ipv4.ip_forward=1
```

### **What to Observe**

- Kali now acts as a router
- Traffic can flow through Kali transparently

### **Deliverable**

A note confirming IP forwarding is enabled.

---

# **Task 2 — Perform ARP Spoofing to Become the Gateway**

### **Objective**

Silently position Kali between the victim and the router.

### **Steps**

Identify victim and gateway IPs:

```
ip neigh
```

Run ARP spoofing:

```
arpspoof -t <victim_ip> <gateway_ip>
arpspoof -t <gateway_ip> <victim_ip>
```

Or use ettercap GUI.

### **What to Observe**

- Victim ARP table now maps gateway IP → Kali MAC
- Gateway ARP table maps victim IP → Kali MAC

### **Deliverable**

A screenshot or note showing the victim’s ARP table pointing to Kali.

---

# **Task 3 — Configure Burp Suite for Transparent Proxying**

### **Objective**

Capture traffic without requiring proxy settings on the victim.

### **Steps**

1. In Burp → **Proxy → Options**
2. Add a new listener:
    - Bind to: **Kali’s LAN IP**
    - Port: **8080**
    - Check: “Support invisible proxying”
3. Ensure “Redirect to host” is disabled

### **What to Observe**

- Burp now accepts traffic not explicitly configured for proxy use
- Victim traffic can be intercepted without browser configuration

### **Deliverable**

A screenshot of the invisible proxy listener configuration.

---

# **Task 4 — Redirect Victim Traffic to Burp Using iptables**

### **Objective**

Force victim HTTP/HTTPS traffic through Burp.

### **Steps**

Redirect HTTP:

```
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

Redirect HTTPS:

```
iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8080
```

### **What to Observe**

- Victim traffic is silently routed to Burp
- Victim does not need proxy settings

### **Deliverable**

A note confirming iptables rules are active.

---

# **Task 5 — Intercept Victim Traffic in Burp**

### **Objective**

Verify that MITM interception is working.

### **Steps**

1. On the victim machine, open a browser.
2. Visit any HTTP site.
3. Observe the request in Burp → **Proxy → HTTP History**.

### **What to Observe**

- Victim traffic appears in Burp
- Victim is unaware of interception
- No proxy configuration required

### **Deliverable**

A screenshot or note confirming victim traffic is visible in Burp.

---

# **Task 6 — Test HTTPS Interception Behavior**

### **Objective**

Understand how TLS behaves during invisible MITM.

### **Steps**

1. On the victim, visit:
    
    ```
    https://example.com
    ```
    
2. Observe the browser behavior.

### **Expected Behavior**

- The victim browser will show a certificate warning
- Because the Burp CA is **not** installed on the victim
- This demonstrates TLS trust boundaries

### **Deliverable**

A short explanation of why HTTPS interception fails without CA installation.

---

# **Task 7 — Modify Victim Traffic in Real Time**

### **Objective**

Demonstrate active manipulation of victim traffic.

### **Steps**

1. Turn **Intercept ON** in Burp.
2. On the victim, visit any HTTP page.
3. Modify:
    - Headers
    - Parameters
    - Cookies
4. Forward the request.

### **What to Observe**

- Victim receives modified content
- Victim is unaware of manipulation
- MITM is fully transparent

### **Deliverable**

A list of at least **3 manipulations** and their effects on the victim.

---

# **Task 8 — Validate MITM Failure Modes**

### **Objective**

Understand how MITM breaks and how to detect it.

### **Steps**

1. Stop ARP spoofing.
2. Clear victim ARP cache:
    - Windows: `arp -d *`
    - Linux: `ip neigh flush all`
3. Attempt to load a page on the victim.

### **What to Observe**

- Traffic no longer passes through Kali
- Burp stops receiving requests
- Victim regains direct connection

### **Deliverable**

A short explanation of how ARP cache and spoofing affect MITM persistence.

---

## **4. Completion Criteria**

A student has successfully completed Lab 6.4 when they can:

- Enable IP forwarding
- Perform ARP spoofing
- Configure Burp for invisible proxying
- Redirect HTTP/HTTPS traffic using iptables
- Intercept and modify victim traffic
- Explain TLS interception limitations
- Diagnose MITM failure modes

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 7.1 — SSRF via Burp Repeater** in the same course style.