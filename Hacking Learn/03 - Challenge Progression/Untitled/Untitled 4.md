Oh hell yes, Liam — **this is the moment we’ve been building toward.**  
The Knowledge Graph is the _capstone artifact_ that ties your entire vault together: skills, tools, machines, concepts, workflows, and challenge paths — all in one unified, visual, navigable map.

And I’m going to give you **two versions**:

1. **ASCII Knowledge Graph** — Obsidian‑friendly, readable, and perfect for embedding in a note
2. **Canvas Knowledge Graph** — node‑based structure you can paste directly into an Obsidian Canvas

This is not a toy diagram.  
This is a **full conceptual map of offensive security**, tailored to _your_ vault structure, _your_ challenge paths, and _your_ 10‑week curriculum.

Let’s build it.

---

# 0400.000 Knowledge Graph (ASCII Master Map)

### “Everything connects to everything.”

```
                                           ┌──────────────────────────────┐
                                           │        Offensive Security     │
                                           │         Knowledge Graph       │
                                           └─────────────────────┬────────┘
                                                                 │
                                                                 ▼
                     ┌──────────────────────────────────────────────────────────┐
                     │                        Core Pillars                      │
                     └──────────────┬──────────────────────────┬────────────────┘
                                    │                          │
                                    │                          │
                                    ▼                          ▼
                 ┌──────────────────────────┐     ┌──────────────────────────┐
                 │   Web Exploitation       │     │   System Exploitation    │
                 └──────────────┬───────────┘     └──────────────┬───────────┘
                                │                                 │
                                ▼                                 ▼
         ┌──────────────────────────────┐         ┌──────────────────────────────┐
         │ Legacy Web Vulns             │         │ Linux Exploitation           │
         │ (DVWA, Mutillidae)           │         │ (MS2)                        │
         └──────────────┬───────────────┘         └──────────────┬───────────────┘
                        │                                         │
                        ▼                                         ▼
         ┌──────────────────────────────┐         ┌──────────────────────────────┐
         │ Enterprise Web Vulns         │         │ Windows Exploitation         │
         │ (WebGoat)                    │         │ (MS3)                        │
         └──────────────┬───────────────┘         └──────────────┬───────────────┘
                        │                                         │
                        ▼                                         ▼
         ┌──────────────────────────────┐         ┌──────────────────────────────┐
         │ Modern Web Attacks           │         │ OSCP‑Style Machines          │
         │ (Juice Shop)                 │         │ (Unknown‑box workflow)       │
         └──────────────┬───────────────┘         └──────────────┬───────────────┘
                        │                                         │
                        ▼                                         ▼
         ┌──────────────────────────────┐         ┌──────────────────────────────┐
         │ API / JWT / NoSQL / CORS     │         │ PrivEsc Playbooks            │
         │ Payload Libraries            │         │ (Linux + Windows)            │
         └──────────────┬───────────────┘         └──────────────┬───────────────┘
                        │                                         │
                        ▼                                         ▼
                     ┌──────────────────────────────────────────────────────────┐
                     │                Active Directory (AD Path)                │
                     │ Kerberoast • AS‑REP • Lateral Movement • DCSync • DA     │
                     └──────────────┬───────────────────────────────────────────┘
                                    │
                                    ▼
                     ┌──────────────────────────────────────────────────────────┐
                     │                     Attack Chains                        │
                     │   (DVWA → Mutillidae → WebGoat → Juice Shop → MS2 →     │
                     │    MS3 → VulnHub → AD Domain Compromise)                │
                     └──────────────┬───────────────────────────────────────────┘
                                    │
                                    ▼
                     ┌──────────────────────────────────────────────────────────┐
                     │                     Workflows & Tools                    │
                     │ Recon • Enum • Exploit • PrivEsc • Post‑Exploitation     │
                     │ nmap • gobuster • sqlmap • CME • Impacket • jwt‑tool     │
                     └──────────────┬───────────────────────────────────────────┘
                                    │
                                    ▼
                     ┌──────────────────────────────────────────────────────────┐
                     │                  Documentation & Templates               │
                     │ Recon Template • Vuln Matrix • Machine Analysis          │
                     │ Attack Chains • Payload Libraries • Daily Logs           │
                     └──────────────────────────────────────────────────────────┘
```

This ASCII map shows the **hierarchy** and **flow** of knowledge:

- Web → System → OSCP → AD
- Tools → Workflows → Attack Chains
- Everything feeds into the AD capstone

---

# 0400.100 Knowledge Graph (Canvas Version)

### Paste directly into an Obsidian Canvas

Below is the **node list**.  
Each bullet = one Canvas node.  
Indented bullets = child nodes.  
Use arrows to connect them exactly as shown.

---

## **Node: Offensive Security Knowledge Graph**

- Root node for the entire map

### **Child: Web Exploitation**

- DVWA (Legacy)
- Mutillidae (Intermediate)
- WebGoat (Enterprise)
- Juice Shop (Modern)
- Payload Libraries (SQLi, XSS, JWT, NoSQLi)
- API Exploitation

### **Child: System Exploitation**

- MS2 (Linux)
- MS3 (Windows)
- Post‑Exploitation
- PrivEsc (Linux)
- PrivEsc (Windows)

### **Child: OSCP Workflow**

- Recon → Enum → Exploit → PrivEsc
- Unknown‑Box Methodology
- Timeboxing
- Reporting

### **Child: Active Directory**

- LDAP / SMB / Kerberos Recon
- Kerberoasting
- AS‑REP Roasting
- Lateral Movement (WinRM / SMBexec / WMI)
- PrivEsc → Domain Admin
- DCSync
- Domain Dominance

### **Child: Tools**

- nmap
- gobuster
- sqlmap
- netcat
- crackmapexec
- impacket‑suite
- jwt‑tool

### **Child: Workflows**

- Recon Workflow
- Enumeration Workflow
- Exploitation Workflow
- PrivEsc Workflow
- Reporting Workflow

### **Child: Documentation**

- Recon Template
- Vulnerability Matrix
- Machine Analysis Template
- Attack Chains
- Daily Lab Logs

---

# 0400.200 Knowledge Graph (Conceptual Summary)

This graph encodes **four truths** about offensive security:

1. **Web exploitation teaches input → output → vulnerability thinking**
2. **System exploitation teaches foothold → privesc → persistence**
3. **OSCP workflow teaches discipline and methodology**
4. **Active Directory ties everything together into enterprise‑grade attack chains**

Your vault is now structured like a **real red team knowledge base**.

---

# ✔ Knowledge Graph Complete.

If you want, I can also generate:

### • A **Skill Tree** (video‑game style progression)

### • A **“Where Do I Go Next?” Navigator**

### • A **Printable Poster Version**

### • A **Machine‑Only Knowledge Graph**

### • A **Tool‑Only Knowledge Graph**

Just tell me which one you want.