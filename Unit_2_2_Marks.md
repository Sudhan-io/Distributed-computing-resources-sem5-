# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT II: LOGICAL TIME AND GLOBAL STATE
### PART A: 2-MARK QUESTIONS AND ANSWERS (EXAM-READY REPOSITORY)

---

### Q1. What is logical time, and why is physical time insufficient in distributed systems?
**Answer:**
* **Logical Time:** An abstraction that captures the causal order of events ($\prec$) based on message passing and local event progression rather than physical oscillator ticks.
* **Why Physical Time is Insufficient:**
  1. Independent crystal quartz oscillators drift at variable rates due to temperature and aging (clock drift and skew).
  2. Unpredictable network transmission latencies prevent instantaneous physical synchronization, making physical timestamps inadequate for determining true cause-and-effect relationships.

---

### Q2. Explain the Clock Condition in Lamport's framework for logical clocks.
**Answer:**
* A clock system satisfies the **Clock Condition** if, for any two events $e_i$ and $e_j$:
  $$e_i \prec e_j \implies C(e_i) < C(e_j)$$
* It decomposes into two structural conditions:
  * **C1 (Internal Monotonicity):** If $e_i$ and $e_j$ are successive events in process $P_k$, then $C_k(e_i) < C_k(e_j)$.
  * **C2 (Message Causality):** If $e_i$ is $\text{send}(m)$ and $e_j$ is $\text{receive}(m)$, then $C(e_i) < C(e_j)$.

---

### Q3. State the rules for updating scalar logical clocks (Lamport's Clocks).
**Answer:**
Each process $P_i$ maintains a scalar integer clock $C_i$:
1. **Rule 1 (Before executing an event):**
   $$C_i \leftarrow C_i + d \quad (d > 0, \text{ typically } d = 1)$$
2. **Rule 2 (Message transmission and receipt):**
   * Message $m$ sent by $P_i$ piggybacks timestamp $ts_m = C_i$.
   * Upon receiving message $(m, ts_m)$, process $P_j$ updates:
     $$C_j \leftarrow \max(C_j, ts_m) + d$$

---

### Q4. Define vector time and state its updating rules.
**Answer:**
* **Definition:** Vector time represents logical time as an array of $N$ integers $V_i[1..N]$ at process $P_i$, where $V_i[j]$ represents $P_i$'s knowledge of the latest event count at process $P_j$.
* **Update Rules:**
  1. **Local Event:** Before executing an internal or send event:
     $$V_i[i] \leftarrow V_i[i] + 1$$
  2. **Message Receipt:** Message $m$ sent by $P_i$ piggybacks vector $V_i$. Upon receipt of $(m, V_m)$, process $P_j$ updates:
     $$\forall k \in [1..N]: \quad V_j[k] \leftarrow \max(V_j[k], V_m[k])$$
     $$V_j[j] \leftarrow V_j[j] + 1$$

---

### Q5. Why cannot scalar clocks detect causal concurrency, whereas vector clocks can?
**Answer:**
* **Scalar Clock Limitation:** Scalar clocks guarantee only one-way implication:
  $$e_i \prec e_j \implies C(e_i) < C(e_j)$$
  However, $C(e_i) < C(e_j)$ **does not imply** $e_i \prec e_j$ (the events might be concurrent).
* **Vector Clock Advantage:** Vector clocks provide an **if-and-only-if (iff)** isomorphism to causality:
  $$e_i \prec e_j \iff V(e_i) < V(e_j)$$
  Hence, two events $e_i$ and $e_j$ are concurrent ($e_i \parallel e_j$) if and only if neither $V(e_i) \le V(e_j)$ nor $V(e_j) \le V(e_i)$ holds.

---

### Q6. What is clock drift and clock skew?
**Answer:**
* **Clock Skew:** The instantaneous difference in reading between two physical clocks at physical time $t$:
  $$\text{Skew} = |C_i(t) - C_j(t)|$$
* **Clock Drift:** The rate difference at which two physical clocks tick relative to a perfect reference clock:
  $$\text{Drift Rate } \rho = \frac{dC(t)}{dt} - 1$$

---

### Q7. Briefly explain the Network Time Protocol (NTP) hierarchy and synchronization mechanism.
**Answer:**
* **Stratum Hierarchy:**
  * Stratum 0: High-precision atomic clocks and GPS receivers.
  * Stratum 1: Primary servers directly attached to Stratum 0 devices.
  * Stratum 2: Secondary servers synchronizing with Stratum 1 servers over networks.
* **Offset & Delay Calculation:** Between client $A$ and server $B$ with timestamps $T_1, T_2, T_3, T_4$:
  $$\text{Round-Trip Delay } \delta = (T_4 - T_1) - (T_3 - T_2)$$
  $$\text{Clock Offset } \theta = \frac{(T_2 - T_1) + (T_3 - T_4)}{2}$$

---

### Q8. What are the message ordering paradigms in distributed systems?
**Answer:**
1. **FIFO Order:** Messages sent by a process along a specific channel are delivered to the recipient in the order sent.
2. **Causal Order (CO):** If the sending of message $m_1$ causally precedes the sending of message $m_2$ ($m_1 \prec m_2$), then $m_1$ is delivered before $m_2$ at all common destinations.
3. **Total Order (TO):** If messages $m_1$ and $m_2$ are delivered to any two processes, they are delivered in the identical relative order at both processes.

---

### Q9. Define Causal Order (CO) of messages.
**Answer:**
* For any two messages $m_1$ and $m_2$ that share a destination $P_k$:
  $$\text{send}(m_1) \prec \text{send}(m_2) \implies \text{deliver}_k(m_1) \prec \text{deliver}_k(m_2)$$
* Enforces causal relationship across all processes, even if messages originate from different senders.

---

### Q10. Define Total Order (TO) of messages.
**Answer:**
* For any two messages $m_1$ and $m_2$ delivered to both processes $P_i$ and $P_j$:
  $$\text{deliver}_i(m_1) \prec \text{deliver}_i(m_2) \iff \text{deliver}_j(m_1) \prec \text{deliver}_j(m_2)$$
* Guarantees all processes observe the exact same sequence of messages, preventing state divergence in replicated state machines.

---

### Q11. What is meant by Asynchronous Execution with Synchronous Communication (AESC) / RSC execution?
**Answer:**
* **AESC / RSC (Realizable with Synchronous Communication):** A property of an asynchronous execution $(E, \prec)$ indicating whether it could have been generated in an execution where all message communications were instantaneous (synchronous rendezvous).
* Formally, an execution is RSC if there exists a linear extension of the partial order $(E, \prec)$ such that every send event is immediately adjacent to its corresponding receive event without intervening events.

---

### Q12. What is the Crown condition/test in message ordering?
**Answer:**
* **Crown:** A cyclic configuration of $k$ messages ($m_0, m_1, \dots, m_{k-1}$) in an execution where:
  $$\text{send}(m_i) \prec \text{receive}(m_{(i+1) \bmod k})$$
* **Crown Criterion:** An execution is RSC (Realizable with Synchronous Communication) if and only if it contains **no crowns**.

---

### Q13. What is the purpose of the Chandy-Lamport distributed snapshot algorithm?
**Answer:**
* To record a **consistent global state** (a snapshot) of an asynchronous distributed system—including local process states and in-flight channel contents—without freezing normal application processing.
* Used for distributed checkpointing, deadlock detection, termination detection, and stable property verification.

---
