# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT I: INTRODUCTION & MODEL OF DISTRIBUTED COMPUTATIONS
### PART A: 2-MARK QUESTIONS AND ANSWERS (EXAM-READY REPOSITORY)

---

### Q1. Define a distributed system and state its key characteristics.
**Answer:**
* **Definition:** A distributed system is a collection of autonomous, geographically dispersed computing nodes connected by a communication network, that coordinate their actions purely via message passing and appear to the end-user as a single coherent, integrated computing facility.
* **Key Characteristics:**
  1. **Absence of Shared Memory & Global Physical Clock:** Nodes do not share physical memory; each node runs an independent operating system and local timer.
  2. **Concurrency & Independent Failures:** Processes execute concurrently, and components may fail independently without causing the entire system to crash.

---

### Q2. How do distributed systems relate to computer system components?
**Answer:**
A distributed system spans across four key structural layers:
1. **Application Layer:** Distributed user applications, services, and workflows.
2. **Middleware Layer:** Sits between the application and local OS; provides location transparency, communication primitives (RPC/RMI), and global naming.
3. **Operating System Layer:** Local kernels managing local memory, processes, and network communication buffers.
4. **Computer System Hardware & Network Layer:** Physical CPUs, storage, switches, routers, and communication links.

---

### Q3. What are the primary motivations for building distributed systems?
**Answer:**
1. **Resource Sharing:** Hardware (printers, specialized compute nodes) and software (databases, files) can be shared across diverse geographic locations.
2. **Enhanced Performance & Throughput:** Parallel execution divides large computational workloads among multiple processors.
3. **High Reliability & Fault Tolerance:** Failure of one node is masked by dynamically routing workloads to replicated healthy nodes.
4. **Scalability:** System capacity can grow incrementally by adding commodity machines without altering software architecture.

---

### Q4. Compare message-passing systems with shared memory systems.
**Answer:**

| Parameter | Message-Passing System | Shared Memory System |
| :--- | :--- | :--- |
| **Communication Medium** | Interconnection network (Send/Receive packets) | Common shared physical address space (Read/Write) |
| **Hardware Scalability** | High (scales to thousands of commodity nodes) | Low (bounded by memory bus and cache coherency) |
| **Synchronization** | Message primitives, channels, timestamps | Semaphores, mutex locks, atomic hardware instructions |
| **Locality & Latency** | Non-uniform, higher network latency | Low latency uniform memory access |

---

### Q5. What are the primitives used for distributed communication?
**Answer:**
Distributed communication relies on message-passing primitives:
* **Send Primitive (`Send(destination, message)`):** Transmits data to a target process.
* **Receive Primitive (`Receive(source, message_buffer)`):** Retrieves arriving data from the channel.
* **Execution Semantics:**
  1. *Blocking (Synchronous):* Invoking thread halts until the message is copied to the network buffer or received by the peer.
  2. *Non-Blocking (Asynchronous):* Invoking thread immediately resumes execution after initiating the transfer.

---

### Q6. Distinguish between synchronous and asynchronous executions in distributed systems.
**Answer:**
* **Synchronous Execution:**
  * Upper bounds exist on message communication delay ($\Delta$) and relative processor execution speeds.
  * Allows execution in lock-step rounds; timeouts reliably detect process crashes.
* **Asynchronous Execution:**
  * No bounds exist on communication latency or processor speeds.
  * Messages take arbitrary but finite time. A slow process cannot be reliably distinguished from a crashed process (FLP Impossibility applies).

---

### Q7. List the primary design issues and challenges in distributed systems.
**Answer:**
1. **Transparency:** Hiding distribution details (Access, Location, Migration, Replication, Concurrency, Failure).
2. **Scalability:** Operating efficiently as the number of users, nodes, and geographic distance expand.
3. **Heterogeneity:** Interoperating across diverse hardware, operating systems, programming languages, and network protocols.
4. **Fault Tolerance & Dependability:** Detecting, masking, and recovering from partial failures without total outage.
5. **Security:** Protecting communication against eavesdropping, spoofing, and tampering across open networks.

---

### Q8. Define a distributed program and a distributed execution.
**Answer:**
* **Distributed Program:** A collection of $N$ asynchronous processes $P = \{P_1, P_2, \dots, P_N\}$ that communicate exclusively by exchanging messages over communication channels.
* **Distributed Execution:** A partially ordered set of internal and communication events $(E, \prec)$ occurring across the $N$ processes, governed by Lamport's happened-before relation ($\prec$).

---

### Q9. What is a space-time diagram in distributed executions?
**Answer:**
* A graphical representation of distributed executions where:
  * Horizontal lines represent the timeline of individual processes progressing from left to right.
  * Points on the timeline denote local events (internal computation, message send, message receive).
  * Directed arrows connecting process timelines represent message transmissions from the send event to the receive event.

---

### Q10. Differentiate between FIFO, Non-FIFO, and Causal communication network models.
**Answer:**
* **FIFO Channels:** Messages sent along a specific channel are delivered to the destination in the exact order they were transmitted:
  $$\text{send}(m_1) \prec \text{send}(m_2) \implies \text{deliver}(m_1) \prec \text{deliver}(m_2)$$
* **Non-FIFO Channels:** Messages may overtake each other in transit due to multi-path network routing.
* **Causal Channels:** If the transmission of message $m_1$ causally precedes the transmission of message $m_2$ ($m_1 \prec m_2$), then $m_1$ must be delivered before $m_2$ at all common destinations.

---

### Q11. Define the global state of a distributed system.
**Answer:**
* The **Global State** of a distributed system is the union of the local states of all $N$ processes and the states of all communication channels at a specific instant:
  $$\text{Global State } GS = \left(\bigcup_{i=1}^N \text{Local State}(P_i)\right) \cup \left(\bigcup_{i=1}^N \bigcup_{j=1}^N \text{Channel State}(C_{ij})\right)$$
* Channel state $C_{ij}$ contains all messages sent by $P_i$ to $P_j$ but not yet delivered to $P_j$.

---

### Q12. What is a consistent cut and an inconsistent cut in distributed executions?
**Answer:**
* **Cut:** A dividing line across a space-time diagram partitioning the set of events $E$ into a Past set ($P$) and a Future set ($F$).
* **Consistent Cut:** A cut where causality is preserved—no message is recorded as received in the past without being recorded as sent in the past:
  $$e_j \in P \land e_i \prec e_j \implies e_i \in P$$
* **Inconsistent Cut:** A cut containing a message receive event whose corresponding send event lies in the future (an orphan message effect).

---
