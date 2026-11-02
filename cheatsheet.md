# One-Page Cheat Sheet — Computer Networks, Ch. 1–5 (Dordal, 2nd ed.)

> Print `cheatsheet.html` for the real one-pager. This Markdown is the readable source.

---

## 1 · OVERVIEW

**Layers (bottom→top):** Physical → Link → Network (IP) → Transport (TCP/UDP) → Application.
**Encapsulation:** data → +TCP/UDP hdr = *segment* → +IP hdr = *packet* → +link hdr = *frame*. Header added going down, stripped going up.

**Metrics**
- **Bandwidth / data rate** = bits/sec (capacity). **Throughput** = actual delivered rate.
- **Bandwidth delay** = `packet_size / bandwidth` (time to put packet on wire).
- **Propagation delay** = `distance / c` (c ≈ 300 m/µs = 3×10⁸ m/s).
- **End-to-end delay** = Σ (bandwidth delay + propagation + queueing).
- **Bandwidth×delay product** = max bits "in the pipe" = window needed to keep a link busy.

**Datagram forwarding**
- Forwarding table: `destination → next_hop`; next_hop **must be a direct neighbour**.
- IP uses **longest-prefix match**; a **default** entry (`0.0.0.0/0`) catches the rest.
- Routers decide **per packet, by destination only** (no history) → functional graph.
- **TTL** decremented each hop, dropped at 0 → bounds loops (default 64).
- **Routing loop:** from inconsistent tables. *Linear* loop = two switches ping-pong.
  Ethernet avoids nonlinear loops via spanning tree; switches never forward back out the arrival port.

**Congestion:** arrivals exceed outgoing bandwidth → queues. **Firewall** filters by port/address.
**Utilities:** `ping` (ICMP echo), `traceroute`, `netstat`, `ifconfig`/`ip addr`.

---

## 2 · ETHERNET BASICS

**Classic 10 Mbps Ethernet:** shared medium, **CSMA/CD** (carrier sense, multiple access, collision detect), half-duplex.
- **Min frame = 64 bytes = 512 bits** (so a collision is always detected): 512 bit-times = **slot time = 51.2 µs** at 10 Mbps; must cover **2 × max propagation**.

**Collision handling**
1. Sender keeps listening; on collision, sends **jam**.
2. **Binary exponential backoff:** after the *n*-th collision choose `k` uniformly in `{0 … 2ⁿ−1}`, wait `k` slot times, retry.
3. Give up after **10 collisions**, max **16 attempts**.

**Addressing:** 48-bit MAC; broadcast = `FF:FF:FF:FF:FF:FF`.

**Switch learning (transparent bridging) — the exam algorithm**
- On frame arriving at port *I* with source *S*: record **`⟨S, I⟩`** (where S lives).
- To deliver to dest *D*: if `⟨D, J⟩` exists and **J ≠ I** → forward out J only.
  - if J = I → drop (already on that side).
  - if **unknown D** → **flood** out every port **except** the arrival port.
- Hosts are never asked; switches self-learn. Tables age out (~5 min).
- **MAC flooding attack:** fake sources fill the table → forced flooding → eavesdropping. Defense: **port security**.

---

## 3 · ADVANCED ETHERNET

**Spanning Tree (802.1D, Perlman)** — build a loop-free subset.
1. Elect **root** = switch with smallest ID.
2. Each switch picks min-cost path to root (hops; bandwdth-aware in practice).
3. **Ties:** smaller **next-hop switch ID** wins (then smaller port number).
4. Disable links not chosen. Recompute on failure. **BPDUs** to multicast `01:80:c2:00:00:00`.

**VLAN (802.1Q):** split one physical switch into logical LANs; **12-bit VLAN ID** tag on trunk links; **access port** (one VLAN) vs **trunk port** (many, tagged). Isolates broadcast domains without new hardware.

**TRILL / SPB:** layer-2 routing that uses *all* links (multiple paths), not just a spanning tree; avoids broadcast storms.

**SDN / OpenFlow**
- Controller installs **flow tables** on switches: **match fields** (src/dst MAC, IP, TCP port, ingress) + **actions** (drop, forward port, flood, **send to controller**, modify field, go to next table).
- **Priority** resolves overlaps; **table-miss** (priority 0) usually → controller.
- **Learning switch, one table:** match `⟨dest,src⟩` pairs (scales badly: 10⁴ addrs → 10⁸ entries).
- **Two tables:** `T0` matches *dest* → forward / flood; `T1` matches *src* → do-nothing (known) / send to controller (learn).
- Typical defaults: `T0: no-match → flood, send to T1`; `T1: no-match → send to controller`.
- **Reversed:** `T0` matches *src*, `T1` matches *dest* (same action pattern).

---

## 4 · WIRELESS LANs

**Radio:** free space power ∝ 1/r². **Two-ray ground model** (LOS + ground reflection, 180° phase flip) → amplitude ∝ 1/r², **power ∝ 1/r⁴**.

**Wi-Fi 802.11 = CSMA/CA** (collision *avoidance*). Book constants:
| Param | Value |
|-------|-------|
| Slot time | **20 µs** |
| DIFS (normal IFS) | **50 µs** |
| SIFS (short IFS) | **10 µs** |
| Backoff initial | k < 2⁵ = 32 slots* |
| c (radio) | 300 m/µs (1 slot ≈ 6000 m) |

\* later attempts double: 64, 128, 256, 512, 1024, 1024.

**Access procedure:** wait **DIFS** of idle → pick random backoff in slots → count down (pause if medium busy) → transmit → receiver ACKs after **SIFS** (no carrier sense first).

**Hidden-node problem:** A and B can't hear each other but both reach C → they transmit together, collide at C.
**RTS/CTS:** A sends RTS after DIFS; C replies CTS after SIFS; CTS tells hidden stations to hold off (**NAV**).
**Exposed node:** a station defers even though its transmission would not actually collide.

**Wi-Fi security — 4-way handshake:** AP→ANonce; STA→SNonce+MIC; AP→GTK+install+MIC; STA→ACK. Fresh nonces ⇒ **replays fail at message 2 (MIC check)**.

**WiMAX / LTE:** base station **schedules** slots (per time & frequency); subscriber stations need *not* hear each other. **Ranging** maintains timing; moving stations need **guard intervals**.

---

## 5 · OTHER LANs

**VPN:** a *virtual link* over the Internet. Packets are **encapsulated (tunnelled)** and sent to the VPN server; virtual interface looks locally attached.
- Route private traffic via the tunnel; often make the tunnel the **default route**, but keep a **host-specific route** to the VPN server via the real interface (prevents intruders; causes triangle routing for outsiders).
- **IP-based** (better) vs **TCP-based** tunnels (TCP-in-TCP → timeout problems).

**Token Ring (802.5):** a **token** circulates; a station transmits only while holding it; 4/16 Mbps. **Token Bus** = same on a bus, order fixed by agreement (e.g. by MAC address).
**Carrier Ethernet:** carrier-grade switched Ethernet services.
**Virtual circuits:** connection **established first**; each link gets a **VCI** (locally unique); switch table keyed by **`⟨VCI_in, port_in⟩ → ⟨VCI_out, port_out⟩`**; VCI rewritten each hop.
- Pros: QoS guarantees, small headers (good for voice). Cons: state per connection, poor failure recovery.
**ATM:** 53-byte **cells** (5-byte header + 48 data), AAL-5. Cell-loss recovery: send N cells + 1 XOR cell (SM90).

---

### Constants & quick numbers
`bandwidth×delay` = window to fill a link · 64-byte min Ethernet frame · Wi-Fi 20/50/10 µs (slot/DIFS/SIFS) · c = 300 m/µs · ATM cell = 53 B · MAC = 48-bit · VLAN ID = 12-bit · TTL = 8-bit.