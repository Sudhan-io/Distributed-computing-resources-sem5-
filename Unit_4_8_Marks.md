# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT IV: CONSENSUS AND RECOVERY
### PART B: 8-MARK QUESTIONS AND ANSWERS (IN-DEPTH EXAM REPOSITORY)

---

## QUESTION 1: Illustrate the Different Types of Failure in Distributed Systems and Explain How to Prevent/Tolerate Them.

### 1. Introduction and Failure Taxonomy
A **failure** in a distributed system occurs when an active component (process, communication link, or storage device) deviates from its specified expected behavior. Failures are classified into a formal severity hierarchy ranging from benign crash failures to arbitrary Byzantine behaviors.

```
+-----------------------------------------------------------------------------------------+
|                               HIERARCHY OF SYSTEM FAILURES                              |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       +------------------------------------------------------------------+              |
|       |                     BYZANTINE / ARBITRARY FAILURES               |  Most Severe |
|       |       (Malicious actions, counterfeit data, forged messages)     |      |       |
|       +---------------------------------+--------------------------------+      |       |
|                                         |                                       |       |
|       +---------------------------------v--------------------------------+      |       |
|       |                     TIMING / PERFORMANCE FAILURES                |      |       |
|       |      (Responses arrive outside prescribed deadline bounds)       |      |       |
|       +---------------------------------+--------------------------------+      |       |
|                                         |                                       |       |
|       +---------------------------------v--------------------------------+      |       |
|       |                       OMISSION FAILURES                          |      |       |
|       |        (Send-omission, Receive-omission, Network packet drops)   |      |       |
|       +---------------------------------+--------------------------------+      |       |
|                                         |                                       |       |
|       +---------------------------------v--------------------------------+      v       |
|       |                  CRASH-STOP / CRASH-RECOVERY FAILURES            |  Least Severe|
|       |           (Process completely halts, loses volatile state)       |              |
|       +------------------------------------------------------------------+              |
+-----------------------------------------------------------------------------------------+
```

---

### 2. Detailed Classification of Failure Types

1. **Crash Failures (Fail-Stop & Fail-Recovery):**
   * **Fail-Stop:** A process ceases execution abruptly and halts permanently. Other processes can reliably detect the failure (e.g., via hardware watchdogs or heartbeat probes).
   * **Fail-Recovery:** A process crashes, loses volatile memory state, but subsequently reboots and rejoins the system using state stored on non-volatile stable storage.
2. **Omission Failures:**
   * **Send-Omission:** A process fails to place a message into the communication channel.
   * **Receive-Omission:** A process fails to read a message arriving at its incoming network buffer.
   * **Channel Omission:** The communication network drops packets in transit due to buffer exhaustion.
3. **Timing (Performance) Failures:**
   * Occurs exclusively in **synchronous systems** where processes or communication links violate specified lower or upper latency bounds.
4. **Byzantine (Arbitrary) Failures:**
   * The most severe failure class. A faulty process can exhibit arbitrary, chaotic, or malicious behavior: fabricating false values, forging sender signatures, or transmitting conflicting decisions to different peers (equivocation).

---

### 3. Prevention, Masking, and Tolerance Mechanisms

```
+-----------------------------------------------------------------------------------------+
|                          FAILURE PREVENTION & TOLERANCE MATRIX                          |
+--------------------------+--------------------------------------------------------------+
| FAILURE TYPE             | TOLERANCE / PREVENTION TECHNIQUE                             |
+--------------------------+--------------------------------------------------------------+
| Crash / Fail-Stop        | - State Machine Replication (Primary-Backup, Paxos, Raft).  |
|                          | - Heartbeats and Unreliable Failure Detectors (Chandra-Toueg).|
|                          | - Checkpointing to Stable Storage.                           |
|                          |                                                              |
| Omission Failures        | - Reliable Transmission Control Protocol (TCP acknowledgments|
|                          |   and retransmission timers).                                |
|                          | - Sequence numbering and duplicate suppression.              |
|                          |                                                              |
| Timing Failures          | - Dynamic timeout adjustment algorithms.                     |
|                          | - Quality of Service (QoS) bandwidth guarantees.             |
|                          |                                                              |
| Byzantine Failures       | - Byzantine Fault Tolerant (BFT) protocols (PBFT).           |
|                          | - Asymmetric Digital Signatures (RSA/ECDSA authentication).  |
|                          | - Quorum-based voting systems requiring N >= 3m + 1 nodes.   |
+--------------------------+--------------------------------------------------------------+
```

---

## QUESTION 2: Illustrate Briefly the Two Kinds of Checkpoints for Checkpoint Algorithm.

### 1. Conceptual Need for Dual Checkpointing
In distributed checkpointing protocols (such as the **Koo-Toueg Coordinated Checkpointing Algorithm**), taking an immediate permanent checkpoint whenever an update occurs is prohibitively expensive and prone to recording inconsistent global states.
To guarantee that the resulting global state contains **no orphan messages** and forms a valid **consistent recovery line**, modern protocols decouple checkpointing into two explicit phases:
1. **Tentative Checkpoints** (Phase 1: Verification)
2. **Permanent Checkpoints** (Phase 2: Commitment)

---

### 2. Detailed Contrast: Tentative vs. Permanent Checkpoints

```
+-----------------------------------------------------------------------------------------+
|                         TENTATIVE VS. PERMANENT CHECKPOINTS                             |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       +------------------------------------+                                            |
|       |   Initiator issues Checkpoint REQ  |                                            |
|       +-----------------+------------------+                                            |
|                         |                                                               |
|                         v                                                               |
|       +------------------------------------+                                            |
|       |    PHASE 1: TENTATIVE CHECKPOINT   |                                            |
|       | - Temporary in-memory / disk state |                                            |
|       | - Old permanent checkpoint REMAINS |                                            |
|       | - Does NOT overwrite stable state  |                                            |
|       | - Can be CANCELLED / DISCARDED     |                                            |
|       +-----------------+------------------+                                            |
|                         | All processes successfully create tentative checkpoint        |
|                         v                                                               |
|       +------------------------------------+                                            |
|       |    PHASE 2: PERMANENT CHECKPOINT   |                                            |
|       | - Committed to stable storage      |                                            |
|       | - Becomes active Recovery Line     |                                            |
|       | - Previous permanent checkpoint    |                                            |
|       |   is safely garbage-collected      |                                            |
|       +------------------------------------+                                            |
+-----------------------------------------------------------------------------------------+
```

#### Detailed Characteristics:

| Feature | Tentative Checkpoint | Permanent Checkpoint |
| :--- | :--- | :--- |
| **Creation Phase** | First Phase (Proposal / Preparation) | Second Phase (Commit / Finalization) |
| **Storage Location** | Temporary volatile or staged scratch storage | Non-volatile, fault-tolerant **Stable Storage** |
| **Commitment** | Reversible; can be aborted without side effects | Irreversible; forms an official baseline state |
| **Recovery Role** | Cannot be used directly for rollback recovery | Used directly by rollback protocols to restore nodes |
| **Old State Handling**| Preserves the previous permanent checkpoint | Overwrites/retires the older permanent checkpoint |

---

### 3. Execution Mechanics in Koo-Toueg Protocol
1. **Tentative Checkpoint Phase:**
   * Initiator process $P_i$ takes a tentative checkpoint and sends a `Take_Tentative_Checkpoint` request to all processes upon which it depends.
   * If a dependent process $P_j$ has received messages from $P_i$ since its last checkpoint, $P_j$ takes a tentative checkpoint and responds with `ACK`.
2. **Permanent Checkpoint Phase:**
   * If all dependent processes reply with `ACK`, $P_i$ broadcasts a `Commit_Checkpoint` message.
   * Each recipient converts its tentative checkpoint into a **permanent checkpoint** and discards its previous permanent checkpoint.
   * If any process replies with `NACK` (or times out), $P_i$ broadcasts an `Abort_Checkpoint` message, and all tentative checkpoints are safely discarded.

---

## QUESTION 3: Construct Byzantine Agreement Problem in Distributed System.

### 1. Problem Formulation (The Byzantine Generals Problem)
Formulated by Leslie Lamport, Robert Shostak, and Marshall Pease in 1982, the **Byzantine Agreement Problem** abstracts the challenge of reaching consensus in a network containing arbitrarily failing or malicious nodes.

* **Scenario:**
  * Several divisions of the Byzantine army are camped outside an enemy city. Each division is commanded by its own General ($P_1, P_2, \dots, P_N$).
  * One General is designated as the **Commander** (source), while the others are **Lieutenants**.
  * The Commander must send an order: either **ATTACK** (1) or **RETREAT** (0).
  * Traitors (Byzantine nodes) may exist among both the Commander and Lieutenants, attempting to sow discord and cause loyal generals to make conflicting decisions.

```
+-----------------------------------------------------------------------------------------+
|                           BYZANTINE GENERALS ARCHITECTURE                               |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|                            [ COMMANDER (Source) ]                                       |
|                                     |                                                   |
|                    +----------------+----------------+                                  |
|     Sends "ATTACK" |                                 | Sends "RETREAT"                  |
|     (Traitors lie!)|                                 | (Conflicting commands!)          |
|                    v                                 v                                  |
|            [ LIEUTENANT 1 ]                  [ LIEUTENANT 2 ]                           |
|               (Loyal)                           (Traitor)                               |
|                    |                                 |                                  |
|                    +------- Exchanges Orders --------+                                  |
|                                                                                         |
|       ===> GOAL: All LOYAL Lieutenants must decide upon the SAME action!                |
+-----------------------------------------------------------------------------------------+
```

---

### 2. Formal Correctness Conditions
The Byzantine Agreement protocol is formally correct if and only if it satisfies two conditions:
1. **Agreement:** All loyal (non-faulty) lieutenants obey the **same** command.
2. **Validity:** If the commanding general is loyal, then every loyal lieutenant obeys the command that the commander originally sent.

---

### 3. The $3m + 1$ Impossibility Boundary (Oral Messages)

* **Fundamental Theorem:** In a synchronous distributed system with unauthenticated (oral) communication messages, **no solution is possible if the number of faulty (Byzantine) processes $m$ is greater than or equal to one-third of the total processes $N$**:
  $$N \ge 3m + 1$$
* **Proof for $N = 3, m = 1$ (Why 3 Generals Fail to Tolerate 1 Traitor):**

```
CASE 1: Traitor Lieutenant                        CASE 2: Traitor Commander
=========================================         =========================================
          [ Commander (Loyal) ]                              [ Commander (Traitor) ]
             Sends "ATTACK"                                  Sends "ATTACK" / "RETREAT"
             /             \                                 /                       \
            v               v                               v                         v
     [ Lieut 1 ] <=====> [ Lieut 2 ]                 [ Lieut 1 ] <===========> [ Lieut 2 ]
       (Loyal)   Exchanges (Traitor)                   (Loyal)     Exchanges     (Loyal)
                 "RETREAT"                                         "ATTACK"
                                                                   & "RETREAT"

Lieut 1 receives:                                 Lieut 1 receives:
- From Commander: "ATTACK"                        - From Commander: "ATTACK"
- From Lieut 2:   "RETREAT"                       - From Lieut 2:   "RETREAT"
===> LIEUT 1 CANNOT DISTINGUISH CASE 1 FROM CASE 2! DECISION AMBIGUITY OCCURS!
```

* **Conclusion:** To tolerate $m = 1$ traitor, the system requires at least:
  $$N \ge 3(1) + 1 = 4 \text{ processes}$$
  This forms the basis of the **Oral Messages Algorithm $OM(m)$**, which operates over $m + 1$ synchronous communication rounds.

---

## QUESTION 4: Comparison of Agreement Protocols in Distributed Systems.

### 1. The Three Classical Agreement Protocols
In distributed systems literature, agreement protocols are categorized into three core formulations based on how initial inputs are distributed and how decisions are reached:
1. **The Consensus Problem**
2. **The Byzantine Agreement Problem**
3. **The Interactive Consistency Problem**

---

### 2. Comparative Matrix

| Evaluation Parameter | Consensus Problem | Byzantine Agreement Problem | Interactive Consistency Problem |
| :--- | :--- | :--- | :--- |
| **Initial Input Source** | Every process holds an initial private value $v_i$. | A single designated **Commander** holds an input value $v$. | Every process holds an initial private value $v_i$. |
| **Final Decision Value** | All non-faulty processes agree on a **single common value** $V$. | All non-faulty processes agree on the **commander's command** $V$. | All non-faulty processes agree on an **entire vector of values** $V = [v_1, v_2, \dots, v_N]$. |
| **Agreement Condition** | All non-faulty processes decide on the identical value $V$. | All non-faulty processes agree on the same value $V$. | All non-faulty processes agree on the identical agreement vector $V$. |
| **Validity Condition** | If all non-faulty processes propose the same initial value $v$, then $V = v$. | If the commander is non-faulty, then the decided value $V$ equals the commander's value $v$. | For every non-faulty process $P_k$, the $k$-th element in vector $V$ must satisfy $V[k] = v_k$. |
| **Fault Bound (Oral)** | $N \ge 3m + 1$ | $N \ge 3m + 1$ | $N \ge 3m + 1$ |
| **Rounds Required** | $m + 1$ rounds | $m + 1$ rounds | $m + 1$ rounds |
| **Typical Use Cases** | Distributed transaction commit (2PC/3PC), leader election. | Aircraft flight-control redundancy, mission-critical commands. | Distributed state snapshot, sensor data aggregation across nodes. |

---

## QUESTION 5: Explain the Agreement Statement That Should Be Followed in Synchronous System with Failure.

### 1. Synchronous System Model with Failures
In a **synchronous system**:
* There is a known fixed upper bound $\Delta$ on message communication delay.
* Processor execution clock drift is strictly bounded.
* Execution proceeds in clearly demarcated **rounds**. In round $r$, a process transmits messages, peers receive them within $\Delta$, and internal state updates occur before round $r+1$.

---

### 2. Agreement Statements by Failure Model

```
+-----------------------------------------------------------------------------------------+
|                       AGREEMENT STATEMENTS IN SYNCHRONOUS SYSTEMS                       |
+--------------------------+--------------------------------------------------------------+
| SYSTEM CONDITION         | FORMAL AGREEMENT SPECIFICATION                               |
+--------------------------+--------------------------------------------------------------+
| 1. Crash Failure Model   | Agreement: All non-faulty processes decide on the same      |
|    (Fail-Stop)           | value V.                                                     |
|                          | Validity: If all processes propose v, all decide v.          |
|                          | Termination: Decided in at most f + 1 rounds (where f = max  |
|                          | number of crash failures).                                   |
|                          |                                                              |
| 2. Byzantine Model       | Agreement: Every non-faulty process decides on value V.      |
|    (Oral / Unsigned)     | Weak/Strong Validity: If source is non-faulty, all loyal     |
|                          | decide its value; else decide a default NIL value.           |
|                          | Bound: Requires N >= 3m + 1 processes and m + 1 rounds.      |
|                          |                                                              |
| 3. Authenticated         | Agreement: Same as Byzantine.                                |
|    Byzantine Model       | Advantage: Digital signatures prevent message forgery.      |
|    (Signed Messages)     | Bound: N >= m + 1 processes is sufficient!                   |
+--------------------------+--------------------------------------------------------------+
```

---

## QUESTION 6: Construct the Background and Definitions of System Model for Distributed Recovery.

### 1. Distributed Execution and Checkpoint Model
A distributed system consists of $N$ processes $P_1, P_2, \dots, P_N$ executing across independent sites and communicating exclusively via message passing across reliable network links.

```
+-----------------------------------------------------------------------------------------+
|                         DISTRIBUTED RECOVERY STATE SPACE                                |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|  P1  ---[C1,0]-----------------[C1,1]-------------[C1,2]-------------------> (Time)    |
|                          \          \                                                   |
|                           \          \  m2 (Message in-transit)                         |
|                            \ m1       \                                                 |
|                             v          v                                                |
|  P2  ---[C2,0]-----------------[C2,1]-------------[C2,2]------------------->            |
|                                                                                         |
|       =========================================  <--- RECOVERY LINE (Consistent Cut)    |
+-----------------------------------------------------------------------------------------+
```

#### Key Formal Definitions:
1. **Local Checkpoint ($C_{i,k}$):** An atomic recording of the local state of process $P_i$ saved on stable storage, representing the $k$-th checkpoint of process $P_i$.
2. **Global State:** A collection of local checkpoints, exactly one from each process:
   $$G = \{C_{1, k_1}, C_{2, k_2}, \dots, C_{N, k_N}\}$$
3. **Consistent Global State (Consistent Cut):** A global state $G$ is consistent if no message is recorded as received in $G$ without being recorded as sent in $G$:
   $$\forall m: \text{receive}(m) \in G \implies \text{send}(m) \in G$$
4. **Recovery Line:** The most recent consistent global checkpoint across all processes. When a failure occurs, the system rolls back to the recovery line.
5. **Stable Storage:** Fault-tolerant, non-volatile storage (e.g., mirrored disk or distributed persistent storage) that survives process crashes and reboots.

---

## QUESTION 7: Classification of Agreement Protocols in Distributed Systems.

### 1. Classification Architecture

```
+-----------------------------------------------------------------------------------------+
|                         AGREEMENT PROTOCOL CLASSIFICATION                               |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|                               AGREEMENT PROTOCOLS                                       |
|                                        |                                                |
|            +---------------------------+---------------------------+                    |
|            |                                                       |                    |
|            v                                                       v                    |
|     SYNCHRONOUS SYSTEMS                                     ASYNCHRONOUS SYSTEMS        |
|            |                                                       |                    |
|     +------+------+                                         +------+------+             |
|     |             |                                         |             |             |
|     v             v                                         v             v             |
|   CRASH        BYZANTINE                                RANDOMIZED     FAILURE DETECTOR |
|  MODELS         MODELS                                   ALGORITHMS       BASED (PAXOS) |
| (f+1 rounds)  (3m+1 nodes)                             (Ben-Or, Rabin)    (Chandra-Toueg)
+-----------------------------------------------------------------------------------------+
```

#### Key Categories:
1. **Based on Timing Assumptions:**
   * **Synchronous Protocols:** Guaranteed message delays; deterministic round-based protocols terminate within a known time bound.
   * **Asynchronous Protocols:** No delay bounds; governed by FLP impossibility. Must rely on randomization or failure detectors to guarantee termination.
2. **Based on Failure Semantics:**
   * **Crash-Tolerant Protocols:** Handles benign crash-stop failures. Requires $N \ge 2f + 1$ majority quorum.
   * **Byzantine Fault-Tolerant (BFT) Protocols:** Handles arbitrary, malicious failures. Requires $N \ge 3m + 1$ nodes without signatures, or $N \ge m + 1$ with digital signatures.

---

## QUESTION 8: Elaborate Checkpointing and Rollback Recovery.

### 1. Overview and Core Purpose
**Checkpointing and Rollback Recovery** is a fault-tolerance technique designed to restore a distributed application to a valid, consistent state following a node crash, without requiring all processes to re-execute from the beginning of time ($t = 0$).

---

### 2. The Domino Effect (Cascading Rollbacks)
In **uncoordinated (asynchronous) checkpointing**, processes take checkpoints independently without synchronizing with peers. This can trigger the **Domino Effect**, where a single failure causes an endless chain of rollbacks backward in time.

```
+-----------------------------------------------------------------------------------------+
|                               THE DOMINO EFFECT PHENOMENON                              |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|  P1  ----[C1,0]---------------------[C1,1]--------------------------[C1,2]--->          |
|                                       \                    /                            |
|                                        \ m1               / m2                          |
|                                         v                v                              |
|  P2  ----[C2,0]------------[C2,1]---------------------[C2,2]---- (CRASHES!)             |
|                                                                                         |
|  1. P2 crashes and rolls back to checkpoint C2,2.                                       |
|  2. Message m2 becomes an ORPHAN message (received by P2 at C2,2, but sent by P1        |
|     after C1,1).                                                                        |
|  3. P1 must roll back to C1,1 to eliminate the orphan send.                             |
|  4. But rolling P1 back to C1,1 turns message m1 into an orphan message!                |
|  5. P2 must roll back further to C2,1!                                                  |
|  6. This cascading chain forces both processes all the way back to C1,0 and C2,0!       |
|     ===> TOTAL LOSS OF COMPUTATION!                                                     |
+-----------------------------------------------------------------------------------------+
```

---

### 3. Checkpointing Methodologies

1. **Uncoordinated Checkpointing:**
   * High autonomy; zero messaging overhead during normal execution.
   * Severe risk of the domino effect, high recovery latency, and complex garbage collection of obsolete checkpoints.
2. **Coordinated Checkpointing:**
   * Processes synchronize via control messages (e.g., Koo-Toueg) to guarantee that every checkpoint set forms a consistent recovery line.
   * Eliminates the domino effect entirely. Rollback is limited to the single most recent permanent checkpoint set.
3. **Communication-Induced Checkpointing (CIC):**
   * Processes take autonomous checkpoints, but piggyback dependency info on application messages.
   * Forces nodes to take **forced checkpoints** whenever a hazardous communication pattern ($Z$-cycle) that could cause a domino effect is detected.

---
