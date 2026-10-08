# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT IV: CONSENSUS AND RECOVERY
### PART A: 2-MARK QUESTIONS AND ANSWERS (SET 2 - ALTERNATE QUESTION BANK)

---

### Q1. What is the fundamental difference between Two-Phase Commit (2PC) and Three-Phase Commit (3PC)?
**Answer:**
* **2PC is a blocking protocol:** If the coordinator crashes after participants vote "YES", participants remain blocked indefinitely waiting for the final decision.
* **3PC is a non-blocking protocol:** Introduces a third state (**Pre-Commit**) that removes uncertainty. Participants can reach a consistent decision independently if the coordinator crashes.

---

### Q2. What are the three roles in the Paxos consensus algorithm?
**Answer:**
1. **Proposers:** Advocate client requests by proposing values associated with unique proposal numbers.
2. **Acceptors:** Form a consensus quorum that votes on proposed values.
3. **Learners:** Act on the agreed value once a quorum of Acceptors accepts a proposal.

---

### Q3. State the Leader Completeness property in Raft.
**Answer:**
* **Leader Completeness Property:** If a log entry is committed in a given term, then that entry will be present in the logs of the leaders for all higher-numbered terms.
* Ensures that a new leader never overwrites or discards committed entries.

---

### Q4. What is a Z-path in distributed checkpointing?
**Answer:**
* A **Z-path** (zigzag path) between two checkpoints $C_{i,m}$ and $C_{j,n}$ is a sequence of messages $\langle m_1, m_2, \dots, m_k \rangle$ such that each message is either sent after or received before the previous message's checkpoint boundary.
* Formalizes causal message patterns that span across checkpoint boundaries.

---

### Q5. What is a Z-cycle, and what is its significance?
**Answer:**
* A **Z-cycle** is a $Z$-path that starts and ends at the **same** checkpoint.
* **Significance:** A checkpoint belongs to a consistent recovery line if and only if it is **not involved in any Z-cycle**. $Z$-cycles cause the domino effect.

---

### Q6. Define the Piecewise Deterministic (PWD) assumption in message logging.
**Answer:**
* The **Piecewise Deterministic (PWD) Assumption** states that all non-deterministic events (such as message receipt order and external interrupts) can be identified, and their determinants can be recorded on stable storage.
* Between consecutive non-deterministic events, execution is completely deterministic and reproducible.

---

### Q7. What is an orphan process in message logging?
**Answer:**
* An **Orphan Process** is a surviving process whose active state depends on a non-deterministic event that was not logged before a failure and therefore cannot be reproduced upon recovery.

---

### Q8. How does pessimistic logging prevent the creation of orphan processes?
**Answer:**
* **Pessimistic Logging Rule:** The determinant of every received message is synchronously flushed to non-volatile stable storage **before** the process is permitted to act on the message or send any subsequent message.
* Guarantees that uncommitted non-deterministic events are never observed by peers.

---

### Q9. What is a recovery line?
**Answer:**
* The **Recovery Line** is the most recent set of local checkpoints (one per process) that forms a causally consistent global state containing no orphan messages.
* When a failure occurs, the system rolls back to the recovery line.

---

### Q10. State the minimum number of nodes required to tolerate m Byzantine faults using digital signatures.
**Answer:**
* With digital signatures (authenticated messages), the bound is:
  $$N \ge m + 1$$
* *Contrast:* Without digital signatures (oral messages), the bound is strictly $N \ge 3m + 1$.

---

### Q11. What is Atomic Broadcast?
**Answer:**
* **Atomic Broadcast (Total Order Broadcast):** A broadcast primitive where all non-faulty processes deliver the exact same sequence of messages, and all sent messages are eventually delivered.
* Equivalent in solvability to distributed consensus.

---

### Q12. Differentiate between uncoordinated and coordinated checkpointing.
**Answer:**
* **Uncoordinated Checkpointing:** Processes record local checkpoints autonomously without synchronizing with peers; vulnerable to the domino effect.
* **Coordinated Checkpointing:** Processes synchronize via control messages (e.g., Koo-Toueg) to guarantee that every checkpoint set forms a consistent cut, eliminating the domino effect.

---
