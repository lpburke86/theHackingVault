### _Week 06 — MITM, Proxying, Tunneling, Traffic Manipulation_

**Estimated Time:** 90–120 minutes  
**Difficulty:** Intermediate → Advanced  
**Tools:** Burp Suite Community Edition, SSH client, Firefox/Chromium, DVWA/Mutillidae, internal lab network

---

## **1. Purpose of This Lab**

This lab teaches students how to route Burp Suite traffic through an **SSH dynamic proxy (SOCKS5 tunnel)** to access internal systems that are not directly reachable from their attacking machine.

By the end of this lab, students will be able to:

- Create SSH dynamic tunnels
- Configure Burp Suite to use SOCKS5 proxy chaining
- Access internal services through a compromised host
- Understand how tunneling bypasses segmentation
- Combine Burp + SSH for internal reconnaissance
- Validate tunnel behavior using controlled tests

This lab is foundational for pivoting, SSRF chaining, and internal network exploitation in Weeks 7–9.

---

## **2. Pre‑Lab Requirements**

Students must have:

- Completed Lab 6.1 (request/response manipulation)
- A reachable SSH target (Ubuntu server or similar)
- Burp Suite configured as the browser proxy
- At least one internal service that is **not** directly reachable from Kali

Example topology:

```
Kali → (SSH) → Ubuntu → Internal Web App (10.10.10.x)
```

---

## **3. Lab Tasks**

---

# **Task 1 — Create an SSH Dynamic Proxy (SOCKS5 Tunnel)**

### **Objective**

Establish a SOCKS5 proxy that routes traffic through the SSH host.

### **Steps**

Run the following command from Kali:

```
ssh -D 9050 -C -N user@192.168.20.20
```

Explanation:

- `-D 9050` → creates a SOCKS5 proxy on port 9050
- `-C` → compresses traffic
- `-N` → no remote command; tunnel only
- `user@host` → SSH target

Keep this terminal open; it is your tunnel.

### **What to Observe**

- No output is expected
- The SSH session stays open
- Port 9050 is now a SOCKS5 proxy

### **Deliverable**

A note confirming the tunnel is active and listening.

---

# **Task 2 — Configure Burp Suite to Use the SSH Tunnel**

### **Objective**

Chain Burp’s outbound traffic through the SOCKS5 proxy.

### **Steps**

1. Open Burp → **User Options → Connections**
2. Under **SOCKS Proxy**, configure:
    - Host: `127.0.0.1`
    - Port: `9050`
    - SOCKS version: **5**
3. Enable “Use SOCKS proxy for DNS lookups”

### **What to Observe**

- Burp now routes all outbound traffic through the SSH tunnel
- DNS resolution happens on the remote host

### **Deliverable**

A screenshot or note confirming Burp’s SOCKS settings.

---

# **Task 3 — Verify Tunnel Functionality Using a Public IP Checker**

### **Objective**

Confirm that Burp’s outbound traffic is now originating from the SSH host.

### **Steps**

1. With Burp proxy enabled, visit:
    
    ```
    https://ifconfig.me
    ```
    
2. Observe the IP address returned.

### **Expected Result**

The IP should be the **SSH host’s public IP**, not Kali’s.

### **Deliverable**

A note confirming the IP change and what it implies.

---

# **Task 4 — Access an Internal Web Application Through the Tunnel**

### **Objective**

Use the tunnel to reach an internal service that Kali cannot access directly.

### **Steps**

1. Identify an internal service reachable from the SSH host (e.g., `10.10.10.5:8080`).
2. Attempt to access it directly from Kali (should fail).
3. Access it through the browser with Burp enabled:
    
    ```
    http://10.10.10.5:8080
    ```
    
4. Observe the request in Burp → **HTTP History**.

### **What to Observe**

- The request succeeds through the tunnel
- Burp logs the request normally
- The internal service is now reachable

### **Deliverable**

A short explanation of how the tunnel bypassed segmentation.

---

# **Task 5 — Perform Internal Reconnaissance Through the Tunnel**

### **Objective**

Use Burp to map internal applications.

### **Steps**

1. Browse the internal app normally.
2. Watch Burp’s **Target → Site Map** populate.
3. Identify:
    - Hidden endpoints
    - Admin panels
    - API routes
    - Parameterized endpoints

### **What to Observe**

- Internal apps often expose more functionality
- Authentication may be weaker
- Debug endpoints may be present

### **Deliverable**

A list of at least **5 internal endpoints** discovered through the tunnel.

---

# **Task 6 — Validate DNS Behavior Through the Tunnel**

### **Objective**

Understand how DNS resolution changes when using SOCKS5.

### **Steps**

1. Visit a domain that only resolves internally (e.g., `internal.local`).
2. Observe Burp’s DNS behavior.
3. Confirm that DNS resolution occurs on the SSH host.

### **What to Observe**

- Kali cannot resolve the domain
- The SSH host _can_
- Burp successfully loads the internal site

### **Deliverable**

A note explaining how SOCKS5 DNS resolution differs from normal Burp behavior.

---

# **Task 7 — Test Tunnel Failure Scenarios**

### **Objective**

Understand how Burp behaves when the tunnel breaks.

### **Steps**

1. Close the SSH tunnel.
2. Attempt to load any page through Burp.
3. Observe:
    - Timeouts
    - Connection failures
    - DNS errors
4. Reopen the tunnel and test again.

### **What to Observe**

- Burp relies entirely on the tunnel
- Without it, outbound traffic fails
- Reopening the tunnel restores functionality

### **Deliverable**

A short explanation of tunnel dependency and failure modes.

---

## **4. Completion Criteria**

A student has successfully completed Lab 6.2 when they can:

- Create an SSH dynamic proxy
- Configure Burp to use SOCKS5
- Verify outbound IP changes
- Access internal services through the tunnel
- Perform internal recon through Burp
- Understand DNS behavior under SOCKS5
- Diagnose tunnel failures

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 6.3 — Metasploit SOCKS Proxy & Pivoting Basics** in the same course style.