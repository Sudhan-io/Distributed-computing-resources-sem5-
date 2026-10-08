# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT III: DISTRIBUTED MUTUAL EXCLUSION AND DEADLOCK DETECTION
### PART C: 16-MARK COMPREHENSIVE STUDY GUIDE & ANSWERS (SET 2 - ALTERNATE QUESTION BANK)

---

### Q1. Deep Architectural and Mathematical Analysis of Maekawa's Voting/Quorum Algorithm: Finite Projective Plane Construction, Algorithmic Protocol, Deadlock Vulnerability, Auxiliary Message Resolution (Inquire, Yield, Failed), and Correctness Proof.

---

#### 1. Architectural Philosophy and Quorum Foundations
In 1985, Mamoru Maekawa proposed the first **voting-based (quorum-based)** distributed mutual exclusion algorithm. In classical permission-based algorithms (such as Lamport’s or Ricart-Agrawala’s), a requesting node must obtain explicit permissions from all other $N-1$ nodes, resulting in $O(N)$ message complexity.

Maekawa recognized that unanimous consent is unnecessary. It is sufficient for a site to secure permission from a carefully structured subset of nodes, termed a **Request Set (Quorum)** $R_i \subset \{S_1, S_2, \dots, S_N\}$.

```
+---------------------------------------------------------------------------------------------------+
|                                  MAEKAWA'S QUORUM VOTING TOPOLOGY                                 |
+---------------------------------------------------------------------------------------------------+
|                                                                                                   |
|       Request Set R1 = {S1, S2}                      Request Set R2 = {S2, S3}                    |
|       +-------------------------+                    +-------------------------+                  |
|       | Site S1                 |                    |                 Site S3 |                  |
|       |                         |                    |                         |                  |
|       |                +--------+--------------------+--------+                |                  |
|       |                |           COMMON ARBITER             |                |                  |
|       |                |              Site S2                 |                |                  |
|       |                +--------+--------------------+--------+                |                  |
|       |                         |                    |                         |                  |
|       +-------------------------+                    +-------------------------+                  |
|                                                                                                   |
|       * Fundamental Invariant: Any two request sets overlap at a common arbiter!                  |
|       * Arbiter S2 grants only ONE vote at a time!                                                |
|       * Mutual exclusion is mathematically guaranteed!                                            |
|                                                                                                   |
+---------------------------------------------------------------------------------------------------+
```

##### 1.1 The Three Formal Conditions for Request Sets
To ensure safety, symmetry, and load balancing, the collection of request sets $\{R_1, R_2, \dots, R_N\}$ must satisfy three structural axioms:
1. **The Non-Empty Intersection Property (Safety Condition):**
   $$\forall i, j \in \{1, 2, \dots, N\}, \quad R_i \cap R_j \ne \emptyset$$
   Every pair of request sets must share at least one common site. This common site acts as an arbiter between any two competing sites.
2. **Self-Membership Property:**
   $$\forall i \in \{1, 2, \dots, N\}, \quad S_i \in R_i$$
   Every site belongs to its own request set and votes for itself.
3. **Equal Size and Equal Responsibility (Fairness Condition):**
   * Every request set contains the same number of sites: $|R_i| = K$.
   * Every site $S_j$ is contained in the exact same number of request sets: $D = K$.
   * To minimize message overhead while preserving these conditions:
     $$K \approx \sqrt{N}$$

---

#### 2. Quorum Construction via Finite Projective Planes

The mathematical problem of constructing optimal request sets satisfying Maekawa's axioms is equivalent to constructing a **Finite Projective Plane** of order $q$ (where $q$ is a prime power, $q = p^k$).

##### 2.1 Projective Plane Properties
A finite projective plane of order $q$ possesses the following geometric properties:
* Number of points (sites): $N = q^2 + q + 1$.
* Number of lines (request sets): $N = q^2 + q + 1$.
* Each line contains $K = q + 1$ points.
* Each point lies on $D = q + 1$ lines.
* Any two lines intersect at **exactly one point**.
* Therefore, the optimal quorum size is:
  $$K = q + 1 \approx \sqrt{N}$$

##### 2.2 Worked Construction Example for $N = 7$ ($q = 2$)
For $q = 2$:
$$N = 2^2 + 2 + 1 = 7 \text{ sites} \quad \{S_1, S_2, S_3, S_4, S_5, S_6, S_7\}$$
$$K = 2 + 1 = 3 \text{ sites per quorum}$$

The 7 request sets (lines of the Fano Plane) are:
* $R_1 = \{S_1, S_2, S_3\}$
* $R_2 = \{S_1, S_4, S_5\}$
* $R_3 = \{S_1, S_6, S_7\}$
* $R_4 = \{S_2, S_4, S_6\}$
* $R_5 = \{S_2, S_5, S_7\}$
* $R_6 = \{S_3, S_4, S_7\}$
* $R_7 = \{S_3, S_5, S_6\}$

*Verification:*
* $|R_1 \cap R_2| = |\{S_1\}| = 1 \ne \emptyset$.
* $|R_1 \cap R_4| = |\{S_2\}| = 1 \ne \emptyset$.
* Every site appears in exactly 3 quorums. Perfect symmetry and load balance!

---

#### 3. Algorithmic Protocol Specification

##### 3.1 Local State Variables at Site $S_i$
* `voted`: Boolean flag. `voted == TRUE` if $S_i$ has currently granted its vote to some site; `FALSE` otherwise.
* `vote_holder`: Node ID of the site to which $S_i$ currently granted its vote.
* `request_queue`: Priority queue storing pending incoming requests ordered by Lamport timestamp $\langle TS, ID \rangle$.

##### 3.2 Formal Protocol Steps

```
SITE Si (REQUESTER)                                        ARBITER Sj (MEMBER OF R_i)
      |                                                                 |
      | ====================== REQUEST PHASE ========================== |
      |                                                                 |
      | --- REQUEST(TS_i, i) -----------------------------------------> |
      |                                                                 |
      |                                              [Check: voted == FALSE ?]
      |                                              [If FALSE: voted = TRUE; send GRANT]
      |                                              [If TRUE: queue request; check priority]
      |                                                                 |
      | <--- GRANT ---------------------------------------------------- |
      |                                                                 |
      | [Collect K GRANTs from all members of R_i]                      |
      |                                                                 |
      | ===================== CRITICAL SECTION ======================== |
      |                                                                 |
      | [Execute Critical Section]                                      |
      |                                                                 |
      | ====================== RELEASE PHASE ========================== |
      |                                                                 |
      | --- RELEASE --------------------------------------------------> |
      |                                                                 |
      |                                              [If queue not empty:]
      |                                              [  pop next highest priority request Sk]
      |                                              [  send GRANT to Sk]
      |                                              [Else: voted = FALSE]
```

---

#### 4. The Deadlock Vulnerability in Basic Maekawa's Algorithm

Unlike Lamport's algorithm, Maekawa's algorithm is **inherently vulnerable to deadlocks** because a site can grant its vote to only one requester at a time, creating circular wait conditions.

##### 4.1 Deadlock Trace Example ($N = 3$)
Let $R_1 = \{S_1, S_2\}$, $R_2 = \{S_2, S_3\}$, $R_3 = \{S_3, S_1\}$.
Suppose $S_1, S_2, S_3$ request the Critical Section simultaneously:

| Event Step | Action | State of Votes Granted | Waiting Status |
| :---: | :--- | :--- | :--- |
| **1** | $S_1$ sends `REQUEST` to $R_1 = \{S_1, S_2\}$ | $S_1$ votes for $S_1$ (`voted` = TRUE) | $S_1$ waits for $S_2$ |
| **2** | $S_2$ sends `REQUEST` to $R_2 = \{S_2, S_3\}$ | $S_2$ votes for $S_2$ (`voted` = TRUE) | $S_2$ waits for $S_3$ |
| **3** | $S_3$ sends `REQUEST` to $R_3 = \{S_3, S_1\}$ | $S_3$ votes for $S_3$ (`voted` = TRUE) | $S_3$ waits for $S_1$ |
| **4** | $S_1$'s request arrives at $S_2$ | $S_2$ already voted for $S_2$; enqueues $S_1$ | $S_1$ blocked at $S_2$ |
| **5** | $S_2$'s request arrives at $S_3$ | $S_3$ already voted for $S_3$; enqueues $S_2$ | $S_2$ blocked at $S_3$ |
| **6** | $S_3$'s request arrives at $S_1$ | $S_1$ already voted for $S_1$; enqueues $S_3$ | $S_3$ blocked at $S_1$ |

$$\text{Circular Wait Dependency: } S_1 \xrightarrow{\text{waits for}} S_2 \xrightarrow{\text{waits for}} S_3 \xrightarrow{\text{waits for}} S_1$$
**Deadlock occurs!** None of the sites can ever acquire all $K=2$ votes.

---

#### 5. Deadlock Resolution: Auxiliary Message Protocol

To prevent deadlocks, Maekawa introduced three auxiliary control messages that allow an arbiter to temporarily reclaim a vote from a site that cannot currently enter the Critical Section:

```
+---------------------------------------------------------------------------------------------------+
|                                 AUXILIARY DEADLOCK MESSAGE SYSTEM                                 |
+---------------------------------------------------------------------------------------------------+
|                                                                                                   |
|  1. FAILED Message:                                                                               |
|     - Sent by arbiter Sj to requester Sk when Sk's request arrives and cannot be granted,        |
|       and Sk has lower priority than an earlier request.                                          |
|                                                                                                   |
|  2. INQUIRE Message:                                                                              |
|     - Sent by arbiter Sj to its current vote_holder (Si) IF a new request from Sk arrives with    |
|       HIGHER priority (TS_k < TS_i).                                                              |
|     - Asks: "Are you blocked? If so, can you temporarily yield your vote?"                        |
|                                                                                                   |
|  3. YIELD Message:                                                                                |
|     - Sent by vote_holder Si back to arbiter Sj in response to INQUIRE IF AND ONLY IF Si has       |
|       received at least one FAILED message from another member of its quorum.                     |
|     - Si yields the vote to let Sk proceed, breaking the cycle!                                   |
|                                                                                                   |
+---------------------------------------------------------------------------------------------------+
```

##### 5.1 Step-by-Step Cycle Resolution
1. Arbiter $S_j$ had granted its vote to $S_i$.
2. A new request from $S_k$ arrives at $S_j$ with higher priority ($TS_k < TS_i$).
3. $S_j$ sends `INQUIRE` to $S_i$.
4. If $S_i$ has received a `FAILED` message from another site, $S_i$ knows it cannot enter the CS anyway.
5. $S_i$ replies with `YIELD` to $S_j$.
6. $S_j$ reclaims its vote and grants it to $S_k$ (`GRANT` sent to $S_k$).
7. When $S_k$ finishes the CS and releases, $S_j$ restores the vote to $S_i$. Deadlock is provably prevented!

---

#### 6. Formal Correctness Proof & Complexity Analysis

##### 6.1 Safety (Mutual Exclusion) Proof
* **Theorem:** *At most one site can execute in the Critical Section at any given time.*
* **Proof by Contradiction:**
  1. Suppose two distinct sites $S_i$ and $S_j$ ($i \ne j$) are simultaneously executing in the Critical Section.
  2. For $S_i$ to enter the CS, it must have received a `GRANT` message from every site in its request set $R_i$.
  3. For $S_j$ to enter the CS, it must have received a `GRANT` message from every site in its request set $R_j$.
  4. By the Intersection Axiom of Maekawa's request sets:
     $$R_i \cap R_j \ne \emptyset \implies \exists S_{arb} \in R_i \cap R_j$$
  5. The site $S_{arb}$ is a member of both $R_i$ and $R_j$.
  6. By definition of the arbiter protocol, an arbiter can grant its vote to at most one site at any time (`voted` remains `TRUE` until `RELEASE` is received).
  7. Therefore, $S_{arb}$ could not have granted its vote to both $S_i$ and $S_j$ simultaneously.
  8. Contradiction! Hence, mutual exclusion is strictly guaranteed.

##### 6.2 Complexity Metrics
* **Message Complexity:**
  * Without conflicts: $3\sqrt{N}$ messages ($\sqrt{N}$ `REQUEST` $+ \sqrt{N}$ `GRANT` $+ \sqrt{N}$ `RELEASE`).
  * With conflicts and auxiliary messages: bounded between **$3\sqrt{N}$ and $5\sqrt{N}$ messages**.
* **Synchronization Delay:** **$2T$** (one message delay $T$ for `RELEASE` to reach arbiter, plus one $T$ for `GRANT` to reach the next requester).

---

### Q2. Architectural and Algorithmic Study of Raymond's Tree-Based Token Algorithm: Logical Spanning Tree Topology, Data Structures, Message Flow, Starvation-Freedom, and Proof of Correctness.

---

#### 1. Conceptual Foundation
Proposed by Kerry Raymond in 1989, this algorithm achieves distributed mutual exclusion by organizing nodes into a **logical directed spanning tree**, drastically reducing message overhead from $O(N)$ or $O(\sqrt{N})$ down to **$O(\log N)$**.

##### 1.1 Structural Philosophy
* Rather than broadcasting requests globally across the network, requests and the token travel strictly along the edges of a pre-established directed spanning tree.
* Edges in the tree are directed toward the node that currently holds the token.
* No node needs global knowledge of the entire cluster; each node knows only its immediate neighbors in the spanning tree.

```
+---------------------------------------------------------------------------------------------------+
|                                 RAYMOND'S LOGICAL DIRECTED TREE                                   |
+---------------------------------------------------------------------------------------------------+
|                                                                                                   |
|                                       [ Node 1 ] (TOKEN HOLDER)                                   |
|                                      holder = self                                                |
|                                       /          \                                                |
|                         holder = 1   /            \  holder = 1                                   |
|                                     v              v                                              |
|                               [ Node 2 ]        [ Node 3 ]                                        |
|                                /      \                 \                                         |
|                   holder = 2  /        \ holder = 2      \ holder = 3                             |
|                              v          v                 v                                       |
|                          [ Node 4 ]   [ Node 5 ]        [ Node 6 ]                                |
|                                                                                                   |
|       * Invariant: Directed edges always point towards the current token holder!                  |
|       * When the token moves from Node 1 to Node 4, all edges on that path reverse direction!     |
|                                                                                                   |
+---------------------------------------------------------------------------------------------------+
```

---

#### 2. Local State Variables and Data Structures

Each node $P_i$ in the spanning tree maintains two critical state variables:
1. `holder`: A variable indicating the location of the token relative to $P_i$:
   * If $P_i$ holds the token: `holder = self`.
   * If $P_i$ does not hold the token: `holder` stores the ID of the immediate neighbor on the directed path towards the current token holder.
2. `request_q`: A FIFO queue holding the IDs of neighbors that have requested the token through $P_i$.
   * If $P_i$ itself wishes to enter the CS, it pushes `self` into its own `request_q`.
3. `using`: Boolean flag indicating whether $P_i$ is actively executing inside the CS.
4. `asked`: Boolean flag indicating whether $P_i$ has already sent a `REQUEST` to its `holder` and is currently waiting for the token.

---

#### 3. Formal Algorithmic Specification

##### 3.1 Procedure `AssignToken()`
Executed when a node holds the token (`holder == self`) and its `request_q` is not empty:
```pascal
procedure AssignToken():
begin
    if (holder == self) and (not using) and (request_q is not empty) then
    begin
        next_node := dequeue(request_q);
        if (next_node == self) then
        begin
            using := true;
            enter_critical_section();
        end
        else
        begin
            holder := next_node;
            asked := false;
            send TOKEN to next_node;
        end;
    end;
end;
```

##### 3.2 Procedure `MakeRequest()`
Executed to propagate a request towards the token holder:
```pascal
procedure MakeRequest():
begin
    if (holder != self) and (request_q is not empty) and (not asked) then
    begin
        send REQUEST to holder;
        asked := true;
    end;
end;
```

##### 3.3 Message Handlers

###### On Receiving `REQUEST` from Neighbor $X$:
```pascal
enqueue(request_q, X);
AssignToken();
MakeRequest();
```

###### On Receiving `TOKEN`:
```pascal
holder := self;
AssignToken();
MakeRequest();
```

###### On Exiting Critical Section:
```pascal
using := false;
AssignToken();
MakeRequest();
```

---

#### 4. End-to-End Execution Trace

Consider the 6-node tree shown earlier. Initially, **Node 1 holds the token**.
Suppose **Node 4** wishes to enter the Critical Section:

```
Step 1: Node 4 requests CS
        - Node 4 enqueues 'self' into its request_q.
        - Since holder == 2 and asked == false, Node 4 sends REQUEST to Node 2.
        - Node 4 sets asked = true.

Step 2: Node 2 processes REQUEST from Node 4
        - Node 2 enqueues '4' into its request_q.
        - Since holder == 1 and asked == false, Node 2 sends REQUEST to Node 1.
        - Node 2 sets asked = true.

Step 3: Node 1 processes REQUEST from Node 2
        - Node 1 enqueues '2' into its request_q.
        - Node 1 calls AssignToken():
          * dequeues '2' from request_q.
          * sets holder = 2.
          * sends TOKEN to Node 2!

Step 4: Node 2 receives TOKEN
        - Node 2 sets holder = self.
        - Node 2 calls AssignToken():
          * dequeues '4' from request_q.
          * sets holder = 4.
          * sends TOKEN to Node 4!

Step 5: Node 4 receives TOKEN
        - Node 4 sets holder = self.
        - Node 4 dequeues 'self' from request_q.
        - Node 4 enters Critical Section!
```

##### Path Inversion Property
Notice how the directed edges on the path $4 \to 2 \to 1$ have completely reversed:
$$\text{Before: } 4 \xrightarrow{} 2 \xrightarrow{} 1 \quad \implies \quad \text{After: } 1 \xrightarrow{} 2 \xrightarrow{} 4$$
The tree remains a directed tree pointing directly to the new token holder (Node 4).

---

#### 5. Complexity and Correctness Analysis

##### 5.1 Correctness Proofs
* **Mutual Exclusion:** Guaranteed because exactly one physical token exists in the network at all times, and only the node with `holder == self` and `using == true` can execute in the CS.
* **Deadlock Freedom:** The network topology is a tree (an acyclic connected graph). Every request travels strictly toward the token holder along the unique acyclic path. Hence, circular wait is impossible.
* **Starvation Freedom:** Each node services requests strictly in First-In-First-Out (FIFO) queue order. Because the number of nodes is finite, every request in the tree is guaranteed to reach the front of all intermediate queues and be serviced in finite time.

##### 5.2 Performance Metrics
* **Average Message Complexity:**
  * In a balanced tree of degree $d$: average path length is $O(\log_d N)$.
  * Message complexity per CS entry is **$O(\log N)$ messages**.
  * Best case: $0$ messages (if node already holds token).
  * Worst case (unbalanced linear tree): $O(N)$ messages.
* **Synchronization Delay:**
  * Average: **$T \log N$**.
  * Worst case: $T \times \text{diameter of tree}$.

---

### Q3. Priority-Based Distributed Deadlock Prevention Schemes: Theoretical Foundations, Algorithmic Rules of Wait-Die vs. Wound-Wait, Execution Traces, and Exhaustive Comparison.

---

#### 1. Theoretical Foundations of Distributed Deadlock Prevention

In distributed database and transaction systems, transactions acquire shared and exclusive locks on distributed data items. When transactions wait for locks held by others, deadlocks can occur.
* **Deadlock Prevention Philosophy:** Unlike deadlock detection (which allows deadlocks to occur and subsequently aborts transactions), deadlock prevention **ensures before a transaction waits that a circular wait condition can never form**.

##### 1.1 Priority Assignment via Lamport Timestamps
* Every transaction $T_i$ is assigned a unique Lamport timestamp $TS(T_i)$ at the moment of its creation:
  $$TS(T_i) = \langle \text{creation\_time}, \text{site\_id} \rangle$$
* **Total Order Priority:**
  * If $TS(T_i) < TS(T_j)$, transaction $T_i$ is **older** and has **higher priority**.
  * If $TS(T_i) > TS(T_j)$, transaction $T_i$ is **younger** and has **lower priority**.
* **Starvation-Freedom Invariant:** When a transaction is aborted and restarted, it **retains its original creation timestamp**. As older transactions finish, the restarted transaction progressively becomes the oldest in the system, guaranteeing eventual completion without starvation!

---

#### 2. Algorithmic Specifications: Wait-Die vs. Wound-Wait

```
+---------------------------------------------------------------------------------------------------+
|                                  WAIT-DIE VS. WOUND-WAIT LOGIC                                    |
+---------------------------------------------------------------------------------------------------+
|                                                                                                   |
|    WAIT-DIE SCHEME (Non-Preemptive)                                                               |
|    Suppose transaction Ti requests a resource currently held by Tj:                               |
|                                                                                                   |
|    * If TS(Ti) < TS(Tj)  [Ti is OLDER]:                                                           |
|          Ti is allowed to WAIT.                                                                   |
|    * If TS(Ti) > TS(Tj)  [Ti is YOUNGER]:                                                         |
|          Ti DIES (Ti is aborted, rolls back, and restarts with original timestamp).              |
|                                                                                                   |
|    --------------------------------------------------------------------------------------------   |
|                                                                                                   |
|    WOUND-WAIT SCHEME (Preemptive)                                                                 |
|    Suppose transaction Ti requests a resource currently held by Tj:                               |
|                                                                                                   |
|    * If TS(Ti) < TS(Tj)  [Ti is OLDER]:                                                           |
|          Ti WOUNDS Tj (Tj is preempted/aborted, yields lock to Ti, and restarts).                 |
|    * If TS(Ti) > TS(Tj)  [Ti is YOUNGER]:                                                         |
|          Ti is allowed to WAIT.                                                                   |
|                                                                                                   |
+---------------------------------------------------------------------------------------------------+
```

##### 2.1 The Direction of Wait-For Edges
* **In Wait-Die:** An edge $T_i \to T_j$ in the Wait-For Graph exists **only if** $TS(T_i) < TS(T_j)$ (edges point strictly from Older to Younger).
* **In Wound-Wait:** An edge $T_i \to T_j$ in the Wait-For Graph exists **only if** $TS(T_i) > TS(T_j)$ (edges point strictly from Younger to Older).
* **Acyclicity Theorem:** In both protocols, all directed edges in the Wait-For Graph point strictly in one direction along the total timestamp ordering. Since a strictly monotonic order cannot form a cycle, **deadlocks are mathematically impossible!**

---

#### 3. Step-by-Step Scenario Analysis & Comparative Traces

Consider three transactions:
* $T_{10}$ (Oldest, $TS = 10$)
* $T_{20}$ (Middle, $TS = 20$)
* $T_{30}$ (Youngest, $TS = 30$)

##### Scenario A: Resource Contention under Wait-Die

```
1. T10 holds Resource R1.
2. T20 requests Resource R1:
   - Requester: T20 (Younger), Holder: T10 (Older).
   - Rule: Younger requests Older ===> T20 DIES!
   - T20 is rolled back and restarts.

3. T30 holds Resource R2.
4. T10 requests Resource R2:
   - Requester: T10 (Older), Holder: T30 (Younger).
   - Rule: Older requests Younger ===> T10 WAITS!
   - T10 blocks until T30 commits.
```

##### Scenario B: Resource Contention under Wound-Wait

```
1. T10 holds Resource R1.
2. T20 requests Resource R1:
   - Requester: T20 (Younger), Holder: T10 (Older).
   - Rule: Younger requests Older ===> T20 WAITS!
   - T20 blocks until T10 releases R1.

3. T30 holds Resource R2.
4. T10 requests Resource R2:
   - Requester: T10 (Older), Holder: T30 (Younger).
   - Rule: Older requests Younger ===> T10 WOUNDS T30!
   - T30 is immediately aborted; lock R2 is granted to T10.
   - T30 restarts with original timestamp 30.
```

---

#### 4. Exhaustive 8-Dimension Comparison Matrix

| Evaluation Dimension | Wait-Die Scheme | Wound-Wait Scheme |
| :--- | :--- | :--- |
| **Preemption Paradigm** | **Non-preemptive:** A transaction holding a resource is never forcibly preempted. | **Preemptive:** An older transaction forcibly preempts a younger holder. |
| **Rule: Old requests Young** | Older transaction is permitted to **Wait**. | Older transaction **Wounds** (aborts) the younger transaction. |
| **Rule: Young requests Old** | Younger transaction **Dies** (aborted and restarted). | Younger transaction is permitted to **Wait**. |
| **Frequency of Rollbacks** | **High:** A younger transaction may die repeatedly as it competes with older transactions. | **Low:** Younger transactions wait; older transactions abort younger holders only when necessary. |
| **Wasted Computation** | Higher: transactions that are aborted may have already performed extensive work. | Lower: older transactions minimize aborts of work nearing completion. |
| **WFG Edge Direction** | Directed edges point strictly from **Older $\to$ Younger**. | Directed edges point strictly from **Younger $\to$ Older**. |
| **Message Overhead** | Lower message traffic during aborts (requester self-aborts). | Slightly higher message traffic (wound message sent to holder). |
| **Starvation Prevention** | Guaranteed (restarted transactions keep original timestamps). | Guaranteed (restarted transactions keep original timestamps). |

---

### Q4. Centralized and Hierarchical Distributed Deadlock Detection: Ho-Ramamoorthy Two-Phase and One-Phase Algorithms, and the Menasce-Muntz Hierarchical Algorithm.

---

#### 1. Centralized Deadlock Detection: Foundations & False Deadlocks

In centralized deadlock detection, individual distributed sites maintain local status tables, and a designated **Central Coordinator** periodically collects this information to construct a global Wait-For Graph (WFG) and detect cycles.

##### 1.1 The Phantom (False) Deadlock Problem
In asynchronous distributed networks, messages experience unpredictable transit delays.
* Suppose transaction $T_1$ releases lock $L_1$ and requests lock $L_2$.
* If the release message is delayed in transit while the request message arrives quickly at the coordinator:
  * The coordinator may record both the old wait dependency and the new wait dependency simultaneously.
  * A **phantom cycle** appears in the global WFG that does not exist in reality!
  * Aborting a transaction based on a phantom deadlock wastes system resources.

```
Site 1: T1 releases Resource A ----------------(Release msg delayed)---------> [ COORDINATOR ]
Site 2: T1 requests Resource B ----------(Request msg arrives FAST)--------->   Records false cycle!
```

---

#### 2. Ho-Ramamoorthy Deadlock Detection Algorithms

To eliminate phantom deadlocks, Ho and Ramamoorthy developed two robust centralized algorithms:

##### 2.1 Ho-Ramamoorthy Two-Phase Algorithm

```
COORDINATOR                                                  DISTRIBUTED SITES
    |                                                               |
    | ================= PHASE 1: STATUS COLLECTION ================ |
    |                                                               |
    | --- Request Status Tables ----------------------------------> |
    | <-- Status Tables (Resources Allocated & Requested) --------- |
    |                                                               |
    | [Construct Global WFG]                                        |
    | [Search for Cycles]                                           |
    | [If NO cycle: System is Deadlock-Free!]                       |
    | [If CYCLE detected: Could be PHANTOM! Proceed to Phase 2]     |
    |                                                               |
    | ================= PHASE 2: CYCLE VERIFICATION =============== |
    |                                                               |
    | --- Request Re-transmission of Tables for Cycle Sites ------> |
    | <-- Updated Status Tables ----------------------------------- |
    |                                                               |
    | [Construct Verified Sub-WFG using ONLY edges present in BOTH] |
    | [If cycle PERSISTS in Verified Sub-WFG: GENUINE DEADLOCK!]    |
```

* **Phase 1 (Collection):**
  * Coordinator requests every site to transmit its local Status Tables (Resource Allocation Table and Resource Wait Table).
  * Coordinator builds a tentative global WFG and runs a cycle detection algorithm (e.g., Tarjan's strongly connected components or DFS).
  * If no cycle is found, the system is clean.
* **Phase 2 (Verification):**
  * If a cycle is detected, the coordinator requests a second snapshot *only* from the sites involved in the detected cycle.
  * The coordinator constructs a **verified WFG** using only edges that appear in **both the Phase 1 and Phase 2 reports**.
  * **Theorem:** An edge present in both snapshots represents a stable dependency that persisted across the entire observation interval. If the cycle persists, a **genuine deadlock is confirmed**.

##### 2.2 Ho-Ramamoorthy One-Phase Algorithm
* Eliminates the second communication phase by requiring sites to maintain two status tables:
  1. **Resource Status Table:** Lists resources held and requested by transactions.
  2. **Process Status Table:** Lists transactions executing, resources allocated, and resources awaited.
* Sites maintain epoch sequence numbers. A single snapshot of both tables allows the coordinator to mathematically verify consistency, resolving deadlocks in **a single communication phase**.

---

#### 3. Menasce-Muntz Hierarchical Deadlock Detection Algorithm

##### 3.1 Architectural Hierarchy
Centralized detection suffers from poor scalability and creates a single point of failure. The **Menasce-Muntz Algorithm** organizes deadlock controllers into a logical tree hierarchy:

```
+---------------------------------------------------------------------------------------------------+
|                            MENASCE-MUNTZ HIERARCHICAL TOPOLOGY                                    |
+---------------------------------------------------------------------------------------------------+
|                                                                                                   |
|                                     [ ROOT CONTROLLER ]                                           |
|                                     Detects global system-wide                                    |
|                                     deadlocks across all sites                                    |
|                                            /           \                                          |
|                                           /             \                                         |
|                                          v               v                                        |
|                                  [ REGION 1 ]         [ REGION 2 ]                                |
|                                  Intermediate         Intermediate                                |
|                                  Controller           Controller                                  |
|                                  /          \         /          \                                |
|                                 v            v       v            v                               |
|                              [Site 1]    [Site 2]  [Site 3]   [Site 4]                            |
|                              (Leaves: manage local databases & local WFGs)                        |
|                                                                                                   |
+---------------------------------------------------------------------------------------------------+
```

##### 3.2 Protocol Mechanics and WFG Propagation Rules
1. **Local Controllers (Leaf Nodes):**
   * Maintain the local Wait-For Graph ($LWFG$) for transactions and resources physically located at their site.
   * If a cycle contains only local transactions, the local controller resolves it immediately without notifying ancestors!
2. **External Transaction Representation:**
   * When a local transaction waits for a remote resource, it is represented in the local WFG as an external dependency arc pointing to a pseudo-node:
     $$T_i \xrightarrow{} T_{\text{external}}$$
3. **Upward Propagation to Parent Controllers:**
   * Whenever a change occurs in an external dependency edge, the controller transmits that dependency change to its immediate parent controller in the hierarchy.
4. **Intermediate & Root Controllers:**
   * Maintain an aggregated sub-WFG composed of the union of all dependency paths reported by their descendants.
   * When a cycle is detected at an intermediate controller, the cycle involves only sites within its subtree. The lowest common ancestor resolves the deadlock!
   * Only deadlocks that span across distinct regions propagate to the Root Controller.

##### 3.3 Advantages and Limitations
* **Scalability:** Reduces communication overhead: purely local deadlocks are resolved locally without global network traffic.
* **Fault Isolation:** Failure of an intermediate controller impacts only its local subtree; other subtrees continue normal operation.
* **Complex Race Conditions:** Care must be taken to prevent phantom cycles caused by out-of-order message propagation from multiple child controllers.
