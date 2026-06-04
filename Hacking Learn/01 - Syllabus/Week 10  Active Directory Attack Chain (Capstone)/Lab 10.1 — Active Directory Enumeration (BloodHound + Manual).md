
# **Lab 10.1 — Active Directory Enumeration (BloodHound + Manual)**

### _Week 10 — Active Directory (AD) Exploitation & Domain Dominance_

**Estimated Time:** 150–210 minutes  
**Difficulty:** Advanced  
**Tools:** BloodHound, SharpHound, neo4j, PowerView, ADExplorer, Meterpreter, proxychains, internal Windows domain environment

---

## **1. Purpose of This Lab**

This lab introduces **Active Directory enumeration**, the foundation of all AD exploitation.  
You will learn to:

- Identify the AD structure (domains, OUs, users, groups, computers)
- Enumerate AD using **BloodHound + SharpHound**
- Enumerate AD manually using **PowerView**
- Identify privilege escalation paths
- Identify misconfigurations (ACLs, GPOs, delegation, unconstrained delegation)
- Build a complete AD attack surface map

This is the first lab of Week 10 and sets the stage for Kerberoasting, AS‑REP roasting, delegation abuse, and full domain compromise.

---

## **2. Pre‑Lab Requirements**

You must have:

- Completed Week 9 (pivoting, lateral movement, persistence)
- SYSTEM or high‑priv access on at least one internal Windows host
- Access to a domain‑joined machine (user or admin)
- SOCKS proxy + proxychains configured
- neo4j + BloodHound installed on Kali

Example topology:

```
Kali → Pivot → Domain-Joined Host → Domain Controller
```

---

## **3. Lab Tasks**

---

# **Task 1 — Confirm Domain Membership & Identify the Domain Controller**

### **Objective**

Determine if the compromised host is part of a domain and identify the DC.

### **Steps**

From a shell on the compromised host:

```
systeminfo | findstr /i "Domain"
```

Identify the DC:

```
nltest /dclist:<domain>
```

Or:

```
nltest /dsgetdc:<domain>
```

### **What to Observe**

- Domain name
- Domain controller hostname
- Forest name
- Site name

### **Deliverable**

A note confirming domain membership and DC identification.

---

# **Task 2 — Enumerate Domain Users (Manual)**

### **Objective**

Use built‑in Windows tools to enumerate users.

### **Steps**

```
net user /domain
```

List details for a specific user:

```
net user <username> /domain
```

### **What to Observe**

- Password last set
- Password expiration
- Group membership
- Privileged accounts

### **Deliverable**

A list of domain users and interesting attributes.

---

# **Task 3 — Enumerate Domain Groups (Manual)**

### **Objective**

Identify high‑value groups.

### **Steps**

```
net group /domain
```

Check group membership:

```
net group "Domain Admins" /domain
net group "Enterprise Admins" /domain
```

### **What to Observe**

- Domain Admins
- Enterprise Admins
- Backup Operators
- Account Operators
- Server Operators

### **Deliverable**

A list of high‑value groups and members.

---

# **Task 4 — Enumerate Domain Computers (Manual)**

### **Objective**

Identify domain‑joined machines.

### **Steps**

```
net group "Domain Computers" /domain
```

Or:

```
net view /domain
```

### **What to Observe**

- Workstations
- Servers
- Domain controllers
- Naming conventions

### **Deliverable**

A list of domain computers.

---

# **Task 5 — Use PowerView for Deep AD Enumeration**

### **Objective**

Use PowerView to enumerate AD objects and permissions.

### **Steps**

Upload PowerView:

```
upload PowerView.ps1
```

Load it:

```
powershell -ep bypass
. .\PowerView.ps1
```

### **Enumerate Users**

```
Get-DomainUser
```

### **Enumerate Groups**

```
Get-DomainGroup
```

### **Enumerate Computers**

```
Get-DomainComputer
```

### **Enumerate ACLs**

```
Get-ObjectAcl -Identity <user/group/computer>
```

### **What to Observe**

- Interesting ACLs
- Write permissions
- GenericAll / GenericWrite
- GPO links
- Delegation

### **Deliverable**

A list of PowerView findings.

---

# **Task 6 — Deploy SharpHound for BloodHound Collection**

### **Objective**

Collect AD data for graph‑based analysis.

### **Steps**

Upload SharpHound:

```
upload SharpHound.exe
```

Run it:

```
SharpHound.exe -c All
```

Or PowerShell version:

```
Invoke-BloodHound -CollectionMethod All
```

### **What to Observe**

- Output ZIP file
- Collection of ACLs, sessions, groups, GPOs, trusts

### **Deliverable**

The SharpHound ZIP file.

---

# **Task 7 — Import SharpHound Data into BloodHound**

### **Objective**

Visualize AD relationships and attack paths.

### **Steps**

Start neo4j:

```
neo4j start
```

Start BloodHound:

```
bloodhound
```

Upload the SharpHound ZIP.

### **What to Observe**

- Nodes: Users, Groups, Computers, OUs
- Edges: MemberOf, AdminTo, HasSession, GenericWrite, Owns

### **Deliverable**

A screenshot of the BloodHound graph.

---

# **Task 8 — Identify Attack Paths in BloodHound**

### **Objective**

Find privilege escalation paths.

### **Steps**

Use built‑in queries:

- **Shortest Path to Domain Admins**
- **Users with DCSync Rights**
- **Kerberoastable Users**
- **AS‑REP Roastable Users**
- **Unconstrained Delegation Computers**
- **Users with GenericWrite on GPOs**

### **What to Observe**

- Misconfigurations
- Privilege escalation opportunities
- Lateral movement paths

### **Deliverable**

A list of identified attack paths.

---

# **Task 9 — Enumerate Active Sessions**

### **Objective**

Identify where privileged users are logged in.

### **Steps**

PowerView:

```
Get-NetSession -ComputerName <hostname>
```

BloodHound:

- **Find Principals with Active Sessions**

### **What to Observe**

- Admin sessions on workstations
- Opportunities for token theft
- Opportunities for lateral movement

### **Deliverable**

A list of active sessions.

---

# **Task 10 — Build an AD Attack Surface Map**

### **Objective**

Document everything discovered.

### **Include:**

- Domain structure
- Users, groups, computers
- ACL misconfigurations
- Delegation issues
- Kerberoastable accounts
- AS‑REP roastable accounts
- GPO misconfigurations
- Attack paths
- High‑value targets

### **Deliverable**

A complete AD attack surface map in your Obsidian vault.

---

## **4. Completion Criteria**

You have completed Lab 10.1 when you can:

- Enumerate AD manually and with PowerView
- Collect and analyze AD data with BloodHound
- Identify misconfigurations and attack paths
- Map the entire AD environment
- Prepare for AD exploitation (Kerberoasting, DCSync, delegation abuse)

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 10.2 — Kerberoasting & AS‑REP Roasting** in the same course style.
