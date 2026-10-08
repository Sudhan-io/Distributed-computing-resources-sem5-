# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT IV: CONSENSUS AND RECOVERY
### PART C: 16-MARK COMPREHENSIVE STUDY GUIDE & ANSWERS (SET 2 - ALTERNATE QUESTION BANK)

---

### Q1. Comprehensive Architectural and Formal Analysis of the Paxos Consensus Protocol: System Model, Roles, Multi-Phase Message Exchanges, Quorum Intersection Proof of Safety, Liveness Solutions, and Multi-Paxos Optimization.

---

#### 1. Introduction and Problem Context
The Distributed Consensus problem requires a set of processes in an asynchronous, fault-prone network to agree on a single value proposed by one or more participants. Leslie Lamport's **Paxos algorithm** is universally regarded as the foundational, provably correct consensus algorithm for crash-stop distributed systems.

##### 1.1 The Environment Model
* **Process Model:** $N$ processes run on independent machines. Nodes fail only by stopping (**crash-stop** or **crash-recovery** model). Byzantine behaviors (arbitrary or malicious corruption) are strictly assumed not to occur.
* **Network Model:** Processes communicate asynchronously via messages. Messages may take arbitrarily long to arrive, can be duplicated or reordered, but messages cannot be corrupted.
* **Storage Model:** Each process has access to non-volatile stable storage (persisted across restarts) where it logs critical state variables before sending responses.
* **Quorums:** A majority quorum is any subset of nodes containing at least $W = \lfloor N/2 \rfloor + 1$ members. The fundamental quorum intersection invariant states:
  $$\forall Q_1, Q_2 \subseteq \mathcal{N} \text{ such that } |Q_1| > N/2 \text{ and } |Q_2| > N/2 \implies Q_1 \cap Q_2 \ne \emptyset$$

##### 1.2 Consensus Correctness Requirements
1. **Validity (Non-triviality):** Only a value that was proposed by some proposer can be chosen.
2. **Safety (Agreement):** No two processes can ever decide on different values (at most one value is chosen).
3. **Liveness (Termination):** If a value has been proposed, eventually some process will learn the chosen value, provided a majority of nodes are operational and network partitions eventually heal.

---

#### 2. Paxos Architectural Roles and State Variables

In classic Paxos, nodes play three distinct logical roles (a single physical machine usually executes all three roles simultaneously):

```
       +-------------------------------------------------------------+
       |                      PAXOS NODE ROLES                       |
       +-------------------------------------------------------------+
       |                                                             |
       |   +------------+        +------------+       +-----------+  |
       |   |  PROPOSER  | -----> |  ACCEPTOR  | ----> |  LEARNER  |  |
       |   +------------+        +------------+       +-----------+  |
       |   Advocates for         Maintains state      Discovers &    |
       |   client values;        and acts as the      acts upon      |
       |   drives rounds         consensus memory     chosen value   |
       |                                                             |
       +-------------------------------------------------------------+
```

##### 2.1 State Variables Persisted on Acceptors
Every Acceptor $A_j$ must persistently maintain three state variables on stable storage:
* $\text{minProposal}[A_j]$: The highest proposal number seen in any `Prepare` message that $A_j$ has promised to respect. Initialized to $0$ (or $-\infty$).
* $\text{acceptedProposal}[A_j]$: The proposal number of the highest proposal that $A_j$ has accepted so far. Initialized to $\text{null}$ (or $0$).
* $\text{acceptedValue}[A_j]$: The value corresponding to $\text{acceptedProposal}[A_j]$. Initialized to $\text{null}$.

##### 2.2 Proposal Number Composition
Proposal numbers must be strictly unique and globally totally ordered across all proposers:
$$n = \langle \text{round\_number}, \text{proposer\_id} \rangle$$
Where $\text{round\_number} \in \mathbb{N}$ and $\text{proposer\_id} \in \{1, 2, \dots, N\}$.
For two proposals $n_1 = \langle r_1, p_1 \rangle$ and $n_2 = \langle r_2, p_2 \rangle$:
$$n_1 > n_2 \iff (r_1 > r_2) \lor (r_1 = r_2 \land p_1 > p_2)$$

---

#### 3. Formal Multi-Phase Protocol Mechanics

```
PROPOSER                                                    ACCEPTORS
   |                                                            |
   | ================== PHASE 1: PREPARE ====================== |
   |                                                            |
   | ----- Prepare(n) ----------------------------------------> |
   |                                                            |
   |                                              [Check: n > minProposal]
   |                                              [Set: minProposal = n  ]
   |                                              [Persist to disk       ]
   |                                                            |
   | <---- Promise(n, acceptedProposal, acceptedValue) -------- |
   |                                                            |
   | [Collect majority promises Q1]                             |
   | [Pick v: value of highest acceptedProposal, or own value]  |
   |                                                            |
   | ================== PHASE 2: ACCEPT ======================= |
   |                                                            |
   | ----- Accept(n, v) --------------------------------------> |
   |                                                            |
   |                                              [Check: n >= minProposal]
   |                                              [Set: acceptedProposal = n]
   |                                              [Set: acceptedValue = v ]
   |                                              [Persist to disk        ]
   |                                                            |
   | <---- Accepted(n, v) ------------------------------------- |
   |                                                            |
   | [Collect majority accepted Q2]                             |
   | [VALUE v IS CHOSEN!]                                       |
   |                                                            |
   | ================== PHASE 3: LEARN ======================== |
   |                                                            |
   | ----- InformLearners(v) ---------------------------------> LEARNERS
```

##### 3.1 Phase 1a: Prepare
* A Proposer picks a new proposal number $n$ such that $n$ is higher than any proposal number it has previously used.
* It transmits `Prepare(n)` to a majority quorum of Acceptors.

##### 3.2 Phase 1b: Promise
* When Acceptor $A_j$ receives `Prepare(n)`:
  $$\text{if } n > \text{minProposal}[A_j] \text{ then:}$$
  $$\quad \text{minProposal}[A_j] \leftarrow n$$
  $$\quad \text{persist}(\text{minProposal}[A_j])$$
  $$\quad \text{send } \text{Promise}(n, \text{acceptedProposal}[A_j], \text{acceptedValue}[A_j]) \text{ to Proposer}$$
  $$\text{else:}$$
  $$\quad \text{reject or ignore the Prepare request}$$

##### 3.3 Phase 2a: Accept Request
* When the Proposer gathers `Promise` messages from a majority quorum $Q_1$:
  * It examines all received tuples $(\text{acceptedProposal}, \text{acceptedValue})$.
  * Let $k = \max(\{ \text{acceptedProposal} \})$.
  * If $k \ne \text{null}$, the Proposer **must adopt** the value $v$ associated with $k$.
  * If all responses reported $\text{acceptedProposal} = \text{null}$, the Proposer is free to choose its own initial client value $v$.
* The Proposer broadcasts `Accept(n, v)` to a majority quorum of Acceptors.

##### 3.4 Phase 2b: Accepted
* When Acceptor $A_j$ receives `Accept(n, v)`:
  $$\text{if } n \ge \text{minProposal}[A_j] \text{ then:}$$
  $$\quad \text{minProposal}[A_j] \leftarrow n$$
  $$\quad \text{acceptedProposal}[A_j] \leftarrow n$$
  $$\quad \text{acceptedValue}[A_j] \leftarrow v$$
  $$\quad \text{persist}(\text{minProposal}, \text{acceptedProposal}, \text{acceptedValue})$$
  $$\quad \text{send } \text{Accepted}(n, v) \text{ to Proposer and Learners}$$
  $$\text{else:}$$
  $$\quad \text{reject the Accept request}$$

---

#### 4. Mathematical Safety Proof: Quorum Intersection

##### 4.1 Definition of "Chosen Value"
A value $v$ is **chosen** at proposal number $n$ if and only if a majority quorum of Acceptors $Q_{\text{chosen}}$ has accepted proposal $(n, v)$.

##### 4.2 Theorem (Paxos Safety Invariant)
*If a proposal $(n, v)$ is chosen, then every proposal $(m, w)$ that is chosen with $m > n$ has $w = v$.*

##### 4.3 Inductive Proof
* **Base Case:** Let $n$ be the lowest proposal number at which a value is chosen. By definition, $(n, v)$ is chosen because a majority quorum $Q_1$ accepted $(n, v)$.
* **Inductive Hypothesis:** Assume that for all proposals $k$ where $n \le k < m$, if proposal $(k, u)$ is issued, then $u = v$.
* **Inductive Step:** We must prove that if proposal $(m, w)$ is issued, then $w = v$.
  1. For proposal $m$ to be issued, its Proposer must have received `Promise` responses from a majority quorum of Acceptors $Q_2$.
  2. Because $Q_1$ and $Q_2$ are both majorities of the same set of $N$ nodes, they must intersect:
     $$\exists A \in Q_1 \cap Q_2$$
  3. Since $A \in Q_1$, node $A$ accepted $(n, v)$ during Phase 2 of round $n$. Thus, at some point, $A$'s $\text{acceptedProposal}$ was at least $n$.
  4. Since $A \in Q_2$, node $A$ sent a `Promise` message for proposal $m$.
  5. Because $m > n$, and node $A$ responded to $m$, the `Promise` message from $A$ to the proposer of $m$ reported an accepted proposal number $k$ such that:
     $$n \le k < m$$
  6. By our Inductive Hypothesis, since $n \le k < m$, the value associated with proposal $k$ was $v$.
  7. The Proposer of $m$ chooses $w$ to be the value associated with the highest accepted proposal number returned in $Q_2$. Since $k \ge n$ was reported, the highest accepted proposal number reported is at least $n$, and by inductive hypothesis, all such proposals had value $v$.
  8. Therefore, the Proposer of $m$ must set $w = v$.
* **Conclusion:** By mathematical induction, all proposals chosen with numbers greater than $n$ must carry the exact same value $v$. Safety is inviolable!

---

#### 5. Liveness Challenges and Multi-Paxos Optimization

##### 5.1 The Livelock Problem: Dueling Proposers
In basic Paxos, competing proposers can continuously preempt each other, preventing termination:

```
Proposer P1                 Acceptor Quorum                 Proposer P2
    |                              |                              |
    | ----- Prepare(n1) ---------> |                              |
    | <---- Promise(n1) ---------- |                              |
    |                              | <---- Prepare(n2) ---------- | (n2 > n1)
    |                              | ---- Promise(n2) ----------> |
    | ----- Accept(n1, v1) ------> |                              |
    |       [REJECTED: n1 < n2]    |                              |
    |                              |                              |
    | ----- Prepare(n3) ---------> |                              | (n3 > n2)
    | <---- Promise(n3) ---------- |                              |
    |                              | ----- Accept(n2, v2) ------> |
    |                              |       [REJECTED: n2 < n3]    |
```

* **Resolution:**
  * Proposers use **randomized exponential backoff** before restarting Phase 1.
  * Alternatively, elect a single **distinguished proposer** (Leader) using a failure detector.

##### 5.2 Multi-Paxos Architecture
Basic Paxos decides only a single value (**Single-Decree Paxos**) and requires 2 round-trip times (RTT) per command. **Multi-Paxos** amortizes Phase 1 across a continuous stream of log entries:
1. A stable Leader executes Phase 1 **once** for all future log slots with proposal number $n$.
2. For all subsequent client commands, the Leader skips Phase 1 entirely and executes **only Phase 2**:
   $$\text{Client} \xrightarrow{} \text{Leader} \xrightarrow{\text{Phase 2a: Accept}(i, n, v)} \text{Acceptors} \xrightarrow{\text{Phase 2b: Accepted}(i, n, v)} \text{Learners}$$
3. Cuts consensus latency from 2 RTT to **1 RTT** in steady state!

---

### Q2. In-Depth Architectural Study of the Raft Distributed Consensus Protocol: Server States, Leader Election Mechanism, Log Replication & Consistency Guarantee, Joint Consensus for Dynamic Membership Changes, and Formal Verification of the Five Core Safety Invariants.

---

#### 1. Architectural Motivation and Design Goals
Developed by Ongaro and Ousterhout (Stanford University), **Raft** was designed with explicit priority on **understandability** and **operational practicality**, addressing the notorious pedagogical and implementation hurdles of Paxos.

##### 1.1 Structural Decomposition
Raft decomposes the consensus state space into three decoupled pillars:
1. **Leader Election:** A single leader is chosen; if it crashes, a new leader is recognized cleanly.
2. **Log Replication:** The leader accepts log entries from clients and forces followers' logs to match its own.
3. **Safety:** Ensuring that any server that applies a log entry to its state machine will apply the identical log entry at that same index as all other servers.

---

#### 2. Server States, Term Chronology, and RPC Interfaces

```
                           [Timeout, Starts Election]
                  +------------------------------------------+
                  |                                          |
                  v                                          |
          +---------------+                         +-----------------+
  ------> |   FOLLOWER    |                         |    CANDIDATE    |
          +---------------+                         +-----------------+
                  ^                                          |
                  |     [Discovers Leader or Higher Term]    | [Receives Votes
                  | <----------------------------------------+  from Majority]
                  |                                          |
                  |     [Discovers Server with Higher Term]  v
                  +--------------------------------- +---------------+
                                                     |    LEADER     |
                                                     +---------------+
```

##### 2.1 Term Progression as Logical Clocks
* Time is partitioned into numerical **Terms** ($1, 2, 3, \dots$).
* Terms act as Lamport logical clocks: every server maintains `currentTerm` in persistent storage.
* Rules:
  * Whenever a server communicates with another server with a higher term, it updates its own term and immediately drops down to `Follower`.
  * If a Candidate or Leader discovers its term is stale, it instantly reverts to `Follower`.
  * RPCs containing stale term numbers are unconditionally rejected.

##### 2.2 Formal Remote Procedure Call (RPC) Signatures

###### `RequestVote` RPC (Invoked by Candidates)
* **Arguments:**
  * `term`: Candidate's current term.
  * `candidateId`: Candidate identifier requesting vote.
  * `lastLogIndex`: Index of candidate's last log entry.
  * `lastLogTerm`: Term of candidate's last log entry.
* **Results:**
  * `term`: Receiver's current term (for candidate to update itself).
  * `voteGranted`: Boolean indicating whether vote is given.

###### `AppendEntries` RPC (Invoked by Leader for Replication and Heartbeats)
* **Arguments:**
  * `term`: Leader's current term.
  * `leaderId`: Leader identifier.
  * `prevLogIndex`: Index of log entry immediately preceding new ones.
  * `prevLogTerm`: Term of entry at `prevLogIndex`.
  * `entries[]`: Log entries to store (empty for heartbeats).
  * `leaderCommit`: Leader's `commitIndex`.
* **Results:**
  * `term`: Receiver's current term.
  * `success`: Boolean, true if follower contained entry matching `prevLogIndex` and `prevLogTerm`.

---

#### 3. Protocol Mechanics

##### 3.1 Leader Election Procedure
1. **Heartbeat Monitoring:** Followers remain passive while receiving periodic heartbeats. Each follower sets a randomized **election timeout** (drawn uniformly from $150\text{ ms} - 300\text{ ms}$).
2. **Election Initiation:** If a follower's election timer elapses without hearing from a valid leader:
   * Increments `currentTerm`.
   * Transitions to `Candidate`.
   * Votes for itself.
   * Broadcasts `RequestVote` to all cluster peers.
3. **Voting Rules (Enforcing Election Safety):**
   A receiver grants its vote if and only if:
   * The candidate's term $\ge \text{currentTerm}$.
   * The receiver has not voted for any other candidate in this term (`votedFor == null || votedFor == candidateId`).
   * **Log Completeness Rule:** The candidate's log is **at least as up-to-date** as receiver's own log:
     $$\text{candidate.lastLogTerm} > \text{local.lastLogTerm} \lor$$
     $$(\text{candidate.lastLogTerm} == \text{local.lastLogTerm} \land \text{candidate.lastLogIndex} \ge \text{local.lastLogIndex})$$

##### 3.2 Log Replication and the Log Matching Invariant
A log entry contains: $\langle \text{index}, \text{term}, \text{command} \rangle$.

```
Log Index:      1     2     3     4     5     6     7     8
Leader:      [t1,x] [t1,y] [t1,z] [t2,a] [t3,b] [t3,c] [t3,d] [t3,e]
Follower 1:  [t1,x] [t1,y] [t1,z] [t2,a] [t3,b]  --> IN SYNC
Follower 2:  [t1,x] [t1,y] [t1,z] [t2,a]          --> MISSING ENTRIES
Follower 3:  [t1,x] [t1,y] [t1,z] [t2,w]          --> CONFLICT AT INDEX 4!
```

* **Step-by-Step Reconciliation:**
  1. Leader maintains for each peer:
     * $\text{nextIndex}[i]$: Index of next log entry to send to peer $i$ (initialized to $\text{leader's lastIndex} + 1$).
     * $\text{matchIndex}[i]$: Highest index known to be replicated on peer $i$ (initialized to $0$).
  2. When Leader sends `AppendEntries`, it includes $\text{prevLogIndex} = \text{nextIndex}[i] - 1$ and its corresponding term $\text{prevLogTerm}$.
  3. If Follower rejects (`success = false`): Leader decrements $\text{nextIndex}[i]$ by 1 and retransmits `AppendEntries`.
  4. Once a match is found, the follower's conflicting log entries are **overwritten** with the leader's entries, restoring strict uniformity.

---

#### 4. The Five Core Safety Invariants and Formal Verification

| Invariant | Description |
| :--- | :--- |
| **1. Election Safety** | At most one leader can be elected in any given term. |
| **2. Leader Append-Only** | A leader never overwrites or truncates its own log; it only appends new entries. |
| **3. Log Matching** | If two logs contain an entry with the same index and term, then the logs are identical in all entries up through that index. |
| **4. Leader Completeness** | If a log entry is committed in a given term, then that entry will be present in the logs of the leaders for all higher-numbered terms. |
| **5. State Machine Safety** | If a server has applied an entry at a given index to its state machine, no other server will ever apply a different entry for that index. |

##### 4.1 Proof of Leader Completeness Invariant
* **Contradiction Setup:** Suppose a log entry from term $T$ is committed at index $I$ by Leader $L_T$, but is *missing* from the log of Leader $L_U$ elected in some subsequent term $U > T$. Let $U$ be the *smallest* term greater than $T$ whose leader $L_U$ lacks this entry.
1. Since the entry was committed at term $T$, it must have been stored on a majority of cluster nodes $Q_{\text{commit}}$.
2. For Leader $L_U$ to be elected, it must have received votes from a majority of nodes $Q_{\text{vote}}$.
3. By Quorum Intersection, $Q_{\text{commit}} \cap Q_{\text{vote}} \ne \emptyset$. Let $V$ be a node belonging to both quorums:
   $$V \in Q_{\text{commit}} \cap Q_{\text{vote}}$$
4. Node $V$ accepted the entry from Leader $L_T$ before voting for Leader $L_U$.
5. When $L_U$ requested $V$'s vote, $V$ would grant its vote only if $L_U$'s log was at least as up-to-date as $V$'s log.
6. Since $V$ already possessed the entry at index $I$, $V$'s last log term is at least $T$.
7. If $L_U$'s last log term equals $V$'s last log term, $L_U$'s log must be at least as long as $V$'s, meaning $L_U$ must contain the entry.
8. If $L_U$'s last log term is strictly greater than $V$'s last log term, then $L_U$'s last log entry was created by a leader of some term $U' < U$. By our hypothesis that $U$ was the *minimal* term lacking the entry, Leader $L_{U'}$ must have contained the entry, and by the Log Matching Invariant, $L_U$ must also have inherited it.
9. Contradiction! Hence, no such leader $L_U$ can exist. The Leader Completeness property holds universally.

---

#### 5. Dynamic Cluster Membership Changes: Joint Consensus

Directly switching cluster configuration from $C_{\text{old}}$ to $C_{\text{new}}$ is unsafe because independent servers transition at different times, creating overlapping split-brain majorities:

```
Server 1: --------[ C_old ]------------------[ C_new ]---------------->
Server 2: --------[ C_old ]------------------[ C_new ]---------------->
Server 3: --------[ C_old ]--------[ C_new ]-------------------------->
                      ^
                      | TWO DISJOINT MAJORITIES COULD SIMULTANEOUSLY FORM!
```

##### 5.1 The Two-Phase Joint Consensus Solution
Raft prevents split-brain using a two-phase configuration change via an intermediate configuration $C_{\text{old,new}}$:
1. Leader receives configuration change request to $C_{\text{new}}$.
2. Leader logs and replicates configuration entry $C_{\text{old,new}}$.
3. While $C_{\text{old,new}}$ is active, all decisions (election and commit) require **independent majorities from both $C_{\text{old}}$ and $C_{\text{new}}$**:
   $$\text{Decision} \iff \text{Majority}(C_{\text{old}}) \land \text{Majority}(C_{\text{new}})$$
4. Once $C_{\text{old,new}}$ is committed, Leader logs and replicates $C_{\text{new}}$.
5. Once $C_{\text{new}}$ is committed, old nodes not in $C_{\text{new}}$ can be safely decommissioned.

---

### Q3. Distributed Commit Protocols: End-to-End Architectural and State Analysis of Two-Phase Commit (2PC) vs. Three-Phase Commit (3PC), Coordinator Failure Handling, Cooperative Termination Protocols, and Optimizations.

---

#### 1. The Distributed Atomic Commitment Problem (ACP)
In distributed transaction systems, a transaction $T$ executes across multiple independent database sites. The ACP requires all participating sites to reach a uniform decision regarding the transaction outcome.

##### 1.1 Formal ACP Correctness Conditions
* **AC1 (Uniform Agreement):** All participants that reach a decision reach the same decision ($Commit$ or $Abort$).
* **AC2 (Uniform Validity):** If any participant decides $Commit$, all participants must have previously voted $Yes$.
* **AC3 (Non-triviality):** If all participants vote $Yes$ and no failures occur, the transaction must decide $Commit$.
* **AC4 (Termination):** All non-faulty participants must eventually reach a definitive decision.

---

#### 2. Two-Phase Commit (2PC) Protocol

##### 2.1 Protocol Execution Flow

```
COORDINATOR                                                  PARTICIPANTS
    |                                                             |
    | ================= PHASE 1: VOTING PHASE =================== |
    |                                                             |
    | --- Prepare ----------------------------------------------> |
    |                                                             |
    |                                               [Execute subtransaction]
    |                                               [Write prepare log]
    |                                               [Vote Yes or No]
    |                                                             |
    | <-- Vote-Commit (YES) / Vote-Abort (NO) ------------------- |
    |                                                             |
    | [Write global decision to stable log]                       |
    |                                                             |
    | ================= PHASE 2: DECISION PHASE ================= |
    |                                                             |
    | --- Global-Commit / Global-Abort -------------------------> |
    |                                                             |
    |                                               [Apply commit/abort]
    |                                               [Release locks]
    |                                               [Write decision log]
    |                                                             |
    | <-- Acknowledgment (ACK) ---------------------------------- |
    |                                                             |
    | [Write End-of-Transaction record]                           |
```

##### 2.2 Finite State Machine (FSM) Analysis

###### Coordinator FSM
* $\text{INITIAL} \xrightarrow{\text{send Prepare}} \text{WAIT}$
* $\text{WAIT} \xrightarrow{\text{all YES received}} \text{COMMIT (send Global-Commit)}$
* $\text{WAIT} \xrightarrow{\text{any NO or timeout}} \text{ABORT (send Global-Abort)}$

###### Participant FSM
* $\text{INITIAL} \xrightarrow{\text{recv Prepare, vote YES}} \text{READY (write PREPARE log)}$
* $\text{INITIAL} \xrightarrow{\text{recv Prepare, vote NO}} \text{ABORT (write ABORT log)}$
* $\text{READY} \xrightarrow{\text{recv Global-Commit}} \text{COMMIT}$
* $\text{READY} \xrightarrow{\text{recv Global-Abort}} \text{ABORT}$

##### 2.3 The Inherent Blocking Vulnerability
* **The Blocking State:** The `READY` state of a participant is **indefinitely blocking**.
* If the Coordinator crashes after all participants enter `READY`, the participants cannot communicate to reach a decision:
  * The Coordinator might have logged `COMMIT` and notified no one before crashing.
  * The Coordinator might have crashed before logging anything.
* If all surviving participants are in `READY`, they must hold all database locks and **block indefinitely** until the Coordinator recovers.

##### 2.4 Optimizations: Presumed Abort and Presumed Commit
* **Presumed Abort (PA):** In the absence of any log record for a transaction, recovery assumes it was aborted. Eliminates the need to force-log abort decisions or acknowledge abort messages. Highly effective for systems where aborts are common.
* **Presumed Commit (PC):** Assumes transactions commit unless explicit abort records exist. Reverses log force requirements, optimizing the failure-free commit path.

---

#### 3. Three-Phase Commit (3PC) Protocol

##### 3.1 Skeen's Non-Blocking Operational Criteria
Dale Skeen proved that a commit protocol is non-blocking under synchronous crash-stop conditions if and only if its state graph satisfies two fundamental rules:
1. **Rule 1:** There exists no state from which it is possible to transition directly to both `COMMIT` and `ABORT`.
2. **Rule 2:** There exists no state that is adjacent to both a commit state and an abort state.

2PC violates Rule 2 because `READY` is directly connected to both `COMMIT` and `ABORT`. 3PC solves this by inserting a buffer state: **Pre-Commit**.

##### 3.2 3PC Phase Breakdown and Message Exchanges

```
COORDINATOR                                                  PARTICIPANTS
    |                                                             |
    | ================= PHASE 1: CAN-COMMIT ==================== |
    |                                                             |
    | --- Can-Commit? ------------------------------------------> |
    | <-- Vote-YES / Vote-NO ----------------------------------- |
    |                                                             |
    | ================= PHASE 2: PRE-COMMIT ==================== |
    |                                                             |
    | --- Pre-Commit -------------------------------------------> |
    | <-- ACK -------------------------------------------------- |
    |                                                             |
    | ================= PHASE 3: DO-COMMIT ===================== |
    |                                                             |
    | --- Do-Commit --------------------------------------------> |
    | <-- Have-Committed (ACK) --------------------------------- |
```

##### 3.3 State Transition Graph

```
COORDINATOR:
  [INITIAL] --Send Can-Commit--> [WAIT] --All YES--> [PRECOMMIT] --All ACK--> [COMMIT]
                                   |                     |
                                   +--Any NO/Timeout--> [ABORT]

PARTICIPANT:
  [INITIAL] --Vote YES---------> [READY] --Recv PreCommit-> [PRECOMMIT] --Recv Do-Commit-> [COMMIT]
    |                               |
    +--Vote NO/Timeout------------> [ABORT]
```

##### 3.4 Cooperative Termination Protocol under Coordinator Failure
When participants detect a coordinator timeout, they elect a new coordinator which interrogates the states of surviving participants:
1. **If any operational participant is in `ABORT`:** The new coordinator broadcasts `Global-Abort`.
2. **If any operational participant is in `COMMIT`:** The new coordinator broadcasts `Global-Commit`.
3. **If all operational participants are in `PRECOMMIT`:** The transaction has passed the point of no return; the coordinator broadcasts `Do-Commit`.
4. **If all operational participants are in `READY` (none in `PRECOMMIT`):** The coordinator safely broadcasts `Global-Abort`.
5. **If some are in `READY` and some in `PRECOMMIT`:** The coordinator advances all nodes to `PRECOMMIT` by sending `PreCommit`, gathers ACKs, and then issues `Do-Commit`.

---

#### 4. Exhaustive 10-Dimension Architectural Comparison

| Dimension | Two-Phase Commit (2PC) | Three-Phase Commit (3PC) |
| :--- | :--- | :--- |
| **1. Number of Phases** | 2 phases (Prepare, Commit) | 3 phases (Can-Commit, Pre-Commit, Do-Commit) |
| **2. Blocking Behavior** | **Blocking:** Nodes block if coordinator fails in `READY`. | **Non-blocking:** Nodes terminate cooperatively under crash-stop. |
| **3. Failure Model Assumption** | Asynchronous network with crash-recovery. | **Synchronous network** with crash-stop and bounded message delay. |
| **4. Network Partition Vulnerability** | Partitions cause blocking, but preserve data consistency. | Vulnerable to catastrophic **split-brain inconsistency** under partitions. |
| **5. Message Complexity** | Normal case: $3(N-1)$ messages. | Normal case: $5(N-1)$ messages. |
| **6. Latency Overhead** | 2 Round Trip Times (RTT). | 3 Round Trip Times (RTT). |
| **7. Disk Write Overhead** | 2 synchronous disk force-logs per participant. | 3 synchronous disk force-logs per participant. |
| **8. Lock Holding Duration** | Shorter duration. | Longer lock duration (held across 3 full phases). |
| **9. Practical Adoption** | Universally adopted (XA transactions, databases). | Rarely used in production due to partition vulnerability. |
| **10. Transition Buffer** | No buffer state between voting and commit. | Buffered state (`PRECOMMIT`) prevents dual transitions. |

---

### Q4. Formal Foundations of Distributed Rollback Recovery: Coordinated vs. Uncoordinated Checkpointing, The Domino Effect, Netzer-Xu Z-Path & Z-Cycle Theory, Communication-Induced Checkpointing (HMNR), and Message Logging Architectures.

---

#### 1. System Execution Model and Theoretical Preliminaries

A distributed system comprises $N$ processes $\{P_1, P_2, \dots, P_N\}$ communicating strictly through message passing over FIFO channels.
* **Local Checkpoint:** A saved snapshot of a process's local state, denoted $C_{i,k}$ ($k$-th checkpoint of process $P_i$).
* **Global State (Cut):** A collection of local states, one from each process: $C = \{C_{1, k_1}, C_{2, k_2}, \dots, C_{N, k_N}\}$.
* **In-Flight Messages:** Messages sent before a cut but received after the cut.
* **Orphan Messages:** Messages received before a cut but sent after the cut.
* **Consistent Global State (Consistent Cut):** A cut that contains **no orphan messages**:
  $$\forall m, \quad \text{recv}(m) \in C \implies \text{send}(m) \in C$$

```
              CONSISTENT CUT                          INCONSISTENT CUT
Process P1: ---[C1,1]-----(Send m1)--------    Process P1: ---[C1,1]--------(Send m1)--
                             \                                               \
                              \                                               \
Process P2: -------------------(Recv m1)---    Process P2: ----------(Recv m1)-[C2,1]--
                                                    (ORPHAN MESSAGE: Recv without Send!)
```

---

#### 2. The Domino Effect in Uncoordinated Checkpointing

In uncoordinated checkpointing, processes record checkpoints autonomously without consulting other nodes. When a crash occurs, dependencies between sent and received messages can trigger an unbounded cascade of rollbacks.

```
P1: ---[C1,0]-------------------(Send m1)---------[C1,1]----------------(Crash!)
                                  \
P2: ---[C2,0]--------[C2,1]--------(Recv m1)--------------(Send m2)----
                                                             \
P3: ---[C3,0]-----------------[C3,1]--------------------------(Recv m2)-
```

##### Cascading Rollback Step-by-Step
1. $P_1$ crashes and rolls back to its latest checkpoint $C_{1,1}$.
2. Rolling back to $C_{1,1}$ is safe for $m_1$, but suppose $P_1$ rolled back beyond the send of $m_1$.
3. Then $P_2$ holds the receipt of an un-sent message $m_1$, turning $m_1$ into an orphan. $P_2$ is forced to roll back to $C_{2,1}$.
4. Rolling back $P_2$ to $C_{2,1}$ invalidates the sending of $m_2$.
5. $P_3$ received $m_2$, so $P_3$ is forced to roll back to $C_{3,1}$.
6. If checkpoints interleave adversely, this cascading dependency cascades all the way back to the initial system state $\{C_{1,0}, C_{2,0}, C_{3,0}\}$.
* **Definition:** The **Domino Effect** is the uncontrolled recursive rollback of processes during recovery caused by orphan message dependencies, potentially discarding all computational progress.

---

#### 3. Netzer-Xu Theory: Z-Paths and Z-Cycles

In 1995, Robert Netzer and Jian Xu formulated the exact necessary and sufficient conditions governing consistent checkpoint sets using the mathematics of **Zigzag paths**.

##### 3.1 Formal Definition of a Z-Path
A **Z-path (Zigzag path)** exists from checkpoint $C_{i,x}$ to checkpoint $C_{j,y}$ if there exists a sequence of messages $\langle m_1, m_2, \dots, m_k \rangle$ such that:
1. $m_1$ is sent by $P_i$ after checkpoint $C_{i,x}$.
2. For every $r \in \{1, 2, \dots, k-1\}$:
   * Message $m_{r+1}$ is sent by the process that received $m_r$, and the send event of $m_{r+1}$ occurs in the same checkpoint interval as the receive event of $m_r$, OR in a subsequent checkpoint interval.
3. Message $m_k$ is received by $P_j$ before checkpoint $C_{j,y}$.

##### 3.2 Z-Paths vs. Lamport Causal Paths
* **Causal Path:** Requires that $m_r$ is received *before* $m_{r+1}$ is sent ($\text{recv}(m_r) \to \text{send}(m_{r+1})$).
* **Z-Path (Non-Causal):** Allows $m_{r+1}$ to be sent *before* $m_r$ is received, provided both occur within the same checkpoint interval!

##### 3.3 The Z-Cycle and the Netzer-Xu Theorem
* **Definition (Z-Cycle):** A $Z$-path that starts and ends at the **same local checkpoint** $C_{i,x}$.
* **Fundamental Theorem (Netzer & Xu, 1995):**
  *A local checkpoint $C_{i,x}$ can belong to at least one consistent global recovery line if and only if it is NOT involved in any Z-cycle.*
* **Corollary:** Checkpoints involved in $Z$-cycles are permanently **useless checkpoints** and can never be utilized for fault recovery.

```
Process P1: ---[Ci,x]----(Send m1)----------------(Recv m2)---[Ci,x+1]---
                            \                     /
                             \                   /
Process P2: ------------------(Recv m1)---(Send m2)----------------------

   * m1 sent after Ci,x.
   * m1 received by P2; m2 sent by P2 in same interval.
   * m2 received by P1 before next checkpoint, but causally after Ci,x.
   ===> FORMS A Z-CYCLE! Checkpoint Ci,x is permanently rendered useless!
```

---

#### 4. Communication-Induced Checkpointing (CIC) and the HMNR Protocol

Communication-Induced Checkpointing (CIC) allows nodes to take autonomous checkpoints, but piggybacks control metadata on application messages to detect potential $Z$-cycles and **force** checkpoints before $Z$-cycles close.

##### 4.1 HMNR (Hélary, Mostéfaoui, Netzer, Raynal) Algorithm
* **Clock Propagation:** Each process $P_i$ maintains a logical checkpoint sequence counter $c_i$.
* **Timestamping:** Every application message $m$ piggybacks the sender's current checkpoint index $m.c$.
* **Induction Rule:** When process $P_j$ receives a message $m$:
  $$\text{if } m.c > c_j \text{ then:}$$
  $$\quad \text{Take a FORCED CHECKPOINT before delivering } m$$
  $$\quad c_j \leftarrow m.c$$
* By enforcing monotonicity over checkpoint intervals, the HMNR protocol provably guarantees that no $Z$-cycles can form, completely eliminating the domino effect while avoiding coordinated synchronization locks.

---

#### 5. Message Logging Protocols Under the Piecewise Deterministic (PWD) Model

While checkpointing saves coarse-grained state snapshots, message logging records every individual message receipt to allow fine-grained deterministic re-execution.

##### 5.1 The Piecewise Deterministic (PWD) Assumption
* Execution inside any process is purely deterministic, driven by state and code.
* Non-deterministic events (NDEs) occur solely due to message arrival orders, async I/O, and external interrupts.
* **Determinant:** The unique parameters of an NDE, represented as $\langle \text{dest\_id}, \text{seq\_num}, \text{content\_hash} \rangle$.
* If all determinants are recorded, a failed process can recover to its exact pre-crash state by restoring its previous checkpoint and deterministically replaying all logged messages in order.

##### 5.2 Comparative Architectural Analysis

```
+-----------------------------------------------------------------------------------+
|                        MESSAGE LOGGING PARADIGMS                                  |
+-----------------------------------------------------------------------------------+
|                                                                                   |
|  [PESSIMISTIC LOGGING]        [OPTIMISTIC LOGGING]        [CAUSAL LOGGING]        |
|  - Sync write before send.    - Async write to buffer.    - Determinants in msgs. |
|  - No orphan processes.       - High speed, but creates   - Zero I/O penalty,     |
|  - Slow failure-free exec.      orphan processes on crash.- strictly local crash. |
|                                                                                   |
+-----------------------------------------------------------------------------------+
```

##### 5.3 Formal Evaluation Matrix

| Metric | Pessimistic Logging | Optimistic Logging | Causal Logging |
| :--- | :--- | :--- | :--- |
| **Logging Timing** | Synchronous (blocking disk flush before sending output). | Asynchronous (buffered in RAM, flushed periodically). | Asynchronous local logging; determinants piggybacked on msgs. |
| **Creation of Orphans** | **Strictly Impossible:** Unlogged states never generate output. | **Possible:** Surviving nodes may depend on unlogged crashed state. | **Strictly Impossible:** Dependents receive determinants before acting. |
| **Failure-Free Overhead** | High performance penalty due to frequent disk syncs. | Near zero runtime overhead during failure-free execution. | Zero disk sync latency; moderate network bandwidth overhead. |
| **Recovery Complexity** | **Local Recovery:** Only the crashed process restarts and replays. | **Global Cascade:** Orphan processes must be detected and rolled back. | **Local Recovery:** Surviving peers provide missing determinants from RAM. |
| **Garbage Collection** | Trivial; determinants pruned up to stable checkpoint. | Complex dependency tracking across distributed epochs. | Moderate; pruned via distributed vector timestamps. |
