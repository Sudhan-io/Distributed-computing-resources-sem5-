# CS3551 / 23CST504 - DISTRIBUTED COMPUTING
## UNIT V: CLOUD COMPUTING
### PART A: 2-MARK QUESTIONS AND ANSWERS (EXAM-READY REPOSITORY)

---

### Q1. Name the different cloud deployment models.
**Answer:**
According to the National Institute of Standards and Technology (NIST), the four primary cloud deployment models are:
1. **Public Cloud:** Infrastructure is owned by a third-party cloud provider and made available to the general public over the Internet (e.g., AWS, Microsoft Azure, GCP).
2. **Private Cloud:** Infrastructure is provisioned for exclusive use by a single organization, either managed internally or by a third party (e.g., OpenStack, VMware vSphere).
3. **Hybrid Cloud:** Composition of two or more distinct clouds (private and public) bound together by standardized technology enabling data and workload portability.
4. **Community Cloud:** Infrastructure shared by several organizations with shared concerns, compliance mandates, or missions (e.g., government agency clouds, healthcare research clouds).

---

### Q2. Recall the driving factors behind the adoption of cloud computing.
**Answer:**
1. **CapEx to OpEx Shift:** Eliminates upfront capital expenditure on hardware, servers, and data centers in favor of pay-as-you-go operational expenditure.
2. **Rapid Elasticity & Scalability:** Instant provisioning and de-provisioning of resources to match fluctuating application workloads.
3. **High Availability & Fault Tolerance:** Multi-region redundancy and automated failover guarantee 99.99%+ service level agreements (SLAs).
4. **Faster Time to Market:** Pre-configured platforms (PaaS) and serverless environments accelerate application deployment.

---

### Q3. Define virtualization and its role in cloud computing.
**Answer:**
* **Definition:** Virtualization is the foundational technology that creates a software-based (virtual) abstraction of physical computing resources, including hardware platforms, processors, storage, and networks, using a Virtual Machine Monitor (VMM) or Hypervisor.
* **Role in Cloud:**
  * Enables **Multi-Tenancy:** Multiple isolated Virtual Machines (VMs) share the same physical server securely.
  * Facilitates **Dynamic Resource Allocation:** Rapid creation, live migration, pausing, and scaling of virtual instances.

---

### Q4. What are the key attributes of scalability and elasticity in the context of cloud computing?
**Answer:**
* **Scalability:** The capability of an infrastructure to handle growing workloads by provisioning additional hardware or software resources:
  * *Vertical Scalability (Scaling Up):* Increasing the CPU, RAM, or storage of an existing instance.
  * *Horizontal Scalability (Scaling Out):* Adding more server instances to a cluster.
* **Elasticity:** The dynamic, automated ability to scale resources **both up and down** in real time to match actual demand, minimizing idle resource waste.

---

### Q5. How do different cloud deployment models differ in terms of resource ownership and management?
**Answer:**

| Deployment Model | Resource Ownership | Infrastructure Location | Management & Operation |
| :--- | :--- | :--- | :--- |
| **Public Cloud** | Third-party Cloud Service Provider (CSP) | Off-premises CSP data centers | Managed entirely by the CSP |
| **Private Cloud** | Single enterprise organization | On-premises or hosted data center | Managed by internal IT or third-party MSP |
| **Hybrid Cloud** | Shared: Enterprise + CSP | Mixed: On-premises + Off-premises | Jointly orchestrated via hybrid tools |
| **Community Cloud**| Shared consortium of organizations | On-premises or third-party facility| Managed by community members or provider |

---

### Q6. Can you explain why security is a significant challenge in cloud computing?
**Answer:**
1. **Multi-Tenancy Vulnerabilities:** Co-location of multiple tenant VMs on shared physical hardware creates side-channel attack hazards and hypervisor escape risks.
2. **Loss of Physical Data Governance:** Storing sensitive enterprise data on external, third-party infrastructure increases concerns over compliance (GDPR, HIPAA) and data breaches.
3. **Shared Responsibility Ambiguity:** Misconfigurations of cloud access policies (e.g., publicly exposed Amazon S3 buckets or IAM role leaks).
4. **Vendor Lock-In & Portability:** Incompatible proprietary security and authentication standards across providers.

---

### Q7. How does load balancing contribute to optimizing resource utilization in cloud computing?
**Answer:**
* **Uniform Traffic Distribution:** Distributes incoming network requests across multiple healthy backend server instances, preventing any single instance from becoming a bottleneck.
* **Elastic Scaling Integration:** Coordinates with auto-scaling engines to register newly spun-up instances and remove drained nodes.
* **Fault Masking:** Continuously performs health checks; routes traffic away from degraded or failed instances to maintain continuous service availability.

---

### Q8. Describe the role of replication in ensuring data availability and reliability in the cloud.
**Answer:**
* **Fault Tolerance:** Storing multiple identical replicas of data across independent failure domains (availability zones, regions) prevents data loss upon hardware crashes.
* **Read Throughput Acceleration:** Allows client requests to be serviced concurrently by multiple distributed replicas.
* **Low Latency Access:** Geographically distributes data closer to end-users (e.g., CDNs, geo-replicated databases like DynamoDB Global Tables).

---

### Q9. Design a load balancing strategy for distributing incoming requests across multiple cloud servers.
**Answer:**
A production-grade dual-tier strategy:
1. **Tier 1 (Global Traffic Management):** Use DNS-based latency routing (e.g., AWS Route 53) to direct users to the nearest cloud region.
2. **Tier 2 (Regional Layer 7 Application Load Balancer):**
   * Employs the **Least Outstanding Requests** algorithm for computationally intensive workloads (or **Weighted Round-Robin** for homogeneous microservices).
   * Incorporates **Active Health Checks** to automatically quarantine failing containers.
   * Leverages **SSL Termination** at the balancer to offload cryptographic CPU overhead from backend application servers.

---

### Q10. Develop a scalability plan for a cloud-based application to accommodate fluctuating user demands.
**Answer:**
1. **Stateless Application Architecture:** Decouple application state into external distributed caches (Redis) and managed databases (Aurora/RDS).
2. **Target Tracking Auto-Scaling Policies:**
   * Set a dynamic threshold (e.g., maintain average cluster CPU utilization at 65%).
3. **Cooldown Intervals & Buffer Capacity:** Configure a 300-second cooldown period to prevent thrashing, with a minimum capacity baseline to absorb sudden bursts.
4. **Scheduled Scaling:** Pre-scale cluster capacity ahead of predictable seasonal traffic spikes (e.g., flash sales).

---

### Q11. Configure monitoring tools to track resource usage and performance metrics in a cloud infrastructure.
**Answer:**
1. **Agent Installation:** Deploy lightweight monitoring agents (e.g., AWS CloudWatch Agent, Prometheus node-exporter) on all virtual instances.
2. **Core Metrics Collection:**
   * Compute: CPU Utilization, Memory Usage, Disk I/O ops.
   * Network: Inbound/Outbound throughput, packet drop rate.
   * Application: Request response latency, HTTP 5xx error rates.
3. **Alarm & Action Triggers:** Configure SNS alerts to notify engineers via email/Slack when error rates exceed 1%, and link threshold breaches directly to Auto-Scaling Groups.

---

### Q12. Analyze the impact of scalability on the cost-effectiveness of cloud solutions.
**Answer:**
* **Elimination of Over-Provisioning:** Traditional data centers require purchasing peak-capacity hardware that sits idle 80% of the time. Auto-scaling aligns costs directly with actual consumption.
* **Elimination of Under-Provisioning Losses:** Automatically scales out during traffic spikes, preventing downtime that leads to lost revenue.
* **Spot Instance Utilization:** Stateless scalable workloads can leverage discounted Spot/Preemptible instances, reducing compute expenses by up to 70–90%.

---
