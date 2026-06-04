### Table of Contents 📚

[[#Overview and Goals 🔎]]  
[[#Lab Topology and Preconditions 🧭]]  
[[#OpenVPN Server Configuration Deep Dive 🔐]]  
[[#OpenVPN Client Configuration Deep Dive 🧑‍💻]]  
[[#Capture Strategy and Where to Capture 🎯]]  
[[#Decrypting and Inspecting OpenVPN Traffic 🔓]]  
[[#Automated Evidence Extraction with tshark and Scapy 🤖]]  
[[#Correlating Logs PCAPs and Server State Timeline 🕒]]  
[[#Packaging Evidence and Manifest Format 📦]]  
[[#Common Failures and Forensic Remedies 🛠️]]  
[[#Quick Reference Commands and Example Artifacts 🧾]]

---

### Overview and Goals 🔎

This is a **forensic‑grade**, end‑to‑end walkthrough for OpenVPN in pfSense and GNS3 labs. The objective is to teach you how to configure OpenVPN server and client, capture the right packets at the right places, decrypt and inspect traffic in a controlled lab, extract minimal proof artifacts, and assemble a compact, reproducible evidence package that proves authentication, tunnel establishment, and data egress. Every step explains **what** to do, **why** it matters, and **what to collect** as evidence. The procedures assume a lab environment where you control both endpoints and have authorization to decrypt traffic.

[[#Table of Contents 📚]]

---

### Lab Topology and Preconditions 🧭

Design intent and minimal topology. The lab models a pfSense OpenVPN server on the edge, a remote client (Kali or Windows), and an internal web service reachable through the tunnel. The server has a WAN interface and a LAN interface. The client is external and connects over the Internet to the server’s WAN IP. The internal web service sits on the LAN behind pfSense. The goal is to prove three things: the client successfully authenticates to OpenVPN, the client receives a tunnel IP, and the client’s traffic egresses through pfSense to the Internet or internal resources.

Preconditions. Ensure NTP is synchronized on all systems. Ensure you have administrative access to pfSense and the client. Prepare a controlled attacker server (your logging endpoint) if you need to demonstrate SSRF or outbound fetches. Use short, deterministic test markers (unique strings with timestamps) to avoid ambiguity in logs.

Evidence you will collect. The minimal evidence set includes a WAN pcap showing the OpenVPN handshake, a tunnel pcap showing decrypted traffic (captured on the server side before encryption or decrypted via keys), server logs showing client authentication and assigned tunnel IP, client logs showing successful connection, and a JSON manifest that ties commands to packet numbers and timestamps.

[[#Table of Contents 📚]]

---

### OpenVPN Server Configuration Deep Dive 🔐

What to configure and why. Use pfSense’s OpenVPN wizard for a correct baseline, but understand each option. Create a Certificate Authority and a server certificate under **System Cert Manager**. The server certificate authenticates the server to clients and enables TLS. Choose a TLS cipher suite that balances security and performance; prefer AES‑GCM ciphers and TLS 1.2/1.3 where supported. Use a dedicated tunnel network (for example 10.8.0.0/24) to avoid overlapping routes.

Exact server settings that matter. In the OpenVPN server configuration choose `Protocol` UDP for performance, set `Device Mode` to `tun` for routed IP tunnels, set `Interface` to WAN, set `Local Port` to 1194 (or a custom port), and set `Tunnel Network` to `10.8.0.0/24`. Enable `Redirect Gateway` only if you intend to route client Internet traffic through the server; otherwise restrict to internal resources. Enable `Compression` only if you understand the security implications; compression can leak data in some attacks. Use `TLS Authentication` (ta.key) to mitigate UDP port scanning and DoS.

Commands and expected server artifacts. After enabling the server, pfSense will create server processes and log entries. The server log lines you will capture include TLS handshake messages and client common name. Example evidence lines to capture from **Status → OpenVPN** and system logs show the client CN and assigned IP. The server will also create a `tun` interface (for example `ovpns1`) with the assigned tunnel network. Use the pfSense shell to list the interface and check routes:

```bash
# show OpenVPN status and assigned clients
cat /var/log/openvpn.log
# show tunnel interface and IPs
ifconfig ovpns1
# show routing table
netstat -rn
```

Why each artifact matters. The `openvpn.log` proves authentication and the client CN. The `ifconfig` output proves the server has the tunnel interface and the assigned IPs. The routing table shows how traffic will be forwarded. Capture these outputs and include timestamps.

[[#Table of Contents 📚]]

---

### OpenVPN Client Configuration Deep Dive 🧑‍💻

What to configure on the client and why. Use the exported client configuration from pfSense’s Client Export package. The client config contains the server address, port, TLS settings, and the client certificate or inline PKCS#12. For reproducible evidence, embed the `tls-auth` key and the client certificate in the `.ovpn` file so the client can be started non‑interactively.

Minimal client commands and logs. On a Linux client, start OpenVPN with verbose logging to capture the handshake and assigned IP:

```bash
sudo openvpn --config client.ovpn --log /tmp/openvpn_client.log --verb 4
```

Expected client log artifacts. The client log will show the TLS handshake, certificate verification, and the assigned tunnel IP (for example `Initialization Sequence Completed` and `ifconfig 10.8.0.6/24`). Capture the client log and the output of `ip a` to show the tunnel interface and IP.

Why the client artifacts matter. The client log proves the client initiated the connection and received a tunnel IP. The `ip a` output proves the OS created the tunnel interface. These artifacts correlate with server logs and pcaps to show a complete handshake.

[[#Table of Contents 📚]]

---

### Capture Strategy and Where to Capture 🎯

Capture locations and rationale. Capture at three strategic points to build an irrefutable timeline.

Capture 1 WAN ingress on pfSense. This pcap shows the encrypted OpenVPN handshake and the client’s source IP. It proves the client reached the server’s public endpoint. Use the pfSense packet capture UI or SSH and `tcpdump`:

```bash
# capture OpenVPN traffic on WAN
tcpdump -i em0 host <client-ip> and port 1194 -s 0 -w /tmp/openvpn_wan.pcap
```

Capture 2 Tunnel interface on pfSense. Capture on the `ovpnsX` interface to see decrypted traffic if captured on the server before encryption, or to capture decrypted traffic if you run OpenVPN in a mode that allows local capture of plaintext. Use:

```bash
# capture decrypted traffic on the tunnel interface
tcpdump -i ovpns1 -s 0 -w /tmp/openvpn_tun.pcap
```

Capture 3 Client side capture. On the client, capture the local tunnel interface to show the client’s outbound requests and the assigned tunnel IP:

```bash
# on client
sudo tcpdump -i tun0 -s 0 -w /tmp/client_tun.pcap
```

Why these captures are complementary. The WAN capture proves the encrypted handshake and the client’s public IP. The tunnel capture on the server proves decrypted payloads and the actual application traffic that traverses the tunnel. The client capture proves the client’s local behavior and the exact request that generated the traffic. Together they show authentication, tunnel establishment, and data flow.

Timing discipline. Start all captures before initiating the client connection and stop them immediately after you have the evidence. Use epoch timestamps to correlate captures. Use `tshark` to export epoch times:

```bash
tshark -r /tmp/openvpn_wan.pcap -T fields -e frame.number -e frame.time_epoch -e ip.src -e ip.dst -e udp.srcport -e udp.dstport > wan_index.txt
```

[[#Table of Contents 📚]]

---

### Decrypting and Inspecting OpenVPN Traffic 🔓

When and how you can decrypt. In a lab where you control the server and client, you can decrypt OpenVPN traffic by capturing on the server’s tunnel interface (where traffic is already decrypted) or by using the server’s private keys and OpenVPN session keys if you captured encrypted WAN traffic and have access to the keys. OpenVPN uses TLS for control and symmetric keys for data; decrypting encrypted UDP payloads captured on the WAN is nontrivial unless you have the session keys or captured the plaintext on the server.

Preferred method: capture decrypted traffic on the server’s tunnel interface. This avoids key extraction and is the most defensible approach. The `ovpns1` capture will contain plaintext application traffic.

Alternative method: use the OpenVPN `--key-direction` and `tls-auth` secrets plus the server’s `static.key` and session keys if you recorded them. This is complex and error prone; prefer server‑side decrypted captures.

Inspecting decrypted traffic. Once you have a decrypted pcap, use Wireshark or `tshark` to extract HTTP requests, DNS queries, and other application payloads. Example to list HTTP URIs and packet numbers:

```bash
tshark -r /tmp/openvpn_tun.pcap -Y http.request -T fields -e http.request.full_uri -e frame.number -e frame.time_epoch > http_requests.txt
```

If you need to inspect TLS payloads inside the tunnel (for example the client is making HTTPS requests through the tunnel), use the `SSLKEYLOGFILE` method on the client browser to decrypt TLS in Wireshark. Set `SSLKEYLOGFILE` on the client before launching the browser, capture the tunnel interface, and configure Wireshark to use the pre‑master secret log.

[[#Table of Contents 📚]]

---

### Automated Evidence Extraction with tshark and Scapy 🤖

Why automation matters. Reviewers want minimal artifacts. Scripts let you extract exact packet numbers, byte offsets, and small binary/text files that contain only the proof. Below are repeatable commands and a Scapy script to extract an HTTP response body or an Authorization header.

Index HTTP requests and packet numbers:

```bash
tshark -r /tmp/openvpn_tun.pcap -Y http.request -T fields -e http.request.full_uri -e frame.number -e frame.time_epoch > /tmp/http_requests_index.txt
```

Extract a minimal pcap slice by packet numbers (example packets 120–130):

```bash
editcap -r /tmp/openvpn_tun.pcap /tmp/evidence_slice.pcap 120-130
```

Hex dump of a key packet:

```bash
tshark -r /tmp/evidence_slice.pcap -Y "frame.number == 123" -x > /tmp/packet_123_hex.txt
```

Scapy script to extract HTTP payloads and write them to small files:

```python
# language: python
from scapy.all import rdpcap, TCP, Raw, IP
pcap = rdpcap('/tmp/openvpn_tun.pcap')
counter = 0
for i, pkt in enumerate(pcap, start=1):
    if pkt.haslayer(TCP) and pkt.haslayer(Raw):
        payload = bytes(pkt[Raw].load)
        if b'HTTP/' in payload or b'GET ' in payload or b'POST ' in payload:
            fname = f'evidence_http_{i}.bin'
            with open(fname, 'wb') as fh:
                fh.write(payload)
            counter += 1
print(f'Wrote {counter} evidence files')
```

Why these artifacts are useful. The `evidence_slice.pcap` is the minimal pcap for validation. The hex dump and small binary files show the exact bytes you will present as proof. Include SHA256 checksums for each artifact.

[[#Table of Contents 📚]]

---

### Correlating Logs PCAPs and Server State Timeline 🕒

Building a timeline. Use epoch timestamps from `tshark` and log timestamps from pfSense and the client. Export pcap frame times:

```bash
tshark -r /tmp/openvpn_wan.pcap -T fields -e frame.number -e frame.time_epoch -e ip.src -e ip.dst > /tmp/wan_times.txt
tshark -r /tmp/openvpn_tun.pcap -T fields -e frame.number -e frame.time_epoch -e ip.src -e ip.dst > /tmp/tun_times.txt
```

Extract server log lines with timestamps and convert to epoch if necessary. On pfSense, system logs include timestamps; use `clog` to read circular logs:

```bash
clog /var/log/openvpn.log | sed -n '1,200p' > /tmp/openvpn_log_excerpt.txt
```

Aligning events. Use the epoch times to align the client’s `Initialization Sequence Completed` log line with the WAN handshake packets and the first decrypted application packet on the tunnel interface. Document any clock skew and include it in the manifest. If clocks differ, compute the offset and annotate the manifest with the offset and method used to compute it.

Example timeline narrative. At epoch `1654320005.123`, the WAN pcap shows the client’s TLS ClientHello. At epoch `1654320006.456`, the server log shows `VERIFY OK` and `Peer Connection Initiated`. At epoch `1654320007.789`, the tunnel pcap shows the first HTTP GET from `10.8.0.6` to `192.168.2.10`. This sequence proves causality: the client handshake led to an authenticated tunnel and subsequent application traffic.

[[#Table of Contents 📚]]

---

### Packaging Evidence and Manifest Format 📦

What to include in the package. A compact evidence package contains the minimal pcap slice, the client and server logs, the small extracted artifacts (HTTP payloads or hex dumps), a JSON manifest, and checksums. Keep the package encrypted when sharing.

Example manifest structure. The manifest must be machine readable and human friendly. Include commands used, packet numbers, timestamps, and confidence.

```json
{
  "test_name": "OpenVPN Full Tunnel Proof",
  "hypothesis": "Client authenticates and routes traffic through pfSense OpenVPN server",
  "commands": [
    "tcpdump -i em0 host <client-ip> and port 1194 -s 0 -w /tmp/openvpn_wan.pcap",
    "tcpdump -i ovpns1 -s 0 -w /tmp/openvpn_tun.pcap",
    "openvpn --config client.ovpn --log /tmp/openvpn_client.log --verb 4"
  ],
  "pcap_files": [
    {"file":"openvpn_wan.pcap","packets":[10,11,12],"timestamps":["1654320005.123","1654320005.456"]},
    {"file":"openvpn_tun.pcap","packets":[45,46,47],"timestamps":["1654320007.789"]}
  ],
  "evidence_files": [
    {"file":"evidence_slice.pcap","packets":[45,46],"sha256":"<sha256sum>"},
    {"file":"openvpn_client.log","sha256":"<sha256sum>"}
  ],
  "confidence":"high",
  "notes":"All captures started before initiating client connection. NTP synchronized across hosts."
}
```

Why this packaging works. The manifest is the index that lets a reviewer reproduce the steps and validate the claim without parsing full pcaps. Include SHA256 checksums and a short narrative tying the artifacts to the claim.

[[#Table of Contents 📚]]

---

### Common Failures and Forensic Remedies 🛠️

Failure mode 1 Client fails to authenticate. Forensics: capture the WAN handshake and server logs. Look for certificate verification errors or `VERIFY ERROR`. Remedy: verify client certificate CN, check CA trust, and ensure `tls-auth` key direction matches. Evidence to collect: `openvpn.log` lines showing `VERIFY ERROR` and the WAN pcap showing the ClientHello.

Failure mode 2 Tunnel established but no application traffic. Forensics: capture the tunnel interface and check routing. Use `netstat -rn` on pfSense and `ip route` on the client. Remedy: ensure `push "route ..."` or `redirect-gateway` is configured as intended and that firewall rules allow traffic from the tunnel network. Evidence: `ifconfig` showing tunnel IP, `netstat -rn` showing route, and `tcpdump` showing no application packets.

Failure mode 3 High packet loss or disconnects. Forensics: capture both WAN and tunnel interfaces and check for retransmissions, MTU issues, or UDP fragmentation. Remedy: lower MTU on the client or enable `mssfix` in OpenVPN. Evidence: `tshark` output showing retransmissions and ICMP fragmentation messages.

Failure mode 4 Unable to decrypt TLS inside tunnel. Forensics: use `SSLKEYLOGFILE` on the client browser to decrypt HTTPS payloads. If you cannot set `SSLKEYLOGFILE`, capture decrypted traffic on the server side instead. Evidence: `sslkeys.log` and Wireshark decrypted view or server tunnel pcap.

[[#Table of Contents 📚]]

---

### Quick Reference Commands and Example Artifacts 🧾

Start a verbose OpenVPN client and log to file:

```bash
sudo openvpn --config client.ovpn --log /tmp/openvpn_client.log --verb 4
```

Capture WAN OpenVPN traffic on pfSense:

```bash
tcpdump -i em0 host <client-ip> and port 1194 -s 0 -w /tmp/openvpn_wan.pcap
```

Capture decrypted tunnel traffic on pfSense:

```bash
tcpdump -i ovpns1 -s 0 -w /tmp/openvpn_tun.pcap
```

Index HTTP requests and packet numbers:

```bash
tshark -r /tmp/openvpn_tun.pcap -Y http.request -T fields -e http.request.full_uri -e frame.number -e frame.time_epoch > /tmp/http_requests_index.txt
```

Extract minimal evidence slice:

```bash
editcap -r /tmp/openvpn_tun.pcap /tmp/evidence_slice.pcap 120-130
tshark -r /tmp/evidence_slice.pcap -x > /tmp/evidence_hex.txt
sha256sum /tmp/evidence_slice.pcap > /tmp/evidence_slice.pcap.sha256
```

Example artifacts to deliver. `openvpn_wan.pcap`, `openvpn_tun.pcap`, `evidence_slice.pcap`, `openvpn_client.log`, `openvpn_server.log`, `http_requests_index.txt`, `manifest.json`, and SHA256 checksum files for each artifact.

[[#Table of Contents 📚]]

---

If you want, I will expand any single subsection into a runnable script set that automates capture, extraction, and manifest generation for your lab environment. Which would you like next: an automated evidence extraction script for the server, a client automation script that sets `SSLKEYLOGFILE` and runs the browser, or a ready‑to‑run `tshark`/Scapy packager that produces the final manifest and checksums?