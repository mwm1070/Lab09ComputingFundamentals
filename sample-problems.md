# Extra Sample Problems (exam-style) — with worked answers

These are *not* from the textbook; they're new questions in the same style, to check that you
can apply the methods to unfamiliar topologies/numbers. Try each before reading the answer.

---

## 1. Forwarding tables on a ring

Switches S1–S2–S3–S4–S1 form a ring. Hosts: A on S1, B on S2, C on S3, D on S4. Give forwarding
tables (neighbour form), using shortest paths and choosing consistently where there's a tie.

<details><summary>Answer</summary>

| Switch | A | B | C | D |
|--------|---|---|---|---|
| S1 | local | S2 | S2 or S4 | S4 |
| S2 | S1 | local | S3 | S1 or S3 |
| S3 | S2 or S4 | S2 | local | S4 |
| S4 | S1 | S1 or S3 | S3 | local |

Opposite-corner destinations (A↔C, B↔D) each have two equal-cost routes.
</details>

---

## 2. Default entry and implied entries

Topology: hosts X,Y,Z on switches S1,S2,S3; S1–S2, S2–S3; **default route toward S3**. Give the
smallest forwarding table for each switch after deleting entries implied by the default.

<details><summary>Answer</summary>

- S1 (default next hop S2, toward S3): ⟨X, local⟩, ⟨default, S2⟩.
- S2: ⟨Y, local⟩, ⟨default, S3⟩ (reaches X via S1 but default is *toward S3*, so keep ⟨X,S1⟩!).  
  **Careful:** the default points toward S3, so it does **not** cover X (which is toward S1). So
  S2 = ⟨X,S1⟩, ⟨Y,local⟩, ⟨default,S3⟩.
- S3: ⟨Z, local⟩ and entries implied? Its default would point "toward the other side" — if the
  default is not defined for S3, list ⟨X,S2⟩, ⟨Y,S2⟩, ⟨Z,local⟩.

Lesson: only delete an entry if the default's **next hop** matches it.
</details>

---

## 3. TTL vs a routing loop

A packet starts with TTL = 8 and enters a persistent routing loop of 4 switches. How many times
does it traverse the loop, and what happens?

<details><summary>Answer</summary>

TTL is decremented once per router and the packet is dropped when TTL = 0. Starting at 8 it
survives 8 forwardings and is dropped on the 8th, i.e. it goes **around the 4-switch loop exactly
twice**. (Ethernet frames have no such limit — that's why loops are forbidden.)
</details>

---

## 4. Switch-learning sequence

Topology:
```
   A──S1──S2──B
       │
       S3──C
```
All tables empty. Transmissions: A→B, B→A, C→A. Give every switch's final table.

<details><summary>Answer</summary>

- A→B: B unknown ⇒ flooded; every switch learns A.
- B→A: S2 knows A ⇒ forward to S1; S1 knows A ⇒ deliver; **S3 never sees it**; S2 and S1 learn B.
- C→A: S3 knows A ⇒ forward to S1; S1 knows A ⇒ deliver; **S2 never sees it**; S3 and S1 learn C.

| Switch | Known |
|--------|-------|
| S1 | A, B, C |
| S2 | A, B |
| S3 | A, C |
</details>

---

## 5. CSMA/CD backoff

Three stations A, B, C all transmit at T=0 and collide (detected at T=1). Their first backoffs
are A: k=0, B: k=1, C: k=0. Continue the simulation until one succeeds.

<details><summary>Answer</summary>

- T=1: A and C transmit (k=0) → collide; detected T=2. n=2, k∈{0..3}. Suppose A picks k=2 (→T=4),
  C picks k=0 (→T=2).
- T=2: B transmits (its earlier k=1 → T=2) and C transmits (k=0) → collide; detected T=3.
- (You can then pick k values for B and C so that at T=3 or later exactly one is ready.)

The point being tested: apply "next attempt = collision-detection time + k slots", keep n per
station, and stop at the first single-station slot. Many valid traces exist.
</details>

---

## 6. Spanning tree pruning

2×3 grid, root = **S2** (smallest ID):
```
S1──S2──S3
│   │   │
S4──S5──S6
```
Edges: S1–S2, S2–S3, S1–S4, S4–S5, S5–S6, S3–S6, S2–S5. Root S2.

<details><summary>Answer</summary>

Cost to root / parent (tie → smaller neighbour ID):

| Switch | cost | parent |
|--------|------|--------|
| S1 | 1 | S2 |
| S3 | 1 | S2 |
| S5 | 1 | S2 |
| S4 | 2 | S1 (1 < 5) |
| S6 | 2 | S3 (3 < 5) |

Enabled: S1–S2, S2–S3, S2–S5, S1–S4, S3–S6.
**Pruned: S4–S5 and S5–S6.**
</details>

---

## 7. OpenFlow two-table learning

A 2-table learning switch uses T0 = destination match, T1 = source match, with defaults
`T0: flood,→T1` and `T1: →controller`. A packet h4→h1 arrives where h4 is unknown and h1 is
known. What happens?

<details><summary>Answer</summary>

- T0: no destination rule for h1? *h1 is known* ⇒ T0 matches, forwards h1 out its port, and sends
  the packet to T1.
- T1: no source rule for h4 ⇒ default fires ⇒ packet is **sent to the controller**, which installs
  a rule for h4 (source, arrival port).
So the packet is forwarded normally **and** the controller learns h4. No flooding.
</details>

---

## 8. Wi-Fi contention cost

Average contention interval = 32 slot times; slot = 20 µs. Packets are 1500 bytes at 54 Mbps.

<details><summary>Answer</summary>

- Contention T = 32 × 20 = **640 µs**.
- Packet transmit = (8 × 1500)/54 ≈ **222 µs**.
- Lost fraction = T/(T+222) = 640/862 ≈ **74%**.
</details>

---

## 9. WiMAX guard intervals

Max speed 30 m/s, ranging updates once every 90 s, 8000 senders/s.

<details><summary>Answer</summary>

- Movement in 90 s = 30 × 90 = 2700 m → delay change = 2700/300 = **9 µs**.
- Guard per sender = 2 × 9 = **18 µs**.
- Per second: 8000 × 18 = 144 000 µs = **0.144 s ⇒ 14.4%**.
</details>

---

## 10. Hidden or exposed?

Classify: (a) X is transmitting to Y; Z can hear X but not Y, and defers although it wanted to send
to a station Z can reach without colliding. (b) X and Y are in range of Z but not of each other, and
both send to Z at once.

<details><summary>Answer</summary>

- (a) **Exposed node** — Z defers unnecessarily.
- (b) **Hidden node** — X and Y can't hear each other, so they collide at Z.
</details>

---

## 11. Two-ray scaling

Under the two-ray model, if the distance doubles, by what factor does received power change?

<details><summary>Answer</summary>

Power ∝ 1/r⁴ ⇒ doubling r multiplies power by 1/2⁴ = **1/16** (a 12 dB loss).
</details>

---

## 12. VPN routing

A host wants *all* its Internet traffic tunnelled to its workplace, but still needs the tunnel to
actually reach the VPN server. What route is required?

<details><summary>Answer</summary>

A **host-specific route** for the VPN server's IP out the *real* interface, with everything else via
the tunnel (default route through the virtual interface). Without the host route, the tunnel's own
packets are fed back into the tunnel (an encapsulation loop).
</details>

---

## 13. Build a VC table

Network: A–S1, B–S1, S1–S2, S2–C, S2–D. Establish a connection A→C with link VCIs 3 (A–S1) and 7
(S1–S2), then 4 (S2–C). Write S1's and S2's table entries.

<details><summary>Answer</summary>

- S1: ⟨VCI_in 3, port A⟩ → ⟨VCI_out 7, port S2⟩.
- S2: ⟨VCI_in 7, port S1⟩ → ⟨VCI_out 4, port C⟩.

And the reverse (if the connection is bidirectional) would carry its own VCIs on the return links.
</details>

---

## 14. Token-ring average wait

10 Mbps ring, 64-byte token, 100 stations, idle network.

<details><summary>Answer</summary>

Token transmit = 512 bits / 10 Mbps = 51.2 µs. Average hops before the would-be sender gets it =
100/2 = 50. Wait ≈ 50 × 51.2 = **2560 µs ≈ 2.56 ms**.
</details>

---

## 15. ATM XOR break-even

N = 10 data cells plus one XOR cell. At what per-cell loss rate p do the two schemes cost the same?

<details><summary>Answer</summary>

Without: 10 + (10p)×10 = 10 + 100p. With: 11. Equal when 100p = 1 ⇒ **p = 0.01 = 1%**.
</details>

---

## 16. Window to fill a link

100 Mbps link, round-trip time 50 ms. How many bits/packets can be in flight?

<details><summary>Answer</summary>

Bandwidth×delay = 100×10⁶ × 0.05 = **5×10⁶ bits = 625 KB**. At 1500 B (12 000 bits) per packet
that's ≈ **417 packets** in flight.
</details>

---

## 17. Trace a packet

Using the Ch. 3 Ex 9.0 tables, a packet is sent from S1 to destination **E**. Where does it go, and
does it ever pass S5 *before* S6?

<details><summary>Answer</summary>

S1→S2 (E,S2) → S3 (E,S3) → S6 (E,S6) → S5 (E,S5) → S4 (E,S4) → S10. So it passes **S6 before S5**,
and it visits four switches on the "right" side before looping back — a deliberately circuitous
route.
</details>

---

## 18. Short answer

**(a)** Why can't a datagram routing loop be a simple linear chain longer than two switches?
**(b)** Give one advantage of virtual circuits over datagrams and one advantage of datagrams.

<details><summary>Answer</summary>

(a) Forwarding is deterministic per destination, so revisiting a switch means the packet repeats the
same sequence — a genuine cycle (or a 2-switch ping-pong). A "linear" loop with distinct switches
would have to terminate, not loop.

(b) VC: QoS guarantees / small headers (good for voice). Datagram: little router state and easy
rerouting after failure. (Either direction accepted.)
</details>