# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT V: CLOUD COMPUTING
### PART B: 8-MARK QUESTIONS AND ANSWERS (IN-DEPTH EXAM REPOSITORY)

---

## QUESTION 1: Demonstrate the Cloud Computing Architecture.

### 1. Architectural Overview
Cloud computing architecture refers to the components and sub-components required for cloud computing. These components typically consist of a **front-end platform** (client devices, web browsers, mobile apps), **back-end platforms** (servers, storage, virtualization layers, operating systems), a **cloud-based delivery network**, and **management/security subsystems**.

```
+-----------------------------------------------------------------------------------------+
|                         CLOUD COMPUTING ARCHITECTURE TOPOLOGY                           |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       +-------------------------------------------------------------------------+       |
|       |                     FRONT-END LAYER (Client Environment)                |       |
|       |          Web Browsers, Mobile Apps, Thin Clients, Enterprise Portals    |       |
|       +------------------------------------+------------------------------------+       |
|                                            |                                            |
|                                            | Internet / Direct Connect / VPN            |
|                                            v                                            |
|       +-------------------------------------------------------------------------+       |
|       |                     BACK-END LAYER (Cloud Provider Infrastructure)      |       |
|       |                                                                         |       |
|       |   [ APPLICATION SERVICE LAYER ]    SaaS (Salesforce, Office 365, etc.)  |       |
|       |   [ PLATFORM SERVICE LAYER ]       PaaS (Elastic Beanstalk, App Engine) |       |
|       |   [ INFRASTRUCTURE SERVICE LAYER ] IaaS (EC2, Virtual Machines, S3)     |       |
|       |   -------------------------------------------------------------------   |       |
|       |   [ VIRTUALIZATION LAYER ]         Hypervisors (KVM, Xen, Hyper-V, ESXi)|       |
|       |   [ PHYSICAL INFRASTRUCTURE ]      Servers, Storage SAN, Fast Switches  |       |
|       |   -------------------------------------------------------------------   |       |
|       |   [ CLOUD MANAGEMENT & SECURITY ]  Identity & Access, Monitoring, Billing|      |
|       +-------------------------------------------------------------------------+       |
+-----------------------------------------------------------------------------------------+
```

---

### 2. Core Components of Cloud Architecture

1. **Front-End Platform:**
   * Interfaces used by end-users to interact with cloud services (web user interfaces, CLI tools, mobile SDKs).
2. **Back-End Platform:**
   * The powerhouse containing all computing engines, storage subsystems, virtual machines, and management mechanisms.
3. **Cloud Service Delivery Layers:**
   * **SaaS (Software-as-a-Service):** Finished end-user software delivered over the web.
   * **PaaS (Platform-as-a-Service):** Execution environments, runtime engines, and database backends for developers.
   * **IaaS (Infrastructure-as-a-Service):** Virtualized raw hardware (compute instances, block storage, virtual networks).
4. **Cloud Management and Governance Subsystem:**
   * **Resource Provisioning Engine:** Automates container/VM deployment.
   * **Monitoring & Billing:** Tracks resource consumption metrics for pay-as-you-go charging.
   * **Security & Compliance:** Identity & Access Management (IAM), data encryption at rest and in transit.

---

## QUESTION 2: Analyze the Cloud Computing Technologies.

### 1. Foundational Technologies Enabling the Cloud
Cloud computing is an amalgamation of multiple mature computing technologies.

```
+-----------------------------------------------------------------------------------------+
|                         CORE ENABLING TECHNOLOGIES OF CLOUD                             |
+--------------------------+--------------------------------------------------------------+
| TECHNOLOGY               | ROLE AND IMPACT ON CLOUD COMPUTING                           |
+--------------------------+--------------------------------------------------------------+
| 1. Virtualization        | Decouples operating systems from underlying physical         |
|                          | hardware, enabling multi-tenancy and dynamic resource slicing|
|                          |                                                              |
| 2. Service-Oriented      | Structures applications as collections of loosely coupled,   |
|    Architecture (SOA)    | discoverable services communicating via standard APIs.       |
|                          |                                                              |
| 3. Web Services & RESTful| Standardized communication protocols (HTTP, JSON, XML)      |
|    APIs                  | enabling programmatic interaction across heterogeneous tools.|
|                          |                                                              |
| 4. Multi-Tenancy         | Architectural approach allowing a single software instance   |
|                          | or hardware cluster to serve multiple tenants securely.      |
|                          |                                                              |
| 5. Utility Computing     | Business model packaging compute/storage resources as a      |
|                          | metered utility service (analogous to electricity or water). |
+--------------------------+--------------------------------------------------------------+
```

---

### 2. Deep Dive: Virtualization and Hypervisors
* **Type-1 (Bare-Metal) Hypervisors:** Run directly on physical server hardware without a host OS (e.g., VMware ESXi, KVM, Xen, Microsoft Hyper-V). Delivers maximum performance and low latency; standard in enterprise cloud data centers.
* **Type-2 (Hosted) Hypervisors:** Run on top of an existing host operating system (e.g., VirtualBox, VMware Workstation). Suited for development and client workstations.

```
Type-1 (Bare-Metal) Hypervisor             Type-2 (Hosted) Hypervisor
+-------------------------------+          +-------------------------------+
|   VM 1   |   VM 2   |   VM 3  |          |   VM 1   |   VM 2   |   VM 3  |
+-------------------------------+          +-------------------------------+
|     Type-1 Hypervisor (KVM)   |          |     Type-2 Hypervisor         |
+-------------------------------+          +-------------------------------+
|     Physical Hardware         |          |     Host Operating System     |
+-------------------------------+          +-------------------------------+
                                           |     Physical Hardware         |
                                           +-------------------------------+
```

---

## QUESTION 3: Difference between Public Cloud, Private Cloud, Hybrid Cloud, and Community Cloud.

### 1. Comparative Analysis Matrix

| Feature | Public Cloud | Private Cloud | Hybrid Cloud | Community Cloud |
| :--- | :--- | :--- | :--- | :--- |
| **Ownership** | Third-party CSP (AWS, Google, Microsoft) | Single enterprise organization | Shared (Enterprise + CSP) | Shared by community consortium |
| **Infrastructure Location** | Off-premises CSP global data centers | On-premises data center or dedicated hosting | Mixed (On-premises + Public Cloud) | On-premises or third-party facility |
| **Security & Privacy**| Multi-tenant; isolation managed by CSP software | Highest; dedicated single-tenant hardware | High; sensitive data kept in private cloud | High; tailored to community compliance |
| **Cost Model** | Pay-as-you-go; Zero capital expenditure (OpEx) | High upfront CapEx + recurring maintenance | Balanced; baseline on-prem, burst to public | Costs distributed across participants |
| **Scalability** | Near-infinite, immediate elasticity | Limited by physical hardware budget | High; elastic cloud bursting available | Moderate; bounded by consortium capacity |
| **Control & Customization** | Standardized; limited hardware control | Full administrative and architectural control | High flexibility across environments | Shared governance policies |
| **Typical Use Cases**| Web apps, dev/test, startups, SaaS apps | Banking, defense, healthcare records | Cloud bursting, disaster recovery, analytics | Government joint agencies, universities |

---

## QUESTION 4: Construct the Cloud Deployment Model.

### 1. Conceptual Framework
A **Cloud Deployment Model** defines the specific environment architecture, access boundaries, resource ownership, and operational governance of cloud infrastructure.

```
+-----------------------------------------------------------------------------------------+
|                             CLOUD DEPLOYMENT ARCHITECTURES                              |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       +------------------------------------+   +------------------------------------+   |
|       |            PUBLIC CLOUD            |   |           PRIVATE CLOUD            |   |
|       | - Multi-tenant shared hardware     |   | - Single-tenant dedicated hardware |   |
|       | - Accessible to public via Internet|   | - Enterprise intranet access       |   |
|       | - CSP manages all infrastructure   |   | - High compliance and control      |   |
|       +-----------------+------------------+   +-----------------+------------------+   |
|                         \                                       /                       |
|                          \                                     /                        |
|                           \                                   /                         |
|                            v                                 v                          |
|       +-----------------------------------------------------------------------------+   |
|       |                                 HYBRID CLOUD                                |   |
|       | - Interconnects Private Cloud and Public Cloud via secure VPN / Direct Link |   |
|       | - Enables "Cloud Bursting" during peak workload spikes                      |   |
|       | - Balances security for core data with elasticity for public-facing apps    |   |
|       +-----------------------------------------------------------------------------+   |
|                                                                                         |
|       +-----------------------------------------------------------------------------+   |
|       |                               COMMUNITY CLOUD                               |   |
|       | - Shared by multiple organizations with common missions / compliance rules  |   |
|       | - Example: Multi-hospital medical research cloud, joint government cloud    |   |
|       +-----------------------------------------------------------------------------+   |
+-----------------------------------------------------------------------------------------+
```

---

### 2. Operational Mechanics of Cloud Bursting in Hybrid Cloud
1. Baseline, mission-critical operations execute on the enterprise's internal private cloud infrastructure under steady-state conditions.
2. When demand exceeds private capacity (e.g., Black Friday traffic surge), the orchestration layer triggers **Cloud Bursting**:
   * Temporarily spawns additional stateless application instances in the public cloud.
   * Balances load between internal and external instances transparently.
   * De-provisions public cloud instances as traffic returns to baseline, minimizing costs.

---

## QUESTION 5: Contrast between AWS, Azure, and Google Cloud Platform (GCP).

### 1. Architectural Overview of Top Cloud Providers
Amazon Web Services (AWS), Microsoft Azure, and Google Cloud Platform (GCP) represent the dominant global public cloud hyperscalers.

---

### 2. Comparative Technology Matrix across Key Service Domains

| Service Category | Amazon Web Services (AWS) | Microsoft Azure | Google Cloud Platform (GCP) |
| :--- | :--- | :--- | :--- |
| **Primary Strength** | Pioneer, broadest service catalog, vast ecosystem | Enterprise integration (Windows, Active Directory) | Big Data, Machine Learning, container leadership (K8s) |
| **Virtual Compute (IaaS)** | **Amazon EC2** (Elastic Compute Cloud) | **Azure Virtual Machines** | **Google Compute Engine (GCE)** |
| **Platform / Serverless** | AWS Lambda, Elastic Beanstalk | Azure Functions, App Service | Cloud Functions, Cloud Run |
| **Object Storage** | **Amazon S3** (Simple Storage Service) | **Azure Blob Storage** | **Google Cloud Storage (GCS)** |
| **Block Storage** | Amazon EBS (Elastic Block Store) | Azure Managed Disks | Google Persistent Disks |
| **Relational Database** | Amazon RDS, Amazon Aurora | Azure SQL Database | Cloud SQL, Cloud Spanner |
| **NoSQL Database** | Amazon DynamoDB | Azure Cosmos DB | Cloud Firestore, Cloud Bigtable |
| **Networking & CDN** | Amazon VPC, CloudFront, Route 53 | Virtual Network (VNet), Azure CDN | Virtual Private Cloud, Cloud CDN |
| **Identity & Access** | AWS IAM | Azure Active Directory (Entra ID)| Cloud IAM |
| **Container Engine** | Amazon ECS, Amazon EKS | Azure Kubernetes Service (AKS) | Google Kubernetes Engine (GKE) |

---

## QUESTION 6: Elaborate the Scaling in Cloud Computing.

### 1. Scaling Paradigms: Vertical vs. Horizontal Scaling
Scaling is the ability of an application or infrastructure to adapt its resource capacity in response to changing workload demands.

```
+-----------------------------------------------------------------------------------------+
|                         VERTICAL SCALING VS. HORIZONTAL SCALING                         |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       VERTICAL SCALING (Scale Up / Scale Down)                                          |
|       +----------+          +------------------+                                        |
|       | Instance |  ===>    | Bigger Instance  |  (Increase CPU / RAM / Disk)           |
|       | 2 vCPU   |          | 8 vCPU, 32GB RAM |  - Hardware ceilings exist             |
|       | 4GB RAM  |          +------------------+  - Requires instance restart (downtime)|
|       +----------+                                                                      |
|                                                                                         |
|       HORIZONTAL SCALING (Scale Out / Scale In)                                         |
|       +----------+          +----------+  +----------+  +----------+                    |
|       | Instance |  ===>    | Instance |  | Instance |  | Instance | (Add more nodes)   |
|       +----------+          +----------+  +----------+  +----------+ - Infinite ceiling |
|                                                                      - Zero downtime    |
+-----------------------------------------------------------------------------------------+
```

---

### 2. Auto-Scaling Lifecycle and Strategies

```
+-----------------------------------------------------------------------------------------+
|                               AUTO-SCALING LIFECYCLE                                    |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|       [ MONITORING ]  Collects real-time metrics (CPU utilization, Queue depth)         |
|             |                                                                           |
|             v                                                                           |
|       [ EVALUATION ]  Alarm triggers if Metric > Threshold for N evaluation periods    |
|             |                                                                           |
|             v                                                                           |
|       [ SCALING ACTION ] Auto-Scaling Group launches new instances from template       |
|             |                                                                           |
|             v                                                                           |
|       [ REGISTRATION ] Load Balancer performs health checks; routes traffic to new node|
|             |                                                                           |
|             v                                                                           |
|       [ COOLDOWN ]    Suspends further scaling for T seconds to prevent thrashing       |
+-----------------------------------------------------------------------------------------+
```

* **Scaling Strategies:**
  1. **Reactive (Dynamic) Scaling:** Scales based on real-time metric thresholds (e.g., launch 2 instances when CPU > 75% for 5 minutes).
  2. **Proactive (Scheduled) Scaling:** Pre-scales ahead of known recurring spikes (e.g., scale up at 9:00 AM on weekdays).
  3. **Predictive Scaling:** Uses machine learning models trained on historical traffic patterns to anticipate spikes before they occur.

---
