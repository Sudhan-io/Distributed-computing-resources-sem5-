# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT II: LOGICAL TIME AND GLOBAL STATE
### PART C: 16-MARK ESSAY QUESTIONS (EXHAUSTIVE 5-6 PAGE EXAMINATION ANSWERS)

---

# QUESTION 1
> ### **Explain in detail the framework for a system of logical clocks, Lamport's scalar logical clocks, and Mattern-Fidge vector logical clocks. Compare scalar and vector clocks with mathematical properties, space-time diagrams, and complete update traces.**

---

### 1. CONCEPTUAL FRAMEWORK OF LOGICAL CLOCKS
In distributed computing systems without shared memory or a global physical clock, determining the temporal order of events cannot rely on physical clocks. Leslie Lamport (1978) established that what matters fundamentally in distributed systems is not the physical time an event occurs, but the **causal relationship** between events.

```
+---------------------------------------------------------------------------------------------------------+
|                                  LOGICAL TIME ABSTRACTION ARCHITECTURE                                  |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       REAL PHYSICAL TIME AXIS (Hardware Quartz Clocks drift unpredictably!)                             |
|       t0 -------------------- t1 -------------------- t2 --------------------->                         |
|                                                                                                         |
|       MAPPED VIA CLOCK FUNCTION C(e):                                                                   |
|       Event e_a -----------> C(e_a) (Scalar Integer or Vector Array)                                    |
|                                                                                                         |
|       FUNDAMENTAL CLOCK CONDITION:                                                                      |
|       e_a < e_b   ===>   C(e_a) < C(e_b)                                                                |
+---------------------------------------------------------------------------------------------------------+
```

---

### 2. LAMPORT'S SCALAR LOGICAL CLOCKS

#### A. Mathematical Formulation
A scalar logical clock system assigns a monotonically increasing non-negative integer $C(e)$ to every event $e$.
* **Clock Condition:**
  $$e_i \prec e_j \implies C(e_i) < C(e_j)$$
* **Total Ordering Extension:**
  To break ties between concurrent events having identical scalar timestamps, Lamport introduced a total order relation $\prec$:
  $$e_i \prec e_j \iff (C(e_i) < C(e_j)) \lor (C(e_i) = C(e_j) \land i < j)$$
  where $i$ and $j$ are unique process identifiers.

#### B. Update Protocol Rules:
1. **Rule 1 (Local Event Progress):**
   Before executing an internal event or send event at process $P_i$:
   $$C_i \leftarrow C_i + d \quad (d > 0, \text{ typically } d = 1)$$
2. **Rule 2 (Message Sending):**
   When process $P_i$ transmits message $m$, it attaches its current clock value:
   $$\text{Packet} = (m, ts_m = C_i)$$
3. **Rule 3 (Message Reception):**
   When process $P_j$ receives message $(m, ts_m)$, it updates its local clock:
   $$C_j \leftarrow \max(C_j, ts_m) + d$$

---

### 3. MATTERN-FIDGE VECTOR LOGICAL CLOCKS

#### A. The Limitation of Scalar Clocks
While scalar clocks guarantee $e_i \prec e_j \implies C(e_i) < C(e_j)$, the converse does **not** hold:
$$C(e_i) < C(e_j) \not\implies e_i \prec e_j$$
Two events might have $C(e_i) < C(e_j)$ yet be completely concurrent ($e_i \parallel e_j$).

#### B. Vector Clock Structure & Isomorphism
Vector clocks assign an array of $N$ integers $V(e) = [v_1, v_2, \dots, v_N]$ to every event $e$:
* **Causal Isomorphism:**
  $$e_i \prec e_j \iff V(e_i) < V(e_j)$$
  $$e_i \parallel e_j \iff \neg(V(e_i) \le V(e_j)) \land \neg(V(e_j) \le V(e_i))$$

#### C. Vector Update Protocol Rules:
1. **Rule 1 (Local Event):**
   $$V_i[i] \leftarrow V_i[i] + 1$$
2. **Rule 2 (Message Sending):**
   Process $P_i$ attaches its entire vector timestamp to message $m$:
   $$\text{Packet} = (m, V_m = V_i)$$
3. **Rule 3 (Message Reception):**
   When $P_j$ receives $(m, V_m)$:
   $$\forall k \in [1..N]: \quad V_j[k] \leftarrow \max(V_j[k], V_m[k])$$
   $$V_j[j] \leftarrow V_j[j] + 1$$

---

### 4. SIDE-BY-SIDE SPACE-TIME TRACE WALKTHROUGH

Consider 3 processes $P_1, P_2, P_3$ executing events concurrently.

```
+---------------------------------------------------------------------------------------------------------+
|                            COMPARATIVE TRACE: SCALAR CLOCKS VS. VECTOR CLOCKS                           |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       P1  --- e1,1: C=1 ------------ e1,2: C=2 ------------------------------------->                   |
|               V=[1,0,0]              V=[2,0,0]                                                          |
|                                        \                                                                |
|                                         \ m1 (ts=2 / V=[2,0,0])                                         |
|                                          v                                                              |
|       P2  --- e2,1: C=1 ------------ e2,2: C=3 ------------ e2,3: C=4 ------------->                   |
|               V=[0,1,0]              V=[2,2,0]              V=[2,3,0]                                   |
|                                                               \                                         |
|                                                                \ m2 (ts=4 / V=[2,3,0])                  |
|                                                                 v                                       |
|       P3  -------------------------- e3,1: C=1 ------------ e3,2: C=5 ------------->                   |
|                                      V=[0,0,1]              V=[2,3,2]                                   |
+---------------------------------------------------------------------------------------------------------+
```

* **Detailed Comparison Analysis:**
  1. Compare $e_1^1$ and $e_2^1$:
     * Scalar Clocks: $C(e_1^1) = 1, C(e_2^1) = 1 \implies$ Equal scalar timestamps.
     * Vector Clocks: $V(e_1^1) = [1, 0, 0], V(e_2^1) = [0, 1, 0] \implies [1,0,0] \parallel [0,1,0]$.
     * **Identifies true concurrency ($e_1^1 \parallel e_2^1$)!**
  2. Compare $e_1^1$ and $e_3^1$:
     * Scalar Clocks: $C(e_1^1) = 1, C(e_3^1) = 1 \implies$ Identical values, yet independent.
     * Vector Clocks: $[1,0,0] \parallel [0,0,1] \implies$ Confirms independence.
  3. Compare $e_1^2$ and $e_3^2$:
     * Vector Clocks: $V(e_1^2) = [2,0,0]$ and $V(e_3^2) = [2,3,2]$.
     * Since $[2,0,0] < [2,3,2]$, we mathematically deduce that $e_1^2 \prec e_3^2$ without tracking intermediate hops!

---

### 5. IN-DEPTH COMPARATIVE MATRIX

| Evaluation Dimension | Lamport's Scalar Clocks | Mattern-Fidge Vector Clocks |
| :--- | :--- | :--- |
| **Representation** | Single non-negative integer ($1$ scalar value) | Array of $N$ integers ($N$-dimensional vector) |
| **Space Overhead (Message)** | $O(1)$ scalar in packet header | $O(N)$ integer array attached to each message |
| **Space Overhead (Process)** | $O(1)$ integer variable | $O(N)$ integer array in local memory |
| **Causal Ordering Equivalence**| One-way implication: $e_a \prec e_b \implies C_a < C_b$ | **Exact If-and-only-if:** $e_a \prec e_b \iff V_a < V_b$ |
| **Concurrency Detection** | **Cannot detect concurrency** ($C_a < C_b$ may be concurrent)| **Directly detects concurrency** via incomparable vectors |
| **Dynamic Membership** | Trivial: new nodes join easily | Complex: changing $N$ requires resizing vectors everywhere |
| **Typical Use Cases** | Mutual Exclusion (Lamport/Ricart-Agrawala) | Causally ordered delivery, distributed debugging, Git DAGs |

---
---

# QUESTION 2
> ### **Discuss physical clock synchronization in distributed systems in detail, covering clock drift, Cristian's algorithm, Berkeley algorithm, and the complete architecture, protocol equations, and stratum hierarchy of NTP.**

---

### 1. FUNDAMENTALS OF PHYSICAL CLOCK SYNCHRONIZATION
Physical clock synchronization aims to maintain agreement among hardware clocks across distributed computers, bounded within an acceptable maximum skew $\delta$:
$$|C_i(t) - C_j(t)| \le \delta \quad \forall i, j, \forall t$$

#### Mathematical Clock Drift Model:
* An ideal clock satisfies $\frac{dC}{dt} = 1$.
* Physical quartz crystals experience clock drift within maximum drift rate $\rho$:
  $$1 - \rho \le \frac{dC}{dt} \le 1 + \rho$$
* Two independent clocks with drift rate $\rho$ can drift apart by at most $2\rho \Delta t$ over elapsed time $\Delta t$. To guarantee maximum skew $\delta$, clocks must synchronize at least every:
  $$\tau \le \frac{\delta}{2\rho}$$

---

### 2. CLASSICAL PHYSICAL CLOCK SYNCHRONIZATION ALGORITHMS

#### A. Cristian's Algorithm (Probabilistic Centralized Client-Server)
* Designed for systems with a centralized authoritative time server equipped with a UTC receiver.
* **Protocol Steps:**
  1. Client sends request at time $T_0$ according to its local clock.
  2. Server receives request and replies with current server time $T_{server}$.
  3. Client receives reply at local time $T_1$.
* **Formulas:**
  $$\text{Round-Trip Time } RTT = T_1 - T_0$$
  $$\text{Adjusted Client Time } C_{new} = T_{server} + \frac{RTT}{2}$$
* **Accuracy Bound:** If minimum one-way transit delay is $T_{min}$, accuracy is:
  $$\pm \left(\frac{RTT}{2} - T_{min}\right)$$

#### B. The Berkeley Algorithm (Active Master-Slave Averaging)
* Designed for internal clock synchronization without an external UTC reference.
* **Protocol Steps:**
  1. A master node periodically polls all slave machines for their local times.
  2. Slaves report their local clock readings.
  3. Master estimates round-trip latency, calculates clock readings, discards outliers (faulty clocks), and computes an average time across all healthy nodes.
  4. Master sends **clock adjustments** (relative delta $+/-$) back to each slave, gradually slewing clocks to prevent backward jumps.

---

### 3. THE NETWORK TIME PROTOCOL (NTP) IN DEPTH

NTP is the global Internet standard for clock synchronization, providing microsecond accuracy over LANs and millisecond accuracy over WANs.

#### A. Stratum Hierarchy Topology

```
+---------------------------------------------------------------------------------------------------------+
|                                       NTP STRATUM TIER HIERARCHY                                        |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       [ STRATUM 0 ]  Atomic Reference Clocks (Cesium), GPS, CDMA Transceivers                           |
|                            |                                                                            |
|                            v Direct interface (RS-232, PPS)                                             |
|       [ STRATUM 1 ]  Primary NTP Servers (Directly synchronized to Stratum 0)                           |
|                            |                                                                            |
|                            v Cross-network NTP protocol                                                 |
|       [ STRATUM 2 ]  Secondary NTP Servers (Regional enterprise master servers)                         |
|                            |                                                                            |
|                            v Intranet / LAN NTP protocol                                                |
|       [ STRATUM 3 ]  Client Workstations, Cloud Virtual Instances                                       |
+---------------------------------------------------------------------------------------------------------+
```

#### B. Four-Timestamp Packet Exchange and Mathematical Equations

```
+---------------------------------------------------------------------------------------------------------+
|                                   NTP 4-TIMESTAMP MESSAGE PROTOCOL                                      |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       Client A                                       Server B                                           |
|          |                                              |                                               |
|       T1 | (Transmit Request)                           |                                               |
|          |--------------------------------------------->|                                               |
|          |                                           T2 | (Receive Request)                             |
|          |                                              |                                               |
|          |                                           T3 | (Transmit Reply)                              |
|          |<---------------------------------------------|                                               |
|       T4 | (Receive Reply)                              |                                               |
+---------------------------------------------------------------------------------------------------------+
```

* **Round-Trip Propagation Delay ($\delta$):**
  $$\delta = (T_4 - T_1) - (T_3 - T_2)$$
* **Clock Offset of Client relative to Server ($\theta$):**
  Assuming transmission delays are symmetric in both directions:
  $$\theta = \frac{(T_2 - T_1) + (T_3 - T_4)}{2}$$
* **Slewing Mechanism:**
  If $\theta > 0$, client clock is slow; it speeds up clock rate. If $\theta < 0$, client clock is fast; it slows down clock rate. The clock is **never stepped backward**, preserving monotonicity for databases and logs.

---
---

# QUESTION 3
> ### **Discuss message ordering paradigms in distributed group communication in depth. Formulate FIFO, Causal Order, and Total Order, and explain algorithms for Causal Order and Total Order.**

---

### 1. GROUP COMMUNICATION AND DELIVERY SEMANTICS
In group communication, a message sent by one process is destined for a group of multiple processes (multicast/broadcast). Because messages travel over independent network links, different recipients can observe different message sequences unless ordered by protocol.

```
+---------------------------------------------------------------------------------------------------------+
|                                MESSAGE ORDERING PARADIGMS FORMULATION                                   |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|  1. FIFO ORDER:                                                                                         |
|     Messages sent by the SAME sender are delivered in transmission order.                               |
|     send_i(m1) < send_i(m2)  ===>  deliver_j(m1) < deliver_j(m2)                                        |
|                                                                                                         |
|  2. CAUSAL ORDER (CO):                                                                                  |
|     Messages causally related are delivered in causal order across ALL senders.                         |
|     send(m1) < send(m2)  ===>  deliver_k(m1) < deliver_k(m2)                                            |
|                                                                                                         |
|  3. TOTAL ORDER (TO):                                                                                   |
|     All destinations deliver messages in the IDENTICAL sequence.                                        |
|     deliver_i(m1) < deliver_i(m2)  <===>  deliver_j(m1) < deliver_j(m2)                                 |
|                                                                                                         |
|  4. TOTAL CAUSAL ORDER (Atomic Broadcast):                                                              |
|     Enforces BOTH Causal Order AND Total Order simultaneously!                                          |
+---------------------------------------------------------------------------------------------------------+
```

---

### 2. CAUSAL MESSAGE ORDERING ALGORITHMS

#### A. The Raynal-Schiper-Toueg (RST) Algorithm
* **Data Structures:** Each process $P_i$ maintains an $N \times N$ matrix $M_i$:
  * $M_i[j, k]$: Number of messages process $P_i$ knows that process $P_j$ has sent to process $P_k$.
* **Transmission Protocol:**
  * When $P_i$ multicasts $m$, it increments $M_i[i, j]$ for each destination $P_j$.
  * Piggybacks matrix $M_i$ onto message $m$.
* **Delivery Condition at Destination $P_j$ from Sender $P_k$:**
  * Message $(m, M_m)$ is buffered until:
    1. $M_m[k, j] = M_j[k, j] + 1$ (This is the next expected message from $P_k$).
    2. $\forall x \ne k: M_m[x, j] \le M_j[x, j]$ (All messages sent by other processes causally preceding $m$ have already been delivered).
  * Upon delivery, $P_j$ merges knowledge: $M_j = \max(M_j, M_m)$.

#### B. The Kshemkalyani-Singhal (KS) Algorithm
* **Optimization:** The RST algorithm has high network overhead because it attaches an $N \times N$ matrix to every message.
* The **KS Algorithm** prunes redundant causal history, transmitting only updated vector components since the last communication. This reduces message overhead from $O(N^2)$ to $O(N)$ or lower in common traffic patterns.

---

### 3. TOTAL MESSAGE ORDERING ALGORITHMS

#### A. Centralized Sequencer Approach
* A designated process acts as the **Sequencer**.
* Senders transmit messages to the sequencer.
* The sequencer assigns a global sequence number $S = 1, 2, 3, \dots$ and multicasts the message to the group.
* Receivers deliver messages strictly in increasing sequence number order.
* **Trade-off:** Fast and simple ($O(N)$ messages), but the sequencer is a single point of failure and bottleneck.

#### B. Distributed Consensus / 3-Phase Agreement (Skeen's Algorithm)
* Decentralized; requires no single sequencer.
* **Three Phases:**
  1. **Phase 1 (Request):** Sender multicasts message $m$.
  2. **Phase 2 (Proposed Timestamps):** Each destination buffers $m$ as *uncommitted*, assigns a proposed timestamp using its local clock, and replies to the sender.
  3. **Phase 3 (Commit Final Timestamp):** Sender collects proposals, computes $\text{Final\_TS} = \max(\text{Proposals})$, and broadcasts the commit message. Destinations mark $m$ as *committed* and deliver messages from the head of the priority queue.

---
---

# QUESTION 4
> ### **Explain in detail the Chandy-Lamport distributed snapshot algorithm for FIFO channels. Provide the complete system model, marker rules, correctness proof, and a comprehensive end-to-end worked example with channel message states.**

---

### 1. CONCEPTUAL NEED AND SYSTEM MODEL
In distributed systems, capturing a global snapshot is essential for checkpointing, failure recovery, deadlock detection, and termination detection.
The **Chandy-Lamport Algorithm** (1985) captures a consistent global state without freezing distributed execution.

#### System Assumptions:
1. The communication channels are **unidirectional, reliable, and strictly FIFO**.
2. The network graph is strongly connected.
3. Message delays are arbitrary but finite.
4. Any process can initiate the snapshot independently and concurrently.

---

### 2. FORMAL MARKER PROTOCOL RULES

A special control message called the **MARKER** divides application messages into those sent before the snapshot and those sent after.

```
+---------------------------------------------------------------------------------------------------------+
|                                       CHANDY-LAMPORT PROTOCOL RULES                                     |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|  MARKER SENDING RULE (Site Pi):                                                                         |
|  1. Pi records its own local process state.                                                             |
|  2. For every outgoing channel C_ik, Pi sends a MARKER message before sending any                       |
|     subsequent application messages along C_ik.                                                         |
|                                                                                                         |
|  MARKER RECEIVING RULE (Site Pj receiving MARKER along incoming channel C_ij):                          |
|                                                                                                         |
|  CASE A (Pj has NOT recorded its local state yet):                                                      |
|  - Pj records its local state immediately.                                                              |
|  - Pj records the state of incoming channel C_ij as EMPTY.                                              |
|  - Pj executes the Marker Sending Rule along ALL its outgoing channels.                                 |
|  - Pj begins recording incoming messages on all OTHER incoming channels C_kj (k != i).                  |
|                                                                                                         |
|  CASE B (Pj has ALREADY recorded its local state):                                                      |
|  - Pj STOPS recording incoming messages on channel C_ij.                                                |
|  - The state of channel C_ij is the sequence of all application messages received                       |
|    on C_ij between when Pj took its snapshot and when this MARKER arrived.                              |
+---------------------------------------------------------------------------------------------------------+
```

---

### 3. COMPREHENSIVE END-TO-END BANKING EXAMPLE

Consider two bank branches $P_1$ and $P_2$ connected by two unidirectional FIFO channels $C_{12}$ ($P_1 \to P_2$) and $C_{21}$ ($P_2 \to P_1$).
* **Initial State:**
  * Balance($P_1$) = \$1,000
  * Balance($P_2$) = \$500
  * Total System Balance = \$1,500

```
  Process P1 ($1000)                               Process P2 ($500)
       |                                                |
       |--- m1: Transfer $200 ------------------------->|
       |    (Balance becomes $800)                      |
       |                                                |
 [P1 INITIATES SNAPSHOT!]                               |
 - Records local state: P1 = $800                       |
 - Sends MARKER along C12                               |
       |                                                |
       |--- MARKER ------------------------------------>|
       |                                                |
       |                                     [P2 receives m1: $200]
       |                                     - Balance P2 becomes $700
       |                                                |
       |                                     [P2 receives MARKER on C12]
       |                                     - First marker for P2!
       |                                     - Records local state: P2 = $700
       |                                     - Records state(C12) = EMPTY
       |                                     - Sends MARKER along C21
       |                                                |
       |<-- MARKER -------------------------------------|
       |                                                |
 [P1 receives MARKER on C21]                            |
 - P1 already recorded state!                           |
 - Stops recording on C21:                              |
   State(C21) = EMPTY                                   |
 - SNAPSHOT COMPLETE!                                   |
```

#### Final Global State Recorded:
* $\text{State}(P_1) = \$800$
* $\text{State}(P_2) = \$700$
* $\text{State}(C_{12}) = \emptyset$
* $\text{State}(C_{21}) = \emptyset$
* **Total Accounted Money:** $\$800 + \$700 + \$0 + \$0 = \$1,500$.
* The snapshot is completely consistent!

---

### 4. MATHEMATICAL CORRECTNESS PROOF
* **Proof of Consistency:**
  Let $e_i$ be an event in $P_i$ and $e_j$ be an event in $P_j$ such that $e_i \prec e_j$. We must prove that if $e_j$ is in the snapshot, $e_i$ must also be in the snapshot.
  If $e_j$ is in the snapshot, $e_j$ occurred before $P_j$ received the marker. Because channels are strictly FIFO, any message sent during event $e_i$ must have preceded the marker along channel $C_{ij}$. Thus, $e_i$ must have occurred before $P_i$ sent the marker, ensuring $e_i$ is recorded in $P_i$'s local state. Hence, the cut is consistent.

---
