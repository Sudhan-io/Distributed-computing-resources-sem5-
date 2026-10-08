# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT IV: CONSENSUS AND RECOVERY
### PART B: 8-MARK QUESTIONS AND DETAILED ANSWERS (SET 2 - ALTERNATE QUESTION BANK)

---

### Q1. Explain the Paxos Consensus Algorithm with Phase 1 and Phase 2 Message Exchanges, State Variables, and Majority Quorum Mechanics.

#### 1. Introduction and Architectural Model
Paxos, introduced by Leslie Lamport, is a foundational consensus protocol designed to achieve agreement on a single value in an asynchronous network where processors may crash-stop, messages can be delayed, reordered, or duplicated, but cannot be corrupted (non-Byzantine).
* **System Model:** $N$ nodes communicate via asynchronous message passing. A majority quorum $Q = \lfloor N/2 \rfloor + 1$ is required to ensure that any two quorums overlap by at least one node:
  $$Q_1 \cap Q_2 \ne \emptyset$$
* **Roles:**
  * **Proposer:** Proposes client values with globally unique, monotonically increasing proposal numbers $n = \langle \text{seq\_num}, \text{node\_id} \rangle$.
  * **Acceptor:** Acts as the memory of the protocol; stores votes and guarantees safety.
  * **Learner:** Detects when a consensus decision has been reached by a quorum of Acceptors.

```
+----------+                     +------------+                     +----------+
| Proposer |                     | Acceptors  |                     | Learners |
+----------+                     +------------+                     +----------+
     |                                 |                                 |
     |--- Phase 1a: Prepare(n) ------->|                                 |
     |<-- Phase 1b: Promise(n, v_k) ---| (If n > minProposal)            |
     |                                 |                                 |
     |--- Phase 2a: Accept(n, v) ----->|                                 |
     |<-- Phase 2b: Accepted(n, v) ----| (If n >= minProposal)           |
     |                                 |--- Decision Notification ------>|
```

#### 2. Protocol Specification (Phases 1 and 2)

##### Phase 1: Prepare and Promise (Election & Discovery)
1. **Phase 1a (Prepare):**
   * A Proposer selects a unique proposal number $n$ and broadcasts `Prepare(n)` to a majority of Acceptors.
2. **Phase 1b (Promise):**
   * When an Acceptor receives `Prepare(n)`:
     * If $n > \text{minProposal}$, it updates its local state $\text{minProposal} \leftarrow n$.
     * It responds with `Promise(n, acceptedProposal, acceptedValue)`, promising **never to accept any future proposals numbered less than $n$**.
     * If it has already accepted a proposal, it returns the highest-numbered accepted proposal and its value.
     * Otherwise, if $n \le \text{minProposal}$, it rejects or ignores the message.

##### Phase 2: Accept and Accepted (Commitment)
1. **Phase 2a (Accept Request):**
   * If the Proposer receives `Promise` responses from a majority quorum $Q$:
     * It sets $v$ to the value corresponding to the highest `acceptedProposal` among all responses.
     * If no Acceptor reported an already accepted value, the Proposer chooses its own client-proposed value $v$.
     * It broadcasts `Accept(n, v)` to the Acceptor quorum.
2. **Phase 2b (Accepted):**
   * When an Acceptor receives `Accept(n, v)`:
     * It accepts the proposal $(n, v)$ **if and only if** $n \ge \text{minProposal}$.
     * If accepted, it updates $\text{acceptedProposal} \leftarrow n$ and $\text{acceptedValue} \leftarrow v$, and broadcasts `Accepted(n, v)` to the Proposer and all Learners.

#### 3. State Variables Maintained at Acceptors
Each Acceptor maintains three critical persistent variables on stable storage:
* $\text{minProposal}$: The highest proposal number seen in any `Prepare` request.
* $\text{acceptedProposal}$: The highest proposal number accepted so far (initialized to $-\infty$).
* $\text{acceptedValue}$: The value associated with $\text{acceptedProposal}$ (initialized to $\text{null}$).

#### 4. Safety Guarantee via Quorum Intersection
* **Theorem:** If a value $v$ is chosen with proposal number $n$, then every proposal chosen with proposal number $m > n$ has value $v$.
* **Proof Sketch:**
  * For $v$ to be chosen at $n$, a majority quorum $Q_1$ accepted $(n, v)$.
  * For any proposal $m > n$ to succeed, the proposer must receive promises from a majority quorum $Q_2$.
  * By Pigeonhole Principle, $Q_1 \cap Q_2 \ne \emptyset$. There exists at least one node $A \in Q_1 \cap Q_2$.
  * Node $A$ accepted $(n, v)$ in Phase 2, so $A$ returns $(n, v)$ in its Phase 1b response for proposal $m$.
  * The proposer for $m$ is forced to adopt value $v$. Thus, consistency is preserved.

---

### Q2. Explain the Raft Consensus Algorithm Focusing on Leader Election and Log Replication with Step-by-Step State Diagrams.

#### 1. Architectural Philosophy and State Decomposition
Raft decomposes the consensus problem into three independent, comprehensible subproblems:
1. **Leader Election:** Choosing a single server to act as coordinator.
2. **Log Replication:** Propagating and securing client state transitions across the cluster.
3. **Safety:** Guaranteeing state machine consistency across all server terms.

#### 2. Node Roles and Finite State Machine (FSM)
At any point, a Raft server exists in one of three states:
* **Follower:** Completely passive; responds only to incoming RPCs from Leaders and Candidates.
* **Candidate:** Active during elections; attempts to gather votes from a majority.
* **Leader:** Handles all client requests, coordinates log replication, and issues heartbeats.

```
          +-------------------------------------------------+
          |                                                 |
          v   Times out, starts election                    |
     +----------+ -----------------------------> +-----------+
     | Follower |                                | Candidate |
     +----------+ <----------------------------- +-----------+
          ^       Discovers current leader            |
          |       or higher term                      | Votes from
          |                                           | majority
          |       Discovers server with higher term   v
          +-------------------------------------- +--------+
                                                  | Leader |
                                                  +--------+
```

#### 3. Leader Election Mechanism
* **Terms:** Time is divided into arbitrary numerical `terms` ($e.g., t_1, t_2, \dots$), acting as logical clocks.
* **Heartbeat & Election Timers:**
  * Leaders continuously send periodic empty `AppendEntries` RPCs (heartbeats, typically every $50\text{ ms}$).
  * Followers reset their randomized **election timer** (typically $150\text{ ms} - 300\text{ ms}$) upon receiving heartbeats.
* **Election Trigger:**
  * If an election timer expires without receiving a heartbeat:
    1. Follower transitions to `Candidate`.
    2. Increments its `currentTerm`.
    3. Votes for itself.
    4. Broadcasts `RequestVote(term, candidateId, lastLogIndex, lastLogTerm)` to all peers.
* **Election Outcome:**
  * **Wins Election:** Receives votes from a majority of servers. Transitions to `Leader` and immediately broadcasts heartbeats.
  * **Loses Election:** Receives `AppendEntries` from another server with term $\ge \text{currentTerm}$. Transitions to `Follower`.
  * **Split Vote:** Multiple candidates split votes, neither achieves a majority. The randomized election timer triggers a new election with an incremented term.

#### 4. Log Replication Mechanism
1. Client sends command $c$ to the Leader.
2. Leader appends entry $\langle \text{term}, \text{index}, c \rangle$ to its local log.
3. Leader broadcasts `AppendEntries(term, leaderId, prevLogIndex, prevLogTerm, entries[], leaderCommit)` to all Followers.
4. **Follower Consistency Check:**
   * Follower checks if its log contains an entry at `prevLogIndex` with term `prevLogTerm`.
   * If yes: appends new entries and responds `Success = True`.
   * If no: rejects the request (`Success = False`).
   * Leader decrements `nextIndex` for that follower and retries until the logs match.
5. **Commitment:**
   * Once entry is replicated on a majority of servers, the Leader marks it **committed** and applies it to its local State Machine.
   * Leader returns the execution result to the client and updates `leaderCommit` in subsequent heartbeats.

---

### Q3. Compare Two-Phase Commit (2PC) and Three-Phase Commit (3PC) in Detail with State Transition Diagrams and Crash Recovery Analysis.

#### 1. Conceptual Motivation
Distributed transaction processing requires that an atomic action either commits at all participating sites or aborts at all sites.
* **Two-Phase Commit (2PC):** Simple and widely deployed, but **inherently blocking**. If the coordinator fails while participants are in the `READY` state, participants must block indefinitely.
* **Three-Phase Commit (3PC):** Proposed by Dale Skeen as a **non-blocking** commit protocol under synchronous assumptions and fail-stop node crashes without network partitions.

#### 2. Comparative State Machine Analysis

##### Two-Phase Commit (2PC) State Diagram
```
COORDINATOR:
  [INITIAL] --Send PREPARE--> [WAIT] --All Voted YES--> [COMMIT]
                                 |
                                 +---Any Voted NO-----> [ABORT]

PARTICIPANT:
  [INITIAL] --Recv PREPARE, Vote YES--> [READY] --Recv COMMIT--> [COMMIT]
                 |                         |
                 +--Vote NO-> [ABORT]      +--Recv ABORT---> [ABORT]
                               (BLOCKING STATE: If Coord crashes in READY!)
```

##### Three-Phase Commit (3PC) State Diagram
3PC introduces an intermediate state `PRECOMMIT` on both coordinator and participants, ensuring that no state can directly transition to both `COMMIT` and `ABORT`.

```
COORDINATOR:
  [INITIAL] --Send PREPARE--> [WAIT] --All Voted YES--> [PRECOMMIT] --All ACK--> [COMMIT]
                                 |
                                 +---Any Voted NO-----> [ABORT]

PARTICIPANT:
  [INITIAL] --Vote YES--> [READY] --Recv PRECOMMIT--> [PRECOMMIT] --Recv COMMIT--> [COMMIT]
                 |           |
                 +-Vote NO---+--Recv ABORT---------> [ABORT]
```

#### 3. Key Architectural Comparison Matrix

| Dimension | Two-Phase Commit (2PC) | Three-Phase Commit (3PC) |
| :--- | :--- | :--- |
| **Phases** | 2 (Prepare/Vote, Commit/Abort) | 3 (Can-Commit/Vote, Pre-Commit, Do-Commit) |
| **Blocking Property** | **Blocking:** Participants block if coordinator crashes in `READY`. | **Non-blocking:** Guaranteed under crash-stop without network partitions. |
| **Message Complexity** | $3(N-1)$ messages in normal commit. | $5(N-1)$ messages in normal commit. |
| **Latency Rounds** | 2 round trips. | 3 round trips. |
| **Skeen's Non-Blocking Rule** | Violates rule: `READY` is adjacent to both `COMMIT` and `ABORT`. | Satisfies rule: No state has transitions to both `COMMIT` and `ABORT`. |
| **Partition Tolerance** | Safe under partitions (preserves consistency, may block). | Vulnerable to split-brain inconsistency under network partitions. |

#### 4. Crash Recovery in 3PC
* If the coordinator crashes while participants are in `PRECOMMIT`:
  * A replacement coordinator is elected via termination protocol.
  * If any participant has reached `COMMIT`, all are progressed to `COMMIT`.
  * If all operational participants are in `READY` or `PRECOMMIT`, the system safely progresses to `PRECOMMIT` then `COMMIT`.
  * The protocol never aborts once any node enters `PRECOMMIT`.

---

### Q4. Explain Communication-Induced Checkpointing (CIC) Along with the Concepts of Z-paths, Z-cycles, and How Z-cycles Induce the Domino Effect.

#### 1. Overview of Communication-Induced Checkpointing (CIC)
Communication-Induced Checkpointing avoids the overhead of coordination messages by piggybacking control information on application messages:
* **Autonomous Checkpoints (Basic Checkpoints):** Taken independently by processes based on local time or workload.
* **Forced Checkpoints:** Induced by the protocol upon receiving an application message to prevent the formation of inconsistent recovery patterns.

#### 2. Z-Paths and Z-Cycles (Netzer & Xu Theory)
Let $C_{i,x}$ denote the $x$-th checkpoint taken by process $P_i$.

##### Definition of Z-Path (Zigzag Path)
A $Z$-path exists from checkpoint $C_{i,x}$ to $C_{j,y}$ if there is a sequence of messages $\langle m_1, m_2, \dots, m_k \rangle$ such that:
1. $m_1$ is sent after $C_{i,x}$.
2. For each message $m_r$ ($1 \le r < k$):
   * $m_{r+1}$ is sent either by the receiver of $m_r$ in the same checkpoint interval, or after a subsequent checkpoint.
3. $m_k$ is received by $P_j$ before $C_{j,y}$.

##### Types of Z-Paths
* **Causal Z-Path:** Corresponds to standard Lamport causality ($m_r$ is received before $m_{r+1}$ is sent).
* **Non-Causal Z-Path:** $m_{r+1}$ is sent *before* $m_r$ is received, but within the same checkpoint interval!

```
Process P1: ----[C1,1]----(Send m1)----------------------------->
                             \
Process P2: --(Send m2)-------\-----[C2,1]----------------------->
                 \             \
Process P3: ------\-------------(Recv m1)----[C3,1]--(Recv m2)-->
```

##### Definition of Z-Cycle
* A **Z-cycle** is a $Z$-path that starts and ends at the **exact same checkpoint** $C_{i,x}$.
* **Fundamental Theorem (Netzer & Xu, 1995):** A local checkpoint $C_{i,x}$ can belong to at least one consistent global recovery line **if and only if** it is not involved in any $Z$-cycle.

```
       Process P1: ---[C1,1]----(Send m1)------------(Recv m2)---
                                  \                  /
                                   \                /
       Process P2: -----------------(Recv m1)------(Send m2)----
                                      ^
                                      |
                      [Z-CYCLE: C1,1 -> m1 -> m2 -> C1,1]
              Checkpoint C1,1 is permanently USELESS and INCONSISTENT!
```

#### 3. How Z-Cycles Cause the Domino Effect
* When a $Z$-cycle exists, rolling back $P_1$ beyond $C_{1,1}$ requires un-sending $m_1$, which invalidates $P_2$'s receipt of $m_1$.
* Rolling back $P_2$ invalidates the sending of $m_2$, which in turn invalidates $P_1$'s state prior to $C_{1,1}$.
* This mutual recursive causal dependency forces both processes to cascade backwards, potentially reaching the beginning of execution ($\text{Initial State}$).

#### 4. Prevention: Model-Based Checkpointing (HMNR Heuristic)
* Protocols like **HMNR (Helary-Mostefaoui-Netzer-Raynal)** track checkpoint sequence numbers on messages.
* When a process receives a message whose piggybacked timestamp indicates that processing it without a checkpoint would close a $Z$-cycle, the process is **forced to take a checkpoint** *before* consuming the message.

---

### Q5. Classify and Analyze Message Logging Protocols: Pessimistic, Optimistic, and Causal Logging Under the Piecewise Deterministic (PWD) Model.

#### 1. Piecewise Deterministic (PWD) Assumption
* Distributed execution is modeled as alternating sequence of deterministic state transitions and non-deterministic events (NDEs).
* **Deterministic Event:** Internal compute steps whose output is strictly determined by current state.
* **Non-Deterministic Event (NDE):** Message receipt order, thread scheduling, clock interrupts.
* **Determinant:** An NDE is uniquely characterized by a tuple $\langle \text{dest}, \text{seq\_num}, \text{send\_time} \rangle$.
* **Goal of Message Logging:** Ensure determinants are safely recorded on stable storage so that a recovering process can replay execution deterministically.

#### 2. Classification of Logging Protocols

```
                          Message Logging Protocols
                                      |
         +----------------------------+----------------------------+
         |                                                         |
  Pessimistic Logging                                      Optimistic Logging
  (Synchronous Logging,                                    (Asynchronous Logging,
   Never creates orphans)                                   May create orphans,
         |                                                  Rollback cascades)
         +----------------------------+
                                      |
                                Causal Logging
                          (Piggybacks determinants,
                           Non-blocking, No orphans)
```

#### 3. Detailed Comparative Analysis

##### 1. Pessimistic Logging
* **Rule:** A process never sends a message until the determinant of the event that caused it has been synchronously written to stable storage.
  $$\forall e, \quad \text{State}(e) \to \text{send}(m) \implies \text{Logged}(e)$$
* **Advantages:**
  * No orphan processes can ever be created.
  * Recovery is strictly local: only the crashed process needs to restart and replay.
* **Disadvantages:**
  * High performance penalty due to synchronous disk I/O on critical message path.

##### 2. Optimistic Logging
* **Rule:** Determinants are buffered in volatile memory and written to stable storage asynchronously in bulk batches.
* **Advantages:**
  * Minimal runtime latency penalty during failure-free execution.
* **Disadvantages:**
  * If a process crashes before its volatile buffer is flushed, surviving processes that observed its unlogged states become **orphans** and must be rolled back.
  * Complex garbage collection and tracking of orphan states.

##### 3. Causal Logging
* **Rule:** Determinants of non-deterministic events are piggybacked on outgoing application messages and replicated in the volatile memory of causal descendants.
* **Key Invariant:** An event $e$ is logged if its determinant is stored on stable storage OR replicated in the volatile memory of at least $f+1$ independent processes.
* **Trade-off:** Combines the benefits of both: non-blocking asynchronous logging (low failure-free latency) and zero orphan creation (strictly local recovery).

#### 4. Summary Comparison

| Property | Pessimistic Logging | Optimistic Logging | Causal Logging |
| :--- | :--- | :--- | :--- |
| **I/O Latency** | High (synchronous disk write) | Zero (asynchronous buffer flush) | Zero (in-memory piggybacking) |
| **Orphan Processes** | Impossible | Possible (requires rollback) | Impossible |
| **Recovery Scope** | Strictly local | Global cascade possible | Strictly local |
| **Message Overhead** | Zero piggybacked data | Zero piggybacked data | High (vector determinants on msgs) |

---

### Q6. Describe the Byzantine Agreement Oral Messages Algorithm $OM(m)$ with an Illustrative 4-General Execution Trace ($n=4, m=1$).

#### 1. Problem Formulation & Theoretical Bounds
* **Byzantine Agreement Problem (Lamport, Shostak, Pease, 1982):** A commander process must send an order $v \in \{\text{Attack}, \text{Retreat}\}$ to $n-1$ lieutenants such that:
  1. **Agreement:** All loyal lieutenants agree on the same value.
  2. **Validity:** If the commander is loyal, then every loyal lieutenant obeys the commander's actual order.
* **Lamport's Impossibility Theorem:** No solution exists using oral messages if the total number of processes $n \le 3m$, where $m$ is the number of Byzantine (traitor) processes.
* Therefore, to tolerate $m=1$ traitor, we require at least $n = 3(1) + 1 = 4$ nodes.

#### 2. Oral Messages Algorithm $OM(m)$ Specification
The algorithm is defined recursively for $m \ge 0$:

##### Base Case $OM(0)$:
1. The commander sends its value $v$ to every lieutenant.
2. Each lieutenant adopts the value received from the commander, or a default value `RETREAT` if no value is received.

##### Recursive Step $OM(m)$ for $m > 0$:
1. The commander sends its value $v$ to every lieutenant.
2. For each lieutenant $i$:
   * Let $v_i$ be the value lieutenant $i$ receives from the commander (or default `RETREAT`).
   * Lieutenant $i$ acts as commander in algorithm $OM(m-1)$ and sends $v_i$ to the other $n-2$ lieutenants.
3. For each lieutenant $i$, and each $j \ne i$:
   * Let $v_j$ be the value lieutenant $i$ received from lieutenant $j$ in step 2 (via $OM(m-1)$).
   * Lieutenant $i$ computes its final decision as:
     $$\text{Decision}_i = \text{majority}(v_1, v_2, \dots, v_{n-1})$$

#### 3. Execution Walkthrough: 4 Nodes, 1 Traitor ($n=4, m=1$)

##### Scenario A: Traitor is a Lieutenant ($L_3$ is Traitor, Commander $C$ is Loyal)
* **Setup:** Commander $C$ is loyal and sends $v = \text{ATTACK}$ to all lieutenants $\{L_1, L_2, L_3\}$.
* **Step 1:**
  * $L_1$ receives $\text{ATTACK}$ from $C$.
  * $L_2$ receives $\text{ATTACK}$ from $C$.
  * $L_3$ (Traitor) receives $\text{ATTACK}$ from $C$.
* **Step 2 (Lieutenants cross-relay via $OM(0)$):**
  * $L_1$ relays $\text{ATTACK}$ to $L_2, L_3$.
  * $L_2$ relays $\text{ATTACK}$ to $L_1, L_3$.
  * $L_3$ (Traitor) maliciously sends $\text{RETREAT}$ to $L_1$, and $\text{ATTACK}$ to $L_2$.
* **Step 3 (Majority Decision):**
  * **$L_1$'s vector:** $\langle \text{from } C: \text{ATTACK}, \text{from } L_2: \text{ATTACK}, \text{from } L_3: \text{RETREAT} \rangle$.
    $$\text{majority}(\text{ATTACK}, \text{ATTACK}, \text{RETREAT}) = \mathbf{ATTACK}$$
  * **$L_2$'s vector:** $\langle \text{from } C: \text{ATTACK}, \text{from } L_1: \text{ATTACK}, \text{from } L_3: \text{ATTACK} \rangle$.
    $$\text{majority}(\text{ATTACK}, \text{ATTACK}, \text{ATTACK}) = \mathbf{ATTACK}$$
* **Outcome:** Both loyal lieutenants $L_1, L_2$ decide $\mathbf{ATTACK}$. Agreement and Validity are satisfied!

##### Scenario B: Traitor is the Commander ($C$ is Traitor, Lieutenants $L_1, L_2, L_3$ are Loyal)
* **Step 1:** Traitor $C$ sends conflicting orders:
  * Sends $\text{ATTACK}$ to $L_1$.
  * Sends $\text{RETREAT}$ to $L_2$.
  * Sends $\text{ATTACK}$ to $L_3$.
* **Step 2 (Lieutenants cross-relay via $OM(0)$):**
  * Loyal $L_1$ sends $\text{ATTACK}$ to $L_2, L_3$.
  * Loyal $L_2$ sends $\text{RETREAT}$ to $L_1, L_3$.
  * Loyal $L_3$ sends $\text{ATTACK}$ to $L_1, L_2$.
* **Step 3 (Majority Decision):**
  * **$L_1$'s vector:** $\langle C: \text{ATTACK}, L_2: \text{RETREAT}, L_3: \text{ATTACK} \rangle \implies \mathbf{ATTACK}$.
  * **$L_2$'s vector:** $\langle C: \text{RETREAT}, L_1: \text{ATTACK}, L_3: \text{ATTACK} \rangle \implies \mathbf{ATTACK}$.
  * **$L_3$'s vector:** $\langle C: \text{ATTACK}, L_1: \text{ATTACK}, L_2: \text{RETREAT} \rangle \implies \mathbf{ATTACK}$.
* **Outcome:** All loyal lieutenants reach the exact same decision ($\mathbf{ATTACK}$). Agreement is preserved despite the traitorous commander!
