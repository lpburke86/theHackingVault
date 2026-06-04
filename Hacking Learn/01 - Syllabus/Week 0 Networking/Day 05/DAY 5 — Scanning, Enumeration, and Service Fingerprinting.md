### Table of Contents 📚

[[#I. Why Enumeration Is the Beating Heart of Hacking 🔥]]  
[[#II. The Philosophy of Enumeration 🧭]]  
[[#III. nmap — The First Conversation You Ever Have With a Machine 🕵️]]  
[[#IV. Scan Types The Different Ways to Knock on a Door 🚪]]
[[#V. NSE The nmap Scripting Engine 🛠️]]
[[#VI. What Enumeration Looks Like in the Real World 🧾]]
[[#VII. LAB Scanning, Enumeration, and Fingerprinting 🧪]]
[[#VIII. Interpreting Results and Building an Attack Surface Map 🗺️]]
[[#IX. Operational Tradeoffs Timing, Stealth, and Ethics ⚖️]]
[[#X. End of Day Outcomes 🎯]]
[[#Appendix Useful Commands and Parsing Examples 🧾]]

---

# 🧨 DAY 5 — Scanning, Enumeration, and Service Fingerprinting

_2 hours_

_“Today you learn how to make machines talk.”_

This day teaches you how to move from passive observation to active interrogation. Where Day 4 taught you to read the network, Day 5 teaches you to ask precise questions and interpret the answers. Scanning and enumeration are not mindless noise; they are structured conversations with predictable grammar and signals. The goal is to extract reliable facts about a target: which ports accept connections, which services answer, which versions are running, which protocols are misconfigured, and which behaviors reveal weakness. Every command you run must have a hypothesis and an expected signal. You will learn how to craft those hypotheses, run the probes, and convert responses into a reproducible profile that becomes the foundation of any later exploit or report.

---

## I. Why Enumeration Is the Beating Heart of Hacking 🔥

### Summary and purpose

Enumeration answers the single question that precedes every exploit: what am I dealing with. Without a precise inventory of services, versions, and reachable interfaces you cannot choose an exploit, craft a payload, or prove impact. Enumeration transforms an opaque target into a map of reachable services and likely vulnerabilities. The map is not a list; it is a structured dataset that includes service, version, protocol quirks, timing characteristics, and confidence scores. When you enumerate correctly you reduce uncertainty: you know which doors are locked, which doors are open, which doors creak when you push them, and which doors will trigger alarms. This knowledge is the difference between wandering and navigating, between guessing and knowing.

### Forensic mindset to adopt

Treat enumeration as evidence collection. Every finding must be reproducible and tied to raw artifacts: the exact command used, the raw output file, the pcap snippet, and packet numbers with timestamps. Your deliverable is not “open ports” but a defensible claim: port 22 responded with an OpenSSH banner at packet 12345 in `scan_capture.pcap` at 2026‑06‑04T04:00:12Z. That level of precision is what turns reconnaissance into intelligence.

---

## II. The Philosophy of Enumeration 🧭

Enumeration is disciplined interrogation. It begins with a question and ends with evidence. The question might be simple: “What ports are open?” or complex: “Which web application frameworks and middleware are present and how do they handle malformed input?” Tools are only instruments; the real skill is designing the probe. A good probe is minimal, targeted, and interpretable. Minimal means it sends the smallest possible stimulus that will elicit the signal you need. Targeted means it focuses on a single hypothesis. Interpretable means you know what a positive and negative response look like and how to measure confidence. Enumeration is iterative: run a probe, observe the response, refine the probe, and repeat until the hypothesis space is exhausted or you have sufficient evidence to proceed.

---

## III. nmap — The First Conversation You Ever Have With a Machine 🕵️

nmap is the canonical interrogator because it implements many probe types and interprets responses into human‑readable findings. Think of nmap as a negotiator that speaks TCP, UDP, and protocol‑specific dialects. It sends SYNs to see whether a port will accept a connection, it completes handshakes when necessary, it sends malformed or protocol‑specific payloads to elicit banners and quirks, and it runs scripts to exercise higher‑level behaviors. The output of nmap is not the final answer; it is a set of observations with confidence levels. Each line in an nmap report is a claim: port X is open; service Y responded with banner Z; version detection suggests product P at version V with confidence C. Your job is to treat those claims as hypotheses to be validated with follow‑up probes or cross‑correlation with other data sources such as banner grabs, service banners captured in Wireshark, or application responses.

---

## IV. Scan Types The Different Ways to Knock on a Door 🚪

SYN scan (`-sS`) is the classic stealth probe. It sends a SYN and waits for SYN/ACK; when it receives SYN/ACK it sends RST to avoid completing the handshake. The signal you look for is the SYN/ACK response; the absence of SYN/ACK combined with RST or ICMP unreachable indicates closed or filtered ports. SYN scans are fast and leave minimal state on the target, but they are still visible to network monitoring and can trigger IDS signatures that look for many SYNs in a short window.

Connect scan (`-sT`) completes the full TCP handshake by letting the OS perform the connect call. Use this when raw sockets are unavailable or when you want to ensure the service logs the connection. The signal is a completed handshake and any service banner that appears after the handshake. Connect scans are loud: they create full sessions and are more likely to be logged by the target.

UDP scan (`-sU`) is slow and unreliable because UDP is connectionless and many services respond only on error or not at all. The probe sends UDP datagrams to target ports and interprets ICMP port unreachable messages as closed ports; lack of response is ambiguous and often treated as open|filtered. UDP is where DNS, SNMP, NTP, and other critical services live, and careful UDP enumeration is essential because many vulnerabilities and misconfigurations exist in UDP services.

Version detection (`-sV`) is the fingerprinting phase. nmap sends protocol‑specific probes and malformed inputs to elicit banners, quirks, and timing differences. The responses are matched against a signature database to produce a version guess and a confidence score. Version detection is not infallible: some services intentionally obfuscate banners, and some middleboxes alter responses. Treat `-sV` output as a high‑value hypothesis that requires validation.

OS detection (`-O`) is a personality test that examines low‑level TCP/IP stack behavior: TTL, window size, TCP options ordering, and ICMP responses. Each OS and kernel version has characteristic fingerprints. OS detection is probabilistic; the output includes a confidence metric and should be cross‑checked with service banners and other telemetry.

---

## V. NSE The nmap Scripting Engine 🛠️

NSE turns nmap from a scanner into a framework. Scripts can enumerate SMB shares, query LDAP, brute force FTP, test SSL ciphers, enumerate DNS records, and check for specific vulnerabilities such as Heartbleed or Log4Shell. Each script is a small program that performs a targeted probe and returns structured output. Use NSE when you need protocol‑aware enumeration beyond simple banner grabs. Scripts can be combined and tuned with arguments to control timing, credentials, and output format. Because NSE can perform intrusive checks, treat scripts as higher‑impact probes and run them only when you understand the potential side effects and have authorization.

---

## VI. What Enumeration Looks Like in the Real World 🧾

A real enumeration profile is more than a table of ports. It is a narrative that ties ports to services, services to versions, versions to known issues, and timing/behavioral signals to confidence. For example, a profile that shows SSH on port 22 with an OpenSSH 7.6p1 banner, HTTP on port 80 with nginx 1.14.0, and MySQL on 3306 with MySQL 5.7.33 suggests a particular attack surface: SSH may allow user enumeration or weak ciphers, nginx may have path traversal or misconfigured proxy rules, and MySQL exposure suggests credential harvesting or SQL injection pivoting. Each observation must be accompanied by the evidence: the exact banner bytes, the packet numbers or pcap snippet, the nmap command used, and the confidence level. This evidence is what turns an observation into actionable intelligence.

---

## VII. LAB Scanning, Enumeration, and Fingerprinting 🧪

The lab section is hands‑on and forensic. Each exercise below is expanded with a full procedure, capture discipline, expected signals, how to extract evidence, and an explicit “Expected results” paragraph that states what you should observe if the target behaves as described.

---

### Lab 1 — Basic Discovery: full procedure, evidence, interpretation, and expected results ✅

Procedure and purpose. Run a default nmap scan to discover which ports respond and to collect the first set of claims you will validate. The command below probes the most common ports and writes three output formats so you have human‑readable text, XML for programmatic parsing, and grepable output for quick grepping.

```bash
nmap -oA day5_basic 192.168.20.10
```

Why you run it first. nmap’s default scan is a low‑effort reconnaissance pass that gives you a quick inventory. It is not exhaustive, but it produces the initial hypotheses you will validate with deeper probes. Save the `day5_basic.nmap`, `day5_basic.xml`, and `day5_basic.gnmap` files because they are the canonical record of this pass.

Capture discipline. Start a packet capture on the scanning interface before launching nmap so you can correlate each reported open port with the actual packets on the wire. Use tcpdump to capture everything and write to a pcap.

```bash
sudo tcpdump -i eth0 -w /tmp/day5_basic_capture.pcap
```

How to validate and extract evidence. Stop tcpdump after the scan completes. Open the pcap in Wireshark and locate the first SYN packet to each reported open port. Note the packet number and timestamp for each SYN and for the corresponding SYN/ACK (if present). These packet numbers are the primary evidence you will attach to any claim that a port is open. For each open port, extract the first few bytes of any banner or pre‑handshake data from the pcap and save them as a small evidence file (for example `evidence_port22_pkt123.bin`) so you can include raw bytes in a report.

How to extract banner bytes from the pcap using tshark. Identify the packet number of the first non‑SYN packet in the session and extract its payload.

```bash
tshark -r /tmp/day5_basic_capture.pcap -Y "frame.number == 123" -x
```

Record the nmap command, the pcap filename, and the packet numbers that support each claim.

Expected results. If the target responds normally, nmap will list several open ports with service names. In the pcap you will find SYN → SYN/ACK exchanges for open ports and RST or ICMP unreachable for closed/filtered ports. For each open port you should be able to extract at least one packet containing banner bytes or application data that corroborates the nmap claim. If the target is heavily filtered, nmap may report many ports as filtered and the pcap will show missing replies or ICMP messages.

---

### Lab 2 — SYN Scan: timing, stealth, correlating SYN/ACKs, and expected results ⏱️

Procedure and purpose. Run a SYN scan to enumerate open TCP ports quickly while minimizing completed connections. The command below scans all ports, uses a faster timing template, and writes output for later parsing.

```bash
sudo nmap -sS -Pn -p- -T4 -oA day5_syn 192.168.20.10
```

Why use SYN scans. SYN scans send a single SYN and interpret a SYN/ACK as an open port, then send RST to avoid completing the handshake. This reduces the amount of state left on the target and is faster than connect scans. The tradeoff is that SYN scans are still visible to network monitoring and can trigger IDS rules that look for many SYNs in a short interval.

Capture and analysis. Start tcpdump before running nmap so you capture every SYN and every response.

```bash
sudo tcpdump -i eth0 -w /tmp/day5_syn_capture.pcap
```

Open the pcap in Wireshark and apply a display filter to isolate SYN probes.

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

What to look for and how to quantify. A SYN from your scanner to a destination port followed by a SYN/ACK from the target indicates an open port. If you see RST or ICMP unreachable, the port is closed or filtered. If you see no reply, the port is likely filtered or the probe was dropped. For each SYN/ACK you find, note the packet number of the SYN and the SYN/ACK and extract any subsequent bytes that the service sends before the RST (some services send a banner immediately after SYN/ACK). Save those bytes as evidence.

Programmatic quantification. Use tshark to list destination ports that responded with SYN/ACK and count occurrences.

```bash
tshark -r /tmp/day5_syn_capture.pcap -Y 'tcp.flags.syn==1 && tcp.flags.ack==1' -T fields -e tcp.dstport | sort -n | uniq -c
```

Handling noisy networks. If you see SYN/ACKs from multiple MAC addresses or unexpected source ports, cross‑check ARP and routing to ensure you are not seeing responses from a middlebox or a load balancer. Record anomalies and include them in your confidence assessment.

Expected results. A successful SYN scan will produce a concise list of open ports in the nmap output and corresponding SYN/SYN‑ACK pairs in the pcap. You should be able to extract at least one SYN/ACK packet per open port and, where present, any immediate banner bytes. If the target employs rate limiting or IDS, you may observe intermittent SYN/ACKs or RSTs; document this variability as it affects exploitability.

---

### Lab 3 — Version Detection: probes, intensity, banner validation, and expected results 🔎

Procedure and purpose. Run version detection to convert open port claims into service and version hypotheses. The command below targets specific ports and increases probe intensity to elicit richer responses.

```bash
sudo nmap -sV --version-intensity 5 -p 22,80,3306 -oN day5_version 192.168.20.10
```

Why version detection matters. `-sV` sends protocol‑aware probes and malformed inputs to elicit banners, quirks, and timing differences that nmap matches against its signature database. The result is a version guess with a confidence score. Treat this output as a high‑value hypothesis that you must validate.

Validation with direct probes. For SSH, use `nc` or `ssh -vv` to capture the banner. For HTTP, use `curl -I` to fetch headers and `curl` to fetch the body. For MySQL, use `nc` to connect and read the initial handshake bytes. Always capture these interactions in a pcap so you can show the exact bytes.

SSH banner grab example:

```bash
nc 192.168.20.10 22
# expect something like: SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3
```

HTTP banner validation example:

```bash
curl -I http://192.168.20.10/
```

MySQL handshake grab example:

```bash
printf '\n' | nc 192.168.20.10 3306 | head -c 200 | hexdump -C
```

Extracting banner bytes from pcap. Find the packet where the server first sends non‑SYN data and extract the payload with tshark.

```bash
tshark -r /tmp/day5_syn_capture.pcap -Y "ip.dst == 192.168.20.10 && tcp.dstport == 22 && frame.number == 456" -x
```

Handling obfuscated banners. If banners are obfuscated, use protocol‑level interactions and correlate TCP/IP stack fingerprints from the pcap to build a composite confidence score.

Expected results. `-sV` should return service names and version guesses with confidence levels. Direct probes should reproduce the same banner bytes visible in the pcap. If nmap reports a version but your direct probe returns a different banner, document both artifacts and lower confidence until reconciled. If the service intentionally hides its banner, you will need to rely on protocol behavior and stack fingerprints to increase confidence.

---

### Lab 4 — OS Detection: fingerprints, TTLs, cross‑validation, and expected results 🖥️

Procedure and purpose. Run OS detection to get a probabilistic guess of the target’s operating system.

```bash
sudo nmap -O -v -oN day5_os 192.168.20.10
```

What OS detection measures. nmap examines low‑level TCP/IP stack behaviors such as initial TTL values, TCP window sizes, TCP options ordering, and ICMP responses. These characteristics vary by OS and kernel version, producing a fingerprint that nmap matches against its database. Because network devices, NAT, and middleboxes can alter these signals, OS detection is probabilistic.

Validating OS guesses with pcap evidence. Open the pcap captured during your scans and measure the TTL values and TCP window sizes of packets originating from the target.

```bash
tshark -r /tmp/day5_syn_capture.pcap -Y "ip.src == 192.168.20.10" -T fields -e ip.ttl -e tcp.window_size_value
```

Also inspect TCP options ordering and presence of options such as SACK, Timestamps, and Window Scale. These details strengthen or weaken nmap’s OS hypothesis.

Reconciling conflicting signals. If nmap reports Linux but you observe TTLs of 128 and Windows‑style TCP options, consider NAT, a load balancer, or a proxy that rewrites packets. Cross‑validate with service banners and application behavior. If ambiguity remains, record conflicting evidence and lower your confidence score.

Extracting TCP option ordering for reporting. Find a SYN or SYN/ACK packet from the target and print the TCP options.

```bash
tshark -r /tmp/day5_syn_capture.pcap -Y "tcp.flags.syn==1 && ip.src==192.168.20.10" -T fields -e tcp.options
```

Expected results. A successful OS detection will produce a best‑guess OS with a confidence percentage. In the pcap you should observe TTLs and TCP option patterns consistent with that guess. If the environment includes NAT or proxies, expect lower confidence and conflicting signals; document these conflicts and include both the nmap output and the pcap evidence.

---

### Lab 5 — Full Aggressive Scan: scope, impact, evidence hygiene, and expected results ⚠️

Procedure and purpose. Run an aggressive scan only in lab environments or with explicit authorization. The `-A` option bundles OS detection, version detection, NSE scripts, and traceroute into a single high‑impact pass.

```bash
sudo nmap -A -T4 -oA day5_full 192.168.20.10
```

Why this scan is high impact. `-A` performs many probes, some of which are intrusive or may trigger application logic. Use it when you need a complete profile quickly and you have permission. Always capture the network traffic during this scan and be prepared to stop it if you observe adverse effects.

Parsing and triage. After the scan completes, parse the XML output to extract open ports, service versions, and script results. Use the Python XML snippet below to extract ports and script outputs for triage.

```python
import xml.etree.ElementTree as ET
tree = ET.parse('day5_full.xml')
root = tree.getroot()
for host in root.findall('host'):
    addr = host.find('.//address').get('addr')
    for port in host.findall('.//port'):
        portid = port.get('portid')
        state = port.find('state').get('state')
        service = port.find('service').get('name') if port.find('service') is not None else 'unknown'
        scripts = [s.get('id') + ':' + (s.text or '') for s in port.findall('.//script')]
        print(addr, portid, state, service, scripts)
```

Validating script results. For each script result that indicates a potential vulnerability, extract the exact request and response bytes from the pcap and save them as evidence. Record the script name, the arguments used, and the raw script output file produced by nmap. If a script performed an authenticated check or used a wordlist, document the inputs so the test is reproducible.

Risk management. Aggressive scans produce many findings; prioritize them by exploitability and impact. For each high‑impact finding, validate with a targeted, minimal probe that reproduces the behavior without the full `-A` noise. For example, if a script reports a vulnerable SSL cipher, run a focused `openssl s_client` test to confirm.

```bash
openssl s_client -connect 192.168.20.10:443 -cipher 'ALL' -servername 192.168.20.10
```

Expected results. `-A` should produce a comprehensive report including open ports, service versions, script outputs, and traceroute hops. The pcap will contain the raw probes and responses used to generate those results. Expect some false positives from scripts; each high‑impact finding should be validated with a focused probe and pcap evidence. If the target is fragile, you may observe application errors or service restarts—stop the scan and document the observed impact.

---

### Lab 6 — NSE Targeted Enumeration: scripts, arguments, safe validation, and expected results 🧰

Procedure and purpose. Use NSE scripts when you need protocol‑aware enumeration beyond banner grabs. Scripts are small programs that perform targeted probes and return structured output. The examples below show common, low‑impact scripts and how to validate their output.

HTTP title script.

```bash
nmap --script http-title -p 80 -oN day5_http_title 192.168.20.10
```

Why it helps. `http-title` returns the HTML `<title>` content and sometimes server headers that help identify web applications. Validate the script output by performing a direct HTTP request and capturing the response body.

```bash
curl -sS http://192.168.20.10/ | sed -n '1,200p'
```

DNS brute force.

```bash
nmap --script dns-brute -p 53 -oN day5_dns_brute 192.168.20.10
```

Why it helps. `dns-brute` attempts to resolve common hostnames and can reveal internal naming conventions or forgotten hosts. Validate discovered hostnames by performing direct DNS queries and capturing the DNS responses in the pcap.

SMB OS discovery.

```bash
nmap --script smb-os-discovery -p 445 -oN day5_smb 192.168.20.10
```

Why it helps. SMB scripts can reveal OS, domain membership, and share lists. Because SMB scripts can be intrusive, validate findings by connecting with `smbclient` and capturing the SMB handshake and responses.

Script arguments and credentials. Many NSE scripts accept `--script-args` to pass credentials, timeouts, or wordlists. When you use credentials, treat the probe as authenticated testing and record the credentials used, the script arguments, and the exact script output. Because scripts can be destructive or trigger alarms, always run them in a controlled environment first and document expected side effects.

Extracting structured script output. nmap’s XML output includes `<script>` elements with `id` and `output`. Parse these elements and attach the raw script output files to your report so reviewers can see the exact probe and response.

Expected results. Each script should return structured output relevant to the protocol: page titles for HTTP, discovered hostnames for DNS, and OS/share information for SMB. Validate each script finding with a direct protocol client and pcap evidence. If a script returns nothing, it may indicate the service is not present, is protected, or the script requires credentials—document the negative result and the reasoned hypothesis.

---

### Lab 7 — Capture Scans in Wireshark: fingerprinting behavior, extracting artifacts, and expected results 🕵️‍♂️

Procedure and purpose. While running any of the scans above, capture the traffic with tcpdump so you can analyze the scanner’s probes and the target’s responses in Wireshark. Start the capture before launching nmap and stop it after the scan completes.

```bash
sudo tcpdump -i eth0 -w /tmp/scan_capture.pcap
```

Isolate SYN probes in Wireshark.

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

What to inspect and why. Examine the timing between SYNs, the source port patterns used by the scanner, and the TTL and TCP option patterns in responses. Timing patterns reveal the scanner’s aggressiveness and whether the target is rate limiting. Source port patterns can indicate whether the scanner uses ephemeral ports or fixed source ports; some IDS signatures rely on unusual source port behavior. TCP option ordering and window sizes in responses are useful for OS fingerprinting and for detecting middleboxes that rewrite packets.

Extracting banner bytes during scanning. Some services send banners immediately after SYN/ACK or during handshake. Find the first packet in the session that contains application data and extract its payload with tshark.

```bash
tshark -r /tmp/scan_capture.pcap -Y "tcp.stream == 12 && tcp.len > 0" -T fields -e data.text
```

Detecting fingerprinting behavior from the scanner. nmap’s version detection and NSE scripts often send protocol‑specific probes that are visible in the pcap. Look for unusual payloads, malformed packets, or repeated application‑level requests that do not match normal client behavior. These probes are the raw material nmap uses to fingerprint services; capturing them allows you to reproduce the fingerprinting logic and to demonstrate exactly what was sent to elicit a given response.

Advanced programmatic extraction. Use Scapy to iterate the pcap and extract streams that contain HTTP requests, SSH banners, or other protocol signatures, and write each stream to a separate evidence file with metadata (packet numbers, timestamps, byte offsets).

```python
from scapy.all import rdpcap, TCP, Raw, IP
pcap = rdpcap('/tmp/scan_capture.pcap')
streams = {}
for pkt in pcap:
    if pkt.haslayer(TCP) and pkt.haslayer(Raw) and pkt.haslayer(IP):
        key = (pkt[IP].src, pkt[IP].dst, pkt[TCP].sport, pkt[TCP].dport)
        streams.setdefault(key, []).append((pkt.time, bytes(pkt[Raw].load)))
for k, frames in streams.items():
    src, dst, sport, dport = k
    filename = f"evidence_{src}_{dst}_{sport}_{dport}.bin"
    with open(filename, 'wb') as fh:
        for ts, data in frames:
            fh.write(data)
```

Expected results. The pcap will contain the raw probes and responses used by nmap. You should be able to identify the scanner’s SYN timing, source port behavior, and any protocol‑specific probes sent by `-sV` or NSE scripts. For each finding in your nmap output, you should be able to point to the exact packets in the pcap that produced the claim and extract the raw bytes for reporting.

---

## VIII. Interpreting Results and Building an Attack Surface Map 🗺️

Interpreting enumeration results is a process of evidence correlation and hypothesis refinement. Start by grouping findings by service and by exposure. For each open port, record the service name, version guess, evidence source (nmap output, banner grab, pcap packet numbers), and confidence level. Then map each service to potential attack paths: for example, an exposed MySQL instance suggests credential harvesting and lateral movement; an outdated nginx suggests known CVEs and path traversal; an SSH service with weak ciphers suggests possible downgrade or brute force attacks. Prioritize findings by exploitability and impact: a high‑confidence, internet‑exposed service with known remote code execution is higher priority than a low‑confidence internal service. Always include the raw evidence: the exact banner bytes, the nmap command used, the pcap packet numbers, and any script output. This reproducible evidence is what allows others to validate your findings and for you to escalate to exploitation or remediation.

---

## IX. Operational Tradeoffs Timing, Stealth, and Ethics ⚖️

Every scan has operational tradeoffs. Faster timing templates (`-T4`, `-T5`) complete scans quickly but increase noise and the chance of detection. Slower templates (`-T1`, `-T2`) are stealthier but take longer and may be impractical for large ranges. Evasion techniques such as source port spoofing, decoy hosts, or fragmented packets can bypass naive defenses but also increase complexity and the risk of false positives. Ethical constraints are paramount: run intrusive scans only in lab environments or with explicit authorization. Maintain evidence hygiene: capture only what you need, redact sensitive data in reports, and store pcaps securely. When working in a client environment, coordinate with defenders and provide clear scope and rules of engagement. The goal of enumeration is to produce reliable, reproducible evidence, not to create noise or cause outages.

---

## X. End of Day Outcomes 🎯

By the end of Day 5 you will be able to design and execute targeted enumeration probes, interpret nmap and NSE outputs as evidence rather than raw text, cross‑validate findings with packet captures, and build a prioritized attack surface map that ties each observation to raw evidence. You will understand the tradeoffs between speed and stealth, know how to extract exact banner bytes and packet offsets for reporting, and be able to justify each probe you run with a hypothesis and an expected signal. You will no longer treat scanning as noise; you will treat it as disciplined interrogation that yields reproducible intelligence.

---

## Appendix Useful Commands and Parsing Examples 🧾

Capture scans to a pcap for correlation. The command below captures traffic on `eth0` while you run your scans and writes to `/tmp/scan_capture.pcap`. Use `sudo` to access the interface.

```bash
sudo tcpdump -i eth0 -w /tmp/scan_capture.pcap
```

Run a stealthy SYN scan across all ports and save output in multiple formats for later parsing.

```bash
sudo nmap -sS -Pn -p- -T4 -oA day5_syn 192.168.20.10
```

Run version detection on specific ports and save a human‑readable output.

```bash
sudo nmap -sV --version-intensity 5 -p 22,80,3306 -oN day5_version 192.168.20.10
```

Run NSE scripts for targeted enumeration. The example below grabs the HTTP title which is useful for identifying web applications quickly.

```bash
nmap --script http-title -p 80 -oN day5_http_title 192.168.20.10
```

Extract URIs and script outputs programmatically using `tshark` and XML parsing. The `tshark` command below extracts HTTP URIs from a pcap for triage.

```bash
tshark -r /tmp/scan_capture.pcap -Y http.request -T fields -e http.request.full_uri
```

Parse nmap XML to build a CSV of open ports and services for an attack surface map. The Python snippet below reads `day5_full.xml` and prints port, state, and service for each host.

```python
import xml.etree.ElementTree as ET
tree = ET.parse('day5_full.xml')
root = tree.getroot()
with open('day5_ports.csv','w') as out:
    out.write('host,port,state,service\n')
    for host in root.findall('host'):
        addr = host.find('.//address').get('addr')
        for port in host.findall('.//port'):
            portid = port.get('portid')
            state = port.find('state').get('state')
            svc = port.find('service').get('name') if port.find('service') is not None else 'unknown'
            out.write(f'{addr},{portid},{state},{svc}\n')
```

Evidence hygiene reminder. For every finding include the command used, the raw output file, the pcap packet numbers, and a short interpretation that ties the observation to a hypothesis about impact. This reproducible trail is what makes enumeration useful and defensible.

---

If you want, I will convert this into a repo‑ready Markdown file with sanitized filename and optional YAML frontmatter, or expand any single lab exercise into a full forensic walkthrough that includes exact expected packet bytes and the precise `tshark`/Scapy commands to extract them. Which would you like next: repo file or a deep forensic walkthrough of a specific lab exercise?