# 🧭 Table of Contents

- [[#Day 1 — Kali Foundations, Networking, and Workflow Setup (Full, Detailed Chapter) 🧭💻]]
- [[#Day 1 — Short intro and main goals 🧭💻]]
- [[#1 — The mental model you should carry through Day 1 🧠]]
- [[#2 — Before we begin: environment checklist (do this first) ✅]]
- [[#3 — Kali basics: terminal, file system, and safety (slow, explicit, with why) 🖥️🔒]]
- [[#4 — Networking verification lab (Lab 1) — deep walkthrough (45–60 minutes) 🌐🔍]]
- [[#5 — Host discovery & recon (Lab 2) — deep walkthrough (60–90 minutes) 🔎🧭]]
- [[#6 — First full port scan (Lab 3) — deep walkthrough (60–90 minutes) ⚔️]]
- [[#7 — Quick tool primers 🧰]]
- [[#8 — Documentation: how to record recon (Assignment 1 prep) 📝]]
- [[#9 — Mini‑Project: Build your offensive workflow (1.5 hours) 🧩]]
- [[#10 — Assignments (detailed expectations and grading hints) 🧾]]
- [[#11 — Instructor notes and pedagogy (how to use this day effectively) 🧑‍🏫]]
- [[#12 — Reflection prompts (short, honest, useful) 🤔]]
- [[#13 — Deliverables checklist (what to hand in) 📦]]
- [[#14 — Extra troubleshooting appendix (common errors and exact fixes) 🛠️]]
- [[#15 — Short practice exercises (do these now, with expected outputs) 🏁]]

---

# Day 1 — Kali Foundations, Networking, and Workflow Setup (Full, Detailed Chapter) 🧭💻

**Short intro — what Day 1 is for**  
Day 1 is the day you stop treating Kali Linux as a mysterious “hacker OS” and start treating it like a toolbench. You will install and configure Kali (or verify an image), learn the terminal basics you’ll use every day, validate and fix your networking so your multi‑VM labs behave predictably, and run your first reconnaissance scans. The goal is not to memorize flags; it’s to build a repeatable, reliable workflow you can use for every lab in the course.

**Main goals for Day 1**

- Get comfortable in the Kali terminal and file system. 🧰
- Ensure networking is correct for Host‑Only + NAT lab topologies. 🌐
- Learn and practice core tools (`nmap`, `curl`, `netcat`, `gobuster`, `ssh`) with exact commands and expected outputs. 🔎
- Capture and interpret the network evidence you produce (logs, screenshots, pcap). 📸
- Create a documented, repeatable workflow in your vault (Obsidian). 📁

---

## Day 1 — Short intro and main goals 🧭💻

Day 1 establishes the practical foundation for everything you’ll do later. You will not only run tools; you will learn why each tool exists, what problem it solves, and how its output becomes evidence in your notes. The emphasis is on reproducibility: every command you run should produce an artifact you can store, reference, and explain.

---

## 1 — The mental model you should carry through Day 1 🧠

Before you type anything, hold this simple mental picture: Kali is your workshop. The terminal is your primary tool. Every command you run is an experiment that produces artifacts: text output, files, screenshots, and packet captures. Your job today is to make those artifacts reliable and reproducible.

When you scan a host, you are asking a question: “What services does this machine offer?” When you document, you are answering: “Here’s what I found, how I found it, and what I would try next.” The quality of your future attacks depends on how well you can ask, record, and reason about those answers.

[[#🧭 Table of Contents]]

---

## 2 — Before we begin: environment checklist (do this first) ✅

Make sure the following are true before you start any commands. These are not optional; they prevent hours of wasted debugging later.

1. Your GNS3 VM (or virtualization platform) is running and connected to the GUI. You can open the canvas and see your appliances.
2. Kali VM boots and you can log in. If you use an image, confirm username/password. If you installed Kali, confirm SSH is enabled if you plan to remote into it.
3. The target VM (Metasploitable 2 or another lab VM) is imported and powered off until you wire the topology.
4. You have a place in your vault (Obsidian) for Week 1 notes and a `scans/` folder for outputs. Create `week1/` with subfolders `scans/`, `screenshots/`, `pcaps/`, and `notes/`.

If any of these are missing, stop and fix them now. The rest of Day 1 assumes these basics are in place.

[[#🧭 Table of Contents]]

---

## 3 — Kali basics: terminal, file system, and safety (slow, explicit, with why) 🖥️🔒

When you open Kali, you will usually work in a terminal. The terminal is where you type commands and see immediate results. Treat it like a conversation with the machine.

### Opening a terminal and confirming identity

Open a terminal window and run:

```bash
whoami
hostname
uname -a
```

**What you should see and why it matters**

`whoami` prints the current user. If you’re `root` or a sudo user, you can run privileged tools. `hostname` shows the machine name — useful when you have multiple terminals open. `uname -a` prints kernel and architecture info; it helps you confirm you’re on the expected image (e.g., `x86_64`).

**Why this matters**  
Privilege level and architecture affect which tools you can run and how you install packages. If you’re not root, prefix privileged commands with `sudo`. If you are root, be careful — a single mistyped command can change system files.

### File system basics and where to store artifacts

Create a directory for Week 1 artifacts:

```bash
mkdir -p ~/week1/{scans,screenshots,pcaps,notes}
cd ~/week1
ls -la
```

**Why this matters**  
Keeping a consistent folder structure makes your recon reproducible. When you hand in deliverables, everything should be in `~/week1`.

### Safety note (important)

Never run destructive commands on your host machine. Keep all experiments inside the lab VMs. If you use `rm -rf`, double‑check the path. Use `--no-preserve-root` only when you know exactly what you’re doing (and you probably don’t).

[[#🧭 Table of Contents]]

---

## 4 — Networking verification lab (Lab 1) — deep walkthrough (45–60 minutes) 🌐🔍

**Purpose:** Ensure Kali is correctly configured for Host‑Only + NAT topologies and can reach the lab network. This lab prevents the most common failures in later weeks.

### The topology we expect

Kali will be on a host‑only network (e.g., `192.168.56.0/24`) or a GNS3 internal network (e.g., `192.168.10.0/24`) depending on your setup. The target VM (MS2) will be on a different subnet behind pfSense in the Day 0 topology, or on the same host‑only network for simple setups. The important part is that you know the IP ranges and gateways.

### Step A — Check interfaces and addresses

Run:

```bash
ip addr show
ip -4 route show
```

**Expected output and how to read it**

`ip addr show` lists interfaces and their IPs. Look for `eth0`, `ens3`, or `enp0s3` with an `inet` line like `inet 192.168.10.10/24`. `ip -4 route show` shows the routing table; you should see a `default via 192.168.10.1` or similar.

**Why this matters**  
If the interface has no IP, the VM isn’t connected to the right virtual network. If the default route is missing, the VM won’t reach other subnets.

### Step B — Ping the gateway and a known host

First ping your gateway:

```bash
ping -c 4 192.168.10.1
```

Then ping another VM you expect to exist (replace `<other-VM-IP>`):

```bash
ping -c 4 <other-VM-IP>
```

**Interpreting results**

A successful ping shows `64 bytes from ... time=... ms`. If you see `Destination Host Unreachable` or `100% packet loss`, the packet didn’t reach the destination. Use the troubleshooting chain below.

### Step C — Troubleshooting chain (theory + commands)

If ping fails, follow this order:

1. **Is the interface up?**
    
    ```bash
    ip link show eth0
    ```
    
    If it’s `DOWN`, bring it up:
    
    ```bash
    sudo ip link set eth0 up
    ```
    
2. **Is the IP assigned?**  
    If `ip addr` shows no `inet` line, assign a static IP (temporary for testing):
    
    ```bash
    sudo ip addr add 192.168.10.10/24 dev eth0
    sudo ip route add default via 192.168.10.1
    ```
    
3. **Is the virtual network connected?**  
    Check GNS3/VirtualBox network settings: the VM’s NIC should be attached to the correct host‑only or internal network.
    
4. **Is the gateway alive?**  
    From the host or another VM, ping the gateway. If the gateway doesn’t respond, the problem is outside Kali.
    
5. **Is a firewall blocking ICMP?**  
    On pfSense or the host firewall, ICMP may be blocked. Try a TCP connection test instead (see netcat below).
    

**Why this chain works**  
Each step tests a different layer: link (interface up), network (IP assigned), virtualization (network attached), and policy (firewall). This is the same reasoning you’ll use for every network problem.

[[#🧭 Table of Contents]]

---

## 5 — Host discovery & recon (Lab 2) — deep walkthrough (60–90 minutes) 🔎🧭

**Purpose:** Learn the core reconnaissance tools and how to interpret their outputs. Recon is the foundation of every pentest.

### Tool 1 — `nmap` (host discovery and port scanning)

Start with a ping sweep to find live hosts on a subnet:

```bash
sudo nmap -sn 192.168.10.0/24 -oN ~/week1/scans/nmap_ping_sweep.txt
```

**What this does and why**  
`-sn` performs host discovery (no port scan). `sudo` is used because some discovery techniques require raw sockets. The output file records the hosts that responded.

**Expected output snippet**

```
Nmap scan report for 192.168.10.1
Host is up (0.0010s latency).
Nmap scan report for 192.168.10.10
Host is up (0.0020s latency).
```

**Theory**  
Nmap uses ICMP, ARP, and TCP/UDP probes to determine if a host is up. ARP discovery is the most reliable on local networks.

### Tool 2 — `netdiscover` and `arp-scan` (alternate discovery)

If `nmap -sn` misses hosts, try:

```bash
sudo netdiscover -r 192.168.10.0/24
sudo arp-scan --interface=eth0 192.168.10.0/24
```

**Why use these**  
`netdiscover` and `arp-scan` use ARP to find hosts on the local link. ARP is often more reliable for local discovery because it doesn’t rely on ICMP being allowed.

### Tool 3 — `nmap` service scan (first full port scan)

Once you have a target IP (e.g., `192.168.20.10` for MS2), run a service scan:

```bash
sudo nmap -sS -sV -p 1-1000 192.168.20.10 -oN ~/week1/scans/nmap_service_scan.txt
```

**What the flags mean**

- `-sS` = SYN scan (stealthy, fast).
- `-sV` = service/version detection.
- `-p 1-1000` = scan the first 1000 ports.
- `-oN` = save output in normal format.

**Expected output snippet**

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2
80/tcp   open  http    Apache httpd 2.4.18
3306/tcp open  mysql   MySQL 5.5.60
```

**Theory**  
SYN scans send a SYN and interpret SYN‑ACK as open, RST as closed. `-sV` attempts to speak to the service to get a banner. Banners are useful for identifying versions and potential vulnerabilities.

### Interpreting results and documenting

For each host you discover, record:

- IP address and MAC (if available).
- Open ports and services.
- Service versions and any banners.
- Any oddities (filtered ports, unusual TTLs).

Save screenshots of the terminal output and the `nmap` output files in `~/week1/scans/`.

---
[[#🧭 Table of Contents]]
## 6 — First full port scan (Lab 3) — deep walkthrough (60–90 minutes) ⚔️

**Purpose:** Perform a full TCP scan on MS2 and capture the output for analysis.

### Full TCP scan command

```bash
sudo nmap -p- -T4 192.168.20.10 -oN ~/week1/scans/week1_ms2_fullscan.txt
```

**What this does**

`-p-` scans all 65535 TCP ports. `-T4` increases speed (use with caution on noisy networks). This scan is heavier and takes longer.

**Expected behavior and outputs**

The scan will take several minutes. You’ll see progress and then a list of open ports. Save the output and a screenshot.

**Theory**

Full scans are noisy and obvious to defenders, but they reveal services that live on nonstandard ports. Use them in lab environments where you have permission.

---
[[#🧭 Table of Contents]]
## 7 — Quick tool primers 🧰

### `curl` — fetch HTTP resources

```bash
curl -I http://192.168.20.10:80
```

`-I` fetches headers only. You’ll see `HTTP/1.1 200 OK` and `Server: Apache/2.4.18`.

**Why**: `curl` is the simplest way to interact with web services and read headers.

### `netcat` — raw TCP/UDP connections

Start a listener on Ubuntu:

```bash
# on Ubuntu
nc -l -p 4444
```

Connect from Kali:

```bash
# on Kali
nc 192.168.20.10 4444
```

Type text and observe it appear on the other side.

**Why**: `nc` is the Swiss Army knife for quick service checks and simple file transfers.

### `gobuster` — directory brute force for web apps

```bash
gobuster dir -u http://192.168.20.10 -w /usr/share/wordlists/dirb/common.txt -t 20 -o ~/week1/scans/gobuster_root.txt
```

**Why**: `gobuster` finds hidden directories and files that may host admin panels or vulnerable scripts.

### `ssh` — remote shell

```bash
ssh user@192.168.20.10
```

If SSH is open, you’ll be prompted for credentials. In lab environments you may have default credentials to test.

**Why**: SSH is the primary remote access method; understanding it is essential for pivoting and post‑exploitation.

---
[[#🧭 Table of Contents]]
## 8 — Documentation: how to record recon (Assignment 1 prep) 📝

Every scan you run must be reproducible. For each recon action, record:

1. The exact command you ran (copy/paste).
2. The timestamp.
3. The output file path (e.g., `~/week1/scans/nmap_service_scan.txt`).
4. A short interpretation: what the output means and what you would try next.

Create a Recon Template note in your vault (use `[[0502 Recon Template]]` if you have it) and paste the commands and outputs. Screenshots go in `~/week1/screenshots/`.

**Why this matters**  
Good documentation turns noisy scans into actionable intelligence. It also makes your work auditable and repeatable.

---

## 9 — Mini‑Project: Build your offensive workflow (1.5 hours) 🧩

**Goal:** Create a single page in your vault `[[0601 Offensive Workflow]]` that becomes your master checklist for future labs.

Your workflow should include:

- How you start a lab (power on VMs, verify network).
- The first three recon commands you always run.
- Where you store outputs and naming conventions (e.g., `nmap_<target>_<date>.txt`).
- How you take screenshots and name them.
- How you create a report skeleton for assignments.

**Why this matters**  
A consistent workflow saves time and reduces mistakes. It also makes your results easier to share and review.

[[#🧭 Table of Contents]]

---

## 10 — Assignments (detailed expectations and grading hints) 🧾

### Assignment 1 — Recon Documentation (1 hour)

Deliverable: A filled Recon Template with:

- Network diagram (simple ASCII or image) showing Kali, pfSense, MS2, and IP ranges.
- Commands run and output file paths.
- Screenshots of `nmap` and `curl` outputs.
- A short analysis of MS2’s attack surface (services, versions, likely vulnerabilities).

**Grading hints:** Show the exact commands and outputs. Explain why each open port matters. If you claim a vulnerability, cite a CVE or explain why the version is suspicious.

### Assignment 2 — Tooling Deep Dive (1 hour)

Pick one tool and produce a 1–2 page analysis that includes:

- Purpose and typical use cases.
- Common flags and what they do.
- Example commands with expected output.
- When to use this tool in a real pentest and what artifacts it produces.

**Grading hints:** Demonstrate the tool with a short example from your lab and include the output file.

[[#🧭 Table of Contents]]

---

## 11 — Instructor notes and pedagogy (how to use this day effectively) 🧑‍🏫

This day is foundational. Don’t rush. Spend time making your scans reproducible and your notes readable. Recon is the majority of real pentesting: the better your recon, the fewer blind alleys you follow later.

If networking breaks, fix it now. Many students postpone network debugging and then waste hours later. Use the troubleshooting chain above and ask for help with exact command outputs if you get stuck.

[[#🧭 Table of Contents]]

---

## 12 — Reflection prompts (short, honest, useful) 🤔

After you finish Day 1, write short answers in your notes:

- Which tool felt most intuitive and why?
- Which command or concept confused you the most?
- What took the longest and why?
- How would you change your workflow to be faster tomorrow?

Reflection turns practice into learning.

[[#🧭 Table of Contents]]

---

## 13 — Deliverables checklist (what to hand in) 📦

Place the following in `~/week1/` and in your vault:

- `scans/` — all scan outputs (`nmap`, `gobuster`, etc.).
- `screenshots/` — terminal screenshots showing key outputs.
- `pcaps/` — any packet captures you made.
- `notes/` — Recon Template filled and Offensive Workflow page created.
- Assignment files: Recon Documentation and Tooling Deep Dive.

[[#🧭 Table of Contents]]

---

## 14 — Extra troubleshooting appendix (common errors and exact fixes) 🛠️

**Problem:** `nmap` returns “Failed to open device eth0 (You must be root).”  
**Fix:** Run with `sudo` or become root: `sudo -i`. Theory: raw socket operations require elevated privileges.

**Problem:** `curl` times out to a host that pings fine.  
**Fix:** The service may be down or blocked by a firewall. Use `nc -vz <ip> <port>` to test TCP connectivity. Theory: ICMP and TCP are different layers; one can be allowed while the other is blocked.

**Problem:** `gobuster` finds nothing but you know the site has content.  
**Fix:** Try a different wordlist and increase threads: `-w /usr/share/wordlists/dirb/big.txt -t 50`. Also check for virtual hosts or authentication. Theory: directory brute force depends on the wordlist and server behavior.

[[#🧭 Table of Contents]]

---

## 15 — Short practice exercises (do these now, with expected outputs) 🏁

1. **Confirm network**: Run `ip addr show` and `ip route show`. Copy the output into `~/week1/notes/network_status.txt`.
2. **Ping sweep**: Run `sudo nmap -sn 192.168.10.0/24 -oN ~/week1/scans/ping_sweep.txt`. Open the file and paste the first three lines into your notes with a one‑sentence interpretation.
3. **Service scan**: Run `sudo nmap -sS -sV -p 22,80,8080 192.168.20.10 -oN ~/week1/scans/service_scan.txt`. Take a screenshot of the output and save it to `~/week1/screenshots/`.
4. **HTTP fetch**: Start a Python HTTP server on Ubuntu: `python3 -m http.server 8000`. From Kali run `curl -I http://192.168.20.10:8000` and save the headers to `~/week1/scans/http_headers.txt`.

Each exercise produces an artifact you will include in your deliverables.

[[#🧭 Table of Contents]]

---

If you want, I’ll now expand any single section into a full, clickable walkthrough with exact menu names and described screenshots: for example, a **pfSense web UI walkthrough** (change admin password, create firewall rules, enable NAT logging), a **GNS3 troubleshooting guide** (interface mapping, common VM errors), or a **Wireshark tutorial** (filters, follow stream, interpret fields) — each in the same patient, theory‑rich style with commands and expected outputs. Tell me which one and I’ll produce it next.