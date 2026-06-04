### _Week 09 — Internal Network Exploitation & Lateral Movement_

**Estimated Time:** 120–180 minutes  
**Difficulty:** Advanced  
**Tools:** Metasploit Framework, Meterpreter, proxychains, nmap, Burp Suite (optional), internal lab network

---

## **1. Purpose of This Lab**

This lab teaches students how to use **Meterpreter as a pivot point** to access internal networks that are not directly reachable from the attacker machine.

By the end of this lab, students will be able to:

- Identify internal networks from a compromised host
- Add Metasploit routes for pivoting
- Use Meterpreter’s built‑in pivoting capabilities
- Use proxychains to run external tools through the pivot
- Perform internal recon (nmap, curl, SMB, HTTP)
- Access internal web apps through Burp Suite
- Build a structured lateral movement workflow

This lab is the foundation for Week 9’s exploitation and Week 10’s AD labs.

---

## **2. Pre‑Lab Requirements**

Students must have:

- Completed Week 5 (Metasploit Recon)
- Completed Week 6 (SOCKS proxying, SSH tunneling, MITM)
- Completed Week 8 (Metasploitable exploitation)
- A working Meterpreter session on a compromised host
- An internal network reachable only from that host

Example topology:

```
Kali → (Meterpreter) → Metasploitable → Internal Network (10.10.10.x)
```

---

## **3. Lab Tasks**

---

# **Task 1 — Identify Internal Network Interfaces**

### **Objective**

Determine what networks the compromised host can see.

### **Steps**

From Meterpreter:

```
ifconfig
```

Or:

```
ipconfig
```

### **What to Observe**

- Internal IP ranges (10.x.x.x, 172.16.x.x, 192.168.x.x)
- Multiple NICs
- Virtual interfaces

### **Deliverable**

A list of internal subnets discovered.

---

# **Task 2 — Scan Internal Networks Using Meterpreter Modules**

### **Objective**

Perform internal recon directly from the compromised host.

### **Steps**

Use ARP scan:

```
run arp_scanner -r 10.10.10.0/24
```

Use TCP port scan:

```
run portscan -r 10.10.10.0/24 -p 1-1000
```

### **What to Observe**

- Live hosts
- Open ports
- Internal services

### **Deliverable**

A table of discovered hosts and open ports.

---

# **Task 3 — Add a Route Through the Compromised Host**

### **Objective**

Tell Metasploit to route traffic through the Meterpreter session.

### **Steps**

Identify session ID:

```
sessions
```

Add route:

```
route add 10.10.10.0/24 1
```

(Replace `1` with your session ID.)

### **What to Observe**

- Metasploit now knows how to reach internal hosts
- Auxiliary modules will use the pivot automatically

### **Deliverable**

A screenshot or note showing:

```
route print
```

---

# **Task 4 — Use Metasploit Auxiliary Modules Through the Pivot**

### **Objective**

Perform recon on internal hosts using Metasploit.

### **Steps**

Example:

```
use auxiliary/scanner/smb/smb_version
set RHOSTS 10.10.10.5
run
```

Try:

- SMB enumeration
- FTP version
- SSH version
- HTTP fingerprinting

### **What to Observe**

- Internal services that Kali cannot reach directly
- Version info for future exploitation

### **Deliverable**

A list of internal services discovered.

---

# **Task 5 — Enable Metasploit’s SOCKS Proxy for External Tools**

### **Objective**

Use proxychains to run external tools (nmap, curl, smbclient) through the pivot.

### **Steps**

Start SOCKS proxy:

```
use auxiliary/server/socks_proxy
set VERSION 4a
set SRVPORT 1080
run
```

Configure proxychains:

```
nano /etc/proxychains.conf
socks4 127.0.0.1 1080
```

### **Deliverable**

A note confirming SOCKS proxy is active.

---

# **Task 6 — Run nmap Through the Pivot Using proxychains**

### **Objective**

Perform internal scanning using nmap.

### **Steps**

```
proxychains nmap -sT -Pn 10.10.10.5
```

### **What to Observe**

- nmap traffic routed through Meterpreter
- Internal ports discovered
- Differences from Metasploit scans

### **Deliverable**

A screenshot or note showing nmap results.

---

# **Task 7 — Access Internal Web Apps Through Burp Suite**

### **Objective**

Use Burp to browse internal services through the pivot.

### **Steps**

Configure Burp → User Options → SOCKS Proxy:

- Host: `127.0.0.1`
- Port: `1080`
- Version: SOCKS4a

Visit:

```
http://10.10.10.5:8080
```

### **What to Observe**

- Internal web apps load
- Burp logs requests
- You can map internal attack surfaces

### **Deliverable**

A list of internal endpoints discovered via Burp.

---

# **Task 8 — Validate Pivot Failure Modes**

### **Objective**

Understand how pivoting breaks.

### **Steps**

1. Kill the Meterpreter session:
    
    ```
    sessions -K
    ```
    
2. Try:
    
    ```
    proxychains nmap 10.10.10.5
    ```
    
3. Observe failure.
4. Re‑establish session and route.

### **What to Observe**

- Pivoting depends entirely on the Meterpreter session
- Routes disappear when session dies
- SOCKS proxy remains but is useless

### **Deliverable**

A short explanation of pivot dependency and recovery.

---

# **Task 9 — Build a Complete Pivoting Attack Surface Map**

### **Objective**

Document everything discovered through the pivot.

### **Steps**

Record:

- Internal hosts
- Open ports
- Services
- Web apps
- SMB shares
- SSH/FTP/HTTP versions
- Potential exploitation paths

### **Deliverable**

A complete pivoting attack surface map in your Obsidian vault.

---

## **4. Completion Criteria**

A student has successfully completed Lab 9.1 when they can:

- Identify internal networks from a compromised host
- Add routes and pivot through Meterpreter
- Use proxychains for internal scanning
- Access internal web apps through Burp
- Diagnose pivot failures
- Document a full internal recon workflow

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 9.2 — Internal Service Exploitation via Pivot** in the same course style.