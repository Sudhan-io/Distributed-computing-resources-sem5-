# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT I: INTRODUCTION & MODEL OF DISTRIBUTED COMPUTATIONS
### PART C: 16-MARK ESSAY QUESTIONS (EXHAUSTIVE 5-6 PAGE EXAMINATION ANSWERS)

---

# QUESTION 1
> ### **Discuss in detail the definition, motivation, and relation of distributed systems to computer system components, and elaborate on the major design issues and challenges in distributed systems.**

---

### 1. FORMAL DEFINITION & SYSTEM CHARACTERISTICS
A **Distributed System** is formally defined as a collection of autonomous computing nodes that communicate over a network via message passing, sharing no physical memory and no global physical clock, yet coordinating to appear to end-users as a single unified, coherent computing environment.

```
+---------------------------------------------------------------------------------------------------------+
|                                    DISTRIBUTED SYSTEM TOPOLOGY                                          |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       +-------------------+       +-------------------+       +-------------------+                     |
|       | Computer Node 1   |       | Computer Node 2   |       | Computer Node N   |                     |
|       | [CPU] [Local RAM] |       | [CPU] [Local RAM] |       | [CPU] [Local RAM] |                     |
|       | [Local OS Kernel] |       | [Local OS Kernel] |       | [Local OS Kernel] |                     |
|       +---------+---------+       +---------+---------+       +---------+---------+                     |
|                 |                           |                           |                               |
|                 +---------------------------+---------------------------+                               |
|                                             |                                                           |
|                            +----------------v----------------+                                          |
|                            |    INTERCONNECTION NETWORK      |                                          |
|                            |  (Packet-Switched LAN / WAN)    |                                          |
|                            +---------------------------------+                                          |
|                                                                                                         |
|       * Core Invariants:                                                                                |
|         1. Concurrent execution of processes across independent CPUs.                                   |
|         2. No common shared physical memory.                                                            |
|         3. Absence of a synchronized global physical clock.                                             |
|         4. Independent partial failure modes.                                                           |
+---------------------------------------------------------------------------------------------------------+
```

---

### 2. RELATION TO COMPUTER SYSTEM COMPONENTS

```
+---------------------------------------------------------------------------------------------------------+
|                  RELATION TO COMPUTER SYSTEM COMPONENTS ARCHITECTURE                                    |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|   +-------------------------------------------------------------------------------------------------+   |
|   |                       DISTRIBUTED APPLICATIONS & SERVICES LAYER                                 |   |
|   |         Web Search Engines, Distributed Ledgers, E-Commerce Portals, Streaming Platforms        |   |
|   +-----------------------------------------------+-------------------------------------------------+   |
|                                                   |                                                     |
|   +-----------------------------------------------v-------------------------------------------------+   |
|   |                                  MIDDLEWARE LAYER                                               |   |
|   |      Location Transparency, RPC / gRPC, RMI, Distributed Naming (DNS), Security Protocols       |   |
|   +-----------------------------------------------+-------------------------------------------------+   |
|                                                   |                                                     |
|   +-----------------------------------------------v-------------------------------------------------+   |
|   |                            OPERATING SYSTEM KERNEL LAYER                                        |   |
|   |       Process Schedulers, Virtual Memory Managers, Network Socket Buffers, Local IPC            |   |
|   +-----------------------------------------------+-------------------------------------------------+   |
|                                                   |                                                     |
|   +-----------------------------------------------v-------------------------------------------------+   |
|   |                            COMPUTER HARDWARE & NETWORK LAYER                                    |   |
|   |        Multi-Core x86 CPUs, ECC Memory, Storage Disks, Optical Switches, Routers, NICs          |   |
|   +-------------------------------------------------------------------------------------------------+   |
+---------------------------------------------------------------------------------------------------------+
```

1. **Application Layer:** Contains user-facing distributed applications that solve domain problems across nodes.
2. **Middleware Layer:** Sits between the application and local OS. Provides a uniform programming model, abstracting away differences in hardware architectures, operating systems, and network protocols.
3. **Operating System Layer:** Local kernels manage local hardware resources, process scheduling, memory allocation, and low-level network communication sockets.
4. **Hardware and Network Layer:** The physical computing processors, local storage drives, and network links connecting the nodes.

---

### 3. DRIVING MOTIVATIONS FOR DISTRIBUTED SYSTEMS
1. **Inherent Distribution:** Many applications are geographically dispersed by nature (e.g., automated teller machines, airline reservation systems, supply chain logistics).
2. **Resource Sharing:** Enables expensive hardware (GPU compute clusters) and software assets (replicated database tables, distributed file systems) to be shared among thousands of concurrent users.
3. **High Performance via Concurrency:** Large computational workloads can be decomposed and executed in parallel across hundreds of nodes.
4. **Fault Tolerance and High Availability:** System availability is preserved through redundancy. If an individual node crashes, workloads are automatically redirected to healthy replicas without total system outage.
5. **Incremental Scalability:** System capacity can grow incrementally by adding inexpensive commodity servers rather than purchasing expensive monolithic mainframes.

---

### 4. MAJOR DESIGN ISSUES AND ENGINEERING CHALLENGES

#### A. Transparency
Transparency is the architectural property of presenting the distributed system as a single unified computer system. The ISO/ODP model defines seven critical forms:

```
+---------------------------------------------------------------------------------------------------------+
|                                  DIMENSIONS OF DISTRIBUTION TRANSPARENCY                                |
+--------------------------+------------------------------------------------------------------------------+
| TRANSPARENCY FORM        | DEFINITION & ARCHITECTURAL MECHANISM                                         |
+--------------------------+------------------------------------------------------------------------------+
| 1. Access Transparency   | Hides differences in data representation (endianness) and invocation syntax. |
| 2. Location Transparency | Hides where a resource is physically located in the network.                 |
| 3. Migration             | Hides that a resource can move dynamically from one host to another.        |
| 4. Relocation            | Hides that a resource moved while currently in use by an active client.      |
| 5. Replication           | Hides that multiple identical copies of a resource exist for safety.         |
| 6. Concurrency           | Hides that multiple processes access shared resources concurrently.          |
| 7. Failure Transparency  | Hides the failure and recovery of system components from the user.           |
+--------------------------+------------------------------------------------------------------------------+
```

#### B. Scalability
A distributed system must remain efficient along three independent dimensions:
1. **Size Scalability:** Handling increasing numbers of users and processing nodes without degrading performance.
2. **Geographical Scalability:** Operating efficiently over wide-area networks despite high propagation delays and packet drops.
3. **Administrative Scalability:** Spanning multiple independent administrative domains while maintaining security and privacy policies.

#### C. Heterogeneity
Distributed systems consist of diverse processor architectures (ARM vs. x86), operating systems (Linux vs. Windows), and network protocols. **Middleware** bridges these differences using standardized Remote Procedure Calls (RPC), Interface Definition Languages (IDL), and data serialization formats (Protocol Buffers, JSON).

#### D. Concurrency & Synchronization
Managing concurrent access to shared resources without a shared physical clock or shared memory requires distributed mutual exclusion algorithms, logical clocks, and distributed transaction protocols.

---
---

# QUESTION 2
> ### **Compare and contrast message-passing systems with shared memory systems in depth, and explain the various primitives for distributed communication with syntax, semantics, and timing diagrams.**

---

### 1. ARCHITECTURAL COMPARISON: SHARED MEMORY VS. MESSAGE PASSING

```
+---------------------------------------------------------------------------------------------------------+
|                             SHARED MEMORY VS. MESSAGE PASSING PARADIGMS                                 |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       SHARED MEMORY SYSTEM (SMP / NUMA)                       MESSAGE-PASSING SYSTEM (CLUSTERS)         |
|                                                                                                         |
|   +--------+  +--------+  +--------+                    +-----------+       +-----------+               |
|   | CPU 1  |  | CPU 2  |  | CPU N  |                    |  Node 1   |       |  Node 2   |               |
|   +---+----+  +---+----+  +---+----+                    | CPU + RAM |       | CPU + RAM |               |
|       |           |           |                         +-----+-----+       +-----+-----+               |
|   ====+===========+===========+==== System Bus                |                   |                     |
|                   |                                     +-----v-------------------v-----+               |
|         +---------+---------+                           |    INTERCONNECTION NETWORK    |               |
|         | Shared Memory RAM |                           |    (Explicit Send / Receive)  |               |
|         +-------------------+                           +-------------------------------+               |
+---------------------------------------------------------------------------------------------------------+
```

| Technical Parameter | Shared Memory Systems | Message-Passing Systems |
| :--- | :--- | :--- |
| **Physical Architecture** | Tightly coupled multiprocessors sharing common memory buses. | Loosely coupled independent nodes connected by LAN/WAN networks. |
| **Data Exchange** | Implicit via shared variables (`LOAD`, `STORE` memory instructions). | Explicit via message packets (`SEND`, `RECEIVE` system calls). |
| **Synchronization** | Hardware locks, semaphores, mutexes, atomic instructions (CAS). | Messages, tokens, logical timestamps, distributed barriers. |
| **Scalability Bound** | Strictly bounded (typically 16–128 CPUs before bus saturation). | Massive (scales to millions of nodes in global data centers). |
| **Fault Containment** | Poor: kernel panic or memory corruption crashes the whole host. | High: single node crash leaves independent peers running. |
| **Cache Coherency** | Complex hardware snooping or directory-based coherency protocols. | Not applicable; memory is isolated locally per node. |

---

### 2. DISTRIBUTED COMMUNICATION PRIMITIVES AND SEMANTICS

Distributed communication relies on two fundamental operations:
$$\text{Send}(Destination, Message)$$
$$\text{Receive}(Source, MessageBuffer)$$

#### A. Synchronous vs. Asynchronous Primitives
1. **Synchronous Communication (Rendezvous):**
   * The sending and receiving processes synchronize temporally.
   * `Send` halts until the receiving process has invoked `Receive` and accepted the message. Both processes meet at a temporal rendezvous point.
2. **Asynchronous Communication:**
   * The sender places the message into a local buffer and resumes computation immediately, with no temporal coupling to the receiver.

```
+---------------------------------------------------------------------------------------------------------+
|                                TIMING DIAGRAMS: SYNCHRONOUS VS ASYNCHRONOUS                             |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       SYNCHRONOUS (RENDEZVOUS)                          ASYNCHRONOUS (BUFFERED)                         |
|                                                                                                         |
|  Sender                Receiver                    Sender                Receiver                       |
|    |                      |                          |                      |                           |
|    | Send()               |                          | Send()               |                           |
|    |                      |                          | (Copies to buffer;   |                           |
|    | (BLOCKED)            |                          |  resumes work!)      |                           |
|    |                      |                          |                      |                           |
|    |                      | Receive()                |                      |                           |
|    |===============>=====>| (Rendezvous!)            |======== Message =====>| Receive()                |
|    | (Resumes)            |                          |                      |                           |
+---------------------------------------------------------------------------------------------------------+
```

#### B. Blocking vs. Non-Blocking Primitives
1. **Blocking Send:**
   * The invoking process is suspended until the message is safely transferred to the underlying communication buffer or successfully transmitted.
2. **Non-Blocking Send:**
   * Control returns to the calling thread immediately. A handle, future, or callback notifies the application when the send buffer can be safely modified.
3. **Blocking Receive:**
   * The invoking process halts until a message arrives matching the receive criteria.
4. **Non-Blocking Receive:**
   * The process probes the network interface; if data is available, it retrieves it; otherwise, it receives a status code and continues alternative work.

---
---

# QUESTION 3
> ### **Explain in detail the model of distributed executions, formal definition of a distributed program, causal precedence relation, space-time diagrams, and network communication models.**

---

### 1. FORMAL MODEL OF A DISTRIBUTED PROGRAM AND EXECUTIONS
A distributed program consists of $N$ asynchronous processes denoted by $P = \{P_1, P_2, \dots, P_N\}$.
* There is no shared memory or global physical clock.
* Processes communicate exclusively via discrete message passing across directed communication channels $C_{ij}$.
* An execution of process $P_i$ is a sequence of actions modeled as discrete **Events**.

---

### 2. EVENT TYPES AND LOCAL HISTORIES
1. **Event Types:**
   * **Internal Event:** Local state change within a process (e.g., variable assignment).
   * **Send Event ($\text{send}(m)$):** Puts message $m$ into channel $C_{ij}$.
   * **Receive Event ($\text{receive}(m)$):** Removes message $m$ from channel $C_{ji}$.
2. **Local History ($h_i$):**
   The strictly ordered sequence of events executed by process $P_i$:
   $$h_i = \langle e_i^1, e_i^2, e_i^3, \dots, e_i^k \rangle$$
   The local order between events is determined by process execution order ($\prec_i$).
3. **Global History ($H$):**
   The set union of all local histories across the distributed system:
   $$H = h_1 \cup h_2 \cup \dots \cup h_N$$

---

### 3. LAMPORT'S CAUSAL PRECEDENCE RELATION ($\prec$)
The **Happened-Before relation** ($\prec$), introduced by Leslie Lamport in 1978, defines a strict **partial order** on the global set of events $H$:

```
+---------------------------------------------------------------------------------------------------------+
|                                    LAMPORT'S HAPPENED-BEFORE RULES                                      |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|  RULE 1 (Local Process Order):                                                                          |
|  If events e_i^a and e_i^b occur in the same process P_i, and a < b, then:                              |
|                              e_i^a < e_i^b                                                              |
|                                                                                                         |
|  RULE 2 (Message Causality):                                                                            |
|  If event e_i is the send(m) of message m by P_i, and event e_j is the                                  |
|  receive(m) of the same message m by P_j, then:                                                         |
|                                e_i < e_j                                                                |
|                                                                                                         |
|  RULE 3 (Transitivity):                                                                                 |
|  If e_a < e_b and e_b < e_c, then:                                                                      |
|                                e_a < e_c                                                                |
|                                                                                                         |
|  CONCURRENCY DEFINITION:                                                                                |
|  Two distinct events e_x and e_y are CONCURRENT (e_x || e_y) if and only if:                            |
|                     NOT(e_x < e_y)  AND  NOT(e_y < e_x)                                                 |
+---------------------------------------------------------------------------------------------------------+
```

---

### 4. SPACE-TIME DIAGRAMS
Space-time diagrams graphically represent distributed executions:
* Horizontal axes represent the timelines of individual processes progressing to the right.
* Vertices represent execution events.
* Directed arrows represent message transmissions.

```
+---------------------------------------------------------------------------------------------------------+
|                                        SPACE-TIME DIAGRAM TRACE                                         |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       P1  ---(e1,1)------(e1,2)-----------------(e1,3)------------>                                     |
|                     \                  \                                                                |
|                      \ m1               \ m2                                                            |
|                       v                  v                                                              |
|       P2  ---------(e2,1)--------------(e2,2)-----------(e2,3)---->                                     |
|                                  \                                                                      |
|                                   \ m3                                                                  |
|                                    v                                                                    |
|       P3  -----------------------(e3,1)-----------------(e3,2)---->                                     |
|                                                                                                         |
|       - Causal Paths: e1,1 < e2,1 < e3,1 (Transitive: e1,1 < e3,1)                                      |
|       - Concurrent Events: e1,2 || e2,1 (Neither causally influences the other)                         |
+---------------------------------------------------------------------------------------------------------+
```

---

### 5. NETWORK COMMUNICATION MODELS
Distributed systems operate under one of three network communication abstractions:
1. **FIFO Model:**
   * Messages transmitted along a given unidirectional channel arrive at the destination in the order they were sent:
     $$\text{send}(m_1) \prec \text{send}(m_2) \implies \text{deliver}(m_1) \prec \text{deliver}(m_2)$$
2. **Non-FIFO Model:**
   * Messages on the same channel may be delivered out of order due to dynamic network routing, link congestion, or retransmissions.
3. **Causal Ordering (CO) Model:**
   * If the transmission of message $m_1$ causally precedes the transmission of message $m_2$ ($m_1 \prec m_2$), then $m_1$ must be delivered before $m_2$ at all common destinations, even if sent by different processes.

---
---

# QUESTION 4
> ### **Discuss in detail the global state of a distributed system, past and future cones of an event, cuts, consistent cuts, runs, and lattice of global states with comprehensive illustrative examples and diagrams.**

---

### 1. THE GLOBAL STATE PROBLEM
In a centralized system, the global state can be captured by reading memory and registers at a single clock tick. In a distributed system, because there is no synchronized global physical clock and message transit times are non-zero, **an instantaneous physical snapshot is impossible**.
Instead, distributed systems define global states in terms of **Causal Cuts** across local histories and channel contents.

```
+---------------------------------------------------------------------------------------------------------+
|                                  COMPONENTS OF A DISTRIBUTED GLOBAL STATE                               |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|   GLOBAL STATE GS = { State(P1), State(P2), ..., State(PN), State(Channels Cij) }                       |
|                                                                                                         |
|   - Local Process State: Local variables, registers, program counter.                                   |
|   - Channel State C_ij: Messages sent by P_i to P_j that have not yet been delivered                    |
|     to P_j:                                                                                             |
|            State(C_ij) = { m | send_i(m) in GS  AND  receive_j(m) NOT in GS }                           |
+---------------------------------------------------------------------------------------------------------+
```

---

### 2. PAST AND FUTURE CONES OF AN EVENT
For any event $e$ in a distributed execution $(E, \prec)$:
1. **Past Cone of $e$ ($\text{Past}(e)$):**
   * The set of all events that causally precede $e$:
     $$\text{Past}(e) = \{ e' \in E \mid e' \prec e \}$$
   * Represents all events that could have influenced event $e$.
2. **Future Cone of $e$ ($\text{Future}(e)$):**
   * The set of all events that are causally affected by $e$:
     $$\text{Future}(e) = \{ e' \in E \mid e \prec e' \}$$
3. **Concurrent Event Set ($\text{Concurrent}(e)$):**
   * The set of events that lie outside both cones:
     $$\text{Concurrent}(e) = \{ e' \in E \mid e' \parallel e \}$$

```
+---------------------------------------------------------------------------------------------------------+
|                                       PAST AND FUTURE CONES OF AN EVENT                                 |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|              \                   /                                                                      |
|               \  FUTURE CONE    /     Events affected by e                                              |
|                \   Future(e)   /                                                                        |
|                 \             /                                                                         |
|                  \           /                                                                          |
|                   +---------+                                                                           |
|                   | EVENT e |                                                                           |
|                   +---------+                                                                           |
|                  /           \                                                                          |
|                 /             \                                                                         |
|                /   PAST CONE   \      Events that influenced e                                          |
|               /    Past(e)      \                                                                       |
|              /                   \                                                                      |
|                                                                                                         |
|       All events outside the cones belong to the CONCURRENT SET Concurrent(e)                          |
+---------------------------------------------------------------------------------------------------------+
```

---

### 3. CUTS, CONSISTENT CUTS, AND INCONSISTENT CUTS
* **Cut:** A mathematical frontier dividing the event set $E$ into a Past set $P$ and a Future set $F$.
* **Consistent Cut:** A cut is consistent if the Past set is closed under the causal precedence relation:
  $$\forall e, e': (e \in P \land e' \prec e) \implies e' \in P$$
  *(No message receipt can appear in the past without its corresponding send event also being in the past).*
* **Inconsistent Cut:** Contains an event $e \in P$ such that $e' \prec e$ but $e' \in F$. This represents an **orphan message** anomaly (an effect exists without its cause).

```
+---------------------------------------------------------------------------------------------------------+
|                                   CONSISTENT VS. INCONSISTENT CUT TRACE                                 |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|       P1  ---(e1,1)---------|--------(e1,2)------------------------->                                   |
|                    \        |                                                                           |
|                     \ m1    | Consistent Cut C1                                                         |
|                      v      |                                                                           |
|       P2  ---------(e2,1)---|-----------------(e2,2)---------------->                                   |
|                                                                                                         |
|       ---------------------------------------------------------------                                   |
|                                                                                                         |
|       P1  ---(e1,1)-----------------|--------(e1,2)----------------->                                   |
|                                     |           \                                                       |
|                                     |            \ m2 (Orphan message!)                                 |
|                                     |             v                                                     |
|       P2  ---(e2,1)-----------------(e2,2)------|------------------->                                   |
|                                                 | Inconsistent Cut C2                                   |
|                                                                                                         |
|       In Cut C2: e2,2 is in the past, but the send event e1,2 is in the FUTURE!                         |
|       ===> INCONSISTENT GLOBAL STATE!                                                                   |
+---------------------------------------------------------------------------------------------------------+
```

---

### 4. RUNS AND THE LATTICE OF GLOBAL STATES
* **Run:** A total ordering of all events in $E$ that is consistent with the partial order $\prec$. A run represents a possible sequential execution path observed by a global monitor.
* **Lattice of Global States:**
  * The set of all consistent global states forms a **distributive lattice** under the subset inclusion relation ($\subseteq$).
  * The bottom element represents the initial state $(S_0)$.
  * The top element represents the completed state $(S_{final})$.
  * Paths through the lattice from $S_0$ to $S_{final}$ represent distinct valid sequential runs of the distributed program.

```
+---------------------------------------------------------------------------------------------------------+
|                                      LATTICE OF GLOBAL STATES                                           |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|                                              S_0 (Initial)                                              |
|                                              /           \                                              |
|                                             v             v                                             |
|                                          S(1,0)         S(0,1)                                          |
|                                             \             /                                             |
|                                              v           v                                              |
|                                                 S(1,1)                                                  |
|                                              /           \                                              |
|                                             v             v                                             |
|                                          S(2,1)         S(1,2)                                          |
|                                             \             /                                             |
|                                              v           v                                              |
|                                            S_final (Terminated)                                         |
+---------------------------------------------------------------------------------------------------------+
```

---
