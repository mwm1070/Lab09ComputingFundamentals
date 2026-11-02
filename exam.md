# Lab 09 Quiz — Studying with LLM

**CS 2010 Fall 2026** · Textbook: *An Introduction to Computer Networks* (intronetworks.cs.luc.edu), chapters 1–5

- **Time limit:** 10 minutes (12 minutes recommended)
- **Allowed materials:** your one-page written cheat sheet
- **No LLM assistance** — this quiz is the LLMs-down part of the lab
- **Point values:** 10 points per question, 50 points total

All questions are exercises from the textbook. Show your work on the calculation questions; partial credit is given for correct setup.

---

## Question 1 (10 pts) — Textbook Ch. 1, Exercise 1.0

Give forwarding tables for each of the switches S1–S4 in the following network with destinations A, B, C, D. For the next_hop column, give the neighbor on the appropriate link rather than the interface number.

```
A B C
│ │ │
S1────────S2────────S3
 │
 │
 S4───D
```

**S1:**

| destination | next_hop |
|-------------|----------|
|      A       |    S1   |
|      B       |    S1   |
|      C       |    S1   |
|      D       |    S4   |

**S2:**

| destination | next_hop |
|-------------|----------|
|        A     |     local  |
|        B     |       S2   |
|        C     |       S2   |
|        D     |       S4   |

**S3:**

| destination | next_hop |
|-------------|----------|
|      A       |     local  |
|      B       |      S2    |
|      C       |      S2    |
|      D       |      S4    |

**S4:**

| destination | next_hop |
|-------------|----------|
|     A        |    local  |
|     B       |      S1    |
|     C        |     S1    |
|     D        |     S4    |

---

## Question 2 (10 pts) — Textbook Ch. 2, Exercise 2.0

Suppose we have Ethernet switches S1 through S3 arranged as below; each switch uses the learning algorithm of Chapter 2. All forwarding tables are initially empty.

```
S1────────S2────────S3───D
│ │ │
A B C
```

For each part, list which switches see the packet.

**(a)** If A sends to B, which switches see this packet?
Answer: S1

**(b)** If B then replies to A, which switches see this packet?
Answer: S1

**(c)** If C then sends to B, which switches see this packet?
Answer: S2

**(d)** If C then sends to D, which switches see this packet?
Answer: S2 and S3
---

## Question 3 (10 pts) — Textbook Ch. 3, Exercise 1.0

The following network is like that of §3.1.1 Spanning Tree Example 1: Switches Only, except that the switches are numbered differently. Again, the ID of switch S_n is n, so S1 will be the root. Which links end up "pruned" by the spanning-tree algorithm, and why?

```
S1──────S4──────S6
│ │ │ │ │ │
│ │ │ │ │ │
│ │ │ │ │ │
S3──────S5──────S2
```

**Pruned links: The links between S1 and S4, S3 and S5, as well as the extra one to the right of S4 and S5**

**Why: Because the Spanning Tree does not need those extra paths to send information**

---

## Question 4 (10 pts) — Textbook Ch. 4, Exercise 3.0

Suppose the average contention interval in a Wi-Fi network (802.11g) is 64 SlotTimes. The average packet size is 1 kB, and the data rate is 54 Mbps. At that data rate, it takes about (8×1024)/54 = 151 µsec to transmit a packet.

**(a)** How long is the average contention interval, in µsec?

**(b)** What fraction of the total potential data rate is lost to contention?

Show your work.

---

## Question 5 (10 pts) — Textbook Ch. 5, Exercise 3.0(a)

Token Bus was a proprietary Ethernet-based network. It worked like Token Ring in that a small token packet was sent from one station to the next in agreed-upon order, and a station could transmit only when it had just received the token.

If the data rate is 10 Mbps and the token is 64 bytes long (the 10-Mbps Ethernet minimum packet size), what is the average wait to receive the token on an idle network with 40 stations? (The average number of stations the token must pass through is 40/2 = 20.) Ignore the propagation delay and the gap Ethernet requires between packets.

Show your work.


---

## Question 6 (10 pts) — Textbook Ch. 1, Exercise 12.0

Suppose we have the following three Class C IP networks, joined by routers R1–R4. There is no connection to the outside Internet. Give the forwarding table for each router. For networks directly connected to a router (eg 200.0.1/24 and R1), include the network in the table but list the next hop as direct or local.

```
R1── 200.0.1/24
│
│
│
R4──────────R2── 200.0.2/24
│
│
│
R3── 200.0.3/24
```

**R1:**

| destination | next_hop |
|-------------|----------|
|200.0.1/24   |   direct |
|200.0.2/24   |   R2     |
|200.0.3/24   |   R3     |

**R2:**

| destination | next_hop |
|-------------|----------|
|200.0.1/24   |    R1    |
|200.0.2/24   |    direct|
|200.0.3/24   |    R3    |

**R3:**

| destination | next_hop |
|-------------|----------|
|200.0.1/24   |    R1    |
|200.0.2/24   |    R2    |
|200.0.3/24   |    direct|

**R4:**

| destination | next_hop |
|-------------|----------|
|200.0.1/24   |     R1   |
|200.0.2/24   |     R2   |
|200.0.3/24   |     R3   |
