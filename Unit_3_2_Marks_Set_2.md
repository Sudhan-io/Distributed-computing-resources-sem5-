# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT III: DISTRIBUTED MUTUAL EXCLUSION AND DEADLOCK DETECTION
### PART A: 2-MARK QUESTIONS AND ANSWERS (SET 2 - ALTERNATE QUESTION BANK)

---

### Q1. What is a quorum in distributed mutual exclusion?
**Answer:**
* A **Quorum** (or Request Set $R_i$) is a subset of sites associated with site $S_i$ such that $S_i$ only needs permission from members of $R_i$ to enter the Critical Section.
* Quorums reduce message complexity from $O(N)$ to $O(\sqrt{N})$ by avoiding global broadcasts.

---

### Q2. State the two essential conditions that Maekawa's request sets must satisfy.
**Answer:**
1. **Intersection Property (Pairwise Non-Empty Intersection):**
   $$\forall i, j \in \{1, \dots, N\}: \quad R_i \cap R_j \ne \emptyset$$
   Guarantees that any two requesting sites share at least one common arbiter site, ensuring mutual exclusion.
2. **Self-Containment & Equal Size:**
   $$\forall i: S_i \in R_i \quad \text{and} \quad |R_i| = K \approx \sqrt{N}$$
   Every site belongs to its own request set and all quorums have approximately equal size to balance load.

---

### Q3. Why is Maekawa's algorithm prone to deadlock?
**Answer:**
* Maekawa's algorithm is prone to deadlock because request sets can form circular wait dependencies across sites.
* If three sites $S_1, S_2, S_3$ request CS simultaneously, an arbiter site in $R_1 \cap R_2$ may grant permission to $S_1$, while another arbiter in $R_2 \cap R_3$ grants to $S_2$, and an arbiter in $R_3 \cap R_1$ grants to $S_3$, resulting in a cyclic wait where none can collect all $\sqrt{N}$ permissions.

---

### Q4. What is the role of INQUIRE, YIELD, and FAILED messages in Maekawa's algorithm?
**Answer:**
These messages are used to prevent and resolve deadlocks:
* **`FAILED`:** Sent by an arbiter to a requester when it has already granted its vote to another site with higher priority.
* **`INQUIRE`:** Sent by an arbiter to a site currently holding its vote, asking if it can yield the vote if it has received a `FAILED` message elsewhere.
* **`YIELD`:** Sent by a requester back to the arbiter, relinquishing the previously granted vote to break a potential deadlock.

---

### Q5. What is the average message complexity of Raymond's Tree Algorithm?
**Answer:**
* **Average Message Complexity:** $O(\log N)$ messages per Critical Section execution.
* **Best Case:** $0$ messages (if the requesting node already holds the token).
* **Worst Case:** $2D$ messages (where $D$ is the tree diameter; at most $2(N-1)$ in a degenerate linear tree).

---

### Q6. How does the 'holder' pointer function in Raymond's Tree Algorithm?
**Answer:**
* Each node $P_i$ maintains a local pointer variable called `holder`.
* `holder` points to the neighboring node on the directed path leading towards the current location of the token.
* If a node currently possesses the token, its `holder` points to **itself** (`holder = self`).

---

### Q7. Differentiate between Wait-Die and Wound-Wait deadlock prevention schemes.
**Answer:**

| Feature | Wait-Die Scheme | Wound-Wait Scheme |
| :--- | :--- | :--- |
| **Strategy** | Non-preemptive | Preemptive |
| **Old requests Young** | Old transaction **waits** for young transaction | Old transaction **wounds** (preempts/aborts) young |
| **Young requests Old** | Young transaction **dies** (aborts and restarts) | Young transaction **waits** for old transaction |
| **Rollback Frequency** | Higher number of aborts and restarts | Fewer aborts; older transactions proceed quickly |

---

### Q8. What is a phantom deadlock, and why does it occur in distributed systems?
**Answer:**
* **Phantom Deadlock:** A false deadlock detected by a distributed deadlock detection algorithm that does not actually exist in the system.
* **Cause:** Occurs due to variable network message delays. If a resource release message is delayed in transit while a new request message arrives early, the detection algorithm observes an obsolete state and incorrectly reports a cycle.

---

### Q9. What is a knot in a Wait-For Graph (WFG)?
**Answer:**
* A **Knot** $K$ in a directed graph is a non-empty subset of vertices such that for every vertex $u \in K$, all reachable vertices from $u$ are also in $K$, and every vertex in $K$ can reach all other vertices in $K$.
* In the **OR model** of distributed deadlocks, a deadlock exists if and only if a blocked process belongs to a knot.

---

### Q10. Distinguish between the AND model and the OR model of deadlocks.
**Answer:**
* **AND Model:** A process requests multiple resources and remains blocked until **all** requested resources are granted. Deadlock condition $\iff$ existence of a **directed cycle** in the WFG.
* **OR Model:** A process requests multiple resources and unblocks if **any one** resource is granted. Deadlock condition $\iff$ existence of a **knot** in the WFG.

---

### Q11. What is an edge-chasing algorithm?
**Answer:**
* An edge-chasing algorithm is a distributed deadlock detection technique where sites do not construct or centralize the global Wait-For Graph.
* Instead, special diagnostic messages called **probes** propagate along the directed wait-for edges of the distributed WFG. A deadlock is detected when a probe returns to its initiator.

---

### Q12. How does the Mitchell-Merritt algorithm detect deadlocks using public and private labels?
**Answer:**
* Each node maintains two integer variables: a **private label** (unique monotonically increasing integer) and a **public label**.
* When a node blocks, it transmits a probe carrying its public label along the wait edge.
* When a transmission causes a public label to become smaller than the target's public label, labels are updated.
* If a node receives a probe with a public label equal to its own public label, a cycle is detected and deadlock is declared.

---

### Q13. State the synchronization delay of Suzuki-Kasami vs. Ricart-Agrawala algorithms.
**Answer:**
* **Suzuki-Kasami Algorithm:** Synchronization delay is **$T$** (the time for a single direct unicast transmission of the Token from the exiting node to the next requesting node).
* **Ricart-Agrawala Algorithm:** Synchronization delay is **$T$** (the time required for a deferred `REPLY` message to reach the next requesting node).

---
