# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT III: DISTRIBUTED MUTUAL EXCLUSION AND DEADLOCK DETECTION
### PART B: 8-MARK QUESTIONS AND ANSWERS (IN-DEPTH EXAM REPOSITORY)

---

## QUESTION 1: Outline Lamport's Algorithm with a Neat Example.

### 1. Introduction and Underlying Concept
Lamport's Distributed Mutual Exclusion algorithm (proposed by Leslie Lamport in 1978) is a classical **non-token-based (permission-based)** distributed mutual exclusion algorithm. It relies on **Lamport's scalar logical clocks** to establish a total ordering of all request events across geographically distributed processes without shared memory.

* **Key Objective:** Ensure mutually exclusive access to a Critical Section (CS) such that at most one site enters the CS at any instant, while guaranteeing starvation freedom and deadlock freedom.
* **Basic Philosophy:** Requests for the critical section are granted in the strict order of their logical timestamps.

---

### 2. System Model and Data Structures
1. **System Model:**
   * A distributed system comprising $N$ distinct sites/processes denoted as $S_1, S_2, \dots, S_N$.
   * Nodes communicate exclusively via message passing across reliable, FIFO communication channels. Message transit delays are unpredictable but finite.
2. **Local Data Structures at each Site $S_i$:**
   * `request_queue_i`: A local priority queue storing pending critical section requests. Entries are pairs of the form $(ts_k, k)$, where $ts_k$ is the logical timestamp of site $S_k$'s request and $k$ is the unique process identifier.
   * Total Ordering Relation ($\prec$):
     $$(ts_i, i) \prec (ts_j, j) \iff (ts_i < ts_j) \lor (ts_i = ts_j \land i < j)$$
   * `C_i`: Local scalar logical clock updated via Lamport's clock conditions.

---

### 3. Step-by-Step Algorithmic Protocol

```
+-----------------------------------------------------------------------------------------+
|                               LAMPORT'S ALGORITHM PHASES                                |
+------------------------------------+----------------------------------------------------+
|  PHASE 1: REQUEST                  |  PHASE 2: EXECUTION & RELEASE                      |
|                                    |                                                    |
|  Site Si wants CS:                 |  CS Execution Condition:                           |
|  1. Increments local clock: Ci++   |  1. (tsi, i) is at HEAD of request_queue_i         |
|  2. Enqueues (Ci, i) in queue_i    |  2. Received msgs with timestamp > (tsi, i)        |
|  3. Broadcasts REQUEST(Ci, i)      |     from ALL other N-1 sites                       |
|     to all other N-1 sites.        |                                                    |
|                                    |  CS Release:                                       |
|  Receiving Site Sj:                |  1. Site Si exits CS                               |
|  1. Updates Cj = max(Cj, Ci) + 1   |  2. Removes (tsi, i) from request_queue_i          |
|  2. Enqueues (Ci, i) in queue_j    |  3. Broadcasts RELEASE(Ci, i) to all N-1 sites     |
|  3. Sends REPLY(Cj, j) back to Si  |  Receiving sites delete (tsi, i) from local queues |
+------------------------------------+----------------------------------------------------+
```

#### Detailed Message Rules:
* **Rule 1 (Request Generation):**
  When site $S_i$ intends to enter the CS:
  $$C_i \leftarrow C_i + 1$$
  $S_i$ puts $(C_i, i)$ into `request_queue_i` and sends $REQUEST(C_i, i)$ to all other $N-1$ sites.
* **Rule 2 (Request Processing):**
  When site $S_j$ receives $REQUEST(C_i, i)$:
  $$C_j \leftarrow \max(C_j, C_i) + 1$$
  $S_j$ places $(C_i, i)$ into `request_queue_j` and immediately returns a timestamped $REPLY(C_j, j)$ to $S_i$.
* **Rule 3 (Critical Section Entry):**
  Site $S_i$ enters the CS when:
  1. $(C_i, i)$ is at the top/head of `request_queue_i` (lowest timestamp among all pending requests).
  2. $S_i$ has received a message (either $REPLY$, $REQUEST$, or $RELEASE$) with timestamp greater than $(C_i, i)$ from *every* other site in the system.
* **Rule 4 (Release Phase):**
  Upon finishing execution in the CS:
  $S_i$ removes its request $(C_i, i)$ from `request_queue_i` and broadcasts a timestamped $RELEASE(C_i, i)$ to all other $N-1$ sites.
  Upon receiving $RELEASE(C_i, i)$, each site $S_j$ deletes $(C_i, i)$ from `request_queue_j`.

---

### 4. Comprehensive Walkthrough Example with 3 Sites ($S_1, S_2, S_3$)

Consider 3 sites $S_1, S_2, S_3$ with initial clocks $C_1 = 0, C_2 = 0, C_3 = 0$.

```
      Site S1                       Site S2                       Site S3
         |                             |                             |
(Req CS) |                             |                             |
 C1=1    |--- REQUEST(1, 1) ---------->|                             |
 Queue1: |-----------------------------+---- REQUEST(1, 1) --------->|
[(1,1)]  |                             | (Enqueues (1,1))            | (Enqueues (1,1))
         |<-- REPLY(2, 2) -------------|                             |
         |                             | (Req CS) C2=3               |
         |                             | Queue2: [(1,1), (3,2)]      |
         |                             |--- REQUEST(3, 2) ---------->|
         |                             |<-- REPLY(4, 3) -------------|
         |<----------------------------+---- REPLY(3, 3) ------------|
         |                             |                             |
[S1 ENTERS CS!]                        |                             |
         |                             |                             |
(Exit CS)|                             |                             |
         |--- RELEASE(1, 1) ---------->| (Deletes (1,1))             |
         |-----------------------------+---- RELEASE(1, 1) --------->| (Deletes (1,1))
         |                             |                             |
         |                             | [S2 HEAD OF QUEUE &         |
         |                             |  ENTERS CS!]                |
```

1. **State 1:** $S_1$ requests CS at $C_1 = 1$. It sends $REQUEST(1, 1)$ to $S_2$ and $S_3$.
   * `request_queue_1` = `[(1, 1)]`
2. **State 2:** $S_2$ receives $REQUEST(1, 1)$, updates $C_2 = 2$, puts `(1, 1)` in `request_queue_2`, and sends $REPLY(2, 2)$ to $S_1$.
3. **State 3:** $S_3$ receives $REQUEST(1, 1)$, updates $C_3 = 2$, enqueues `(1, 1)`, and sends $REPLY(2, 3)$ to $S_1$.
4. **State 4:** $S_2$ concurrently decides to enter CS at local clock $C_2 = 3$. It sends $REQUEST(3, 2)$ to $S_1$ and $S_3$.
   * `request_queue_2` = `[(1, 1), (3, 2)]`
5. **State 5:** $S_1$ receives $REPLY(2, 2)$ from $S_2$ and $REPLY(2, 3)$ from $S_3$.
   * Both reply timestamps ($2 > 1$) are greater than $S_1$'s request timestamp.
   * `(1, 1)` is at the top of `request_queue_1`.
   * **$S_1$ successfully enters the Critical Section!**
6. **State 6:** $S_1$ exits CS, deletes `(1, 1)`, and broadcasts $RELEASE(1, 1)$ to $S_2$ and $S_3$.
7. **State 7:** $S_2$ receives $RELEASE(1, 1)$ and removes `(1, 1)`. Now `(3, 2)` becomes the head of `request_queue_2`. Since $S_2$ has replies from all peers, $S_2$ safely enters the CS.

---

### 5. Correctness Proof
* **Safety (Mutual Exclusion):** Suppose two sites $S_i$ and $S_j$ enter CS concurrently, with $(C_i, i) \prec (C_j, j)$. For $S_j$ to enter CS, its request must be at the head of `request_queue_j`, meaning $(C_j, j) \prec (C_i, i)$, or it has not yet received $S_i$'s request. However, due to FIFO channels and the condition that $S_j$ must receive a message with timestamp $> C_j$ from $S_i$, $S_i$'s request $(C_i, i)$ must have reached $S_j$ prior to $S_j$ receiving $S_i$'s reply. This creates a contradiction. Hence, safety holds.
* **Liveness (Deadlock & Starvation Freedom):** Total ordering ensures a strict global linear order. The request with the lowest timestamp is guaranteed to become the head of all queues, preventing deadlock and starvation.

---

### 6. Performance Metrics
* **Message Complexity:** Exactly $3(N - 1)$ messages per CS entry:
  * $N - 1$ Request messages
  * $N - 1$ Reply messages
  * $N - 1$ Release messages
* **Synchronization Delay:** Average message transmission delay $T$ (time between one site exiting and the next entering).
* **Fault Tolerance:** Low. A crash in any single site halts the entire system because all sites must wait for replies.

---

## QUESTION 2: Explain Ricart-Agrawala Algorithm with an Example.

### 1. Introduction and Architectural Motivation
The **Ricart-Agrawala algorithm** (developed by Glenn Ricart and Ashok Agrawala in 1981) is an optimization of Lamport's mutual exclusion algorithm. It belongs to the **permission-based (non-token)** category.

* **Primary Motivation:** In Lamport's algorithm, an explicit $RELEASE$ message must be broadcast to all $N-1$ nodes. Ricart and Agrawala recognized that this explicit release is redundant if permission is granted conditionally through **deferred replies**.
* **Key Innovation:** A node defers sending a $REPLY$ if it has higher priority for the critical section, effectively merging the permission and release mechanisms.

---

### 2. System Model and Data Structures
* $N$ independent sites communicating over reliable channels.
* Local state at each site $S_i$:
  * `our_sequence_number`: Timestamp of $S_i$'s pending request.
  * `highest_sequence_number`: Highest sequence number observed so far across received requests.
  * `requesting_critical_section`: Boolean flag indicating whether $S_i$ wants to enter CS.
  * `reply_deferred[]`: Boolean array of size $N$, where `reply_deferred[j] = true` denotes that $S_i$ has received a request from $S_j$ but has deferred sending a reply.

---

### 3. Step-by-Step Algorithmic Protocol

```
+-----------------------------------------------------------------------------------------+
|                              RICART-AGRAWALA PROTOCOL                                   |
+------------------------------------+----------------------------------------------------+
|  REQUESTING CS (Site Si):          |  PROCESSING REQUEST AT SITE Sj:                    |
|  1. requesting_critical_section = T|  Receives REQUEST(k, i) from Si:                   |
|  2. our_seq_num = highest_seq + 1  |  1. highest_seq = max(highest_seq, k)             |
|  3. Broadcast REQUEST(our_seq, i)  |  2. defer = requesting_CS AND                      |
|     to all other N-1 sites         |        ((k > our_seq) OR (k == our_seq AND i > j)) |
|  4. Wait for N-1 REPLY messages    |  3. IF defer == TRUE:                              |
|                                    |        reply_deferred[i] = TRUE                    |
|  CS ENTRY & EXIT:                  |     ELSE:                                          |
|  1. Enter CS when replies == N-1   |        Send REPLY immediately to Si                |
|  2. On Exit:                       |                                                    |
|     requesting_critical_section = F|                                                    |
|     For each k where deferred[k]:  |                                                    |
|        Send REPLY to Sk            |                                                    |
|        reply_deferred[k] = FALSE   |                                                    |
+------------------------------------+----------------------------------------------------+
```

---

### 4. Comprehensive Walkthrough Example with 3 Sites ($S_1, S_2, S_3$)

Let sites $S_1$ and $S_2$ request the CS concurrently, while site $S_3$ is idle.

```
       Site S1 (Req: ts=2)             Site S2 (Req: ts=4)             Site S3 (Idle)
               |                               |                              |
 (Sends REQ)   |------- REQUEST(2, 1) -------->|                              |
               |-------------------------------+------- REQUEST(2, 1) ------->|
               |                               | (Sends REQ)                  | (Replies to S1)
               |<------ REQUEST(4, 2) ---------|                              |<-- REPLY -----
               |                               |------- REQUEST(4, 2) ------->|
               |                               |                              | (Replies to S2)
 (S1 checks    |                               |                              |--- REPLY ---->
  ts: 2 < 4 -> |                               |                              |
  DEFERS reply)|                               |                              |
               |                               | (S2 checks ts: 4 > 2 ->      |
               |                               |  S2 sends REPLY to S1!)      |
               |<------ REPLY -----------------|                              |
               |                               |                              |
 [S1 has 2     |                               |                              |
  REPLIES ->   |                               |                              |
  ENTERS CS!]  |                               |                              |
       :       |                               |                              |
 (S1 exits CS: |                               |                              |
  Sends def.   |------- REPLY ---------------->|                              |
  REPLY to S2) |                               |                              |
               |                               | [S2 has 2 REPLIES ->         |
               |                               |  ENTERS CS!]                 |
```

1. **Step 1:** $S_1$ requests CS with timestamp $(2, 1)$. It broadcasts $REQUEST(2, 1)$ to $S_2$ and $S_3$.
2. **Step 2:** Concurrently, $S_2$ requests CS with timestamp $(4, 2)$. It broadcasts $REQUEST(4, 2)$ to $S_1$ and $S_3$.
3. **Step 3 (Site $S_3$):** $S_3$ is not requesting CS. It immediately sends $REPLY$ to both $S_1$ and $S_2$.
4. **Step 4 (Site $S_1$ receives $REQUEST(4, 2)$):**
   * $S_1$ is requesting CS with timestamp $(2, 1)$.
   * Since $(2, 1) \prec (4, 2)$, $S_1$ has higher priority.
   * $S_1$ sets `reply_deferred[2] = true` and **defers** its reply.
5. **Step 5 (Site $S_2$ receives $REQUEST(2, 1)$):**
   * $S_2$ is requesting CS with timestamp $(4, 2)$.
   * Since $(2, 1) \prec (4, 2)$, $S_2$ has lower priority.
   * $S_2$ immediately sends $REPLY$ to $S_1$.
6. **Step 6 (CS Entry):** $S_1$ has received replies from both $S_3$ and $S_2$ ($N-1 = 2$ replies). $S_1$ enters the Critical Section!
7. **Step 7 (CS Exit):** $S_1$ exits the CS, sets `requesting_critical_section = false`, and transmits the deferred $REPLY$ to $S_2$.
8. **Step 8:** $S_2$ now receives its second reply and enters the Critical Section.

---

### 5. Performance Evaluation and Comparison
* **Message Complexity:** Exactly **$2(N - 1)$** messages per CS entry ($N-1$ Requests + $N-1$ Replies). This represents a **33% reduction** in message overhead compared to Lamport's $3(N-1)$.
* **Synchronization Delay:** $T$ (the time required for a deferred reply to travel from the exiting node to the next requesting node).
* **Failure Sensitivity:** Fails completely if any node crashes. A single dead node will never send a reply, permanently blocking all other nodes.

---

## QUESTION 3: Justify the Drawbacks of Suzuki–Kasami Algorithm.

### 1. Overview of Suzuki–Kasami Algorithm
The **Suzuki–Kasami algorithm** (1985) is a token-based distributed mutual exclusion algorithm. A unique privilege message called the **Token** circulates among nodes. Only the site holding the token is authorized to execute in the Critical Section.

* **Data Structures:**
  * At site $S_i$: Array $RN_i[1..N]$ where $RN_i[j]$ records the largest sequence number received from site $S_j$.
  * Within Token:
    * Vector $LN[1..N]$, where $LN[j]$ denotes the sequence number of the most recent request granted to site $S_j$.
    * Token Queue $Q$: FIFO queue of requesting sites waiting for the token.

---

### 2. Core Drawbacks and Justification

```
+-----------------------------------------------------------------------------------------+
|                         DRAWBACKS OF SUZUKI-KASAMI ALGORITHM                            |
+-----------------------------------------------------------------------------------------+
|  1. BROADCAST STORM:                                                                    |
|     Every request requires broadcasting REQUEST(j, sn) to ALL N-1 sites,                |
|     leading to high network load regardless of system state.                            |
|                                                                                         |
|  2. LOSS OF THE TOKEN (CRITICAL SINGLE POINT OF FAILURE):                              |
|     If the site holding the token crashes, the token is permanently destroyed.          |
|     Complex election & regeneration algorithms are required.                           |
|                                                                                         |
|  3. NODE FAILURE SENSITIVITY:                                                           |
|     Token cannot be passed to a node that has crashed while waiting in Queue Q,         |
|     causing delivery hangs or message timeouts.                                         |
|                                                                                         |
|  4. STATIC TOPOLOGY ASSUMPTION:                                                         |
|     All nodes must know the exact total number of sites N. Joining or leaving           |
|     requires resizing global arrays RN and LN dynamically.                              |
+-----------------------------------------------------------------------------------------+
```

#### Detailed Justifications:

1. **High Overhead of Global Broadcasts ($O(N)$ Request Broadcasts):**
   * Even though token transfer is unicast ($1$ message), requesting the token requires broadcasting to all $N-1$ sites. When $N$ is large (e.g., thousands of cloud instances), $N-1$ messages generate network congestion and CPU interrupts on idle sites.
2. **Catastrophic Failure on Token Loss:**
   * The token is a physical state construct. If the site currently holding the token crashes (or if the network drops the token message during transit), **mutual exclusion halts entirely**.
   * Detecting token loss is non-trivial: a site cannot easily distinguish between a very slow token holder and a dead token holder without complex, failure-prone timeout-based election algorithms.
3. **Queue Head Invalidation upon Process Crash:**
   * If site $S_k$ places itself in the token queue $Q$ and subsequently crashes before receiving the token, the current holder will eventually transmit the token to the dead site $S_k$. This stalls the critical section until timeout and recovery routines intervene.
4. **Storage Overhead on Message Size:**
   * Unlike lightweight permission messages, the token message carries the entire array $LN[1..N]$ and queue $Q$. The size of the token message scales linearly with $N$, creating overhead on low-bandwidth networks.
5. **No Dynamicity (Rigid Group Membership):**
   * Array dimensions for $RN$ and $LN$ are fixed to $N$. Adding or removing nodes requires stopping the algorithm, synchronizing global state, and reinitializing arrays.

---

## QUESTION 4: Discuss with a Suitable Example to Show That a Deadlock Cannot Occur If Any One of the Four Coffman Conditions Is Absent.

### 1. Introduction: Coffman's Four Conditions
In 1971, Edward G. Coffman Jr. proved that a system deadlock can arise **if and only if** all four of the following conditions hold simultaneously:
1. **Mutual Exclusion:** Resources cannot be shared; only one process holds a resource at a time.
2. **Hold and Wait:** A process currently holding at least one resource is actively waiting to acquire additional resources held by other processes.
3. **No Preemption:** Resources cannot be forcibly confiscated from a process; they can only be released voluntarily after task completion.
4. **Circular Wait:** A closed chain of processes exists such that each process holds a resource needed by the next process in the chain.

---

### 2. Demonstration: Absence of Each Condition Prevents Deadlock

```
+-----------------------------------------------------------------------------------------+
|                             ELIMINATING COFFMAN CONDITIONS                              |
+--------------------------+--------------------------------------------------------------+
| CONDITION NEGATED        | IMPACT AND PREVENTION MECHANISM                              |
+--------------------------+--------------------------------------------------------------+
| 1. No Mutual Exclusion   | Resources are sharable (e.g., read-only files). Multiple    |
|                          | processes access simultaneously. NO BLOCKING OCCURS.         |
|                          |                                                              |
| 2. No Hold and Wait      | Conservative Protocol: Process requests ALL resources at     |
|                          | once. If any unavailable, it holds NOTHING. NO CYCLES.       |
|                          |                                                              |
| 3. Preemption Allowed    | If process P holding R1 is denied R2, R1 is PREEMPTED        |
|                          | and given to others. CYCLE IS BROKEN IMMEDIATELY.            |
|                          |                                                              |
| 4. No Circular Wait      | Global Resource Ordering: Resources indexed 1..M. Processes  |
|                          | must request in strictly increasing order. NO CYCLES EXIST.  |
+--------------------------+--------------------------------------------------------------+
```

#### Scenario Setup:
Consider two processes $P_1, P_2$ and two resources $R_1, R_2$.
* $P_1$ holds $R_1$ and requests $R_2$.
* $P_2$ holds $R_2$ and requests $R_1$.

```
     [P1] ---- Holds ----> (R1) <---- Requests ---- [P2]
       |                                              |
    Requests                                        Holds
       |                                              |
       v                                              v
      (R2) <-------------------------------------------
```

#### Detailed Proof of Deadlock Prevention:

1. **Eliminating Condition 1 (Allowing Shared Access):**
   * If $R_1$ and $R_2$ are read-only database records or reentrant code routines, both $P_1$ and $P_2$ access $R_1$ and $R_2$ concurrently without waiting.
   * **Result:** No process is blocked. Deadlock is impossible.
2. **Eliminating Condition 2 (Eliminating Hold-and-Wait):**
   * Protocol rule: A process must request all required resources simultaneously upfront.
   * If $P_1$ requests $\{R_1, R_2\}$ and only $R_1$ is available, $P_1$ is granted *neither*. $P_1$ holds 0 resources while waiting.
   * Consequently, $P_2$ can acquire both $R_1$ and $R_2$, finish execution, and release them.
   * **Result:** No process ever holds a resource while waiting for another. Deadlock cannot occur.
3. **Eliminating Condition 3 (Enforcing Preemption):**
   * Protocol rule: If $P_1$ holds $R_1$ and requests $R_2$ (held by $P_2$), the resource manager preempts $R_1$ from $P_1$ and allocates it to $P_2$.
   * $P_2$ completes its execution with both resources and releases them, allowing $P_1$ to restart safely.
   * **Result:** The hold chain is severed. Deadlock is resolved dynamically.
4. **Eliminating Condition 4 (Preventing Circular Wait via Linear Ordering):**
   * Protocol rule: Let all resources be assigned unique global integer IDs: $F(R_1) = 1, F(R_2) = 2$. Processes must request resources in strictly ascending numerical order.
   * To hold both resources, both $P_1$ and $P_2$ must request $R_1$ first, then $R_2$.
   * If $P_1$ acquires $R_1$, $P_2$ is blocked waiting for $R_1$ *before* it can ever request $R_2$. $P_1$ proceeds to acquire $R_2$ without contention, finishes, and releases both.
   * **Result:** Mathematical cycles cannot form in an acyclic topological order. Deadlock is strictly prevented.

---

## QUESTION 5: Demonstrate the Chandy-Misra-Haas Detection Algorithm.

### 1. Introduction
The **Chandy-Misra-Haas (CMH) algorithm** (1983) is an **edge-chasing distributed deadlock detection algorithm**. Instead of gathering the complete Wait-For Graph (WFG) at a central site, it sends small special diagnostic messages called **probes** along the directed edges of the distributed WFG.

---

### 2. Chandy-Misra-Haas Algorithm for the AND Model

In the **AND model**, a process requests a set of resources and remains blocked until **all** requested resources are acquired. A deadlock corresponds to a directed cycle in the WFG.

#### Probe Structure:
A probe is a 3-tuple:
$$\text{probe}(i, j, k)$$
* $i$: Initiator of the deadlock detection computation.
* $j$: Sender of the probe message.
* $k$: Receiver of the probe message.

```
+-----------------------------------------------------------------------------------------+
|                             CMH ALGORITHM LOGIC (AND MODEL)                             |
+-----------------------------------------------------------------------------------------+
|  1. INITIATION:                                                                         |
|     When process Pi blocks on resources held by dependent processes:                    |
|     For each dependent process Pk:                                                      |
|         Send probe(i, i, k)                                                             |
|                                                                                         |
|  2. PROPAGATION:                                                                        |
|     When Pk receives probe(i, j, k):                                                    |
|     IF Pk is locally blocked AND has not forwarded probe(i, *, *) before:               |
|         For each process Pm that Pk is waiting for:                                     |
|             Send probe(i, k, m)                                                         |
|                                                                                         |
|  3. DEADLOCK DECLARATION:                                                               |
|     If the initiator Pi receives its OWN probe: probe(i, *, i)                          |
|     ===> A DIRECTED CYCLE EXISTS! DEADLOCK IS DETECTED!                                 |
+-----------------------------------------------------------------------------------------+
```

#### Step-by-Step Trace Example:

```
    Site 1                      Site 2                      Site 3
    [ P1 ] --- waits for --->   [ P2 ] --- waits for --->   [ P3 ]
      ^                                                       |
      |___________________ waits for (cross-site) ____________|
```

1. **Step 1:** $P_1$ blocks waiting for $P_2$. $P_1$ initiates detection: sends `probe(1, 1, 2)`.
2. **Step 2:** $P_2$ receives `probe(1, 1, 2)`. $P_2$ is blocked waiting for $P_3$. $P_2$ forwards `probe(1, 2, 3)`.
3. **Step 3:** $P_3$ receives `probe(1, 2, 3)`. $P_3$ is blocked waiting for $P_1$. $P_3$ forwards `probe(1, 3, 1)`.
4. **Step 4:** $P_1$ receives `probe(1, 3, 1)`. Since the initiator ID matches the receiving process ($1 == 1$), $P_1$ concludes it belongs to a cycle. **Deadlock detected!**

---

### 3. Chandy-Misra-Haas Algorithm for the OR Model
In the **OR model**, a blocked process can proceed if **any one** of its requested resources is granted. A deadlock occurs if and only if the blocked process is part of a **knot** (a set of nodes from which every reachable node can reach all other nodes in the set, with no paths to active nodes).

#### Differences from AND Model:
1. Probes alone are insufficient because a cycle does not imply a knot.
2. Uses a two-phase protocol:
   * **Forward Probes:** `probe(i, j, k)` traverses all reachable paths.
   * **Backward Echos:** `echo(i, k, j)` returns confirmations along the reverse path.
3. Each node maintains a count of outstanding replies:
   $$\text{num}(i) = \text{number of dependent outgoing edges}$$
4. A node sends an `echo` to its predecessor only when it receives `echo` messages from **all** its downstream dependencies.
5. If initiator $P_i$ receives `echo` messages for all initial probes sent, no path leads to an active process; hence, $P_i$ is trapped in a knot and deadlocked.

---

## QUESTION 6: Explain the "Snapshot" Algorithm of Lamport (Chandy-Lamport Distributed Snapshot).

### 1. Motivation and Problem Definition
The **Chandy-Lamport Snapshot Algorithm** (1985) captures a **consistent global state** (a snapshot) of an asynchronous distributed system without freezing or halting ongoing process execution.

* **Components of Global State:**
  1. The local state of every process $P_i$.
  2. The state of all incoming communication channels $C_{ij}$ (messages that were sent before the snapshot cut but received after it).
* **Consistent Cut Definition:** If a message receipt event is recorded in the snapshot, the corresponding message send event **must also** be recorded in the snapshot:
  $$\text{receive}(m) \in \text{Cut} \implies \text{send}(m) \in \text{Cut}$$

---

### 2. System Assumptions
1. Channels are unidirectional, error-free, and **strictly FIFO**.
2. Network graph is strongly connected.
3. Message delay is arbitrary but finite.
4. Processes can initiate a snapshot independently and concurrently.

---

### 3. Marker Rules Protocol

```
+-----------------------------------------------------------------------------------------+
|                             CHANDY-LAMPORT MARKER RULES                                 |
+-----------------------------------------------------------------------------------------+
|  MARKER SENDING RULE (Site Pi):                                                         |
|  1. Pi records its own local state.                                                     |
|  2. Pi sends one MARKER message along each outgoing channel Cij before sending any      |
|     subsequent regular messages.                                                        |
|                                                                                         |
|  MARKER RECEIVING RULE (Site Pj receiving MARKER on channel Cij):                       |
|                                                                                         |
|  CASE A: Pj has NOT yet recorded its state:                                             |
|     - Pj records its own local state.                                                   |
|     - Pj records channel Cij as EMPTY.                                                  |
|     - Pj executes the Marker Sending Rule (sends MARKER along all outgoing channels).  |
|     - Pj begins recording incoming messages on all other incoming channels C_kj (k != i)|
|                                                                                         |
|  CASE B: Pj has ALREADY recorded its state:                                             |
|     - Pj stops recording incoming messages on channel Cij.                              |
|     - The recorded state of channel Cij is the sequence of all messages received        |
|       between when Pj took its snapshot and when this MARKER arrived on Cij.            |
+-----------------------------------------------------------------------------------------+
```

---

### 4. Illustrative Example: Distributed Banking System

Consider two bank branches $P_1$ and $P_2$.
* $P_1$ initial balance: \$1000
* $P_2$ initial balance: \$500
* Total money in system = \$1500

```
  Process P1 ($1000)                               Process P2 ($500)
       |                                                |
       |--- Sends $200 transfer to P2 ----------------->|
       |    (Balance becomes $800)                      |
       |                                                |
 [TAKES SNAPSHOT: P1 = $800]                            |
       |                                                |
       |--- Sends MARKER to P2 ------------------------>|
       |                                                |
       |                                     [P2 receives $200: bal=$700]
       |                                                |
       |                                     [P2 receives MARKER]
       |                                     - P2 records state: $700
       |                                     - Records channel (P1->P2): EMPTY
```

* **Resulting Global Snapshot:**
  * $State(P_1) = \$800$
  * $State(P_2) = \$700$
  * $State(Channel) = \emptyset$
  * **Total Recorded = \$1500 (Consistent Snapshot preserved!)**

---

## QUESTION 7: What Is a Deadlock? How Can a Deadlock Be Recovered? Explain Distributed Deadlocks.

### 1. Definition of Deadlock and Distributed Deadlock
* **Deadlock Definition:** A system state where a set of processes are permanently blocked because each process holds resources and waits for other resources held by other processes in the set.
* **Distributed Deadlock Characteristics:**
  * Resources and processes are dispersed across geographically separated autonomous sites.
  * Wait dependencies span network boundaries (cross-site wait-for edges).
  * No single physical node has instantaneous, consistent global visibility of lock tables.
  * Prone to **phantom deadlocks** caused by communication delays.

---

### 2. Distributed Deadlock Recovery Mechanisms

```
+-----------------------------------------------------------------------------------------+
|                       DISTRIBUTED DEADLOCK RECOVERY TECHNIQUES                          |
+-----------------------------------------------------------------------------------------+
|  1. VICTIM SELECTION:                                                                   |
|     Choose one or more deadlocked transactions to abort based on:                       |
|     - Lowest CPU computation time used so far.                                          |
|     - Fewest resources held.                                                            |
|     - Highest transaction priority / youngest timestamp.                                |
|                                                                                         |
|  2. TRANSACTION ROLLBACK:                                                               |
|     - Partial Rollback: Rollback victim to an intermediate savepoint where deadlock is  |
|       broken.                                                                           |
|     - Total Rollback: Abort transaction completely, release all locks, restart task.    |
|                                                                                         |
|  3. RESOURCE PREEMPTION:                                                                |
|     Forcibly seize locks from the victim, assign to waiting processes, and suspend the  |
|     victim until resources are free.                                                    |
|                                                                                         |
|  4. STARVATION PREVENTION:                                                              |
|     Ensure the same transaction is not repeatedly chosen as victim by incrementing      |
|     priority with each rollback.                                                        |
+-----------------------------------------------------------------------------------------+
```

---
