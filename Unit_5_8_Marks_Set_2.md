# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT V: CLOUD COMPUTING
### PART B: 8-MARK QUESTIONS AND DETAILED ANSWERS (SET 2 - ALTERNATE QUESTION BANK)

---

### Q1. Explain the Taxonomy of Virtualization in Cloud Computing: Full Virtualization, Paravirtualization, and Hardware-Assisted Virtualization (Intel VT-x / AMD-V).

#### 1. Conceptual Foundation
Virtualization abstracts underlying physical hardware into multiple isolated logical execution environments known as Virtual Machines (VMs). In traditional x86 architecture, the CPU operates in four rings:
* **Ring 0 (Kernel Mode):** Unrestricted access to hardware instructions (privileged).
* **Ring 1 & 2:** Device drivers (rarely used).
* **Ring 3 (User Mode):** Application programs (unprivileged).
* **Popek-Goldberg Virtualization Requirement:** A virtualization architecture is classically virtualizable if and only if all sensitive instructions (instructions that modify hardware state or behave differently in user mode) are a strict subset of privileged instructions (instructions that trap when executed in Ring 3).

```
   RING 3: User Applications
   RING 2: (Unused / Device Drivers)
   RING 1: (Unused / Paravirtualized Guest OS)
   RING 0: Host OS / Hypervisor (Privileged execution)
```

#### 2. Taxonomy of Hardware Virtualization Techniques

```
                         Hardware Virtualization Techniques
                                         |
         +-------------------------------+-------------------------------+
         |                               |                               |
  Full Virtualization             Paravirtualization            Hardware-Assisted
  (Binary Translation,           (Hypercalls, Modified          (Intel VT-x, AMD-V,
   Unmodified Guest OS)           Guest OS Kernel)               Root/Non-Root Rings)
```

##### 1. Full Virtualization (Binary Translation + Direct Execution)
* **Principle:** The guest operating system is completely unaware it is virtualized and runs **completely unmodified**.
* **Mechanism:**
  * Non-sensitive instructions from guest user applications execute directly on physical CPU at native speed (Direct Execution).
  * Sensitive, unprivileged instructions in Ring 0 are intercepted at runtime by the hypervisor and translated dynamically into safe instruction sequences (**Binary Translation**).
* **Pros/Cons:** Maximum portability (runs legacy Windows/Linux kernels out-of-the-box), but incurs high runtime CPU emulation overhead.

##### 2. Paravirtualization (Hypercalls)
* **Principle:** The guest OS kernel is explicitly modified to be aware that it runs inside a virtualized environment.
* **Mechanism:**
  * Sensitive instructions in guest OS are replaced at compile time with direct software function calls (**Hypercalls**) to the hypervisor API.
  * Similar to system calls (syscalls) made by applications to an OS kernel.
* **Pros/Cons:** Superior near-native execution performance; however, guest OS cannot run unmodified (closed-source OS like proprietary Windows requires special PV drivers).

##### 3. Hardware-Assisted Virtualization (Intel VT-x / AMD-V)
* **Principle:** Hardware CPU manufacturers introduced dedicated virtualization silicon modes directly on chip.
* **Mechanism:**
  * Introduces two hardware operating modes: **VMX Root Operation** (Hypervisor) and **VMX Non-Root Operation** (Guest OS).
  * Both modes support all Rings 0 through 3.
  * When a guest OS in VMX Non-Root Ring 0 executes a sensitive instruction, the CPU triggers a clean hardware trap called a **VM-Exit**, suspending the guest and passing control to the hypervisor.
  * The hypervisor handles the event and returns control to guest via **VM-Entry**.
* **Pros/Cons:** Completely unmodified guest operating systems run with direct hardware enforcement, zero binary translation overhead.

#### 3. Comparative Summary

| Metric | Full Virtualization | Paravirtualization | Hardware-Assisted Virtualization |
| :--- | :--- | :--- | :--- |
| **Guest OS Modification** | None (Unmodified) | Extensive (Modified kernel) | None (Unmodified) |
| **Privilege Interception** | Binary Translation | Hypercalls | Hardware VM-Exit / VM-Entry |
| **Performance Efficiency** | Moderate | High | Near-Native |
| **Silicon Requirement** | Standard x86 CPU | Standard x86 CPU | Intel VT-x or AMD-V required |
| **Examples** | Early VMware ESXi | Xen PV | KVM, Hyper-V, modern ESXi |

---

### Q2. Explain Containerization and Operating System-Level Virtualization: Architecture, Docker Engine Workflow, and Comparison with Traditional Hypervisor VMs.

#### 1. Concept of Operating System-Level Virtualization
Unlike hypervisor-based virtualization (which virtualizes raw hardware and boots a complete guest operating system kernel for each VM), **Containerization** virtualizes the operating system itself. Multiple isolated user-space instances (containers) share a single, common underlying host OS kernel.

#### 2. Linux Kernel Isolation Primitives
Containerization is made possible by two fundamental Linux kernel subsystems:
1. **Namespaces (Isolation):** Carves out private views of global operating system resources:
   * `pid`: Isolated process tree (container PID 1 is distinct from host PID 1).
   * `net`: Independent virtual network interfaces, routing tables, and IP addresses.
   * `mnt`: Private filesystem mount points (root filesystem isolation).
   * `ipc`: Isolated System V IPC and POSIX message queues.
   * `uts`: Independent hostname and domain name.
   * `user`: Independent user and group ID mappings.
2. **Control Groups (cgroups - Resource Governance):** Meters, allocates, and bounds hardware resources:
   * CPU quota and shares (`cpu.cfs_quota_us`).
   * Memory limits and Out-Of-Memory (OOM) triggers (`memory.limit_in_bytes`).
   * Block I/O read/write bandwidth throttles (`blkio.throttle.read_bps_device`).

#### 3. Architectural Comparison: Virtual Machines vs. Containers

```
    HYPERVISOR VIRTUAL MACHINE                      CONTAINER ARCHITECTURE
+---------------------------------+           +---------------------------------+
| App A     | App B     | App C   |           | App A     | App B     | App C   |
+-----------+-----------+---------+           +-----------+-----------+---------+
| Bins/Libs | Bins/Libs |Bins/Libs|           | Bins/Libs | Bins/Libs |Bins/Libs|
+-----------+-----------+---------+           +-----------+-----------+---------+
| Guest OS  | Guest OS  |Guest OS |           |       Docker / Container Engine |
+-----------+-----------+---------+           +---------------------------------+
|      Hypervisor (Type 1/2)      |           |        Host Operating System    |
+---------------------------------+           +---------------------------------+
|        Physical Hardware        |           |        Physical Hardware        |
+---------------------------------+           +---------------------------------+
```

#### 4. The Docker Engine Architecture
The Docker platform encapsulates containers using a client-server architecture:
* **Docker Client (`docker` CLI):** User-facing interface that transmits commands to the daemon.
* **Docker Daemon (`dockerd`):** Persistent background service managing images, containers, networks, and storage volumes.
* **Containerd & runc:** Low-level container runtime adhering to OCI (Open Container Initiative) specifications that interfaces directly with Linux cgroups and namespaces.
* **Union File System (OverlayFS):** Implements layered storage with Copy-on-Write (CoW), allowing multiple containers to share base image layers while maintaining a thin writable layer.

#### 5. Comparison Matrix

| Attribute | Hypervisor Virtual Machines | Operating System Containers |
| :--- | :--- | :--- |
| **Virtualization Level** | Hardware abstraction | OS Kernel abstraction |
| **Startup Time** | Minutes (boots full OS) | Milliseconds to seconds (starts process) |
| **Storage Footprint** | Gigabytes per VM (full OS image) | Megabytes (shared base layers) |
| **RAM Utilization** | Heavy (dedicated guest OS RAM) | Minimal (shared host kernel memory) |
| **Isolation Strength** | Strong (hardware hypervisor boundary) | Moderate (kernel namespace isolation) |

---

### Q3. Analyze Cloud Storage Models: Block Storage, File Storage, and Object Storage with Architectural Comparisons and Use-Case Analysis.

#### 1. Architectural Taxonomies
Cloud storage systems are classified into three primary paradigms based on data presentation, accessibility protocols, and internal indexing:

```
                                Cloud Storage Models
                                         |
         +-------------------------------+-------------------------------+
         |                               |                               |
   Block Storage                   File Storage                    Object Storage
   (Raw LBA volumes,               (Hierarchical tree,             (Flat namespace,
    iSCSI / NVMe-oF)                NFS / SMB protocols)            REST API / HTTP S3)
```

#### 2. Detailed Technical Characteristics

##### 1. Block Storage (e.g., AWS EBS, Azure Managed Disks)
* **Architecture:** Treats physical disks as arrays of fixed-size blocks (typically $512\text{ bytes}$ or $4\text{ KB}$), referenced solely by Logical Block Addressing (LBA). The storage system provides raw unformatted volumes.
* **Filesystem Responsibility:** The client operating system creates its own filesystem (ext4, XFS, NTFS) on top of the block device.
* **Access Protocols:** Low-latency storage protocols like iSCSI, Fibre Channel, or NVMe-oF.
* **Use Cases:** Relational databases (PostgreSQL, Oracle, MySQL), OS boot drives, enterprise transactional workloads requiring sub-millisecond I/O latency and high IOPS.

##### 2. File Storage (e.g., AWS EFS, Azure Files, NFS)
* **Architecture:** Exposes a shared hierarchical filesystem organized into directories, subdirectories, and files.
* **Metadata Support:** Manages native file attributes (permissions, ownership, modification timestamps, POSIX access locks).
* **Access Protocols:** Network-attached storage (NAS) protocols over TCP/IP: NFS (Network File System for Linux) and SMB/CIFS (Server Message Block for Windows).
* **Multi-Instance Access:** Supports concurrent Read-Write mounts from hundreds of virtual compute instances simultaneously.
* **Use Cases:** Shared application directories, CMS content repositories, development home directories, legacy application migration.

##### 3. Object Storage (e.g., AWS S3, Google Cloud Storage, Ceph)
* **Architecture:** Discards hierarchical folders entirely in favor of a massive, flat namespace. Data is stored as self-contained discrete units called **Objects**.
* **Object Structure:**
  * **Payload:** The raw binary data (from 0 bytes to 5 Terabytes).
  * **Globally Unique Identifier (Key):** String key locating the object inside a bucket (e.g., `s3://bucket-name/folder/image.png`).
  * **Extensible Metadata:** Custom key-value pairs associated with the data (e.g., author, content-type, retention policies).
* **Access Protocols:** Standard Web RESTful APIs over HTTP/HTTPS (`GET`, `PUT`, `POST`, `DELETE`).
* **Durability:** Typically provides $99.999999999\%$ (11 nines) durability via automatic multi-region erasure coding.
* **Use Cases:** Unstructured data lakes, backup and archival, static website hosting, big data analytical pipelines.

#### 3. Architectural Comparison Matrix

| Dimension | Block Storage | File Storage | Object Storage |
| :--- | :--- | :--- | :--- |
| **Namespace Structure** | Raw flat sector blocks | Hierarchical directory tree | Completely flat bucket namespace |
| **Access Method** | Block-level SCSI/NVMe commands | File-level POSIX filesystem APIs | RESTful HTTP/HTTPS Web APIs |
| **Data Mutability** | In-place block modification | In-place file modification | Immutable (atomic overwrite of whole object) |
| **Scalability** | Limited to volume sizes (TBs) | Scalable to Petabytes | Virtually infinite (Exabytes) |
| **Latency** | Sub-millisecond (ultra-fast) | Few milliseconds | Tens to hundreds of milliseconds |
| **Metadata Flexibility** | None (managed by host OS) | Fixed POSIX metadata | Rich, user-defined custom metadata |

---

### Q4. Explain Cloud Load Balancing Algorithms: Layer 4 vs Layer 7, Round Robin, Weighted Round Robin, Least Connections, and Consistent Hashing.

#### 1. Role of Load Balancers in Cloud Systems
A Cloud Load Balancer (LB) acts as a high-availability reverse proxy that distributes incoming user traffic across a pool of backend compute instances, preventing resource saturation, maximizing throughput, and masking individual server crashes.

#### 2. Architectural Comparison: Layer 4 vs. Layer 7

```
                     LAYER 4 (NETWORK LOAD BALANCER)
  Clients ===[TCP/UDP: IP + Port]======> [ L4 LB ] ======> [ Backend VM Pool ]
  (No packet payload inspection; raw packet routing; microsecond latency)

                     LAYER 7 (APPLICATION LOAD BALANCER)
  Clients ===[HTTP/HTTPS Request]======> [ L7 LB ] ======> [ /api -> Service A ]
                                                   ======> [ /img -> Service B ]
  (Full TLS termination; inspects URL paths, headers, cookies; rich routing)
```

* **Layer 4 LB:** Operates at Transport Layer. Routes based on IP 5-tuple: $\langle \text{Source IP}, \text{Source Port}, \text{Dest IP}, \text{Dest Port}, \text{Protocol} \rangle$. Extremely high throughput (millions of requests/sec).
* **Layer 7 LB:** Operates at Application Layer. Terminates TCP connection, inspects HTTP URI, headers, cookies, and routes to specialized microservices.

#### 3. Core Load Balancing Algorithms

##### 1. Round Robin & Weighted Round Robin
* **Standard Round Robin:** Distributes requests sequentially across backend instances in circular order:
  $$\text{Target} = i \pmod N$$
  Assumes all backend instances possess identical hardware capacities.
* **Weighted Round Robin:** Assigns a capacity weight $w_i$ to each server $S_i$. Servers with higher weights receive proportionally more requests:
  $$\text{Fraction of Traffic}(S_i) = \frac{w_i}{\sum_{j=1}^N w_j}$$

##### 2. Least Connections & Weighted Least Connections
* **Least Connections:** Directs incoming requests to the server currently maintaining the smallest number of active concurrent TCP connections.
* **Advantage:** Ideal for long-lived transactions or requests with unpredictable processing times (e.g., file downloads, database queries).
* **Weighted Variant:** Computes ratio $\frac{\text{Active Connections}}{w_i}$ and chooses server with minimum ratio.

##### 3. Consistent Hashing
* **Problem with Modulo Hashing:** Mapping a request to server using $\text{hash}(\text{client\_ip}) \pmod N$ causes massive cache invalidation when $N$ changes during autoscaling.
* **Consistent Hashing Mechanics:**
  1. Both backend servers and client keys are mapped to a continuous circular hash ring of size $2^{32}-1$.
  2. A request travels clockwise along the ring until it encounters the first server.
  3. **Virtual Nodes (Vnodes):** Each physical server is assigned multiple positions on the ring (e.g., 100-200 vnodes) to prevent hotspots and ensure uniform load balance.
  4. When a server is added or removed, only $1/N$ fraction of traffic is relocated!

---

### Q5. Describe the MapReduce Programming Model: Architecture, Map Phase, Shuffle & Sort Phase, Reducer, Combiner, and Partitioner Mechanisms.

#### 1. Foundations of MapReduce
Pioneered by Jeffrey Dean and Sanjay Ghemawat at Google, MapReduce is a distributed computational framework designed for parallel processing of massive datasets across clusters of commodity hardware.

#### 2. The Data Flow Pipeline

```
Input Data (HDFS)
      |
      v
+------------------+     Intermediate Pairs     +--------------------+
|   SPLIT 1        | ------------------------>  |    MAPPER 1        |
|   SPLIT 2        | ------------------------>  |    MAPPER 2        |
+------------------+                            +--------------------+
                                                          |
                                                          v [Optional: COMBINER]
                                                          |
                                                +--------------------+
                                                |   PARTITIONER      |
                                                +--------------------+
                                                          |
                                                          v
                                              ======================
                                              SHUFFLE & SORT PHASE
                                              (Network all-to-all)
                                              ======================
                                                          |
                                                          v
                                                +--------------------+
                                                |    REDUCER 1       |
                                                |    REDUCER 2       |
                                                +--------------------+
                                                          |
                                                          v
                                                Final Output to HDFS
```

#### 3. Detailed Phase Breakdown

##### 1. Map Phase
* Input data in HDFS is split into fixed-size logical chunks (**InputSplits**, typically $128\text{ MB}$).
* Each Mapper processes records in parallel:
  $$\text{map}: (k_1, v_1) \to \text{list}(k_2, v_2)$$
* The mapper applies application-specific logic and emits intermediate key-value pairs.

##### 2. Combiner Phase (Local Aggregation)
* An optional optimization executed locally on the mapper node.
* Aggregates intermediate keys before they are written to local disk or transmitted over the network:
  $$\text{combiner}: (k_2, \text{list}(v_2)) \to \text{list}(k_2, v_2)$$
* Example: In WordCount, transforms thousand pairs of $\langle \text{"cloud"}, 1 \rangle$ into a single pair $\langle \text{"cloud"}, 1000 \rangle$, reducing network traffic by orders of magnitude.

##### 3. Partitioner Mechanism
* Determines which Reducer instance receives which intermediate key $k_2$.
* Default implementation uses hash partitioning:
  $$\text{Reducer\_ID} = \text{hash}(k_2) \pmod{\text{numReducers}}$$
* Guarantees that all values associated with an identical key arrive at the **exact same Reducer**.

##### 4. Shuffle and Sort Phase
* **Shuffle:** The system transfers intermediate data partitions from all mapper nodes to the corresponding reducer nodes across the network.
* **Sort:** The framework automatically merges and sorts all incoming intermediate key-value pairs by key $k_2$.
* Groups values into an iterable collection: $(k_2, [v_{2,1}, v_{2,2}, \dots])$.

##### 5. Reduce Phase
* The Reducer processes each unique key and its grouped collection of values:
  $$\text{reduce}: (k_2, \text{list}(v_2)) \to \text{list}(k_3, v_3)$$
* Computes final aggregated summaries and writes outputs directly to persistent HDFS blocks.

---

### Q6. Analyze Cloud Security Threats in Multi-Tenant Environments: Hypervisor Vulnerabilities, VM Escape, Side-Channel Attacks, and Data Protection Mechanisms.

#### 1. Multi-Tenancy Security Paradigm
In public clouds (AWS, GCP, Azure), physical infrastructure is co-located and shared among competing or mutually hostile tenants. Security relies entirely on the mathematical and hardware integrity of the isolation boundary.

```
+-------------------------------------------------------------------------+
|                      PHYSICAL MULTI-TENANT SERVER                       |
+-------------------------------------------------------------------------+
|   [Tenant A: Innocent Corp]             [Tenant B: Malicious Attacker]  |
|   - Private database                    - Port scanning, cache timing   |
|   - Proprietary keys                    - Hypervisor escape exploits    |
+-------------------------------------------------------------------------+
|                  HYPERVISOR ISOLATION BOUNDARY                          |
+-------------------------------------------------------------------------+
|        SHARED HARDWARE (L3 CPU Cache, Memory Bus, Shared NIC)           |
+-------------------------------------------------------------------------+
```

#### 2. Critical Cloud Vulnerabilities

##### 1. Virtual Machine Escape (VM Escape)
* **Threat Mechanism:** An attacker with root privileges inside a guest VM exploits an unpatched vulnerability in the hypervisor (e.g., flaws in virtualized device drivers such as VENOM vulnerability in virtual floppy controllers).
* **Impact:** The malicious code breaks out of guest Ring 0/Ring 3, executing arbitrary code directly inside the hypervisor (Ring -1 / Root mode). The attacker achieves unrestricted administrative control over all co-located tenant VMs on that physical server.

##### 2. Microarchitectural Side-Channel Attacks
* **Threat Mechanism:** Even when hypervisor software isolation is flawless, modern CPUs share physical microarchitectural hardware structures across cores (e.g., L1/L2/L3 caches, Branch Target Buffers).
* **Spectre & Meltdown:** Exploit speculative execution and out-of-order execution in CPUs.
* **Cache Timing Attacks (FLUSH+RELOAD, PRIME+PROBE):** A malicious VM continuously primes shared CPU cache lines and observes access latencies caused by a neighboring tenant VM. By analyzing cache timing patterns, attackers reconstruct cryptographic private keys (RSA/AES) across VM boundaries.

##### 3. Resource Starvation (Noisy Neighbor Effect)
* A rogue tenant exhausts shared physical resources (disk I/O bus, PCIe bandwidth, network interfaces) through high-throughput burst activity, degrading the QoS of co-located tenants.

#### 3. Defensive Countermeasures and Data Protection

##### 1. Hypervisor Hardening & MicroVMs
* Adoption of ultra-minimalist hypervisors written in memory-safe languages with near-zero legacy device emulation (e.g., AWS Firecracker microVMs using Rust and Linux KVM).

##### 2. Core Pinning & Cache Allocation Technology (CAT)
* Dedicating physical CPU cores exclusively to specific tenants and utilizing Intel CAT to partition hardware L3 cache lines, preventing cross-tenant cache sharing.

##### 3. Cryptographic Enclaves (Confidential Computing)
* Hardware technologies like AMD SEV (Secure Encrypted Virtualization) and Intel SGX (Software Guard Extensions) encrypt guest VM memory in hardware using CPU-managed keys, ensuring even a compromised hypervisor cannot inspect tenant RAM.

##### 4. Key Management Services (KMS) & Envelope Encryption
* Data at rest is encrypted using unique Data Encryption Keys (DEKs), which are in turn encrypted under a customer-managed Master Key (KEK) stored inside hardware security modules (HSMs).
