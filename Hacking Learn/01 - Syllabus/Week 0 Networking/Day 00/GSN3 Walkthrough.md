### Table of Contents 📚

[[#1. Overview and Goals]]  
[[#2. System Requirements and Prerequisites]]  
[[#3. Downloading and Installing GNS3 (Windows, macOS, Linux)]]  
[[#4. Installing and Configuring the GNS3 VM (VirtualBox / VMware)]]  
[[#5. Integrating QEMU and Adding VMs (Kali, Windows, Ubuntu)]]  
[[#6. Adding Vendor Images (Cisco IOS, IOS‑XE, ASA, JunOS) — Licensing and Legal]]  
[[#7. Building Your First Topology End‑to‑End (Router, Switch, Host, Internet)]]  
[[#8. Advanced Topologies (Firewalls, NAT, Cloud, L2/L3, MPLS)]]  
[[#9. Traffic Capture, Wireshark Integration, and Forensics]]  
[[#10. Performance Tuning, Resource Planning, and Troubleshooting]]  
[[#11. Backups, Snapshots, Exporting Projects, and Reproducible Labs]]  
[[#12. Example Full Lab Walkthroughs (3 complete labs)]]  
[[#13. Security, Hygiene, and Legal Considerations]]  
[[#14. Appendix: Useful Commands, Config Snippets, and References]]

---

# 1. Overview and Goals

This document is a complete, step‑by‑step manual for installing, configuring, and using **GNS3** to build realistic network labs with working virtual machines. The goal is to take you from zero to a fully functioning lab that includes routers, switches, firewalls, Linux and Windows hosts, traffic capture, and Internet access — all reproducible and easy to follow.

What you will get by following this guide:

- A working GNS3 installation with the GNS3 VM integrated.
- QEMU/KVM/VirtualBox VMs (Kali, Ubuntu, Windows) connected to GNS3 topologies.
- Vendor images (Cisco IOS/IOS‑XE/ASA, JunOS) loaded and usable (legal notes included).
- Examples of realistic labs: SSRF/SSHD pivot lab, multi‑site routing, firewall + web app.
- Forensic capture and evidence extraction workflows (Wireshark + tshark + Scapy).
- Performance tuning and troubleshooting checklist so your lab runs reliably.

Why this matters: GNS3 lets you emulate real network behavior with real OS images. That means your experiments are not theoretical — they are reproducible, debuggable, and defensible. Every step below explains **what** you do and **why** it matters.

[[#Table of Contents 📚]]

---

# 2. System Requirements and Prerequisites

This section explains the hardware, host OS, virtualization choices, and software prerequisites. Read it carefully and match your host to the recommended configuration.

Minimum vs recommended hardware

- **Minimum (small labs)**: 8 GB RAM, 4 CPU cores, 50 GB free disk. Use only tiny topologies (a few routers + 1 VM).
- **Recommended (realistic labs)**: 16–32 GB RAM, 6–12 CPU cores, 200+ GB SSD. Use for multi‑VM labs (Kali, Windows, multiple routers).
- **High‑end (large labs)**: 64+ GB RAM, 16+ cores, NVMe storage. Use for many VMs, heavy packet capture, or long‑running labs.

Why: GNS3 runs a GUI plus a VM host (GNS3 VM) that runs QEMU instances. Each router image and VM consumes RAM and CPU. Insufficient resources cause slow boots, dropped packets, and corrupted captures.

Host OS choices and virtualization

- **Windows 10/11**: Use VMware Workstation Player/Pro or VirtualBox. Hyper‑V can conflict; if you use Hyper‑V, install GNS3 with the GNS3 VM in VMware or use the Hyper‑V backend (advanced).
- **macOS**: Use VirtualBox or VMware Fusion. Note: macOS has stricter kernel extension policies; follow vendor docs.
- **Linux (Ubuntu/Debian)**: Best experience. Use KVM/QEMU for performance. Install `libvirt`, `qemu-kvm`, and `virt-manager`. Linux hosts allow direct QEMU integration (faster).

Software prerequisites (install before GNS3)

- Python 3.8+ (GNS3 GUI bundles this on Windows/macOS installers).
- Virtualization platform: VirtualBox or VMware (install latest stable).
- On Linux: `qemu-kvm`, `libvirt-daemon-system`, `libvirt-clients`, `bridge-utils`, `tcpdump`, `wireshark` (for capture).
- GNS3 VM image (downloaded separately).
- Optional: Docker (for container‑based appliances).

Why: GNS3 orchestrates virtual devices using these virtualization backends. KVM/QEMU on Linux is fastest; VirtualBox is easiest cross‑platform.

Network and firewall considerations

- Allow GNS3 GUI and GNS3 VM to communicate over the host network (TCP port 3080 by default).
- If using VMware, ensure the GNS3 VM network adapter is set to NAT or bridged depending on your needs.
- Disable host firewall blocking between GNS3 GUI and GNS3 VM during setup; re‑enable with rules after testing.

Why: GNS3 GUI talks to the GNS3 VM via a REST API. Firewalls or misconfigured NAT will break device startup and image transfers.

[[#Table of Contents 📚]]

---

# 3. Downloading and Installing GNS3 (Windows, macOS, Linux)

This section walks you through obtaining the official GNS3 installer and the GNS3 VM. Every command and UI step includes the reason behind it.

3.1. Download sources and verification

**What to download**

- GNS3 GUI installer for your OS from the official site.
- GNS3 VM OVA (for VMware) or GNS3 VM image for VirtualBox.
- Optional: appliance templates from the GNS3 Marketplace.

**Why**: The GUI alone can run local appliances, but the GNS3 VM provides a controlled environment for QEMU images and improves performance and compatibility.

3.2. Windows installation (step‑by‑step)

**What to do**

1. Download `GNS3-<version>-setup.exe` from the official site.
2. Run the installer as Administrator. Accept the bundled dependencies (WinPcap/Npcap, Wireshark, SolarWinds, etc.) if you want integrated capture. Choose Npcap over WinPcap (Npcap is maintained).
3. Install VirtualBox or VMware if not already installed. If you plan to use VMware Workstation Player, install it now.
4. After GUI install, download the GNS3 VM OVA and import it into VMware Workstation: `File → Open → select GNS3 VM.ova`.
5. Power on the GNS3 VM and note its IP address (console shows it).
6. In GNS3 GUI: `Edit → Preferences → GNS3 VM` → enable and set the VM type (VMware) and the IP address. Click **Test Settings**.

**Why each step matters**

- Running as Administrator ensures the installer can register network drivers and services.
- Npcap enables packet capture; Wireshark integration requires it.
- Importing the GNS3 VM into VMware isolates QEMU processes and avoids permission issues on Windows.

3.3. macOS installation

**What to do**

1. Download the `.dmg` installer and open it. Drag GNS3 to Applications.
2. Install VirtualBox or VMware Fusion. For VirtualBox, allow kernel extensions in System Preferences if macOS blocks them.
3. Import the GNS3 VM OVA into VirtualBox/VMware. Start the VM.
4. In GNS3 GUI: `Preferences → GNS3 VM` → select VirtualBox/VMware and test.

**Why**: macOS requires explicit approval for kernel extensions; missing this step causes VirtualBox to fail.

3.4. Linux installation (Ubuntu example)

**Commands and steps**

```bash
# update and install prerequisites
sudo apt update && sudo apt upgrade -y
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager \
                    python3-pip python3-venv wireshark tcpdump

# add your user to libvirt and kvm groups
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER
```

**Install GNS3 GUI**

```bash
# recommended: use the official PPA (example for Ubuntu)
sudo add-apt-repository ppa:gns3/ppa
sudo apt update
sudo apt install -y gns3-gui gns3-server
```

**Why**: On Linux, using KVM/QEMU is faster and more stable. Adding your user to `libvirt` and `kvm` avoids running GUI as root.

3.5. Post‑install verification

**What to check**

- GNS3 GUI launches and shows the server status as connected.
- GNS3 VM is running and reachable (Test Settings passes).
- Wireshark can capture on host interfaces (run `wireshark` or `sudo tcpdump -D`).

**Why**: Early verification prevents confusing failures later when adding images or starting devices.

[[#Table of Contents 📚]]

---

# 4. Installing and Configuring the GNS3 VM (VirtualBox / VMware)

The GNS3 VM is the recommended runtime for QEMU appliances. This section explains how to import, configure, and tune the VM.

4.1. Importing the GNS3 VM (OVA)

**VirtualBox**

1. `File → Import Appliance` → select `GNS3 VM.ova`.
2. On the appliance settings screen, increase RAM and CPU to match your host (e.g., 8 GB, 4 cores).
3. Set network adapter to `Bridged Adapter` or `Host‑only Adapter` depending on whether you want the VM to have external network access.
4. Finish import and start the VM.

**VMware**

1. `File → Open` → select `GNS3 VM.ova`.
2. Edit VM settings: increase RAM and CPU. Ensure the network adapter is NAT or Bridged as required.
3. Power on the VM.

**Why**: The GNS3 VM runs QEMU instances and provides a consistent environment. Allocating sufficient RAM/CPU prevents resource contention.

4.2. GNS3 VM initial configuration

**Console steps (inside GNS3 VM)**

- Login: default credentials are shown in the VM console (follow the release notes).
- Configure the VM to use DHCP or set a static IP if you prefer. Example to set static IP (Debian/Ubuntu style):

```bash
# edit /etc/netplan/01-netcfg.yaml (example)
sudo nano /etc/netplan/01-netcfg.yaml
# example content:
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
      addresses: [192.168.56.101/24]
      gateway4: 192.168.56.1
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
# apply
sudo netplan apply
```

**Why**: A stable IP makes it easier to configure the GNS3 GUI to connect to the VM.

4.3. Connect GNS3 GUI to the GNS3 VM

**GUI steps**

1. `Edit → Preferences → GNS3 VM`.
2. Enable the GNS3 VM and select the virtualization type (VMware/VirtualBox/KVM).
3. If using remote VM, set the VM IP and port (default 3080). Click **Test Settings**.
4. If the test fails, check firewall rules and ensure the VM is reachable from the host.

**Why**: The GUI uses the VM to spawn QEMU instances. If the GUI cannot reach the VM, devices will not start.

4.4. Shared folders and image transfer

**What to do**

- Use the GNS3 VM’s shared folder feature to transfer large images (IOS, QCOW2) into the VM. In VirtualBox, set a shared folder and mount it inside the VM.
- Alternatively, use `scp` to copy images into the VM.

**Why**: Router images can be large; copying them into the GNS3 VM avoids slow transfers via the GUI.

4.5. Tuning VM resources

**Guidelines**

- For each IOSv/IOSvL2/QEMU VM, allocate at least 512 MB–1 GB RAM depending on image.
- For Windows VMs, allocate 2–4 GB RAM minimum.
- Reserve CPU cores for the host OS; do not allocate all cores to the GNS3 VM.

**Why**: Overcommitting resources causes swapping and packet loss. Plan resource allocation per device.

[[#Table of Contents 📚]]

---

# 5. Integrating QEMU and Adding VMs (Kali, Windows, Ubuntu)

This section shows how to add common VMs to GNS3 and connect them to topologies. Each VM type includes download, install, and expected behavior.

5.1. QEMU vs VirtualBox vs VMware in GNS3

**What to choose**

- **QEMU**: Best for Linux guests and vendor images; integrates tightly with GNS3 VM.
- **VirtualBox**: Good for Windows and GUI Linux guests; easier to import OVA.
- **VMware**: High performance on Windows/macOS; requires VMware Workstation/Fusion.

**Why**: QEMU is the native emulator used by GNS3 VM; VirtualBox/VMware are supported but may require extra configuration.

5.2. Adding a Kali Linux VM (recommended: QEMU)

**Download**

- Get the official Kali QCOW2 image from Offensive Security.

**Import into GNS3**

1. `Edit → Preferences → QEMU → QEMU VMs → New`.
2. Name: `Kali`. Select the QCOW2 image path (copy into GNS3 VM shared folder first).
3. Set RAM (2–4 GB), NICs (1 or 2), and console type (VNC or SPICE).
4. Finish and drag the Kali VM into the topology.

**Boot and verify**

- Start the VM from the topology. Use the GNS3 console (VNC) to log in.
- Verify network connectivity: `ip a`, `ping 8.8.8.8`.

**Why**: Kali provides attacker tools (curl, nmap, tcpdump) for labs. Using QCOW2 reduces disk usage and boots faster.

5.3. Adding a Windows VM (VirtualBox recommended)

**Download**

- Use a Windows evaluation ISO or your licensed image. Convert to VDI or use OVA.

**Import**

1. Create a VirtualBox VM with the Windows image.
2. In GNS3: `Edit → Preferences → VirtualBox VMs → New` and select the VM.
3. Drag into topology and connect to a GNS3 switch or cloud.

**Why**: Windows is useful for testing browser‑based attacks, CORS, and client behavior.

5.4. Adding Ubuntu Server (QEMU or VirtualBox)

**Quick steps**

- Use an Ubuntu cloud image (QCOW2) or ISO. For cloud images, use cloud‑init to set a default user.
- Add as QEMU VM in GNS3 and configure networking.

**Why**: Ubuntu is a lightweight target for web apps and internal services.

5.5. Networking the VMs inside GNS3

**What to do**

- Connect VMs to GNS3 switches or routers using the GUI drag‑and‑drop.
- For Internet access, connect a GNS3 cloud node to your host’s NAT or bridge interface.

**Example: connect Kali to a router**

1. Drag a `Ethernet switch` into the topology.
2. Connect Kali’s NIC to the switch.
3. Connect a router’s interface to the same switch.
4. Start devices and configure IP addresses.

**Why**: GNS3’s virtual switch provides L2 connectivity; routers provide L3 routing and NAT.

5.6. Console access and file transfer

**Console**

- Use the GNS3 console (VNC or telnet) to access VM consoles.
- For SSH, ensure the VM has an IP and `sshd` running.

**File transfer**

- Use `scp` or shared folders to move files into VMs. Example:

```bash
# from host to Kali VM (if SSH reachable)
scp file.txt kali@192.168.20.50:/home/kali/
```

**Why**: Console access is essential for configuration; scp is reliable for evidence artifacts.

[[#Table of Contents 📚]]

---

# 6. Adding Vendor Images (Cisco IOS, IOS‑XE, ASA, JunOS) — Licensing and Legal

This section explains how to legally obtain and add vendor images. It emphasizes licensing and the correct way to add images to GNS3.

6.1. Legal and licensing notes (read this first)

**Key rule**: Do not download vendor images from unauthorized sources. Use vendor evaluation images, your company’s licensed images, or images you are permitted to use.

**Why**: Distributing vendor images is illegal and violates terms of service. Always document the source and license for images used in reports.

6.2. Cisco IOSv / IOSvL2 / IOS‑XE (CSR1000v) in GNS3

**What to do**

- Obtain images from Cisco’s official download portal (requires account and entitlement).
- Copy the image into the GNS3 VM shared folder.
- In GNS3: `Edit → Preferences → Dynamips` for legacy IOS, or `QEMU → QEMU VMs` for IOSv. Add a new appliance and point to the image.

**Example: adding IOSv**

```text
# copy image to GNS3 VM
scp csr1000v-universalk9.16.09.03.qcow2 user@gns3vm:/opt/gns3/images/qemu/
# in GNS3 GUI: add QEMU VM and select the qcow2 image
```

**Why**: IOSv runs under QEMU and provides realistic router behavior for routing labs.

6.3. ASA and Firewalls

**What to do**

- ASA images are available to Cisco customers. ASA 9.x images can be added as QEMU appliances.
- For Palo Alto, FortiGate, and other vendors, use official virtual appliance images and follow vendor docs.

**Why**: Firewalls are essential for realistic segmentation and tunneling labs.

6.4. JunOS and other vendor images

**What to do**

- JunOS vSRX images are available from Juniper with entitlement. Add as QEMU VM.

**Why**: Vendor images provide realistic CLI and feature parity for advanced labs.

6.5. Appliance templates and the GNS3 Marketplace

**What to do**

- Use the GNS3 Marketplace to import appliance templates. These templates include preconfigured settings for memory, NICs, and console types.
- Always verify the image path and license after importing.

**Why**: Marketplace templates speed up setup and reduce configuration errors.

[[#Table of Contents 📚]]

---

## 7. Building Your First Topology End‑to‑End (Router, Switch, Host, Internet) — Deep Dive 🏗️

### Design intent and learning objectives

This section is a surgical walkthrough: you will build a minimal but realistic lab that demonstrates routing, NAT, and host connectivity. The goal is not just to make packets flow; it is to understand every decision that makes them flow, to capture the exact artifacts that prove the behavior, and to be able to explain why a failure occurs and how to fix it. By the end you will be able to reproduce the topology, collect forensic evidence (pcaps, logs, packet hex), and reason about routing, NAT, and troubleshooting.

### Topology overview and why each element exists

The topology contains four logical pieces: a router that performs L3 and NAT, a switch that provides L2 connectivity, an internal host (Kali) that acts as the attacker/target, and a cloud node that represents the host’s NAT/Internet. The router models an edge device in a real network; the switch models the internal LAN; the Kali VM is the internal machine you will test from; the cloud node provides egress to the real Internet or to the host NAT. This arrangement mirrors real-world setups where an internal host uses an edge router to reach external services and where NAT hides internal addresses.

### Step 1 — Create the topology in GNS3 (exact GUI steps and rationale)

Open GNS3 and create a new project. Drag a Cisco IOSv (or a lightweight router image you have) into the workspace. Drag an Ethernet switch and a Kali QEMU VM. Drag a Cloud node and bind it to the host interface that provides NAT or Internet access (on VirtualBox hosts this is often `vboxnet0` or the host’s NAT adapter; on Linux it may be `virbr0` or `eth0`). Connect the router’s Gig0/0 to the Cloud, the router’s Gig0/1 to the switch, and the Kali VM to the switch. Save the project.

Why this exact wiring: connecting the router to the cloud models the WAN interface; connecting the router to the switch models the LAN interface; connecting the host to the switch models a workstation on the LAN. The cloud node is how the virtual lab reaches the outside world without exposing the GNS3 VM to unpredictable routing.

### Step 2 — Allocate resources and start devices (what to set and why)

Before powering on, set the router’s RAM to the recommended value for your image (for IOSv, 1024–2048 MB). Set the Kali VM to 2048–4096 MB depending on your host. Reserve at least one CPU core for each heavy VM but leave cores for the host. Start the GNS3 VM first (if used), then the router, then the switch (switches are virtual and start instantly), then the Kali VM.

Why ordering matters: the GNS3 VM must be available to spawn QEMU processes. Starting the router before the GNS3 VM is ready can cause device startup failures. Starting the host last ensures the network fabric is ready to hand out addresses or accept static configuration.

### Step 3 — Configure the router (precise commands and expected output)

Enter the router console and configure interfaces and NAT. Use the following commands exactly; they are annotated with the expected output and the forensic signals you will later capture.

```text
enable
configure terminal

interface GigabitEthernet0/0
 description WAN-to-Cloud
 ip address dhcp
 no shutdown

interface GigabitEthernet0/1
 description LAN-to-Switch
 ip address 192.168.20.1 255.255.255.0
 no shutdown

ip nat inside source list 1 interface GigabitEthernet0/0 overload
access-list 1 permit 192.168.20.0 0.0.0.255

end
write memory
```

Expected router console output: after `ip address dhcp` you should see `DHCP client bound to <public-ip>` or similar. After `no shutdown` the interface will show `%LINK-3-UPDOWN: Interface GigabitEthernet0/0, changed state to up`. The NAT configuration will be accepted silently; `show ip nat translations` will initially be empty until traffic flows.

Forensic signals to capture: the router will perform source NAT on outbound packets. In your pcap of the cloud interface you will see packets with the router’s external IP as the source. On the LAN pcap you will see the internal host’s private IP. The NAT translation table (`show ip nat translations`) is a textual artifact you should capture as evidence.

### Step 4 — Configure the Kali host (static addressing and tests)

On the Kali VM, set a static IP and default route to the router. Use these commands:

```bash
sudo ip addr flush dev eth0
sudo ip addr add 192.168.20.10/24 dev eth0
sudo ip link set eth0 up
sudo ip route add default via 192.168.20.1
ip -4 addr show dev eth0
ip route show
```

Expected output: `ip -4 addr show` will list `inet 192.168.20.10/24 scope global eth0`. `ip route show` will show `default via 192.168.20.1 dev eth0`. Ping the router to verify L2/L3:

```bash
ping -c 3 192.168.20.1
```

Expected ping output: three replies with low latency. If pings fail, check `show ip interface brief` on the router and `ip link` on Kali for interface state.

Why static addressing: static addresses remove DHCP as a variable during troubleshooting. In production you would use DHCP, but for reproducible labs static addressing is simpler and more deterministic.

### Step 5 — Verify Internet connectivity and capture evidence (tcpdump, tshark)

From Kali, test external connectivity:

```bash
ping -c 3 8.8.8.8
curl -I https://www.example.com
```

Expected results: ICMP replies from 8.8.8.8 and an HTTP 200/301/302 response header from example.com. If `curl` fails with TLS errors, ensure the cloud node provides NAT and DNS resolution.

Capture evidence on the host/cloud interface and on the router (if possible). On the host:

```bash
sudo tcpdump -i <host-interface> host 192.168.20.10 -w /tmp/host_nat_capture.pcap
```

On the GNS3 VM (if accessible):

```bash
sudo tcpdump -i any host 192.168.20.10 -w /tmp/gns3vm_capture.pcap
```

Use `tshark` to extract the HTTP request URIs and packet numbers for reporting:

```bash
tshark -r /tmp/host_nat_capture.pcap -Y http.request -T fields -e http.request.full_uri -e frame.number -e frame.time
```

Expected artifacts: a pcap showing ICMP echo requests from 192.168.20.10 and NATed packets with the router’s external IP. The `tshark` output will give you packet numbers to extract minimal slices with `editcap`.

### Step 6 — Evidence packaging and minimal slices (how and why)

Do not hand over full pcaps. Use `editcap` to extract only the packets that prove the claim. For example, if `tshark` showed the HTTP request at packet 123, extract a small range:

```bash
editcap -r /tmp/host_nat_capture.pcap /tmp/evidence_slice.pcap 120-130
tshark -r /tmp/evidence_slice.pcap -x > evidence_hex.txt
```

Why this matters: small slices reduce exposure of unrelated sensitive traffic and make validation fast for reviewers. Include a JSON manifest that lists the command used, the packet numbers, timestamps, and a short explanation of why the slice proves the claim.

### Troubleshooting checklist (common failures and exact fixes)

If `ping 8.8.8.8` fails, check these in order: verify Kali’s IP and route, verify router interface states (`show ip interface brief`), verify NAT configuration (`show ip nat translations` after traffic), verify cloud binding (is the cloud node bound to the correct host interface?), and check host firewall rules that might block traffic between the GNS3 VM and the host. Use `tcpdump` on both ends to see where packets stop.

[[#Table of Contents 📚]]

---

## 8. Advanced Topologies (Firewalls, NAT, Cloud, L2/L3, MPLS) — Deep Dive 🧭

### Why advanced topologies matter for realistic labs

Real networks are not single‑router labs. They include firewalls, multiple routing domains, L2 fabrics, and cloud interconnects. Understanding how these pieces interact is essential for reproducing complex bug bounty scenarios such as SSRF to internal services behind a firewall, tunneling through NAT, or exploiting misconfigured VRFs. This section explains how to add these elements, why each configuration choice matters, and how to capture the forensic signals that prove behavior.

### Adding a firewall (pfSense example) — what, why, and exact steps

pfSense is a practical, open‑source firewall appliance that models enterprise behavior. Import a pfSense OVA into VirtualBox or VMware and add it to GNS3 as a VirtualBox VM. Give it two NICs: WAN connected to the Cloud (host NAT) and LAN connected to a GNS3 switch.

Inside pfSense, configure the WAN to use DHCP and the LAN to `192.168.30.1/24`. Create a firewall rule on LAN to allow HTTP/HTTPS outbound. Configure NAT if you want the firewall to perform egress NAT for the LAN.

Why this matters: pfSense will enforce stateful firewalling and NAT, which changes how SSRF and tunneling behave. For example, a server behind pfSense may be able to reach metadata endpoints only if firewall rules allow it. Testing with a firewall in place reveals real-world constraints.

Exact pfSense steps (high level):

Start pfSense, log into the web UI, set LAN IP, create a LAN rule to allow `any` to `any` on ports 80/443 for testing, and enable NAT. Save and apply.

Forensic signals: firewall logs show allowed/blocked flows. Capture pfSense logs (`Status → System Logs → Firewall`) and include the log lines that correspond to your test timestamps. In pcaps you will see whether the firewall rewrites source addresses (SNAT) and whether it drops or allows packets.

### Multi‑site routing (OSPF/BGP) — configuration and reasoning

To model multi‑site routing, create two routers representing Site A and Site B and a transit router. Configure OSPF between them to advertise internal networks. Use the following IOS snippet for OSPF:

```text
router ospf 1
 network 10.0.0.0 0.0.0.255 area 0
```

Why OSPF: OSPF demonstrates dynamic route propagation, which matters when you simulate cloud on‑prem routing or when you want to test how an SSRF from one site can reach another site via routing.

Forensic signals: `show ip ospf neighbor` and `show ip route` provide textual evidence of adjacency and learned routes. Capture routing protocol packets with `tcpdump` (OSPF uses IP protocol 89) to show adjacency formation and LSAs.

### MPLS and VRF basics — when to use them and how to configure minimally

MPLS and VRFs are advanced but important for service provider or multi‑tenant labs. Use IOS images that support MPLS. Configure LDP and enable MPLS on transit interfaces:

```text
mpls ip
interface GigabitEthernet0/0
 mpls ip
```

Create VRFs to isolate tenant networks:

```text
ip vrf TENANT_A
 rd 100:1
 route-target export 100:1
 route-target import 100:1

interface GigabitEthernet0/1
 ip vrf forwarding TENANT_A
 ip address 10.10.1.1 255.255.255.0
```

Why VRFs: VRFs model tenant isolation; they are useful when you want to simulate how a misconfiguration could leak internal routes or allow cross‑tenant access.

Forensic signals: `show ip vrf` and `show ip route vrf TENANT_A` show VRF state. MPLS labels appear in packet captures as shim headers; Wireshark can decode MPLS labels and show the label stack, which is strong evidence of MPLS forwarding.

### Cloud integration (VPN/GRE to cloud VM) — exact steps and rationale

To simulate hybrid cloud, create a cloud VM (a real VM in AWS/GCP or a local VM that represents a cloud endpoint) and establish an IPsec or GRE tunnel from your GNS3 router to the cloud VM. For a GRE tunnel:

On the GNS3 router:

```text
interface Tunnel0
 ip address 10.255.255.1 255.255.255.252
 tunnel source <router-wan-ip>
 tunnel destination <cloud-vm-ip>
```

On the cloud VM (Linux):

```bash
sudo ip tunnel add tun0 mode gre remote <router-wan-ip> local <cloud-vm-ip> ttl 255
sudo ip addr add 10.255.255.2/30 dev tun0
sudo ip link set tun0 up
```

Why: GRE/IPsec tunnels model real hybrid connectivity and are essential when testing SSRF that needs to reach cloud resources or when testing metadata access across NAT boundaries.

Forensic signals: tunnel packets are visible on the WAN interface; inside the tunnel you will see encapsulated packets. Use `tcpdump -i <wan-if>` to capture encapsulated GRE packets and `tcpdump -i tun0` on the cloud VM to capture decapsulated traffic.

### Advanced troubleshooting scenarios and how to prove them

If a tunnel fails, capture both the WAN interface and the tunnel interface. Look for ICMP unreachable messages, TTL expiry, or mismatched MTU causing fragmentation. Use `show ip interface brief`, `show ip route`, and `show crypto isakmp sa` (for IPsec) to gather textual evidence. Correlate timestamps between pcaps and device logs to build a timeline.

[[#Table of Contents 📚]]

---

## 9. Traffic Capture, Wireshark Integration, and Forensics — Deep Dive 🔬

### Capture strategy: where to capture and why it matters

Capturing traffic is not just about pressing “Start capture.” The capture location determines what you can prove. Capture on the link that best demonstrates the claim. Capture on a GNS3 link (right‑click a link → Start capture) yields minimal, precise data. Capture on the host interface yields egress evidence. Capture inside VMs (tcpdump) yields the original packet before NAT or encapsulation. Always start captures before you run the probe and stop them immediately after you have the evidence to minimize unrelated data.

### TLS decryption in lab (step‑by‑step and why it is safe in lab)

To inspect HTTPS payloads in a lab, use `SSLKEYLOGFILE`. On a Linux or Windows browser VM, set the environment variable before launching the browser so the browser writes pre‑master secrets:

On Linux:

```bash
export SSLKEYLOGFILE=/tmp/sslkeys.log
firefox &
```

On Windows, set the environment variable in the system or launch the browser from a PowerShell session with the variable set. In Wireshark, go to `Edit → Preferences → Protocols → TLS` and set the `(Pre)-Master-Secret log filename` to the path of `sslkeys.log`. Wireshark will then decrypt TLS sessions that used those keys.

Why this is safe: you only decrypt sessions in a controlled environment where you control the client. Never attempt to decrypt third‑party traffic without authorization. The decrypted payloads are essential for proving API calls, tokens, or JSON bodies in SSRF/CORS tests.

### Using tshark and Scapy for automated evidence extraction (examples and rationale)

`tshark` is scriptable and ideal for triage. To list all HTTP URIs and packet numbers:

```bash
tshark -r /tmp/capture.pcap -Y http.request -T fields -e http.request.full_uri -e frame.number -e frame.time > http_requests.txt
```

This gives you a compact index to the packets you need. To extract a packet hex dump:

```bash
tshark -r /tmp/capture.pcap -Y "frame.number == 123" -x > packet_123_hex.txt
```

Scapy is useful for byte‑level analysis and for extracting offsets of headers such as `Authorization:`. Example script:

```python
from scapy.all import rdpcap, TCP, Raw, IP
pcap = rdpcap('/tmp/capture.pcap')
for i, pkt in enumerate(pcap, start=1):
    if pkt.haslayer(TCP) and pkt.haslayer(Raw) and pkt.haslayer(IP):
        payload = bytes(pkt[Raw].load)
        idx = payload.find(b"Authorization:")
        if idx != -1:
            print(f"Packet {i} time {pkt.time} Authorization at offset {idx}")
```

Why programmatic extraction: reviewers want minimal artifacts. Scripts let you produce exact byte ranges and small files that contain only the proof, not the entire capture.

### Evidence packaging: manifest, minimal slices, and narrative (how to assemble)

A high‑quality evidence package contains three parts: a narrative, reproducible steps, and artifacts. The narrative explains the hypothesis and the impact in plain language. The reproducible steps list the exact commands and the stop condition. The artifacts include a minimal pcap slice, a small text/binary file with the exact bytes, and a JSON manifest.

Example manifest fields: `test_name`, `hypothesis`, `commands`, `pcap_file`, `evidence_files` (with packet numbers and byte offsets), `timestamps`, `confidence`, `remediation_suggestion`. Include checksums (SHA256) for each artifact.

Why this matters: defenders can validate quickly without sifting through large pcaps. The manifest is the index that makes your findings reproducible and defensible.

### Forensic timeline construction (how to correlate logs, pcaps, and device outputs)

Build a timeline by aligning timestamps from three sources: device logs (router/firewall), pcap timestamps, and server logs (attacker server or target app). Ensure all systems use NTP or note clock offsets. Use `tshark -r capture.pcap -T fields -e frame.number -e frame.time_epoch` to get epoch timestamps and align them with log timestamps. Document any clock skew and include it in the manifest.

Why: a timeline proves causality (e.g., the SSRF request at 12:00:05 caused the outbound fetch at 12:00:06). Without aligned timestamps, defenders may dispute your claim.

[[#Table of Contents 📚]]

---

## 10. Performance Tuning, Resource Planning, and Troubleshooting — Deep Dive ⚙️

### Resource planning: how to size a host for predictable labs

Estimate RAM and CPU per device. Use this conservative formula: each IOSv router 512–1024 MB, each Linux VM 2–4 GB, each Windows VM 4–8 GB, GNS3 VM overhead 2–4 GB. Add 20–30% headroom for the host OS and background processes. For example, a lab with 3 routers, 1 Kali, and 1 Windows VM requires roughly 3×1GB + 3GB + 6GB + 3GB = 15GB; a 24GB host is recommended.

Why headroom matters: swapping kills packet timing and causes dropped packets and corrupted captures. Packet captures are sensitive to timing; if the host swaps, timestamps and packet order can be unreliable.

### CPU and I/O tuning: practical knobs and why they matter

On Linux hosts, use KVM/QEMU with virtio drivers for VMs to reduce CPU overhead. Use SSD/NVMe storage for images and pcaps; spinning disks cause I/O bottlenecks during heavy captures. On Windows hosts, prefer VMware Workstation for performance; ensure VMware Tools/Guest Additions are installed in VMs.

Why: packet processing is CPU and I/O intensive. Slow disk writes during capture cause packet loss. virtio and paravirtualized drivers reduce overhead and improve throughput.

### GNS3 VM tuning: exact settings that improve stability

Increase the GNS3 VM’s RAM to at least 4 GB for medium labs. Set the VM to use a bridged adapter if you need the VM to be on the same network as the host. In VirtualBox, enable I/O APIC and nested paging. In VMware, enable virtualization extensions and set the VM to use multiple cores if your host has them.

Why: the GNS3 VM spawns QEMU processes; insufficient VM resources cause device startup failures and slow consoles.

### Common failure modes and precise remediation steps

Failure: “Device fails to start: QEMU process exited.”  
Remediation: check GNS3 VM connection (`Edit → Preferences → GNS3 VM → Test Settings`). Ensure the image path is correct and the image has correct permissions inside the GNS3 VM. Check `/var/log/gns3/` on the GNS3 VM for error messages. If the image is corrupted, re‑copy it into the VM.

Failure: “High packet loss in captures.”  
Remediation: move captures to the GNS3 VM or capture on the VM rather than the host. Reduce the number of devices or increase host RAM/CPU. Use `tcpdump -s 0 -w` to capture full packets and avoid Wireshark GUI overhead during capture.

Failure: “Slow console or unresponsive GUI.”  
Remediation: disable excessive logging in GNS3 (`Edit → Preferences → Server → Logging`), close unused projects, and increase the GNS3 VM CPU allocation. On Windows, ensure Hyper‑V is disabled if using VirtualBox (Hyper‑V can cause slowdowns).

Failure: “DNS resolution fails inside VMs.”  
Remediation: check the cloud binding and the DNS settings on the GNS3 VM and host. If using host NAT, ensure the host’s DNS resolver is reachable from the GNS3 VM. Use `dig` or `nslookup` inside the VM to test.

### Monitoring and metrics: what to watch during long runs

Monitor host CPU, RAM, and disk I/O. On Linux use `htop`, `iostat`, and `free -m`. On Windows use Task Manager and Resource Monitor. Monitor packet drops in `tcpdump` output (`tcpdump` prints a summary with dropped packets when it exits). Monitor GNS3 server logs for repeated errors.

Why: long labs accumulate resource pressure. Monitoring lets you preemptively scale resources or split the lab across multiple hosts.

### Backup and recovery: how to avoid losing work

Use GNS3 project export regularly. Snapshot critical VMs before destructive tests. Keep a copy of all images and a `README` that lists image versions and checksums. Use `rsync` or cloud backup for your `~/GNS3` directory.

Why: images and pcaps are large; losing them wastes time. Snapshots let you revert quickly after a destructive test.

### Performance checklist (quick reference for troubleshooting)

First, verify device states and interfaces. Second, verify resource usage (CPU/RAM/disk). Third, capture minimal pcaps to see where packets stop. Fourth, check GNS3 VM connectivity and image paths. Fifth, consult logs (`~/.config/GNS3/gns3_server.log` and GNS3 VM logs). Sixth, reduce topology complexity and reintroduce devices incrementally.

Why a checklist: complex labs fail for simple reasons. A consistent checklist reduces time to resolution and produces reproducible fixes.

[[#Table of Contents 📚]]

---

# 11. Backups, Snapshots, Exporting Projects, and Reproducible Labs

This section explains how to make your labs shareable and reproducible.

11.1. Project export

**What to do**

- `File → Export project` in GNS3. This creates a `.gns3project` archive with topology and references.
- Include the images used (or list their sources and checksums) in a `README.md`.

**Why**: Exporting ensures others can reproduce the lab.

11.2. Snapshots and VM snapshots

- Use VirtualBox/VMware snapshots for Windows and pfSense VMs.
- For QEMU images, use QCOW2 snapshots or copy the image file.

**Why**: Snapshots let you revert to a known good state after destructive tests.

11.3. Version control for labs

- Keep a Git repo with `README.md`, `topology.png`, `commands.txt`, and a manifest listing images and checksums.
- Do not commit vendor images to Git; instead, include download instructions and checksums.

**Why**: Version control makes labs auditable and reproducible.

[[#Table of Contents 📚]]

---

# 12. Example Full Lab Walkthroughs (3 complete labs)

This section contains three end‑to‑end labs with expected outputs and evidence artifacts.

12.1. Lab A — SSRF pivot to internal web admin (complete)

**Goal**: Demonstrate SSRF that causes a server to fetch an internal admin page.

**Setup summary**

- Target web app on `vulnerable.example.com` running in GNS3 internal network.
- Internal admin at `http://192.168.20.50/admin`.
- Attacker server logs requests.

**Steps**

1. Start attacker HTTP server: `python3 -m http.server 8000`.
2. Trigger SSRF: `curl "http://vulnerable.example.com/fetch?url=http://192.168.20.50/admin?probe=MARKER123"`.
3. Capture on attacker server: `sudo tcpdump -i eth0 -w /tmp/ssrf_attacker.pcap port 8000`.
4. Evidence: attacker server log line with `MARKER123`, pcap slice showing request, timestamp.

**Expected artifacts**

- `ssrf_attacker.pcap` containing HTTP GET with `MARKER123`.
- `manifest.json` listing commands and packet numbers.

12.2. Lab B — DNS rebinding to access internal API

**Goal**: Show browser can access internal API after DNS flip.

**Steps**

1. Configure attacker DNS with short TTL and alternate A records.
2. Serve JS that flips DNS and issues fetches.
3. Capture DNS and HTTP traffic: `sudo tcpdump -i eth0 -w /tmp/rebind.pcap port 53 or port 80`.
4. Evidence: DNS query/response pairs and browser request to internal IP with Host header set to attacker domain.

12.3. Lab C — SSH SOCKS tunnel bypassing firewall

**Goal**: Use SSH dynamic SOCKS to access blocked internal web service.

**Steps**

1. `ssh -D 9050 user@192.168.20.10` (SSH server inside target network).
2. Configure browser to use `SOCKS5 127.0.0.1:9050`.
3. Access `http://internal.blocked.local`.
4. Capture on SSH server: `sudo tcpdump -i eth0 -w /tmp/socks_server.pcap`.
5. Evidence: pcap showing outbound HTTP from SSH server to `internal.blocked.local`.

[[#Table of Contents 📚]]

---

# 13. Security, Hygiene, and Legal Considerations

This section is non‑negotiable. Follow it.

- Always obtain explicit authorization before testing networks you do not own.
- Do not exfiltrate real user data. Use synthetic test data.
- Keep evidence packages minimal and encrypted when sharing.
- Document scope, time windows, and contact points before testing.
- For vendor images, follow licensing and do not redistribute images.

Why: Ethical and legal compliance protects you and your organization.

[[#Table of Contents 📚]]

---

# 14. Appendix: Useful Commands, Config Snippets, and References

This appendix collects the most used commands and snippets for quick copy/paste.

14.1. Quick capture and extraction

```bash
# capture on interface
sudo tcpdump -i eth0 -w /tmp/capture.pcap

# list HTTP URIs and packet numbers
tshark -r /tmp/capture.pcap -Y http.request -T fields -e http.request.full_uri -e frame.number -e frame.time

# extract packet hex
tshark -r /tmp/capture.pcap -Y "frame.number == 123" -x > packet_123_hex.txt

# slice pcap
editcap -r /tmp/capture.pcap /tmp/slice.pcap 100-200
```

14.2. Useful router snippets

```text
# basic interface
interface GigabitEthernet0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown

# NAT
ip nat inside source list 1 interface GigabitEthernet0/0 overload
access-list 1 permit 192.168.20.0 0.0.0.255
```

14.3. Evidence manifest example (JSON)

```json
{
  "test_name": "SSRF to internal admin",
  "hypothesis": "Target will fetch attacker URL from internal network",
  "commands": [
    "python3 -m http.server 8000",
    "curl \"http://vulnerable.example.com/fetch?url=http://192.168.20.50/admin?probe=MARKER123\""
  ],
  "pcap_file": "ssrf_attacker.pcap",
  "evidence_files": [
    {"file":"ssrf_attacker.pcap","packets":[12],"timestamps":["2026-06-04T05:12:34Z"]}
  ],
  "confidence": "high"
}
```

[[#Table of Contents 📚]]

---

This merged guide contains the full GNS3 installation and configuration instructions plus deep, forensic‑grade walkthroughs for building topologies, advanced labs, capture and evidence workflows, and performance tuning. It preserves every step, command, and rationale from the previous sections and expands the practical, forensic, and troubleshooting detail for sections 7–10 so you can build, test, and prove network behaviors reliably.

If you want, I will convert this into a repo‑ready Markdown structure with `README.md`, `manifest.json` templates, and example evidence files (pcap slices and `tshark` outputs) organized into a reproducible lab folder.