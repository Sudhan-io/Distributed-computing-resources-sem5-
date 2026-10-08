# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT III: DISTRIBUTED MUTUAL EXCLUSION AND DEADLOCK DETECTION
### PART B: 8-MARK QUESTIONS AND ANSWERS (SET 2 - ALTERNATE QUESTION BANK)

---

## QUESTION 1: Explain Maekawa's Quorum-Based Algorithm with an Example and Quorum Requirements.

### 1. Conceptual Framework of Maekawa's Algorithm
Proposed by Mamoru Maekawa in 1985, this algorithm was the first **quorum-based (voting)** distributed mutual exclusion algorithm.
* **Core Innovation:** Unlike Lamport and Ricart-Agrawala, where a site must request permission from **all** $N-1$ sites, Maekawa recognized that a site only needs permission from a subset of sites called a **Request Set (Quorum)** $R_i$.
* **Message Reduction:** Reduces message complexity from $O(N)$ down to **$O(\sqrt{N})$**.

---

### 2. Quorum Construction Requirements
For the system to guarantee mutual exclusion and fairness, the collection of request sets $\{R_1, R_2, \dots, R_N\}$ must satisfy three conditions:
1. **Intersection Property (Pairwise Overlap):**
   $$\forall i, j \in \{1, \dots, N\}: \quad R_i \cap R_j \ne \emptyset$$
   Any two request sets share at least one common arbiter site. This common site acts as an exclusive gatekeeper to prevent concurrent entry.
2. **Self-Containment:**
   $$\forall i \in \{1, \dots, N\}: \quad S_i \in R_i$$
   Every site belongs to its own request set.
3. **Equal Size and Equal Responsibility:**
   $$|R_i| = K \approx \sqrt{N} \quad \text{and each site appears in exactly } K \text{ request sets}$$
   This balances communication and computation overhead uniformly across all nodes.

```
+-----------------------------------------------------------------------------------------+
|                               MAEKAWA QUORUM INTERSECTION                               |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|            Request Set R1 (for S1)              Request Set R2 (for S2)                 |
|            +-----------------------+          +-----------------------+                 |
|            | S1                    |          |                    S2 |                 |
|            |                       |          |                       |                 |
|            |              +--------+----------+--------+              |                 |
|            |              |     COMMON ARBITER         |              |                 |
|            |              |        Site Sk             |              |                 |
|            |              +--------+----------+--------+              |                 |
|            +-----------------------+          +-----------------------+                 |
|                                                                                         |
|       * Site Sk can grant only ONE vote at a time!                                      |
|       * Whichever site acquires Sk's vote first enters the Critical Section!            |
+-----------------------------------------------------------------------------------------+
```

---

### 3. Example of Quorum Assignment for $N = 3$ and $N = 7$
For $N = 3$, using finite projective planes, quorums of size $K = 2$ are:
* $R_1 = \{S_1, S_2\}$
* $R_2 = \{S_2, S_3\}$
* $R_3 = \{S_3, S_1\}$

Notice that:
* $R_1 \cap R_2 = \{S_2\}$
* $R_2 \cap R_3 = \{S_3\}$
* $R_3 \cap R_1 = \{S_1\}$
Every pair intersects in exactly one site.

---

### 4. Performance & Deadlock Vulnerability
* **Message Complexity:** $3\sqrt{N}$ to $5\sqrt{N}$ messages per CS entry.
* **Synchronization Delay:** $2T$.
* **Deadlock Risk:** Maekawa's algorithm is prone to deadlock if requests arrive concurrently. Deadlocks are avoided using `INQUIRE`, `YIELD`, and `FAILED` message extensions.

---

## QUESTION 2: Explain Raymond's Tree-Based Mutual Exclusion Algorithm with a Neat Diagram.

### 1. Architectural Foundations
Proposed by Kerry Raymond in 1989, this is an efficient **tree-based token** mutual exclusion algorithm.
* **Network Topology:** Sites are organized into a logical directed spanning tree.
* **Core Idea:** A site does not broadcast requests globally. Instead, requests and the token travel along edges of the spanning tree, reducing message complexity to **$O(\log N)$**.

---

### 2. Local Data Structures at Each Node
Each node $P_i$ maintains:
1. `holder`: A pointer pointing to the neighbor on the directed path towards the current token holder. (If $P_i$ holds the token, `holder = self`).
2. `request_q`: A FIFO queue holding the identifiers of neighboring nodes that have requested the token.

```
+-----------------------------------------------------------------------------------------+
|                                RAYMOND'S LOGICAL TREE TOPOLOGY                          |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|                                     [ Node 1 ] (Holder of Token)                        |
|                                    holder = self                                        |
|                                     /          \                                        |
|                       holder = 1   /            \  holder = 1                           |
|                                   v              v                                      |
|                             [ Node 2 ]        [ Node 3 ]                                |
|                              /      \                 \                                 |
|                 holder = 2  /        \ holder = 2      \ holder = 3                     |
|                            v          v                 v                               |
|                        [ Node 4 ]   [ Node 5 ]        [ Node 6 ]                        |
+-----------------------------------------------------------------------------------------+
```

---

### 3. Step-by-Step Message Protocol
1. **Requesting the Token (Node $P_i$):**
   * $P_i$ enqueues its own ID into `request_q`.
   * If `request_q` had only 1 element (meaning it has not already forwarded a request), it sends a `REQUEST` message to `holder`.
2. **Forwarding a Request (Intermediate Node $P_j$):**
   * Enqueues the requesting neighbor's ID into its local `request_q`.
   * If this is the only entry in `request_q`, forwards `REQUEST` towards its `holder`.
3. **Passing the Token (Current Token Holder):**
   * Pops the front neighbor from `request_q`.
   * Sets `holder` to point to that neighbor.
   * Transmits the `TOKEN` to that neighbor.
4. **Token Arrival at Requesting Node:**
   * When the token reaches the initiator, it executes in the CS. Upon exit, if `request_q` is non-empty, it forwards the token to the next waiting neighbor.

---

### 4. Complexity Analysis
* **Average Message Complexity:** $O(\log N)$ messages on a balanced tree.
* **Synchronization Delay:** $T \log N$.

---

## QUESTION 3: Contrast Wait-Die and Wound-Wait Deadlock Prevention Schemes with Examples.

### 1. Conceptual Framework
Both **Wait-Die** and **Wound-Wait** (Rosenkrantz et al., 1978) are priority-based deadlock prevention protocols that use transaction creation timestamps $TS(T)$ to enforce an acyclic Wait-For Graph.
* Older transactions have smaller timestamps (higher priority).
* Younger transactions have larger timestamps (lower priority).

---

### 2. Comparative Analysis

```
+-----------------------------------------------------------------------------------------+
|                           WAIT-DIE VS. WOUND-WAIT LOGIC                                 |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       WAIT-DIE (Non-Preemptive Scheme)                                                  |
|       - Older requests Younger: Older is allowed to WAIT.                               |
|       - Younger requests Older: Younger DIES (aborts and restarts with same timestamp). |
|                                                                                         |
|       WOUND-WAIT (Preemptive Scheme)                                                    |
|       - Older requests Younger: Older WOUNDS (preempts/aborts) the younger transaction. |
|       - Younger requests Older: Younger is allowed to WAIT.                             |
+-----------------------------------------------------------------------------------------+
```

| Evaluation Dimension | Wait-Die Scheme | Wound-Wait Scheme |
| :--- | :--- | :--- |
| **Preemption Style** | **Non-preemptive:** A holding transaction is never preempted. | **Preemptive:** An older transaction preempts a younger one. |
| **Rule: Old requests Young** | $T_{old}$ **waits** for $T_{young}$ to release the resource. | $T_{old}$ **wounds** $T_{young}$ ($T_{young}$ aborts and yields lock). |
| **Rule: Young requests Old** | $T_{young}$ **dies** (aborted and restarted). | $T_{young}$ **waits** for $T_{old}$ to finish. |
| **Rollback Frequency** | **Higher:** Younger transactions may be aborted multiple times. | **Lower:** Older transactions proceed with minimal rollbacks. |
| **Starvation Freedom** | Guaranteed (restarted transactions keep original timestamps). | Guaranteed. |

---

### 3. Worked Example
Suppose $T_{10}$ (older, $TS = 10$) and $T_{20}$ (younger, $TS = 20$) compete for a resource.
* **In Wait-Die:**
  * If $T_{10}$ requests a lock held by $T_{20}$: $T_{10}$ waits.
  * If $T_{20}$ requests a lock held by $T_{10}$: $T_{20}$ dies (aborts and restarts).
* **In Wound-Wait:**
  * If $T_{10}$ requests a lock held by $T_{20}$: $T_{10}$ wounds $T_{20}$ ($T_{20}$ is rolled back immediately).
  * If $T_{20}$ requests a lock held by $T_{10}$: $T_{20}$ is permitted to wait.

---

## QUESTION 4: Explain Ho-Ramamoorthy Two-Phase Deadlock Detection Algorithm.

### 1. Background & Centralized Architecture
The **Ho-Ramamoorthy Two-Phase Algorithm** is a centralized deadlock detection protocol that eliminates **phantom deadlocks** by verifying whether a detected cycle persists across two independent observation phases.

---

### 2. The Two-Phase Protocol
1. **Phase 1 (Status Collection):**
   * The central coordinator periodically requests all sites to report their local resource allocation and wait tables.
   * The coordinator constructs a global Wait-For Graph (WFG).
   * It searches for cycles. If **no cycle** is found, the system is deadlock-free.
2. **Phase 2 (Cycle Verification):**
   * If a cycle is detected in Phase 1, it might be a **phantom deadlock** caused by transit delays.
   * The coordinator asks the sites involved in the suspected cycle to re-transmit their transaction and resource status tables.
   * The coordinator constructs a verified sub-WFG using only edges that were present in **both** Phase 1 and Phase 2 reports.
   * If the cycle persists across both reports, a genuine deadlock is confirmed.

---

## QUESTION 5: Explain Mitchell-Merritt Edge-Chasing Deadlock Detection Algorithm.

### 1. Operating Concept
The **Mitchell-Merritt algorithm** (1984) is a distributed edge-chasing algorithm for the single-resource model that uses public and private labels to detect and break cycles.

---

### 2. Data Structures & Algorithm Rules
* Each node maintains:
  * A **private label**: A monotonically increasing unique integer.
  * A **public label**: An integer visible to neighboring nodes.
* **Algorithm Steps:**
  1. **Block Step:** When node $P$ blocks waiting for node $Q$, $P$ updates its public label to be greater than both its own private label and $Q$'s public label, then sends a probe along the edge $(P \to Q)$.
  2. **Transmit Step:** When a node receives a probe with a label greater than its own public label, it absorbs the larger label.
  3. **Detect Step:** If a node receives a probe carrying a public label **equal to its own public label**, the probe has traversed a complete cycle. Deadlock is detected.

---

## QUESTION 6: Discuss How Message Ordering Affects Deadlock Detection and Causes Phantom Deadlocks.

### 1. What Causes Phantom Deadlocks?
A **phantom deadlock** occurs when a detection algorithm observes a cycle that does not exist in real time, caused by out-of-order message arrivals across asynchronous network channels.

```
+-----------------------------------------------------------------------------------------+
|                              PHANTOM DEADLOCK MECHANISM                                 |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       Site 1                                    Central Coordinator                     |
|          |                                              |                               |
|          |-- 1. P1 releases lock on R1 ---------------->| (Message delayed in transit!) |
|          |                                              |                               |
|          |-- 2. P1 requests lock on R2 ---------------->|                               |
|          |                                              | Reads Request(P1, R2) FIRST!  |
|          |                                              | Observes cyclic dependency!   |
|          |                                              | ===> FALSE DEADLOCK DETECTED! |
|          |                                              |                               |
|          |<-- 3. Release(P1, R1) arrives LATER ---------|                               |
+-----------------------------------------------------------------------------------------+
```

### 2. Prevention Strategies
1. **Causal Message Ordering:** Using Lamport timestamps to enforce that release events are processed before subsequent request events.
2. **Two-Phase Verification:** Using algorithms like Ho-Ramamoorthy to verify that cycles persist over time before aborting transactions.

---

## QUESTION 7: Compare Token-Based vs. Non-Token-Based Mutual Exclusion Algorithms.

### 1. Comparative Analysis

| Feature | Non-Token (Permission-Based) | Token-Based |
| :--- | :--- | :--- |
| **Core Concept** | Nodes collect explicit permissions from peers. | A unique privilege token circulates among nodes. |
| **Typical Message Overhead**| $2(N-1)$ to $3(N-1)$ messages per CS entry | $0$ to $N$ messages (or $O(\log N)$ in trees) |
| **Best-Case Message Cost** | $2(N-1)$ messages | **$0$ messages** (if node already holds token) |
| **Deadlock Vulnerability** | Deadlock-free by design (total ordering). | Deadlock-free; only token holder enters CS. |
| **Failure Vulnerability** | Node crashes prevent collecting all replies. | Token loss halts system; requires election protocol. |
| **Representative Algorithms**| Lamport, Ricart-Agrawala, Maekawa | Suzuki-Kasami, Raymond's Tree |

---
