# VPN Performance Benchmark: WireGuard vs L2TP/IPsec vs OpenVPN
### *My Second Project as a Network Engineer*

---

## Topology

**Client PC** → **hAP Lite (VPN Client)** → **RB951/2 (VPN Server)** → **Server PC**

**IP Scheme:**
- Client PC: 192.168.1.10
- hAP Lite: 192.168.1.1 (Client) / 10.0.0.2 (VPN)
- RB951/2: 10.0.0.1 (VPN) / 192.168.2.1 (Server)
- Server PC: 192.168.2.10

> I ran all three VPN protocols through the exact same path so the comparison would be fair.

---

## The Lore

Okay so here's the scenario:

I've got these small MikroTik routers—hAP Lite and RB951/2. Low-power hardware. The kind you'd throw in a small office or home lab.

I needed to set up a VPN. But which protocol?

WireGuard? L2TP/IPsec? OpenVPN?

Everyone talks about security, but on low-end routers, **performance is the real question**. Will the CPU handle it? Will speeds drop? Will it be stable?

I didn't want to guess. So I set up a lab and benchmarked all three.

Same hardware. Same traffic. Same test.

Here's what I found.

---

## The Baseline: No VPN

First, I needed to know what these routers could do without any VPN overhead.

**What I did:**
**Baseline Results:**
- Throughput: 10 Mbps
- Latency: 3 ms
- Packet Loss: 0%
- CPU Usage: 10%

This was my reference point. Anything above this? Pure hardware limit. Anything below? VPN overhead.

---

## The Tests: Three VPN Protocols

I configured each VPN on the routers, then ran the same iperf3 and ping tests. Same parameters. Same duration. Just swapped the protocol.

**Why 10 Mbps?** Safe load for these little routers. Enough to stress the CPU without crashing it.
**Why 60 seconds?** Long enough to catch any instability.
**Why ping -c 50?** Shows latency patterns, not just one lucky packet.

---

## Protocol 1: L2TP/IPsec

First up, L2TP with IPsec. The "traditional" secure VPN.

**What I saw:**
- Throughput: 6.8 Mbps
- Latency: 15 ms
- Packet Loss: 1.2%
- CPU Usage: 60%

It worked. Secure. Stable-ish. But the CPU was sweating.

![L2TP CPU Monitor](https://YOUR-IMAGE-HOST.com/cpu_l2tp.png)

---

## Protocol 2: OpenVPN

Next, OpenVPN. Everyone uses it. Flexible. Lots of options.

**What I saw:**
- Throughput: 4.2 Mbps
- Latency: 30 ms
- Packet Loss: 3.5%
- CPU Usage: 85%

Ouch. The CPU was pinned. Speeds dropped hard. Packet loss creeping up.

![OpenVPN Ping Test](https://YOUR-IMAGE-HOST.com/ping_openvpn.png)

This thing was struggling.

---

## Protocol 3: WireGuard

Finally, WireGuard. The new kid everyone's talking about.

**What I saw:**
- Throughput: 9.5 Mbps
- Latency: 8 ms
- Packet Loss: 0.5%
- CPU Usage: 25%

Wait, what? Almost baseline speeds. Latency barely increased. CPU barely moved.

![WireGuard iperf3](https://YOUR-IMAGE-HOST.com/iperf_wireguard.png)

---

## So What's the Difference?

**WireGuard:**
- 9.5 Mbps throughput
- 8 ms latency
- 25% CPU
- Efficient. Lightweight. Just works.

**L2TP/IPsec:**
- 6.8 Mbps throughput
- 15 ms latency
- 60% CPU
- Secure, but heavy.

**OpenVPN:**
- 4.2 Mbps throughput
- 30 ms latency
- 85% CPU
- Too heavy for low-power hardware.

![Throughput Comparison](https://YOUR-IMAGE-HOST.com/throughput-comparison.gif)

---

## Hiccups I Ran Into

L2TP/IPsec took the longest to configure. IPsec policies, proposals, secrets—lots of moving parts.

OpenVPN was easier to set up but hit the CPU hard during testing. Actually saw packet loss above 5% during the first run. Had to re-test to confirm.

WireGuard? Three lines of config on each router. That was it.

---

## Final Thoughts

These little routers are CPU-bound. Encryption isn't free. The protocol choice decides whether your VPN is usable or painful.

WireGuard delivered almost baseline speeds with 25% CPU. That's the winner for low-power hardware.

L2TP/IPsec works if you need IPsec compatibility. But you're sacrificing throughput.

OpenVPN? On these routers? No. The CPU can't keep up.

**The Metric That Matters: CPU Efficiency**

I measured CPU usage for each protocol under identical load. WireGuard used 25%. OpenVPN used 85%.

**Why This Matters to an Employer:**

If I'm deploying VPNs on low-power hardware, I'm not guessing. I benchmark. I measure throughput, latency, and CPU impact. I pick the protocol that actually works—not just the one everyone talks about.

A 25% CPU VPN that delivers 9.5 Mbps vs an 85% CPU VPN that delivers 4.2 Mbps? That's the difference between a network that works and one that frustrates everyone.

WireGuard wins. By a lot.

---

*Second project down. More to come.* 🔥
