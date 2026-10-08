# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT IV: CONSENSUS AND RECOVERY
### PART A: 2-MARK QUESTIONS AND ANSWERS (EXAM-READY REPOSITORY)

---

### Q1. Define consensus and its importance in distributed systems.
**Answer:**
* **Definition:** The consensus problem requires a set of distributed processes, each starting with an initial private value, to agree on a single common decision value such that:
  1. **Agreement:** All non-faulty processes decide on the same value.
  2. **Validity:** If all non-faulty processes propose the same value $v$, they must decide $v$.
  3. **Termination:** Every non-faulty process eventually reaches a decision.
* **Importance:** Fundamental building block for distributed transactions (atomic commit protocols like 2PC/3PC), state machine replication (Paxos, Raft), leader election, and mutual exclusion.

---

### Q2. Recall the key components of Lamport's algorithm for mutual exclusion.
**Answer:**
1. **Scalar Logical Clock ($C_i$):** Assigns a monotonically increasing timestamp to each critical section request.
2. **Local Priority Queue (`request_queue_i`):** Stores requests ordered by timestamp $(ts_i, i)$ using a total order tie-breaker.
3. **Three-Phase Messaging:**
   * $REQUEST(ts, i)$ broadcast to all $N-1$ sites.
   * $REPLY$ returned by all sites.
   * $RELEASE$ broadcast upon exiting the CS to prune queue heads.

---

### Q3. Can you remember the basic steps of the Chandy-Misra-Haas algorithm for deadlock detection?
**Answer:**
1. **Initiation:** When process $P_i$ blocks waiting for a set of processes, it generates and sends diagnostic probe messages $\text{probe}(i, i, k)$ to all processes $P_k$ it is waiting for.
2. **Propagation:** When blocked process $P_k$ receives $\text{probe}(i, j, k)$, it forwards $\text{probe}(i, k, m)$ along all its outgoing wait-for edges to processes $P_m$.
3. **Detection:** If initiator $P_i$ receives $\text{probe}(i, *, i)$ (its own probe), a directed cycle exists in the distributed Wait-For Graph, confirming a deadlock.

---

### Q4. List the different models of deadlocks in distributed systems.
**Answer:**
1. **Single-Resource Model:** A process requests at most one resource at a time (out-degree $\le 1$; cycle indicates deadlock).
2. **AND Model:** A process waits for ALL requested resources simultaneously (deadlock $\iff$ directed cycle).
3. **OR Model:** A process waits for ANY ONE of the requested resources (deadlock $\iff$ knot).
4. **$C$-out-of-$K$ Model:** A process requests $K$ resources and needs any $C$ of them to proceed.

---

### Q5. Explain why consensus is difficult to achieve in distributed systems.
**Answer:**
* **Asynchronous Unpredictability:** In asynchronous distributed networks, there are no upper bounds on message transit delays or relative processor execution speeds.
* **FLP Impossibility Theorem (Fischer, Lynch, Paterson - 1985):** Proves that in a purely asynchronous system, no deterministic consensus protocol can guarantee termination in the presence of even a **single unannounced fail-stop process failure**.
* **Distinguishing Crash vs. Delay:** A process cannot distinguish whether a peer has crashed or is simply experiencing network congestion.

---

### Q6. Can you describe the process of coordinated checkpointing for failure recovery?
**Answer:**
* **Definition:** Processes coordinate their checkpointing actions by exchanging control messages to produce a **globally consistent recovery line** free of orphan messages.
* **Two-Phase Protocol (e.g., Koo-Toueg):**
  1. **Phase 1 (Tentative Checkpoint):** Initiator sends a checkpoint request. Participating processes take a *tentative checkpoint* and return an acknowledgment.
  2. **Phase 2 (Permanent Checkpoint):** If all processes succeed, initiator broadcasts a commit message converting tentative checkpoints into *permanent checkpoints* and discarding older ones.

---

### Q7. What issues arise in failure recovery in distributed systems?
**Answer:**
1. **Orphan Messages:** A message whose receive event is recorded in a process's checkpoint, but whose corresponding send event is rolled back.
2. **Lost Messages:** A message whose send event is recorded before the rollback, but whose receive event is undone, leaving it lost in transit.
3. **Domino Effect:** Cascading rollbacks caused by uncoordinated checkpoints, forcing processes back to their initial startup state.
4. **Livelock:** Endless rollback cycles caused by processes triggering recovery loops on mutual dependencies.

---

### Q8. What distinguishes synchronous systems with failures from synchronous failure-free systems?
**Answer:**
* **Synchronous Failure-Free Systems:** Known bounds on communication delay ($T$) and processing speed; consensus is trivial in $1$ round because all messages arrive reliably within round boundaries.
* **Synchronous Systems with Failures:** Messages may be lost due to process crashes or Byzantine malicious corruption.
  * In Crash Failure models: Requires at least **$f + 1$ rounds** to reach consensus, where $f$ is the maximum number of failed processes.
  * In Byzantine Failure models: Requires $N \ge 3m + 1$ total nodes to tolerate $m$ faulty nodes.

---

### Q9. Apply Lamport's algorithm to solve mutual exclusion in a distributed system scenario.
**Answer:**
* **Scenario:** Site $S_1$ and $S_2$ request access to a shared replicated log simultaneously.
* **Execution:**
  1. $S_1$ broadcasts $REQUEST(ts=1, id=1)$; $S_2$ broadcasts $REQUEST(ts=1, id=2)$.
  2. Both insert both requests into their local queues.
  3. Tie-breaker rule evaluates: $(1, 1) \prec (1, 2)$ because process ID $1 < 2$.
  4. Both sites recognize $S_1$'s request as the head of the queue.
  5. Upon collecting replies, $S_1$ enters the CS first, releases it, and then $S_2$ enters safely.

---

### Q10. Design an algorithm for asynchronous checkpointing and recovery in distributed systems.
**Answer:**
* **Algorithm (Juang-Venkatesan Approach):**
  1. **Autonomous Checkpoints:** Each process takes checkpoints independently without coordinating with peers.
  2. **Direct Dependency Tracking:** Every message sent piggybacks the sender's current checkpoint index; the receiver records dependencies in an index vector.
  3. **Recovery Algorithm:** When a node crashes, it broadcasts a rollback request containing its rolled-back checkpoint index. Dependent nodes iteratively compute the maximum consistent recovery line using vector dependencies.

---

### Q11. Apply checkpoint-based recovery to restore a distributed system after a failure.
**Answer:**
1. **Identify the Recovery Line:** Find the most recent set of consistent checkpoints across all processes that contains no orphan messages ($\text{receive}(m) \in \text{State} \implies \text{send}(m) \in \text{State}$).
2. **Rollback:** Restore the local state of each affected process to its checkpoint on the recovery line.
3. **Message Replay (if logging exists):** Replay logged deterministic messages from stable storage up to the point of failure to restore the pre-crash state without losing transaction progress.

---

### Q12. Analyze the challenges involved in achieving consensus in distributed systems.
**Answer:**
1. **Network Partitioning:** Network splits prevent nodes from communicating across partitions (split-brain syndrome).
2. **Arbitrary / Byzantine Behaviors:** Faulty nodes may send contradictory messages to different peers (e.g., telling Node A "Yes" and Node B "No").
3. **Impossibility Bounds:** The FLP impossibility theorem proves that asynchronous systems cannot guarantee deterministic termination under crashes.
4. **Message Complexity:** Achieving Byzantine agreement requires exponential message exchanges ($O(N^{m+1})$) or digital signature infrastructure.

---
