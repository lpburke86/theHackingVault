# **Lab 10.8 — Enterprise‑Wide Cleanup, Remediation & Hardening (Blue Team Perspective)**

### _Week 10 — Post‑Exploitation Remediation, AD Hardening & Multi‑Forest Defense_

**Estimated Time:** 180–240 minutes  
**Difficulty:** Expert  
**Tools:** PowerShell, AD DS tools, GPO management, AD CS management console, BloodHound (Blue Team queries), Sysmon, Windows Event Logs, Defender for Identity (optional), Group Policy, CA management tools  
**Environment:** Multi‑forest AD environment (e.g., `corp.local` + `dev.local`) with forest trust

---

## **1. Purpose of This Lab**

This lab flips the perspective:  
You’ve spent Weeks 9–10 learning how to compromise an enterprise.  
Now you learn how to **clean up, remediate, and harden** an Active Directory environment after a breach.

You will learn to:

- Identify persistence mechanisms
- Remove Golden/Silver Ticket access
- Reset KRBTGT safely
- Remediate AD CS vulnerabilities (ESC1–ESC8)
- Remediate delegation abuse
- Remediate SIDHistory abuse
- Harden forest trusts
- Harden GPOs, ACLs, and privileged groups
- Build a full remediation plan

This is the **Blue Team counterpart** to Labs 10.1–10.7.

---

## **2. Pre‑Lab Requirements**

You must have:

- Completed **Labs 10.1–10.7**
- Full understanding of AD exploitation paths
- Domain Admin in both forests
- Access to CA servers (if AD CS is present)
- Access to GPO management
- Access to DCs and event logs

---

## **3. Lab Tasks**

---

# **SECTION A — ENTERPRISE‑WIDE INCIDENT SCOPING**

---

# **Task 1 — Identify All Persistence Mechanisms (Red → Blue Mapping)**

### **Objective**

Locate persistence mechanisms attackers may have installed.

### **Check for:**

- Golden Tickets
- Silver Tickets
- RBCD persistence
- Rogue machine accounts
- AD CS certificates
- ACL backdoors
- GPO backdoors
- Delegation abuse
- SIDHistory abuse
- Local admin backdoors
- Startup scripts
- Scheduled tasks
- WMI subscriptions

### **Deliverable**

A complete list of persistence mechanisms discovered.

---

# **Task 2 — Use BloodHound (Blue Team Queries)**

### **Objective**

Identify misconfigurations and persistence paths.

### **Steps**

Run Blue Team queries:

- **“Users with DCSync Rights”**
- **“Unconstrained Delegation Computers”**
- **“Kerberoastable Accounts”**
- **“AS‑REP Roastable Accounts”**
- **“RBCD Paths”**
- **“Vulnerable Certificate Templates”**
- **“Accounts with SIDHistory”**

### **Deliverable**

A BloodHound remediation report.

---

---

# **SECTION B — REMEDIATING KERBEROS ABUSE**

---

# **Task 3 — Reset KRBTGT (Twice)**

### **Objective**

Invalidate all Golden Tickets.

### **Steps**

1. Reset KRBTGT password.
2. Wait for replication.
3. Reset again.

### **Why twice?**

Golden Tickets remain valid until both old and new keys are rotated.

### **Deliverable**

KRBTGT reset confirmation.

---

# **Task 4 — Reset Machine Account Passwords**

### **Objective**

Invalidate Silver Tickets.

### **Steps**

```
Reset-ComputerMachinePassword
```

### **Deliverable**

A list of machine accounts reset.

---

# **Task 5 — Remove SIDHistory Entries**

### **Objective**

Eliminate cross‑forest privilege escalation.

### **Steps**

```
Set-ADUser <user> -Remove @{SIDHistory="<SID>"}
```

### **Deliverable**

A list of accounts with SIDHistory removed.

---

---

# **SECTION C — REMEDIATING AD CS (ESC1–ESC8)**

---

# **Task 6 — Identify Vulnerable Templates**

### **Objective**

Find templates that allow privilege escalation.

### **Steps**

Use Certipy:

```
certipy find -vulnerable
```

### **Deliverable**

A list of vulnerable templates.

---

# **Task 7 — Fix Template Permissions**

### **Objective**

Remove dangerous enrollment rights.

### **Steps**

- Remove `ENROLLEE_SUPPLIES_SUBJECT`
- Remove `Client Authentication` EKU
- Remove enrollment rights from low‑priv users

### **Deliverable**

A list of remediated templates.

---

# **Task 8 — Fix CA Permissions**

### **Objective**

Prevent template or CA abuse.

### **Steps**

- Remove dangerous ACLs
- Disable vulnerable templates
- Restrict enrollment agents

### **Deliverable**

CA hardening report.

---

# **Task 9 — Revoke Compromised Certificates**

### **Objective**

Invalidate attacker‑issued certificates.

### **Steps**

- Revoke certificates
- Publish CRLs
- Force certificate renewal

### **Deliverable**

A list of revoked certificates.

---

---

# **SECTION D — REMEDIATING DELEGATION ABUSE**

---

# **Task 10 — Remove Unconstrained Delegation**

### **Objective**

Eliminate the most dangerous delegation type.

### **Steps**

```
Set-ADComputer <computer> -TrustedForDelegation $false
```

### **Deliverable**

A list of hosts with unconstrained delegation removed.

---

# **Task 11 — Fix Constrained Delegation**

### **Objective**

Ensure only legitimate services can delegate.

### **Steps**

- Remove unnecessary SPNs
- Remove unnecessary `msDS-AllowedToDelegateTo` entries

### **Deliverable**

A constrained delegation remediation list.

---

# **Task 12 — Fix RBCD Misconfigurations**

### **Objective**

Remove rogue machine accounts and RBCD entries.

### **Steps**

```
Set-ADComputer <target> -Remove @{ 'msDS-AllowedToActOnBehalfOfOtherIdentity' = $value }
```

### **Deliverable**

A list of RBCD entries removed.

---

---

# **SECTION E — REMEDIATING ACL & GPO BACKDOORS**

---

# **Task 13 — Identify ACL Backdoors**

### **Objective**

Find unauthorized ACL modifications.

### **Steps**

PowerView:

```
Get-ObjectAcl -ResolveGUIDs
```

### **Deliverable**

A list of ACLs requiring remediation.

---

# **Task 14 — Remove Unauthorized ACL Entries**

### **Objective**

Restore secure ACLs.

### **Steps**

```
Set-DomainObjectAcl -TargetIdentity <object> -RestoreDefault
```

### **Deliverable**

ACL remediation report.

---

# **Task 15 — Audit & Fix GPO Permissions**

### **Objective**

Remove GPO‑based persistence.

### **Steps**

- Remove unauthorized GPO editors
- Remove malicious scripts
- Remove scheduled tasks

### **Deliverable**

A list of GPOs remediated.

---

---

# **SECTION F — FOREST TRUST HARDENING**

---

# **Task 16 — Enable SID Filtering**

### **Objective**

Prevent SIDHistory abuse across forests.

### **Steps**

```
netdom trust <trusting-domain> /domain:<trusted-domain> /enablesidhistory:no /quarantine:yes
```

### **Deliverable**

SID filtering enabled.

---

# **Task 17 — Enable Selective Authentication**

### **Objective**

Prevent cross‑forest lateral movement.

### **Steps**

```
netdom trust <trusting-domain> /domain:<trusted-domain> /selectiveauth:yes
```

### **Deliverable**

Selective authentication enabled.

---

---

# **SECTION G — VALIDATION & HARDENING**

---

# **Task 18 — Validate All Remediation Steps**

### **Objective**

Ensure all persistence is removed.

### **Steps**

- Attempt Golden Ticket injection
- Attempt Silver Ticket injection
- Attempt RBCD abuse
- Attempt AD CS abuse
- Attempt cross‑forest escalation

### **Deliverable**

A validation checklist.

---

# **Task 19 — Build an Enterprise‑Wide Hardening Plan**

### **Objective**

Document all remediation and hardening steps.

### **Include:**

- Kerberos hardening
- AD CS hardening
- Delegation hardening
- ACL hardening
- GPO hardening
- Trust hardening
- Logging & monitoring improvements
- Long‑term security recommendations

### **Deliverable**

A complete enterprise hardening plan in your Obsidian vault.

---

## **4. Completion Criteria**

You have completed Lab 10.8 when you can:

- Identify and remove enterprise‑wide persistence
- Remediate AD CS, delegation, ACL, and Kerberos abuse
- Harden forest trusts
- Validate remediation
- Build a complete enterprise hardening plan

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 10.9 — Week 10 Capstone: Full‑Scope AD Attack & Defense Simulation** in the same course style.