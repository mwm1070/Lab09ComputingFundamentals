# Flashcards — Chapters 1–5

Cover the answer, say it aloud, then check. `flashcards.csv` has the same cards for Anki/Quizlet.

---

## Ch. 1 — Overview

**Q:** Name the five TCP/IP layers from bottom to top.

**A:** Physical · Link · Network (IP) · Transport (TCP/UDP) · Application.

**Q:** What is encapsulation?

**A:** Each layer adds a header going down: data → segment (+TCP/UDP) → packet (+IP) → frame (+link). Stripped on the way up.

**Q:** Bandwidth vs throughput?

**A:** Bandwidth = capacity (bits/s). Throughput = actual delivered rate (≤ bandwidth).

**Q:** What is the "bandwidth delay"?

**A:** Time to put a packet on the wire = packet_size / bandwidth.

**Q:** How is propagation delay computed?

**A:** distance / signal speed (c ≈ 300 m/µs).

**Q:** What does the bandwidth×delay product mean?

**A:** The number of bits that can be "in the pipe" — the window needed to keep the link busy.

**Q:** What must a forwarding table's next_hop be?

**A:** A *directly connected* neighbour.

**Q:** How does IP choose among overlapping routes?

**A:** Longest-prefix match; a default route (0.0.0.0/0) catches everything else.

**Q:** What is TTL and why does it exist?

**A:** An 8-bit field decremented at each router; dropped at 0. It bounds routing loops (default 64).

**Q:** Two kinds of routing loop?

**A:** *Nonlinear* (a cycle in the graph) and *linear* (two switches ping-pong). Ethernet forbids graph loops; TTL/arrival-port rules stop ping-pong.

**Q:** What causes congestion?

**A:** Packets arrive faster than they can be sent out (in-rate > out-rate) → queueing.

**Q:** Name networking utilities and what they do.

**A:** `ping` (ICMP echo, reachability), `traceroute` (path/hops), `netstat` (connections), `ifconfig`/`ip addr` (interfaces).

---

## Ch. 2 — Ethernet Basics

**Q:** What does CSMA/CD stand for?

**A:** Carrier Sense, Multiple Access, Collision Detection.

**Q:** Why is the Ethernet minimum frame 64 bytes?

**A:** So a sender is still transmitting when a collision returns: 512 bits = 51.2 µs slot = 2 × max propagation at 10 Mbps.

**Q:** Describe binary exponential backoff.

**A:** After the *n*-th collision, pick k uniformly in {0 … 2ⁿ−1}, wait k slot times, retry. Give up after 10 collisions / 16 tries.

**Q:** What is the Ethernet broadcast MAC address?

**A:** FF:FF:FF:FF:FF:FF.

**Q:** State the Ethernet switch learning algorithm.

**A:** Record ⟨src, arrival port⟩; known destination → forward out that port only; unknown destination → flood all ports except the arrival port; never forward back out the arrival port.

**Q:** What is "fallback to flooding"?

**A:** Sending an unknown-unicast frame out every port but the one it arrived on.

**Q:** Why is a switched Ethernet loop catastrophic?

**A:** Flooded frames have no TTL → they circulate forever (broadcast storm) and saturate the network.

**Q:** What is a MAC-flooding attack and its defence?

**A:** A host sends frames with many fake source addresses to overflow switch tables → forced flooding → eavesdropping. Defence: port security + secure MACs.

**Q:** Speeds/dates of classic, Fast, Gigabit Ethernet?

**A:** 10 Mbps (shared, half-duplex), 100 Mbps (Fast), 1 Gbps (Gigabit, full-duplex point-to-point).

---

## Ch. 3 — Advanced Ethernet

**Q:** Why does Ethernet need a spanning tree?

**A:** To keep redundancy but remove loops, so flooding/broadcast can't storm.

**Q:** How is the spanning-tree root chosen?

**A:** The switch with the smallest ID.

**Q:** Spanning-tree tie-break rule?

**A:** Pick the minimum-cost path to the root; on ties choose the neighbour (next switch) with the smaller ID, then the smaller port number.

**Q:** What messages does spanning tree use?

**A:** BPDUs ("Hello"), sent to multicast 01:80:c2:00:00:00.

**Q:** What is a VLAN? How is it tagged?

**A:** A logical LAN within physical switches; 802.1Q adds a 12-bit VLAN ID. Access port = one VLAN; trunk port = many, tagged.

**Q:** What problem does TRILL/SPB solve?

**A:** Layer-2 forwarding that uses all links (multiple paths) instead of a single spanning tree, without broadcast storms.

**Q:** In OpenFlow, what is a flow-table entry?

**A:** Match fields (src/dst MAC, IP, TCP port, ingress …) + actions (drop, forward port, flood, send to controller, modify field, go to next table), plus a priority.

**Q:** OpenFlow table-miss behaviour?

**A:** The priority-0 "match everything" entry — typically send the packet to the controller.

**Q:** Why is the one-table ⟨dest,src⟩ learning scheme impractical?

**A:** It needs ~N² entries (10⁴ host pairs → 10⁸), so it doesn't scale.

**Q:** How do the two OpenFlow tables split the work?

**A:** T0 matches destination (forward/flood), T1 matches source (do nothing if known, else send to controller).

---

## Ch. 4 — Wireless LANs

**Q:** Why does Wi-Fi use CSMA/CA instead of CSMA/CD?

**A:** A station can't hear collisions while transmitting, so it *avoids* them (backoff + ACKs) rather than detecting them.

**Q:** Book values for Wi-Fi slot / DIFS / SIFS?

**A:** 20 µs / 50 µs / 10 µs.

**Q:** Walk through the Wi-Fi transmit procedure.

**A:** Sense idle for DIFS → random backoff slots → (pause if busy) → transmit → receiver ACKs after SIFS.

**Q:** Does the ACK sender listen first?

**A:** No — ACK is sent after SIFS regardless of medium state.

**Q:** What is the hidden-node problem?

**A:** Two stations can't hear each other but both reach a common receiver → they transmit simultaneously and collide there.

**Q:** How does RTS/CTS fix hidden nodes?

**A:** Sender sends RTS (after DIFS); receiver answers CTS (after SIFS); CTS is heard by hidden stations, which set their NAV and defer.

**Q:** What is the exposed-node problem?

**A:** A station defers even though transmitting would not actually collide (needless loss of throughput).

**Q:** Two-ray ground model: how does power scale with distance?

**A:** LOS + ground reflection (180° flip) → amplitude ∝ 1/r² → power ∝ 1/r⁴.

**Q:** Wi-Fi (WPA2) 4-way handshake messages?

**A:** AP→ANonce; STA→SNonce+MIC; AP→GTK+install+MIC; STA→ACK.

**Q:** Why does a replay of the handshake fail?

**A:** Each handshake uses fresh nonces; the replayed MIC was computed over the old ANonce, so it fails at message 2.

**Q:** How is WiMAX/LTE access different from Wi-Fi?

**A:** The base station *schedules* each station's slots (time/frequency); subscribers need not hear each other.

**Q:** Why do mobile WiMAX stations need guard intervals?

**A:** Motion changes propagation delay; ranging keeps it updated, and slack is left for early/late arrivals.

---

## Ch. 5 — Other LANs

**Q:** What is a VPN, in one sentence?

**A:** A virtual link across the Internet: packets are encapsulated (tunnelled) to a VPN server, making a remote host look locally attached.

**Q:** Why send *all* traffic (not just private traffic) through the tunnel?

**A:** So outsiders can't reach the host via its ordinary Internet address (security). It needs a host-specific route to the VPN server, and causes triangle routing for non-work traffic.

**Q:** IP-based vs TCP-based tunnel?

**A:** IP-based is preferred; a TCP-based tunnel suffers TCP-in-TCP timeout problems.

**Q:** How does Token Ring control access?

**A:** A token circulates; a station may transmit only while holding the token (4/16 Mbps).

**Q:** How can Token Ring stations order themselves?

**A:** By MAC-address order (smallest = first), or via a bidding/claim process.

**Q:** Datagram switching vs virtual-circuit switching?

**A:** Datagram: address by destination, per-packet routing, no setup. VC: establish a connection first, address by connection ID (VCI), switch keeps per-connection state.

**Q:** What is a VCI and how unique must it be?

**A:** Virtual Circuit Identifier; only needs to be unique per link, not globally.

**Q:** What is the virtual-circuit switch table key?

**A:** ⟨VCI_in, port_in⟩ → ⟨VCI_out, port_out⟩; the packet's VCI is rewritten each hop.

**Q:** Two advantages of virtual circuits?

**A:** QoS guarantees (capacity reserved) and smaller headers (good for voice). *(Disadvantage: per-connection state, poor failure recovery.)*

**Q:** ATM cell size and makeup?

**A:** 53 bytes = 5-byte header + 48-byte payload; AAL-5 adaptation.

**Q:** How does the SM90 XOR cell help?
**A:** Send N data cells plus one XOR cell; any one lost cell can be reconstructed, avoiding retransmitting the whole packet.
