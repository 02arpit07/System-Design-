# Horizontal and Vertical Scaling

## Q1. Can you explain the difference between horizontal and vertical scaling?

**Best Answer:**

- **Vertical Scaling (Scaling Up):** Increasing the capacity of a single machine (e.g., upgrading CPU, RAM, SSD).  
  - Pros: Simple, no code changes, works well for small to medium workloads.  
  - Cons: Hardware limits, expensive, single point of failure.  

- **Horizontal Scaling (Scaling Out):** Adding more machines (servers/instances) to handle load.  
  - Pros: Practically unlimited scalability, better fault tolerance, cost-effective with commodity hardware.  
  - Cons: Complex — requires load balancing, distributed caching, consistent data management.  

---

## Q2. When would you prefer vertical scaling over horizontal scaling in a Java-based system?

**Best Answer:**

Prefer vertical scaling when:  
- Application is monolithic or not yet designed for distributed systems.  
- Scaling needs are small or temporary.  
- Latency-sensitive operations (e.g., in-memory processing like JVM heap tuning).  
- Quick fix in production without redesigning the architecture.  

**Example:** Increasing JVM heap size and CPU on a single machine running a Spring Boot monolith before re-architecting into microservices.  

---

## Q3. What challenges do we face with horizontal scaling in Java applications?

**Best Answer:**  
- Session Management – if sessions are stored in-memory, scaling across nodes becomes tricky. Solution: use sticky sessions or external session stores (Redis).  
- Distributed Caching – maintaining consistency across multiple nodes.  
- Database Bottlenecks – a single DB may still be a choke point; need replication, sharding, or CQRS.  
- Concurrency Issues – race conditions when multiple nodes write to the same resource.  
- Operational Overhead – load balancing, service discovery, orchestration (K8s).  

---

## Q4. How does scaling affect the JVM specifically in Java applications?

**Best Answer:**  
- **Vertical Scaling:** Increasing heap size gives more room for objects but leads to longer GC pauses. JVM tuning (G1GC, ZGC) is critical.  
- **Horizontal Scaling:** Each JVM runs on separate nodes, so GC pauses are isolated. But distributed coordination (e.g., across Kafka consumers, distributed locks) is required.  

**Key Point:** Java devs must balance JVM tuning with architectural scaling. Sometimes running multiple smaller JVM instances horizontally works better than one huge vertically scaled JVM.  

---

## Q5. Suppose your system is read-heavy (like Instagram feed). How would you scale it?

**Best Answer:**  
- For read-heavy workloads, horizontal scaling is preferred.  
  - Use CDNs for static content.  
  - Use replicated read-only databases for handling read queries.  
  - Introduce caching layers (Redis/Memcached) for hot data.  
  - Use load balancers to distribute traffic across app instances.  

Vertical scaling could help temporarily but won’t solve bottlenecks at scale.  

---

## Q6. Suppose your system is write-heavy (like financial transactions). How would you approach scaling?

**Best Answer:**  
- Writes are trickier — vertical scaling might help temporarily with stronger DB hardware, but long-term:  
  - Use partitioning/sharding strategies (e.g., based on userId).  
  - Apply write-optimized databases (e.g., Cassandra, DynamoDB).  
  - Use asynchronous writes + queues (Kafka, SQS) to buffer load.  
  - Ensure idempotency to handle retries across horizontally scaled services.  

**Example:** In a Java/Kafka system, producers can write asynchronously while consumers batch process into sharded databases.  

---

## Q7. If you had to explain trade-offs to a non-technical stakeholder, how would you summarize horizontal vs vertical scaling?

**Best Answer:**  
- Vertical Scaling = "Make one server stronger" → simpler, faster fix but has limits and can fail entirely.  
- Horizontal Scaling = "Add more servers to share the work" → more complex, but highly scalable and reliable.  

**Analogy:**  
- Vertical = hiring one superhuman employee.  
- Horizontal = hiring a team of average people working together.  

---

## What happens when there is a GC pause?

During a GC pause, the JVM executes a **Stop-The-World (STW)** event:  
- All application threads are frozen so the Garbage Collector can safely analyze memory.  
- GC identifies live vs dead objects, reclaims memory, and possibly compacts the heap.  
- No requests or background tasks are processed during this time.  

**Effects of a GC pause:**  
- Latency spikes (requests may stall for hundreds of ms or seconds).  
- Throughput drops (no new work is done during the pause).  
- Cascading failures if other services timeout and retry.  

**Mitigations:**  
- Use modern collectors like G1GC, ZGC, Shenandoah.  
- Prefer multiple smaller JVMs (horizontal scaling) over one huge heap.  
- Use off-heap caching (Redis/Memcached).  
- Monitor GC logs to track pause frequency and duration.  
