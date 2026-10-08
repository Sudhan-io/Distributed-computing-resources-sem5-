# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT III: DISTRIBUTED MUTUAL EXCLUSION AND DEADLOCK DETECTION
### PART A: 2-MARK QUESTIONS AND ANSWERS (EXAM-READY REPOSITORY)

---

### Q1. What is distributed mutual exclusion, and why is it important in distributed systems?
**Answer:**
* **Definition:** Distributed Mutual Exclusion is a concurrency control mechanism ensuring that concurrent processes across autonomous nodes without shared memory or a global physical clock access a shared critical resource (Critical Section - CS) in a mutually exclusive manner, so that at most one process executes in the CS at any given instant.
* **Importance:**
  1. **Data Consistency & Integrity:** Prevents race conditions, data corruption, and dirty updates on shared distributed databases, replicated objects, and physical peripherals.
  2. **Absence of Shared Memory:** Traditional operating system primitives (mutex locks, semaphores, test-and-set) rely on shared kernel memory, which is absent in networked clusters. Hence, message-passing protocols are mandatory.

---

### Q2. What is deadlock detection, and why is it necessary in distributed systems?
**Answer:**
* **Definition:** Deadlock detection is the run-time process of inspecting the distributed system's state to ascertain whether a set of processes are permanently blocked because each holds resources while waiting for resources held by others in the set.
* **Necessity:**
  1. Deadlock prevention and avoidance are overly restrictive and incur severe communication penalties in distributed networks, degrading throughput.
  2. Detection allows the system to operate with maximal resource concurrency and resolves deadlocks dynamically via preemption/rollback only when they actually manifest.

---

### Q3. Why is it important to detect deadlocks in distributed systems?
**Answer:**
* **Resource Starvation Avoidance:** Unresolved deadlocks permanently lock memory buffers, records, and CPU bandwidth, causing system-wide performance degradation.
* **Elimination of Phantom Deadlocks:** Detection algorithms differentiate genuine deadlocks from *phantom deadlocks* (false deadlocks detected due to message transit delays that resolved themselves), preventing unnecessary transaction aborts.
* **System High Availability:** Ensures long-running distributed transactions terminate correctly and maintain ACID properties.

---

### Q4. How does message passing contribute to deadlock situations in distributed systems?
**Answer:**
* **Communication Deadlocks:** Processes waiting to receive messages from processes that are simultaneously waiting for messages from them form cyclic wait dependencies purely through communication channels.
* **Unpredictable Latency & Out-of-Order Delivery:** Variable transmission delays cause requests and grants to cross in transit, leading to states where multiple nodes simultaneously believe resources are held by peers.
* **Lack of Instantaneous Global State:** No single node has an atomic global snapshot of all outstanding message buffers and queues.

---

### Q5. Explain the preliminaries required for understanding distributed mutual exclusion algorithms.
**Answer:**
1. **System Model:** $N$ processes ($P_1, P_2, \dots, P_N$) executing on independent sites connected by an error-free, logically fully connected network with finite, non-zero transmission delays.
2. **Process Execution Cycle:**
   $$	ext{Requesting CS} \longrightarrow 	ext{Executing CS} \longrightarrow 	ext{Releasing CS} \longrightarrow 	ext{Non-Critical Section}$$
3. **Correctness Criteria:**
   * **Safety:** At most one process executes in the CS at any time.
   * **Liveness:** Deadlock-free and starvation-free execution.
   * **Fairness:** Requests granted in logical timestamp order (FIFO logical time).

---

### Q6. What are the prerequisites for understanding deadlock detection algorithms?
**Answer:**
1. **Wait-For Graph (WFG):** A directed graph $G = (V, E)$ where vertices $V$ represent processes/transactions and directed edges $(P_i 	o P_j)$ denote that $P_i$ is waiting for $P_j$ to release a resource.
2. **Resource Request Models:**
   * **Single-Unit Model:** Out-degree $\le 1$; cycle indicates deadlock.
   * **AND Model:** Process waits for ALL requested resources; cycle $\iff$ deadlock.
   * **OR Model:** Process waits for ANY ONE resource; knot $\iff$ deadlock.
3. **Causal Ordering:** Logical timestamps to order wait-for notifications and prevent phantom deadlocks.

---

### Q7. Describe Lamport's algorithm for distributed mutual exclusion.
**Answer:**
* **Type:** Permission-based (non-token), timestamp-ordered algorithm.
* **Algorithm Steps:**
  1. **Request:** $P_i$ broadcasts $REQUEST(ts_i, i)$ to all $N-1$ sites and enqueues it into its local sorted `request_queue_i`. Each recipient enqueues the request and returns a timestamped $REPLY$.
  2. **Execute:** $P_i$ enters CS when:
     - Its request $(ts_i, i)$ is at the head of `request_queue_i`.
     - $P_i$ has received a message with timestamp $> (ts_i, i)$ from all other $N-1$ sites.
  3. **Release:** $P_i$ deletes its request from its queue and broadcasts $RELEASE(ts_i, i)$ to all sites.
* **Message Overhead:** $3(N-1)$ messages per CS entry ($N-1$ Requests, $N-1$ Replies, $N-1$ Releases).

---

### Q8. Discuss the key features of Ricart-Agrawala's algorithm for distributed mutual exclusion.
**Answer:**
* **Optimization over Lamport:** Eliminates explicit $RELEASE$ messages by deferring $REPLY$ messages.
* **Key Features:**
  1. **Selective/Deferred Replies:** A node $P_j$ receiving $REQUEST(ts_i, i)$ defers its $REPLY$ if $P_j$ is currently in CS or has a pending request with $(ts_j, j) < (ts_i, i)$; otherwise, it sends an immediate $REPLY$.
  2. **CS Entry Condition:** $P_i$ enters CS immediately upon receiving $N-1$ $REPLY$ messages.
  3. **Message Complexity:** Exactly $2(N-1)$ messages per CS entry ($N-1$ Requests + $N-1$ Replies).
  4. **Fully Symmetric:** No central master or specialized coordinator.

---

### Q9. Compare and contrast Lamport's and Ricart-Agrawala's algorithms for mutual exclusion.
**Answer:**

| Parameter | Lamport's Algorithm | Ricart-Agrawala's Algorithm |
| :--- | :--- | :--- |
| **Category** | Non-token based (Timestamp ordered) | Non-token based (Optimized permission) |
| **Messages per CS** | $3(N - 1)$ ($REQ, REP, REL$) | $2(N - 1)$ ($REQ, REP$) |
| **Release Action** | Broadcasts explicit $RELEASE$ to all $N-1$ peers | Sends deferred $REPLY$ messages to waiting nodes |
| **Data Structure** | Priority queue (`request_queue`) at every site | `reply_deferred[]` array of booleans |
| **Synchronization Delay** | $T$ (where $T$ = average message delay) | $T$ |
| **Fault Vulnerability** | Single point of failure: fails if any node crashes | Fails if any node crashes |

---

### Q10. Discuss the components of the system model used for deadlock detection.
**Answer:**
1. **Nodes/Sites:** Distributed physical hosts executing processes and managing local resources.
2. **Processes:** Execute, request resources, become blocked, and release resources.
3. **Transaction/Resource Managers:** Local modules responsible for granting resource locks and maintaining local sub-WFGs.
4. **Communication Subsystem:** Reliable message-passing infrastructure with arbitrary but finite message propagation latency.
5. **Global Wait-For Graph (WFG):** The distributed union of all local WFGs, tracking cross-site wait dependencies.

---

### Q11. What are the advantages of the Suzuki-Kasami broadcast algorithm for mutual exclusion?
**Answer:**
* **Low Message Complexity:** Requires between $0$ and $N$ messages per CS execution ($0$ messages if the requesting node already holds the idle token; at most $N-1$ $REQUEST$ messages + $1$ $TOKEN$ transfer otherwise).
* **Token Autonomy:** Token holder enters CS unconditionally without requiring permission or replies from other nodes.
* **Synchronization Delay:** Only $T$ (the time required to transfer the token directly from the last holder to the requester).
* **Outdated Request Filtering:** Sequence number arrays ($RN[j]$ and token $LN[j]$) ensure duplicate or stale requests are ignored.

---

### Q12. How does the Chandy-Misra-Haas algorithm differ in the OR model?
**Answer:**
* **AND Model:** A blocked process waits for *all* requested resources simultaneously. A probe is forwarded along all outgoing edges, and deadlock $\iff$ a directed cycle.
* **OR Model Differences:**
  1. A blocked process waits for *any one* resource to become available. Deadlock occurs only if there is a **knot** in the WFG.
  2. Processes use two-way messaging: forward probes `probe(initiator, sender, receiver)` and backward echos `echo(initiator, receiver, sender)`.
  3. A process only declares deadlock if all outgoing probe chains return echos without finding any unblocked, active node.

---

### Q13. Discuss the significance of message ordering in distributed deadlock detection.
**Answer:**
* **Prevents Phantom Deadlocks:** Out-of-order message arrival can cause a deadlock detector to process a resource request before a prior resource release, detecting a cycle that no longer exists in reality.
* **Causal Consistency:** Ensures that probe messages or snapshot slices reflect a valid, causally consistent cut across distributed sites.
* **Correct Termination:** Guarantees that genuine deadlocks are reliably reported within finite time and resolved without false alarms.

---
