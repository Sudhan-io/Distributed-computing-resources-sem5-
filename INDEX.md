# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## MASTER QUESTION BANK & INTERACTIVE NAVIGATION INDEX

> **Quick How-To:**
> 1. Click any link in the **Quick Navigation Table** below to jump directly to that Unit, Part, and Mark section.
> 2. Each question prominently displays its examination **[Part A]**, **[Part B]**, or **[Part C]** category and S.No directly from the official college question bank.
> 3. In each section, click any **Question Title** to immediately open the full answer in the corresponding study markdown file.

---

### Quick Navigation Table

| Unit | Set / Type | Part A (2 Marks) | Part B (8 Marks) | Part C (16 Marks) |
| :--- | :--- | :---: | :---: | :---: |
| **Unit I** (Message Passing & Time) | Standard Bank | [Part A (2M)](#unit-1-message-passing--logical-time---part-a---2-marks) | [Part B (8M)](#unit-1-message-passing--logical-time---part-b---8-marks) | [Part C (16M)](#unit-1-message-passing--logical-time---part-c---16-marks) |
| **Unit II** (Global State & DSM) | Standard Bank | [Part A (2M)](#unit-2-global-state--distributed-shared-memory---part-a---2-marks) | [Part B (8M)](#unit-2-global-state--distributed-shared-memory---part-b---8-marks) | [Part C (16M)](#unit-2-global-state--distributed-shared-memory---part-c---16-marks) |
| **Unit III** (Mutual Exclusion & Deadlocks) | **Set 1** (Exam Bank) | [Part A (2M)](#unit-3-mutual-exclusion--deadlock-detection---set-1-question-bank---part-a---2-marks) | [Part B (8M)](#unit-3-mutual-exclusion--deadlock-detection---set-1-question-bank---part-b---8-marks) | [Part C (16M)](#unit-3-mutual-exclusion--deadlock-detection---set-1-question-bank---part-c---16-marks) |
| **Unit III** (Mutual Exclusion & Deadlocks) | **Set 2** (Alternate Bank) | [Part A (2M)](#unit-3-mutual-exclusion--deadlock-detection---set-2-alternate-bank---part-a---2-marks) | [Part B (8M)](#unit-3-mutual-exclusion--deadlock-detection---set-2-alternate-bank---part-b---8-marks) | [Part C (16M)](#unit-3-mutual-exclusion--deadlock-detection---set-2-alternate-bank---part-c---16-marks) |
| **Unit IV** (Consensus & Recovery) | **Set 1** (Exam Bank) | [Part A (2M)](#unit-4-consensus--recovery---set-1-question-bank---part-a---2-marks) | [Part B (8M)](#unit-4-consensus--recovery---set-1-question-bank---part-b---8-marks) | [Part C (16M)](#unit-4-consensus--recovery---set-1-question-bank---part-c---16-marks) |
| **Unit IV** (Consensus & Recovery) | **Set 2** (Alternate Bank) | [Part A (2M)](#unit-4-consensus--recovery---set-2-alternate-bank---part-a---2-marks) | [Part B (8M)](#unit-4-consensus--recovery---set-2-alternate-bank---part-b---8-marks) | [Part C (16M)](#unit-4-consensus--recovery---set-2-alternate-bank---part-c---16-marks) |
| **Unit V** (Cloud Computing) | **Set 1** (Exam Bank) | [Part A (2M)](#unit-5-cloud-computing---set-1-question-bank---part-a---2-marks) | [Part B (8M)](#unit-5-cloud-computing---set-1-question-bank---part-b---8-marks) | [Part C (16M)](#unit-5-cloud-computing---set-1-question-bank---part-c---16-marks) |
| **Unit V** (Cloud Computing) | **Set 2** (Alternate Bank) | [Part A (2M)](#unit-5-cloud-computing---set-2-alternate-bank---part-a---2-marks) | [Part B (8M)](#unit-5-cloud-computing---set-2-alternate-bank---part-b---8-marks) | [Part C (16M)](#unit-5-cloud-computing---set-2-alternate-bank---part-c---16-marks) |

---

## Unit 1: Message Passing & Logical Time - Part A - 2 Marks
> **Examination Section:** Part A (2 Marks) &nbsp;|&nbsp; **Target File:** [Unit_1_2_Marks.md](./Unit_1_2_Marks.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part A | 2 Marks]`** [**Q1. Define a distributed system and state its key characteristics.**](./Unit_1_2_Marks.md#q1.-define-a-distributed-system-and-state-its-key-characteristics.)
* **`[Part A | 2 Marks]`** [**Q2. How do distributed systems relate to computer system components?**](./Unit_1_2_Marks.md#q2.-how-do-distributed-systems-relate-to-computer-system-components)
* **`[Part A | 2 Marks]`** [**Q3. What are the primary motivations for building distributed systems?**](./Unit_1_2_Marks.md#q3.-what-are-the-primary-motivations-for-building-distributed-systems)
* **`[Part A | 2 Marks]`** [**Q4. Compare message-passing systems with shared memory systems.**](./Unit_1_2_Marks.md#q4.-compare-message-passing-systems-with-shared-memory-systems.)
* **`[Part A | 2 Marks]`** [**Q5. What are the primitives used for distributed communication?**](./Unit_1_2_Marks.md#q5.-what-are-the-primitives-used-for-distributed-communication)
* **`[Part A | 2 Marks]`** [**Q6. Distinguish between synchronous and asynchronous executions in distributed systems.**](./Unit_1_2_Marks.md#q6.-distinguish-between-synchronous-and-asynchronous-executions-in-distributed-systems.)
* **`[Part A | 2 Marks]`** [**Q7. List the primary design issues and challenges in distributed systems.**](./Unit_1_2_Marks.md#q7.-list-the-primary-design-issues-and-challenges-in-distributed-systems.)
* **`[Part A | 2 Marks]`** [**Q8. Define a distributed program and a distributed execution.**](./Unit_1_2_Marks.md#q8.-define-a-distributed-program-and-a-distributed-execution.)
* **`[Part A | 2 Marks]`** [**Q9. What is a space-time diagram in distributed executions?**](./Unit_1_2_Marks.md#q9.-what-is-a-space-time-diagram-in-distributed-executions)
* **`[Part A | 2 Marks]`** [**Q10. Differentiate between FIFO, Non-FIFO, and Causal communication network models.**](./Unit_1_2_Marks.md#q10.-differentiate-between-fifo-non-fifo-and-causal-communication-network-models.)
* **`[Part A | 2 Marks]`** [**Q11. Define the global state of a distributed system.**](./Unit_1_2_Marks.md#q11.-define-the-global-state-of-a-distributed-system.)
* **`[Part A | 2 Marks]`** [**Q12. What is a consistent cut and an inconsistent cut in distributed executions?**](./Unit_1_2_Marks.md#q12.-what-is-a-consistent-cut-and-an-inconsistent-cut-in-distributed-executions)

---

## Unit 1: Message Passing & Logical Time - Part B - 8 Marks
> **Examination Section:** Part B (8 Marks) &nbsp;|&nbsp; **Target File:** [Unit_1_8_Marks.md](./Unit_1_8_Marks.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part B | 8 Marks]`** [**QUESTION 1: Contrast Message-Passing Systems with Shared Memory Systems in Detail with Architectural Diagrams and Trade-offs.**](./Unit_1_8_Marks.md#question-1-contrast-message-passing-systems-with-shared-memory-systems-in-detail-with-architectural-diagrams-and-trade-offs.)
* **`[Part B | 8 Marks]`** [**QUESTION 2: Discuss the Primitives for Distributed Communication (Blocking vs. Non-Blocking, Synchronous vs. Asynchronous Send/Receive).**](./Unit_1_8_Marks.md#question-2-discuss-the-primitives-for-distributed-communication-blocking-vs.-non-blocking-synchronous-vs.-asynchronous-sendreceive.)
* **`[Part B | 8 Marks]`** [**QUESTION 3: Differentiate Between Synchronous and Asynchronous Distributed System Execution Models.**](./Unit_1_8_Marks.md#question-3-differentiate-between-synchronous-and-asynchronous-distributed-system-execution-models.)
* **`[Part B | 8 Marks]`** [**QUESTION 4: Explain the Key Design Issues and Challenges in Distributed Systems.**](./Unit_1_8_Marks.md#question-4-explain-the-key-design-issues-and-challenges-in-distributed-systems.)
* **`[Part B | 8 Marks]`** [**QUESTION 5: Describe the Model of Distributed Executions (Events, Local Histories, Global Histories, Partial Orders, Space-Time Diagrams).**](./Unit_1_8_Marks.md#question-5-describe-the-model-of-distributed-executions-events-local-histories-global-histories-partial-orders-space-time-diagrams.)
* **`[Part B | 8 Marks]`** [**QUESTION 6: Explain the Different Models of Communication Networks (FIFO, Non-FIFO, Causal Order) with Examples.**](./Unit_1_8_Marks.md#question-6-explain-the-different-models-of-communication-networks-fifo-non-fifo-causal-order-with-examples.)
* **`[Part B | 8 Marks]`** [**QUESTION 7: Explain the Concept of Global State, Cuts, and Consistent Cuts in a Distributed System with a Neat Diagram.**](./Unit_1_8_Marks.md#question-7-explain-the-concept-of-global-state-cuts-and-consistent-cuts-in-a-distributed-system-with-a-neat-diagram.)

---

## Unit 1: Message Passing & Logical Time - Part C - 16 Marks
> **Examination Section:** Part C (16 Marks) &nbsp;|&nbsp; **Target File:** [Unit_1_16_Marks.md](./Unit_1_16_Marks.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part C | 16 Marks]`** [**Question 1: Discuss in detail the definition, motivation, and relation of distributed systems to computer system components, and elaborate on the major design issues and challenges in distributed systems.**](./Unit_1_16_Marks.md#question-1)
* **`[Part C | 16 Marks]`** [**Question 2: Compare and contrast message-passing systems with shared memory systems in depth, and explain the various primitives for distributed communication with syntax, semantics, and timing diagrams.**](./Unit_1_16_Marks.md#question-2)
* **`[Part C | 16 Marks]`** [**Question 3: Explain in detail the model of distributed executions, formal definition of a distributed program, causal precedence relation, space-time diagrams, and network communication models.**](./Unit_1_16_Marks.md#question-3)
* **`[Part C | 16 Marks]`** [**Question 4: Discuss in detail the global state of a distributed system, past and future cones of an event, cuts, consistent cuts, runs, and lattice of global states with comprehensive illustrative examples and diagrams.**](./Unit_1_16_Marks.md#question-4)

---

## Unit 2: Global State & Distributed Shared Memory - Part A - 2 Marks
> **Examination Section:** Part A (2 Marks) &nbsp;|&nbsp; **Target File:** [Unit_2_2_Marks.md](./Unit_2_2_Marks.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part A | S.No 46 | 2M]`** [**Q1. What is logical time, and why is physical time insufficient in distributed systems?**](./Unit_2_2_Marks.md#q1.-what-is-logical-time-and-why-is-physical-time-insufficient-in-distributed-systems)
* **`[Part A | 2 Marks]`** [**Q2. Explain the Clock Condition in Lamport's framework for logical clocks.**](./Unit_2_2_Marks.md#q2.-explain-the-clock-condition-in-lamports-framework-for-logical-clocks.)
* **`[Part A | 2 Marks]`** [**Q3. State the rules for updating scalar logical clocks (Lamport's Clocks).**](./Unit_2_2_Marks.md#q3.-state-the-rules-for-updating-scalar-logical-clocks-lamports-clocks.)
* **`[Part A | 2 Marks]`** [**Q4. Define vector time and state its updating rules.**](./Unit_2_2_Marks.md#q4.-define-vector-time-and-state-its-updating-rules.)
* **`[Part A | 2 Marks]`** [**Q5. Why cannot scalar clocks detect causal concurrency, whereas vector clocks can?**](./Unit_2_2_Marks.md#q5.-why-cannot-scalar-clocks-detect-causal-concurrency-whereas-vector-clocks-can)
* **`[Part A | 2 Marks]`** [**Q6. What is clock drift and clock skew?**](./Unit_2_2_Marks.md#q6.-what-is-clock-drift-and-clock-skew)
* **`[Part A | 2 Marks]`** [**Q7. Briefly explain the Network Time Protocol (NTP) hierarchy and synchronization mechanism.**](./Unit_2_2_Marks.md#q7.-briefly-explain-the-network-time-protocol-ntp-hierarchy-and-synchronization-mechanism.)
* **`[Part A | 2 Marks]`** [**Q8. What are the message ordering paradigms in distributed systems?**](./Unit_2_2_Marks.md#q8.-what-are-the-message-ordering-paradigms-in-distributed-systems)
* **`[Part A | 2 Marks]`** [**Q9. Define Causal Order (CO) of messages.**](./Unit_2_2_Marks.md#q9.-define-causal-order-co-of-messages.)
* **`[Part A | 2 Marks]`** [**Q10. Define Total Order (TO) of messages.**](./Unit_2_2_Marks.md#q10.-define-total-order-to-of-messages.)
* **`[Part A | 2 Marks]`** [**Q11. What is meant by Asynchronous Execution with Synchronous Communication (AESC) / RSC execution?**](./Unit_2_2_Marks.md#q11.-what-is-meant-by-asynchronous-execution-with-synchronous-communication-aesc-rsc-execution)
* **`[Part A | 2 Marks]`** [**Q12. What is the Crown condition/test in message ordering?**](./Unit_2_2_Marks.md#q12.-what-is-the-crown-conditiontest-in-message-ordering)
* **`[Part C | S.No 65 | 8M]`** [**Q13. What is the purpose of the Chandy-Lamport distributed snapshot algorithm?**](./Unit_2_2_Marks.md#q13.-what-is-the-purpose-of-the-chandy-lamport-distributed-snapshot-algorithm)

---

## Unit 2: Global State & Distributed Shared Memory - Part B - 8 Marks
> **Examination Section:** Part B (8 Marks) &nbsp;|&nbsp; **Target File:** [Unit_2_8_Marks.md](./Unit_2_8_Marks.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part B | S.No 59 | 8M]`** [**QUESTION 1: Explain Lamport's Scalar Logical Clock Framework, Clock Conditions, and Update Rules with an Example.**](./Unit_2_8_Marks.md#question-1-explain-lamports-scalar-logical-clock-framework-clock-conditions-and-update-rules-with-an-example.)
* **`[Part B | 8 Marks]`** [**QUESTION 2: Explain Vector Time, Its Structure, Update Rules, and Illustrate How It Captures Causal Independence with a Space-Time Diagram.**](./Unit_2_8_Marks.md#question-2-explain-vector-time-its-structure-update-rules-and-illustrate-how-it-captures-causal-independence-with-a-space-time-diagram.)
* **`[Part B | 8 Marks]`** [**QUESTION 3: Describe Physical Clock Synchronization and the Network Time Protocol (NTP) Architecture and Synchronization Formula.**](./Unit_2_8_Marks.md#question-3-describe-physical-clock-synchronization-and-the-network-time-protocol-ntp-architecture-and-synchronization-formula.)
* **`[Part B | 8 Marks]`** [**QUESTION 4: Compare FIFO, Causal Order (CO), and Total Order (TO) Message Ordering Paradigms.**](./Unit_2_8_Marks.md#question-4-compare-fifo-causal-order-co-and-total-order-to-message-ordering-paradigms.)
* **`[Part B | 8 Marks]`** [**QUESTION 5: Explain Asynchronous Execution with Synchronous Communication (AESC), RSC Executions, and the Crown Criterion.**](./Unit_2_8_Marks.md#question-5-explain-asynchronous-execution-with-synchronous-communication-aesc-rsc-executions-and-the-crown-criterion.)
* **`[Part B | 8 Marks]`** [**QUESTION 6: Explain the Raynal-Schiper-Toueg (RST) Algorithm for Causal Message Ordering.**](./Unit_2_8_Marks.md#question-6-explain-the-raynal-schiper-toueg-rst-algorithm-for-causal-message-ordering.)
* **`[Part B | 8 Marks]`** [**QUESTION 7: Explain Skeen's 3-Phase Algorithm for Total Message Ordering.**](./Unit_2_8_Marks.md#question-7-explain-skeens-3-phase-algorithm-for-total-message-ordering.)
* **`[Part C | S.No 65 | 8M]`** [**QUESTION 8: Explain the Chandy-Lamport Snapshot Recording Algorithm for FIFO Channels.**](./Unit_2_8_Marks.md#question-8-explain-the-chandy-lamport-snapshot-recording-algorithm-for-fifo-channels.)

---

## Unit 2: Global State & Distributed Shared Memory - Part C - 16 Marks
> **Examination Section:** Part C (16 Marks) &nbsp;|&nbsp; **Target File:** [Unit_2_16_Marks.md](./Unit_2_16_Marks.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part C | S.No 65 | 8M]`** [**Question 1: Explain in detail the framework for a system of logical clocks, Lamport's scalar logical clocks, and Mattern-Fidge vector logical clocks. Compare scalar and vector clocks with mathematical properties, space-time diagrams, and complete update traces.**](./Unit_2_16_Marks.md#question-1)
* **`[Part C | S.No 91 | 8M]`** [**Question 2: Discuss physical clock synchronization in distributed systems in detail, covering clock drift, Cristian's algorithm, Berkeley algorithm, and the complete architecture, protocol equations, and stratum hierarchy of NTP.**](./Unit_2_16_Marks.md#question-2)
* **`[Part C | 16 Marks]`** [**Question 3: Discuss message ordering paradigms in distributed group communication in depth. Formulate FIFO, Causal Order, and Total Order, and explain algorithms for Causal Order and Total Order.**](./Unit_2_16_Marks.md#question-3)
* **`[Part B | S.No 60 | 8M]`** [**Question 4: Explain in detail the Chandy-Lamport distributed snapshot algorithm for FIFO channels. Provide the complete system model, marker rules, correctness proof, and a comprehensive end-to-end worked example with channel message states.**](./Unit_2_16_Marks.md#question-4)

---

## Unit 3: Mutual Exclusion & Deadlock Detection - Set 1 (Question Bank) - Part A - 2 Marks
> **Examination Section:** Part A (2 Marks) &nbsp;|&nbsp; **Target File:** [Unit_3_2_Marks.md](./Unit_3_2_Marks.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part A | S.No 46 | 2M]`** [**Q1. What is distributed mutual exclusion, and why is it important in distributed systems?**](./Unit_3_2_Marks.md#q1.-what-is-distributed-mutual-exclusion-and-why-is-it-important-in-distributed-systems)
* **`[Part A | S.No 47 | 2M]`** [**Q2. What is deadlock detection, and why is it necessary in distributed systems?**](./Unit_3_2_Marks.md#q2.-what-is-deadlock-detection-and-why-is-it-necessary-in-distributed-systems)
* **`[Part A | S.No 48 | 2M]`** [**Q3. Why is it important to detect deadlocks in distributed systems?**](./Unit_3_2_Marks.md#q3.-why-is-it-important-to-detect-deadlocks-in-distributed-systems)
* **`[Part A | S.No 49 | 2M]`** [**Q4. How does message passing contribute to deadlock situations in distributed systems?**](./Unit_3_2_Marks.md#q4.-how-does-message-passing-contribute-to-deadlock-situations-in-distributed-systems)
* **`[Part A | S.No 50 | 2M]`** [**Q5. Explain the preliminaries required for understanding distributed mutual exclusion algorithms.**](./Unit_3_2_Marks.md#q5.-explain-the-preliminaries-required-for-understanding-distributed-mutual-exclusion-algorithms.)
* **`[Part A | S.No 51 | 2M]`** [**Q6. What are the prerequisites for understanding deadlock detection algorithms?**](./Unit_3_2_Marks.md#q6.-what-are-the-prerequisites-for-understanding-deadlock-detection-algorithms)
* **`[Part A | S.No 52 | 2M]`** [**Q7. Describe Lamport's algorithm for distributed mutual exclusion.**](./Unit_3_2_Marks.md#q7.-describe-lamports-algorithm-for-distributed-mutual-exclusion.)
* **`[Part A | S.No 53 | 2M]`** [**Q8. Discuss the key features of Ricart-Agrawala's algorithm for distributed mutual exclusion.**](./Unit_3_2_Marks.md#q8.-discuss-the-key-features-of-ricart-agrawalas-algorithm-for-distributed-mutual-exclusion.)
* **`[Part A | S.No 54 | 2M]`** [**Q9. Compare and contrast Lamport's and Ricart-Agrawala's algorithms for mutual exclusion.**](./Unit_3_2_Marks.md#q9.-compare-and-contrast-lamports-and-ricart-agrawalas-algorithms-for-mutual-exclusion.)
* **`[Part A | S.No 55 | 2M]`** [**Q10. Discuss the components of the system model used for deadlock detection.**](./Unit_3_2_Marks.md#q10.-discuss-the-components-of-the-system-model-used-for-deadlock-detection.)
* **`[Part A | S.No 56 | 2M]`** [**Q11. What are the advantages of the Suzuki-Kasami broadcast algorithm for mutual exclusion?**](./Unit_3_2_Marks.md#q11.-what-are-the-advantages-of-the-suzuki-kasami-broadcast-algorithm-for-mutual-exclusion)
* **`[Part A | S.No 57 | 2M]`** [**Q12. How does the Chandy-Misra-Haas algorithm differ in the OR model?**](./Unit_3_2_Marks.md#q12.-how-does-the-chandy-misra-haas-algorithm-differ-in-the-or-model)
* **`[Part A | S.No 58 | 2M]`** [**Q13. Discuss the significance of message ordering in distributed deadlock detection.**](./Unit_3_2_Marks.md#q13.-discuss-the-significance-of-message-ordering-in-distributed-deadlock-detection.)

---

## Unit 3: Mutual Exclusion & Deadlock Detection - Set 1 (Question Bank) - Part B - 8 Marks
> **Examination Section:** Part B (8 Marks) &nbsp;|&nbsp; **Target File:** [Unit_3_8_Marks.md](./Unit_3_8_Marks.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part B | S.No 59 | 8M]`** [**QUESTION 1: Outline Lamport's Algorithm with a Neat Example.**](./Unit_3_8_Marks.md#question-1-outline-lamports-algorithm-with-a-neat-example.)
* **`[Part B | S.No 60 | 8M]`** [**QUESTION 2: Explain Ricart-Agrawala Algorithm with an Example.**](./Unit_3_8_Marks.md#question-2-explain-ricart-agrawala-algorithm-with-an-example.)
* **`[Part B | S.No 61 | 8M]`** [**QUESTION 3: Justify the Drawbacks of Suzuki–Kasami Algorithm.**](./Unit_3_8_Marks.md#question-3-justify-the-drawbacks-of-suzuki–kasami-algorithm.)
* **`[Part C | S.No 62 | 8M]`** [**QUESTION 4: Discuss with a Suitable Example to Show That a Deadlock Cannot Occur If Any One of the Four Coffman Conditions Is Absent.**](./Unit_3_8_Marks.md#question-4-discuss-with-a-suitable-example-to-show-that-a-deadlock-cannot-occur-if-any-one-of-the-four-coffman-conditions-is-absent.)
* **`[Part C | S.No 63 | 8M]`** [**QUESTION 5: Demonstrate the Chandy-Misra-Haas Detection Algorithm.**](./Unit_3_8_Marks.md#question-5-demonstrate-the-chandy-misra-haas-detection-algorithm.)
* **`[Part C | S.No 65 | 8M]`** [**QUESTION 6: Explain the "Snapshot" Algorithm of Lamport (Chandy-Lamport Distributed Snapshot).**](./Unit_3_8_Marks.md#question-6-explain-the-snapshot-algorithm-of-lamport-chandy-lamport-distributed-snapshot.)
* **`[Part C | S.No 68 | 8M]`** [**QUESTION 7: What Is a Deadlock? How Can a Deadlock Be Recovered? Explain Distributed Deadlocks.**](./Unit_3_8_Marks.md#question-7-what-is-a-deadlock-how-can-a-deadlock-be-recovered-explain-distributed-deadlocks.)

---

## Unit 3: Mutual Exclusion & Deadlock Detection - Set 1 (Question Bank) - Part C - 16 Marks
> **Examination Section:** Part C (16 Marks) &nbsp;|&nbsp; **Target File:** [Unit_3_16_Marks.md](./Unit_3_16_Marks.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part C | S.No 64 | 16M]`** [**Question 1: Discuss in detail the requirements that mutual exclusion algorithms should satisfy and discuss what metric we use to measure the performance of mutual exclusion algorithms.**](./Unit_3_16_Marks.md#question-1)
* **`[Part C | S.No 66 | 16M]`** [**Question 2: "External synchronization ensures internal synchronization. But the vice versa does not stand true." Justify. Explain Lamport's algorithm in detail with complete message exchanges and proof of correctness.**](./Unit_3_16_Marks.md#question-2)
* **`[Part C | S.No 67 | 16M]`** [**Question 3: Explain in detail about Suzuki-Kasami's algorithm with its critical section execution with an example.**](./Unit_3_16_Marks.md#question-3)
* **`[Part C | S.No 69 | 16M]`** [**Question 4: How can we achieve deadlock detection in distributed systems? Provide various models to carry out the same.**](./Unit_3_16_Marks.md#question-4)

---

## Unit 3: Mutual Exclusion & Deadlock Detection - Set 2 (Alternate Bank) - Part A - 2 Marks
> **Examination Section:** Part A (2 Marks) &nbsp;|&nbsp; **Target File:** [Unit_3_2_Marks_Set_2.md](./Unit_3_2_Marks_Set_2.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part A | 2 Marks]`** [**Q1. What is a quorum in distributed mutual exclusion?**](./Unit_3_2_Marks_Set_2.md#q1.-what-is-a-quorum-in-distributed-mutual-exclusion)
* **`[Part A | 2 Marks]`** [**Q2. State the two essential conditions that Maekawa's request sets must satisfy.**](./Unit_3_2_Marks_Set_2.md#q2.-state-the-two-essential-conditions-that-maekawas-request-sets-must-satisfy.)
* **`[Part A | 2 Marks]`** [**Q3. Why is Maekawa's algorithm prone to deadlock?**](./Unit_3_2_Marks_Set_2.md#q3.-why-is-maekawas-algorithm-prone-to-deadlock)
* **`[Part A | 2 Marks]`** [**Q4. What is the role of INQUIRE, YIELD, and FAILED messages in Maekawa's algorithm?**](./Unit_3_2_Marks_Set_2.md#q4.-what-is-the-role-of-inquire-yield-and-failed-messages-in-maekawas-algorithm)
* **`[Part A | 2 Marks]`** [**Q5. What is the average message complexity of Raymond's Tree Algorithm?**](./Unit_3_2_Marks_Set_2.md#q5.-what-is-the-average-message-complexity-of-raymonds-tree-algorithm)
* **`[Part A | 2 Marks]`** [**Q6. How does the 'holder' pointer function in Raymond's Tree Algorithm?**](./Unit_3_2_Marks_Set_2.md#q6.-how-does-the-holder-pointer-function-in-raymonds-tree-algorithm)
* **`[Part A | 2 Marks]`** [**Q7. Differentiate between Wait-Die and Wound-Wait deadlock prevention schemes.**](./Unit_3_2_Marks_Set_2.md#q7.-differentiate-between-wait-die-and-wound-wait-deadlock-prevention-schemes.)
* **`[Part A | S.No 47 | 2M]`** [**Q8. What is a phantom deadlock, and why does it occur in distributed systems?**](./Unit_3_2_Marks_Set_2.md#q8.-what-is-a-phantom-deadlock-and-why-does-it-occur-in-distributed-systems)
* **`[Part A | 2 Marks]`** [**Q9. What is a knot in a Wait-For Graph (WFG)?**](./Unit_3_2_Marks_Set_2.md#q9.-what-is-a-knot-in-a-wait-for-graph-wfg)
* **`[Part A | 2 Marks]`** [**Q10. Distinguish between the AND model and the OR model of deadlocks.**](./Unit_3_2_Marks_Set_2.md#q10.-distinguish-between-the-and-model-and-the-or-model-of-deadlocks.)
* **`[Part A | 2 Marks]`** [**Q11. What is an edge-chasing algorithm?**](./Unit_3_2_Marks_Set_2.md#q11.-what-is-an-edge-chasing-algorithm)
* **`[Part A | 2 Marks]`** [**Q12. How does the Mitchell-Merritt algorithm detect deadlocks using public and private labels?**](./Unit_3_2_Marks_Set_2.md#q12.-how-does-the-mitchell-merritt-algorithm-detect-deadlocks-using-public-and-private-labels)
* **`[Part A | 2 Marks]`** [**Q13. State the synchronization delay of Suzuki-Kasami vs. Ricart-Agrawala algorithms.**](./Unit_3_2_Marks_Set_2.md#q13.-state-the-synchronization-delay-of-suzuki-kasami-vs.-ricart-agrawala-algorithms.)

---

## Unit 3: Mutual Exclusion & Deadlock Detection - Set 2 (Alternate Bank) - Part B - 8 Marks
> **Examination Section:** Part B (8 Marks) &nbsp;|&nbsp; **Target File:** [Unit_3_8_Marks_Set_2.md](./Unit_3_8_Marks_Set_2.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part B | S.No 60 | 8M]`** [**QUESTION 1: Explain Maekawa's Quorum-Based Algorithm with an Example and Quorum Requirements.**](./Unit_3_8_Marks_Set_2.md#question-1-explain-maekawas-quorum-based-algorithm-with-an-example-and-quorum-requirements.)
* **`[Part B | 8 Marks]`** [**QUESTION 2: Explain Raymond's Tree-Based Mutual Exclusion Algorithm with a Neat Diagram.**](./Unit_3_8_Marks_Set_2.md#question-2-explain-raymonds-tree-based-mutual-exclusion-algorithm-with-a-neat-diagram.)
* **`[Part B | 8 Marks]`** [**QUESTION 3: Contrast Wait-Die and Wound-Wait Deadlock Prevention Schemes with Examples.**](./Unit_3_8_Marks_Set_2.md#question-3-contrast-wait-die-and-wound-wait-deadlock-prevention-schemes-with-examples.)
* **`[Part B | 8 Marks]`** [**QUESTION 4: Explain Ho-Ramamoorthy Two-Phase Deadlock Detection Algorithm.**](./Unit_3_8_Marks_Set_2.md#question-4-explain-ho-ramamoorthy-two-phase-deadlock-detection-algorithm.)
* **`[Part B | 8 Marks]`** [**QUESTION 5: Explain Mitchell-Merritt Edge-Chasing Deadlock Detection Algorithm.**](./Unit_3_8_Marks_Set_2.md#question-5-explain-mitchell-merritt-edge-chasing-deadlock-detection-algorithm.)
* **`[Part B | 8 Marks]`** [**QUESTION 6: Discuss How Message Ordering Affects Deadlock Detection and Causes Phantom Deadlocks.**](./Unit_3_8_Marks_Set_2.md#question-6-discuss-how-message-ordering-affects-deadlock-detection-and-causes-phantom-deadlocks.)
* **`[Part B | 8 Marks]`** [**QUESTION 7: Compare Token-Based vs. Non-Token-Based Mutual Exclusion Algorithms.**](./Unit_3_8_Marks_Set_2.md#question-7-compare-token-based-vs.-non-token-based-mutual-exclusion-algorithms.)

---

## Unit 3: Mutual Exclusion & Deadlock Detection - Set 2 (Alternate Bank) - Part C - 16 Marks
> **Examination Section:** Part C (16 Marks) &nbsp;|&nbsp; **Target File:** [Unit_3_16_Marks_Set_2.md](./Unit_3_16_Marks_Set_2.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part C | 16 Marks]`** [**Q1. Deep Architectural and Mathematical Analysis of Maekawa's Voting/Quorum Algorithm: Finite Projective Plane Construction, Algorithmic Protocol, Deadlock Vulnerability, Auxiliary Message Resolution (Inquire, Yield, Failed), and Correctness Proof.**](./Unit_3_16_Marks_Set_2.md#q1.-deep-architectural-and-mathematical-analysis-of-maekawas-votingquorum-algorithm-finite-projective-plane-construction-algorithmic-protocol-deadlock-vulnerability-auxiliary-message-resolution-inquire-yield-failed-and-correctness-proof.)
* **`[Part C | 16 Marks]`** [**Q2. Architectural and Algorithmic Study of Raymond's Tree-Based Token Algorithm: Logical Spanning Tree Topology, Data Structures, Message Flow, Starvation-Freedom, and Proof of Correctness.**](./Unit_3_16_Marks_Set_2.md#q2.-architectural-and-algorithmic-study-of-raymonds-tree-based-token-algorithm-logical-spanning-tree-topology-data-structures-message-flow-starvation-freedom-and-proof-of-correctness.)
* **`[Part C | 16 Marks]`** [**Q3. Priority-Based Distributed Deadlock Prevention Schemes: Theoretical Foundations, Algorithmic Rules of Wait-Die vs. Wound-Wait, Execution Traces, and Exhaustive Comparison.**](./Unit_3_16_Marks_Set_2.md#q3.-priority-based-distributed-deadlock-prevention-schemes-theoretical-foundations-algorithmic-rules-of-wait-die-vs.-wound-wait-execution-traces-and-exhaustive-comparison.)
* **`[Part C | 16 Marks]`** [**Q4. Centralized and Hierarchical Distributed Deadlock Detection: Ho-Ramamoorthy Two-Phase and One-Phase Algorithms, and the Menasce-Muntz Hierarchical Algorithm.**](./Unit_3_16_Marks_Set_2.md#q4.-centralized-and-hierarchical-distributed-deadlock-detection-ho-ramamoorthy-two-phase-and-one-phase-algorithms-and-the-menasce-muntz-hierarchical-algorithm.)

---

## Unit 4: Consensus & Recovery - Set 1 (Question Bank) - Part A - 2 Marks
> **Examination Section:** Part A (2 Marks) &nbsp;|&nbsp; **Target File:** [Unit_4_2_Marks.md](./Unit_4_2_Marks.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part A | S.No 70 | 2M]`** [**Q1. Define consensus and its importance in distributed systems.**](./Unit_4_2_Marks.md#q1.-define-consensus-and-its-importance-in-distributed-systems.)
* **`[Part A | S.No 71 | 2M]`** [**Q2. Recall the key components of Lamport's algorithm for mutual exclusion.**](./Unit_4_2_Marks.md#q2.-recall-the-key-components-of-lamports-algorithm-for-mutual-exclusion.)
* **`[Part A | S.No 72 | 2M]`** [**Q3. Can you remember the basic steps of the Chandy-Misra-Haas algorithm for deadlock detection?**](./Unit_4_2_Marks.md#q3.-can-you-remember-the-basic-steps-of-the-chandy-misra-haas-algorithm-for-deadlock-detection)
* **`[Part A | S.No 73 | 2M]`** [**Q4. List the different models of deadlocks in distributed systems.**](./Unit_4_2_Marks.md#q4.-list-the-different-models-of-deadlocks-in-distributed-systems.)
* **`[Part A | S.No 74 | 2M]`** [**Q5. Explain why consensus is difficult to achieve in distributed systems.**](./Unit_4_2_Marks.md#q5.-explain-why-consensus-is-difficult-to-achieve-in-distributed-systems.)
* **`[Part A | S.No 75 | 2M]`** [**Q6. Can you describe the process of coordinated checkpointing for failure recovery?**](./Unit_4_2_Marks.md#q6.-can-you-describe-the-process-of-coordinated-checkpointing-for-failure-recovery)
* **`[Part A | S.No 76 | 2M]`** [**Q7. What issues arise in failure recovery in distributed systems?**](./Unit_4_2_Marks.md#q7.-what-issues-arise-in-failure-recovery-in-distributed-systems)
* **`[Part A | S.No 77 | 2M]`** [**Q8. What distinguishes synchronous systems with failures from synchronous failure-free systems?**](./Unit_4_2_Marks.md#q8.-what-distinguishes-synchronous-systems-with-failures-from-synchronous-failure-free-systems)
* **`[Part A | S.No 78 | 2M]`** [**Q9. Apply Lamport's algorithm to solve mutual exclusion in a distributed system scenario.**](./Unit_4_2_Marks.md#q9.-apply-lamports-algorithm-to-solve-mutual-exclusion-in-a-distributed-system-scenario.)
* **`[Part A | S.No 79 | 2M]`** [**Q10. Design an algorithm for asynchronous checkpointing and recovery in distributed systems.**](./Unit_4_2_Marks.md#q10.-design-an-algorithm-for-asynchronous-checkpointing-and-recovery-in-distributed-systems.)
* **`[Part A | S.No 80 | 2M]`** [**Q11. Apply checkpoint-based recovery to restore a distributed system after a failure.**](./Unit_4_2_Marks.md#q11.-apply-checkpoint-based-recovery-to-restore-a-distributed-system-after-a-failure.)
* **`[Part A | S.No 81 | 2M]`** [**Q12. Analyze the challenges involved in achieving consensus in distributed systems.**](./Unit_4_2_Marks.md#q12.-analyze-the-challenges-involved-in-achieving-consensus-in-distributed-systems.)

---

## Unit 4: Consensus & Recovery - Set 1 (Question Bank) - Part B - 8 Marks
> **Examination Section:** Part B (8 Marks) &nbsp;|&nbsp; **Target File:** [Unit_4_8_Marks.md](./Unit_4_8_Marks.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part B | S.No 82 | 8M]`** [**QUESTION 1: Illustrate the Different Types of Failure in Distributed Systems and Explain How to Prevent/Tolerate Them.**](./Unit_4_8_Marks.md#question-1-illustrate-the-different-types-of-failure-in-distributed-systems-and-explain-how-to-preventtolerate-them.)
* **`[Part B | S.No 83 | 8M]`** [**QUESTION 2: Illustrate Briefly the Two Kinds of Checkpoints for Checkpoint Algorithm.**](./Unit_4_8_Marks.md#question-2-illustrate-briefly-the-two-kinds-of-checkpoints-for-checkpoint-algorithm.)
* **`[Part B | S.No 84 | 8M]`** [**QUESTION 3: Construct Byzantine Agreement Problem in Distributed System.**](./Unit_4_8_Marks.md#question-3-construct-byzantine-agreement-problem-in-distributed-system.)
* **`[Part B | S.No 85 | 8M]`** [**QUESTION 4: Comparison of Agreement Protocols in Distributed Systems.**](./Unit_4_8_Marks.md#question-4-comparison-of-agreement-protocols-in-distributed-systems.)
* **`[Part C | S.No 86 | 8M]`** [**QUESTION 5: Explain the Agreement Statement That Should Be Followed in Synchronous System with Failure.**](./Unit_4_8_Marks.md#question-5-explain-the-agreement-statement-that-should-be-followed-in-synchronous-system-with-failure.)
* **`[Part C | S.No 87 | 8M]`** [**QUESTION 6: Construct the Background and Definitions of System Model for Distributed Recovery.**](./Unit_4_8_Marks.md#question-6-construct-the-background-and-definitions-of-system-model-for-distributed-recovery.)
* **`[Part B | S.No 85 | 8M]`** [**QUESTION 7: Classification of Agreement Protocols in Distributed Systems.**](./Unit_4_8_Marks.md#question-7-classification-of-agreement-protocols-in-distributed-systems.)
* **`[Part B | 8 Marks]`** [**QUESTION 8: Elaborate Checkpointing and Rollback Recovery.**](./Unit_4_8_Marks.md#question-8-elaborate-checkpointing-and-rollback-recovery.)

---

## Unit 4: Consensus & Recovery - Set 1 (Question Bank) - Part C - 16 Marks
> **Examination Section:** Part C (16 Marks) &nbsp;|&nbsp; **Target File:** [Unit_4_16_Marks.md](./Unit_4_16_Marks.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part C | S.No 88 | 16M]`** [**Question 1: What are the key assumptions underlying while designing agreement algorithms and brief them in detail.**](./Unit_4_16_Marks.md#question-1)
* **`[Part C | S.No 89 | 16M]`** [**Question 2: Discuss the issues in failure recovery with an example.**](./Unit_4_16_Marks.md#question-2)
* **`[Part C | S.No 90 | 16M]`** [**Question 3: Write about the issues in failure recovery and discuss about any two recovery mechanisms.**](./Unit_4_16_Marks.md#question-3)
* **`[Part C | S.No 93 | 16M]`** [**Question 4: Justify the Checkpoint-based recovery in Distributed Systems.**](./Unit_4_16_Marks.md#question-4)

---

## Unit 4: Consensus & Recovery - Set 2 (Alternate Bank) - Part A - 2 Marks
> **Examination Section:** Part A (2 Marks) &nbsp;|&nbsp; **Target File:** [Unit_4_2_Marks_Set_2.md](./Unit_4_2_Marks_Set_2.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part A | 2 Marks]`** [**Q1. What is the fundamental difference between Two-Phase Commit (2PC) and Three-Phase Commit (3PC)?**](./Unit_4_2_Marks_Set_2.md#q1.-what-is-the-fundamental-difference-between-two-phase-commit-2pc-and-three-phase-commit-3pc)
* **`[Part A | 2 Marks]`** [**Q2. What are the three roles in the Paxos consensus algorithm?**](./Unit_4_2_Marks_Set_2.md#q2.-what-are-the-three-roles-in-the-paxos-consensus-algorithm)
* **`[Part A | 2 Marks]`** [**Q3. State the Leader Completeness property in Raft.**](./Unit_4_2_Marks_Set_2.md#q3.-state-the-leader-completeness-property-in-raft.)
* **`[Part A | 2 Marks]`** [**Q4. What is a Z-path in distributed checkpointing?**](./Unit_4_2_Marks_Set_2.md#q4.-what-is-a-z-path-in-distributed-checkpointing)
* **`[Part A | 2 Marks]`** [**Q5. What is a Z-cycle, and what is its significance?**](./Unit_4_2_Marks_Set_2.md#q5.-what-is-a-z-cycle-and-what-is-its-significance)
* **`[Part A | 2 Marks]`** [**Q6. Define the Piecewise Deterministic (PWD) assumption in message logging.**](./Unit_4_2_Marks_Set_2.md#q6.-define-the-piecewise-deterministic-pwd-assumption-in-message-logging.)
* **`[Part A | 2 Marks]`** [**Q7. What is an orphan process in message logging?**](./Unit_4_2_Marks_Set_2.md#q7.-what-is-an-orphan-process-in-message-logging)
* **`[Part A | 2 Marks]`** [**Q8. How does pessimistic logging prevent the creation of orphan processes?**](./Unit_4_2_Marks_Set_2.md#q8.-how-does-pessimistic-logging-prevent-the-creation-of-orphan-processes)
* **`[Part A | 2 Marks]`** [**Q9. What is a recovery line?**](./Unit_4_2_Marks_Set_2.md#q9.-what-is-a-recovery-line)
* **`[Part A | 2 Marks]`** [**Q10. State the minimum number of nodes required to tolerate m Byzantine faults using digital signatures.**](./Unit_4_2_Marks_Set_2.md#q10.-state-the-minimum-number-of-nodes-required-to-tolerate-m-byzantine-faults-using-digital-signatures.)
* **`[Part A | 2 Marks]`** [**Q11. What is Atomic Broadcast?**](./Unit_4_2_Marks_Set_2.md#q11.-what-is-atomic-broadcast)
* **`[Part A | 2 Marks]`** [**Q12. Differentiate between uncoordinated and coordinated checkpointing.**](./Unit_4_2_Marks_Set_2.md#q12.-differentiate-between-uncoordinated-and-coordinated-checkpointing.)

---

## Unit 4: Consensus & Recovery - Set 2 (Alternate Bank) - Part B - 8 Marks
> **Examination Section:** Part B (8 Marks) &nbsp;|&nbsp; **Target File:** [Unit_4_8_Marks_Set_2.md](./Unit_4_8_Marks_Set_2.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part B | 8 Marks]`** [**Q1. Explain the Paxos Consensus Algorithm with Phase 1 and Phase 2 Message Exchanges, State Variables, and Majority Quorum Mechanics.**](./Unit_4_8_Marks_Set_2.md#q1.-explain-the-paxos-consensus-algorithm-with-phase-1-and-phase-2-message-exchanges-state-variables-and-majority-quorum-mechanics.)
* **`[Part B | 8 Marks]`** [**Q2. Explain the Raft Consensus Algorithm Focusing on Leader Election and Log Replication with Step-by-Step State Diagrams.**](./Unit_4_8_Marks_Set_2.md#q2.-explain-the-raft-consensus-algorithm-focusing-on-leader-election-and-log-replication-with-step-by-step-state-diagrams.)
* **`[Part B | 8 Marks]`** [**Q3. Compare Two-Phase Commit (2PC) and Three-Phase Commit (3PC) in Detail with State Transition Diagrams and Crash Recovery Analysis.**](./Unit_4_8_Marks_Set_2.md#q3.-compare-two-phase-commit-2pc-and-three-phase-commit-3pc-in-detail-with-state-transition-diagrams-and-crash-recovery-analysis.)
* **`[Part B | 8 Marks]`** [**Q4. Explain Communication-Induced Checkpointing (CIC) Along with the Concepts of Z-paths, Z-cycles, and How Z-cycles Induce the Domino Effect.**](./Unit_4_8_Marks_Set_2.md#q4.-explain-communication-induced-checkpointing-cic-along-with-the-concepts-of-z-paths-z-cycles-and-how-z-cycles-induce-the-domino-effect.)
* **`[Part B | 8 Marks]`** [**Q5. Classify and Analyze Message Logging Protocols: Pessimistic, Optimistic, and Causal Logging Under the Piecewise Deterministic (PWD) Model.**](./Unit_4_8_Marks_Set_2.md#q5.-classify-and-analyze-message-logging-protocols-pessimistic-optimistic-and-causal-logging-under-the-piecewise-deterministic-pwd-model.)
* **`[Part B | 8 Marks]`** [**Q6. Describe the Byzantine Agreement Oral Messages Algorithm $OM(m)$ with an Illustrative 4-General Execution Trace ($n=4, m=1$).**](./Unit_4_8_Marks_Set_2.md#q6.-describe-the-byzantine-agreement-oral-messages-algorithm-$omm$-with-an-illustrative-4-general-execution-trace-$n=4-m=1$.)

---

## Unit 4: Consensus & Recovery - Set 2 (Alternate Bank) - Part C - 16 Marks
> **Examination Section:** Part C (16 Marks) &nbsp;|&nbsp; **Target File:** [Unit_4_16_Marks_Set_2.md](./Unit_4_16_Marks_Set_2.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part C | S.No 87 | 8M]`** [**Q1. Comprehensive Architectural and Formal Analysis of the Paxos Consensus Protocol: System Model, Roles, Multi-Phase Message Exchanges, Quorum Intersection Proof of Safety, Liveness Solutions, and Multi-Paxos Optimization.**](./Unit_4_16_Marks_Set_2.md#q1.-comprehensive-architectural-and-formal-analysis-of-the-paxos-consensus-protocol-system-model-roles-multi-phase-message-exchanges-quorum-intersection-proof-of-safety-liveness-solutions-and-multi-paxos-optimization.)
* **`[Part C | 16 Marks]`** [**Q2. In-Depth Architectural Study of the Raft Distributed Consensus Protocol: Server States, Leader Election Mechanism, Log Replication & Consistency Guarantee, Joint Consensus for Dynamic Membership Changes, and Formal Verification of the Five Core Safety Invariants.**](./Unit_4_16_Marks_Set_2.md#q2.-in-depth-architectural-study-of-the-raft-distributed-consensus-protocol-server-states-leader-election-mechanism-log-replication-&-consistency-guarantee-joint-consensus-for-dynamic-membership-changes-and-formal-verification-of-the-five-core-safety-invariants.)
* **`[Part C | 16 Marks]`** [**Q3. Distributed Commit Protocols: End-to-End Architectural and State Analysis of Two-Phase Commit (2PC) vs. Three-Phase Commit (3PC), Coordinator Failure Handling, Cooperative Termination Protocols, and Optimizations.**](./Unit_4_16_Marks_Set_2.md#q3.-distributed-commit-protocols-end-to-end-architectural-and-state-analysis-of-two-phase-commit-2pc-vs.-three-phase-commit-3pc-coordinator-failure-handling-cooperative-termination-protocols-and-optimizations.)
* **`[Part C | 16 Marks]`** [**Q4. Formal Foundations of Distributed Rollback Recovery: Coordinated vs. Uncoordinated Checkpointing, The Domino Effect, Netzer-Xu Z-Path & Z-Cycle Theory, Communication-Induced Checkpointing (HMNR), and Message Logging Architectures.**](./Unit_4_16_Marks_Set_2.md#q4.-formal-foundations-of-distributed-rollback-recovery-coordinated-vs.-uncoordinated-checkpointing-the-domino-effect-netzer-xu-z-path-&-z-cycle-theory-communication-induced-checkpointing-hmnr-and-message-logging-architectures.)

---

## Unit 5: Cloud Computing - Set 1 (Question Bank) - Part A - 2 Marks
> **Examination Section:** Part A (2 Marks) &nbsp;|&nbsp; **Target File:** [Unit_5_2_Marks.md](./Unit_5_2_Marks.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part A | S.No 94 | 2M]`** [**Q1. Name the different cloud deployment models.**](./Unit_5_2_Marks.md#q1.-name-the-different-cloud-deployment-models.)
* **`[Part A | S.No 95 | 2M]`** [**Q2. Recall the driving factors behind the adoption of cloud computing.**](./Unit_5_2_Marks.md#q2.-recall-the-driving-factors-behind-the-adoption-of-cloud-computing.)
* **`[Part A | S.No 96 | 2M]`** [**Q3. Define virtualization and its role in cloud computing.**](./Unit_5_2_Marks.md#q3.-define-virtualization-and-its-role-in-cloud-computing.)
* **`[Part A | S.No 97 | 2M]`** [**Q4. What are the key attributes of scalability and elasticity in the context of cloud computing?**](./Unit_5_2_Marks.md#q4.-what-are-the-key-attributes-of-scalability-and-elasticity-in-the-context-of-cloud-computing)
* **`[Part A | S.No 98 | 2M]`** [**Q5. How do different cloud deployment models differ in terms of resource ownership and management?**](./Unit_5_2_Marks.md#q5.-how-do-different-cloud-deployment-models-differ-in-terms-of-resource-ownership-and-management)
* **`[Part A | S.No 99 | 2M]`** [**Q6. Can you explain why security is a significant challenge in cloud computing?**](./Unit_5_2_Marks.md#q6.-can-you-explain-why-security-is-a-significant-challenge-in-cloud-computing)
* **`[Part A | S.No 100 | 2M]`** [**Q7. How does load balancing contribute to optimizing resource utilization in cloud computing?**](./Unit_5_2_Marks.md#q7.-how-does-load-balancing-contribute-to-optimizing-resource-utilization-in-cloud-computing)
* **`[Part A | S.No 101 | 2M]`** [**Q8. Describe the role of replication in ensuring data availability and reliability in the cloud.**](./Unit_5_2_Marks.md#q8.-describe-the-role-of-replication-in-ensuring-data-availability-and-reliability-in-the-cloud.)
* **`[Part A | S.No 102 | 2M]`** [**Q9. Design a load balancing strategy for distributing incoming requests across multiple cloud servers.**](./Unit_5_2_Marks.md#q9.-design-a-load-balancing-strategy-for-distributing-incoming-requests-across-multiple-cloud-servers.)
* **`[Part A | S.No 103 | 2M]`** [**Q10. Develop a scalability plan for a cloud-based application to accommodate fluctuating user demands.**](./Unit_5_2_Marks.md#q10.-develop-a-scalability-plan-for-a-cloud-based-application-to-accommodate-fluctuating-user-demands.)
* **`[Part A | S.No 104 | 2M]`** [**Q11. Configure monitoring tools to track resource usage and performance metrics in a cloud infrastructure.**](./Unit_5_2_Marks.md#q11.-configure-monitoring-tools-to-track-resource-usage-and-performance-metrics-in-a-cloud-infrastructure.)
* **`[Part A | S.No 105 | 2M]`** [**Q12. Analyze the impact of scalability on the cost-effectiveness of cloud solutions.**](./Unit_5_2_Marks.md#q12.-analyze-the-impact-of-scalability-on-the-cost-effectiveness-of-cloud-solutions.)

---

## Unit 5: Cloud Computing - Set 1 (Question Bank) - Part B - 8 Marks
> **Examination Section:** Part B (8 Marks) &nbsp;|&nbsp; **Target File:** [Unit_5_8_Marks.md](./Unit_5_8_Marks.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part B | S.No 106 | 8M]`** [**QUESTION 1: Demonstrate the Cloud Computing Architecture.**](./Unit_5_8_Marks.md#question-1-demonstrate-the-cloud-computing-architecture.)
* **`[Part B | S.No 107 | 8M]`** [**QUESTION 2: Analyze the Cloud Computing Technologies.**](./Unit_5_8_Marks.md#question-2-analyze-the-cloud-computing-technologies.)
* **`[Part B | S.No 108 | 8M]`** [**QUESTION 3: Difference between Public Cloud, Private Cloud, Hybrid Cloud, and Community Cloud.**](./Unit_5_8_Marks.md#question-3-difference-between-public-cloud-private-cloud-hybrid-cloud-and-community-cloud.)
* **`[Part B | S.No 109 | 8M]`** [**QUESTION 4: Construct the Cloud Deployment Model.**](./Unit_5_8_Marks.md#question-4-construct-the-cloud-deployment-model.)
* **`[Part B | S.No 110 | 8M]`** [**QUESTION 5: Contrast between AWS, Azure, and Google Cloud Platform (GCP).**](./Unit_5_8_Marks.md#question-5-contrast-between-aws-azure-and-google-cloud-platform-gcp.)
* **`[Part B | S.No 111 | 8M]`** [**QUESTION 6: Elaborate the Scaling in Cloud Computing.**](./Unit_5_8_Marks.md#question-6-elaborate-the-scaling-in-cloud-computing.)

---

## Unit 5: Cloud Computing - Set 1 (Question Bank) - Part C - 16 Marks
> **Examination Section:** Part C (16 Marks) &nbsp;|&nbsp; **Target File:** [Unit_5_16_Marks.md](./Unit_5_16_Marks.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part C | S.No 112 | 16M]`** [**Question 1: Originate of Cloud Computing.**](./Unit_5_16_Marks.md#question-1)
* **`[Part C | S.No 113 | 16M]`** [**Question 2: Explain the Components of Cloud Computing Architecture in detail.**](./Unit_5_16_Marks.md#question-2)
* **`[Part C | S.No 114 | 16M]`** [**Question 3: Classify the Different types of Cloud Computing (Deployment Models).**](./Unit_5_16_Marks.md#question-3)
* **`[Part C | S.No 115 | 16M]`** [**Question 4: Classify the Cloud Service Models in detail.**](./Unit_5_16_Marks.md#question-4)
* **`[Part C | S.No 116 | 16M]`** [**Question 5: Categorize of the Cloud Service Provider Companies.**](./Unit_5_16_Marks.md#question-5)

---

## Unit 5: Cloud Computing - Set 2 (Alternate Bank) - Part A - 2 Marks
> **Examination Section:** Part A (2 Marks) &nbsp;|&nbsp; **Target File:** [Unit_5_2_Marks_Set_2.md](./Unit_5_2_Marks_Set_2.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part A | 2 Marks]`** [**Q1. What is the fundamental difference between Type 1 and Type 2 Hypervisors?**](./Unit_5_2_Marks_Set_2.md#q1.-what-is-the-fundamental-difference-between-type-1-and-type-2-hypervisors)
* **`[Part A | 2 Marks]`** [**Q2. Define Paravirtualization and state how it differs from Full Virtualization.**](./Unit_5_2_Marks_Set_2.md#q2.-define-paravirtualization-and-state-how-it-differs-from-full-virtualization.)
* **`[Part A | 2 Marks]`** [**Q3. How do Linux Namespaces and Control Groups (cgroups) enable Containerization?**](./Unit_5_2_Marks_Set_2.md#q3.-how-do-linux-namespaces-and-control-groups-cgroups-enable-containerization)
* **`[Part A | 2 Marks]`** [**Q4. Differentiate between Block Storage and Object Storage in cloud architectures.**](./Unit_5_2_Marks_Set_2.md#q4.-differentiate-between-block-storage-and-object-storage-in-cloud-architectures.)
* **`[Part A | 2 Marks]`** [**Q5. What is the role of the Secondary NameNode in Hadoop HDFS?**](./Unit_5_2_Marks_Set_2.md#q5.-what-is-the-role-of-the-secondary-namenode-in-hadoop-hdfs)
* **`[Part A | 2 Marks]`** [**Q6. Explain the function of the Combiner in the MapReduce execution pipeline.**](./Unit_5_2_Marks_Set_2.md#q6.-explain-the-function-of-the-combiner-in-the-mapreduce-execution-pipeline.)
* **`[Part A | 2 Marks]`** [**Q7. Distinguish between Horizontal Auto-scaling and Vertical Auto-scaling.**](./Unit_5_2_Marks_Set_2.md#q7.-distinguish-between-horizontal-auto-scaling-and-vertical-auto-scaling.)
* **`[Part A | 2 Marks]`** [**Q8. What is a Virtual Machine Escape (VM Escape) attack?**](./Unit_5_2_Marks_Set_2.md#q8.-what-is-a-virtual-machine-escape-vm-escape-attack)
* **`[Part A | 2 Marks]`** [**Q9. Differentiate between Layer 4 (L4) and Layer 7 (L7) Cloud Load Balancers.**](./Unit_5_2_Marks_Set_2.md#q9.-differentiate-between-layer-4-l4-and-layer-7-l7-cloud-load-balancers.)
* **`[Part A | 2 Marks]`** [**Q10. Define Service Level Objective (SLO) versus Service Level Agreement (SLA).**](./Unit_5_2_Marks_Set_2.md#q10.-define-service-level-objective-slo-versus-service-level-agreement-sla.)
* **`[Part A | 2 Marks]`** [**Q11. How does Consistent Hashing minimize data movement in cloud distributed caches?**](./Unit_5_2_Marks_Set_2.md#q11.-how-does-consistent-hashing-minimize-data-movement-in-cloud-distributed-caches)
* **`[Part A | 2 Marks]`** [**Q12. What is Multi-Tenancy in cloud computing, and what security challenge does it introduce?**](./Unit_5_2_Marks_Set_2.md#q12.-what-is-multi-tenancy-in-cloud-computing-and-what-security-challenge-does-it-introduce)

---

## Unit 5: Cloud Computing - Set 2 (Alternate Bank) - Part B - 8 Marks
> **Examination Section:** Part B (8 Marks) &nbsp;|&nbsp; **Target File:** [Unit_5_8_Marks_Set_2.md](./Unit_5_8_Marks_Set_2.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part C | S.No 112 | 16M]`** [**Q1. Explain the Taxonomy of Virtualization in Cloud Computing: Full Virtualization, Paravirtualization, and Hardware-Assisted Virtualization (Intel VT-x / AMD-V).**](./Unit_5_8_Marks_Set_2.md#q1.-explain-the-taxonomy-of-virtualization-in-cloud-computing-full-virtualization-paravirtualization-and-hardware-assisted-virtualization-intel-vt-x-amd-v.)
* **`[Part B | 8 Marks]`** [**Q2. Explain Containerization and Operating System-Level Virtualization: Architecture, Docker Engine Workflow, and Comparison with Traditional Hypervisor VMs.**](./Unit_5_8_Marks_Set_2.md#q2.-explain-containerization-and-operating-system-level-virtualization-architecture-docker-engine-workflow-and-comparison-with-traditional-hypervisor-vms.)
* **`[Part B | 8 Marks]`** [**Q3. Analyze Cloud Storage Models: Block Storage, File Storage, and Object Storage with Architectural Comparisons and Use-Case Analysis.**](./Unit_5_8_Marks_Set_2.md#q3.-analyze-cloud-storage-models-block-storage-file-storage-and-object-storage-with-architectural-comparisons-and-use-case-analysis.)
* **`[Part B | 8 Marks]`** [**Q4. Explain Cloud Load Balancing Algorithms: Layer 4 vs Layer 7, Round Robin, Weighted Round Robin, Least Connections, and Consistent Hashing.**](./Unit_5_8_Marks_Set_2.md#q4.-explain-cloud-load-balancing-algorithms-layer-4-vs-layer-7-round-robin-weighted-round-robin-least-connections-and-consistent-hashing.)
* **`[Part B | 8 Marks]`** [**Q5. Describe the MapReduce Programming Model: Architecture, Map Phase, Shuffle & Sort Phase, Reducer, Combiner, and Partitioner Mechanisms.**](./Unit_5_8_Marks_Set_2.md#q5.-describe-the-mapreduce-programming-model-architecture-map-phase-shuffle-&-sort-phase-reducer-combiner-and-partitioner-mechanisms.)
* **`[Part B | 8 Marks]`** [**Q6. Analyze Cloud Security Threats in Multi-Tenant Environments: Hypervisor Vulnerabilities, VM Escape, Side-Channel Attacks, and Data Protection Mechanisms.**](./Unit_5_8_Marks_Set_2.md#q6.-analyze-cloud-security-threats-in-multi-tenant-environments-hypervisor-vulnerabilities-vm-escape-side-channel-attacks-and-data-protection-mechanisms.)

---

## Unit 5: Cloud Computing - Set 2 (Alternate Bank) - Part C - 16 Marks
> **Examination Section:** Part C (16 Marks) &nbsp;|&nbsp; **Target File:** [Unit_5_16_Marks_Set_2.md](./Unit_5_16_Marks_Set_2.md) &nbsp;|&nbsp; [↑ Back to Navigation Table](#quick-navigation-table)

* **`[Part A | S.No 96 | 2M]`** [**Q1. End-to-End Architectural Study of Virtualization in Cloud Computing: Hardware Virtualization (Full, Para, Hardware-Assisted VT-x/AMD-V), Memory & I/O Virtualization (EPT/NPT, SR-IOV), Hypervisor Models (Type 1 vs Type 2), and Modern Containerization (Namespaces, Cgroups, UnionFS, OCI/Docker).**](./Unit_5_16_Marks_Set_2.md#q1.-end-to-end-architectural-study-of-virtualization-in-cloud-computing-hardware-virtualization-full-para-hardware-assisted-vt-xamd-v-memory-&-io-virtualization-eptnpt-sr-iov-hypervisor-models-type-1-vs-type-2-and-modern-containerization-namespaces-cgroups-unionfs-ocidocker.)
* **`[Part C | 16 Marks]`** [**Q2. Deep Architectural Analysis of Cloud Storage Systems: Technical Taxonomy (Block, File, Object), Distributed Object Store Architecture (AWS S3 / Ceph), Erasure Coding vs Multi-Way Replication, Consistency Models, and Metadata Management.**](./Unit_5_16_Marks_Set_2.md#q2.-deep-architectural-analysis-of-cloud-storage-systems-technical-taxonomy-block-file-object-distributed-object-store-architecture-aws-s3-ceph-erasure-coding-vs-multi-way-replication-consistency-models-and-metadata-management.)
* **`[Part C | 16 Marks]`** [**Q3. Distributed Resource Management and High-Availability Scheduling in the Cloud: Layer 4 vs Layer 7 Load Balancing, Consistent Hashing with Virtual Nodes, Dynamic Autoscaling Architectures, and Formal SLA/SLO Frameworks.**](./Unit_5_16_Marks_Set_2.md#q3.-distributed-resource-management-and-high-availability-scheduling-in-the-cloud-layer-4-vs-layer-7-load-balancing-consistent-hashing-with-virtual-nodes-dynamic-autoscaling-architectures-and-formal-slaslo-frameworks.)
* **`[Part C | 16 Marks]`** [**Q4. Large-Scale Distributed Cloud Data Processing with Hadoop and MapReduce: Master-Worker Architecture, HDFS High Availability (NameNode, DataNode, JournalNodes, ZKFC), MapReduce Execution Engine, Failure Handling, and Speculative Execution.**](./Unit_5_16_Marks_Set_2.md#q4.-large-scale-distributed-cloud-data-processing-with-hadoop-and-mapreduce-master-worker-architecture-hdfs-high-availability-namenode-datanode-journalnodes-zkfc-mapreduce-execution-engine-failure-handling-and-speculative-execution.)

---
