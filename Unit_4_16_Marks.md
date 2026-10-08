# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT IV: CONSENSUS AND RECOVERY
### PART C: 16-MARK ESSAY QUESTIONS (EXHAUSTIVE 5-6 PAGE EXAMINATION ANSWERS)

---

# QUESTION 1
> ### **What are the key assumptions underlying while designing agreement algorithms and brief them in detail.**

---

### 1. COMPREHENSIVE INTRODUCTION & PROBLEM STATEMENT
In distributed computing systems, achieving mutual agreement (consensus) across autonomous, geographically separated nodes is the foundational prerequisite for distributed transaction commit (2PC/3PC), state-machine replication (Paxos, Raft), leader election, lock management (Google Chubby, Apache ZooKeeper), and fault-tolerant cloud clusters.

The **Agreement Problem** requires a set of $N$ processes, each starting with an initial private input value $v_i$, to agree upon a single common decision value $V$ such that the following three fundamental conditions are satisfied under all circumstances:
1. **Agreement:** All non-faulty processes must decide upon the **exact same** value $V$.
2. **Validity (Integrity):** If all non-faulty processes propose the same initial value $v$, then the agreed value must be $V = v$. (Prevents trivial protocols where all nodes unconditionally decide a hardcoded value).
3. **Termination:** Every non-faulty process must eventually reach a decision in finite time.

Designing an algorithm to solve this problem requires defining explicit environmental, structural, and behavioral assumptions. Changing these assumptions can turn a solvable problem into a mathematically impossible one.

```
+---------------------------------------------------------------------------------------------------------+
|                         THE FIVE DIMENSIONS OF AGREEMENT ALGORITHM ASSUMPTIONS                          |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       +-------------------------------------------------------------------------+                       |
|       | 1. SYNCHRONY MODEL        : Synchronous vs. Asynchronous vs. Partially  |                       |
|       |                             Synchronous Timing Assumptions              |                       |
|       +-------------------------------------------------------------------------+                       |
|       | 2. PROCESS FAILURE MODEL  : Fail-Stop, Crash-Recovery, Omission,        |                       |
|       |                             Byzantine (Arbitrary / Malicious)           |                       |
|       +-------------------------------------------------------------------------+                       |
|       | 3. COMMUNICATION CHANNELS : Reliable, FIFO, Lossy, Bounded Latency      |                       |
|       +-------------------------------------------------------------------------+                       |
|       | 4. MESSAGE AUTHENTICITY   : Oral Messages (Unsigned) vs. Signed (PKI)   |                       |
|       +-------------------------------------------------------------------------+                       |
|       | 5. NETWORK CONNECTIVITY   : Complete Graph vs. k-Connected Topologies   |                       |
|       +-------------------------------------------------------------------------+                       |
|                                                                                                         |
+---------------------------------------------------------------------------------------------------------+
```

---

### 2. DIMENSION 1: SYNCHRONY AND TIMING ASSUMPTIONS

#### A. The Synchronous System Model
* **Characteristics:**
  1. **Known Processing Bound:** There exists a known, fixed constant $\Delta_{proc}$ such that every non-faulty process executes a local computation step in time $\le \Delta_{proc}$.
  2. **Known Communication Bound:** There exists a known, fixed constant $\Delta_{msg}$ such that every message transmitted over a channel arrives at the destination in time $\le \Delta_{msg}$.
  3. **Synchronized Clocks:** Local physical clocks have a known maximum drift rate $\rho$.
* **Algorithmic Operation:** Execution proceeds in synchronized **rounds**. If a message expected in round $r$ does not arrive within round deadline $\tau = \Delta_{proc} + \Delta_{msg}$, the receiving node can deduce with 100% certainty that the sending process crashed or omitted the message.
* **Solvability Bound:** In a synchronous system with at most $f$ crash failures, consensus is solvable in **$f + 1$ rounds**.

#### B. The Asynchronous System Model
* **Characteristics:**
  * No upper bound exists on message propagation delay; messages take arbitrary but finite time.
  * No upper bound exists on relative processor execution speeds.
  * No synchronized physical clocks exist.
* **The FLP Impossibility Theorem (Fischer, Lynch, Paterson - 1985):**
  > *Theorem:* In a purely asynchronous distributed system, **no deterministic consensus protocol can guarantee both safety and liveness in the presence of even a single unannounced crash failure ($f \ge 1$).*
* **Significance:** A process cannot distinguish between a peer that has crashed and one that is simply experiencing network congestion.

```
+---------------------------------------------------------------------------------------------------------+
|                                    THE FLP AMBIGUITY DILEMMA                                            |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       Process P1 (Coordinator) waits for vote from Process P2:                                          |
|                                                                                                         |
|       SCENARIO A: Process P2 has CRASHED. (Will NEVER respond)                                          |
|       SCENARIO B: Network link to P2 is CONGESTED. (Response will arrive in 10 minutes)                 |
|                                                                                                         |
|       ===> In an asynchronous network, P1 CANNOT DISTINGUISH Scenario A from Scenario B!                |
|       ===> If P1 waits forever: Violates TERMINATION!                                                   |
|       ===> If P1 times out and decides without P2: Violates AGREEMENT if P2 makes a different decision! |
+---------------------------------------------------------------------------------------------------------+
```

#### C. Bypassing FLP in Practice
Modern distributed systems circumvent FLP impossibility using three techniques:
1. **Partial Synchrony:** Assuming the system behaves asynchrony initially, but eventually becomes synchronous with unknown bounds (e.g., Paxos, Raft).
2. **Randomization:** Using randomized coin tosses to break symmetry probabilistically (e.g., Ben-Or, Rabin algorithms).
3. **Unreliable Failure Detectors (Chandra-Toueg):** Adding an external oracle that suspects failed processes, requiring only eventual weak accuracy ($\diamondsuit\mathcal{W}$).

---

### 3. DIMENSION 2: PROCESS FAILURE SEMANTICS

```
+---------------------------------------------------------------------------------------------------------+
|                                HIERARCHY OF PROCESS FAILURE MODELS                                      |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       [ BYZANTINE / ARBITRARY ]  Malicious actions, forging messages, lying. (Hardest to solve)         |
|                  |                                                                                      |
|                  v                                                                                      |
|       [ TIMING / PERFORMANCE ]   Violating upper/lower bound deadlines in synchronous systems.          |
|                  |                                                                                      |
|                  v                                                                                      |
|       [ OMISSION FAILURES ]      Failing to execute send or receive buffer operations.                  |
|                  |                                                                                      |
|                  v                                                                                      |
|       [ CRASH-RECOVERY ]         Process halts, loses volatile RAM, reboots from stable storage.        |
|                  |                                                                                      |
|                  v                                                                                      |
|       [ CRASH-STOP / FAIL-STOP ] Process halts execution forever; easily detectable. (Easiest to solve) |
+---------------------------------------------------------------------------------------------------------+
```

1. **Fail-Stop Model:** When a process fails, it ceases all execution permanently. Other processes can reliably detect the failure.
2. **Crash-Recovery Model:** A process crashes, loses volatile memory, but recovers using non-volatile logs on stable storage.
3. **Byzantine (Arbitrary) Model:** A faulty process can exhibit arbitrary, chaotic, or coordinated malicious behavior: fabricating false values, forging sender signatures, or transmitting conflicting decisions to different peers (equivocation).

---

### 4. DIMENSION 3: MESSAGE AUTHENTICITY (ORAL VS. SIGNED MESSAGES)

#### A. Oral (Unauthenticated) Messages Model
* Messages are transmitted as plain text.
* A Byzantine process can lie about what another process told it.
* **Solvability Bound:**
  $$N \ge 3m + 1$$
  To tolerate $m$ Byzantine faulty processes, there must be at least $2m + 1$ loyal processes to outvote them.

#### B. Signed (Authenticated) Messages Model
* Each process has a private key for cryptographic digital signatures (PKI).
* Messages cannot be forged or tampered with in transit.
* **Solvability Bound:**
  $$N \ge m + 1$$
  Tolerates $m$ Byzantine traitors with only $m + 1$ nodes because a traitor cannot forge contradictory messages without invalidating cryptographic signatures.

---

### 5. DIMENSION 4 & 5: CHANNEL RELIABILITY AND NETWORK CONNECTIVITY
* **Channel Assumptions:** Reliable channels guarantee that every message sent between non-faulty processes is eventually delivered without duplication or corruption.
* **Graph Connectivity Bound:** To tolerate $m$ Byzantine nodes, the network graph must have vertex connectivity:
  $$\kappa(G) \ge 2m + 1$$
  This ensures that between any two non-faulty processes, there are at least $m + 1$ disjoint paths, guaranteeing that at least one path is entirely free of Byzantine nodes.

---
---

# QUESTION 2
> ### **Discuss the issues in failure recovery with an example.**

---

### 1. INTRODUCTION TO DISTRIBUTED FAILURE RECOVERY
In a distributed system, processes execute concurrently and interact through message passing. When a node crashes, it loses the contents of its volatile memory.
**Rollback Recovery** restores the system to a globally consistent state using previously saved checkpoints and message logs. However, because message passing creates cross-process causal dependencies, simply rolling back a failed process in isolation can introduce severe anomalies across surviving processes.

```
+---------------------------------------------------------------------------------------------------------+
|                                 DISTRIBUTED EXECUTION AND CHECKPOINTS                                   |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       P1  ----[C1,0]-------------------[C1,1]--------------------------[C1,2]---> (Time)               |
|                                         \                    /                                          |
|                                          \ m1               / m2                                        |
|                                           v                v                                            |
|       P2  ----[C2,0]------------[C2,1]---------------------[C2,2]---- (CRASHES at t!)                  |
|                                                                                                         |
|       * C_i,k denotes the k-th local checkpoint recorded by process P_i.                                 |
+---------------------------------------------------------------------------------------------------------+
```

---

### 2. FUNDAMENTAL ISSUES IN DISTRIBUTED FAILURE RECOVERY

```
+---------------------------------------------------------------------------------------------------------+
|                                   FAILURE RECOVERY ANOMALIES                                            |
+--------------------------+------------------------------------------------------------------------------+
| ISSUE                    | CAUSE AND ANOMALOUS SYMPTOM                                                  |
+--------------------------+------------------------------------------------------------------------------+
| 1. Orphan Messages       | Receive event is recorded in a checkpoint, but the send event is undone by   |
|                          | rollback. State depends on a non-existent past event.                        |
|                          |                                                                              |
| 2. Lost Messages         | Send event is recorded, but the receive event is undone by rollback.        |
|                          | Message vanishes from the active execution state.                            |
|                          |                                                                              |
| 3. The Domino Effect     | Uncoordinated checkpoints cause cascading rollbacks across dependent nodes,  |
|                          | erasing all computation back to startup.                                     |
|                          |                                                                              |
| 4. Livelock              | Endless cycles of rollbacks caused by mutual dependencies during recovery.   |
|                          |                                                                              |
| 5. Output Commit Hazard  | System interacts with the outside world (ATM cash dispenser) before the     |
|                          | corresponding state is irrevocably committed.                                |
+--------------------------+------------------------------------------------------------------------------+
```

---

### 3. DETAILED WALKTHROUGH EXAMPLE: ORPHAN MESSAGES AND LOST MESSAGES

Consider two processes $P_1$ and $P_2$ with local checkpoints $C_{1,0}, C_{1,1}$ and $C_{2,0}, C_{2,1}$.

```
+---------------------------------------------------------------------------------------------------------+
|                                ORPHAN MESSAGE AND LOST MESSAGE TRACE                                    |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       P1  ---[C1,0]-------------------[C1,1]------------------- (Crashes!)                             |
|                          \                 /                                                            |
|                           \ m1            / m2 (Orphan Hazard!)                                         |
|                            v             v                                                              |
|       P2  ---[C2,0]------------[C2,1]--------------------------                                         |
|                                                                                                         |
+---------------------------------------------------------------------------------------------------------+
```

#### Detailed Scenario Analysis:
1. **The Orphan Message Problem:**
   * Suppose $P_1$ sends message $m_2$ to $P_2$ **after** $P_1$ takes checkpoint $C_{1,1}$.
   * $P_2$ receives $m_2$ and incorporates its effects into its local state before taking checkpoint $C_{2,1}$.
   * Now, $P_1$ crashes and restarts from its latest checkpoint $C_{1,1}$.
   * At $C_{1,1}$, $P_1$ has **not yet sent** $m_2$.
   * However, $P_2$'s active state (or checkpoint $C_{2,1}$) reflects the **receipt** of $m_2$.
   * Message $m_2$ is an **Orphan Message**. The global state $\{C_{1,1}, C_{2,1}\}$ is inconsistent because an effect exists without its cause.
   * **Required Resolution:** $P_2$ must also be forced to roll back to a point prior to receiving $m_2$.

2. **The Lost Message Problem:**
   * Consider message $m_1$ sent by $P_1$ before $C_{1,1}$.
   * $P_2$ receives $m_1$ *after* checkpoint $C_{2,0}$.
   * If $P_2$ crashes and rolls back to $C_{2,0}$, $P_2$ loses the recorded receipt of $m_1$.
   * But $P_1$ did not roll back, so $P_1$ considers $m_1$ already sent and delivered.
   * Message $m_1$ is **Lost**.
   * **Required Resolution:** The recovery subsystem must log and replay $m_1$ from stable storage to re-deliver it to $P_2$.

---

### 4. THE DOMINO EFFECT (CASCADING ROLLBACKS)

When processes take checkpoints independently without synchronization, recovery dependencies can propagate backward indefinitely.

```
+---------------------------------------------------------------------------------------------------------+
|                                    THE DOMINO EFFECT PHENOMENON                                         |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       P1  ---[C1,0]--------------[C1,1]---------------------[C1,2]--->                                  |
|                    \               \                 /                                                  |
|                     \ m0            \ m1            / m2                                                |
|                      v               v             v                                                    |
|       P2  ---[C2,0]--------[C2,1]-------------[C2,2]---- (Crash at t!)                                  |
|                                                                                                         |
|  1. P2 crashes at time t and rolls back to checkpoint C2,2.                                             |
|  2. Rolling back to C2,2 makes m2 an orphan message, because m2 was sent by P1 after C1,2.             |
|  3. To eliminate the orphan, P1 must roll back to C1,1.                                                 |
|  4. But rolling P1 back to C1,1 makes m1 an orphan message, because m1 was received by P2 prior to C2,2!|
|  5. Consequently, P2 is forced to roll back further to C2,1.                                           |
|  6. Rolling P2 back to C2,1 in turn makes m0 invalid, forcing P1 back to C1,0.                         |
|  7. This chain reaction forces all processes all the way back to their initial state (C1,0, C2,0),      |
|     erasing hours of valid computation. This catastrophe is known as the DOMINO EFFECT.                 |
+---------------------------------------------------------------------------------------------------------+
```

---
---

# QUESTION 3
> ### **Write about the issues in failure recovery and discuss about any two recovery mechanisms.**

---

### 1. OVERVIEW OF RECOVERY MECHANISMS
To resolve the failure recovery issues discussed above (orphan messages, lost messages, and the domino effect), distributed systems deploy two primary recovery mechanisms:
1. **Coordinated Checkpointing (e.g., The Koo-Toueg Algorithm)**
2. **Asynchronous Checkpointing with Dependency Tracking (e.g., The Juang-Venkatesan Algorithm)**

---

### 2. RECOVERY MECHANISM 1: KOO-TOUEG COORDINATED CHECKPOINTING ALGORITHM

The **Koo-Toueg Algorithm** guarantees that a newly recorded global checkpoint set is always consistent, completely eliminating the domino effect.

#### A. Operating Principles:
* Employs a **Two-Phase Commit (2PC)** protocol style.
* Processes take **Tentative Checkpoints** during Phase 1.
* Tentative checkpoints are committed as **Permanent Checkpoints** in Phase 2 only if all dependent processes succeed.
* Uses FIFO channels and blocks message transmission during checkpoint creation to prevent in-flight messages from crossing the checkpoint boundary.

```
+---------------------------------------------------------------------------------------------------------+
|                                    KOO-TOUEG CHECKPOINTING PROTOCOL                                     |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       INITIATOR PROCESS (Pi)                    DEPENDENT PROCESSES (Pj)                                |
|                 |                                          |                                            |
|       [ PHASE 1: PREPARE ]                                 |                                            |
|       Takes tentative checkpoint                           |                                            |
|       Sends "Take_Tentative_CKPT" ------------------------>|                                            |
|                 |                                          | Takes tentative checkpoint                 |
|                 |<----------------- Sends ACK -------------|                                            |
|                 |                                          |                                            |
|       [ PHASE 2: COMMIT ]                                  |                                            |
|       All ACKs received!                                   |                                            |
|       Sends "Make_Permanent" ----------------------------->| Converts tentative to                      |
|                 |                                          | PERMANENT checkpoint!                      |
|                 |                                          | Discards old checkpoint.                   |
+---------------------------------------------------------------------------------------------------------+
```

#### B. Koo-Toueg Rollback Recovery Phase:
When a failure occurs, the initiator coordinates recovery in two phases:
1. **Phase 1 (Prepare to Rollback):** Initiator sends `Prepare_to_Rollback` to all processes in the dependency set. Each process checks if it can roll back to its latest permanent checkpoint and replies with `OK`.
2. **Phase 2 (Rollback Commit):** Initiator broadcasts `Rollback`. All processes restore their state from their permanent checkpoints and resume execution synchronously.

---

### 3. RECOVERY MECHANISM 2: JUANG-VENKATESAN ASYNCHRONOUS CHECKPOINTING AND RECOVERY

The **Juang-Venkatesan Algorithm** allows processes to take checkpoints asynchronously without coordination during normal execution, deferring consistency calculations to recovery time.

#### A. Data Structures & Normal Execution:
1. Each process $P_i$ increments a local counter and tags checkpoints as $C_{i,k}$.
2. Every application message $m$ sent from $P_i$ to $P_j$ piggybacks:
   $$\text{Tag} = (i, k)$$
   where $k$ is $P_i$'s current checkpoint index.
3. $P_j$ maintains a **Direct Dependency Vector** $DDV_j[1..N]$, where:
   $$DDV_j[i] = \max(DDV_j[i], k)$$
   recording that $P_j$'s current state depends on $P_i$'s $k$-th checkpoint.

#### B. Recovery Protocol Upon Crash:
When site $P_i$ fails and restarts:
1. $P_i$ finds its latest intact checkpoint $C_{i,k}$ and broadcasts a `Recovery_Request(i, k)` to all peers.
2. Upon receiving the request, each peer $P_j$ inspects its dependency vector $DDV_j$.
3. If $DDV_j[i] > k$, then $P_j$'s current state depends on computation that $P_i$ has rolled back! $P_j$ must roll back to its latest checkpoint $C_{j, m}$ where $DDV_j[i] \le k$.
4. This dependency propagation repeats iteratively until all processes reach a mutually consistent recovery line.

---

### 4. COMPREHENSIVE COMPARISON OF BOTH MECHANISMS

| Evaluation Parameter | Koo-Toueg Coordinated Protocol | Juang-Venkatesan Asynchronous Protocol |
| :--- | :--- | :--- |
| **Normal Execution Overhead** | Higher (two-phase control message exchanges) | Minimal (piggybacks scalar counter on messages) |
| **Domino Effect Risk** | **Zero** (Guaranteed consistent recovery line) | **Controlled** via dependency vectors |
| **Storage Overhead** | Low (only latest permanent checkpoint needed) | High (must store multiple past checkpoints) |
| **Recovery Complexity** | Trivial (immediate restore to latest line) | Iterative vector dependency calculation |
| **Message Blocking** | Requires temporary channel synchronization | Non-blocking execution during checkpoints |

---
---

# QUESTION 4
> ### **Justify the Checkpoint-based recovery in Distributed Systems.**

---

### 1. CONCEPTUAL JUSTIFICATION
In large-scale distributed computing systems (e.g., enterprise clusters, cloud platforms, financial trading networks), component failures are not exceptional anomalies; they are daily operational certainties.
Without checkpoint-based recovery, any single component crash would require restarting the entire distributed job from $t = 0$. For a job running across 1,000 nodes with a mean time between failures (MTBF) of 10 hours, an uncheckpointed computation would **never finish**.

Checkpoint-based recovery provides:
1. **Bounded Work Loss:** Limits lost execution to the interval between the failure and the last checkpoint.
2. **ACID Durability:** Enforces transactional durability across distributed databases.
3. **High Availability:** Enables fast automatic failover and state reconstruction.

---

### 2. COMPREHENSIVE TAXONOMY OF CHECKPOINT-BASED RECOVERY

```
+---------------------------------------------------------------------------------------------------------+
|                            TAXONOMY OF CHECKPOINT-BASED RECOVERY TECHNIQUES                             |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|                                    CHECKPOINT RECOVERY TECHNIQUES                                       |
|                                                  |                                                      |
|                     +----------------------------+----------------------------+                         |
|                     |                                                         |                         |
|                     v                                                         v                         |
|            CHECKPOINTING ALGORITHMS                                  MESSAGE LOGGING PROTOCOLS          |
|                     |                                                         |                         |
|              +------+------+------+                                    +------+------+------+           |
|              |             |      |                                    |             |      |           |
|              v             v      v                                    v             v      v           |
|          UNCOORDINATED COORDINATED COMMUNICATION-                 PESSIMISTIC   OPTIMISTIC CAUSAL       |
|                        (Blocking & INDUCED (CIC)                                                        |
|                        Non-blocking)                                                                    |
+---------------------------------------------------------------------------------------------------------+
```

---

### 3. DETAILED ANALYSIS OF CHECKPOINTING STRATEGIES

#### A. Uncoordinated Checkpointing
* **Mechanics:** Each process takes local checkpoints autonomously when convenient (e.g., after memory-intensive operations).
* **Advantages:** Zero runtime communication coordination overhead.
* **Drawbacks:** Susceptible to the domino effect; garbage collection of obsolete checkpoints requires global dependency tracking.

#### B. Coordinated Checkpointing
* **Mechanics:** Processes synchronize checkpoint actions using two-phase messaging to ensure every recorded global state forms a consistent cut.
* **Variants:**
  * *Blocking:* Suspends application message transmission while checkpoints are recorded.
  * *Non-blocking (Copy-on-Write):* Uses copy-on-write memory snapshots or index markers, allowing application processing to proceed concurrently.
* **Justification:** Eliminates the domino effect and minimizes stable storage requirements to a single permanent checkpoint per process.

#### C. Communication-Induced Checkpointing (CIC)
* **Mechanics:** Processes take autonomous checkpoints, but also take **forced checkpoints** when dependency information piggybacked on application messages indicates a risk of forming a $Z$-cycle (a pattern that prevents checkpoints from belonging to any consistent recovery line).
* **Justification:** Achieves the autonomy of uncoordinated checkpointing while mathematically guaranteeing freedom from the domino effect.

---

### 4. MESSAGE LOGGING PROTOCOLS FOR ACTIVE STATE RECONSTRUCTION

When combined with checkpointing, **message logging** enables a recovering process to reconstruct its exact pre-crash state without forcing surviving processes to roll back.
Under the **Piecewise Deterministic (PWD)** assumption, execution consists of alternating deterministic computation intervals and non-deterministic events (e.g., message receipts).

```
+---------------------------------------------------------------------------------------------------------+
|                                  MESSAGE LOGGING PROTOCOLS COMPARISON                                   |
+--------------------------+------------------------------------------------------------------------------+
| LOGGING PROTOCOL         | LOGGING COMMIT POLICY & BEHAVIOR                                             |
+--------------------------+------------------------------------------------------------------------------+
| 1. Pessimistic Logging   | Synchronous commit to stable storage BEFORE processing the message. Never   |
|                          | produces orphan processes. Higher latency.                                   |
|                          |                                                                              |
| 2. Optimistic Logging    | Asynchronously buffers message logs in volatile memory. Flushes in batches.  |
|                          | Fast normal execution, but recovery may create orphans and require rollbacks.|
|                          |                                                                              |
| 3. Causal Logging        | Encodes causal dependencies directly into message headers. Determinants are |
|                          | replicated across surviving processes. Zero orphan generation with           |
|                          | non-blocking execution.                                                      |
+--------------------------+------------------------------------------------------------------------------+
```

---
