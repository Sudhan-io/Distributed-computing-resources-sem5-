# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT I: INTRODUCTION & MODEL OF DISTRIBUTED COMPUTATIONS
### PART B: 8-MARK QUESTIONS AND ANSWERS (IN-DEPTH EXAM REPOSITORY)

---

## QUESTION 1: Contrast Message-Passing Systems with Shared Memory Systems in Detail with Architectural Diagrams and Trade-offs.

### 1. Conceptual Architectures
A fundamental distinction in computer system organization lies in whether processors share a common physical address space or coordinate strictly through an interconnection network.

```
+-----------------------------------------------------------------------------------------+
|                  SHARED MEMORY VS. MESSAGE PASSING ARCHITECTURES                        |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       SHARED MEMORY SYSTEM                              MESSAGE-PASSING SYSTEM          |
|   +--------+  +--------+  +--------+              +--------+  +--------+  +--------+    |
|   | CPU 1  |  | CPU 2  |  | CPU N  |              | Node 1 |  | Node 2 |  | Node N |    |
|   +---+----+  +---+----+  +---+----+              | CPU+RAM|  | CPU+RAM|  | CPU+RAM|    |
|       |           |           |                   +---+----+  +---+----+  +---+----+    |
|   ====+===========+===========+==== System Bus        |           |           |         |
|                   |                               +---+-----------+-----------+---+     |
|         +---------+---------+                     |    Interconnection Network    |     |
|         | Shared Memory RAM |                     |  (Ethernet, InfiniBand, WAN)  |     |
|         +-------------------+                     +-------------------------------+     |
+-----------------------------------------------------------------------------------------+
```

---

### 2. Deep Technical Comparison

| Technical Dimension | Shared Memory Systems (SMP / NUMA) | Message-Passing Systems (Clusters / Grids) |
| :--- | :--- | :--- |
| **Communication Mechanism** | Memory `Read` and `Write` instructions to common memory. | Explicit `Send()` and `Receive()` network packets. |
| **Hardware Scalability** | Low to Moderate (typically 16 to 128 cores before memory bus and cache-coherency saturation). | Massive (easily scales to thousands or millions of autonomous commodity nodes). |
| **Synchronization Primitives** | Semaphores, mutex locks, condition variables, atomic hardware primitives (CAS). | Message exchanges, tokens, logical clocks, barrier synchronizations. |
| **Data Integrity & Hazards**| Prone to data races, cache invalidation delays, memory leaks. | Memory isolated locally; hazards limited to network packet loss and communication deadlocks. |
| **Fault Isolation** | Poor: a memory corruption or kernel crash in one thread can panic the entire machine. | High: an individual node crash leaves surviving independent nodes functioning. |
| **Programming Model** | Threads (OpenMP, pthreads); easier to program small tasks. | Distributed processes (MPI, Socket programming, RPC/gRPC); requires explicit partitioning. |

---

## QUESTION 2: Discuss the Primitives for Distributed Communication (Blocking vs. Non-Blocking, Synchronous vs. Asynchronous Send/Receive).

### 1. The Communication Primitives Model
Distributed communication is expressed through two primary primitives:
* **`Send(Destination, Message)`**
* **`Receive(Source, MessageBuffer)`**

The operational semantics depend entirely on whether the operations are **Blocking** or **Non-Blocking**, and whether the interaction is **Synchronous** or **Asynchronous**.

```
+-----------------------------------------------------------------------------------------+
|                   DISTRIBUTED COMMUNICATION PRIMITIVE TAXONOMY                          |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|                 +-------------------------------------------------------+               |
|                 |          COMMUNICATION PRIMITIVE SEMANTICS            |               |
|                 +---------------------------+---------------------------+               |
|                                             |                                           |
|                     +-----------------------+-----------------------+                   |
|                     |                                               |                   |
|                     v                                               v                   |
|           SYNCHRONOUS COMMUNICATION                       ASYNCHRONOUS COMMUNICATION    |
|          - Sender and Receiver synchronize               - Sender resumes immediately   |
|            in time (Rendezvous).                           after copying to buffer.     |
|          - Send blocks until receive completes.          - No temporal coordination.    |
|                     |                                               |                   |
|          +----------+----------+                         +----------+----------+        |
|          |                     |                         |                     |        |
|          v                     v                         v                     v        |
|       BLOCKING              BLOCKING                  BLOCKING              NON-BLOCKING|
|       SEND                  RECEIVE                   SEND                  SEND        |
|    (Waits for ack)       (Waits for msg)          (Waits for buffer)     (Returns now)  |
+-----------------------------------------------------------------------------------------+
```

---

### 2. Detailed Semantic Analysis

1. **Blocking Send:**
   * Control returns to the sending process only after the message data has been safely copied out of the user buffer (either into the local kernel socket buffer, or until an acknowledgment arrives from the receiving host).
2. **Non-Blocking Send:**
   * Control returns to the sending application immediately, even before the message is transmitted. The application must not overwrite the send buffer until an asynchronous signal, callback, or polling status confirms completion.
3. **Blocking Receive:**
   * The receiving process halts execution and blocks until a matching message arrives in its incoming network queue and is unpacked into its memory space.
4. **Non-Blocking Receive:**
   * The receiving process checks for an arriving message. If available, it reads it; if no message is present, it returns an immediate status code without waiting, allowing the process to continue alternative computations.

---

## QUESTION 3: Differentiate Between Synchronous and Asynchronous Distributed System Execution Models.

### 1. Conceptual Framework
The operational boundaries of a distributed algorithm depend fundamentally on the timing guarantees provided by the underlying physical processors and communication channels.

```
+-----------------------------------------------------------------------------------------+
|                        SYNCHRONOUS VS. ASYNCHRONOUS EXECUTIONS                          |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       SYNCHRONOUS SYSTEM MODEL                          ASYNCHRONOUS SYSTEM MODEL       |
|                                                                                         |
|   Process 1 ---[Round 1]---[Round 2]--->            Process 1 ---------------------->   |
|                    |           |                                  \                     |
|                    v           v                                   \ Delay is arbitrary |
|   Process 2 ---[Round 1]---[Round 2]--->            Process 2 ------\--------------->   |
|             (Time is bounded by Delta)                               v (Unbounded)     |
+-----------------------------------------------------------------------------------------+
```

---

### 2. Comprehensive Comparison Matrix

| Property | Synchronous Execution Model | Asynchronous Execution Model |
| :--- | :--- | :--- |
| **Communication Delay Bound** | There exists a known, fixed upper bound $\Delta_{msg}$ on message transmission delay. | No upper bound on message delay; messages arrive after arbitrary but finite time. |
| **Processor Speed Bound** | Known bounds $\Delta_{proc}$ on the execution rate of processors. | Processors run at arbitrary, varying speeds; no bounds exist. |
| **Physical Clock Synchronization**| Physical clocks can be synchronized within a known maximum skew bound $\epsilon$. | No synchronized global physical clock is assumed. |
| **Failure Detection** | **Accurate:** A process that fails to respond within round duration $\Delta$ is definitively crashed. | **Unreliable:** A process cannot distinguish between a crashed peer and a slow communication link. |
| **Consensus Solvability** | Deterministically solvable in $f + 1$ rounds for $f$ crash failures. | **FLP Impossibility:** No deterministic consensus protocol can guarantee termination under crashes. |
| **Real-World Manifestation**| Hard real-time networks, synchronous avionics buses (Fly-by-wire). | The Internet, global cloud data center networks, distributed web services. |

---

## QUESTION 4: Explain the Key Design Issues and Challenges in Distributed Systems.

### 1. System Engineering Challenges
Building robust distributed systems requires addressing fundamental trade-offs across reliability, performance, and decentralization.

```
+-----------------------------------------------------------------------------------------+
|                      KEY DESIGN CHALLENGES IN DISTRIBUTED SYSTEMS                       |
+--------------------------+--------------------------------------------------------------+
| DESIGN ISSUE             | CORE CHALLENGE AND ARCHITECTURAL SOLUTION                    |
+--------------------------+--------------------------------------------------------------+
| 1. Transparency          | Hiding distributed mechanics from end-users and programmers. |
|                          | Dimensions: Access, Location, Migration, Replication, Failure|
|                          |                                                              |
| 2. Scalability           | Sustaining throughput as system grows along three axes:      |
|                          | - Numerical (more users/nodes), - Geographical (latency),    |
|                          | - Administrative (multiple autonomous domains).              |
|                          |                                                              |
| 3. Heterogeneity         | Bridging differences in hardware architectures (Endianness), |
|                          | operating systems, and network protocols using Middleware.   |
|                          |                                                              |
| 4. Concurrency &         | Managing concurrent shared resource access without a shared  |
|    Synchronization       | physical clock or shared memory (Logical Clocks, Mutex).     |
|                          |                                                              |
| 5. Fault Tolerance &     | Detecting and masking partial component failures to maintain |
|    High Availability     | continuous system service (Replication, Checkpointing).      |
|                          |                                                              |
| 6. Distributed Security  | Protecting open communication networks against snooping,     |
|                          | impersonation, and tampering (TLS, PKI, OAuth2).             |
+--------------------------+--------------------------------------------------------------+
```

---

### 2. Dimensions of Distribution Transparency (ISO/ODP Standard)
1. **Access Transparency:** Hides differences in data representation and resource invocation syntax.
2. **Location Transparency:** Hides where a resource is physically located in the network.
3. **Migration Transparency:** Hides that a resource can move dynamically from one host to another.
4. **Replication Transparency:** Hides that multiple identical copies of a resource exist across regions.
5. **Concurrency Transparency:** Hides that multiple competitive processes access shared resources concurrently.
6. **Failure Transparency:** Hides the failure and recovery of system components from the client.

---

## QUESTION 5: Describe the Model of Distributed Executions (Events, Local Histories, Global Histories, Partial Orders, Space-Time Diagrams).

### 1. Formal Foundations
A distributed program consists of $N$ asynchronous processes $P = \{P_1, P_2, \dots, P_N\}$.
* An execution of a process consists of a sequential execution of actions.
* An execution step is abstracted as an **Event** $e$.

---

### 2. Event Classification and Histories
1. **Event Types:**
   * **Internal Event:** Local state modification on a single node (e.g., local arithmetic computation).
   * **Send Event ($\text{send}(m)$):** Placing a message into an outgoing network channel.
   * **Receive Event ($\text{receive}(m)$):** Extracting a message from an incoming network channel.
2. **Local History ($h_i$):**
   The sequence of events executed by process $P_i$ ordered linearly by local occurrence:
   $$h_i = \langle e_i^1, e_i^2, e_i^3, \dots, e_i^k \rangle$$
3. **Global History ($H$):**
   The union of all local histories across all processes:
   $$H = \bigcup_{i=1}^N h_i$$

---

### 3. Lamport's Happened-Before Relation ($\prec$) and Space-Time Diagram

The causal precedence relation $\prec$ defines a strict **partial order** on $H$:
1. If $e_i^k$ and $e_i^m$ belong to the same process $P_i$, and $k < m$, then:
   $$e_i^k \prec e_i^m$$
2. If $e_i$ is a $\text{send}(m)$ event at $P_i$ and $e_j$ is the corresponding $\text{receive}(m)$ event at $P_j$, then:
   $$e_i \prec e_j$$
3. **Transitivity:** If $e_1 \prec e_2$ and $e_2 \prec e_3$, then $e_1 \prec e_3$.
4. **Concurrency ($\parallel$):** If $e_a \not\prec e_b$ and $e_b \not\prec e_a$, then events $e_a$ and $e_b$ are **concurrent**:
   $$e_a \parallel e_b$$

```
+-----------------------------------------------------------------------------------------+
|                               SPACE-TIME DIAGRAM REPRESENTATION                         |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       P1  ---(e1,1)------(e1,2)--------------------(e1,3)-------->                      |
|                     \                  \                                                |
|                      \ m1               \ m2                                            |
|                       v                  v                                              |
|       P2  -----------------(e2,1)--------------(e2,2)------------>                      |
|                                                                                         |
|       - Causal relation: e1,1 < e2,1 (due to message m1)                                |
|       - Concurrent events: e1,2 || e2,1 (no causal path connects them)                  |
+-----------------------------------------------------------------------------------------+
```

---

## QUESTION 6: Explain the Different Models of Communication Networks (FIFO, Non-FIFO, Causal Order) with Examples.

### 1. Network Communication Hierarchy
The delivery ordering guarantees provided by communication channels govern the complexity of distributed protocols.

```
+-----------------------------------------------------------------------------------------+
|                     COMMUNICATION NETWORK ORDERING HIERARCHY                            |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       +-----------------------------------------------------------------+               |
|       |                     CAUSAL ORDERING (CO) CHANNELS               | (Most Strict) |
|       |    If send(m1) < send(m2), then deliver(m1) < deliver(m2)       |               |
|       +--------------------------------+--------------------------------+               |
|                                        |                                                |
|                                        v                                                |
|       +-----------------------------------------------------------------+               |
|       |                       FIFO ORDERING CHANNELS                    |               |
|       |   Messages on the SAME channel arrive in sending order.         |               |
|       +--------------------------------+--------------------------------+               |
|                                        |                                                |
|                                        v                                                |
|       +-----------------------------------------------------------------+               |
|       |                     NON-FIFO ORDERING CHANNELS                  | (Least Strict)|
|       |   Messages can arrive in arbitrary out-of-order sequence.       |               |
|       +-----------------------------------------------------------------+               |
+-----------------------------------------------------------------------------------------+
```

---

### 2. Detailed Exploration and Diagrammatic Examples

1. **Non-FIFO Model:**
   * Packets take multiple routes across a mesh network; later messages can overtake earlier messages.
2. **FIFO Model:**
   * If process $P_i$ sends message $m_1$ before message $m_2$ to the **same** process $P_j$:
     $$\text{send}_i(m_1) \prec \text{send}_i(m_2) \implies \text{deliver}_j(m_1) \prec \text{deliver}_j(m_2)$$
3. **Causal Order (CO) Model:**
   * Generalizes FIFO across multiple channels. If the send event of $m_1$ causally precedes the send event of $m_2$, even across different senders:
     $$\text{send}(m_1) \prec \text{send}(m_2) \implies \text{deliver}(m_1) \prec \text{deliver}(m_2) \quad \forall \text{ common destinations}$$

```
VIOLATION OF CAUSAL ORDER IN A FIFO NETWORK:
P1 --- send(m1) to P2, P3 --------------------------------->
          \
           \ m1 (slow)
            v
P2 --------- receive(m1) --- send(m2) to P3 --------------->
                                \
                                 \ m2 (fast)
                                  v
P3 -------------------------- receive(m2) ---- receive(m1)->
   (P3 receives m2 BEFORE m1! Even though m1 causally caused m2!)
   ===> CAUSAL ORDER IS VIOLATED!
```

---

## QUESTION 7: Explain the Concept of Global State, Cuts, and Consistent Cuts in a Distributed System with a Neat Diagram.

### 1. The Global State Concept
The global state of a distributed system is an instantaneous snapshot of the system consisting of the local states of all processes and the states of all communication channels.
Because message transit times are non-zero and no global clock exists, defining a single "instantaneous" global state requires causal cut analysis.

---

### 2. Cuts, Consistent Cuts, and Inconsistent Cuts

* **Cut:** A dividing curve crossing the space-time diagram that partitions all system events into a **PAST** set and a **FUTURE** set.
* **Consistent Cut:** A cut is causally consistent if it is closed under the causal precedence relation:
  $$e \in \text{PAST} \land e' \prec e \implies e' \in \text{PAST}$$
  *(No effect can exist in the past without its cause also being in the past).*
* **Inconsistent Cut:** A cut that captures a message receipt in the past while its corresponding send event lies in the future.

```
+-----------------------------------------------------------------------------------------+
|                           CONSISTENT VS. INCONSISTENT CUTS                              |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       P1  ---(e1,1)---------|--------(e1,2)------------------------->                   |
|                    \        |                                                           |
|                     \ m1    | Consistent Cut C1                                         |
|                      v      |                                                           |
|       P2  ---------(e2,1)---|-----------------(e2,2)---------------->                   |
|                                                                                         |
|       ---------------------------------------------------------------                   |
|                                                                                         |
|       P1  ---(e1,1)-----------------|--------(e1,2)----------------->                   |
|                                     |           \                                       |
|                                     |            \ m2 (Orphan message!)                 |
|                                     |             v                                     |
|       P2  ---(e2,1)-----------------(e2,2)------|------------------->                   |
|                                                 | Inconsistent Cut C2                   |
|       In Cut C2: e2,2 is recorded as received, but send event e1,2 is in the FUTURE!    |
|       ===> AN INCONSISTENT CUT / ORPHAN MESSAGE ANOMALY!                                |
+-----------------------------------------------------------------------------------------+
```

---
