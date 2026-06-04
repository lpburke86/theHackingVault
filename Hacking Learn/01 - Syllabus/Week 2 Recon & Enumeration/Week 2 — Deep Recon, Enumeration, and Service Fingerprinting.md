# 🧭 Table of Contents

- [[#Week 2 — Deep Recon, Enumeration, and Service Fingerprinting (Full, Detailed Chapter) 🔎🧭]]
- [[#Day 2 — Short intro and main goals 🔎🧭]]
- [[#1 — The mental model for Week 2 🧠]]
- [[#2 — Before you start: environment and artifacts checklist ✅]]
- [[#3 — Core theory: services, fingerprints, and why versions matter 🔬]]
- [[#4 — Lab 1 — Full service enumeration on MS2 — deep walkthrough (60 minutes) 🛠️]]
- [[#5 — Lab 2 — SMB enumeration — deep walkthrough (60 minutes) 🗂️]]
- [[#6 — Lab 3 — Web enumeration — deep walkthrough (60 minutes) 🌐]]
- [[#7 — Lab 4 — Database enumeration — deep walkthrough (60 minutes) 🗄️]]
- [[#8 — Assignments — expectations and grading hints 🧾]]
- [[#9 — Mini‑Project — Attack Surface Map (MS2) 🧩]]
- [[#10 — Documentation and reporting — how to record everything 📝]]
- [[#11 — Troubleshooting and common pitfalls 🛠️]]
- [[#12 — Reflection prompts 🤔]]
- [[#13 — Deliverables checklist 📦]]
- [[#14 — Short practice exercises (do these now) 🏁]]

---

# Week 2 — Deep Recon, Enumeration, and Service Fingerprinting (Full, Detailed Chapter) 🔎🧭

**Short intro — what Day 2 is for**  
Day 2 is the day you stop treating open ports as mere numbers and start treating them as stories. Each service you find is a small narrative about software, configuration, and possible weaknesses. Today you will learn to read those narratives: run careful probes, collect reliable artifacts, and convert raw outputs into a prioritized plan of attack. The emphasis is on interpretation and evidence — not on running every noisy tool, but on running the right probes, saving the outputs, and explaining what they mean.

**Main goals for Day 2**

You will perform full‑spectrum enumeration on a target, identify high‑value services, use Nmap NSE scripts and specialized tools for SMB, FTP, SSH, HTTP, and MySQL, and produce an Attack Surface Map for MS2 that turns raw data into a prioritized attack chain. Every command you run will be saved, timestamped, and interpreted so your work is reproducible and auditable.  
[[#🧭 Table of Contents]]

---

## Day 2 — Short intro and main goals 🔎🧭

This week bridges scanning and exploitation. The difference between a noisy scan and a useful reconnaissance report is interpretation. When you see a banner that says `Apache/2.2.8`, that string is not the final answer — it is a clue you verify with behavior, packet captures, and cross‑checks. Today’s labs are structured experiments: form a hypothesis about a service, run targeted probes to test it, capture the traffic, and record the evidence. By the end of the day you will have a documented, prioritized plan for MS2 that you can hand to a teammate or use as the basis for safe exploitation in later weeks.  
[[#🧭 Table of Contents]]

---

## 1 — The mental model for Week 2 🧠

Treat enumeration like detective work. A service banner is a witness statement; a version number is a fingerprint; a filtered port is a locked door. Your job is to collect multiple, independent pieces of evidence and weigh their reliability. Start every probe with a question: “What am I trying to learn?” Then choose the least destructive probe that answers that question. For example, to confirm whether an FTP server allows anonymous access, a simple `smbclient` or `ftp` connection attempt is better than a full brute force. Always save the raw output and a packet capture so you can show exactly what was sent and received. This habit — evidence first, conclusion second — is what separates good recon from guesswork.  
[[#🧭 Table of Contents]]

---

## 2 — Before you start: environment and artifacts checklist ✅

Before you run any enumeration, make sure your environment is stable and your artifact workflow is ready. Create a `~/week2/` directory with `scans/`, `screenshots/`, `pcaps/`, and `notes/` subfolders and confirm your VM clocks are correct so timestamps line up with packet captures and logs. Verify you can reach MS2 with a simple ping and that you have `sudo` available for raw socket scans. Install or confirm availability of `tcpdump` or Wireshark so you can capture traffic during active probes. The reason for this preparation is practical: enumeration is noisy and time‑sensitive; if you lose the pcap or forget to save the Nmap XML, you lose the evidence that supports your conclusions. Run these commands to prepare and check:

```bash
mkdir -p ~/week2/{scans,screenshots,pcaps,notes}
cd ~/week2
date
sudo -v
which tcpdump || sudo apt update && sudo apt install -y tcpdump
```

If any step fails, fix it before proceeding — a missing pcap or a wrong clock will force you to repeat scans and waste time.  
[[#🧭 Table of Contents]]

---

## 3 — Core theory: services, fingerprints, and why versions matter 🔬

Before you probe, understand what you’re looking for. Service identification answers “what protocol is speaking on this port?” Fingerprinting answers “which implementation and version?” and configuration checks answer “how is it set up?” A banner like `OpenSSH 4.7p1` is a starting point; it tells you which codebase the server uses and suggests which CVEs to check. But banners can be faked or truncated by proxies and load balancers, so always corroborate banners with behavior: try a benign protocol negotiation, capture the handshake, and inspect the raw bytes.

Nmap’s NSE scripts are powerful because they automate common checks — `ftp-anon` tests for anonymous FTP, `smb-enum-shares` lists SMB shares, `http-enum` probes for common web paths. Use them as structured probes, not as final verdicts. When a script reports “anonymous FTP allowed,” follow up manually: connect with `ftp` or `smbclient`, list files, and capture the session. That manual verification is the evidence you will cite in your report.

Finally, remember false positives and filtering. A port that appears “open|filtered” may be silently dropping probes. A slow response may indicate rate limiting or an IDS. When in doubt, capture the traffic and read the packets — the wire never lies.  
[[#🧭 Table of Contents]]

---

## 4 — Lab 1 — Full service enumeration on MS2 — deep walkthrough (60 minutes) 🛠️

Begin with a careful, baseline Nmap scan that runs safe NSE scripts and performs version detection. The command below is the starting point because it balances information with safety and produces artifacts you can parse later.

```bash
sudo nmap -sC -sV -oN ~/week2/scans/ms2_enum.txt <MS2-IP>
```

When this runs, watch the output for open ports and the NSE script lines. A typical result might show FTP, SSH, HTTP, SMB, and MySQL. For each line, copy the exact banner into your notes. For example, if Nmap reports `vsftpd 2.3.4` on port 21, record that string verbatim — it is the primary piece of evidence you will use to justify further tests.

Next, save machine‑readable output for automation and reporting:

```bash
sudo nmap -sC -sV -oA ~/week2/scans/ms2_enum_full <MS2-IP>
```

This produces `.nmap`, `.xml`, and `.gnmap` files. The XML is especially useful when you later generate tables or feed results into scripts.

While the scan runs, start a packet capture on your Kali interface so you can later show the exact probes and responses:

```bash
sudo tcpdump -i eth0 host <MS2-IP> -w ~/week2/pcaps/ms2_enum.pcap
# run the nmap scan in another terminal, then stop tcpdump with Ctrl+C
```

After the scan completes, open the human‑readable output and extract the ports you care about. For each service, write a one‑sentence interpretation: what the service is, how confident you are in the banner, and what you would test next. Put that into `~/week2/notes/ms2_initial_findings.md`. For example:

```
21/tcp ftp vsftpd 2.3.4 — banner present; NSE ftp-anon reports anonymous allowed. Next: attempt anonymous login and list files; capture session.
22/tcp ssh OpenSSH 4.7p1 — banner present; check for weak algorithms and try public key auth enumeration.
80/tcp http Apache 2.2.8 — check robots.txt, run directory discovery, and inspect server headers for modules.
```

This lab is about evidence collection and triage. Don’t rush to exploit; instead, build a prioritized list of what to investigate manually.  
[[#🧭 Table of Contents]]

---

## 5 — Lab 2 — SMB enumeration — deep walkthrough (60 minutes) 🗂️

SMB is a rich source of information: shares, file contents, and sometimes usernames. Start by listing shares with `smbclient` using anonymous login. This is a low‑impact check that often yields immediate results.

```bash
smbclient -L //<MS2-IP>/ -N
```

Read the output carefully. If shares are listed, note their names and any comments. If `smbclient` shows `Anonymous login successful`, that is a high‑value finding because it often leads to readable files or configuration leaks.

Next, run `enum4linux` for a broader sweep. `enum4linux` bundles multiple SMB checks — user enumeration, share listing, and policy queries — into one output. Save that output and treat it as evidence.

```bash
enum4linux -a <MS2-IP> | tee ~/week2/scans/enum4linux_ms2.txt
```

When `enum4linux` returns a `Users:` section, copy those usernames into `~/week2/notes/ms2_usernames.txt`. Usernames are valuable for later password guessing or for checking whether the same names appear in web forms or config files.

If a share appears writable or contains files, mount it or use `smbclient` interactively to list and download files. Mounting via CIFS gives you a filesystem view and lets you search for credentials or scripts:

```bash
sudo mount -t cifs //MS2-IP/public /mnt/ms2_public -o guest,ro
ls -la /mnt/ms2_public
```

Always capture the SMB session if you interact with files so you can show the exact commands and responses. Save any interesting files into `~/week2/scans/` and note their paths and contents in your notes. SMB often yields the most actionable artifacts early in a lab because configuration files and credentials are frequently stored on shares.  
[[#🧭 Table of Contents]]

---

## 6 — Lab 3 — Web enumeration — deep walkthrough (60 minutes) 🌐

Web enumeration is both art and science. Start with lightweight probes: headers and robots. Use `curl -I` to fetch headers and `curl` to fetch `robots.txt`. These quick checks often reveal server software and hidden paths.

```bash
curl -I http://<MS2-IP>
curl -s http://<MS2-IP>/robots.txt
```

Next, run a directory discovery with `gobuster`. Choose a reasonable wordlist and thread count; too many threads can trigger rate limits or WAFs, too few will waste time. Save the output to your scans folder.

```bash
gobuster dir -u http://<MS2-IP> -w /usr/share/wordlists/dirb/common.txt -t 40 -o ~/week2/scans/gobuster_ms2_root.txt
```

When `gobuster` returns 200, 301, or 302 responses, open those URLs in a browser or fetch them with `curl` to inspect content. A 200 on `/admin` or `/uploads` is a clear next step: look for forms, file upload endpoints, or default credentials.

If you want a broader automated check, run `nikto` to surface common misconfigurations and outdated components. Nikto is noisy but useful in a lab:

```bash
nikto -h http://<MS2-IP> -output ~/week2/scans/nikto_ms2.txt
```

Also test virtual host behavior by sending different `Host:` headers. Some staging or admin panels are only accessible via a specific host header:

```bash
curl -I -H "Host: admin.example.com" http://<MS2-IP>
```

Save screenshots of interesting pages and copy any forms or parameters into your notes. For each discovered path, record the HTTP status code, the content type, and why it matters (e.g., “/uploads accepts POST and returns 200 — potential file upload endpoint”). These small, structured observations are what you will use to build your web enumeration report.  
[[#🧭 Table of Contents]]

---

## 7 — Lab 4 — Database enumeration — deep walkthrough (60 minutes) 🗄️

Databases often contain the keys to the kingdom: credentials, configuration, and user data. Start with Nmap’s MySQL NSE scripts to gather handshake information and check for weak authentication or anonymous access.

```bash
sudo nmap --script=mysql* -p 3306 <MS2-IP> -oN ~/week2/scans/mysql_nse_ms2.txt
```

Read the script output for version strings and any notes about empty passwords or anonymous access. If the script indicates the server allows connections, attempt a safe, read‑only query. Try connecting with an empty password first (only in your lab):

```bash
mysql -h <MS2-IP> -u root -p'' -e "SHOW DATABASES;" || echo "no anonymous access"
```

If you can connect, list schemas and inspect tables for configuration files or credentials. Save any schema dumps to `~/week2/scans/` and note which tables look sensitive. If MySQL refuses connections despite the port being open, the server may be bound to `127.0.0.1` only; use Nmap scripts to check binding behavior and document that finding.

Repeat equivalent checks for other DBs if Nmap reveals them (PostgreSQL on 5432, MongoDB on 27017). For each DB, record the version, whether remote connections are allowed, and any evidence of weak authentication. Databases are high‑value targets; even a single credential found in a config file can lead to full compromise.  
[[#🧭 Table of Contents]]

---

## 8 — Assignments — expectations and grading hints 🧾

For the Service Fingerprinting Report, produce a methodical document that lists each discovered service with port, protocol, exact banner, discovery method, and your confidence level. Include the exact commands you ran and the output snippets that support your claims. If you assert a vulnerability, show the banner or behavior that led you to that conclusion and, if possible, cite a CVE or vendor advisory. The Web Enumeration Report should list discovered directories with status codes, screenshots of interesting pages, and a short plan for next steps (e.g., test file upload on `/uploads`, test SQLi on parameter `id`). The graders will look for reproducibility: can they re-run your commands and see the same artifacts. Save all outputs in `~/week2/scans/` and reference them in your reports.  
[[#🧭 Table of Contents]]

---

## 9 — Mini‑Project — Attack Surface Map (MS2) 🧩

Your Attack Surface Map is a single page in your vault that turns enumeration into a plan. Create `[[0602 Attack Surface Map — MS2]]` and include a table of services (port, banner, discovery method, confidence), a prioritized attack chain (why you would test FTP before SSH, for example), and notes on required tools and expected artifacts. For each prioritized step, write the exact command you would run and the artifact you expect to collect (e.g., “Attempt anonymous FTP: `smbclient -L //<MS2-IP>/ -N` — expect file listing; save to `~/week2/scans/ftp_list.txt`). This map is your playbook for safe, methodical testing.  
[[#🧭 Table of Contents]]

---

## 10 — Documentation and reporting — how to record everything 📝

Every command must be recorded with a timestamp, the exact command string, the output file path, and a one‑line interpretation. Use ISO timestamps so your notes align with pcaps. An example entry in your notes might look like this:

```
2026-06-03T20:00Z
Command: sudo nmap -sC -sV -oN ~/week2/scans/ms2_enum.txt 192.168.20.10
Output: ~/week2/scans/ms2_enum.txt
Interpretation: FTP (21) open, vsftpd 2.3.4 banner; NSE ftp-anon reports anonymous allowed. Next: attempt anonymous login and list files; capture traffic.
```

Store raw outputs in `~/week2/scans/`, screenshots in `~/week2/screenshots/`, and pcaps in `~/week2/pcaps/`. This structure makes your work reproducible and auditable. When you hand in assignments, reviewers should be able to re-run your commands and see the same artifacts.  
[[#🧭 Table of Contents]]

---

## 11 — Troubleshooting and common pitfalls 🛠️

If a banner looks generic or missing, capture the packets and inspect the raw bytes; proxies and load balancers often strip or rewrite headers. If `enum4linux` times out, try `smbclient` or `smbmap` and check for SMB signing or firewall rules. If `gobuster` returns many 403s, try different wordlists, reduce thread count, or include 403 in results with `-s 200,301,302,403`. If MySQL refuses connections despite an open port, the service may be bound to localhost; use Nmap scripts to check binding and document the behavior. The troubleshooting approach is always the same: form a hypothesis about why a probe failed, run a targeted test that isolates the variable, capture the evidence, and record the result.  
[[#🧭 Table of Contents]]

---

## 12 — Reflection prompts 🤔

After Day 2, write short answers in your notes about what worked and why. Which enumeration technique produced the most actionable evidence? Which service surprised you and why? What took the longest and how could you speed it up next time? Which findings changed your attack hypothesis? These reflections convert raw practice into durable learning.  
[[#🧭 Table of Contents]]

---

## 13 — Deliverables checklist 📦

Place the following in `~/week2/` and in your vault: all scan outputs (`nmap`, `enum4linux`, `gobuster`, `nikto`), screenshots of interesting pages and terminal outputs, pcaps for key probes, notes including the Service Fingerprinting Report and Web Enumeration Report, and the Attack Surface Map `[[0602 Attack Surface Map — MS2]]`. Ensure filenames are consistent and include dates so reviewers can follow your timeline.  
[[#🧭 Table of Contents]]

---

## 14 — Short practice exercises (do these now) 🏁

Run the full enumeration and save the XML, run `enum4linux` and extract usernames, run `gobuster` and screenshot any 200 responses, and run the MySQL NSE scripts and note any version strings. Each exercise should produce an artifact saved in `~/week2/` and referenced in your reports. For example, run:

```bash
sudo nmap -sC -sV -oA ~/week2/scans/ms2_enum_full <MS2-IP>
enum4linux -a <MS2-IP> | tee ~/week2/scans/enum4linux_ms2.txt
gobuster dir -u http://<MS2-IP> -w /usr/share/wordlists/dirb/common.txt -t 40 -o ~/week2/scans/gobuster_ms2_root.txt
sudo nmap --script=mysql* -p 3306 <MS2-IP> -oN ~/week2/scans/mysql_nse_ms2.txt
```

After each command, write a one‑line interpretation and save it to `~/week2/notes/`. These small, repeatable exercises build the habits you’ll use throughout the course.  
[[#🧭 Table of Contents]]

---

If you want, I will expand one of the labs into a clickable, step‑by‑step walkthrough with exact menu names, annotated screenshots, and packet capture annotations — for example, a fully annotated SMB lab that walks through `enum4linux` output line‑by‑line, or a web enumeration walkthrough that explains each `gobuster` hit and how to validate it. Tell me which lab you want expanded and I’ll produce the clickable walkthrough in the same detailed, narrative style.