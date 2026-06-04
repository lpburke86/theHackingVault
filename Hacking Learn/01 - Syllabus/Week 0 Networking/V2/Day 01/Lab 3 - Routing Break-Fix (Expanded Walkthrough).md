# 🧭 Table of Contents (Heading Links)

- [[# 1. Lab overview — understanding movement at layer 3 ⚔️]]
- [[# 2. Learning goals — what this lab teaches you 🎯]]
- [[# 3. Environment, topology, and preparation 🧰]]
- [[# 4. Deep theory — routing as local decisions creating global behavior 🔬]]
- [[# 5. Designing the experiment — how to break the internet safely 🧪]]
- [[# 6. Walkthrough part 1 — capturing normal routing behavior 🛠️]]
- [[# 7. Walkthrough part 2 — deleting the default route (the break) 💥]]
- [[# 8. Walkthrough part 3 — restoring the route (the fix) 🔧]]
- [[# 9. Packet analysis — what broken routing looks like in a pcap 📡]]
- [[# 10. Artifact strategy — filenames, manifests, and narrative structure 📁]]
- [[# 11. Failure modes, debugging, and deeper reasoning 🧰]]
- [[# 12. Cleanup, packaging, and verification 📦]]
- [[# 13. Reflection prompts — internalizing routing intuition 🤔]]

---

## 1. Lab overview — understanding movement at layer 3 ⚔️

Routing is the part of networking that most people take for granted because it works so reliably that it becomes invisible. But the moment routing breaks, the illusion collapses and you see how dependent every higher‑level protocol is on a single, simple rule: _where should this packet go next?_ This lab is designed to make routing visible by breaking it on purpose, watching the consequences unfold in real time, and then restoring it. You will see packets that leave a host and never return, not because the network is down, but because the host has lost its sense of direction.

This lab is not about memorizing commands. It is about understanding that routing is a belief system. A host does not know the internet. It only knows what its routing table tells it. When you delete the default route, you are not destroying connectivity — you are destroying the host’s _model_ of connectivity. The network still exists. The router still routes. But the host becomes blind. This lab teaches you to see that blindness.

[[#🧭 Table of Contents]]

---

## 2. Learning goals — what this lab teaches you 🎯

By the end of this lab, you should be able to articulate why routing is fundamentally a local decision. You should be able to explain why a host without a default route cannot reach anything beyond its own subnet, even though the router is still functioning perfectly. You should be able to demonstrate, with packet captures, the difference between a successful ICMP exchange and a failed one caused by missing routing information.

You will also learn how to restore routing, verify that the fix worked, and document the entire process with timestamped artifacts. This lab builds intuition that will serve you in real assessments, where routing failures often masquerade as firewall issues, DNS problems, or application bugs.

[[#🧭 Table of Contents]]

---

## 3. Environment, topology, and preparation 🧰

You will use the same three‑node topology as the previous labs: Kali at `192.168.20.10`, pfSense at `192.168.20.1`, and Ubuntu at `192.168.20.20`. Ubuntu is the host whose routing table you will manipulate. pfSense is the router that Ubuntu normally uses as its default gateway. Kali is your observer and secondary test host.

Before you begin, create a directory structure for Lab 3:

```bash
mkdir -p ~/day1_lab3/{pcaps,scans,notes,screenshots,artifacts}
date -u > ~/day1_lab3/scans/lab3_start_time_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

Record Ubuntu’s current routing table:

```bash
ip route show > ~/day1_lab3/scans/ubuntu_route_before_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

This file is your baseline truth. It shows the rule that makes all movement possible.

[[#🧭 Table of Contents]]

---

## 4. Deep theory — routing as local decisions creating global behavior 🔬

Routing is often misunderstood as a global map of the network, but that is not how real hosts behave. A host does not know the entire topology. It does not know where pfSense sends packets next. It does not know how many routers exist between itself and a destination. It only knows its own routing table — a small set of rules that tell it what to do with packets based on their destination IP.

When Ubuntu wants to send a packet to `192.168.20.10`, it recognizes that the destination is on the same subnet, so it sends the packet directly to Kali’s MAC address. No routing is needed. But when Ubuntu wants to send a packet to `8.8.8.8`, it recognizes that the destination is not on its local subnet. It must send the packet to a router — its default gateway. The default route is the rule that says: “if the destination does not match any more specific rule, send it to 192.168.20.1.”

If you delete that rule, Ubuntu has no idea where to send packets that are not on its local subnet. It will still try to send them, but it will not know the next hop. The packets will either be dropped immediately or will be sent to the wrong place. In either case, they will not reach their destination, and no replies will return.

This lab makes that failure visible. You will see packets leave Ubuntu and then disappear. You will see ICMP echo requests with no echo replies. You will see ARP requests that never lead to successful communication. You will see the absence of routing as a concrete, observable phenomenon.

[[#🧭 Table of Contents]]

---

## 5. Designing the experiment — how to break the internet safely 🧪

The experiment has three phases: a baseline phase where you capture normal routing behavior, a break phase where you delete the default route and observe failure, and a fix phase where you restore the route and observe recovery. In the baseline phase, you will capture ICMP traffic as Ubuntu pings an external IP or the gateway. This gives you a reference point for what normal routing looks like.

In the break phase, you will delete the default route on Ubuntu and then attempt the same ping. The packets will leave Ubuntu but will not return. You will capture this behavior in a pcap. In the fix phase, you will restore the default route and repeat the ping. Connectivity will return, and you will capture that as well.

Throughout the experiment, you will save every command output and every pcap with ISO‑timestamped filenames. At the end, you will have a complete record of the break and fix.

[[#🧭 Table of Contents]]

---

## 6. Walkthrough part 1 — capturing normal routing behavior 🛠️

Before you can meaningfully break routing, you need a clean, unambiguous picture of what _working_ routing looks like. This is the baseline against which all later failures will be compared. When Ubuntu sends packets to a destination outside its subnet, it relies entirely on its routing table to decide where those packets should go. The default route — the line that reads `default via 192.168.20.1 dev eth0` — is Ubuntu’s way of saying, “I don’t know where this IP lives, so I will hand the packet to pfSense and trust it to figure it out.” This is the essence of Layer 3 delegation and the core behavior you are trying to see with your own eyes.

To capture this behavior, you begin by placing a packet sniffer on a vantage point that can see traffic flowing between Ubuntu and pfSense. This is usually the LAN interface of pfSense or a Linux VM bridged into the same network. When you start tcpdump with a filter for ICMP and IP traffic, you are essentially placing a stethoscope on the artery of the network, listening for the heartbeat of normal communication. Every ICMP echo request and reply that crosses that link is a tiny proof that routing is working as intended.

Start the capture so that every packet in this “healthy” phase is recorded:

```bash
sudo tcpdump -i eth0 -w ~/day1_lab3/pcaps/routing_baseline_$(date -u +"%Y%m%dT%H%M%SZ").pcap 'icmp or ip' &
echo $! > ~/day1_lab3/artifacts/tcpdump_routing_baseline_pid_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

Now you need to generate traffic that forces Ubuntu to consult its routing table. When you run a ping to an address outside the local subnet, Ubuntu cannot simply ARP for that IP and send directly. It must look at its routing table, realize that the destination is “somewhere else,” and hand the packet to its default gateway. That decision — “send this to 192.168.20.1” — is exactly what you want to see reflected in the pcap.

Trigger that behavior with a ping:

```bash
ping -c 4 8.8.8.8 | tee ~/day1_lab3/scans/ubuntu_ping_baseline_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

If your lab does not have external connectivity, you can instead ping pfSense itself; the key is that packets leave Ubuntu and traverse the link you are capturing on. In the pcap, you will later see ICMP echo requests leaving Ubuntu and echo replies returning, all wrapped in IP packets that move through pfSense.

When you stop the capture, you are freezing this healthy state in time:

```bash
kill $(cat ~/day1_lab3/artifacts/tcpdump_routing_baseline_pid_*.txt) || true
```

This baseline pcap is your control sample — the version of reality before you introduce any faults. It is the “this is what right looks like” reference that makes the broken and restored states meaningful.

[[#🧭 Table of Contents]]

---

## 7. Walkthrough part 2 — deleting the default route (the break) 💥

Now that you have captured normal routing behavior, you are ready to break it. Deleting the default route is not a dramatic act in terms of commands — it is a single line — but conceptually it is huge. You are removing the only instruction Ubuntu has for reaching anything beyond its own subnet. Without that line, Ubuntu becomes a creature of pure locality. It can still talk to neighbors on `192.168.20.0/24`, but the moment it wants to reach something else, it has no idea where to send the packet.

Before you cut that line, you record the routing table again so you have a “just before surgery” snapshot:

```bash
ip route show > ~/day1_lab3/scans/ubuntu_route_before_delete_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

This file will later let you compare the intact and broken states side by side and see exactly what changed.

When you run the delete command, you are not touching pfSense, you are not touching Kali, and you are not touching any cables. You are changing only Ubuntu’s internal belief about where to send packets:

```bash
sudo ip route del default
```

Immediately after, you save the new routing table:

```bash
ip route show > ~/day1_lab3/scans/ubuntu_route_after_delete_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

When you open this file, the absence of the `default via 192.168.20.1` line is the entire story: Ubuntu no longer has a “catch‑all” rule for unknown destinations. Every packet to an off‑subnet IP is now an unsolved problem.

To make this failure visible on the wire, you start a new capture. This pcap will represent the “broken” phase:

```bash
sudo tcpdump -i eth0 -w ~/day1_lab3/pcaps/routing_broken_$(date -u +"%Y%m%dT%H%M%SZ").pcap 'icmp or ip' &
echo $! > ~/day1_lab3/artifacts/tcpdump_routing_broken_pid_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

Now you repeat the exact same ping as before. This is important: by keeping the test identical, you ensure that any difference in behavior is due solely to the routing change, not to a different destination or protocol.

```bash
ping -c 4 8.8.8.8 | tee ~/day1_lab3/scans/ubuntu_ping_after_delete_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

From Ubuntu’s perspective, it is still trying to send ICMP echo requests. But when the kernel looks up the route for `8.8.8.8`, it finds nothing. Depending on the implementation, it may drop the packet immediately and report “Network is unreachable,” or it may attempt some fallback behavior. In any case, the key outcome is that no valid next hop is chosen, so no valid path to the destination exists.

When you stop the capture:

```bash
kill $(cat ~/day1_lab3/artifacts/tcpdump_routing_broken_pid_*.txt) || true
```

you have frozen the broken state in time. Later, when you open this pcap in Wireshark, you will see either no ICMP traffic at all (if the kernel refused to send) or one‑sided traffic that never completes. That asymmetry — requests without replies, or no requests at all — is the concrete manifestation of “no route to host.”

[[#🧭 Table of Contents]]

---

## 8. Walkthrough part 3 — restoring the route (the fix) 🔧

Restoring the default route is the moment where the network comes back to life from Ubuntu’s point of view. When you add the route back with `ip route add default via 192.168.20.1`, you are giving Ubuntu back its sense of direction. Suddenly, the host once again has an answer to the question, “where should I send packets for destinations I don’t explicitly know?”

You perform the restoration with a single command:

```bash
sudo ip route add default via 192.168.20.1
```

Immediately after, you capture the new routing table:

```bash
ip route show > ~/day1_lab3/scans/ubuntu_route_after_restore_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

When you compare this file to the “after delete” version, you will see that the default route line has returned. That one line is the difference between isolation and participation in the wider network.

To observe the recovery on the wire, you start a final capture. This pcap will represent the “healed” phase:

```bash
sudo tcpdump -i eth0 -w ~/day1_lab3/pcaps/routing_restored_$(date -u +"%Y%m%dT%H%M%SZ").pcap 'icmp or ip' &
echo $! > ~/day1_lab3/artifacts/tcpdump_routing_restored_pid_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

Then you run the same ping yet again:

```bash
ping -c 4 8.8.8.8 | tee ~/day1_lab3/scans/ubuntu_ping_after_restore_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

This time, the kernel looks up the route for `8.8.8.8`, finds the default route, and forwards the packet to pfSense. In the pcap, you will later see ICMP echo requests leaving Ubuntu and echo replies returning, just like in the baseline capture. The network path has not changed; only Ubuntu’s belief about that path was broken and then repaired.

When you stop the capture:

```bash
kill $(cat ~/day1_lab3/artifacts/tcpdump_routing_restored_pid_*.txt) || true
```

you now have a complete break/fix sequence: a healthy pcap, a broken pcap, and a restored pcap, each tied to routing table snapshots and ping outputs. Together, they tell a full story of how a single routing rule can make or break connectivity.

[[#🧭 Table of Contents]]

---

## 9. Packet analysis — what broken routing looks like in a pcap 📡

Open the broken routing pcap in Wireshark and apply the filter `icmp`. In a clean break scenario where the kernel refuses to send packets without a route, you may see no ICMP traffic at all — the silence itself is evidence that the host had no path. In other cases, you might see echo requests leaving but never see echo replies returning. That asymmetry is the signature of broken routing: the conversation starts but never completes.

Compare this with the baseline and restored pcaps. In those, you should see echo request and echo reply pairs, with matching identifiers and sequence numbers. The contrast between “request + reply” and “request only” (or “nothing at all”) is what turns an abstract “no route” error into something you can literally see on the wire.

Take screenshots of these patterns and save them with ISO timestamps so you can reference them later in notes or reports.

[[#🧭 Table of Contents]]

---

## 10. Artifact strategy — filenames, manifests, and narrative structure 📁

Your manifest should tell the entire story in chronological order, so that someone else could reconstruct the lab from your notes alone. Each line should capture what you did, when you did it, where the output lives, and what it means in one sentence.

For example:

```text
2026-06-03T03:10Z | ip route show | ~/day1_lab3/scans/ubuntu_route_before_delete_20260603T031000Z.txt | Baseline routing table recorded
2026-06-03T03:12Z | sudo ip route del default | ~/day1_lab3/scans/ubuntu_route_after_delete_20260603T031200Z.txt | Default route removed; host cannot reach external addresses
2026-06-03T03:13Z | ping -c 4 8.8.8.8 | ~/day1_lab3/scans/ubuntu_ping_after_delete_20260603T031300Z.txt | ICMP echo requests attempted; failure due to missing route
2026-06-03T03:15Z | sudo ip route add default via 192.168.20.1 | ~/day1_lab3/scans/ubuntu_route_after_restore_20260603T031500Z.txt | Default route restored
2026-06-03T03:16Z | ping -c 4 8.8.8.8 | ~/day1_lab3/scans/ubuntu_ping_after_restore_20260603T031600Z.txt | ICMP echo requests and replies observed; connectivity restored
```

This manifest, combined with your pcaps and screenshots, becomes a self‑contained narrative of the lab.

[[#🧭 Table of Contents]]

---

## 11. Failure modes, debugging, and deeper reasoning 🧰

If the ping still works after deleting the default route, the most likely explanation is that you are pinging something on the same subnet. In that case, no routing is needed, so the missing default route has no effect. Switch to an off‑subnet IP to force routing.

If the route does not restore after you add it back, check `ip addr show` to ensure the interface is still up and has the expected IP. If the pcap shows replies even when routing is supposedly broken, you may be capturing on the wrong interface or misinterpreting which host is sending what.

Every time something does not behave as expected, treat it as a chance to refine your mental model. Save the outputs of your debugging commands and add them to your manifest; even your mistakes become part of the learning story.

[[#🧭 Table of Contents]]

---

## 12. Cleanup, packaging, and verification 📦

When you are satisfied with your observations, package everything so you can archive or submit it.

Create a zip archive of the entire lab directory:

```bash
zip -r ~/day1_lab3/artifacts/day1_lab3_package_$(date -u +"%Y%m%dT%H%M%SZ").zip ~/day1_lab3/
```

Create a small README that tells a future you (or a reviewer) exactly how to verify your claims:

```bash
cat > ~/day1_lab3/README_verify_$(date -u +"%Y%m%dT%H%M%SZ").md << 'EOF'
Open routing_baseline_*.pcap and routing_restored_*.pcap.
Filter: icmp
Observe: Echo Request/Reply pairs between Ubuntu and destination.

Open routing_broken_*.pcap.
Filter: icmp
Observe: absence of Echo Replies (or absence of ICMP entirely) when default route is missing.

Compare ubuntu_route_before_*, ubuntu_route_after_delete_*, and ubuntu_route_after_restore_* to see the default route removed and restored.
EOF
```

This turns your lab from a personal experiment into a reproducible artifact.

[[#🧭 Table of Contents]]

---

## 13. Reflection prompts — internalizing routing intuition 🤔

Write your reflections in:

`~/day1_lab3/notes/reflections_lab3_$(date -u +"%Y%m%dT%H%M%SZ").md`

Then answer, in full sentences:

What does a host without a default route _believe_ about the world? Describe its perspective as if it were a person trying to navigate with a map that only shows its own neighborhood.

Why does deleting a single line in a routing table break everything above it? Connect this to the idea that every higher‑level protocol assumes that Layer 3 can always find a path.

How does this lab change your understanding of VPNs, pivots, and tunnels? Think about how often those techniques rely on adding or changing routes on a host.

What does “the next hop” mean to you now? Is it just a field in a table, or is it the fundamental decision that makes the rest of the internet possible?

These reflections are how you turn a sequence of commands into durable intuition.

[[#🧭 Table of Contents]]

---

If you’re happy with this version, we can roll straight into **Lab 4** in this exact style.