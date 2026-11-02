# Textbook Exercises, Ch. 1–5 — Fully Worked

Source: *An Introduction to Computer Networks* (Dordal, 2nd ed.), exercise sections 1.18, 2.6,
3.6, 4.6, 5.7. Exercises marked **♢** are the ones the book itself solves (chapter 34); those
answers are cross-checked here.

> **How to study with this file:** cover the *Solution* with a sheet of paper, work the
> exercise yourself, then uncover and compare. The exam is drawn from these.

Notation: `⟨dest, next_hop⟩`; "local/direct" = the destination is on a directly attached link.

---

# Chapter 1 — An Overview of Networks

### Ex 1.0 — Forwarding tables, S1–S4

Topology:

```
A         B         C
│         │         │
S1────────S2────────S3
          │
          │
          S4───D
```

Links: A–S1, B–S2, C–S3, S1–S2, S2–S3, **S2–S4**, S4–D. (Note S4 hangs off **S2**.)

| Switch | A | B | C | D |
|--------|---|---|---|---|
| **S1** | local | S2 | S2 | S2 |
| **S2** | S1 | local | S3 | S4 |
| **S3** | S2 | S2 | local | S2 |
| **S4** | S2 | S2 | S2 | local |

Reasoning: S4 reaches A, B, C only through S2; S1 reaches C and D through S2.

---

### Ex 2.0 — Forwarding tables with a shortest-path choice

```
A───S1──────S2───B
    │       │
    │       │
D───S4──────S3───C
```

Links: A–S1, S1–S2, S2–B, D–S4, S4–S3, S3–C, S1–S4, S2–S3.

| Switch | A | B | C | D |
|--------|---|---|---|---|
| **S1** | local | S2 | **S2 or S4** | S4 |
| **S2** | S1 | local | S3 | **S1 or S3** |
| **S3** | **S2 or S4** | S2 | local | S4 |
| **S4** | S1 | **S1 or S3** | S3 | local |

**New decision:** for a *diagonally opposite* destination there are **two equally short
routes**, so the table must pick one (or load-balance). This could not arise in Ex 1.0.

---

### Ex 3.0 — No loops are possible

Rule: an immediate neighbour is reached directly; a diagonal destination via one of the two
neighbours. Define the *distance* `d` of a switch to the destination's switch. Because every
next hop is a neighbour that is strictly closer to the destination, forwarding reduces `d` by
exactly 1 each hop (2 → 1 → 0). A strictly decreasing distance cannot return to a switch it
has already left, so no loop can form, regardless of which diagonal next hop is chosen.

---

### Ex 4.0 ♢ — Forwarding tables for hosts A–E

Topology (edges: A–B, A–C, A–D, B–C, C–E, D–E):

```
        B
      ╱   ╲
     A─────C
     │     │
     D─────E
```

| Switch | Neighbours (→ direct) | Remaining destinations |
|--------|----------------------|------------------------|
| **A** | B, C, D | E → **D or C** |
| **B** | A, C | D → A; E → C |
| **C** | A, B, E | D → **E or A** |
| **D** | A, E | B → A; C → **E or A** |
| **E** | C, D | A → **C or D**; B → C |

Multiple choices are shown in bold. (Matches the book's solution.)

---

### Ex 5.0 — Forwarding tables with a default entry

Topology: A–S1, S1–S3, C–S3, S3–S2, B–S2, S3–S4, S4–S5, D–S4, E–S4.

```
A───S1
    │              D
    │              │
C───S3────────S4─────────S5──── ... other destinations
    │              │
B───S2            E
```

Default points toward S5. After **removing entries implied by the default**:

| Switch | Entries (next-hop in neighbour form) |
|--------|--------------------------------------|
| **S1** | ⟨A, local⟩, ⟨default, S3⟩ |
| **S2** | ⟨B, local⟩, ⟨default, S3⟩ |
| **S3** | ⟨A, S1⟩, ⟨B, S2⟩, ⟨C, local⟩, ⟨default, S4⟩ |
| **S4** | ⟨A, S3⟩, ⟨B, S3⟩, ⟨C, S3⟩, ⟨D, local⟩, ⟨E, local⟩, ⟨default, S5⟩ |

For S1 and S2 the default (next hop S3) covers B, C, D, E, so those entries are dropped.
S3's default (next hop S4) covers D, E. S4's A/B/C go via S3, not the default's S5, so they stay.

---

### Ex 6.0 — Four switches in a square

```
S1──────S2
│       │
│       │
S4──────S3
```

**(a) Adjacent via the link; diagonal clockwise** (clockwise = S1→S2→S3→S4→S1):

| | S1 | S2 | S3 | S4 |
|--|----|----|----|----|
| S1 | local | S2 | S2 | S4 |
| S2 | S1 | local | S3 | S3 |
| S3 | S4 | S2 | local | S4 |
| S4 | S1 | S1 | S3 | local |

**(b) S1–S4 link not used at all:**

| | S1 | S2 | S3 | S4 |
|--|----|----|----|----|
| S1 | local | S2 | S2 | S2 |
| S2 | S1 | local | S3 | S3 |
| S3 | S2 | S2 | local | S4 |
| S4 | S3 | S3 | S3 | local |

---

### Ex 7.0 — Reading forwardable tables backwards

Given `S2: ⟨S1,S1⟩ ⟨S3,S3⟩ ⟨S4,S3⟩` and `S3: ⟨S1,S2⟩ ⟨S2,S2⟩ ⟨S4,S4⟩`.

**(a)** S2's next hops are S1 and S3 ⇒ S2 is adjacent to S1 and S3. S3's next hops are S2 and
S4 ⇒ S3 is adjacent to S2 and S4. So the topology is the **line S1–S2–S3–S4** (S1 and S4 are
*not* directly connected), and the tables are consistent: S2 reaches S4 via S3, S3 reaches S1
via S2. **Yes**, such a network exists.

**(b)** New `S3: ⟨S1,S4⟩ ⟨S2,S2⟩ ⟨S4,S4⟩`. S3 is adjacent to S2 and S4, and reaches S1 **via S4**.
Since S1–S4 is forbidden, S4 cannot forward the packet straight to S1. Add an **S4–S2** link, so
the path continues S3→S4→**S2**→S1. Consistent topology (edges S1–S2, S2–S3, S3–S4, S4–S2):

```
S1──S2──S3
    │   │
    └S4─┘
```

A loop-free table for S4 is: S1→S2, S2 local, S3 local.

---

### Ex 8.0 — Why a single loop cannot contain both A and C

Topology fragment `... ── A ──── B ──── C ── ...`, where the **only** path between A and C
passes through B.

**(a)** If A and C are both on a loop for destination D, the loop contains a path A→…→C and a
path C→…→A. Each must pass through B (the unique A–C connector), so the packet reaches B twice.
But forwarding is deterministic per destination: whenever the packet is at B it takes the *same*
next hop. So after the first pass through B it follows a fixed path; if it returns to B it simply
repeats and can never reach A again — contradicting that A is on the loop. Hence no single loop
contains both A and C.

**(b)** Suppose `A ── S₁ ── S₂ ── … ── Sₙ ── A` with some `Sᵢ = Sⱼ` (i<j). Again, forwarding is
deterministic, so once the packet returns to the same switch `Sᵢ` it repeats the identical
sequence and loops among `Sᵢ…Sⱼ` forever, never returning to A. Contradiction. So **all Sᵢ are
distinct**. Corollary: a forwarding loop is either (i) a 2-cycle between two adjacent switches,
or (ii) a genuine cycle in the network graph; "linear" loops longer than a 2-cycle are impossible.

---

### Ex 9.0 ♢ — Trace packets from S1

Paths (switches only), using the given tables:

| Dest | Path |
|------|------|
| (a) A | S1 – S4 – S10 |
| (b) B | S1 – S2 – S5 – S11 |
| (c) C | S1 – S4 – S5 – S6 – S12 |
| (d) D ♢ | S1 – S2 – S5 – S6 – S12 |
| (e) E | S1 – S2 – S3 – S6 – S5 – S4 – S10 |
| (f) F | S1 – S4 – S5 – S2 – S3 – S6 – S12 |

---

### Ex 10.0 — Lowest-cost paths with link weights

**(a) A→F ♢:** A–S1–S2–S5–S6–F, cost 1+2+1 = **4** (alternative via S1–S4 costs 5).
**(b) A→D:** A–S1–S2–S5–S4–D, cost 1+2+1 = **4**.
**(c) A→C:** A–S1–S2–S5–S6–S3–C, cost 1+2+1+4 = **8**.

**(d) S2 forwarding table (all cheapest):**

| Dest | Cost | next hop | | Dest | Cost | next hop |
|------|------|----------|-|------|------|----------|
| A | 1 | S1 | | E | 2 | S5 |
| B | 0 | local | | F | 3 | S5 |
| C | 7 | S5 | | S1 | 1 | S1 |
| D | 3 | S5 | | S3 | 7 | S5 |
| (C via S3 direct = 8, so 7 via S5 wins) | | | | S4 | 3 | S5 |
| | | | | S5 | 2 | S5 |
| | | | | S6 | 3 | S5 |

---

### Ex 11.0 — Weight assignments that make E's / F's routes optimal

**(a) E's route** (Ex 9.0(e)) is S1→S2→S3→S6→S5→S4. It avoids the links **S1–S4** and **S2–S5**.
Set every link weight = 1 **except S1–S4 = 10 and S2–S5 = 10**. Then E's route costs 5, and any
route using a "bad" link costs ≥10.

**(b) F's route** (Ex 9.0(f)) is S1→S4→S5→S2→S3→S6. It avoids **S1–S2** and **S5–S6**.
Set all weights = 1 **except S1–S2 = 10 and S5–S6 = 10**.

(Two different weight sets are needed because the two routes use disjoint "bad" links.)

---

### Ex 12.0 — Three Class-C networks joined by R1–R4

Topology: R1–R4, R4–R2, R4–R3; R1 hosts 200.0.1/24, R2 hosts 200.0.2/24, R3 hosts 200.0.3/24.

| Router | 200.0.1/24 | 200.0.2/24 | 200.0.3/24 |
|--------|-----------|-----------|-----------|
| **R1** | local | R4 | R4 |
| **R2** | R4 | local | R4 |
| **R3** | R4 | R4 | local |
| **R4** | R1 | R2 | R3 |

---

# Chapter 2 — Ethernet Basics

### Ex 1.0 — Simulate the contention period (five stations)

Model: on the *n*-th collision choose `k ∈ {0 … 2ⁿ−1}`, retransmit at `collision_detected + k`
slots (one slot to detect). With the coin flips below (one valid run):

| Time | Who transmits | Result | Collision detected | n | Backoff k (→ next attempt) |
|------|---------------|--------|--------------------|---|-----------------------------|
| T=0 | A,B,C,D,E | **collision** | T=1 | 1 | A=1(→T=2); B=0,C=0,D=0,E=0(→T=1) |
| T=1 | B,C,D,E | **collision** | T=2 | 2 | B=3,D=3(→T=5); C=0,E=0(→T=2) |
| T=2 | A,C,E | **collision** | T=3 | A=2,C=3,E=3 | A=0(→T=3); C=2(→T=5); E=6(→T=9) |
| T=3 | **A** | **SUCCESS** | — | — | A transmits; contention ends |

(A's first backoff was k=1, so it waited out round T=1 and only rejoined at T=2.)

---

### Ex 2.0 — Which switches see each frame?

Topology: A–S1, B–S2, C–S3, D–S3, S1–S2, S2–S3. All tables start empty.

| Step | Packet | Switches that see it | Learning |
|------|--------|----------------------|----------|
| (a) | A→B | S1, S2, S3 (all flood) | every switch learns A |
| (b) | B→A | S1, S2 (S2 knows A→S1) | S2, S1 learn B (S3 does **not** see it) |
| (c) | C→B | S3, S2 (S3 floods, S2 knows B) | S3 learns C; S1 does **not** see it |
| (d) | C→D | S3 only (S3 knows D) | S3 forwards straight to D |

---

### Ex 3.0 ♢ — Learning tables after four transmissions

Topology:
```
      B
      │
      S4
      │
A───S1────────S2────────S3───C
      │
      D
```
Links: A–S1, S1–S2, S2–S3, S3–C, B–S4, S4–S2, D–S2.
Transmissions: A→D, D→A, A→B, B→D.

- A→D: D unknown at S1, S2 ⇒ flooded everywhere ⇒ **S1,S2,S3,S4 all learn A**.
- D→A: S2 knows A (via S1) ⇒ forwarded only toward A ⇒ **S2 learns D; S1 learns D**; S3,S4 don't see it.
- A→B: B unknown ⇒ flooded ⇒ nothing new is learned.
- B→D: S4 floods (D unknown), but S2 knows D ⇒ **S4 learns B, S2 learns B**; S1,S3 don't see it.

Final tables:

| Switch | Known destinations |
|--------|--------------------|
| S1 | A, D |
| S2 | A, B, D |
| S3 | A |
| S4 | A, B |

(Matches the book.)

---

### Ex 4.0 — Same topology, different transmissions

Transmissions: A→B, B→A, C→B, D→A.

- A→B: B unknown ⇒ flooded ⇒ S1,S2,S3,S4 learn A.
- B→A: S4 learns B; S4 knows A via S2 ⇒ S2 learns B; S1 learns B; S3 doesn't see.
- C→B: S3 learns C; S2 knows B via S4 ⇒ forwards to S4; S4 learns C.
- D→A: S2 learns D; S2 knows A via S1 ⇒ S1 learns D; S4 doesn't see.

| Switch | Known destinations |
|--------|--------------------|
| S1 | A, B, D |
| S2 | A, B, C, D |
| S3 | A, C |
| S4 | A, B, C |

---

### Ex 5.0 — Two transmissions that make A→D flooded by S2 but not by S3

Topology: A–S1, S1–S2, S2–S3, S3–D; B–S2; C–S3.
Send, in order:

1. **C → A**: S3 learns C (C unknown to it? A unknown) — S3 floods to S2 and D, so **S2 learns C**.
2. **D → C**: S3 now **knows C**, so it forwards straight to C only — **S2 does not see D**.

Now S3 knows D but S2 does not. Finally:

3. **A → D**: S2 does not know D ⇒ **floods** (so **B sees the packet**); reaching S3, S3 *knows* D
   ⇒ forwards only to D (so **C does not see it**). ∎

(Equally valid: C→B then D→C.)

---

### Ex 6.0 — Why the described behaviour is impossible

```
    ?         ?
    |         |
A───S1────────S2───B
```

For S1 to *forward* A→B directly to S2 (rather than flood), S1 must have learned **B via S2**,
i.e. some earlier frame with source B arrived at S1 from S2. Consider that frame: to reach S1 it
arrived at S2 on one of S2's ports and was forwarded by S2 (or originated on S2's B-side), so S2
had source B on some port and therefore **S2 already knows B**. But S2 flooding A→B means S2 does
**not** know B. Contradiction. Hence S1 can forward to S2 while S2 floods only if two switches
disagree, which the learning algorithm prevents.

---

### Ex 7.0 — D attached to S5; D→B

Topology (2.4.1):
```
A──S1────S2────S3──B
         │
         C──S4──S5
```
D is attached to S5. Known before D→B: S5 knows A,C; S4 knows A,C; S2,S3 know A,B; S1 knows A.

D→B path of decisions:

| Switch | Sees D→B? | Learns D? | Knows B? | Action |
|--------|-----------|-----------|----------|--------|
| S5 | yes | yes | no | **flood** (→ S4) |
| S4 | yes | yes | no | **flood** (→ S2, C) |
| S2 | yes | yes | yes | forward → S3 |
| S3 | yes | yes | yes | forward → B |
| S1 | no | no | — | — |

So the switches that **see** D→B are **S5, S4, S2, S3**; those that **flood** (don't know B) are
**S5 and S4**.

---

### Ex 8.0 — Two switches wired in a loop

S1 and S2 joined by **two** links (interfaces 1 and 2). A→B, B unknown: S1 floods out both
interfaces, so S2 receives the frame on both links; S2 floods each copy back out both of *its*
links and toward its hosts; S1 receives and re-floods, etc.

Because **Ethernet frames have no TTL**, the copies keep circulating (and multiply) **forever** —
a broadcast storm that saturates the loop. Nothing stops it except removing a link or running the
spanning-tree algorithm. (This is exactly why Ethernet forbids active loops.)

---

### Ex 9.0 — Propagating VLAN "colour" with each destination

**(a)** Extend normal learning to `⟨address, port, colour⟩`. A switch knows each host-port's
colour, and on a trunk the colour of received frames is carried in the 802.1Q tag. So a switch can
record the colour of each learned source address, and share `⟨address, colour⟩` with the other
switches (a GARP/GVRP/VTP-style protocol) along the trunk links / spanning tree, so every switch
builds a consistent address→colour map. (Equivalently, keep separate per-VLAN forwarding tables,
one per colour, and run learning independently in each.)

**(b)** If a former switch-to-switch port is reused for a host: (i) re-designate it as an **access
port** for the appropriate VLAN (set its colour/PVID); (ii) **expire/delete** all
`⟨address, colour⟩` entries previously learned via that port; (iii) notify the other switches
(via the sharing protocol) so they purge stale colour associations; and (iv) revisit trunk/STP
configuration so the port is no longer treated as a trunk.

---

### Ex 10.0 — The ARP variant of 2.4.1

Before each of the three transmissions the sender **broadcasts** (flooded ⇒ every switch learns
the sender), then the destination **unicast-replies**. Same topology as Ex 3.0/7.0.

| Round | Broadcast → all switches learn | Unicast reply → switches on the path learn |
|-------|-------------------------------|--------------------------------------------|
| A→B | A (all of S1–S5) | B (S3, S2, S1) |
| B→A | B (all of S1–S5) | A (already known) |
| C→B | C (all of S1–S5) | B (already known) |

Final tables:

| Switch | Known destinations |
|--------|--------------------|
| S1 | A, B, C |
| S2 | A, B, C |
| S3 | A, B, C |
| S4 | A, B, C |
| S5 | A, B, C |

(The broadcasts teach every switch the sender, so all five switches know A, B and C.)

---

# Chapter 3 — Advanced Ethernet

### Ex 1.0 — Spanning tree pruning (root = S1)

Topology (a 2×3 ladder):
```
S1──────S4──────S6
│       │       │
│       │       │
S3──────S5──────S2
```
Edges: S1–S4, S4–S6, S1–S3, S4–S5, S6–S2, S3–S5, S5–S2. Root = S1 (smallest ID).

Cost to root / chosen parent (tie → **smaller neighbour ID**):

| Switch | cost | parent |
|--------|------|--------|
| S3 | 1 | S1 |
| S4 | 1 | S1 |
| S6 | 2 | S4 |
| S5 | 2 | **S3** (tie S3 vs S4 → 3 < 4) |
| S2 | 3 | **S5** (tie S5 vs S6 → 5 < 6) |

**Pruned (disabled): S4–S5 and S6–S2.** Surviving tree (5 links):

```
S1──────S4──────S6
│
S3──────S5──────S2
```

---

### Ex 2.0 — Datacenter first two rows

Topology: top row S1,S2; bottom row S3,S4,S5; fully bipartite (S1 and S2 each connect to
S3, S4, S5).

**(a) Root S1 (ID 1).** S1 keeps all three links (S1–S3, S1–S4, S1–S5). S2 must pick the
lowest-ID neighbour toward the root → **S3**, so keep S2–S3; **prune S2–S4 and S2–S5**.
Surviving links: S1–S3, S1–S4, S1–S5, S2–S3.
**Path S2 → S5: S2 – S3 – S1 – S5.**

**(b) Root S4 (ID 0).** S4 keeps both links S4–S1, S4–S2. S1 and S2 each use S4 as parent
(1 hop). S3's neighbours S1,S2 tie at cost 2 → pick **S1**; likewise S5 → **S1**.
Keep S4–S1, S4–S2, S1–S3, S1–S5; prune S2–S3, S2–S5.
**Path S1 → S5: S1 – S5 (direct).**

---

### Ex 3.0 — VLAN colour propagation

Identical to Ex 2.0/9.0 above; see that answer. (Both chapters include the same exercise.)

---

### Ex 4.0 — Distance-vector forwarding on a looped Ethernet

**(a)** Treat switches and directly attached hosts as graph nodes, each link cost 1. Run the
distance-vector algorithm (Ch. 13): every switch learns the minimum cost and next switch for
every host, without any host cooperation. With directly attached hosts identified (part b), a
switch's table is just the DV result restricted to reachable hosts. Because DV converges to
shortest paths and its tables are consistent (ignoring transient loops), this yields optimal
forwarding **even when the topology has loops**.

**(b)** Let switches **mark** every frame they forward (e.g. set a reserved bit). Hosts cannot mark.
So a frame that arrives **already marked** came from another switch, while an **unmarked** frame
came from a directly attached (non-switch) host. A switch therefore concludes that a source
address seen only in unmarked arrivals is *directly connected*.

**(c)** **Broadcast and multicast remain a problem.** They must be delivered to *every* host, so
there is no single next hop — switches must still flood. On a looped topology flooding storms
forever, so broadcast/multicast still needs a spanning tree (or some loop-suppression), which DV
unicast routing does not provide.

---

### Ex 5.0 ♢ — OpenFlow single-table learning (report unknown *destination* only)

Switch S, hosts h1→port 1, h2→port 2, h3→port 3. Table starts empty.

| Packet | Reported? | New rule installed by C | Then |
|--------|-----------|--------------------------|------|
| (a) h1→h2 | **yes** (h2 unknown) | h1 reachable via port 1 | **flood** |
| (b) h2→h1 | no (h1 known) | — | forward via port 1 |
| (c) h3→h1 | no (h1 known) | — | forward via port 1 |
| (d) h2→h3 | **yes** (h3 unknown) | h2 reachable via port 2 | **flood** |

(Matches the book.)

---

### Ex 6.0 — Same, but report if *source or* destination is unknown

| Packet | Reported? | New rule | Then |
|--------|-----------|----------|------|
| (a) h1→h2 | yes (h1,h2 unknown) | h1 via port 1 | flood |
| (b) h2→h1 | yes (h2 source unknown) | h2 via port 2 | forward via port 1 (h1 known) |
| (c) h3→h1 | yes (h3 source unknown) | h3 via port 3 | forward via port 1 |
| (d) h2→h3 | no (both known) | — | forward via port 3 |

---

### Ex 7.0 ♢ — Three switches S1–S3, three hosts h1–h3

Topology: h1–S1, h2–S2, h3–S3, and switches in a **line S1–S2–S3**; all report to controller C.
Rule: report only when the **destination** is unknown.

Key reasoning:
- **h1** is learned by all switches: the very first packet h1→h2 is flooded all the way, so every
  switch sees source h1.
- **h2** is learned by all switches: the packet h2→h3 (dest h3 unknown) is reported and flooded,
  so every switch sees source h2.
- **h3 is never learned**: whenever h3 is the source (h3→h1, h3→h2), the destination (h1 or h2) is
  **already known**, so no report is generated, so the controller never installs a rule for h3.

Final tables:

| Switch | Known destinations |
|--------|--------------------|
| S1 | h1, h2 |
| S2 | h1, h2 |
| S3 | h1, h2 |

(The book's solution gives S1 explicitly: entries for h1 and h2, but **not** h3.)

---

### Ex 8.0 — Reverse the OpenFlow two-table match fields

Original:

| Table | match field | match action | no-match default |
|-------|-------------|--------------|------------------|
| T0 | destaddr | forward, then send to T1 | flood, then send to T1 |
| T1 | srcaddr | do nothing | send to controller |

**Reversed** (T0 matches source, T1 matches destination):

| Table | match field | match action | no-match default |
|-------|-------------|--------------|------------------|
| **T0** | **srcaddr** | forward, then send to T1 | flood, then send to T1 |
| **T1** | **destaddr** | do nothing | send to controller |

---

# Chapter 4 — Wireless LANs

### Ex 1.0 — An ACK sent while the medium is busy, with no collision

Use the **hidden-node** layout: A and B cannot hear each other, but both are in range of C
(C is the middle station).

1. A finishes transmitting a data frame to C.
2. C is about to send its ACK after SIFS (it does **not** sense the medium first).
3. B — unable to hear A (hidden node), so it saw the medium as idle — starts transmitting.

Why no collision occurs: a simultaneous transmission is only a "collision" if some station fails
to read a signal it needs.
- At **A**: it hears only C's ACK (B is hidden from A), so the ACK is decoded correctly.
- At **C**: C is transmitting, so it does not attempt to receive.
- At **B**: B is transmitting, so it does not attempt to receive.
So the **ACK is not corrupted**. (B's own frame may be lost at C, but that is a separate matter —
it is exactly the price of the hidden-node problem, which RTS/CTS exists to fix.)

More generally, the ACK is safe whenever no receiver lies in range of both C's ACK and B's signal
at the same time.

---

### Ex 2.0 ♢ — Three-sender hidden-node collision

Place the three senders A, B, C at the vertices of an **equilateral triangle** and the receiver D
at the **centre**, with D within range of all three but the senders mutually out of range. Then any
two or three transmitting at once collide at D, and none can hear the others.

**More than three senders:** with **impermeable walls** you can build an arrangement for any
number (walls block the senders from one another while all reach D). In open space, senders can be
put at the vertices of a square / tetrahedron / possibly a pentagon around D, but geometry
inevitably limits how many pairwise-hidden senders can all reach one point.

---

### Ex 3.0 ♢ — Contention cost at 802.11g

Slot time = **20 µs** (the book's value). Packet = 1 kB = 8192 bits at 54 Mbps ⇒ ≈ 151 µs.

**(a)** Average contention interval = 64 slots × 20 µs = **1280 µs ≈ 1.28 ms**.

**(b)** Transmission and contention alternate: 151 µs of data, then the contention interval, then
151 µs, … So the fraction of time (hence bandwidth) lost to contention is

```
T / (T + 151) = 1280 / (1280 + 151) = 1280 / 1431 ≈ 0.894  →  ≈ 89% lost
```

(If you use the 802.11g "short slot" of 9 µs instead: T = 576 µs, fraction = 576/727 ≈ 79%.)

---

### Ex 4.0 — How Wi-Fi stations might react to other stations' frames

Stations can react *without transmitting*, because a transmission is only one kind of response:

- **Same access point:** set/refresh the **NAV** from the Duration field (virtual carrier sense);
  defer until the medium is free; keep re-drawing the random backoff; if they hear a frame they
  were not the destination of, treat the medium as busy for the IFS.
- **Different access points (overlapping cells):** still defer while a frame is in progress (so as
  not to collide at the AP boundary); back off / pick a different channel; reduce power; schedule
  around a neighbour AP's beacons.
- **General:** use incoming frames purely as information — update timing, choose backoff, stay off
  the channel — rather than replying. Responses cannot be *required*, because reception between
  non-AP stations is often unreliable (spotty).

---

### Ex 5.0 — WiMAX ranging and guard intervals

Speed of a subscriber = up to 33 m/s; ranging fixes the propagation delay once a minute.

**(a)** In 60 s a station moves 33 × 60 = **1980 m**. Propagation-time change = 1980 / 300 =
**6.6 µs**, so a packet can arrive up to 6.6 µs early or late.

**(b)** The base station cannot know whether the shift is early or late, so allow **2 × 6.6 =
13.2 µs** per sender. With 5000 senders/s: 5000 × 13.2 = **66 000 µs = 0.066 s per second**,
i.e. **6.6%** of each second is spent on guard intervals.

---

### Ex 6.0 — Two-ray ground model

Signal: `(A/r) sin(2πft − (2π/λ) r)`; reflected path is phase-reversed (180°) and delayed by `r′−r`.

**(a)** With sender and receiver at height `h` and horizontal distance `r`:
```
r′ = 2·√( h² + (r/2)² ) = r·(1 + (2h/r)²)^(1/2)  ≈  r·(1 + 2h²/r²)  =  r + 2h²/r
```
so `r′ − r ≈ 2h² / r`.

**(b)** Write the sum as
```
(A/r)sin(θ) − (A/r′)sin(θ′)  with θ = 2πft − (2π/λ)r,  θ′ = θ − Δ,  Δ = (2π/λ)(r′−r)
= (A/r − A/r′) sin θ + (A/r′)[ sin θ − sin(θ − Δ) ]
≈ (A/r − A/r′) sin θ + (A/r′) · Δ · cos θ        (sin θ − sin(θ−Δ) ≈ Δ cos θ)
```
Now `A/r − A/r′ = A(r′−r)/(rr′) ≈ A(2h²/r)/r² = 2Ah²/r³`, and `(A/r′)Δ ≈ (A/r)(2π/λ)(2h²/r) = 4πAh²/(λ r²)`.

**(c)** So
```
sum ≈ (2Ah²/r³) sin θ + (4πAh²/(λ r²)) cos θ
```
For large `r` the second term dominates, giving amplitude ∝ **1/r²**. Power ∝ amplitude² , hence
power ∝ **1/r⁴**. ∎

---

### Ex 7.0 — Where the Wi-Fi (WPA2) replay attack breaks

Four-way handshake: AP→**ANonce**; STA→**SNonce + MIC**; AP→**GTK + install + MIC**;
STA→**ACK**.

B replays A's recorded packets. The AP initiates a **fresh** handshake with a **new ANonce**
(and fresh randomness). When B replays A's old message 2 (SNonce + MIC), the **MIC was computed
over the old ANonce**, so the AP's integrity check **fails at message 2** and the handshake aborts.
The fresh per-handshake nonces are exactly what defeat the replay. ∎

---

# Chapter 5 — Other LANs

### Ex 1.0 — IP-based VPN routing

Setup: A has eth0 = 12.1.2.3 and ppp0 = 10.0.0.44 (VPN to B at 200.0.0.7). Packet to 200.0.0.7
goes via eth0; everything else via ppp0 (encapsulated to B).

**(a)** Intruder M connects to A at 12.1.2.3. M's packets to A arrive **directly** on A's eth0. But
A's *replies* to M are "traffic to anywhere else", so they are routed into **ppp0**, encapsulated,
and sent to the VPN server B, which then forwards them out to M over the ordinary Internet.
So the connection's two directions take different paths — inbound direct, outbound through the
tunnel (B) — which is asymmetric (triangle routing). If the tunnel's addressing/policy hides A's
real source, M cannot complete the connection. This is why the VPN is normally configured to make
the tunnel the **default route**: then A's reply to any outsider goes through the workplace, so
outsiders cannot reach A via the residential connection at all.

**(b)** If **all** outbound traffic uses ppp0, then even the *tunnel* packets (outer packets,
addressed to 200.0.0.7) are routed via ppp0 — i.e. they get encapsulated and sent to B again, and
so on. A's traffic never actually reaches the real network: every packet, including the ones
carrying the tunnel, is fed back into the tunnel — a routing/encapsulation **loop**. A must keep a
**host-specific route** sending 200.0.0.7 out eth0 (the real interface), with only the *rest*
going via ppp0.

---

### Ex 2.0 — TCP-VPN via a NATed virtual machine

A runs VM V (10.0.0.2) on its virtual net (A = 10.0.0.1); V runs the VPN client and is NATed by A.
V reaches B "the usual way" through A; V encapsulates everything else toward B.

**Can A itself use the VPN?** **Yes — but only with a carefully specific route.** A must send its
own Internet-bound traffic to **V (10.0.0.2)** so V can encapsulate it, *while retaining a
host-specific route* for the VPN server's address (200.0.0.7) out the **real** interface. Then:

- A's packet to some Internet host X → default route → V → V encapsulates (outer dst = B) and
  hands it to A (its NAT router) → A matches the host route for B → sends it out the real link → B
  → B decapsulates and forwards the inner packet to X.
- Replies come back to B, are encapsulated to V, arrive at A, and A forwards them to V (the virtual
  link is directly connected) → V decapsulates → delivers to A.

**Without** the host-specific route for B, A would send the *outer* (tunnelled) packets back to V,
producing an infinite encapsulation loop. So the answer is *yes if* the tunnel-to-server traffic
bypasses the tunnel default route. (This is the same "host-specific forwarding-table entry"
trick described in §5.1.)

---

### Ex 3.0 ♢ — Token Bus timing and self-ordering

Token = 64 bytes = 512 bits at 10 Mbps ⇒ each token transmission lasts **51.2 µs**.

**(a)** On an idle network the token circulates through all 40 stations; the average number of
stations the token must pass before reaching a station that wants to send is 40/2 = 20. Ignoring
propagation and the inter-frame gap:

```
average wait = 20 × 51.2 µs = 1024 µs ≈ 1.02 ms
```

**(b)** Self-ordering protocol: **order the stations by the numeric value of their MAC
addresses.** The station with the smallest MAC address becomes S₀, the next-smallest S₁, …, and
Sᵢ passes the token to S₍ᵢ₊₁₎ mod ₙ. (A bidding/claim process can also be used: stations
progressively exclude themselves until a single winner is agreed, then the winner hands the token
to the next-lower address, building the ring.)

---

### Ex 4.0 — SM90 XOR-recovery cell: break-even loss rate

- **Without SM90:** N = 20 cells. Each cell is lost with probability p, so the whole packet is
  lost with probability ≈ 20p; a retransmission costs another 20 cells ⇒ expected extra = 20p × 20
  = 400p cells. Base = 20, so total ≈ **20 + 400p**.
- **With SM90:** always **21 cells** (20 data + 1 XOR).

Set equal:
```
20 + 400p = 21   ⇒   400p = 1   ⇒   p = 1/400 = 0.0025 = 0.25%
```
So the two schemes involve about the same number of cell transmissions when the per-cell loss rate
**p ≈ 0.25%**.

---

### Ex 5.0 — VCI table for S5 (from the §5.4 example)

In the example, S4's outgoing VCIs toward S5 are 8 (connection A→F #1) and 9 (A→F #2); on the
S5–F link they become 5 and 2 respectively. Hence:

| VCI in | port in | VCI out | port out |
|--------|---------|---------|----------|
| 8 | S4 | 5 | F |
| 9 | S4 | 2 | F |

(Ports are shown by neighbour; numeric port numbers depend on the switch's own port labelling.)

---

### Ex 6.0 — Identify all connections beginning at A

Tables:

**S1** | **S2** | **S3** | **S4**
---|---|---|---
1,A,2,S3 | 2,S4,1,B | 2,S1,2,S4 | 2,S3,2,S2
2,A,2,S2 | 2,S1,3,S4 | 3,S4,2,C | 3,S2,3,S3
3,A,3,S2 | 3,S1,4,S4 | | 4,S2,1,D

Following each incoming VCI at S1:

```
Connection 1 (A, VCI 1): A–1–S1–2–S3–2–S4–2–S2–1–B
Connection 2 (A, VCI 2): A–2–S1–2–S2–3–S4–3–S3–2–C
Connection 3 (A, VCI 3): A–3–S1–3–S2–4–S4–1–D
```

(The number between nodes is the VCI on that link.)

---

### Ex 7.0 ♢ — Identify all connections

Tables:

**S1**: (1,A,2,S2), (3,S3,2,A) · **S2**: (2,S1,3,S4), (1,B,2,S4) ·
**S3**: (2,S1,2,S4), (1,S4,3,S1) · **S4**: (3,S2,2,D), (2,S2,1,S3)

There are exactly two connections:

```
A–1–S1–2–S2–3–S4–2–D
B–1–S2–2–S4–1–S3–3–S1–2–A
```

(Matches the book.)

---

### Ex 8.0 — Build shortest-path VC tables for four connections

Topology: A–S1, S1–S2, S2–B, C–S3, S3–S4, S4–D, S1–S3, S2–S4.

Connections and a consistent set of link VCIs:

```
(a) A–D  via S2:  A–1–S1–2–S2–3–S4–4–D      (4 hops)
(b) C–B  via S4:  C–1–S3–2–S4–3–S2–4–B      (C–S3–S4–S2–B)
(c) B–D:          B–1–S2–5–S4–6–D
(d) A–D  via S3:  A–2–S1–7–S3–8–S4–9–D      (uses the corner not used in (a))
```

Switch tables (⟨VCI_out, port_out⟩ keyed by ⟨VCI_in, port_in⟩):

| Switch | VCI in | port in | VCI out | port out |
|--------|--------|---------|---------|----------|
| **S1** | 1 | A | 2 | S2 |
| | 2 | A | 7 | S3 |
| **S2** | 2 | S1 | 3 | S4 |
| | 4 | B | 5 | S4 |
| | 3 | S4 | 4 | B |
| **S3** | 1 | C | 2 | S4 |
| | 7 | S1 | 8 | S4 |
| **S4** | 3 | S2 | 4 | D |
| | 8 | S3 | 9 | D |
| | 2 | S3 | 3 | S2 |
| | 5 | S2 | 6 | D |

(Values are one consistent assignment; other consistent labellings are equally valid as long as
a VCI is unique per link and each ⟨VCI_in, port_in⟩ is unique.)

---

### Ex 9.0 — A virtual circuit that visits each switch twice

Required path: `A → S1 → S2 → S4 → S3 → S1 → S2 → S4 → S3 → B`.

Topology: A–S1, S1–S2, S2–; B–S3, S3–S4; S1–S3, S2–S4.

Because switches are revisited, the two visits to S1 (and to S2, S4, S3) must be distinguished by
different incoming VCIs. Link VCIs (in path order): A–S1 = 1, S1–S2 = 2, S2–S4 = 3, S4–S3 = 4,
S3–S1 = 5, S1–S2 = 6, S2–S4 = 7, S4–S3 = 8, S3–B = 9.

| Switch | VCI in | port in | VCI out | port out |
|--------|--------|---------|---------|----------|
| **S1** | 1 | A | 2 | S2 |
| | 5 | S3 | 6 | S2 |
| **S2** | 2 | S1 | 3 | S4 |
| | 6 | S1 | 7 | S4 |
| **S4** | 3 | S2 | 4 | S3 |
| | 7 | S2 | 8 | S3 |
| **S3** | 4 | S4 | 5 | S1 |
| | 8 | S4 | 9 | B |

Trace: A–1–S1–2–S2–3–S4–4–S3–5–S1–6–S2–7–S4–8–S3–9–B. ∎