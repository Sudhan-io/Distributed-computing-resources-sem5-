# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT III: DISTRIBUTED MUTUAL EXCLUSION AND DEADLOCK DETECTION
### PART C: 16-MARK ESSAY QUESTIONS (EXHAUSTIVE 5-6 PAGE EXAMINATION ANSWERS)

---

# QUESTION 1
> ### **Discuss in detail the requirements that mutual exclusion algorithms should satisfy and discuss what metric we use to measure the performance of mutual exclusion algorithms.**

---

### 1. COMPREHENSIVE INTRODUCTION & ARCHITECTURAL FOUNDATIONS
In modern distributed computing environments, multiple independent processor nodes are interconnected via an asynchronous, packet-switched communication network. Unlike centralized multiprocessor architectures (such as Symmetric Multiprocessing - SMP or Non-Uniform Memory Access - NUMA), distributed nodes:
1. **Share No Physical Memory:** There is no global address space across nodes. Operations like atomic test-and-set, semaphores, or memory bus locking cannot be executed across network boundaries.
2. **Lack a Synchronized Physical Clock:** Relativistic quartz crystal drifts make absolute physical time unreliable for ordering events without error bounds.
3. **Rely Purely on Message Passing:** All synchronization, resource negotiation, and state sharing must occur through message transmission over channels with variable, unpredictable communication latency.

When multiple autonomous processes distributed across nodes concurrently access a non-shareable physical or logical resource (such as a shared file, an account record in a distributed ledger, a communication port, or a hardware controller), they enter what is formally termed a **Critical Section (CS)**. 

If two or more processes execute inside the Critical Section simultaneously, race conditions, dirty writes, unrepeatable reads, and catastrophic state corruption occur. Hence, **Distributed Mutual Exclusion** is the fundamental concurrency control mechanism that guarantees that at most one process can execute inside the Critical Section at any physical instant.

```
+---------------------------------------------------------------------------------------------------------+
|                         DISTRIBUTED PROCESS LIFE-CYCLE IN MUTUAL EXCLUSION                              |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       +------------------------------------+                                                            |
|       |    NON-CRITICAL SECTION (NCS)      |<--------------------------------------------------+        |
|       |  Process executes private tasks,   |                                                   |        |
|       |  local computations, and I/O.      |                                                   |        |
|       +-----------------+------------------+                                                   |        |
|                         |                                                                      |        |
|                         | Process decides it needs shared resource                             |        |
|                         v                                                                      |        |
|       +------------------------------------+                                                   |        |
|       |           ENTRY PROTOCOL           |                                                   |        |
|       |  1. Broadcasts REQUEST message     |                                                   |        |
|       |  2. Collects permissions / waits   |                                                   |        |
|       |     for unique privilege Token     |                                                   |        |
|       |  3. Resolves resource contention   |                                                   |        |
|       +-----------------+------------------+                                                   |        |
|                         |                                                                      |        |
|                         | Entry condition satisfied (All replies received / Token acquired)    |        |
|                         v                                                                      |        |
|       +------------------------------------+                                                   |        |
|       |       CRITICAL SECTION (CS)        |                                                   |        |
|       |  Exclusive resource execution.     |                                                   |        |
|       |  *INVARIANT: Exactly ONE process*  |                                                   |        |
|       |  *executes here across the cluster*|                                                   |        |
|       +-----------------+------------------+                                                   |        |
|                         |                                                                      |        |
|                         | Execution completes; resource operations finalized                   |        |
|                         v                                                                      |        |
|       +------------------------------------+                                                   |        |
|       |           EXIT PROTOCOL            |                                                   |        |
|       |  1. Cleans up local state          |                                                   |        |
|       |  2. Sends RELEASE messages or      |                                                   |        |
|       |     transfers Token to next node   |---------------------------------------------------+        |
|       |  3. Unblocks deferred processes    |                                                            |
|       +------------------------------------+                                                            |
|                                                                                                         |
+---------------------------------------------------------------------------------------------------------+
```

---

### 2. FUNDAMENTAL REQUIREMENTS (CORRECTNESS CRITERIA)
To be formally verified as sound and viable for mission-critical deployments, any distributed mutual exclusion algorithm must strictly satisfy three functional correctness properties under all possible event schedules and channel delays:

```
+---------------------------------------------------------------------------------------------------------+
|                             FORMAL REQUIREMENTS FOR MUTUAL EXCLUSION                                    |
+----------------------------------+----------------------------------------------------------------------+
| REQUIREMENT                      | FORMAL MATHEMATICAL SPECIFICATION & SYSTEM IMPACT                    |
+----------------------------------+----------------------------------------------------------------------+
| 1. SAFETY PROPERTY               | At most one process can execute inside the Critical Section at any   |
|    (Mutual Exclusion Guarantee)  | given instant in physical time:                                      |
|                                  |        forall t in Time,  SUM_{i=1}^N  I(P_i in CS at t) <= 1        |
|                                  | where I is an indicator function. Violations corrupt shared data.   |
|                                  |                                                                      |
| 2. LIVELINESS PROPERTY           | Freedom from Deadlock: If one or more processes attempt to enter the|
|    (Progress & Deadlock Freedom) | CS, at least one must be granted entry within finite time. The entire|
|                                  | system must never enter an indefinite cyclic wait state.             |
|                                  |                                                                      |
| 3. FAIRNESS PROPERTY             | Freedom from Starvation: Every individual process requesting the CS |
|    (Starvation Freedom & Order)  | must eventually enter the CS. Requests must be serviced in the causal|
|                                  | logical order of requests (FIFO by Lamport logical timestamps):     |
|                                  |                Req_i < Req_j  ===>  Entry_i < Entry_j                |
+----------------------------------+----------------------------------------------------------------------+
```

#### Detailed Examination of Each Property:

#### A. Safety Property (The Mutual Exclusion Invariant)
* **What it means:** Safety states that *nothing bad happens*. In mutual exclusion, the "bad thing" is concurrent access to the protected critical section.
* **Failure Consequences:** In a distributed banking system, if Site 1 and Site 2 simultaneously write to the same account balance without synchronization, one transaction will overwrite the other (the Lost Update anomaly), causing permanent accounting discrepancies.
* **Enforcement:** Enforced through physical token possession (Token-based approach) or by gathering explicit quorum permissions (Permission-based approach).

#### B. Liveness Property (Progress and Deadlock Freedom)
* **What it means:** Liveness states that *something good will eventually happen*.
* **Progress:** If no process is in the critical section, and one or more processes wish to enter, the decision of which process enters next cannot be postponed indefinitely.
* **Deadlock Freedom:** If processes $P_1, P_2, \dots, P_k$ concurrently issue requests, they must not enter a cyclic dependency where each waits for another to grant permission.

#### C. Fairness (Freedom from Starvation)
* **What it means:** Fairness guarantees that no single process is starved by an endless stream of higher-priority or newly arriving requests.
* **Logical Timestamp Ordering:** Algorithms ensure fairness by tagging each request with a scalar logical timestamp $(ts, i)$. Because logical timestamps advance monotonically, older requests always have smaller timestamps than newer requests, guaranteeing that an older request cannot be indefinitely overtaken.

---

### 3. QUANTITATIVE PERFORMANCE METRICS
To benchmark and compare different distributed mutual exclusion algorithms, the following five fundamental performance metrics are used:

```
+---------------------------------------------------------------------------------------------------------+
|                               PERFORMANCE METRICS IN MUTUAL EXCLUSION                                   |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       Site Pi                                                                  Site Pj                  |
|          |                                                                        |                     |
|          |-- Process Pi exits CS (Time t1)                                        |                     |
|          |   [Exit Protocol: Sends Release / Transfers Token]                     |                     |
|          |                                                                        |                     |
|          |================== SYNCHRONIZATION DELAY (S_D) =========================|                     |
|          |   (Time required for permission to reach next candidate Pj)            |                     |
|          |                                                                        v                     |
|          |                                                            Process Pj enters CS (Time t2)    |
|          |                                                            S_D = t2 - t1                     |
|          |                                                                        |                     |
|          |                                                            [Pj executes in CS for time E]    |
|          |                                                                        |                     |
|          |                                                            Process Pj exits CS               |
|                                                                                                         |
+---------------------------------------------------------------------------------------------------------+
```

#### 1. Message Complexity ($M$)
* **Definition:** The total number of network messages exchanged across all nodes in the system per single critical section execution (including request, permission, token, and release phases).
* **Two Operating Regimes:**
  * **Low Load Message Complexity:** Evaluated when critical section requests arrive infrequently, and the system is largely idle.
  * **High Load Message Complexity:** Evaluated when every node continuously has a pending request in its queue.
* **Goal:** Minimize $M$ to reduce network congestion and CPU interrupt overhead.

#### 2. Synchronization Delay ($S_D$)
* **Definition:** The time interval that elapses between the instant one process $P_i$ exits the critical section and the instant the next waiting process $P_j$ is permitted to enter the critical section:
  $$S_D = t_{\text{entry}}(P_j) - t_{\text{exit}}(P_i)$$
* **Units:** Typically expressed in terms of $T$, the average message propagation latency across the communication network.
* **Significance:** A smaller synchronization delay minimizes idle resource time between successive critical section executions.

#### 3. Response Time ($R$)
* **Definition:** The total elapsed time between when a process initiates a request to enter the critical section and when it completes execution and exits the critical section:
  $$R = \text{Wait Time in Entry Protocol} + \text{Execution Time inside CS}$$
* **Low Load Response Time:** Governed by network round-trip time ($2T$).
* **High Load Response Time:** Dictated by the number of pending requests waiting ahead in the queue: $O(N \times (E + S_D))$.

#### 4. System Throughput
* **Definition:** The maximum rate at which the distributed system can service critical section executions per unit time:
  $$\text{Throughput} = \frac{1}{S_D + E}$$
  where $E$ is the average execution time inside the Critical Section and $S_D$ is the synchronization delay.
* **Optimization:** When $E$ is fixed by application logic, system throughput is strictly bounded by the synchronization delay $S_D$. Minimizing $S_D$ directly maximizes cluster throughput.

#### 5. Fault Tolerance and Resilience
* Measures how the algorithm handles:
  * **Process Crashes:** Does a crash of a non-critical node halt the entire algorithm?
  * **Token Loss:** In token-based algorithms, does loss of the token require complex election protocols?
  * **Communication Link Failures:** Can the algorithm detect and recover from dropped or duplicated messages?

---

### 4. ARCHITECTURAL CLASSIFICATION OF MUTUAL EXCLUSION ALGORITHMS
Distributed mutual exclusion algorithms fall into two broad architectural paradigms:

```
+---------------------------------------------------------------------------------------------------------+
|                            CLASSIFICATION OF DISTRIBUTED MUTEX ALGORITHMS                               |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|                                  DISTRIBUTED MUTEX ALGORITHMS                                           |
|                                                |                                                        |
|                 +------------------------------+------------------------------+                         |
|                 |                                                             |                         |
|                 v                                                             v                         |
|       NON-TOKEN BASED (PERMISSION)                                      TOKEN BASED                     |
|       - Sites communicate to exchange permissions.                      - A single unique token         |
|       - Logical timestamps determine priority.                            circulates among sites.       |
|       - All nodes (or quorums) must respond.                            - Possession of token grants    |
|                 |                                                         unconditional CS access.      |
|        +--------+--------+                                                    |                         |
|        |                 |                                           +--------+--------+                |
|        v                 v                                           v                 v                |
|   LAMPORT (1978)   RICART-AGRAWALA (1981)                     SUZUKI-KASAMI (1985)  RAYMOND TREE (1989) |
|   - 3(N-1) msgs    - 2(N-1) msgs                              - 0 to N msgs         - O(log N) msgs     |
|   - Priority queues- Deferred replies                         - Broadcast requests  - Logical tree      |
|                                                               - RN, LN arrays       - Request queues    |
+---------------------------------------------------------------------------------------------------------+
```

---

### 5. EXHAUSTIVE COMPARATIVE PERFORMANCE MATRIX

| Performance Dimension | Centralized Coordinator | Lamport's Algorithm | Ricart-Agrawala Algorithm | Maekawa's Voting Algorithm | Suzuki-Kasami Algorithm | Raymond's Tree Algorithm |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Category** | Centralized | Non-Token (Permission) | Non-Token (Permission) | Quorum / Voting | Token Based (Broadcast) | Token Based (Tree) |
| **Messages (Low Load)** | $3$ ($Req, Grant, Rel$) | $3(N - 1)$ | $2(N - 1)$ | $3\sqrt{N} \text{ to } 5\sqrt{N}$ | $0 \text{ to } N$ | $O(\log N)$ |
| **Messages (High Load)** | $3$ | $3(N - 1)$ | $2(N - 1)$ | $O(\sqrt{N})$ | $N$ | $O(\log N)$ |
| **Synchronization Delay** | $2T$ | $T$ | $T$ | $2T$ | $T$ (or $0$) | $T \log N$ |
| **Local Data Structure** | Single queue at coordinator | Sorted `request_queue` of size $N$ | Array `reply_deferred[N]` | Quorum sets $R_i$ of size $\sqrt{N}$ | Vector $RN[N]$, Token $LN[N], Q$ | Local queue, pointer `holder` |
| **Single Point of Failure?**| **Yes** (Coordinator crash stops system) | **Yes** (Any site crash stalls all) | **Yes** (Any site crash stalls all) | **No** (Handles some site failures) | **Yes** (Token loss halts system) | **Yes** (Intermediate node crash halts tree) |
| **Deadlock Prone?** | No | No | No | **Yes** (Requires yield / abort msgs) | No | No |

---

### 6. EXAM SUMMARY: KEY POINTS TO HIGHLIGHT
1. **Define the Problem:** Shared resource, no shared memory, no physical clock, message passing only.
2. **State the 3 Core Requirements:** Safety (at most 1 in CS), Liveness (freedom from deadlock), Fairness (freedom from starvation).
3. **List the 5 Performance Metrics:** Message Complexity ($M$), Synchronization Delay ($S_D$), Response Time ($R$), System Throughput ($1/(S_D + E)$), Fault Tolerance.
4. **Draw the Life-Cycle Diagram:** Non-Critical $\to$ Entry Protocol $\to$ Critical Section $\to$ Exit Protocol.
5. **Draw the Comparison Table:** Contrast Lamport, Ricart-Agrawala, Suzuki-Kasami, and Maekawa.

---
---

# QUESTION 2
> ### **"External synchronization ensures internal synchronization. But the vice versa does not stand true." Justify. Explain Lamport's algorithm in detail with complete message exchanges and proof of correctness.**

---

### 1. PART 1: RIGOROUS JUSTIFICATION OF CLOCK SYNCHRONIZATION

#### A. Formal Definitions of Synchronization Paradigms
In distributed systems, physical clock synchronization algorithms are divided into two distinct classes based on their reference baseline:

```
+---------------------------------------------------------------------------------------------------------+
|                       EXTERNAL VS. INTERNAL PHYSICAL CLOCK SYNCHRONIZATION                              |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       EXTERNAL SYNCHRONIZATION                                 INTERNAL SYNCHRONIZATION                 |
|       (Bound to Real-World Reference S)                        (Bound Only to Peer Clocks)              |
|                                                                                                         |
|            [ UTC / GPS / Atomic Clock S(t) ]                        +------+         +------+           |
|                     /            \                                  | C1(t)|<=======>| C2(t)|           |
|                    /              \                                 +------+         +------+           |
|                   v                v                                    \               /               |
|               +------+          +------+                                 \             /                |
|               | C1(t)|          | C2(t)|                                  v           v                 |
|               +------+          +------+                                     +------+                   |
|                                                                              | C3(t)|                   |
|       |C_i(t) - S(t)| < D for all i                                          +------+                   |
|       Clocks track true solar/astronomical UTC.                |C_i(t) - C_j(t)| < delta for all i, j   |
|                                                                Clocks agree with each other, but        |
|                                                                can drift away from real-world UTC!      |
+---------------------------------------------------------------------------------------------------------+
```

1. **External Synchronization (with Accuracy Bound $D$):**
   Given an authoritative external real-time source $S(t)$ (such as Coordinated Universal Time - UTC supplied via GPS, WWVB radio signals, or Stratum-0 atomic clocks), a distributed system of $N$ processes with clocks $C_1, C_2, \dots, C_N$ is **externally synchronized** to within bound $D$ if and only if:
   $$\forall i \in \{1, \dots, N\}, \forall t: \quad |C_i(t) - S(t)| < D$$

2. **Internal Synchronization (with Precision Bound $\delta$):**
   A distributed system of $N$ processes is **internally synchronized** to within bound $\delta$ if and only if the difference between any two local clocks in the system is bounded:
   $$\forall i, j \in \{1, \dots, N\}, \forall t: \quad |C_i(t) - C_j(t)| < \delta$$

---

#### B. Mathematical Proof: External Synchronization Implies Internal Synchronization
* **Hypothesis:** Let the system be externally synchronized such that for every node $P_k$, $|C_k(t) - S(t)| < D$.
* **To Prove:** The system is internally synchronized within bound $\delta = 2D$.
* **Proof Step-by-Step:**
  Consider any two arbitrary nodes $P_i$ and $P_j$. The absolute difference between their local clock readings is:
  $$|C_i(t) - C_j(t)|$$
  Add and subtract the authoritative external time $S(t)$ inside the expression:
  $$|C_i(t) - C_j(t)| = |(C_i(t) - S(t)) - (C_j(t) - S(t))|$$
  Apply the triangle inequality ($|a - b| \le |a| + |b|$):
  $$|C_i(t) - C_j(t)| \le |C_i(t) - S(t)| + |C_j(t) - S(t)|$$
  Substitute the external synchronization bound $|C_k(t) - S(t)| < D$:
  $$|C_i(t) - C_j(t)| < D + D = 2D$$
* **Conclusion:** The system is guaranteed to be internally synchronized with a maximum skew bound of $\delta = 2D$. **Hence, External Synchronization strictly ensures Internal Synchronization.** $\blacksquare$

---

#### C. Mathematical Counter-Example: Internal Synchronization DOES NOT Imply External Synchronization
* **Hypothesis:** Assume the system has internal synchronization: $|C_i(t) - C_j(t)| < \delta$ for all $i, j$.
* **To Disprove:** That $|C_i(t) - S(t)| < D$ for some fixed bound $D$.
* **Counter-Example Construction:**
  Suppose all $N$ physical clocks run identically fast at double the standard rate, governed by:
  $$C_i(t) = 2t \quad \forall i \in \{1, \dots, N\}$$
  Calculate internal synchronization skew:
  $$|C_i(t) - C_j(t)| = |2t - 2t| = 0 < \delta$$
  *The system exhibits internal synchronization.*
  
  Now compute deviation from true external UTC $S(t) = t$:
  $$|C_i(t) - S(t)| = |2t - t| = t$$
  As physical time $t$ progresses:
  $$\lim_{t \to \infty} |C_i(t) - S(t)| = \lim_{t \to \infty} t = \infty$$
* **Conclusion:** The clocks drift arbitrarily far from external real time while remaining synchronized with each other. **Therefore, Internal Synchronization DOES NOT ensure External Synchronization.** $\blacksquare$

---

### 2. PART 2: LAMPORT'S DISTRIBUTED MUTUAL EXCLUSION ALGORITHM

#### A. System Assumptions & Data Structures
1. **System Model:**
   * A distributed system consisting of $N$ distinct sites $S_1, S_2, \dots, S_N$.
   * Sites communicate purely via message passing over point-to-point, error-free, **strictly FIFO channels**.
   * Transmission delays are finite, non-zero, and arbitrary.
2. **Local Data Structures at each Site $S_i$:**
   * `C_i`: Scalar integer logical clock updated using Lamport's clock conditions.
   * `request_queue_i`: A local priority queue storing pending critical section requests. Entries are pairs of the form $(ts_k, k)$, ordered by the total order relation $\prec$:
     $$(ts_i, i) \prec (ts_j, j) \iff (ts_i < ts_j) \lor (ts_i = ts_j \land i < j)$$
     *(Ties between identical timestamps are broken deterministically by process ID).*

---

#### B. The Complete Algorithmic Protocol

```
+---------------------------------------------------------------------------------------------------------+
|                               LAMPORT'S ALGORITHMIC PROTOCOL SPECIFICATION                              |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|  PHASE 1: REQUESTING THE CRITICAL SECTION (At Site Si)                                                  |
|  1. When site Si intends to enter the CS:                                                               |
|        C_i = C_i + 1                                                                                    |
|        Insert (C_i, i) into request_queue_i                                                             |
|        Broadcast REQUEST(C_i, i) to ALL other N - 1 sites.                                              |
|                                                                                                         |
|  PHASE 2: HANDLING AN INCOMING REQUEST (At Site Sj receiving REQUEST(ts_i, i))                          |
|  1. Upon receiving REQUEST(ts_i, i) from Si:                                                            |
|        C_j = max(C_j, ts_i) + 1                                                                         |
|        Insert (ts_i, i) into request_queue_j sorted by relation <                                       |
|        Send a timestamped REPLY(C_j, j) back to Si.                                                     |
|                                                                                                         |
|  PHASE 3: ENTERING THE CRITICAL SECTION (At Site Si)                                                    |
|  Site Si is permitted to enter the CS if and only if BOTH conditions hold:                              |
|     CONDITION A: Its own request (C_i, i) is at the HEAD of request_queue_i                             |
|                  (lowest timestamp among all pending requests).                                         |
|     CONDITION B: Si has received a message (REPLY, REQUEST, or RELEASE) with a                          |
|                  timestamp strictly greater than (C_i, i) from EVERY other N - 1 sites.                 |
|                                                                                                         |
|  PHASE 4: RELEASING THE CRITICAL SECTION (At Site Si)                                                   |
|  1. When Si completes execution inside the CS:                                                          |
|        Remove (C_i, i) from request_queue_i                                                             |
|        Broadcast a timestamped RELEASE(C_i, i) to ALL other N - 1 sites.                                |
|  2. When any site Sj receives RELEASE(C_i, i):                                                          |
|        Remove (C_i, i) from local request_queue_j.                                                      |
|        If Sj's own request is now at the head of request_queue_j and Condition B holds,                 |
|        Sj enters the Critical Section.                                                                  |
|                                                                                                         |
+---------------------------------------------------------------------------------------------------------+
```

---

#### C. Complete Trace Walkthrough with Space-Time Message Diagram
Consider a 3-site distributed cluster: $S_1, S_2, S_3$.
Initial state: $C_1 = 0, C_2 = 0, C_3 = 0$. All queues are empty.
**Concurrent Scenario:** Sites $S_1$ and $S_2$ request the CS concurrently, while Site $S_3$ is idle.

```
+---------------------------------------------------------------------------------------------------------+
|                                  LAMPORT'S 3-SITE MESSAGE FLOW TRACE                                    |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       Site S1 (Queue Q1)              Site S2 (Queue Q2)              Site S3 (Queue Q3)                |
|               |                               |                               |                         |
|   (t1) Req CS |                               |                               |                         |
|        C1 = 1 |                               |                               |                         |
|        Q1=[(1,1)]                             |                               |                         |
|        Sends REQ(1,1) to S2, S3               |                               |                         |
|               |------- REQ(1,1) ------------->|                               |                         |
|               |-------------------------------+------- REQ(1,1) ------------->|                         |
|               |                               |                               | S3 enqueues (1,1)       |
|               |                               |                               | C3 = max(0,1)+1 = 2     |
|               |                               |                               | Sends REPLY(2,3) to S1  |
|               |                               | (t2) Req CS                   |<-- REPLY(2,3) ----------|
|               |                               |      C2 = 1                   |                         |
|               |                               |      Q2=[(1,2)]               |                         |
|               |                               |      Sends REQ(1,2) to S1, S3 |                         |
|               |<------ REQ(1,2) --------------|------------------------------>|                         |
|               |                               |                               | S3 enqueues (1,2)       |
|               | S2 receives REQ(1,1):         |                               | Q3=[(1,1), (1,2)]       |
|               | C2 = max(1,1)+1 = 2           |                               | Sends REPLY(3,3) to S2  |
|               | Q2 = [(1,1), (1,2)]           |                               |                         |
|               | Sends REPLY(2,2) to S1        |                               |                         |
|               |<------ REPLY(2,2) ------------|                               |                         |
|               |                               |                               |                         |
|   (t3) S1 receives REQ(1,2):                  |                               |                         |
|        C1 = max(1,1)+1 = 2                    |                               |                         |
|        Q1 = [(1,1), (1,2)]                    |                               |                         |
|        Sends REPLY(2,1) to S2                 |                               |                         |
|               |------- REPLY(2,1) ----------->|                               |                         |
|               |                               |                               |                         |
|   (t4) EVALUATION AT SITE S1:                 |                               |                         |
|        - Head of Q1 is (1,1) because (1,1) < (1,2) [Tie-breaker: process ID 1 < 2]                     |
|        - Received REPLY(2,2) from S2 [Timestamp 2 > 1]                                                  |
|        - Received REPLY(2,3) from S3 [Timestamp 2 > 1]                                                  |
|        ===> S1 ENTERS CRITICAL SECTION!       |                               |                         |
|               |                               |                               |                         |
|   (t5) S1 completes CS execution:             |                               |                         |
|        Removes (1,1) from Q1.                 |                               |                         |
|        Broadcasts RELEASE(1,1) to S2, S3      |                               |                         |
|               |------- RELEASE(1,1) --------->|                               |                         |
|               |-------------------------------+------- RELEASE(1,1) --------->|                         |
|               |                               | S2 removes (1,1) from Q2.     | S3 removes (1,1) from Q3|
|               |                               | Q2 head is now (1,2)!         | Q3=[(1,2)]              |
|               |                               | S2 has replies with ts > 1    |                         |
|               |                               | from all sites.               |                         |
|               |                               | ===> S2 ENTERS CS!            |                         |
+---------------------------------------------------------------------------------------------------------+
```

---

#### D. Formal Proof of Correctness

#### 1. Proof of Safety (Mutual Exclusion Property)
* **Theorem:** At most one site can execute in the Critical Section at any time.
* **Proof by Contradiction:**
  Suppose two sites $S_i$ and $S_j$ ($i \ne j$) execute inside the Critical Section concurrently.
  Without loss of generality, assume that $S_i$'s request has a smaller timestamp than $S_j$'s request:
  $$(C_i, i) \prec (C_j, j)$$
  
  For $S_j$ to enter the critical section, it must satisfy **Condition A**: its request $(C_j, j)$ must be at the head of `request_queue_j`. This means $(C_i, i)$ cannot be ahead of $(C_j, j)$ in `request_queue_j`. This implies one of two possibilities:
  1. $S_j$ has not yet received $S_i$'s request $REQUEST(C_i, i)$.
  2. $(C_j, j) \prec (C_i, i)$, which directly contradicts our initial assumption.
  
  Now examine Possibility 1:
  For $S_j$ to enter the CS, it must also satisfy **Condition B**: $S_j$ must have received a message with a timestamp strictly greater than $(C_j, j)$ from site $S_i$.
  Let this message from $S_i$ be denoted as $M$. Thus:
  $$\text{Timestamp}(M) > C_j$$
  Since $(C_i, i) \prec (C_j, j)$, we have:
  $$C_i \le C_j < \text{Timestamp}(M)$$
  Therefore, $S_i$ generated its request $REQUEST(C_i, i)$ *before* it transmitted message $M$.
  Because communication channels are **strictly FIFO**, site $S_i$'s request $REQUEST(C_i, i)$ must arrive at site $S_j$ **before** message $M$ arrives at site $S_j$.
  
  Consequently, $S_j$ must have already received and enqueued $(C_i, i)$ into `request_queue_j` prior to receiving message $M$. Since $(C_i, i) \prec (C_j, j)$, $(C_i, i)$ must be ahead of $(C_j, j)$ in `request_queue_j`, preventing $(C_j, j)$ from being at the head.
  
  This creates a direct contradiction! Thus, two sites cannot hold the CS concurrently. **Safety is formally proven.** $\blacksquare$

#### 2. Proof of Liveness (Freedom from Deadlock & Starvation)
* **Deadlock Freedom:**
  Total ordering ($\prec$) ensures that all pending requests across all local queues are arranged in an identical, acyclic global linear sequence. The request with the minimal timestamp $(C_{min}, min)$ is guaranteed to reach the head of all queues, ensuring that at least one process enters the CS. Deadlock is impossible.
* **Starvation Freedom:**
  Any newly arriving request will be assigned a timestamp strictly greater than all currently pending requests. Because requests are serviced strictly in increasing timestamp order, every pending request eventually advances to the head of the queue and enters the CS. Starvation is impossible. $\blacksquare$

---

#### E. Performance Metrics of Lamport's Algorithm
* **Message Complexity:** Exactly **$3(N - 1)$ messages** per Critical Section execution:
  * $N - 1$ Request messages
  * $N - 1$ Reply messages
  * $N - 1$ Release messages
* **Synchronization Delay:** $T$ (the time required for a $RELEASE$ message to reach the next waiting site).
* **Fault Tolerance:** Extremely low. The crash of any single site halts the entire algorithm, because surviving sites block indefinitely waiting for $REPLY$ messages from the failed node.

---
---

# QUESTION 3
> ### **Explain in detail about Suzuki-Kasami's algorithm with its critical section execution with an example.**

---

### 1. CONCEPTUAL ARCHITECTURE & TOKEN PHILOSOPHY
The **Suzuki-Kasami Algorithm** (developed by Ichiro Suzuki and Tadao Kasami in 1985) is an efficient **broadcast token-based** distributed mutual exclusion algorithm.

#### Fundamental Paradigm Shift from Permission to Token Algorithms:
* In permission-based algorithms (such as Lamport and Ricart-Agrawala), a process must obtain explicit, individual authorization from all $N-1$ sites before every CS entry, incurring $2(N-1)$ to $3(N-1)$ messages every time.
* In the **Suzuki-Kasami Token Algorithm**, authority to enter the Critical Section is concentrated into a single, indivisible logical data object called the **Privilege Token**.
* **Key Invariant:** Whichever site holds the Token has unconditional authority to enter the CS without collecting permissions. If a site already holds the Token and no other site is requesting it, the site enters the CS with **0 messages**!

```
+---------------------------------------------------------------------------------------------------------+
|                              SUZUKI-KASAMI SYSTEM ARCHITECTURAL MODEL                                   |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       Site S1 [Token Holder]               Site S2                             Site S3                  |
|       +-------------------+               +-------------------+               +-------------------+     |
|       | RN1: [0, 0, 0]    |               | RN2: [0, 0, 0]    |               | RN3: [0, 0, 0]    |     |
|       +-------------------+               +-------------------+               +-------------------+     |
|       | PRIVILEGE TOKEN   |                                                                             |
|       | +---------------+ |                                                                             |
|       | |LN: [0, 0, 0]  | |                                                                             |
|       | |Q : []         | |                                                                             |
|       | +---------------+ |                                                                             |
|       +-------------------+                                                                             |
|                                                                                                         |
|       * RN_i[j] : Site Si's view of the largest request sequence number received from Site Sj.          |
|       * LN[j]   : In Token: Largest request sequence number of Site Sj successfully SERVICED so far.   |
|       * Q       : In Token: FIFO Queue of sites waiting to acquire the Token.                          |
+---------------------------------------------------------------------------------------------------------+
```

---

### 2. DETAILED DATA STRUCTURES & OUTSTANDING REQUEST CONDITION

#### A. Local Arrays at Each Site $S_i$:
Each site $S_i$ maintains an integer array of size $N$:
$$\text{RN}_i[1..N]$$
* $\text{RN}_i[j]$ denotes the largest sequence number of a request received by site $S_i$ from site $S_j$.
* When site $S_i$ generates a new request, it increments its own counter:
  $$\text{RN}_i[i] \leftarrow \text{RN}_i[i] + 1$$

#### B. Data Structures Encapsulated Inside the Privilege Token:
The Token carries two distinct structures across the network:
1. **$\text{LN}[1..N]$ (Last Number Vector):**
   * An integer array of size $N$, where $\text{LN}[j]$ records the sequence number of the most recent request of site $S_j$ that has been **successfully granted and completed**.
2. **$\text{Queue } Q$:**
   * A FIFO queue holding the identifiers of sites that have requested the token and are currently waiting for it.

#### C. The Condition for an Outstanding Request:
When a site $S_i$ completes execution in the CS and holds the token, how does it know which sites across the cluster have unfulfilled requests?
* Site $S_j$ has an outstanding, unserved request for the critical section if and only if:
  $$\text{RN}_i[j] = \text{LN}[j] + 1$$
* **Intuition:** $\text{RN}_i[j]$ indicates that $S_j$ has issued request number $k+1$, while $\text{LN}[j] = k$ indicates the token has only serviced up to request $k$. Therefore, request $k+1$ is pending!

---

### 3. COMPLETE STEP-BY-STEP ALGORITHMIC MECHANICS

```
+---------------------------------------------------------------------------------------------------------+
|                                SUZUKI-KASAMI PROTOCOL SPECIFICATION                                     |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|  PROCEDURE 1: REQUESTING THE CRITICAL SECTION (At Site Si)                                              |
|  ---------------------------------------------------------                                              |
|  If site Si already possesses the Token:                                                                |
|      Enters the Critical Section immediately! (Zero messages exchanged).                                |
|  Otherwise:                                                                                             |
|      RN_i[i] = RN_i[i] + 1                                                                              |
|      Broadcast REQUEST(i, RN_i[i]) to ALL other N - 1 sites in the system.                              |
|      Wait until the TOKEN message arrives.                                                              |
|      Upon arrival of TOKEN(LN, Q): Enter the Critical Section.                                          |
|                                                                                                         |
|  PROCEDURE 2: HANDLING AN INCOMING REQUEST (At Site Sj receiving REQUEST(i, sn))                        |
|  ------------------------------------------------------------------------------                         |
|  1. Sj updates its local record:                                                                        |
|         RN_j[i] = max(RN_j[i], sn)                                                                      |
|  2. If Sj holds the Token AND is NOT in the CS AND hasTokenFlag == TRUE:                                |
|         If RN_j[i] == LN[i] + 1:                                                                        |
|             hasTokenFlag = FALSE                                                                        |
|             Send TOKEN(LN, Q) directly to site Si.                                                      |
|                                                                                                         |
|  PROCEDURE 3: RELEASING THE CRITICAL SECTION (At Site Si)                                               |
|  --------------------------------------------------------                                               |
|  1. Update Token execution record:                                                                      |
|         LN[i] = RN_i[i]                                                                                 |
|  2. Check for newly arriving pending requests:                                                          |
|         For every site Sk (k != i) that is NOT already in Queue Q:                                      |
|             If RN_i[k] == LN[k] + 1:                                                                    |
|                 Enqueue Sk into Q.                                                                      |
|  3. Token Handoff Decision:                                                                             |
|         If Queue Q is NOT empty:                                                                        |
|             Pop the front site Sp from Q.                                                               |
|             hasTokenFlag = FALSE                                                                        |
|             Send TOKEN(LN, Q) directly to site Sp.                                                      |
|         Else:                                                                                           |
|             Keep the idle Token locally (hasTokenFlag = TRUE).                                          |
|                                                                                                         |
+---------------------------------------------------------------------------------------------------------+
```

---

### 4. COMPREHENSIVE WORKED TRACE EXAMPLE

Consider a distributed cluster of 3 sites: $S_1, S_2, S_3$.
* **Initial State:** Site $S_1$ holds the token initially.
  * Local vectors: $RN_1 = [0, 0, 0], RN_2 = [0, 0, 0], RN_3 = [0, 0, 0]$.
  * Token at $S_1$: $LN = [0, 0, 0]$, $Q = []$.
  * `hasToken` at $S_1 = \text{TRUE}$, at $S_2 = \text{FALSE}$, at $S_3 = \text{FALSE}$.

```
+---------------------------------------------------------------------------------------------------------+
|                               SUZUKI-KASAMI WORKED EXECUTION TRACE                                      |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|  TIME t1: S2 and S3 Concurrently Request Critical Section                                               |
|  --------------------------------------------------------                                               |
|  - Site S2 wants CS:                                                                                    |
|    RN_2[2] = 0 + 1 = 1                                                                                  |
|    Broadcasts REQUEST(2, 1) to S1 and S3.                                                               |
|  - Site S3 wants CS:                                                                                    |
|    RN_3[3] = 0 + 1 = 1                                                                                  |
|    Broadcasts REQUEST(3, 1) to S1 and S2.                                                               |
|                                                                                                         |
|  TIME t2: Requests Arrive at Site S1 (Token Holder)                                                     |
|  --------------------------------------------------                                                     |
|  - S1 receives REQUEST(2, 1): updates RN_1[2] = max(0, 1) = 1.                                          |
|  - S1 receives REQUEST(3, 1): updates RN_1[3] = max(0, 1) = 1.                                          |
|  Suppose S1 was executing in the CS when these arrived. S1 finishes its CS execution.                   |
|                                                                                                         |
|  TIME t3: Exit Protocol at Site S1                                                                      |
|  ---------------------------------                                                                      |
|  1. S1 updates LN[1] = RN_1[1] = 0.                                                                     |
|  2. S1 inspects pending requests across peers:                                                          |
|     - For S2: RN_1[2] (1) == LN[2] (0) + 1  ===> TRUE! Enqueues S2 into Q.   Q = [S2]                   |
|     - For S3: RN_1[3] (1) == LN[3] (0) + 1  ===> TRUE! Enqueues S3 into Q.   Q = [S2, S3]               |
|  3. S1 checks Q: Q is not empty.                                                                        |
|     - Pops front element: S2.  Remaining Q = [S3].                                                      |
|     - S1 transmits TOKEN(LN=[0,0,0], Q=[S3]) directly to S2.                                            |
|     - S1 sets hasToken = FALSE.                                                                         |
|                                                                                                         |
|  TIME t4: Site S2 Enters and Exits Critical Section                                                     |
|  --------------------------------------------------                                                     |
|  - S2 receives TOKEN(LN=[0,0,0], Q=[S3]).                                                               |
|  - S2 sets hasToken = TRUE and ENTERS CRITICAL SECTION!                                                 |
|  - S2 executes inside CS. During this time, S2 processes REQUEST(3, 1) from S3:                         |
|    RN_2[3] = max(0, 1) = 1.                                                                             |
|  - S2 finishes CS execution and runs Exit Protocol:                                                     |
|    1. Updates LN[2] = RN_2[2] = 1.                                                                      |
|    2. Inspects peers for unfulfilled requests:                                                          |
|       - For S1: RN_2[1] (0) == LN[1] (0) + 1 ===> FALSE.                                                |
|       - For S3: S3 is ALREADY inside Token Queue Q! (No duplication).                                   |
|    3. Token state: LN = [0, 1, 0], Q = [S3].                                                            |
|    4. Pops front element: S3. Remaining Q = [].                                                         |
|    5. Sends TOKEN(LN=[0,1,0], Q=[]) directly to S3.                                                     |
|    6. S2 sets hasToken = FALSE.                                                                         |
|                                                                                                         |
|  TIME t5: Site S3 Enters and Exits Critical Section                                                     |
|  --------------------------------------------------                                                     |
|  - S3 receives TOKEN(LN=[0,1,0], Q=[]).                                                                 |
|  - S3 sets hasToken = TRUE and ENTERS CRITICAL SECTION!                                                 |
|  - S3 completes CS execution:                                                                           |
|    1. Updates LN[3] = RN_3[3] = 1.                                                                      |
|    2. Checks peers: No outstanding requests (RN == LN).                                                 |
|    3. Queue Q is EMPTY.                                                                                 |
|    4. S3 retains the idle token locally!                                                                |
|       Token state at S3: LN = [0, 1, 1], Q = [].                                                        |
|                                                                                                         |
+---------------------------------------------------------------------------------------------------------+
```

---

### 5. CORRECTNESS PROOF & COMPLEXITY ANALYSIS

#### A. Proof of Safety (Mutual Exclusion Guarantee)
* **Theorem:** At most one site can execute in the Critical Section at any time.
* **Proof:**
  1. Entry into the CS strictly requires possession of the unique Privilege Token.
  2. At system initialization, exactly **one** token is created and assigned to a single site.
  3. A site holding the token relinquishes possession *before* transmitting it over the network (`hasToken = FALSE`).
  4. Transmission channels are reliable and error-free, ensuring the token is never duplicated in flight.
  5. Therefore, at any point in physical time $t$, at most one site can possess the token. Mutual exclusion holds unconditionally. $\blacksquare$

#### B. Proof of Liveness (Deadlock & Starvation Freedom)
* **Deadlock Freedom:**
  Token requests are broadcast to all nodes, guaranteeing that the current token holder receives the request within finite network delay. The token holder enqueues all requests into $Q$ and dispatches the token. An idle token is never withheld.
* **Starvation Freedom:**
  The token queue $Q$ operates under strict FIFO ordering. Any site $S_k$ that requests the token is placed into $Q$ and advances toward the front as preceding entries are serviced. Since execution inside the CS is finite, $S_k$ is guaranteed to receive the token within finite time. $\blacksquare$

#### C. Performance Metrics Summary
* **Message Complexity:**
  * **Best Case:** $0$ messages (if the requesting node already holds the idle token).
  * **Worst Case:** Exactly **$N$ messages** ($N - 1$ Request broadcasts $+ 1$ direct Token transfer).
  * *Comparison:* Ricart-Agrawala always requires $2(N-1)$ messages. Suzuki-Kasami cuts this roughly in half!
* **Synchronization Delay:** Only **$T$** (the time for the unicast token transfer from the exiting node to the next requesting node).
* **Vulnerability:** Token loss due to node crash destroys the token, requiring a distributed token regeneration election.

---
---

# QUESTION 4
> ### **How can we achieve deadlock detection in distributed systems? Provide various models to carry out the same.**

---

### 1. FOUNDATIONS OF DISTRIBUTED DEADLOCKS
In distributed database and operating systems, concurrent transactions/processes request shared locks on distributed data items. A **Distributed Deadlock** occurs when a set of processes are permanently blocked because each process holds one or more resources while waiting for another resource held by another process in the set, and these dependencies cross machine boundaries.

```
+---------------------------------------------------------------------------------------------------------+
|                             DISTRIBUTED WAIT-FOR DEPENDENCY GRAPH                                       |
+---------------------------------------------------------------------------------------------------------+
|                                                                                         |
|       SITE 1                                    SITE 2                                  |
|       +----------------------------+            +----------------------------+          |
|       | Process P1                 |            | Process P2                 |          |
|       | Holds Resource R1          |            | Holds Resource R2          |          |
|       | Waits for Resource R2 (on Site 2)       | Waits for Resource R1 (on Site 1)     |
|       +--------------+-------------+            +--------------+-------------+          |
|                      |                                         |                        |
|                      | Cross-Site Wait: (P1 -> P2)             | Cross-Site Wait: (P2 -> P1)
|                      +---------------------> <-----------------+                        |
|                                                                                         |
|       ===> CIRCULAR WAIT ACROSS NETWORK BOUNDARIES! DEADLOCK EXISTS!                    |
+---------------------------------------------------------------------------------------------------------+
```

---

### 2. RESOURCE REQUEST MODELS IN DISTRIBUTED SYSTEMS
The algorithmic complexity of detecting deadlocks depends directly on the expressive power of how processes request resources. There are four classical resource request models:

```
+---------------------------------------------------------------------------------------------------------+
|                              DISTRIBUTED DEADLOCK RESOURCE MODELS                                       |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|  1. SINGLE-RESOURCE MODEL                                                                               |
|     - A process can have at most ONE pending resource request at any time.                              |
|     - Out-degree of any vertex in the Wait-For Graph (WFG) is at most 1.                                |
|     - NECESSARY & SUFFICIENT CONDITION FOR DEADLOCK: Existence of a DIRECTED CYCLE in WFG.              |
|                                                                                                         |
|  2. THE AND MODEL                                                                                       |
|     - A process can request MULTIPLE resources simultaneously.                                          |
|     - Process remains blocked until ALL requested resources are granted.                                |
|     - Out-degree of vertices in WFG can be > 1.                                                         |
|     - NECESSARY & SUFFICIENT CONDITION FOR DEADLOCK: Existence of a DIRECTED CYCLE in WFG.              |
|                                                                                                         |
|  3. THE OR MODEL                                                                                        |
|     - A process can request MULTIPLE resources simultaneously.                                          |
|     - Process unblocks if ANY ONE of the requested resources is granted.                                |
|     - Out-degree > 1.                                                                                   |
|     - A directed cycle DOES NOT imply deadlock!                                                         |
|     - NECESSARY & SUFFICIENT CONDITION FOR DEADLOCK: Existence of a KNOT in WFG.                        |
|                                                                                                         |
|  4. THE C-OUT-OF-K MODEL (Generalized Model)                                                            |
|     - A process requests K resources and specifies that it requires ANY C resources (C <= K) to proceed.|
|     - Subsumes both AND (K-out-of-K) and OR (1-out-of-K) models.                                        |
|     - Detection requires complex WFG reduction algorithms or diffusing computations.                    |
+---------------------------------------------------------------------------------------------------------+
```

#### Detailed Contrast: AND Model vs. OR Model:

```
AND MODEL: Cycle = Deadlock               OR MODEL: Cycle != Deadlock (Requires Knot)
   [ P1 ] ------> [ P2 ]                      [ P1 ] ------> [ P2 ] ------> [ P4 (Active) ]
     ^              |                           ^              |
     |              v                           |              v
     +----------- [ P3 ]                        +----------- [ P3 ]
(P1 waits for P2 AND P3; Deadlocked!)     (P2 can be satisfied by ACTIVE process P4!
                                           P2 unblocks, breaking the cycle! NOT deadlocked!)
```

* **Definition of a Knot ($K$):** A directed subgraph $K$ such that for every node $u \in K$, all reachable nodes from $u$ are also in $K$, and every node in $K$ can reach all other nodes in $K$. In the OR model, a deadlock exists if and only if a blocked process belongs to a knot.

---

### 3. CONTROL ARCHITECTURES FOR DEADLOCK DETECTION

```
+---------------------------------------------------------------------------------------------------------+
|                         DISTRIBUTED DEADLOCK DETECTION ARCHITECTURES                                    |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|  1. CENTRALIZED ARCHITECTURE                                                                            |
|     - A single dedicated site (Central Deadlock Coordinator) maintains the global WFG.                  |
|     - Sites transmit local WFG edge insertions and deletions to the coordinator.                        |
|     - Coordinator runs standard cycle-detection algorithms (Tarjan's strongly connected components).    |
|     - Drawbacks: Single point of failure, bottleneck, and highly susceptible to PHANTOM DEADLOCKS.     |
|                                                                                                         |
|  2. HIERARCHICAL ARCHITECTURE (e.g., Menasce-Muntz Algorithm)                                           |
|     - Sites are organized into a multi-tier tree hierarchy.                                             |
|     - Leaf controllers detect intra-site deadlocks.                                                     |
|     - Non-leaf parent controllers detect cross-site deadlocks among their subordinate children.         |
|     - Eliminates the single central bottleneck, but complex to reconfigure upon node failures.          |
|                                                                                                         |
|  3. FULLY DISTRIBUTED / EDGE-CHASING ARCHITECTURE (e.g., Chandy-Misra-Haas)                             |
|     - Completely decentralized; all sites participate symmetrically.                                    |
|     - No site maintains the global WFG.                                                                 |
|     - Special diagnostic probe messages propagate along the edges of the distributed WFG.               |
|     - Highly robust, message-efficient, and industry-standard.                                          |
+---------------------------------------------------------------------------------------------------------+
```

---

### 4. IN-DEPTH EDGE-CHASING ALGORITHM: CHANDY-MISRA-HAAS (CMH)

#### A. CMH Algorithm for the AND Model
In the AND model, an edge-chasing probe message traverses the directed edges of the Wait-For Graph.

* **Probe Format:**
  $$\text{probe}(i, j, k)$$
  * $i$: Initiator process that started deadlock detection.
  * $j$: Sender process transmitting the probe.
  * $k$: Receiver process receiving the probe.

```
+---------------------------------------------------------------------------------------------------------+
|                            CMH PROBE PROPAGATION LOGIC (AND MODEL)                                      |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|  STEP 1: INITIATION (At Process Pi)                                                                     |
|  When process Pi blocks on resources held by a set of dependent processes:                              |
|      For each process Pk that Pi is waiting for:                                                        |
|          Send probe(i, i, k) along the channel to Pk.                                                   |
|                                                                                                         |
|  STEP 2: PROPAGATION (At Process Pk receiving probe(i, j, k))                                           |
|  When process Pk receives probe(i, j, k):                                                                |
|      If Pk is locally blocked AND has NOT yet forwarded a probe for initiator Pi:                       |
|          dependent(k) = TRUE                                                                            |
|          For each process Pm that Pk is currently waiting for:                                          |
|              Send probe(i, k, m) along the channel to Pm.                                               |
|                                                                                                         |
|  STEP 3: DEADLOCK DETECTION                                                                             |
|  If initiator process Pi receives a probe carrying its OWN identifier:                                  |
|                                   probe(i, *, i)                                                        |
|  ===> The probe has completed a full directed cycle back to its initiator!                              |
|  ===> PROCESS Pi IS DEADLOCKED! Triggers victim selection and recovery!                                 |
|                                                                                                         |
+---------------------------------------------------------------------------------------------------------+
```

#### B. Worked Example of CMH Probe Walkthrough

```
+---------------------------------------------------------------------------------------------------------+
|                                  CMH 3-SITE PROBE PROPAGATION TRACE                                     |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|         SITE 1                           SITE 2                           SITE 3                        |
|       +--------+                       +--------+                       +--------+                      |
|       |   P1   |---- waits for ------->|   P2   |---- waits for ------->|   P3   |                      |
|       +--------+                       +--------+                       +--------+                      |
|            ^                                                                |                           |
|            +--------------------- waits for (cross-site) -------------------+                           |
|                                                                                                         |
|  1. Process P1 blocks waiting for P2.                                                                   |
|     P1 initiates detection: sends probe(1, 1, 2) to Site 2.                                             |
|                                                                                                         |
|  2. Site 2 receives probe(1, 1, 2). P2 is blocked waiting for P3.                                       |
|     P2 forwards probe: sends probe(1, 2, 3) to Site 3.                                                  |
|                                                                                                         |
|  3. Site 3 receives probe(1, 2, 3). P3 is blocked waiting for P1.                                       |
|     P3 forwards probe: sends probe(1, 3, 1) to Site 1.                                                  |
|                                                                                                         |
|  4. Site 1 receives probe(1, 3, 1).                                                                     |
|     Receiving process (P1) matches initiator ID (1)!                                                    |
|     ===> DIRECTED CYCLE DETECTED: P1 -> P2 -> P3 -> P1!                                                 |
|     ===> DEADLOCK CONFIRMED! P1 triggers recovery.                                                      |
+---------------------------------------------------------------------------------------------------------+
```

---

### 5. RESOLUTION AND RECOVERY FROM DEADLOCKS
Once a deadlock cycle is detected, the system must recover by breaking the cycle through **Victim Selection**:
1. **Victim Selection Criteria:**
   * **Compute Cost:** Select the transaction that has consumed the least CPU time.
   * **Resource Footprint:** Select the transaction holding the fewest resource locks.
   * **Restart Penalty:** Favor transactions that can be rolled back to an intermediate savepoint rather than complete abortion.
2. **Rollback Mechanics:**
   * **Total Rollback:** Completely abort the victim transaction, release all locks, undo modified database buffers using write-ahead logs, and restart the transaction after a randomized backoff delay.
   * **Partial Rollback:** Roll back the victim only to the savepoint immediately preceding the conflicting lock acquisition.
3. **Starvation Prevention:**
   * Each time a transaction is chosen as a victim, its age or priority counter is incremented, ensuring it is not repeatedly aborted in favor of older transactions.

---
