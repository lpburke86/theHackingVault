### Table of Contents 📚

[[#Overview and Goals]]  
[[#How Nmap Works Under the Hood]]  
[[#Host Discovery Techniques]]  
[[#TCP Scans Deep Dive]]  
[[#UDP Scanning Deep Dive]]  
[[#Service Fingerprinting and OS Detection]]  
[[#Nmap Scripting Engine NSE Deep Dive]]  
[[#Timing Performance and Stealth Tuning]]  
[[#Evidence Capture and Forensic Discipline]]  
[[#Practical Labs Step by Step]]
- [[#Lab 1 Discovery and Baseline Inventory]]
- [[#Lab 2 Full Port Discovery and Fingerprinting]]
- [[#Lab 3 NSE Vulnerability Verification]]
- [[#Lab 4 UDP Enumeration and Reliability]]
- [[#Lab 5 Custom NSE Script Trace and Validation]]  
[[#Detection Defensive Signals and Evasion Considerations]]  
[[#Common Problems and Troubleshooting]]  
[[#Reporting and Packaging Findings]]  
[[#Appendix Commands Examples and Templates]]

---

# Overview and Goals

This is a complete, **zero‑to‑hero** manual for **Nmap** written as a practical, forensic, and teaching reference. It covers everything from installation and basic discovery to advanced fingerprinting, the Nmap Scripting Engine, evidence capture, and safe vulnerability verification. The guide is organized so you can read it straight through or jump to the labs and appendices for copy‑paste commands.

**What you will learn**

- How Nmap probes networks and why each probe behaves the way it does.
- When to use each scan type and how to tune timing and retries.
- How to fingerprint services and operating systems reliably.
- How to use NSE to enumerate, verify, and triage vulnerabilities safely.
- How to capture packet‑level evidence that proves your findings.
- How to package minimal artifacts for reports that defenders can validate quickly.

This guide emphasizes **what** each command does and **why** you run it, and it includes forensic discipline: start captures before scans, extract minimal slices, compute checksums, and produce a manifest that ties commands to packet numbers and timestamps. Every code block includes an explanation.

[[#Table of Contents 📚]]

---

# How Nmap Works Under the Hood

Nmap is a packet generator and response interpreter. At its core it sends crafted network packets and interprets the responses to infer host state, service versions, and operating system characteristics. Understanding the packet flow is essential: the same port can appear differently depending on whether you send a SYN, a full TCP handshake, a UDP datagram, or an application‑level probe. Nmap’s heuristics combine timing, TCP/IP stack quirks, banner text, and protocol behavior to produce a best‑effort fingerprint.

When you run a SYN scan, Nmap sends a TCP packet with only the SYN flag set. If the target responds with SYN/ACK, Nmap infers the port is open and sends a RST to avoid completing the handshake. If the target responds with RST, the port is closed. If there is no response or an ICMP unreachable, the port is considered filtered. These three outcomes are the raw signals Nmap uses to build higher‑level conclusions.

Nmap’s version detection sends small, protocol‑specific probes to open ports and matches the responses against a large signature database. OS detection sends a set of probes that exercise subtle differences in TCP/IP stack behavior: TTL values, window sizes, options ordering, and ICMP responses. The Nmap Scripting Engine (NSE) runs Lua scripts in phases and can perform discovery, authentication, brute force, or vulnerability checks. NSE scripts are categorized by risk and purpose; understanding those categories is critical to avoid causing harm.

[[#Table of Contents 📚]]

---

# Host Discovery Techniques

Host discovery is the first step in any scan plan. The goal is to determine which IPs are live so you can focus enumeration where it matters. On an Ethernet LAN, ARP is the most reliable and fastest method because ARP is a link‑layer protocol that cannot be filtered by host firewalls. When you run an ARP sweep, Nmap sends ARP who‑has requests and receives ARP replies from any host that owns the IP. This gives you a definitive list of live hosts on the segment.

When scanning across routed networks, ARP is not available and you must rely on IP‑level probes. Nmap’s default host discovery uses a combination of ICMP echo requests, TCP SYN to common ports, and TCP ACK probes. Each probe has tradeoffs. ICMP is simple and fast but is often blocked by firewalls. TCP SYN to port 443 or 80 can succeed where ICMP fails because many hosts allow outbound web traffic. TCP ACK is useful to elicit responses from stateful firewalls that will reply with RST for unreachable ports.

**Example ARP discovery**

```bash
sudo nmap -sn -PR 192.168.20.0/24 -oA discovery_arp
```

**What and why**: `-sn` performs host discovery only. `-PR` forces ARP discovery on the local LAN; ARP cannot be filtered by host firewalls and gives a definitive live host list. `-oA` saves normal, XML, and grepable outputs for automation and reporting. Run this when you are on the same Ethernet segment as the targets.

**Mixed IP probes when ARP is not available**

```bash
sudo nmap -sn -PE -PS22,80,443 -PA3389 10.0.0.0/24 -oA discovery_mixed
```

**What and why**: `-PE` sends ICMP echo requests. `-PS22,80,443` sends TCP SYN probes to ports 22, 80, and 443. `-PA3389` sends TCP ACK to port 3389. This combination increases the chance of discovering hosts that block ICMP but allow specific TCP ports. Save outputs and capture traffic to prove which probe elicited the reply.

**Evidence discipline**: start `tcpdump` before running discovery, index ARP/ICMP frames with `tshark`, and include packet numbers in your manifest.

[[#Table of Contents 📚]]

---

# TCP Scans Deep Dive

TCP scanning is the most common form of port discovery. The two primary modes are SYN scan and Connect scan. SYN scan is often called a half‑open scan because it does not complete the TCP handshake. Connect scan uses the operating system’s connect call and completes the handshake. SYN scan is faster and leaves less evidence in application logs because it avoids completing connections, but it still appears in network logs and IDS signatures.

**SYN scan example**

```bash
sudo nmap -sS -p- --min-rate 1000 192.168.20.10 -oA syn_full
```

**What and why**: `-sS` runs a SYN scan. `-p-` instructs Nmap to scan all 65,535 TCP ports. `--min-rate 1000` forces Nmap to send at least 1000 probes per second, which is useful in lab environments to finish quickly; do not use such aggressive rates on production. Running as root is required for raw socket operations. The `-oA` saves the outputs for later analysis. Use SYN scans for broad discovery when you have permission and when you want to minimize application‑level logs.

**Connect scan example**

```bash
nmap -sT -p 22,80,443 192.168.20.10 -oN connect_scan.txt
```

**What and why**: `-sT` performs a full TCP connect scan using the OS networking stack. This is necessary on systems where raw sockets are not available (for example, some Windows environments). Connect scans are noisier because they complete the handshake and may trigger application logs or IDS alerts. Use `-sT` when you cannot run `-sS` or when you need to ensure the service accepts full connections.

There are specialized TCP scans for specific use cases. ACK scans help map firewall rules by observing whether a firewall responds with RST or drops the packet. Window and Idle scans are advanced techniques that can be used to infer port state without sending probes from your own IP, but they require careful setup and are rarely necessary for routine assessments.

**Service fingerprinting after discovery**

```bash
sudo nmap -sS -p22,80,443 --version-intensity 5 -sV 192.168.20.10 -oA syn_sv
```

**What and why**: This command runs a SYN scan on the three most common service ports and then runs `-sV` to detect service versions. `--version-intensity` controls how aggressive the version probes are; higher values increase accuracy at the cost of time and potential side effects. Use moderate intensity on production systems and higher intensity in labs.

**Forensic capture**: always run `tcpdump` before the scan and extract the exact SYN and SYN/ACK packets that prove a port is open. Use `tshark` to index frames and `editcap` to slice minimal evidence.

[[#Table of Contents 📚]]

---

# UDP Scanning Deep Dive

UDP scanning is fundamentally different from TCP scanning because UDP is connectionless and many services do not respond unless the payload is valid. Nmap sends UDP datagrams and interprets ICMP port unreachable messages as evidence that the port is closed. If there is no response, the port is reported as open|filtered because it could be open and silently dropping probes or filtered by a firewall.

UDP scans are slow because Nmap must wait for timeouts and may need to retry probes. You must tune retries and timeouts for reliability. For common UDP services such as DNS, NTP, SNMP, and syslog, target the specific ports rather than scanning the entire UDP space.

**UDP scan example**

```bash
sudo nmap -sU -p 53,123,161 --max-retries 5 --host-timeout 2m 192.168.20.10 -oN udp_scan.txt
```

**What and why**: `-sU` runs a UDP scan. `-p 53,123,161` targets DNS, NTP, and SNMP. `--max-retries 5` increases the number of retransmissions for UDP probes to improve reliability. `--host-timeout 2m` prevents the scan from hanging indefinitely on a single host. UDP scanning is essential for a complete surface assessment, but expect many `open|filtered` results that require follow‑up with application‑level probes or service‑specific tools.

When Nmap reports a UDP port as open, follow up with a protocol‑aware probe. For DNS, send a valid DNS query and capture the response. For SNMP, attempt a safe community string query that does not alter state. Always capture the traffic so you can show the exact request and response in your evidence package.

[[#Table of Contents 📚]]

---

# Service Fingerprinting and OS Detection

Service fingerprinting (`-sV`) and OS detection (`-O`) provide the context needed to triage findings and select appropriate NSE scripts. Version detection sends a series of small probes tailored to common protocols and matches responses against a signature database. The accuracy of `-sV` depends on the service’s willingness to respond and on the version database. When `-sV` returns ambiguous results, manual banner grabs or application‑level queries can clarify the version.

**Aggressive version detection example**

```bash
sudo nmap -sV --version-all -p 80,443 192.168.20.10 -oN aggressive_sv.txt
```

**What and why**: `--version-all` tells Nmap to use all available probes for version detection, increasing the chance of an accurate match. Use this in a lab or when you have permission to be more intrusive. On production systems, prefer `--version-light` or default intensity to reduce the risk of triggering IDS or causing service issues.

**OS detection example**

```bash
sudo nmap -O --osscan-guess 192.168.20.10 -oN os_guess.txt
```

**What and why**: `-O` enables OS detection. `--osscan-guess` allows Nmap to make less certain guesses when the match is not exact. Use this option when you need a best‑effort OS classification but document the confidence level in your report.

OS detection is probabilistic and should be corroborated with other evidence. When Nmap reports an OS guess, capture the `tcpdump` output and include the `nmap` fingerprint lines in your report so reviewers can validate the inference.

[[#Table of Contents 📚]]

---

# Nmap Scripting Engine NSE Deep Dive

The Nmap Scripting Engine is the most powerful part of Nmap. NSE scripts are written in Lua and run in phases: prerule, hostrule, portrule, and postrule. Scripts declare categories such as discovery, safe, vuln, intrusive, auth, and brute. Understanding these categories is essential to avoid causing harm.

Start with safe enumeration. The `-sC` option runs a curated set of default scripts that perform non‑intrusive checks such as banner grabs, HTTP title extraction, and basic SSL certificate inspection. After you have service versions from `-sV`, run targeted vulnerability scripts from the `vuln` category. Do not run `intrusive` or `brute` scripts on production systems unless you have explicit permission and a clear stop condition.

**Default and vuln script example**

```bash
sudo nmap -sV -sC --script=vuln -p 22,80,443 192.168.20.10 -oA nse_run
```

**What and why**: `-sC` runs default safe scripts. `--script=vuln` runs scripts in the vulnerability category. Combining `-sV` with `--script=vuln` ensures Nmap has version context to select relevant checks. Save all outputs for correlation. Inspect `--script-help` for any script you plan to run to understand its network actions and required arguments.

You can run specific scripts by name when you want to limit impact. For example, to enumerate SMB shares and users without brute forcing credentials:

```bash
sudo nmap -p445 --script=smb-enum-shares,smb-os-discovery,smb-enum-users 192.168.20.10 -oN smb_enum.txt
```

**What and why**: This command runs three SMB enumeration scripts that gather share lists, user lists, and OS information. These scripts are discovery and enumeration focused; they do not attempt password guessing. Use them to gather evidence of misconfiguration or exposed shares.

**Script tracing and debugging**

```bash
sudo nmap --script-trace --script ./scripts/my-custom.nse -p 8080 192.168.20.10 -oN script_trace.txt
```

**What and why**: `--script-trace` prints the script’s network calls and internal decisions. Use it to map script actions to packets in your capture and to debug custom scripts.

**Writing and customizing NSE scripts** requires knowledge of Lua and the Nmap libraries. A minimal script declares a description, author, categories, a rule that determines when it runs, and an action function that performs the probe and returns results. Test custom scripts in a lab and use `--script-trace` to validate their behavior.

[[#Table of Contents 📚]]

---

# Timing Performance and Stealth Tuning

Timing and rate control are critical. Nmap’s timing templates (`-T0` through `-T5`) provide a simple way to adjust probe aggressiveness. `-T0` is paranoid and extremely slow; `-T3` is the default balanced setting; `-T4` is aggressive and suitable for fast lab scans; `-T5` is very aggressive and likely to trigger IDS or overwhelm devices.

For production scans, use `-T2` or `-T3` and avoid `--min-rate` or `--max-rate` unless you understand the network’s capacity. For UDP scans, increase `--max-retries` and `--host-timeout` because UDP responses are often delayed or filtered.

**Production safe example**

```bash
sudo nmap -sS -p22,80,443 -T2 --max-retries 3 192.168.20.10 -oN safe_scan.txt
```

**What and why**: `-T2` reduces probe rate and increases timeouts to be polite to the target. `--max-retries 3` limits retransmissions. Use this pattern when scanning customer environments or production systems.

When you need speed in a lab, use `-T4` with `--min-rate` and parallelization, but monitor your host’s CPU and network. Nmap can saturate a network if you push too many probes. For very large scans, split the address space into chunks and run parallel processes with different output directories to avoid overwhelming a single Nmap instance.

[[#Table of Contents 📚]]

---

# Evidence Capture and Forensic Discipline

Every scan must be accompanied by packet captures and a reproducible manifest. Packet captures prove the exact probes you sent and the responses you received. Start `tcpdump` or `tshark` before running Nmap and stop it immediately after the scan completes. Use `-w` to write a pcap file and compute a SHA256 checksum for integrity.

**Capture, scan, and checksum example**

```bash
sudo tcpdump -i eth0 host 192.168.20.10 -s 0 -w /tmp/scan_capture.pcap &
sudo nmap -sS -p- 192.168.20.10 -oA scan_full
sudo pkill -f "tcpdump -i eth0 host 192.168.20.10 -s 0 -w /tmp/scan_capture.pcap"
sha256sum /tmp/scan_capture.pcap > /tmp/scan_capture.pcap.sha256
```

**What and why**: The first command starts a background `tcpdump` capturing all packets to and from the target. The `-s 0` option captures full packet payloads. After the scan, kill the capture process and compute a SHA256 checksum to prove the capture’s integrity. Include the pcap and checksum in your evidence package.

**Extract minimal slices for reporting**

```bash
tshark -r /tmp/scan_capture.pcap -Y "tcp.port == 80 and http.request" -T fields -e frame.number -e frame.time_epoch -e http.request.full_uri > http_index.txt
# Suppose the key packet is 456
editcap -r /tmp/scan_capture.pcap /tmp/evidence_slice.pcap 450-460
tshark -r /tmp/evidence_slice.pcap -x > evidence_hex.txt
sha256sum /tmp/evidence_slice.pcap > /tmp/evidence_slice.pcap.sha256
```

**What and why**: The `tshark` command indexes HTTP requests and their packet numbers. `editcap` extracts a small range around the key packet. `tshark -x` produces a hex dump for inclusion in a report. Provide the slice, the hex dump, and the checksum so reviewers can validate the claim without parsing the full capture.

**Manifest principles**: minimality, reproducibility, integrity, and timeline alignment. Include exact commands, packet numbers, timestamps, and SHA256 checksums.

[[#Table of Contents 📚]]

---

# Practical Labs Step by Step

Each lab below is a complete exercise with commands, expected outputs, capture instructions, evidence extraction steps, and troubleshooting notes. Every lab ends with a manifest template and the exact `tshark`/`editcap`/Scapy commands you will use to produce compact evidence packages.

---

## Lab 1 Discovery and Baseline Inventory

### Lab 1 Overview and Goals

The goal is to produce a reliable inventory of live hosts on a /24 and to capture the exact probe/response pairs that prove host liveness. This lab demonstrates ARP discovery on a LAN and mixed IP discovery across routed networks. The deliverable is a small evidence package containing the Nmap outputs, the pcap, an index of ARP/ICMP frames, and a manifest.

### Lab 1 Step by Step

Start a capture on your scanning interface. Capture ARP and ICMP so you can prove which probe elicited the reply.

```bash
sudo tcpdump -i eth0 -s 0 -w /tmp/lab1_discovery.pcap arp or icmp &
```

What and why: `tcpdump` captures ARP and ICMP frames. `-s 0` captures full packets. Save the pcap for evidence.

Run ARP discovery on the LAN.

```bash
sudo nmap -sn -PR 192.168.20.0/24 -oA lab1_discovery
```

What and why: `-sn` performs host discovery only. `-PR` forces ARP discovery. `-oA` saves outputs in multiple formats for automation and reporting.

If you are scanning across routers, run mixed probes.

```bash
sudo nmap -sn -PE -PS22,80,443 -PA3389 10.0.0.0/24 -oA lab1_discovery_mixed
```

What and why: uses ICMP echo and TCP SYN/ACK/ACK probes to discover hosts that block ICMP.

Stop the capture and compute checksums.

```bash
sudo pkill -f "tcpdump -i eth0 -s 0 -w /tmp/lab1_discovery.pcap"
sha256sum /tmp/lab1_discovery.pcap > /tmp/lab1_discovery.pcap.sha256
```

What and why: stop capture and compute SHA256 for integrity.

### Lab 1 Evidence Extraction

Index ARP and ICMP frames to find packet numbers for each live host.

```bash
tshark -r /tmp/lab1_discovery.pcap -Y "arp or icmp" -T fields -e frame.number -e frame.time_epoch -e arp.src.proto_ipv4 -e arp.dst.proto_ipv4 -e ip.src -e ip.dst > lab1_index.txt
```

What and why: produces a CSV‑like index of frames with packet numbers and timestamps. Use these packet numbers to extract minimal slices for each host.

Extract a slice for host 192.168.20.10 (example packet 123):

```bash
editcap -r /tmp/lab1_discovery.pcap /tmp/lab1_host_192.168.20.10_slice.pcap 120-126
tshark -r /tmp/lab1_host_192.168.20.10_slice.pcap -x > lab1_host_192.168.20.10_hex.txt
sha256sum /tmp/lab1_host_192.168.20.10_slice.pcap > /tmp/lab1_host_192.168.20.10_slice.pcap.sha256
```

What and why: extracts a small pcap slice proving the ARP reply or ICMP echo reply that demonstrates host liveness.

### Lab 1 Expected Results

You should have:

- `lab1_discovery.nmap` listing live hosts.
- `lab1_discovery.pcap` full capture.
- `lab1_index.txt` mapping frames to hosts.
- One or more `*_slice.pcap` files with SHA256 checksums.
- A manifest JSON describing commands, packet numbers, timestamps, and checksums.

### Lab 1 Troubleshooting

If no hosts appear, verify your scanner’s interface and IP. Use `ip a` to confirm the interface is on the correct network. If ARP replies are missing, check switch port isolation or VLANs. If ICMP probes fail, try TCP SYN probes to common ports.

[[#Table of Contents 📚]]

---

## Lab 2 Full Port Discovery and Fingerprinting

### Lab 2 Overview and Goals

The goal is to discover all open TCP ports on a target, fingerprint services and OS, and extract the exact probe/response packets that prove each finding. Deliverables: `nmap` outputs, full pcap, minimal pcap slices for key services (HTTP, SSH, etc.), hex dumps, and a manifest.

### Lab 2 Step by Step

Start a capture for the target host.

```bash
sudo tcpdump -i eth0 host 192.168.20.10 -s 0 -w /tmp/lab2_full.pcap &
```

What and why: capture all traffic to/from the target so you can prove the probes and responses.

Run a SYN full port scan to discover open TCP ports.

```bash
sudo nmap -sS -p- -T3 --min-rate 500 192.168.20.10 -oA lab2_ports
```

What and why: `-sS` SYN scan across all ports. `-T3` normal timing. `--min-rate` speeds the scan in labs. `-oA` saves outputs.

Run targeted version detection and OS detection on discovered ports.

```bash
sudo nmap -sV -O --version-intensity 5 -p 22,80,443,3306 192.168.20.10 -oN lab2_fingerprint.txt
```

What and why: `-sV` fingerprints services; `-O` attempts OS detection. Use moderate intensity for production; higher intensity in labs.

Stop the capture and compute checksum.

```bash
sudo pkill -f "tcpdump -i eth0 host 192.168.20.10 -s 0 -w /tmp/lab2_full.pcap"
sha256sum /tmp/lab2_full.pcap > /tmp/lab2_full.pcap.sha256
```

What and why: stop capture and compute SHA256.

### Lab 2 Evidence Extraction and Examples

Index HTTP requests and packet numbers

```bash
tshark -r /tmp/lab2_full.pcap -Y http.request -T fields -e frame.number -e frame.time_epoch -e http.request.full_uri > lab2_http_index.txt
```

What and why: lists HTTP requests with packet numbers so you can extract the exact packet proving a banner or request.

Extract a minimal slice for an HTTP request (example packet 456)

```bash
editcap -r /tmp/lab2_full.pcap /tmp/lab2_http_slice.pcap 450-460
tshark -r /tmp/lab2_http_slice.pcap -x > lab2_http_hex.txt
sha256sum /tmp/lab2_http_slice.pcap > /tmp/lab2_http_slice.pcap.sha256
```

What and why: extracts a small pcap slice and hex dump for reporting.

Extract SSH banner packet numbers and slice

```bash
tshark -r /tmp/lab2_full.pcap -Y "tcp.port == 22 and tcp.flags.syn == 0 and tcp.flags.ack == 1" -T fields -e frame.number -e frame.time_epoch -e tcp.payload > lab2_ssh_index.txt
# slice around the banner packet
editcap -r /tmp/lab2_full.pcap /tmp/lab2_ssh_slice.pcap 200-210
```

What and why: finds the SSH server banner packet and extracts a slice proving the banner.

### Lab 2 Expected Results

You should have:

- `lab2_ports.nmap` listing open ports.
- `lab2_fingerprint.txt` with service versions and OS guess.
- `lab2_full.pcap` full capture and `lab2_http_slice.pcap`, `lab2_ssh_slice.pcap` minimal slices.
- Hex dumps and SHA256 checksums for each slice.
- A manifest mapping commands to packet numbers and timestamps.

### Lab 2 Troubleshooting

If `-sV` returns ambiguous versions, increase `--version-intensity` or run application‑level probes (curl, banner grabs). If OS detection fails, capture the OS probe packets and include them in the manifest so reviewers can validate the inference. If the SYN scan misses ports, try `-sT` to confirm or increase timing.

[[#Table of Contents 📚]]

---

## Lab 3 NSE Vulnerability Verification

### Lab 3 Overview and Goals

The goal is to run targeted NSE scripts to verify vulnerabilities non‑destructively and to produce packet‑level evidence for each finding. The workflow is: fingerprint with `-sV`, select safe `vuln` or specific scripts, capture traffic, run scripts, and extract the probe/response packets that demonstrate the vulnerability or misconfiguration.

### Lab 3 Step by Step

Start a capture for the target and the ports you will test.

```bash
sudo tcpdump -i eth0 host 192.168.20.10 and \(port 22 or port 80 or port 443\) -s 0 -w /tmp/lab3_vuln.pcap &
```

What and why: capture traffic for the ports you will test so you can prove the script probes and responses.

Run `-sV` to get accurate versions.

```bash
sudo nmap -sV -p 22,80,443 192.168.20.10 -oN lab3_sv.txt
```

What and why: version detection provides the context NSE uses to select relevant checks.

Run targeted vulnerability scripts

```bash
sudo nmap -sV --script "vuln and (http* or ssh-vuln*)" -p 22,80,443 192.168.20.10 -oN lab3_vuln.txt
```

What and why: `--script` selects scripts in the `vuln` category that match HTTP or SSH patterns. This limits scope and reduces risk. Inspect `--script-help` for any script before running.

Stop the capture and compute checksum.

```bash
sudo pkill -f "tcpdump -i eth0 host 192.168.20.10"
sha256sum /tmp/lab3_vuln.pcap > /tmp/lab3_vuln.pcap.sha256
```

What and why: stop capture and compute SHA256.

### Lab 3 Evidence Extraction and Examples

For each script output that reports a CVE, find the corresponding probe packet in the pcap. Use `tshark` to search for the probe payload or the script’s unique marker.

Example: find HTTP probe packets that contain a unique path used by the script

```bash
tshark -r /tmp/lab3_vuln.pcap -Y 'http.request.uri contains "nmap"' -T fields -e frame.number -e frame.time_epoch -e http.request.full_uri > lab3_http_probe_index.txt
```

What and why: many NSE scripts include unique URIs or payload markers. Index them to find packet numbers.

Extract the slice and hex dump for the probe and the server response

```bash
# suppose key packets are 512 and 513
editcap -r /tmp/lab3_vuln.pcap /tmp/lab3_vuln_slice.pcap 510-520
tshark -r /tmp/lab3_vuln_slice.pcap -x > lab3_vuln_hex.txt
sha256sum /tmp/lab3_vuln_slice.pcap > /tmp/lab3_vuln_slice.pcap.sha256
```

What and why: extract minimal evidence proving the script’s probe and the server’s vulnerable response.

### Lab 3 Expected Results

You should have:

- `lab3_vuln.txt` with NSE script outputs and CVE references.
- `lab3_vuln.pcap` full capture and `lab3_vuln_slice.pcap` minimal slices for each finding.
- Hex dumps and SHA256 checksums.
- A manifest mapping script outputs to packet numbers and timestamps.

### Lab 3 Safety and Troubleshooting

If a script is marked `intrusive` or `brute`, do not run it on production. If a script times out, run it against a lab instance to debug and use `--script-trace` to see the script’s network calls. If the script reports a CVE but you cannot find the probe in the pcap, re-run the script with `--packet-trace` and capture again.

[[#Table of Contents 📚]]

---

## Lab 4 UDP Enumeration and Reliability

### Lab 4 Overview and Goals

UDP services are often overlooked. This lab focuses on DNS and SNMP enumeration, reliable UDP scanning, and extracting packet‑level proof for UDP responses. The deliverable is a set of validated UDP findings with pcaps and a manifest.

### Lab 4 Step by Step

Start a capture for DNS and SNMP traffic.

```bash
sudo tcpdump -i eth0 host 192.168.20.10 and \(port 53 or port 161\) -s 0 -w /tmp/lab4_udp.pcap &
```

What and why: capture DNS and SNMP traffic to prove queries and responses.

Run a UDP scan with increased retries.

```bash
sudo nmap -sU -p 53,123,161 --max-retries 5 --host-timeout 2m -oN lab4_udp.txt 192.168.20.10
```

What and why: `-sU` runs UDP scan. `--max-retries 5` increases retransmissions to reduce false negatives. `--host-timeout` prevents long hangs.

For DNS, send a valid query to confirm the service.

```bash
dig @192.168.20.10 example.com +short
```

What and why: `dig` sends a proper DNS query and elicits a response that proves the service is functional. Capture the request and response in the pcap.

For SNMP, run a safe read-only query if permitted.

```bash
snmpwalk -v2c -c public 192.168.20.10 system
```

What and why: `snmpwalk` with a public community string may return system info. Use only safe read queries and capture the PDU.

Stop the capture and compute checksum.

```bash
sudo pkill -f "tcpdump -i eth0 host 192.168.20.10 and (port 53 or port 161)"
sha256sum /tmp/lab4_udp.pcap > /tmp/lab4_udp.pcap.sha256
```

What and why: stop capture and compute SHA256.

### Lab 4 Evidence Extraction and Examples

Index DNS queries and responses

```bash
tshark -r /tmp/lab4_udp.pcap -Y dns -T fields -e frame.number -e frame.time_epoch -e dns.qry.name -e dns.a > lab4_dns_index.txt
```

What and why: lists DNS queries and answers with packet numbers for extraction.

Extract a DNS query/response slice

```bash
# suppose key packets are 210 and 211
editcap -r /tmp/lab4_udp.pcap /tmp/lab4_dns_slice.pcap 208-214
tshark -r /tmp/lab4_dns_slice.pcap -x > lab4_dns_hex.txt
sha256sum /tmp/lab4_dns_slice.pcap > /tmp/lab4_dns_slice.pcap.sha256
```

What and why: extracts the minimal evidence proving DNS resolution.

Index SNMP PDUs and extract slice

```bash
tshark -r /tmp/lab4_udp.pcap -Y "snmp" -T fields -e frame.number -e frame.time_epoch -e snmp.community > lab4_snmp_index.txt
# extract around the SNMP response
editcap -r /tmp/lab4_udp.pcap /tmp/lab4_snmp_slice.pcap 300-306
```

What and why: finds SNMP PDUs and extracts a slice proving the SNMP response.

### Lab 4 Expected Results

You should have:

- `lab4_udp.txt` with Nmap UDP scan results.
- `lab4_udp.pcap` full capture and `lab4_dns_slice.pcap`, `lab4_snmp_slice.pcap` minimal slices.
- Hex dumps and SHA256 checksums.
- A manifest mapping commands to packet numbers and timestamps.

### Lab 4 Troubleshooting

If UDP scans return many `open|filtered`, increase retries and timeouts. If `dig` returns no answer but Nmap shows open|filtered, try sending a specific query type (A, TXT) or use `tcpdump` to confirm whether the probe left your host. For SNMP, if community strings are unknown, do not brute force; document the `open|filtered` result and recommend defensive controls.

[[#Table of Contents 📚]]

---

## Lab 5 Custom NSE Script Trace and Validation

### Lab 5 Overview and Goals

When you write or adapt an NSE script, you must prove exactly what the script did on the wire. This lab shows how to run a custom script with `--script-trace`, capture traffic, correlate script actions to packets, and produce a compact evidence package that includes the script source, `--script-trace` output, and pcap slices.

### Lab 5 Step by Step

Place your custom script in the local scripts directory (or reference it directly).

Start a capture for the target port.

```bash
sudo tcpdump -i eth0 host 192.168.20.10 and port 8080 -s 0 -w /tmp/lab5_custom.pcap &
```

What and why: capture the script’s network actions.

Run the script with trace enabled

```bash
sudo nmap -sV --script ./scripts/my-custom.nse --script-trace -p 8080 192.168.20.10 -oN lab5_custom.txt
```

What and why: `--script-trace` prints the script’s network calls and internal decisions. This output is essential to map script actions to packets.

Stop the capture and compute checksum.

```bash
sudo pkill -f "tcpdump -i eth0 host 192.168.20.10 and port 8080"
sha256sum /tmp/lab5_custom.pcap > /tmp/lab5_custom.pcap.sha256
```

What and why: stop capture and compute SHA256.

### Lab 5 Evidence Extraction and Examples

Use the `--script-trace` output to find the exact probe payloads and then search the pcap for those payloads.

Example: script trace shows it sent `GET /nse-test?probe=ABC123`. Find the packet

```bash
tshark -r /tmp/lab5_custom.pcap -Y 'http.request.uri contains "nse-test?probe=ABC123"' -T fields -e frame.number -e frame.time_epoch -e http.request.full_uri > lab5_probe_index.txt
```

Extract the slice and include the script source and trace output in the package

```bash
editcap -r /tmp/lab5_custom.pcap /tmp/lab5_custom_slice.pcap 120-128
tshark -r /tmp/lab5_custom_slice.pcap -x > lab5_custom_hex.txt
sha256sum /tmp/lab5_custom_slice.pcap > /tmp/lab5_custom_slice.pcap.sha256
```

What and why: the script source and `--script-trace` output show intent and network calls; the pcap slice proves the actual packets on the wire.

### Lab 5 Expected Results

You should have:

- `lab5_custom.pcap` full capture and `lab5_custom_slice.pcap` minimal slice.
- `lab5_custom.txt` with `--script-trace` output.
- The custom script source file.
- Hex dumps and SHA256 checksums.
- A manifest mapping script actions to packet numbers and timestamps.

### Lab 5 Troubleshooting

If the script trace shows no network calls, ensure the `portrule` or `hostrule` matches the target. If the pcap does not contain the probe, verify you captured the correct interface and that the script did not use a different source IP or port. Use `--packet-trace` in Nmap to get additional probe details.

[[#Table of Contents 📚]]

---

# Detection Defensive Signals and Evasion Considerations

This section explains what defenders see when you scan and how to make your findings defensible and ethical.

Defenders see network IDS/IPS alerts for SYN floods, port sweeps, and known Nmap signatures. Host logs record connection attempts and failed handshakes. Firewall logs show blocked packets and rate limiting events. When you scan, collect IDS alerts, firewall logs, and host logs with timestamps to correlate with your pcaps.

Evasion techniques such as fragmentation, decoys, and spoofing exist but are ethically risky and often illegal. Use stealth techniques only in authorized red team engagements and document them thoroughly. For reporting, include exact `nmap` commands, the matching pcap slice, and a manifest so defenders can reproduce and validate your findings.

[[#Table of Contents 📚]]

---

# Common Problems and Troubleshooting

If Nmap finds no hosts, verify your scanner’s interface and routing. Use ARP discovery on LANs. If SYN scans return no open ports but Connect scans do, you may lack raw socket privileges or the target may treat half‑open connections differently; run as root or use `-sT`. UDP scans often return `open|filtered`; increase `--max-retries` and follow up with protocol‑aware probes. If NSE scripts time out, run `--script-trace` and test the script in a lab. If Nmap is slow or consumes too much CPU, reduce timing (`-T2`) or split the address space.

When correlating Nmap output with pcaps, use `--packet-trace` and `--reason` to get probe details in the Nmap output. Search the pcap for the probe payload or the source port used by Nmap. Use `tshark` to index frames by time and port to find the exact packets to extract.

[[#Table of Contents 📚]]

---

# Reporting and Packaging Findings

A high‑quality report contains a narrative, reproducible steps, and a compact evidence package. The narrative explains the hypothesis and impact. The reproducible steps list exact commands and stop conditions. The evidence package contains minimal pcap slices, small extracted artifacts, `nmap` outputs, and a JSON manifest.

**Example manifest template**

```json
{
  "test_name": "Nmap SYN discovery and HTTP fingerprint",
  "hypothesis": "Host 192.168.20.10 responds to SYN probes and serves HTTP with a version header",
  "commands": [
    "sudo tcpdump -i eth0 host 192.168.20.10 -s 0 -w /tmp/scan_capture.pcap &",
    "sudo nmap -sS -p- 192.168.20.10 -oA scan_full",
    "sudo nmap -sV -O 192.168.20.10 -oN lab2_fingerprint.txt"
  ],
  "pcap_files": [
    {"file":"scan_capture.pcap","packets":[450,451,452],"timestamps":["1654320007.789"]}
  ],
  "evidence_files": [
    {"file":"lab2_http_slice.pcap","sha256":"<sha256sum>","packets":[450]}
  ],
  "confidence":"high",
  "remediation":"Remove version header or configure server to not disclose version"
}
```

Include remediation suggestions tied to the evidence and prioritize findings by exploitability and impact. Provide remediation steps that map directly to the evidence (for example, remove version headers, restrict management access, patch vulnerable services).

[[#Table of Contents 📚]]

---

# Appendix Commands Examples and Templates

Install Nmap on Debian or Ubuntu

```bash
sudo apt update && sudo apt install -y nmap
```

What and why: installs the Nmap package and dependencies. Use the distribution package for convenience; for the latest features consider building from source.

Essential Nmap commands with explanations

```bash
# discovery with ARP on LAN
sudo nmap -sn -PR 192.168.20.0/24 -oA discovery_arp
# SYN full port scan (lab)
sudo nmap -sS -p- --min-rate 500 192.168.20.10 -oA syn_full
# version detection and default scripts
sudo nmap -sV -sC -p 22,80,443 192.168.20.10 -oA sv_default
# targeted vuln scripts after fingerprinting
sudo nmap -sV --script=vuln -p 80,443 192.168.20.10 -oN vuln_scan
```

Packet capture and extraction examples

```bash
# capture before scan
sudo tcpdump -i eth0 host 192.168.20.10 -s 0 -w /tmp/scan_capture.pcap &
# index HTTP requests
tshark -r /tmp/scan_capture.pcap -Y http.request -T fields -e http.request.full_uri -e frame.number -e frame.time_epoch > http_index.txt
# extract packet slice
editcap -r /tmp/scan_capture.pcap /tmp/evidence_slice.pcap 120-130
# hex dump and checksum
tshark -r /tmp/evidence_slice.pcap -x > /tmp/evidence_hex.txt
sha256sum /tmp/evidence_slice.pcap > /tmp/evidence_slice.pcap.sha256
```

Scapy extraction example

```python
from scapy.all import rdpcap, TCP, Raw
pcap = rdpcap('/tmp/evidence_slice.pcap')
for i,pkt in enumerate(pcap, start=1):
    if pkt.haslayer(TCP) and pkt.haslayer(Raw):
        payload = bytes(pkt[Raw].load)
        if b'HTTP' in payload:
            with open(f'http_payload_{i}.bin','wb') as fh:
                fh.write(payload)
```

What and why: extracts HTTP payloads for precise byte‑level evidence.

---

This merged guide contains the full Nmap installation and configuration instructions plus deep, forensic‑grade walkthroughs for discovery, enumeration, NSE usage, capture and evidence workflows, and performance tuning. It preserves every step, command, and rationale from the previous sections and expands the practical, forensic, and troubleshooting detail so you can build, test, and prove network behaviors reliably.

If you want, I will convert any single lab into a runnable automation script that starts captures, runs scans, extracts slices, computes checksums, and generates a ready `manifest.json` for reporting. Which lab should I automate first: Lab 2 Full Port and Fingerprint, Lab 3 NSE Vulnerability Verification, or Lab 4 UDP Enumeration?