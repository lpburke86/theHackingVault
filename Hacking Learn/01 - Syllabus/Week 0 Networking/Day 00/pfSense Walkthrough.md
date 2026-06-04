### Table of Contents 📚

[[#1. Overview and Goals]]  
[[#2. System Requirements and Prerequisites]]  
[[#3. Downloading pfSense Community Edition and Verification]]  
[[#4. Installing pfSense (VirtualBox, VMware, Bare Metal)]]  
[[#5. Initial Configuration and Secure Baseline]]  
[[#6. Networking Fundamentals in pfSense — Interfaces, NAT, and Routing]]  
[[#7. Firewall Rules, Aliases, and Advanced Filtering]]  
[[#8. VPNs, Tunnels, and Remote Access (OpenVPN, IPsec, WireGuard)]]  
[[#9. High Availability, CARP, and Failover]]  
[[#10. Logging, Monitoring, and Forensics (Packet Capture, Suricata, pfSense Logs)]]  
[[#11. Performance Tuning, Resource Planning, and Troubleshooting]]  
[[#12. Building Reproducible Labs with pfSense (3 complete labs)]]  
[[#13. Backups, Upgrades, Snapshots, and Disaster Recovery]]  
[[#14. Security, Hardening, and Operational Hygiene]]  
[[#15. Appendix: Useful Commands, Config Snippets, and References]]

---

# 1. Overview and Goals

This is a complete, **forensic‑grade**, step‑by‑step manual for **pfSense Community Edition** (CE). It takes you from download to a hardened, production‑like firewall appliance, and then into advanced use: VPNs, tunnels, IDS/IPS, high availability, packet capture, and forensic evidence packaging. The guide is intentionally exhaustive and practical: every command, every configuration choice, and every capture technique includes **what** to do and **why** it matters.

What you will get by following this guide:

- A working pfSense CE installation on VirtualBox, VMware, or bare metal.
- A secure baseline configuration and hardening checklist.
- Deep explanations of interfaces, NAT, routing, and firewall semantics.
- VPN and tunneling labs: OpenVPN, IPsec, WireGuard, and GRE.
- IDS/IPS integration (Suricata), logging, and packet capture workflows for evidence.
- High availability with CARP and state synchronization.
- Performance tuning, resource planning, and troubleshooting checklists.
- Reproducible lab scenarios with expected artifacts and evidence packaging.

Why this matters: pfSense is widely used in labs, small businesses, and production. Understanding not only how to configure it but why each setting exists is essential for building realistic labs, performing security testing, and producing defensible evidence.

[[#Table of Contents 📚]]

---

# 2. System Requirements and Prerequisites

**Minimum vs Recommended hardware**

- **Minimum**: 2 CPU cores, 2 GB RAM, 8 GB disk. Use only for basic routing and small NAT labs.
- **Recommended**: 4 CPU cores, 8–16 GB RAM, 50+ GB SSD. Use for VPN termination, IDS/IPS, and multiple tunnels.
- **High‑performance**: 8+ cores, 32+ GB RAM, NVMe storage, multiple NICs (1 Gbps or 10 Gbps). Use for heavy IDS/IPS, many concurrent VPN clients, or high throughput NAT.

**Why**: pfSense runs FreeBSD and services such as Suricata, OpenVPN, or WireGuard consume CPU and memory. Disk I/O matters for logging and for Suricata rulesets. Underprovisioning leads to dropped packets, missed IDS alerts, and unreliable VPN performance.

**Network interface recommendations**

- Use **dedicated physical NICs** for WAN and LAN on bare metal. Prefer Intel NICs for driver stability.
- For virtual deployments, use **virtio** or **VMXNET3** paravirtualized drivers for performance. Avoid emulated NICs (e1000) if you can.
- Plan for additional NICs for DMZ, management, and optional monitoring/span ports.

**Host OS and virtualization choices**

- **Bare metal**: best performance and stability. Use for production or high‑throughput labs.
- **VMware Workstation/ESXi**: excellent performance on Windows/macOS hosts. Use VMXNET3 and reserve CPU/memory.
- **VirtualBox**: convenient for desktop labs; ensure you enable paravirtualization and use bridged networking for WAN.
- **Proxmox/KVM**: great on Linux hosts; use virtio drivers and pass‑through NICs for best performance.

**Security and network prerequisites**

- Ensure the management network is isolated from WAN. Use a separate management interface or VLAN.
- Time synchronization: configure NTP on pfSense and on all lab hosts. Accurate timestamps are essential for forensic timelines.
- Backup plan: snapshot VMs before major changes; export configs regularly.

[[#Table of Contents 📚]]

---

# 3. Downloading pfSense Community Edition and Verification

**What to download**

- Official pfSense CE ISO from the Netgate website. Choose the architecture (AMD64) and installer type (USB memstick image or ISO). For virtual labs, use the ISO; for bare metal, use the memstick image.

**Why**: Always use official images to avoid tampered binaries. The memstick image is recommended for USB installs; ISO is convenient for VMs.

**Checksum verification**

After download, verify the SHA256 checksum against the value published on the Netgate site.

```bash
# example on Linux/macOS
sha256sum pfSense-CE-<version>-RELEASE-amd64.iso
# compare the output to the published checksum
```

**Why**: Checksum verification prevents installing compromised images. If the checksum does not match, re‑download and verify network integrity.

**Signing and GPG (if available)**

If Netgate publishes GPG signatures, verify the signature with the vendor’s public key. Import the key and verify:

```bash
gpg --import netgate_pubkey.asc
gpg --verify pfSense-<version>.asc pfSense-<version>.iso
```

**Why**: GPG verification adds an additional layer of supply‑chain assurance.

[[#Table of Contents 📚]]

---

# 4. Installing pfSense (VirtualBox, VMware, Bare Metal)

This section provides step‑by‑step installation instructions for the three common deployment modes. Each step includes the expected output and the forensic artifacts to collect (console logs, installer output).

## 4.1 VirtualBox installation (desktop lab)

**Create VM**

- **Name**: pfSense-CE
- **Type**: BSD → FreeBSD (64‑bit)
- **Memory**: 2048–4096 MB (recommended 4096 for VPN/IDS labs)
- **CPUs**: 2 cores minimum
- **Storage**: 20–50 GB VDI (dynamically allocated OK for labs)
- **Network adapters**: Adapter 1: Bridged (WAN); Adapter 2: Host‑only or Internal (LAN); Adapter 3: Host‑only or Internal (optional DMZ/Management)

**Why**: Bridged WAN lets the VM obtain an IP from your physical network or host NAT. Host‑only or Internal networks isolate lab traffic.

**Attach ISO and boot**

- Attach the pfSense ISO to the VM’s optical drive and boot. At the installer menu choose `Install` and follow prompts. Use `Auto (UFS)` for simple installs or `Manual (ZFS)` if you want ZFS features (ZFS requires more RAM).

**Expected installer output**

- Installer will detect disks and present partitioning options. After install, the system will prompt to remove the ISO and reboot. The console will display interface assignments (e.g., `em0` for WAN, `em1` for LAN).

**Post‑install console**

- On first boot, pfSense will present the console menu with options to assign interfaces, set IP addresses, and enable DHCP on LAN. Record the console output (screenshot or copy) as the initial evidence of a clean install.

**Why**: Console output proves the initial state and interface mapping for later troubleshooting.

## 4.2 VMware Workstation / ESXi installation

**Create VM**

- Use the `FreeBSD 12/13` template or `Other 64‑bit`. Use `VMXNET3` NICs for performance. Allocate 4 GB RAM for VPN/IDS labs.

**Install**

- Attach ISO and follow the same installer steps as VirtualBox. On ESXi, consider using thin provisioning and ensure the datastore has enough IOPS for logging.

**Why**: VMware provides better performance and driver support on Windows/macOS hosts.

## 4.3 Bare metal installation (USB memstick)

**Create bootable USB**

- Use `dd` on Linux/macOS or Rufus on Windows to write the memstick image to a USB drive.

```bash
# example (Linux/macOS)
sudo dd if=pfSense-CE-<version>-memstick.img of=/dev/sdX bs=1M status=progress && sync
```

**Install**

- Boot the target machine from USB, follow the installer, and assign NICs. For production, use mirrored disks or ZFS with redundancy.

**Why**: Bare metal gives best throughput and is recommended for production or high‑throughput IDS.

## 4.4 Initial interface assignment and management access

After install, pfSense console will prompt to assign WAN and LAN. Typical minimal assignment:

- **WAN**: DHCP or static public IP (connected to upstream router or host bridged network)
- **LAN**: 192.168.1.1/24 (management and internal network)

From a LAN host, open a browser to `http://192.168.1.1` and log in with the default credentials (admin/pfsense). Immediately change the admin password.

**Why**: Changing the default password is the first security step. The web UI is the primary management interface; ensure it is reachable only from management networks.

[[#Table of Contents 📚]]

---

# 5. Initial Configuration and Secure Baseline

This section is a prescriptive hardening checklist with exact UI steps and CLI commands. Each item includes the **why** and the **evidence** you should collect.

## 5.1 First steps: change admin password and set hostname/time

**Web UI steps**

- **System → General Setup**: set **Hostname**, **Domain**, and **DNS servers**. Configure **NTP servers** under **System → General Setup** or **System → NTP**.

**Why**: Hostname and NTP are essential for logs and timeline correlation. DNS ensures reliable resolution for package updates and rule testing.

**Evidence**: screenshot of General Setup page and `cat /etc/rc.conf` or `date` output showing NTP sync.

## 5.2 Secure management access

**Disable web GUI on WAN**

- **System → Advanced → Admin Access**: ensure **WebGUI** is not accessible on WAN. If remote management is required, restrict to specific IPs and use HTTPS only.

**Enable HTTPS**

- Generate or import a CA and certificate under **System → Cert Manager** and enable HTTPS on the GUI.

**Why**: Exposing the GUI on WAN is a common misconfiguration that leads to compromise. HTTPS prevents credential interception.

**Evidence**: firewall rule showing allowed management IPs and `openssl s_client -connect <pfsense-ip>:443 -showcerts` output showing the certificate.

## 5.3 Harden SSH and console access

**System → Advanced → Secure Shell**

- Disable SSH if not needed. If required, change the default port, restrict to management IPs, and use key‑based authentication.

**Why**: SSH is a high‑value attack surface. Key‑based auth and IP restrictions reduce risk.

**Evidence**: `sshd_config` excerpt and firewall rules permitting SSH only from management network.

## 5.4 Disable unnecessary services and packages

- Under **System → Packages**, avoid installing packages you do not need. Disable services such as UPnP, NAT‑PMP, or captive portal unless required.

**Why**: Each service increases the attack surface and resource usage.

**Evidence**: package list and `ps aux` showing running services.

## 5.5 Baseline firewall rules

**LAN rules**

- Allow only necessary outbound traffic. Use **Aliases** for networks and ports to simplify rules. Example: allow DNS (53), HTTP/HTTPS (80/443), and NTP to specific servers.

**WAN rules**

- Deny all inbound by default. Create explicit rules for required services (e.g., port forwarding for a DMZ web server) and log matches.

**Why**: Default deny on WAN and least privilege on LAN reduce exposure.

**Evidence**: export of firewall rules (`Diagnostics → Backup & Restore` or `pfctl -sr`) and screenshots of rule pages.

## 5.6 Logging and remote syslog

- Configure **Status → System Logs → Settings** to send logs to a remote syslog server (SIEM) and enable log rotation. For forensic labs, capture logs centrally.

**Why**: Centralized logs survive device reboots and are easier to analyze.

**Evidence**: syslog server entries with pfSense logs and timestamps.

## 5.7 Package management and updates

- **System → Update**: configure update server and schedule. Apply security updates promptly. For production, test updates in a staging environment first.

**Why**: Timely updates patch vulnerabilities. Testing prevents regressions.

**Evidence**: update history and package versions (`pkg info`).

[[#Table of Contents 📚]]

---

# 6. Networking Fundamentals in pfSense — Interfaces, NAT, and Routing

This section explains the core networking behaviors in pfSense and how to configure them correctly. Each subsection includes **what** the setting does, **why** it matters, and **how** to prove behavior with captures and commands.

## 6.1 Interface types and mapping

pfSense exposes physical and virtual interfaces. Common names: `em0`, `igb0`, `re0`, `vtnet0` (virtio). The console shows interface mapping on boot.

**How to view**

```bash
ifconfig -a
```

**Why**: Knowing the interface names is essential for assigning WAN/LAN and for capturing traffic.

**Evidence**: `ifconfig` output and console screenshot showing interface mapping.

## 6.2 NAT types and behavior

pfSense supports **1:1 NAT**, **Port Forwarding**, and **Outbound NAT** (automatic or manual). Understand the difference:

- **Automatic outbound NAT**: pfSense creates rules for internal networks to NAT to the WAN IP. Good for simple setups.
- **Manual outbound NAT**: you control SNAT rules, necessary for complex multi‑WAN or VPN scenarios.
- **1:1 NAT**: maps a public IP to a private IP for inbound services.
- **Port Forwarding**: maps a public port to an internal host/port.

**How to configure**

- **Firewall → NAT → Outbound**: switch to Manual if you need explicit control.
- **Firewall → NAT → Port Forward**: create a rule mapping WAN port to internal host.

**Why**: NAT behavior affects how internal services are reachable and how logs and pcaps appear. For example, with NAT, the source IP on outbound packets will be the WAN IP; without NAT, the internal IP may be visible if routing allows.

**Evidence**: `pfctl -s nat` output, `tcpdump` on WAN showing SNATed packets, and `show ip nat translations` equivalent in pfSense logs.

## 6.3 Routing and gateway groups

pfSense supports static routes and dynamic routing via packages (e.g., FRR). For multi‑WAN, use **Gateway Groups** to implement failover and load balancing.

**How to configure**

- **System → Routing → Gateways**: add gateways and create groups.
- **System → Routing → Static Routes**: add routes for remote networks.

**Why**: Proper routing ensures traffic flows through intended paths and that failover behaves predictably.

**Evidence**: `netstat -rn` output, `ping` and `traceroute` tests showing path selection, and `tcpdump` showing which WAN interface carries traffic.

## 6.4 VLANs and trunking

pfSense supports VLANs on physical interfaces. Create VLANs under **Interfaces → Assignments → VLANs** and assign them to interfaces.

**Why**: VLANs segment traffic and are essential for DMZs, management networks, and multi‑tenant labs.

**Evidence**: `ifconfig vlanX` output, switch trunk configuration, and `tcpdump` showing 802.1Q tags.

## 6.5 Proving network behavior with captures

**Capture locations**

- **LAN interface**: shows original internal packets.
- **WAN interface**: shows NATed or encapsulated packets.
- **Bridge or DMZ interface**: shows traffic between zones.

**Commands**

```bash
# capture on pfSense (Diagnostics → Packet Capture) or via SSH
tcpdump -i em0 -s 0 -w /tmp/wan_capture.pcap host 192.168.20.10
```

**Why**: Captures at different points prove NAT, routing, and firewall behavior. Always start captures before the test and stop immediately after to minimize unrelated data.

[[#Table of Contents 📚]]

---

## 7. Firewall Rules, Aliases, and Advanced Filtering

This section explains how pfSense evaluates rules, how to design rules for least privilege, and how to use aliases and schedules. Each concept includes the **why** and the **evidence** to collect.

## 7.1 Rule evaluation order and stateful behavior

pfSense uses a top‑to‑bottom rule evaluation per interface. The first matching rule applies. pfSense is stateful: an established connection is allowed back regardless of return rules, unless state is cleared.

**Why**: Misunderstanding evaluation order leads to unexpected traffic being allowed or blocked.

**How to prove**

- Create a rule that blocks a port and then attempt a connection. Use `pfctl -s state` to view states. Use `pfctl -sr` to list rules.

**Evidence**: `pfctl -s state` output showing established states and `tcpdump` showing whether packets are allowed or blocked.

## 7.2 Aliases and schedules

**Aliases** simplify rule management by grouping IPs, networks, or ports. **Schedules** allow time‑based rules.

**Why**: Aliases reduce errors and make rules auditable. Schedules are useful for maintenance windows or temporary access.

**How to configure**

- **Firewall → Aliases**: create alias `WEB_SERVERS` with internal IPs.
- Use alias in a rule: allow `WEB_SERVERS` to WAN on port 80.

**Evidence**: screenshot of alias and rule, `pfctl -sr` showing the rule with alias expansion.

## 7.3 Floating rules and advanced matching

Floating rules apply to multiple interfaces and can be used for global blocking or traffic shaping. Use advanced options for matching on direction, state, and interface.

**Why**: Floating rules are powerful for global policies like blocking malicious IPs before they hit internal interfaces.

**Evidence**: `pfctl -sr` showing floating rules and `tcpdump` demonstrating the rule in action.

## 7.4 Layer 7 and application awareness

pfSense supports packages like **Snort** or **Suricata** for deeper inspection. For HTTP/HTTPS application logic, use reverse proxies or web application firewalls in front of services.

**Why**: Layer 3/4 rules are insufficient for modern web threats. IDS/IPS provides signatures and anomaly detection.

**Evidence**: Suricata alerts, Snort logs, and packet captures showing payloads that triggered signatures.

## 7.5 Logging and rule hits

Enable logging on rules to capture matches. Use **Status → System Logs → Firewall** to view hits. For high‑volume environments, forward logs to a SIEM.

**Why**: Logs provide the evidence trail for rule matches and are essential for incident response.

**Evidence**: firewall log entries with timestamps and packet details, correlated with pcaps.

[[#Table of Contents 📚]]

---

## 8. VPNs, Tunnels, and Remote Access (OpenVPN, IPsec, WireGuard)

This section covers the most common VPN technologies in pfSense, how to configure them, and how to capture and prove tunnel behavior. Each VPN type includes **what**, **why**, and **how to evidence**.

## 8.1 OpenVPN — server and client modes

**What**: OpenVPN supports TLS‑based tunnels, client‑server or site‑to‑site. pfSense provides a GUI wizard for server setup.

**Why**: OpenVPN is flexible and widely supported. It supports client certificates and TLS auth for strong authentication.

**How to configure (server)**

1. **System → Cert Manager**: create a CA and server certificate.
2. **VPN → OpenVPN → Servers**: create a server, choose `Local User Access` or `RADIUS`, set tunnel network (e.g., 10.8.0.0/24), and enable `Redirect Gateway` if you want full tunnel.
3. **Firewall → NAT → Outbound**: if clients need Internet access via the tunnel, configure outbound NAT for the tunnel network.

**Client configuration**: export client config via **VPN → OpenVPN → Client Export** package.

**Evidence**

- Capture on WAN: TLS handshake to OpenVPN port (1194 UDP/TCP).
- Capture on tunnel interface: decrypted traffic if you have keys (use `openvpn --show-tls` or client logs).
- Logs: **Status → OpenVPN** shows connected clients and bytes transferred.

## 8.2 IPsec — site‑to‑site and roadwarrior

**What**: IPsec is a standards‑based VPN with IKEv1/IKEv2. pfSense supports both and can act as a gateway for site‑to‑site tunnels.

**Why**: IPsec is common in enterprise and cloud interconnects. It supports strong ciphers and is interoperable.

**How to configure**

- **VPN → IPsec → Tunnels**: create Phase 1 (IKE) and Phase 2 (ESP) settings. Use pre‑shared keys or certificates. For site‑to‑site, configure remote gateway and local networks.

**Evidence**

- `tcpdump` on WAN shows IKE (UDP 500/4500) and ESP packets.
- `ipsec status` or **Status → IPsec** shows SA state.
- For forensic proof, capture the IKE exchange and the subsequent ESP traffic; correlate with `ipsec` logs.

## 8.3 WireGuard — lightweight modern VPN

**What**: WireGuard is a modern, fast VPN with simple key management. pfSense supports WireGuard via package.

**Why**: WireGuard offers high performance and simpler configuration for many use cases.

**How to configure**

- Install the WireGuard package, create keys, configure peers, and assign an interface. Use `AllowedIPs` to control routing.

**Evidence**

- `tcpdump` shows UDP packets on the WireGuard port.
- `wg show` (on pfSense shell) shows peer handshake times and transfer counters.

## 8.4 GRE and other tunnels

**What**: GRE encapsulates L3 packets and is useful for routing experiments and hybrid cloud tunnels.

**How to configure**

- Use **Interfaces → Assignments → GIF/TUN/TAP** or configure GRE on the underlying OS if needed. For GRE over IPsec, combine both.

**Evidence**

- Capture on WAN shows GRE encapsulation; capture on the tunnel interface shows decapsulated traffic.

## 8.5 Proving tunnel behavior and routing

**Tests**

- From a client behind the tunnel, ping an internal host across the tunnel and capture on both ends.
- Use `traceroute` to show path through the tunnel.
- For VPN client authentication, capture the TLS/IKE handshake and include server logs showing successful authentication.

**Why**: Proof of tunnel behavior requires both packet captures and server logs to show authentication and data flow.

[[#Table of Contents 📚]]

---

## 9. High Availability, CARP, and Failover

This section explains how to configure pfSense for redundancy and how to validate failover behavior. Each step includes **what** to configure, **why** it matters, and **how** to capture evidence.

## 9.1 CARP basics

**What**: CARP (Common Address Redundancy Protocol) provides a virtual IP shared between two or more pfSense nodes for high availability. One node is MASTER, others are BACKUP.

**Why**: CARP provides seamless failover for gateway IPs and services.

## 9.2 Configuring CARP

**Prerequisites**

- Two pfSense nodes with synchronized interfaces and identical firewall rules.
- A dedicated synchronization interface (optional but recommended) for state sync.

**Steps**

1. **System → High Avail Sync**: configure XMLRPC sync to replicate config and user accounts.
2. **Firewall → Virtual IPs**: create a CARP VIP on the LAN and WAN as needed.
3. **System → Routing → Gateways**: ensure both nodes have the same gateway configuration.
4. **Status → CARP**: monitor CARP status and VHID groups.

**Why**: XMLRPC sync ensures configuration parity; CARP VIPs ensure clients use a single virtual IP.

## 9.3 State synchronization

- **System → High Avail Sync**: enable `pfsync` to synchronize state tables so established connections survive failover.

**Evidence**

- `tcpdump` on the sync interface shows `pfsync` packets.
- `pfctl -s state` on the backup node shows synchronized states after failover.

## 9.4 Testing failover and proving it

**Test plan**

1. Start traffic from a client through the MASTER node to an external service.
2. Force failover by shutting down the MASTER node or disabling the WAN interface.
3. Observe that the BACKUP node becomes MASTER and traffic continues.

**Evidence to collect**

- `tcpdump` on the client showing continuous traffic.
- `tcpdump` on both pfSense nodes showing CARP advertisements and pfsync state transfers.
- `Status → CARP` screenshots showing role change with timestamps.
- `pfctl -s state` before and after failover showing preserved states.

**Why**: Demonstrating failover requires correlated captures and logs to prove continuity and correct role transitions.

[[#Table of Contents 📚]]

---

## 10. Logging, Monitoring, and Forensics (Packet Capture, Suricata, pfSense Logs)

This section is a forensic playbook: how to capture packets, decrypt TLS in lab, run Suricata for IDS/IPS, and package evidence for reports. Each subsection includes **exact commands**, **expected output**, and **how to assemble an evidence package**.

## 10.1 Packet capture methods in pfSense

**Web UI capture**

- **Diagnostics → Packet Capture**: choose interface, host filter, port, and packet count. Click **Start** and download the pcap.

**CLI capture**

- SSH into pfSense and use `tcpdump`:

```bash
# capture full packets on WAN for a specific host
tcpdump -i em0 host 192.168.20.10 -s 0 -w /tmp/wan_capture.pcap
```

**Why**: Web UI is convenient; CLI is more flexible and scriptable.

**Evidence**: Save the pcap with a descriptive filename and compute SHA256 checksum:

```bash
sha256sum /tmp/wan_capture.pcap > wan_capture.pcap.sha256
```

## 10.2 Decrypting TLS in lab

**Browser SSLKEYLOGFILE method**

- On a browser VM, set `SSLKEYLOGFILE` before launching the browser:

```bash
export SSLKEYLOGFILE=/tmp/sslkeys.log
firefox &
```

- In Wireshark, set the TLS pre‑master secret log file to `/tmp/sslkeys.log`.

**Why**: Decrypting TLS is necessary to inspect HTTPS payloads in lab tests such as SSRF or CORS. Only do this in controlled environments.

**Evidence**: Include the `sslkeys.log` (safely stored) and the decrypted Wireshark output showing the HTTP payload.

## 10.3 Suricata IDS/IPS integration

**Installation**

- **System → Package Manager → Available Packages**: install **Suricata**.

**Configuration**

- **Services → Suricata**: add an interface to monitor (e.g., WAN or LAN), enable rulesets (Emerging Threats), and configure logging and EVE JSON output.

**Why**: Suricata provides signature and anomaly detection. In IPS mode, it can block traffic; in IDS mode, it alerts.

**Evidence**

- Suricata alerts in `/var/log/suricata/eve.json`.
- Correlate alert timestamps with pcap captures. Use `jq` to extract relevant alerts:

```bash
jq 'select(.alert != null) | {timestamp: .timestamp, src: .src_ip, dest: .dest_ip, signature: .alert.signature}' /var/log/suricata/eve.json
```

**Forensic workflow**

1. Capture traffic with `tcpdump`.
2. Run Suricata on the pcap or monitor live.
3. Extract alerts and correlate with pcap packet numbers using `tshark` and `frame.time_epoch`.

## 10.4 Log retention and remote logging

**Configure remote syslog**

- **Status → System Logs → Settings**: add remote syslog server IP and facility. Use TLS if supported by your SIEM.

**Why**: Remote logs survive device reboots and are easier to analyze.

**Evidence**: syslog entries with pfSense tags and timestamps. Include SHA256 checksums of log files.

## 10.5 Evidence packaging and manifest

**Minimal evidence package**

- `evidence_slice.pcap` (extracted with `editcap`)
- `packet_123_hex.txt` (tshark hex dump of the key packet)
- `suricata_alerts.json` (filtered alerts)
- `pfSense_config.xml` (exported config)
- `manifest.json` describing the test, commands, packet numbers, timestamps, and SHA256 checksums

**Example manifest fields**

```json
{
  "test_name": "OpenVPN client authentication",
  "hypothesis": "Client can authenticate and route traffic through OpenVPN",
  "commands": ["tcpdump -i em0 port 1194 -w openvpn.pcap"],
  "pcap_file": "openvpn_slice.pcap",
  "evidence_files": [{"file":"openvpn_slice.pcap","packets":[45,46],"timestamps":["2026-06-04T06:12:34Z"]}],
  "confidence": "high"
}
```

**Why**: A manifest makes validation fast and defensible. Include checksums and a short narrative tying artifacts to the claim.

[[#Table of Contents 📚]]

---

## 11. Performance Tuning, Resource Planning, and Troubleshooting

This section is a practical operations manual: how to size pfSense, tune for throughput, and resolve common failures. Each failure includes exact remediation steps and commands to collect evidence.

## 11.1 Resource planning and sizing

**Guidelines**

- **Small office / lab**: 2 cores, 4 GB RAM, 50 GB SSD.
- **Medium**: 4 cores, 8–16 GB RAM, 100+ GB SSD.
- **High throughput / IDS**: 8+ cores, 32+ GB RAM, NVMe, multiple NICs (10 Gbps).

**Why**: Suricata and VPNs are CPU‑bound. Disk matters for logging and Suricata rulesets.

## 11.2 NIC and driver tuning

- Use **Intel** NICs where possible. On virtual platforms, use **virtio** or **VMXNET3**.
- Enable **RSS** (Receive Side Scaling) on NICs if supported to distribute interrupts across cores.

**Commands**

```bash
# check NICs and driver
pciconf -lv
ifconfig -v
```

**Why**: Proper NIC drivers and RSS improve throughput and reduce CPU bottlenecks.

## 11.3 Tuning Suricata and OpenVPN

**Suricata**

- Use AF_PACKET or PF_RING for high performance capture on Linux hosts (not applicable to pfSense CE directly without custom builds). On pfSense, tune Suricata rules and disable expensive rulesets. Use multi‑threading and set appropriate `runmode`.

**OpenVPN**

- Use UDP mode for performance. Use AES‑GCM ciphers for speed and security. Use `tun` devices for routing and avoid `tap` unless L2 bridging is required.

**Why**: Rule complexity and cipher choices directly affect CPU usage.

## 11.4 Common failure modes and remediation

**Failure**: High CPU and dropped packets.  
**Remediation**: Reduce Suricata rules, increase CPU cores, move Suricata to a dedicated sensor, or offload to a separate IDS appliance.

**Failure**: VPN clients disconnect under load.  
**Remediation**: Check MTU and fragmentation. Lower MTU on tunnel interfaces or enable `mssfix` in OpenVPN client config.

**Failure**: CARP failover not preserving states.  
**Remediation**: Ensure `pfsync` is configured on a dedicated sync interface and that firewall rules are identical. Check `pfctl -s state` and `pfsync` logs.

**Commands for troubleshooting**

```bash
# check CPU and memory
top
vmstat 1 5

# check pf state
pfctl -s state

# check Suricata stats (if installed)
cat /var/run/suricata/stats.log

# check OpenVPN status
cat /var/log/openvpn.log
```

## 11.5 Monitoring and alerting

- Use **Netdata**, **Zabbix**, or **Prometheus** exporters for pfSense metrics. For logs, forward to a SIEM and create alerts for high CPU, Suricata critical alerts, or repeated failed logins.

**Why**: Proactive monitoring prevents outages and provides early warning for incidents.

[[#Table of Contents 📚]]

---

## 12. Building Reproducible Labs with pfSense (3 complete labs)

Each lab below is a complete, reproducible scenario with commands, expected results, and evidence packaging instructions.

### Lab 1 — Basic NAT and Port Forwarding (web server in DMZ)

**Goal**: Host a web server in a DMZ and expose it via port forwarding while keeping LAN isolated.

**Topology**

- pfSense with WAN (bridged), LAN (192.168.1.1/24), DMZ (192.168.2.1/24)
- Web server in DMZ at 192.168.2.10

**Steps**

1. Create DMZ interface and assign IP 192.168.2.1/24.
2. Connect web server VM to DMZ network and set IP 192.168.2.10.
3. **Firewall → NAT → Port Forward**: create rule mapping WAN TCP 80 to 192.168.2.10:80. Ensure automatic firewall rule creation or create a matching WAN allow rule.
4. Test from external host: `curl -I http://<wan-ip>`.

**Expected results**

- External request reaches web server and returns HTTP headers.
- `tcpdump` on WAN shows SYN to WAN IP port 80 and forwarded packet to 192.168.2.10.
- Evidence: `wan_capture.pcap` slice showing the inbound SYN and the forwarded request, web server access logs with timestamp, and `pfctl -sr` showing NAT rule.

### Lab 2 — OpenVPN Roadwarrior with Full Tunnel and Logging

**Goal**: Configure OpenVPN server on pfSense, connect a remote client, and route all client traffic through the tunnel. Capture and prove traffic egress via the tunnel.

**Steps**

1. Create CA and server cert in **System → Cert Manager**.
2. **VPN → OpenVPN → Wizards**: create server with tunnel network 10.8.0.0/24 and enable `Redirect Gateway`.
3. Export client config and import into a remote client (Kali VM).
4. Start capture on pfSense WAN and on the OpenVPN interface.
5. From client, `curl -I https://www.example.com` and `ip a` to verify tunnel IP.

**Expected results**

- OpenVPN handshake on WAN (UDP/TCP) and tunnel traffic on `tun` interface.
- `tcpdump` on WAN shows encrypted OpenVPN packets; `tcpdump` on `tun` shows decrypted HTTP traffic (if captured on the server side before encryption).
- Evidence: `openvpn_wan.pcap`, `openvpn_tun.pcap`, client `curl` output, and `Status → OpenVPN` showing client connected.

### Lab 3 — Suricata IDS detecting a simulated exploit

**Goal**: Use Suricata to detect a known signature (e.g., an HTTP exploit string) and produce an alert correlated with a pcap.

**Steps**

1. Install Suricata package and enable monitoring on LAN or DMZ.
2. Enable Emerging Threats ruleset and update rules.
3. From an attacker VM, send a crafted HTTP request that matches a rule (use a safe test signature or `ET INFO` test signature). Example:

```bash
curl -v -H "User-Agent: test-ET-ALERT" http://192.168.2.10/test
```

4. Capture traffic on the interface and collect Suricata alerts.

**Expected results**

- Suricata generates an alert in `eve.json` with signature name and timestamp.
- Pcap contains the HTTP request that triggered the alert.
- Evidence: `suricata_alerts.json` filtered for the test signature, `suricata_capture.pcap` slice, and manifest linking packet numbers to alert entries.

[[#Table of Contents 📚]]

---

## 13. Backups, Upgrades, Snapshots, and Disaster Recovery

This section explains how to back up pfSense configs, upgrade safely, and recover from failures.

## 13.1 Configuration backup and restore

**Web UI**

- **Diagnostics → Backup & Restore**: download `config.xml`. Store it securely and compute SHA256.

**CLI**

```bash
# backup config to /cf/conf/config.xml
cp /cf/conf/config.xml /root/config-$(date +%F).xml
sha256sum /root/config-$(date +%F).xml > /root/config-$(date +%F).xml.sha256
```

**Why**: `config.xml` is the canonical configuration artifact. Keep it encrypted in backups.

## 13.2 Upgrades and testing

- Test upgrades in a staging VM. For production, schedule maintenance windows and snapshot VMs before upgrading. Use **System → Update** and read release notes for breaking changes.

**Why**: Upgrades can change package behavior and break custom configurations.

## 13.3 Disaster recovery

**Bare metal**

- Keep a bootable USB with the pfSense memstick image and a copy of `config.xml`. To recover, reinstall and restore `config.xml`.

**VM**

- Snapshot the VM before major changes. Export the VM or use `rsync` to back up the VM disk images.

**Why**: Fast recovery reduces downtime and preserves evidence for post‑incident analysis.

[[#Table of Contents 📚]]

---

## 14. Security, Hardening, and Operational Hygiene

This section consolidates hardening practices and operational rules you must follow.

**Essential hardening checklist**

- Change default admin password and use strong passphrases.
- Restrict management access to a dedicated management network.
- Use HTTPS with a trusted certificate for the GUI.
- Disable unused services and packages.
- Enforce least privilege in firewall rules and use aliases.
- Enable remote logging to a secure SIEM.
- Keep pfSense and packages up to date; test upgrades in staging.
- Use multi‑factor authentication for remote admin access where possible.
- Encrypt backups and store them offsite.

**Why**: These controls reduce the attack surface and make incidents detectable and recoverable.

[[#Table of Contents 📚]]

---

## 15. Appendix: Useful Commands, Config Snippets, and References

This appendix collects the most used commands, config snippets, and example manifests for quick copy/paste.

### Quick capture and extraction

```bash
# capture on interface
tcpdump -i em0 -s 0 -w /tmp/capture.pcap

# list HTTP URIs and packet numbers
tshark -r /tmp/capture.pcap -Y http.request -T fields -e http.request.full_uri -e frame.number -e frame.time

# extract packet hex
tshark -r /tmp/capture.pcap -Y "frame.number == 123" -x > packet_123_hex.txt

# slice pcap
editcap -r /tmp/capture.pcap /tmp/slice.pcap 100-200

# compute checksum
sha256sum /tmp/slice.pcap > /tmp/slice.pcap.sha256
```

### Useful pfSense CLI snippets

```bash
# show interfaces
ifconfig -a

# show routing table
netstat -rn

# show pf rules
pfctl -sr

# show pf state
pfctl -s state

# show package list
pkg info

# tail system log
clog -f /var/log/system.log
```

### Example firewall NAT snippet (conceptual)

Use the web UI for NAT rules; the underlying `pf` rules are managed by pfSense. For documentation, export the config and include the NAT rule XML snippet.

### Evidence manifest example (JSON)

```json
{
  "test_name": "pfSense OpenVPN full tunnel proof",
  "hypothesis": "Client traffic is routed through pfSense OpenVPN server",
  "commands": [
    "tcpdump -i em0 port 1194 -w openvpn_wan.pcap",
    "tcpdump -i tun0 -w openvpn_tun.pcap"
  ],
  "pcap_file": "openvpn_tun.pcap",
  "evidence_files": [
    {"file":"openvpn_wan.pcap","packets":[45,46],"timestamps":["2026-06-04T07:12:34Z"]},
    {"file":"openvpn_tun.pcap","packets":[12,13],"timestamps":["2026-06-04T07:12:35Z"]}
  ],
  "confidence": "high"
}
```

[[#Table of Contents 📚]]

---

This pfSense Community Edition master guide is designed to be both a teaching manual and a reference. It explains not only the **how** but the **why** behind each configuration and includes forensic workflows for capturing, extracting, and packaging evidence. If you want, I will convert this into a repo‑ready Markdown structure with `README.md`, `manifest.json` templates, and example evidence files (pcap slices and `tshark` outputs) organized into reproducible lab folders. Which lab or subsection would you like me to expand into a full forensic walkthrough next: **OpenVPN evidence extraction**, **Suricata alert correlation**, or **CARP failover timeline**?