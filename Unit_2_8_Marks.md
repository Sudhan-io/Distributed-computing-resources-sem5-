# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT II: LOGICAL TIME AND GLOBAL STATE
### PART B: 8-MARK QUESTIONS AND ANSWERS (IN-DEPTH EXAM REPOSITORY)

---

## QUESTION 1: Explain Lamport's Scalar Logical Clock Framework, Clock Conditions, and Update Rules with an Example.

### 1. Conceptual Need for Logical Time
In distributed systems without shared memory or a global physical clock, determining which event occurred "before" another cannot rely on physical timestamps because physical quartz clocks drift unpredictably. Leslie Lamport (1978) proposed **Logical Clocks**, which map system events to monotonically increasing scalar integers that capture the causal structure of the execution.

---

### 2. The Clock Condition
A logical clock system maps an event $e$ to a scalar value $C(e)$.
* **Formal Clock Condition:** For any two events $e_i$ and $e_j$:
  $$e_i \prec e_j \implies C(e_i) < C(e_j)$$
* **Decomposition:**
  * **Condition C1 (Internal Monotonicity):** If $e_i$ and $e_j$ are successive events in process $P_k$ such that $e_i \prec_k e_j$, then $C_k(e_i) < C_k(e_j)$.
  * **Condition C2 (Message Causality):** If $e_i$ is the sending of message $m$ by $P_a$ and $e_j$ is the receipt of $m$ by $P_b$, then $C_a(e_i) < C_b(e_j)$.

---

### 3. Implementation Rules (Update Protocol)

```
+-----------------------------------------------------------------------------------------+
|                              LAMPORT'S SCALAR CLOCK RULES                               |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|  RULE 1 (Local Event Advancement):                                                      |
|  Before executing an internal event or a send event at process Pi:                      |
|                           C_i = C_i + d   (typically d = 1)                             |
|                                                                                         |
|  RULE 2 (Message Piggybacking & Reception):                                             |
|  - When Pi transmits message m, it attaches its current clock: (m, ts_m = C_i)         |
|  - When Pj receives message (m, ts_m), it advances its local clock:                     |
|                   C_j = max(C_j, ts_m) + d   (typically d = 1)                          |
+-----------------------------------------------------------------------------------------+
```

---

### 4. Illustrative Example with Space-Time Trace

Consider 3 processes $P_1, P_2, P_3$ starting with initial clocks $C_1 = 0, C_2 = 0, C_3 = 0$.

```
+-----------------------------------------------------------------------------------------+
|                             SCALAR LOGICAL CLOCKS TRACE                                 |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       P1  ---(1)--------(2)-------------------------(3)-------------------->            |
|                           \                           \                                 |
|                            \ m1 (ts=2)                 \ m2 (ts=3)                      |
|                             v                           v                               |
|       P2  ---(1)-----------(3)--------(4)---------------(5)---------------->            |
|                                          \                                              |
|                                           \ m3 (ts=4)                                   |
|                                            v                                            |
|       P3  ---(1)---------------------------(5)---------(6)----------------->            |
+-----------------------------------------------------------------------------------------+
```

* **Step-by-Step Evolution:**
  1. $P_1$ executes internal event $e_1^1 \to C_1 = 1$.
  2. $P_1$ sends $m_1$ at $e_1^2 \to C_1 = 2$. Transmits $(m_1, ts = 2)$.
  3. $P_2$ executes internal event $e_2^1 \to C_2 = 1$.
  4. $P_2$ receives $m_1 \to C_2 = \max(1, 2) + 1 = 3$.
  5. $P_2$ sends $m_3$ at $e_2^3 \to C_2 = 4$. Transmits $(m_3, ts = 4)$.
  6. $P_3$ receives $m_3 \to C_3 = \max(1, 4) + 1 = 5$.
* **Limitation:** $C(e_1^1) = 1$ and $C(e_2^1) = 1$, yet neither causally precedes the other ($e_1^1 \parallel e_2^1$). Scalar clocks **cannot detect concurrency**.

---

## QUESTION 2: Explain Vector Time, Its Structure, Update Rules, and Illustrate How It Captures Causal Independence with a Space-Time Diagram.

### 1. Fundamentals of Vector Time
Developed independently by Colin Fidge and Friedemann Mattern (1988), **Vector Clocks** overcome the fundamental limitation of scalar clocks by establishing an **exact equivalence (isomorphism)** between logical timestamps and causal precedence:
$$e_i \prec e_j \iff V(e_i) < V(e_j)$$

---

### 2. Structure and Update Protocol
Each process $P_i$ maintains an integer array $V_i[1..N]$, where $N$ is the number of processes:
* $V_i[i]$: Number of events executed locally by process $P_i$.
* $V_i[j]$ ($j \ne i$): Number of events executed by process $P_j$ that causally precede $P_i$'s current state.

```
+-----------------------------------------------------------------------------------------+
|                                VECTOR CLOCK UPDATE RULES                                |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|  RULE 1 (Local Event Progress):                                                         |
|  Before executing an internal or send event at process Pi:                              |
|                           V_i[i] = V_i[i] + 1                                           |
|                                                                                         |
|  RULE 2 (Message Transmission and Receipt):                                             |
|  - When Pi transmits message m, it piggybacks vector V_i: (m, V_m = V_i).               |
|  - When Pj receives message (m, V_m), it merges knowledge:                              |
|         For all k = 1 to N:   V_j[k] = max(V_j[k], V_m[k])                              |
|         V_j[j] = V_j[j] + 1                                                             |
+-----------------------------------------------------------------------------------------+
```

---

### 3. Comparison Relations on Vector Timestamps
Given two vector timestamps $u$ and $v$ of size $N$:
* $u \le v \iff \forall k \in [1..N]: u[k] \le v[k]$
* $u < v \iff (u \le v) \land (\exists k \in [1..N]: u[k] < v[k])$
* **Causal Concurrency ($u \parallel v$):**
  $$u \parallel v \iff \neg(u < v) \land \neg(v < u)$$

---

### 4. Space-Time Diagram Walkthrough

```
+-----------------------------------------------------------------------------------------+
|                                VECTOR CLOCK SPACE-TIME TRACE                            |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       P1  ---[1,0,0]-----[2,0,0]------------------------------------>                   |
|                            \                                                            |
|                             \ m1 (ts=[2,0,0])                                           |
|                              v                                                          |
|       P2  ---[0,1,0]-------[2,2,0]-------[2,3,0]-------------------->                   |
|                                            \                                            |
|                                             \ m2 (ts=[2,3,0])                           |
|                                              v                                          |
|       P3  -----------------[0,0,1]---------[2,3,2]------------------>                   |
+-----------------------------------------------------------------------------------------+
```

* **Detecting Causal Ordering:**
  * Event $e_1^2$ has vector $[2, 0, 0]$.
  * Event $e_2^3$ has vector $[2, 3, 0]$.
  * Comparing: $[2, 0, 0] \le [2, 3, 0]$ and $0 < 3 \implies [2, 0, 0] < [2, 3, 0]$.
  * Hence, $e_1^2 \prec e_2^3$ (Causal precedence verified mathematically!).
* **Detecting Concurrency:**
  * Event $e_1^1$ has vector $[1, 0, 0]$.
  * Event $e_2^1$ has vector $[0, 1, 0]$.
  * $1 > 0$ (first component) but $0 < 1$ (second component).
  * Neither vector is less than the other $\implies [1, 0, 0] \parallel [0, 1, 0]$.
  * **$e_1^1$ and $e_2^1$ are strictly concurrent!**

---

## QUESTION 3: Describe Physical Clock Synchronization and the Network Time Protocol (NTP) Architecture and Synchronization Formula.

### 1. The Clock Synchronization Challenge
Every computer maintains an internal hardware quartz oscillator. Due to environmental temperature shifts and crystal aging, physical clocks drift:
$$\text{Clock Drift Rate: } \rho = \left|\frac{dC(t)}{dt} - 1\right| \approx 10^{-5} \text{ to } 10^{-6} \text{ seconds/sec}$$
Without periodic synchronization, clocks drift apart by several seconds each week.

---

### 2. Network Time Protocol (NTP) Stratum Architecture
NTP (RFC 5905) organizes time servers across the global Internet into a hierarchical tiered structure called **Strata**:

```
+-----------------------------------------------------------------------------------------+
|                             NTP HIERARCHICAL STRATUM TOPOLOGY                           |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       [ STRATUM 0 ]  Atomic Clocks (Cesium), GPS Satellites, Radio Receivers            |
|                            |                                                            |
|                            v (Direct physical connection)                               |
|       [ STRATUM 1 ]  Primary Time Servers (Directly attached to Stratum 0)              |
|                            |                                                            |
|                            v (Network NTP requests)                                     |
|       [ STRATUM 2 ]  Secondary Time Servers (Enterprise NTP servers)                    |
|                            |                                                            |
|                            v (LAN NTP requests)                                         |
|       [ STRATUM 3 ]  End-User Client Machines / Workstations                            |
+-----------------------------------------------------------------------------------------+
```

---

### 3. NTP Four-Timestamp Message Exchange and Formulas

A client $A$ synchronizes with an upstream server $B$ by exchanging UDP packets containing four high-precision timestamps:

```
+-----------------------------------------------------------------------------------------+
|                           NTP TIMING MESSAGE PROTOCOL EXCHANGE                          |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       Client A                                       Server B                           |
|          |                                              |                               |
|       T1 | (Client sends request)                       |                               |
|          |--------------------------------------------->|                               |
|          |                                           T2 | (Server receives request)     |
|          |                                              |                               |
|          |                                           T3 | (Server transmits reply)      |
|          |<---------------------------------------------|                               |
|       T4 | (Client receives reply)                      |                               |
+-----------------------------------------------------------------------------------------+
```

#### Mathematical Formulas:
1. **Total Round-Trip Network Delay ($\delta$):**
   $$\delta = (T_4 - T_1) - (T_3 - T_2)$$
   *(Total transit time in both directions minus the server's internal processing turnaround).*
2. **Clock Offset of Client relative to Server ($\theta$):**
   Assuming symmetric network transit delays:
   $$\theta = \frac{(T_2 - T_1) + (T_3 - T_4)}{2}$$
3. **Clock Adjustment:**
   The client adjusts its local physical clock by adding offset $\theta$. To avoid backward clock jumps (which corrupt transactional file systems), the clock is **slewed** (gradually slowed down or sped up).

---

## QUESTION 4: Compare FIFO, Causal Order (CO), and Total Order (TO) Message Ordering Paradigms.

### 1. Delivery Ordering Taxonomy
In distributed systems, message delivery can be subjected to increasingly strict ordering guarantees:

```
+-----------------------------------------------------------------------------------------+
|                         MESSAGE ORDERING PARADIGMS COMPARISON                           |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|  1. FIFO ORDER:                                                                         |
|     Enforces order PER-SENDER CHANNEL. Messages from the same sender arrive in order.  |
|     send_i(m1) < send_i(m2) ==> deliver_j(m1) < deliver_j(m2)                          |
|                                                                                         |
|  2. CAUSAL ORDER (CO):                                                                  |
|     Enforces GLOBAL CAUSAL DEPENDENCY across all senders.                               |
|     send(m1) < send(m2) ==> deliver_k(m1) < deliver_k(m2)                               |
|                                                                                         |
|  3. TOTAL ORDER (TO):                                                                   |
|     Enforces UNANIMOUS IDENTICAL ORDER across all recipients.                           |
|     deliver_i(m1) < deliver_i(m2) <==> deliver_j(m1) < deliver_j(m2)                    |
+-----------------------------------------------------------------------------------------+
```

---

### 2. Comprehensive Comparative Matrix

| Evaluation Parameter | FIFO Order | Causal Order (CO) | Total Order (TO) |
| :--- | :--- | :--- | :--- |
| **Ordering Scope** | Single point-to-point channel | System-wide causal relationships | Global execution order across all nodes |
| **Cross-Process Coordination**| None required (sequence numbers suffice) | Vector timestamps or dependency lists | Consensus, central sequencer, or logical clocks |
| **Message Overhead** | Small scalar sequence number in header | Array/matrix of timestamps ($O(N)$ or $O(N^2)$) | Multi-phase voting or sequencer tokens |
| **Delivery Delay** | Immediate upon in-order arrival | Messages buffered until causal predecessors arrive | Messages buffered until global sequence finalized |
| **Typical Use Cases** | TCP streaming, simple file download | Collaborative editing, social media chat threads | Replicated state machines, stock exchange ledgers |

---

## QUESTION 5: Explain Asynchronous Execution with Synchronous Communication (AESC), RSC Executions, and the Crown Criterion.

### 1. Conceptual Framework
In an asynchronous execution, communication channels have non-zero, variable transmission delays (messages can be in transit). In a **synchronous communication model** (CSP, Ada rendezvous), communication is instantaneous: sender and receiver must synchronize simultaneously.
* **The Question:** *Can a distributed execution generated on an asynchronous network run without behavioral changes on a synchronous communication network?*
* An execution satisfying this property is called **RSC (Realizable with Synchronous Communication)**.

---

### 2. The Crown Criterion
R. Bagrodia and Charron-Bost proved that an asynchronous execution $(E, \prec)$ is RSC if and only if it contains **no crowns**.

* **Formal Definition of a Crown:**
  A crown of size $k$ ($k \ge 2$) is a cyclic sequence of $k$ distinct messages $\langle m_0, m_1, \dots, m_{k-1} \rangle$ such that:
  $$\text{send}(m_i) \prec \text{receive}(m_{(i+1) \bmod k}) \quad \forall i \in \{0, \dots, k-1\}$$

```
+-----------------------------------------------------------------------------------------+
|                                CROWN OF SIZE 2 (NON-RSC)                                |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       P1  --- send(m0) -----------------------------------> receive(m1) --->            |
|                   \                                            ^                        |
|                    \                                          /                         |
|                     v                                        /                          |
|       P2  --- receive(m0) <------------------------------ send(m1) -------->           |
|                                                                                         |
|       - send(m0) < receive(m1)  AND  send(m1) < receive(m0)                             |
|       - Requires both messages to be in transit simultaneously!                         |
|       - IMPOSSIBLE TO SCHEDULE WITH INSTANTANEOUS SYNCHRONOUS RENDEZVOUS!               |
+-----------------------------------------------------------------------------------------+
```

---

## QUESTION 6: Explain the Raynal-Schiper-Toueg (RST) Algorithm for Causal Message Ordering.

### 1. Objective and Operating Principle
The **RST Algorithm** (1991) enforces Causal Order (CO) delivery for broadcast messages over an asynchronous, reliable network.
* **Core Philosophy:** Every process maintains a local **matrix clock** $M_i[1..N, 1..N]$ representing its view of messages sent and received across all pairs of processes.

---

### 2. Data Structures and Algorithm Rules
* **Local Data Structure at $P_i$:**
  $M_i[j, k]$: Number of messages that process $P_i$ knows have been sent by process $P_j$ to process $P_k$.
* **Algorithm Steps:**
  1. **Sending Message $m$ to all peers:**
     * $P_i$ increments its send counter: $M_i[i, j] = M_i[i, j] + 1 \quad \forall j \ne i$.
     * $P_i$ attaches matrix $M_i$ to message $m$ and broadcasts it.
  2. **Delivering Message $(m, M_m)$ at receiver $P_j$ from sender $P_k$:**
     * $P_j$ buffers message $m$ until:
       $$M_m[k, j] = M_j[k, j] + 1 \quad (\text{Next in sequence from } P_k)$$
       $$\forall x \ne k: \quad M_m[x, j] \le M_j[x, j] \quad (\text{All causally preceding messages have been delivered!})$$
     * Upon delivery, $P_j$ updates:
       $$\forall x, y: \quad M_j[x, y] = \max(M_j[x, y], M_m[x, y])$$

---

## QUESTION 7: Explain Skeen's 3-Phase Algorithm for Total Message Ordering.

### 1. Conceptual Foundation
Skeen's algorithm achieves **Total Message Ordering (TO)** in a distributed system without a central coordinator by assigning mutually agreed global timestamps to messages.

---

### 2. The Three Phases of Skeen's Algorithm

```
+-----------------------------------------------------------------------------------------+
|                              SKEEN'S 3-PHASE PROTOCOL WORKFLOW                          |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|  PHASE 1: PROPOSAL REQUEST                                                              |
|  - Sender P_s broadcasts message m to all destination processes.                        |
|                                                                                         |
|  PHASE 2: TIMESTAMP PROPOSALS                                                           |
|  - Each destination P_i marks m as UNCOMMITTED in its priority queue.                   |
|  - P_i assigns a proposed timestamp: proposed_ts = max(last_ts, local_clock) + 1        |
|  - P_i sends proposed_ts back to P_s.                                                   |
|                                                                                         |
|  PHASE 3: FINAL COMMIT                                                                  |
|  - P_s collects proposed timestamps from all destinations.                              |
|  - Computes final timestamp: final_ts = max(all proposed_ts).                           |
|  - Broadcasts COMMIT(m, final_ts) to all destinations.                                  |
|  - Each destination updates m's timestamp to final_ts, marks it as COMMITTED,          |
|    and re-sorts its queue.                                                              |
|  - Messages at the head of the queue marked COMMITTED are delivered to the app!         |
+-----------------------------------------------------------------------------------------+
```

---

## QUESTION 8: Explain the Chandy-Lamport Snapshot Recording Algorithm for FIFO Channels.

### 1. System Assumptions and Definitions
* System consists of $N$ processes communicating over **unidirectional, reliable, FIFO channels**.
* Channels have arbitrary but finite transmission delays.
* The goal is to record a consistent global state without freezing application execution.

---

### 2. Chandy-Lamport Marker Rules Protocol

```
+-----------------------------------------------------------------------------------------+
|                             CHANDY-LAMPORT PROTOCOL RULES                               |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|  MARKER SENDING RULE FOR INITIATOR / PROCESS Pi:                                        |
|  1. Pi records its own local state.                                                     |
|  2. For every outgoing channel C_ik, Pi sends a special control MARKER message          |
|     before sending any further application messages on that channel.                    |
|                                                                                         |
|  MARKER RECEIVING RULE FOR PROCESS Pj ON CHANNEL C_ij:                                  |
|                                                                                         |
|  CASE 1 (First Marker Received by Pj):                                                  |
|  - Pj has NOT yet recorded its local state.                                             |
|  - Pj records its own local state immediately.                                          |
|  - Pj records the state of incoming channel C_ij as EMPTY.                              |
|  - Pj applies the Marker Sending Rule along ALL its outgoing channels.                  |
|  - Pj starts recording all incoming messages on all OTHER incoming channels C_kj (k!=i).|
|                                                                                         |
|  CASE 2 (Subsequent Marker Received by Pj on Channel C_ij):                             |
|  - Pj has ALREADY recorded its local state.                                             |
|  - Pj STOPS recording messages on channel C_ij.                                         |
|  - The state of channel C_ij is all messages recorded since Pj took its snapshot.       |
+-----------------------------------------------------------------------------------------+
```

---
