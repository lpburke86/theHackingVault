### _Week 10 — Offensive + Defensive Enterprise Simulation_

**Estimated Time:** 6–12 hours (multi‑stage)  
**Difficulty:** Expert / Red Team + Blue Team Hybrid  
**Tools:**  
Everything from Weeks 9–10, including:  
BloodHound, SharpHound, PowerView, Rubeus, Mimikatz, Impacket, Certipy, Certify, ADExplorer, Sysmon, Event Logs, GPO tools, Meterpreter, proxychains, nmap, Burp Suite, winPEAS, linPEAS, ACL tools, CA management tools.

**Environment:**  
Multi‑forest AD environment with:

- Forest A (`corp.local`)
- Forest B (`dev.local`)
- Two‑way forest trust
- AD CS deployed in at least one forest
- Multiple Windows + Linux internal hosts
- At least one externally reachable foothold

---

## **1. Purpose of This Capstone**

This is the **full enterprise kill‑chain simulation**:

### **Red Team Objectives**

- Gain initial access
- Pivot internally
- Enumerate AD
- Escalate privileges
- Exploit AD CS
- Exploit delegation
- Exploit forest trusts
- Achieve domain dominance in both forests
- Establish multi‑layer persistence

### **Blue Team Objectives**

- Detect the attack
- Identify persistence
- Remediate AD CS, delegation, ACLs, trusts
- Reset KRBTGT
- Harden the environment
- Validate remediation

This is the most comprehensive lab in the entire course.

---

## **2. Capstone Structure**

The capstone is divided into **four phases**, each with tasks.

---

# **PHASE 1 — INITIAL ACCESS & INTERNAL PIVOTING**

---

# **Task 1 — Gain Initial Foothold**

Choose one:

- Web app exploit
- Phishing payload
- Exposed RDP
- Vulnerable service

Deliverable:  
**Low‑priv shell on a domain‑joined host.**

---

# **Task 2 — Establish Pivoting**

- Set up Meterpreter pivot
- Add routes
- Enable SOCKS proxy
- Confirm proxychains works

Deliverable:  
**Internal subnet reachable.**

---

# **Task 3 — Internal Recon**

Use:

- proxychains nmap
- SMB enumeration
- HTTP enumeration
- LDAP enumeration

Deliverable:  
**Internal network map.**

---

---

# **PHASE 2 — AD ENUMERATION & DOMAIN COMPROMISE**

---

# **Task 4 — AD Enumeration**

Use:

- PowerView
- BloodHound
- SharpHound

Deliverable:  
**AD attack surface map.**

---

# **Task 5 — Credential Attacks**

Perform:

- Kerberoasting
- AS‑REP Roasting
- Password spraying
- Hash extraction

Deliverable:  
**Cracked credentials + lateral movement.**

---

# **Task 6 — Privilege Escalation**

Choose one:

- Local privilege escalation
- Token impersonation
- Service misconfig
- Kernel exploit

Deliverable:  
**SYSTEM on at least one host.**

---

# **Task 7 — Domain Admin Compromise**

Choose one:

- DCSync
- Delegation abuse
- AD CS abuse
- ACL abuse
- Kerberos abuse

Deliverable:  
**Domain Admin in Forest A.**

---

---

# **PHASE 3 — MULTI‑FOREST EXPLOITATION & ENTERPRISE DOMINANCE**

---

# **Task 8 — Enumerate Forest Trusts**

Use:

- nltest
- PowerView
- BloodHound

Deliverable:  
**Forest trust map.**

---

# **Task 9 — Cross‑Forest Exploitation**

Choose one:

- SIDHistory abuse
- Cross‑forest Kerberos
- Cross‑forest delegation
- Cross‑forest AD CS

Deliverable:  
**Domain Admin in Forest B.**

---

# **Task 10 — Enterprise‑Wide Persistence**

Implement at least **three**:

- Golden Tickets
- Silver Tickets
- RBCD persistence
- AD CS certificate persistence
- ACL backdoors
- Rogue machine accounts
- GPO persistence

Deliverable:  
**Multi‑forest persistence map.**

---

---

# **PHASE 4 — BLUE TEAM REMEDIATION & HARDENING**

---

# **Task 11 — Identify All Persistence**

Use:

- BloodHound Blue Team queries
- PowerView
- Event logs
- Sysmon

Deliverable:  
**List of all persistence mechanisms.**

---

# **Task 12 — Remediate Kerberos Abuse**

- Reset KRBTGT twice
- Reset machine accounts
- Remove SIDHistory

Deliverable:  
**Kerberos remediation report.**

---

# **Task 13 — Remediate AD CS**

- Fix templates
- Fix CA ACLs
- Revoke certificates
- Disable vulnerable templates

Deliverable:  
**AD CS remediation report.**

---

# **Task 14 — Remediate Delegation**

- Remove unconstrained delegation
- Fix constrained delegation
- Remove RBCD entries

Deliverable:  
**Delegation remediation report.**

---

# **Task 15 — Harden Forest Trusts**

- Enable SID filtering
- Enable selective authentication

Deliverable:  
**Forest trust hardening report.**

---

# **Task 16 — Validate Remediation**

Attempt:

- Golden Ticket
- Silver Ticket
- RBCD
- AD CS abuse
- Cross‑forest escalation

Deliverable:  
**Validation checklist.**

---

# **Task 17 — Build Final Capstone Report**

Include:

- Attack chain
- Persistence chain
- Remediation steps
- Hardening plan
- Lessons learned

Deliverable:  
**Full enterprise attack + defense report.**

---

## **3. Completion Criteria**

You have completed the Week 10 Capstone when you can:

- Execute a full enterprise kill chain
- Compromise multiple forests
- Establish multi‑layer persistence
- Detect and remove persistence
- Harden AD, AD CS, delegation, ACLs, and trusts
- Produce a full enterprise‑grade report

This is the final lab of Week 10.

---

## **4. Next Steps**

If you want to continue, you can say:

### **“Start Week 11.”**

or

### **“Give me the next module.”**

Week 11 begins the **Red Team Automation & Infrastructure** arc (C2 frameworks, OPSEC, automation, scripting, infrastructure‑as‑attack).