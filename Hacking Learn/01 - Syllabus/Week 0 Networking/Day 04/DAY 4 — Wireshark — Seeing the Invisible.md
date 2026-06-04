### Table of Contents 📚

[[#I. Why Wireshark Matters 🔍]]  
[[#II. What Wireshark Actually Shows 🧠]]  
[[#III. The Anatomy of a Packet And Why It Matters 🧩]]  
[[#IV. Filters The Language of Wireshark 🗣️]]  
[[#V. Following Streams Reading Conversations 🔗]]  
[[#VI. Attack Traffic Signatures What to Recognize ⚠️]]  
[[#VII. LAB Wireshark Deep Dive 🧪]]  
[[#VIII. Practical Investigation Checklist ✅]]  
[[#IX. Why Wireshark Makes You Dangerous ⚔️]]  
[[#X. End of Day Outcomes 🎯]]  
[[#Appendix Quick Commands and Filters Cheat Sheet 🧾]]

---

# 🧨 DAY 4 — Wireshark — Seeing the Invisible

**Changelog — stylistic adjustments applied**  
This version replaces lists with fully explicit paragraphs and clear subheadings inside each section. All code blocks are language‑tagged. The TOC uses Obsidian‑style local section links as requested. Each procedural step includes the _what_, _how_, and _why_ with precise expectations and forensic signals to look for.

_Estimated hands‑on time: 2–4 hours depending on depth of practice._

_“Today you learn to read the network like a book.”_

---

## I. Why Wireshark Matters 🔍

### Summary of purpose

Wireshark converts raw network bytes into human‑readable evidence. Where logs and dashboards summarize behavior, a packet capture is the primary source: it contains timestamps, exact bytes, and the ordering of events. This makes captures the single most reliable artifact for proving what happened on a network.

### What you gain by mastering it

When you can read captures you can prove whether a cookie was set, whether a token was transmitted in plaintext, whether a firewall silently dropped packets, or whether a TLS handshake negotiated a weak cipher. This moves you from speculation to reproducible proof: capture the packet, extract the byte range, and include that snippet in a report with packet number and timestamp.

### Forensic mindset to adopt

Treat every capture as a case file. Start by asking: who sent this, what did they send, when did they send it, where did it go, why was it sent, and how did the recipient respond. Your job is to answer those questions with exact bytes and timestamps, not impressions.

---

## II. What Wireshark Actually Shows 🧠

### Intent versus content

Wireshark reveals both intent and content. Intent is visible in the sequence and timing of packets: a POST followed by a redirect indicates a state change; repeated failed POSTs indicate brute force. Content is visible in application layer payloads: HTTP headers, query strings, cookies, and, when decrypted, JSON or form bodies.

### Mistakes and misconfigurations

Malformed headers, truncated TLS handshakes, mismatched content lengths, and unexpected status codes are all visible in captures. These mistakes are often the fastest path to exploitation because they reveal how the stack handles unexpected input.

### Secrets and leakage

Plaintext protocols leak credentials and tokens. Even with TLS, misconfigurations such as SNI exposure or improper certificate chains can reveal metadata. In a lab where you control the client, TLS key logging allows full decryption so you can inspect payloads that would otherwise be opaque.

### Behavior and timing

Retransmissions, RSTs, and window size changes reveal how endpoints and middleboxes behave under load or interference. Timing patterns can indicate scanning, rate limiting, or congestion. These behavioral signals are essential for diagnosing why an exploit or test failed.

---

## III. The Anatomy of a Packet And Why It Matters 🧩

### Link Layer inspection

At the link layer inspect source and destination MAC addresses, EtherType, and VLAN tags. Unexpected MAC addresses for known IPs or gratuitous ARP replies indicate local spoofing or ARP poisoning. When you see duplicate MACs or frequent gratuitous ARP, treat it as a red flag for local interception.

### Network Layer inspection

At the IP layer inspect source and destination IPs, TTL, fragmentation flags, and IP options. TTL anomalies can fingerprint operating systems or reveal routing loops. Fragmentation can be used to split malicious payloads across packets; reassembly is required to see the full payload. If you see many small fragments that reassemble into a suspicious payload, extract the reassembled bytes for analysis.

### Transport Layer inspection

At the transport layer inspect ports, sequence numbers, acknowledgements, window sizes, and flags. A stream of SYNs without ACKs is a scan. Frequent retransmissions or RST storms indicate instability or active blocking. Sequence and ACK patterns let you reconstruct session progress and detect injected or out‑of‑order packets.

### Application Layer inspection

At the application layer inspect headers and payloads for HTTP, DNS, TLS handshakes, SMTP, and other protocols. This is where user input, tokens, cookies, and error messages live. For HTTP, capture the request line, headers, and body. For TLS, capture ClientHello and ServerHello to inspect SNI and cipher negotiation. For DNS, inspect query names and response types to detect exfiltration or tunneling.

### Practical reading order

Begin with the capture timeline to narrow the timeframe. Then inspect a suspicious packet from link → network → transport → application. Each layer reduces the hypothesis space and points you to the next question to ask.

---

## IV. Filters The Language of Wireshark 🗣️

### Capture filters versus display filters

Capture filters (BPF) are applied while capturing and permanently exclude traffic you did not capture. Use capture filters only when you must limit storage or privacy exposure. Display filters are applied after capture and let you slice the stored pcap in many ways. Prefer display filters for investigation because they preserve context.

### How to craft a forensic query

When you write a display filter, be explicit about the host, protocol, and signal. For example, to find HTTP requests from a host use `http && ip.addr == 192.168.20.10`. This asks the capture to show only HTTP frames involving that host, which reduces noise while preserving context.

### Example filters with intent and expected signals

The filter `http` isolates web traffic so you can read request lines and headers. The filter `dns` isolates name lookups so you can detect exfiltration via DNS. The filter `ip.addr == 192.168.20.10` profiles a single host. The filter `tcp.flags.syn == 1 && tcp.flags.ack == 0` surfaces connection attempts and scanning behavior. The filter `tcp.analysis.retransmission` surfaces retransmissions that indicate packet loss or blocking. The filter `tls.handshake.type == 1` shows ClientHello messages so you can inspect offered cipher suites and SNI values.

### Advanced filter composition

Combine filters to narrow to precise signals. For example, `http.request.method == "POST" && http.content_type contains "application/json" && ip.addr == 192.168.20.10` finds JSON API POSTs from a host. Use such composed filters when hunting for tokens or API misuse.

---

## V. Following Streams Reading Conversations 🔗

### Why follow streams

Following a stream reconstructs the ordered bytes of a conversation so you can read the full request and response as a single transcript. Many vulnerabilities are only visible when you see the full exchange: reflected XSS, SQL injection responses, authentication flows, and multi‑step API interactions.

### Step‑by‑step stream analysis

First, identify a suspicious packet and note its packet number and timestamp. Second, right‑click and choose Follow → TCP Stream to reconstruct the conversation. Third, inspect the request line and headers to identify method, path, Host, and User‑Agent. Fourth, inspect query parameters and body to locate user input and potential payloads. Fifth, inspect the response status and body to determine whether the input was reflected, executed, or caused an error. Sixth, export the stream as raw bytes and record the packet numbers and byte offsets for reporting.

### Example extraction with tshark

Use `tshark` to extract URIs for triage. The command below reads the pcap, filters for HTTP requests, and prints the full URI for each request so you can prioritize streams for manual follow‑up.

```bash
tshark -r /tmp/capture.pcap -Y http.request -T fields -e http.request.full_uri
```

This command is scriptable and useful for automated evidence collection pipelines.

---

## VI. Attack Traffic Signatures What to Recognize ⚠️

### SQL injection signature and interpretation

When you see query parameters containing unescaped quotes and boolean logic such as `id=1' OR '1'='1`, inspect the server response for SQL errors, unexpected rows, or schema leaks. If the response contains database error messages or returns more data than expected, you have evidence of injection. Capture the request and the corresponding response, note packet numbers, and extract the exact bytes that demonstrate the injection and the server’s reaction.

### XSS signature and interpretation

When you see script tags in parameters such as `<script>alert(1)</script>`, follow the stream and inspect the response body. If the response contains the script unescaped in HTML, you have evidence of reflected XSS. If the payload persists across pages or appears in stored content, you have evidence of stored XSS. For proof, capture the request and the response that contains the payload and include the exact HTML snippet and byte offsets.

### Command injection signature and interpretation

When you see shell metacharacters in inputs such as `; cat /etc/passwd`, inspect the response for system file contents or error messages referencing system paths. If the response contains system output, you have evidence of command execution. Capture the request and the server response and extract the bytes that show the system output.

### SSRF signature and interpretation

When you see requests to internal addresses such as `http://127.0.0.1:80/admin` originating from an externally accessible context, inspect whether the server made an outbound request to an internal resource. Evidence of SSRF is the server’s outbound request and the internal resource’s response captured in the server’s response. Capture both the inbound request and the server’s outbound request if possible.

### Brute force and scanning signatures

Brute force appears as repeated authentication attempts to the same endpoint with different credentials. Port scanning appears as many SYN packets to many ports with few established sessions. For both, quantify the pattern: count attempts, note timing, and extract the packets that show the repeated behavior.

### MITM indicators

MITM is indicated by duplicate ARP replies mapping an IP to an unexpected MAC and by TLS certificate chains issued by an unexpected CA. When you see both an ARP anomaly and a certificate mismatch, capture the ARP packet and the TLS handshake that follows; together they prove local interception.

---

## VII. LAB Wireshark Deep Dive 🧪

This lab is prescriptive. Each exercise includes the command to run, the exact Wireshark filters to apply, the expected observations, and the forensic interpretation you must produce.

### Lab 0 Capture discipline with tcpdump

Run `tcpdump` to capture traffic to a pcap for offline analysis. The command below captures all traffic on `eth0` and writes it to `/tmp/capture.pcap`. Use Ctrl+C to stop.

```bash
sudo tcpdump -i eth0 -w /tmp/capture.pcap
```

This command records raw frames. If you must limit capture for privacy or disk reasons, use a capture filter such as the example below to capture only HTTP and DNS traffic.

```bash
sudo tcpdump -i eth0 -w /tmp/http_dns.pcap 'tcp port 80 or udp port 53'
```

After capture, open the pcap in Wireshark for GUI inspection.

```bash
wireshark /tmp/capture.pcap
```

When you open the pcap, note the capture start and end timestamps and the interface used. These contextual details are essential for correlating with logs.

### Exercise 1 Capture HTTP traffic and read the web raw

From the attacker VM run a `curl` command to generate a simple HTTP request. The `-v` flag prints client behavior to the console and helps you correlate client logs with captured packets.

```bash
curl -v http://192.168.20.10/search?q=test
```

In Wireshark apply the display filter `http`. Right‑click a packet and choose Follow → TCP Stream. You will see the request line `GET /search?q=test HTTP/1.1`, headers such as `Host`, `User-Agent`, and `Cookie`, and the response status and body. The reason this matters is that HTTP is plaintext: parameters and cookies are visible and often contain tokens or injection points. For automated extraction of URIs, run:

```bash
tshark -r /tmp/capture.pcap -Y http.request -T fields -e http.request.full_uri
```

Use the output to prioritize streams for manual follow‑up.

### Exercise 2 Capture HTTPS handshake and decrypt in lab

Visit the HTTPS site from a browser on the attacker VM using `https://192.168.20.10`. In Wireshark apply the display filter `tls.handshake` to see ClientHello and ServerHello messages. The ClientHello contains SNI and offered cipher suites; the ServerHello contains the chosen cipher and the server certificate. To decrypt TLS in a lab where you control the client, set the `SSLKEYLOGFILE` environment variable before launching the browser so the browser writes pre‑master secrets to the key log file. Point Wireshark to that file in Preferences → Protocols → TLS.

```bash
export SSLKEYLOGFILE=/tmp/sslkeys.log
# start the browser from the same shell so it writes keys to /tmp/sslkeys.log
```

With the key log configured, Wireshark will decrypt TLS sessions and show HTTP/JSON payloads. This is essential for inspecting HTTPS payloads in a controlled lab.

### Exercise 3 Capture port scans and extract open ports

Run a SYN scan from the attacker VM to enumerate open ports. The `-sS` option performs a stealthy SYN scan and `-Pn` disables host discovery.

```bash
nmap -sS -Pn 192.168.20.10 -p 1-1024
```

In Wireshark apply the display filter `tcp.flags.syn == 1 && tcp.flags.ack == 0` to show SYN packets. You will observe rapid SYNs to many ports and responses that are either SYN/ACK for open ports or RST for closed ports. To extract ports that responded with SYN/ACK from the pcap, run:

```bash
tshark -r /tmp/capture.pcap -Y 'tcp.flags.syn==1 && tcp.flags.ack==1' -T fields -e tcp.dstport | sort -n | uniq -c
```

This pipeline counts SYN/ACK responses per destination port and helps prioritize services to investigate.

### Exercise 4 Capture SQL injection attempt and prove impact

Send a classic injection payload with `curl`. Capture the request and the server response. The presence of SQL error messages or unexpected data in the response is evidence of injection.

```bash
curl -v "http://192.168.20.10/?id=1' OR '1'='1"
```

In Wireshark apply the display filter `http.request` and follow the stream. To extract request bodies for review, run:

```bash
tshark -r /tmp/capture.pcap -Y http.request -T fields -e http.request.method -e http.request.uri -e http.file_data
```

If the response contains SQL errors or returns more rows, capture the exact response bytes and record packet numbers and timestamps for reporting.

### Exercise 5 Capture XSS payload and demonstrate reflection

Send a script payload and inspect the response body for reflection. If the response contains the script unescaped, you have evidence of reflected XSS. For stored XSS, repeat the request and then visit other pages to see if the payload persists.

```bash
curl -v "http://192.168.20.10/?q=<script>alert(1)</script>"
```

Follow the HTTP stream in Wireshark and capture the response that contains the payload. Export the response bytes and include the HTML snippet and byte offsets in your evidence.

### Exercise 6 Capture blocked or filtered traffic to detect firewall behavior

Attempt an SSH connection with verbose output to correlate client retries with network retransmissions. Capture the traffic and inspect whether the server replies or whether packets are silently dropped.

```bash
ssh -vvv 192.168.30.10
```

In Wireshark apply the display filter `tcp.port == 22`. If you see SYNs from the client and no SYN/ACK from the server, the traffic is being silently dropped. If you see ICMP unreachable messages, the traffic is actively rejected. Quantify retransmissions and record packet numbers and timestamps.

### Exercise 7 Detect MITM indicators by correlating ARP and TLS anomalies

Inspect ARP traffic for gratuitous ARP replies and TLS handshakes for unexpected certificate issuers. Use the following display filters to surface these signals.

```bash
arp.opcode == 2
```

```bash
tls.handshake.certificate
```

When you find an ARP reply mapping a known IP to an unexpected MAC, capture that packet and the subsequent TLS handshake that uses a certificate issued by an unexpected CA. These two artifacts together prove local interception.

### Advanced extraction with Scapy for precise byte offsets

Use Scapy to parse a pcap and extract HTTP URIs and the byte offsets where suspicious headers appear. The script below iterates packets, finds raw TCP payloads, and locates Authorization headers by byte offset. Use this to produce exact byte ranges for reporting.

```python
from scapy.all import rdpcap, TCP, Raw, IP
pcap = rdpcap('/tmp/capture.pcap')
for pkt in pcap:
    if pkt.haslayer(TCP) and pkt.haslayer(Raw) and pkt.haslayer(IP):
        payload = bytes(pkt[Raw].load)
        if b'HTTP/1.1' in payload or b'GET ' in payload or b'POST ' in payload:
            src = pkt[IP].src
            dst = pkt[IP].dst
            ts = pkt.time
            print(ts, src, dst, len(payload))
            idx = payload.find(b"Authorization:")
            if idx != -1:
                print("Authorization header at byte offset", idx, "in packet with timestamp", ts)
```

Use the output to reference exact packet numbers and byte offsets in your report.

---

## VIII. Practical Investigation Checklist ✅

When you open a capture, follow a disciplined workflow that produces reproducible evidence. First, identify the timeframe of interest by noting capture start and end timestamps and narrowing to the window where the event occurred. Second, note the capture interface and any capture filters used because capture context affects interpretation. Third, filter by host or service using `ip.addr` or `tcp.port` to reduce noise and focus on the actor or service. Fourth, scan for anomalies such as retransmissions, duplicate ARP, or certificate mismatches using `tcp.analysis.retransmission` and ARP/TLS filters. Fifth, follow suspicious streams to reconstruct the full context of requests and responses. Sixth, inspect headers and payloads to find credentials, tokens, or injection points and extract the exact bytes for evidence. Seventh, export the relevant packets or streams as separate pcaps or raw byte files for inclusion in reports. Eighth, correlate your findings with server logs, IDS alerts, or system timestamps to build a timeline. Ninth, document your findings with packet numbers, timestamps, and byte offsets so others can reproduce your analysis.

---

## IX. Why Wireshark Makes You Dangerous ⚔️

Mastering Wireshark changes your workflow from guesswork to evidence‑driven analysis. You will be able to demonstrate credential leakage by showing the exact Authorization header bytes and the packet number. You will be able to show how an injection payload was reflected and caused a server error by exporting the request and response bytes. You will be able to decrypt TLS in a lab and show the JSON body of an API call that contains a secret. This ability to produce unambiguous, reproducible evidence is what separates casual testers from skilled investigators.

---

## X. End of Day Outcomes 🎯

By the end of Day 4 you will be able to read packets like sentences and follow streams like conversations. You will be able to identify attack patterns by sight, detect misconfigurations and trust issues, and produce evidence that proves behavior and impact. You will be able to extract URIs and headers with `tshark`, decrypt TLS sessions in a lab using `SSLKEYLOGFILE`, quantify retransmissions and scans, and export exact byte ranges for reporting. Your reports will include packet numbers, timestamps, and byte offsets so that any reviewer can reproduce your findings.

---

## Appendix Quick Commands and Filters Cheat Sheet 🧾

### Capture with tcpdump

Capture everything on `eth0` and write to `capture.pcap`. Use Ctrl+C to stop.

```bash
sudo tcpdump -i eth0 -w capture.pcap
```

Capture only HTTP and DNS using a BPF capture filter to reduce disk usage and privacy exposure.

```bash
sudo tcpdump -i eth0 -w http_dns.pcap 'tcp port 80 or udp port 53'
```

### Open pcap in Wireshark

Launch the GUI for deep inspection.

```bash
wireshark capture.pcap
```

### Use tshark for scripted extraction

List all HTTP URIs found in the pcap.

```bash
tshark -r capture.pcap -Y http.request -T fields -e http.request.full_uri
```

Extract TLS ClientHello SNI values.

```bash
tshark -r capture.pcap -Y 'tls.handshake.type==1' -T fields -e tls.handshake.extensions_server_name
```

Count retransmissions.

```bash
tshark -r capture.pcap -Y 'tcp.analysis.retransmission' -T fields -e frame.number | wc -l
```

### TLS decryption in a lab

Set the environment variable before launching the browser so the browser writes pre‑master secrets to the key log file. Start the browser from the same shell so it writes keys to the file.

```bash
export SSLKEYLOGFILE=/tmp/sslkeys.log
# start browser from same shell
```

Point Wireshark to `/tmp/sslkeys.log` in Preferences → Protocols → TLS to decrypt sessions.

### Useful display filters as forensic queries

Use these filters to reduce noise and surface the signal you need.

```text
http
dns
tls.handshake
ip.addr == 192.168.20.10
tcp.port == 22
tcp.flags.syn == 1 && tcp.flags.ack == 0
tcp.analysis.retransmission
http.request.method == "POST" && http.content_type contains "application/json"
```

### Common attacker commands used to generate observable traffic

Use these commands in a lab to generate traffic you can capture and analyze.

```bash
curl http://192.168.20.10
```

```bash
curl -v "http://192.168.20.10/?id=1' OR '1'='1"
```

```bash
curl -v "http://192.168.20.10/?q=<script>alert(1)</script>"
```

```bash
nmap -sS 192.168.20.10
```

```bash
ssh -vvv 192.168.30.10
```

---

### Evidence hygiene and legal notes 🧼

Always capture with explicit consent on real networks. Use lab environments for offensive testing. Minimize retention of sensitive data by redacting or hashing credentials in reports unless the raw evidence is required for proof. Ensure host clocks are synchronized using NTP so timestamps align across captures and logs. Export evidence with packet numbers, timestamps, and byte offsets to make your findings reproducible.

---

If you want deeper expansion of any single subheading into a step‑by‑step walkthrough that includes exact expected packet bytes and the precise `tshark` or Scapy commands to extract them, tell me which subheading to expand and I will produce a full forensic walkthrough with exact byte offsets and example output.