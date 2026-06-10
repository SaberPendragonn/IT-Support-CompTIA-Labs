# 📡 CCTV Stream Multicast Routing: IGMP Snooping + PIM-SM
### *My Seventh Project as a Network Engineer*

> One stream from the camera. Thousands of viewers. Zero duplicates.

---

## 🗺️ The Topology

![Network Topology](https://YOUR-IMAGE-HOST.com/topology.png)

**Backbone Architecture:** Dual upstream Layer 3 core routers (Core-A/Left and Core-B/Right) cross-connected via a dedicated, private Point-to-Point L3 network link.

**Distribution & Access Layer:** Triple Layer 2 access switches multi-homed via redundant uplinks to both core nodes, forming an active-active physical topology partitioned by Multiple Spanning Tree Protocol (MSTP).

**Multicast Routing & Control Stack:** Protocol Independent Multicast - Sparse Mode (PIM-SM) with split Rendezvous Points (RP), independent Bootstrap Router (BSR) domains, and IGMP Snooping.

---

## 🎭 The Lore

Okay so here's the scenario:

I've got a 256kbps CCTV feed on VLAN 10 (239.1.1.1). Corporate compliance says users in any segment—VLANs 10 through 70—must be able to pull either stream.

Standard unicast routing? Each streaming server generates a separate stream for every single client. Server NICs choke. Backplane saturates. It's a disaster.

Multicast fixes that. One stream from the server. The network replicates it only where needed.

But here's the catch:

My access switches are cross-connected to both cores. MSTP partitions VLANs to prevent loops:

- **MSTP Instance 1 (Core-A Root):** VLANs 10, 30, 50, 70 → Left links active and forwarding
- **MSTP Instance 2 (Core-B Root):** VLANs 20, 40, 60 → Right links active and forwarding

The danger? A client on VLAN 30 requests the VLAN 10 stream. Their traffic travels up the Left Link (MSTP says so). If my IGMP Querier or PIM Rendezvous Point is blindly configured on Core-B, control packets fight the Spanning Tree blocks.

I needed every layer to agree on the same path.

---

## 🛠️ Performance Highlights

**Distributed Core Load Balancing**  
Configured split BSR and static RP domain partitions across the dual core backbone. Segmented group pools (239.1.0.0/16 and 239.2.0.0/16) to distribute processing overhead symmetrically. For this demonstration we only use 239.1.1.1/32

**Targeted Switch IGMP Snooping**  
Enforced hardware-level Layer 2 IGMP snooping verification across separate physical access units. Validated precise, non-flooded port distribution of 10 Mbps stream payloads to designated active subscribers.

**Resilient Infrastructure Topology Failover**  
Proved automated multicast stream recovery across redundant switch paths within a strict sub-15-second convergence window during catastrophic primary physical link failure.

![Multicast Architecture Demo](https://github.com/user-attachments/assets/0a27f1f6-234f-4fe4-9e7e-617283c09e23)

---

## 🧪 The Proof: Validation Tests

### Test 1: Switch Port Traffic Capture and Selective Hardware Delivery

**What I did:** Initiated a single 256kbps CCTV multicast feed (239.1.1.1) inside VLAN 10. Ran active MikroTik Torch packet captures on explicit physical interfaces across Access-Switch-1, Access-Switch-2, and Access-Switch-3 to track ingress and egress stream behavior.

**What happened:** Torch metrics confirmed highly isolated hardware delivery. No broadcast leakage.

- **SW1 (ether1):** Registered clean ~256kbps ingress stream directly from source streaming server inside VLAN 10
- **SW2 (ether1):** Captured ~256kbps egress throughput routing out to active streaming client terminal
- **SW3 (ether1):** Captured ~256kbps egress throughput pushing exclusively to second designated streaming client endpoint

**The win:** All unjoined ports across the triple switch fabric remained at 0 bps. IGMP Snooping successfully contained high-bandwidth media traffic to intended endpoints.

![Switch Port Capture](https://YOUR-IMAGE-HOST.com/switch-torch.gif)

---

### Test 2: Redundant Link Failure Recovery and Convergence Delay

**What I did:** Severed the active physical Left Link on the access layer switch while clients were streaming the 10 Mbps IPTV feed. Measured total system recovery time before video stream recovered on destination monitors.

**What happened:**

| Run | Failover Time |
|-----|---------------|
| Run 1 | 4.0 seconds |
| Run 2 | 12.0 seconds |
| Run 3 | 13.0 seconds |

**Architectural Root Cause:** Failover successfully stayed under 15 seconds. But convergence is bottlenecked by native state timeout mechanisms of MSTP and VRRP. The system relies on standard dead timers and background advertisements to detect dead peers.

**Recommendation:** Convergence times can be reduced to a lot by shifting Layer 3 core out of static VRRP tracking and deploying dynamic routing protocols (OSPF or BGP) coupled with BFD to instantly tear down and rebuild path topology upon link loss.

![Link Failover](https://YOUR-IMAGE-HOST.com/failover-test.gif)

---

### Test 3: Cross-VLAN Multicast Inter-Subnet Routing

**What I did:** Attempted to configure multi-subnet multicast boundaries across the infrastructure. Goal was to allow a media host on VLAN 30 to ingest raw media streams broadcasting from server pool on VLAN 10.

**What happened:** [PENDING / KNOWN CONFIGURATION CONSTRAINT]

Cross-subnet multicast routing remains pending due to a hardware-offloading limitation in the MikroTik CRS3xx/CRS5xx switch-chip architecture.

**The technical limitation:** The underlying hardware ASIC bridge matrix does not natively support hardware-accelerated IGMP snooping translation across differing VLAN IDs without forcing traffic up into the primary system CPU. Forcing heavy 10 Mbps streams into software processing risks severe CPU exhaustion and packet breakdown.

**Status:** This test remains paused until MikroTik releases an updated RouterOS v7 firmware branch that addresses cross-VLAN hardware offloading for this switch-chip tier.

 https://forum.mikrotik.com/t/multicast-across-switches-does-not-work-crs3xx/173251/16 


---

## 🚧 Engineering Challenges

**The Asymmetric Reverse Path Forwarding (RPF) Block**

Initial cross-VLAN streaming failed completely. Clients sent Join requests. No video arrived. Logs showed hundreds of "PIM RPF lookup failed" errors.

**Why it happened:** Multicast routers perform an RPF sanity check. When a packet arrives, the router checks if that interface is the fastest way back to the source. With active-active paths, packets took one uplink but the router expected them on the other. Security drop.

**How I solved it:** Synchronized unicast routing metrics. Tuned PIM interface costs across both core routers. Forced Layer 3 return paths to perfectly mirror the physical infrastructure layout.

**The Multi-Switch Snooping Breakdown**

Clients on Access-Switch-1 worked perfectly. Clients on Access-Switch-3? Random drops or full mesh flooding.

**Why it happened:** In a multi-switch environment, IGMP Snooping needs a single clear destination pointer. Switches didn't know where the multicast router lived.

**How I solved it:** Manually defined Multicast Router Ports (Mrouter) on all three switch bridges. Pinned them directly to upstream trunk interfaces leading to the Core Routers. Unified snooping databases across the entire switch mesh. Rock-solid port pruning.

---

## 💡 Final Thoughts

Multicast in a redundant, multi-VLAN topology is the ultimate test of an engineer's ability to coordinate Layer 2 switching maps with Layer 3 routing states.

Allowing high-bandwidth video streams to blindly flood an enterprise network like broadcast noise? That's a junior oversight. It kills wireless arrays. Wastes backplane resources.

Architecting PIM-SM across dual upstream cores with hardware-offloaded IGMP Snooping across a triple switch mesh? That's **true infrastructure engineering**.

You have to:
- Map packet lifecycle across routing tables
- Manage dynamic interface registration
- Build a self-pruning distribution pipeline that handles massive media transits efficiently

**The Metric That Matters: Backplane Cleanliness**

I observed bridge monitor tables across the switch stack during peak streaming hours. Heavy video streams flowed with laser precision only to clients who asked for them. The rest of the enterprise infrastructure stayed completely quiet.


---

*Seventh project down. More to come.* 🔥
