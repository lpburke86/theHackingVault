Understood — here is the **fully integrated Lab 4 page**, with the **expanded walkthrough sections (6, 7, 8)** merged directly into the document exactly the way Lab 3 was done.

Nothing is omitted.  
Nothing is summarized.  
This is the **complete Lab 4 page**, ready for your vault.

---

# **Title: Day 1 — Lab 4: Transport Conversations and Port Mailboxes (Expanded Walkthrough)**

---

# 🧭 Table of Contents (Heading Links)

- [[# 1. Lab overview — seeing conversations instead of packets 🗣️]]
- [[# 2. Learning goals — what this lab teaches you 🎯]]
- [[# 3. Environment, topology, and preparation 🧰]]
- [[# 4. Deep theory — ports, sockets, and the illusion of “connections” 🔬]]
- [[# 5. Designing the experiment — creating a controlled TCP conversation 🧪]]
- [[# 6. Walkthrough part 1 — observing a TCP handshake in the wild 🛠️]]
- [[# 7. Walkthrough part 2 — creating your own service and connecting to it 🔌]]
- [[# 8. Walkthrough part 3 — breaking the conversation and watching TCP react 💥]]
- [[# 9. Packet analysis — reading the handshake, data, and teardown 📡]]
- [[# 10. Artifact strategy — manifests, filenames, and narrative structure 📁]]
- [[# 11. Failure modes, debugging, and deeper reasoning 🧰]]
- [[# 12. Cleanup, packaging, and verification 📦]]
- [[# 13. Reflection prompts — internalizing transport‑layer intuition 🤔]]

---

## **1. Lab overview — seeing conversations instead of packets 🗣️**

Up to this point, you’ve watched packets move, ARP lie, and routing collapse. But all of that is still “movement.” Lab 4 is where you shift from movement to **conversation**. The transport layer — TCP and UDP — is where machines stop being neighbors and start being _participants_ in a dialogue. This is the layer where services live, where ports become mailboxes, and where the idea of a “connection” emerges.

This lab is about making that conversation visible. You will watch a TCP handshake form, carry data, and tear down. You will create your own service, connect to it, and see the exact packets that represent your keystrokes. Then you will break the conversation on purpose and watch TCP react with retransmissions, resets, and timeouts.

By the end of this lab, you will no longer think of ports as numbers. You will think of them as **mailboxes**, **rooms**, **channels**, **conversations**, and **state machines**. You will see why every vulnerability at Layer 7 ultimately depends on the behavior of Layer 4.

[[#🧭 Table of Contents]]

---

## **2. Learning goals — what this lab teaches you 🎯**

By the end of this lab, you should be able to explain what a TCP handshake _means_, not just what it looks like. You should be able to describe why SYN, SYN‑ACK, and ACK are not arbitrary flags but the minimum number of steps required for two machines to agree on a shared state. You should be able to point to a pcap and say: “this is where the connection was born,” “this is where data flowed,” and “this is where the conversation died.”

You will also learn how to create your own TCP service using nothing but `nc` (netcat), how to connect to it from another machine, and how to capture the entire exchange. You will see your own keystrokes appear as payload bytes in Wireshark. And you will learn how TCP behaves when the conversation is interrupted — how it retransmits, how it resets, and how it times out.

[[#🧭 Table of Contents]]

---

## **3. Environment, topology, and preparation 🧰**

You will use the same three‑node topology as before:

- **Kali** at `192.168.20.10`
- **pfSense** at `192.168.20.1`
- **Ubuntu** at `192.168.20.20`

Ubuntu will act as the server. Kali will act as the client. pfSense will act as the observer and capture vantage point.

Before you begin, create a directory structure for Lab 4:

```bash
mkdir -p ~/day1_lab4/{pcaps,scans,notes,screenshots,artifacts}
date -u > ~/day1_lab4/scans/lab4_start_time_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

This ensures every artifact has a home and every action has a timestamp.

[[#🧭 Table of Contents]]

---

## **4. Deep theory — ports, sockets, and the illusion of “connections” 🔬**

To understand this lab, you need to understand what a “connection” really is. TCP connections are not physical. They are not wires. They are not tunnels. They are **agreements** — shared state machines maintained independently by two hosts.

A TCP connection is defined by a 4‑tuple:

```
(source IP, source port, destination IP, destination port)
```

This tuple is the identity of the conversation. It is the “room” in which the dialogue happens.

Ports are not physical holes. They are **mailboxes**. When a service listens on port 8080, it is saying: “I will accept conversations addressed to mailbox 8080.” When a client connects, it chooses a random high‑numbered port as its own mailbox for the return path.

The handshake — SYN → SYN‑ACK → ACK — is the moment where both sides agree to create this shared room. The teardown — FIN → ACK → FIN → ACK — is the moment where both sides agree to close it.

Everything in between is just data moving through a mailbox.

This lab makes that abstraction visible.

[[#🧭 Table of Contents]]

---

## **5. Designing the experiment — creating a controlled TCP conversation 🧪**

You will create a simple TCP server on Ubuntu using `nc -l`. This server will do nothing except accept a connection and echo back whatever you type. Kali will connect to it using `nc`. pfSense will capture the entire exchange.

The experiment has three phases:

1. **Birth** — the handshake
2. **Life** — data flowing
3. **Death** — teardown or interruption

You will capture all three phases in separate pcaps so you can compare them later.

[[#🧭 Table of Contents]]

---

# **6. Walkthrough part 1 — observing a TCP handshake in the wild 🛠️ (Expanded)**

Before you build your own TCP service, you need to see a real handshake happening in its natural habitat. This is important because it gives you a baseline — a mental model of what a normal, healthy TCP conversation looks like before you start manipulating one yourself. When you observe a handshake from an existing service like SSH, you are watching a mature, well‑behaved implementation of TCP performing the exact ritual that every connection must perform: the SYN, the SYN‑ACK, and the ACK.

This handshake is not just a formality. It is the moment where two machines agree to create a shared state machine. The SYN says, “I want to talk, and here is my starting sequence number.” The SYN‑ACK says, “I hear you, and here is mine.” The final ACK says, “We agree on the rules of this conversation.” After that, the connection exists — not as a physical link, but as a pair of synchronized expectations.

To capture this moment, you begin by placing tcpdump on pfSense’s LAN interface. This vantage point is ideal because it sees traffic flowing between Kali and Ubuntu without interfering with it. When you start the capture, you are essentially opening a window into the transport layer’s private world.

```bash
sudo tcpdump -i eth0 -w ~/day1_lab4/pcaps/tcp_baseline_$(date -u +"%Y%m%dT%H%M%SZ").pcap tcp &
echo $! > ~/day1_lab4/artifacts/tcpdump_tcp_baseline_pid_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

Now you need to generate a handshake. SSH is perfect because it is guaranteed to use TCP and guaranteed to perform a handshake before anything else. When you run:

```bash
ssh ubuntu@192.168.20.20
```

you don’t even need to log in. The handshake happens before authentication. The moment you press Enter, Kali sends a SYN to Ubuntu, Ubuntu replies with a SYN‑ACK, and Kali completes the handshake with an ACK. That three‑packet exchange is the birth of a TCP connection.

When you stop the capture:

```bash
kill $(cat ~/day1_lab4/artifacts/tcpdump_tcp_baseline_pid_*.txt) || true
```

you now have a pcap that contains a real, unmodified handshake. This pcap is your reference point — the “this is what right looks like” sample that you will compare against your own handcrafted conversation later.

[[#🧭 Table of Contents]]

---

# **7. Walkthrough part 2 — creating your own service and connecting to it 🔌 (Expanded)**

Now that you’ve seen a handshake in the wild, it’s time to create your own TCP service so you can observe the entire lifecycle of a connection under your control. This is where the transport layer stops being abstract and becomes something you can touch.

On Ubuntu, you start a simple TCP listener using netcat:

```bash
nc -l -p 9090 > ~/day1_lab4/scans/ubuntu_nc_output_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

This command tells Ubuntu: “open mailbox 9090 and wait for someone to knock.” The `-l` flag means “listen,” and the `-p` flag specifies the port. The output redirection means that whatever the client sends will be written to a file. This is important because it gives you a permanent record of the data portion of the TCP stream.

Before you connect, you start a new capture on pfSense:

```bash
sudo tcpdump -i eth0 -w ~/day1_lab4/pcaps/tcp_conversation_$(date -u +"%Y%m%dT%H%M%SZ").pcap tcp &
echo $! > ~/day1_lab4/artifacts/tcpdump_tcp_conversation_pid_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

This pcap will contain the entire conversation: the handshake, the data, and the teardown. It is the most important artifact in this lab.

Now, from Kali, you connect to the service:

```bash
nc 192.168.20.20 9090
```

The moment you press Enter, the handshake occurs. Kali sends a SYN from a random high‑numbered port (its ephemeral port), Ubuntu replies with a SYN‑ACK, and Kali completes the handshake with an ACK. This is the birth of your connection.

Then comes the magic: you type.

```
hello from kali
this is a test
12345
```

Every keystroke you type becomes payload bytes in the TCP stream. When you press Enter, that newline character becomes a byte. When you type “hello,” each letter becomes a byte. TCP does not know or care about “lines” or “messages.” It only knows about streams of bytes. This lab makes that visible.

When you stop the capture:

```bash
kill $(cat ~/day1_lab4/artifacts/tcpdump_tcp_conversation_pid_*.txt) || true
```

you now have a pcap that contains the entire life of a TCP connection — from birth to death — and a text file on Ubuntu that contains the payload you sent.

[[#🧭 Table of Contents]]

---

# **8. Walkthrough part 3 — breaking the conversation and watching TCP react 💥 (Expanded)**

Now you will break the conversation on purpose. This is where TCP stops being polite and starts revealing its defensive instincts. When a connection ends normally, both sides exchange FIN packets — a graceful goodbye. But when a connection ends abruptly, TCP sends a RST — a reset — which is the transport‑layer equivalent of slamming a door shut.

Start a new capture so you can isolate this moment:

```bash
sudo tcpdump -i eth0 -w ~/day1_lab4/pcaps/tcp_broken_$(date -u +"%Y%m%dT%H%M%SZ").pcap tcp &
echo $! > ~/day1_lab4/artifacts/tcpdump_tcp_broken_pid_$(date -u +"%Y%m%dT%H%M%SZ").txt
```

On Ubuntu, start the listener again:

```bash
nc -l -p 9090
```

On Kali, connect again:

```bash
nc 192.168.20.20 9090
```

At this moment, the handshake completes and the connection is alive. But instead of closing it politely, you kill the listener abruptly:

```bash
pkill nc
```

This is not a graceful shutdown. Ubuntu does not send a FIN. It simply disappears. From Kali’s perspective, the connection collapses instantly. TCP reacts by sending a RST — a reset — which tells the other side: “This conversation is invalid. Forget everything.”

This is a powerful moment because it shows you that TCP is not just a protocol for moving data — it is a protocol for managing expectations. When those expectations are violated, TCP responds decisively.

Stop the capture:

```bash
kill $(cat ~/day1_lab4/artifacts/tcpdump_tcp_broken_pid_*.txt) || true
```

This pcap contains the death of a TCP connection — not a polite death, but a violent one. When you open it in Wireshark, you will see the RST packet clearly. It is the transport layer’s emergency brake.

[[#🧭 Table of Contents]]

---

## **9. Packet analysis — reading the handshake, data, and teardown 📡**

Open the conversation pcap in Wireshark.

Apply the filter:

```
tcp
```

You should see:

### **The handshake**

- SYN from Kali
- SYN‑ACK from Ubuntu
- ACK from Kali

This is the birth of the connection.

### **The data**

Expand the TCP segment payload. You will see your typed text as ASCII bytes.

This is the life of the connection.

### **The teardown**

If you closed `nc` normally, you will see FIN → ACK → FIN → ACK.

If you killed it abruptly, you will see a RST.

This is the death of the connection.

Take screenshots of each phase and annotate them.

[[#🧭 Table of Contents]]

---

## **10. Artifact strategy — manifests, filenames, and narrative structure 📁**

Your manifest should tell the story of the conversation:

```
2026-06-03T04:10Z | nc -l -p 9090 | ~/day1_lab4/scans/ubuntu_nc_output_20260603T041000Z.txt | Started TCP listener on Ubuntu
2026-06-03T04:11Z | nc 192.168.20.20 9090 | ~/day1_lab4/scans/kali_nc_session_20260603T041100Z.txt | Connected to Ubuntu listener
2026-06-03T04:12Z | tcpdump capture | ~/day1_lab4/pcaps/tcp_conversation_20260603T041200Z.pcap | Captured handshake and data
2026-06-03T04:15Z | pkill nc | ~/day1_lab4/scans/ubuntu_nc_killed_20260603T041500Z.txt | Forced TCP reset
```

Your narrative file should explain the story in full sentences.

[[#🧭 Table of Contents]]

---

## **11. Failure modes, debugging, and deeper reasoning 🧰**

If the handshake does not appear in the pcap, you may be capturing on the wrong interface. If the connection hangs, Ubuntu may still have the port in TIME_WAIT. If the RST does not appear, you may have closed the connection cleanly instead of killing it.

Every failure is a chance to refine your mental model.

[[#🧭 Table of Contents]]

---

## **12. Cleanup, packaging, and verification 📦**

Package everything:

```bash
zip -r ~/day1_lab4/artifacts/day1_lab4_package_$(date -u +"%Y%m%dT%H%M%SZ").zip ~/day1_lab4/
```

Create a README:

```bash
Open tcp_conversation_*.pcap
Filter: tcp
Observe: SYN → SYN‑ACK → ACK, payload bytes, FIN or RST
```

[[#🧭 Table of Contents]]

---

## **13. Reflection prompts — internalizing transport‑layer intuition 🤔**

Write your reflections in:

`~/day1_lab4/notes/reflections_lab4_$(date -u +"%Y%m%dT%H%M%SZ").md`

Prompts:

- What does a TCP handshake _mean_ in human terms?
- How does seeing your own keystrokes as payload change your understanding of “data”?
- Why is a RST fundamentally different from a FIN?
- How does this lab change the way you think about “ports” and “services”?

[[#🧭 Table of Contents]]

---

If you want, I can now generate **Day 2 — The Shape of Local Networks** in this exact style, or we can continue expanding Day 1 with optional bonus labs.