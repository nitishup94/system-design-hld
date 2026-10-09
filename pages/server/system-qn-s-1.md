# System Design Interview – Questions &amp; Answers

**Golden rule for almost every question:** *Find the bottleneck → measure → fix the root cause → protect dependencies → monitor.*

## Table of Contents

1. [Scalability &amp; High Availability](#1-scalability--high-availability)
2. [Databases](#2-databases)
3. [Caching (Redis)](#3-caching-redis)
4. [Kafka, Queues &amp; Async Processing](#4-kafka-queues--async-processing)
5. [Payments, Consistency &amp; Distributed Transactions](#5-payments-consistency--distributed-transactions)
6. [Node.js Internals](#6-nodejs-internals)
7. [File Uploads](#7-file-uploads)
8. [Production Debugging &amp; Observability](#8-production-debugging--observability)
9. [Rate Limiting, Auth &amp; Networking](#9-rate-limiting-auth--networking)
10. [Architecture Components](#10-architecture-components)

---

# 1. Scalability &amp; High Availability

## Q1. Traffic grew from 1K to 10M requests. What would you change in the architecture?

**Mistake:** "Just add more EC2 servers."

**Answer (step by step):**

1. **Scale the API** – Load Balancer + multiple stateless API servers + Auto Scaling.
2. **Add caching** – Redis in front of the DB to cut read traffic.
3. **Scale the database** – Query optimization → indexing → read replicas → partitioning → sharding. Separate reads and writes.
4. **Async processing** – Queue/Kafka + workers for emails, notifications, reports, heavy jobs.
5. **Distribute traffic** – CDN for static/cacheable content.
6. **Protect the system** – Rate limiting, circuit breaker, timeouts, retries with backoff, monitoring/alerting.

**Flow:** `CDN → Load Balancer → Auto-scaled APIs → Redis → DB/Read Replicas → Kafka/Workers`

**Interview line:** "I identify where the bottleneck moves as traffic grows, then scale each layer independently."

**Remember:** Scale → Cache → Async → Database → Protect → Observe

---

## Q2. Normal traffic is 100K requests/min, but during a flash sale/IPL match it jumps to 10M requests/min. What changes?

**Answer:**

1. **Find the bottleneck first** – API → Cache → DB → Queue → External services. Which fails first?
2. **CDN + caching** – serve static/hot data from the edge.
3. **Load balancer + auto scaling** – scale horizontally.
4. **Protect the DB** – Redis, read replicas, connection pooling, query optimization.
5. **Control traffic** – rate limiting, queue/buffering, backpressure, request prioritization.
6. **Graceful degradation** – critical APIs stay available; non-critical features are limited or deferred.

**Interview line:** "I don't just scale the app tier. I identify the bottleneck, protect downstream dependencies, absorb spikes with caching and queues, and prioritize critical traffic."

**Remember:** Measure → Cache → Scale → Protect → Queue → Degrade Gracefully

---

## Q3. You built a Netflix-like app for 100K users. Tomorrow you expect 5M. How do you scale?

**Answer – layer by layer:**


| Layer           | Bottleneck                                       | Solution                                                                                              |
| --------------- | ------------------------------------------------ | ----------------------------------------------------------------------------------------------------- |
| API             | CPU, memory, connections                         | Load Balancer + multiple stateless Node.js APIs + Auto Scaling                                        |
| Database        | More API servers → more queries                  | Query optimization → indexes → Redis → read replicas → partitioning/sharding (don't jump to sharding) |
| Video           | Huge bandwidth                                   | CDN + object storage; never stream video through Node.js                                              |
| Background work | Transcoding, thumbnails, analytics overload APIs | API → Kafka/Queue → Workers                                                                           |
| Traffic spikes  | Blockbuster launch                               | Rate limiting, bounded concurrency, circuit breaker, autoscaling                                      |


**Interview line:** "Identify the bottleneck, measure it, and scale each layer independently instead of scaling blindly."

---

## Q4. How do you scale an application from 0 to 10 million users?

**Answer (in order):**

1. Vertical scaling (quick win)
2. Horizontal scaling
3. Load balancer
4. DB indexing
5. Redis cache
6. Read replicas
7. CDN
8. Microservices
9. Kafka/RabbitMQ
10. Auto scaling
11. Monitoring

**Takeaway:** Scaling is a journey. Remove one bottleneck at a time.

---

## Q5. Your app serves 10M users at \~100K requests/sec peak. How do you make it highly available?

**Answer:**

1. **Remove single points of failure** – no single EC2, DB or service instance.
2. **Application** – multiple instances, multi-AZ, auto scaling, health checks.
3. **Database** – primary + read replicas, automated failover, backups.
4. **Redis** – replication/managed HA, failover; never make the cache the only source of truth.
5. **Async** – Kafka/queue, multiple consumers, retry + DLQ.

**Follow-ups:**

- *Entire AZ goes down?* → Traffic automatically moves to healthy instances in another AZ.
- *DB primary goes down?* → Failover to a healthy replica with minimal interruption.
- *Traffic becomes 10×?* → Auto-scale + caching + rate limiting + queues to protect downstream.

**Key point:** HA ≠ just adding more servers. It needs **Redundancy + Failover + Health Checks + Monitoring + Recovery**, and you must *test failure scenarios*, not assume HA from a diagram.

**Remember:** Design → Fail → Recover → Continue Serving

---

## Q6. Node.js app on a single EC2 hits 5,000 RPS. You need HA + zero-downtime deployment. How do you redesign it?

**Problems with single EC2:** single point of failure, limited capacity, deploy causes downtime, spike can crash it.

**Answer:**

1. **VPC** – public/private subnets across multiple AZs.
2. **ALB** – distributes traffic to healthy EC2 instances.
3. **Auto Scaling** – add/remove instances based on traffic/CPU.
4. **IAM Role** for EC2 – don't store AWS credentials in the app.
5. **Zero-downtime deploy** – New EC2 instances → health check → ALB → shift traffic → remove old instances.

**Architecture:** `Users → ALB → EC2-A / EC2-B (Auto Scaling) → Multi-AZ VPC`

---

## Q7. You have 5 EC2 instances; one is dedicated to Payment Service and it crashes. Users are still paying. What happens?

**Answer:**

- Clients must never choose an EC2 directly: `User → Load Balancer → Healthy Payment Instance`.
- LB health checks remove the unhealthy instance and route to healthy ones.
- **Key catch:** if there is no second Payment instance, the service is still down. Run multiple stateless instances, ideally across AZs.
- **Auto Scaling** launches a replacement EC2.
- **Idempotency key** on the Payment API → safe retries, no duplicate payments.
- **DB** – transactions + unique constraints.
- **Monitoring** – EC2 health, 5xx, latency, payment failures, alerts.

**Interview line:** "I'd never make Payment Service depend on a single instance."

**Remember:** Detect → Remove → Reroute → Replace → Retry Safely → Recover

---

## Q8. How does Hotstar handle 10M+ users during the IPL final?

**Answer:**

1. CDN for video delivery
2. Load balancer
3. Auto scaling
4. API gateway
5. Microservices (independent scaling)
6. Redis caching
7. Kafka for async processing
8. DB replicas for read scalability
9. Monitoring

**Takeaway:** Don't scale one server. Scale every layer of the architecture.

---

## Q9. How do companies handle millions of users without crashing a single server?

**Answer (Load Balancer):**

1. Deploy multiple servers
2. Put a Load Balancer in front
3. Distribute requests evenly
4. Health checks
5. Redirect traffic if a server fails
6. Auto-scale at peak
7. Improves response time and availability

**Tech:** AWS ALB, Nginx, HAProxy, Kubernetes, EC2

**Follow-up:** *Round Robin vs Least Connections?* – Round Robin for similar, short requests; Least Connections when request durations vary a lot.

---

## Q10. How do you deploy a new API version with ZERO downtime?

**Problem:** Replacing servers directly (Old → DOWN → New) causes 5xx errors.

**Answer – Blue/Green (or Canary):**

1. Deploy v2 (Green) separately from v1 (Blue).
2. Run health checks + smoke tests.
3. Shift traffic gradually: 100/0 → 90/10 → 50/50 → 0/100.
4. **Rollback:** route traffic back to Blue instantly – no redeploy.

**Check before switching:** API health, error rate, p95/p99 latency, DB compatibility, logs &amp; metrics.

**DB changes:** use backward-compatible migrations – **Expand → Deploy → Migrate → Contract**.

**Interview line:** "Blue-Green or Canary behind a load balancer, validate with health checks and metrics, shift traffic gradually, keep the old version for instant rollback, keep DB changes backward compatible."

**Takeaway:** Zero downtime ≠ just multiple servers. It needs traffic shifting + health checks + observability + rollback + DB compatibility.

---

# 2. Databases

## Q11. How would you scale a database from 1M to 10M+ users?

**Mistake:** Jumping straight to sharding.

**Answer:**

1. **Find the bottleneck** – CPU, I/O, storage, connections, reads or writes?
2. **Optimize first** – indexes, query optimization, connection pooling, pagination.
3. **Read bottleneck** – Read replicas (primary = writes, replicas = reads).
4. **Cache** hot data in Redis.
5. **Write/storage bottleneck** – vertical scaling first, then sharding.
6. **Sharding** – stable shard key (e.g. `userId`) across DB nodes.
7. **Huge tables** – partitioning (e.g. by date).
8. **High availability** – replication within shards + failover.

**Senior mindset:** Don't ask "Which technology?" Ask "What is the bottleneck, and what is the least complex solution?"

**Follow-up:** What if one shard becomes a hot shard? → see Q12.

---

## Q12. One database shard is overloaded (hot shard). What do you do?

**Bottleneck:** One shard has high CPU/I/O/connections/queries while others are idle.

**Don't:** Immediately add more shards – a bad shard key recreates the hotspot.

**Approach:** First find *why* it's hot – query patterns, shard-key distribution, read/write ratio, CPU, I/O, connections.

**Solution:**

1. Redis – cache hot/read-heavy data
2. Read replicas – spread reads
3. Rate limiting – protect the shard
4. Rebalancing – move data/traffic away
5. Resharding – change the shard key if distribution is fundamentally bad
6. Consistent hashing – avoid massive redistribution when nodes change

**Key insight:** *Equal data distribution does NOT mean equal traffic distribution.*

**Mindset:** Detect → Protect → Distribute → Rebalance → Reshard

**Follow-up:** What if ONE user generates millions of requests to the same shard? → use `userId + bucket` (Q13), caching, replicas.

---

## Q13. Users grow from 1K to 10K to 10M. How do you choose the partition key?

**Don't just pick `user_id`.** First ask: *How is the data accessed and how will traffic be distributed?*

**Answer:**

- Mostly "get orders by userId" → `partitionKey = userId`.
- `partitionKey = country` → hot partition if one country dominates.
- Even `userId` can be hot (one heavy user) → use **`userId + bucket`** to spread across partitions.

**Interview line:** "I choose the partition key based on access patterns, cardinality and traffic distribution – even data and traffic, no hot partitions."

**Takeaway:** A good partition key is not just unique: Access Pattern + Distribution + Hotspots + Future Scale.
1K → simple, 10K → validate, 10M → design for hotspots.

---

## Q14. The table grew from 10M to 500M rows and the indexed query now takes 5 seconds. What do you investigate?

**Don't:** Just add another index.

**Answer:**

1. **Query plan** – `EXPLAIN ANALYZE`: is the index actually used?
2. **Selectivity** – if the query reads most of 500M rows, an index won't help.
3. **Composite index** – must match WHERE + JOIN + ORDER BY.
4. **Table growth** – partitioning, archiving old data, better pagination.
5. **Statistics** – outdated stats make the optimizer pick bad plans.

**Fixes:** optimize query, correct composite index, update statistics, partition, archive cold data, keyset/cursor pagination, cache.

**Remember:** Slow query → EXPLAIN → Find bottleneck → Optimize → Measure again.

---

## Q15. DB CPU is at 95% with 20 EC2 instances in front. What do you do?

**Don't:** Add more EC2 – it sends even more traffic to the DB.

**Answer:**

1. **Investigate** – slow queries, missing indexes, too many connections, read-heavy load, expensive JOINs. Use `EXPLAIN ANALYZE`; monitor CPU, latency, connections, slow-query log.
2. **Optimize** – fix expensive queries, add the *right* indexes (not everywhere).
3. **Separate reads/writes** – reads → replicas, writes → primary.
4. **Cache** with Redis; reduce unnecessary DB calls; use connection pooling.
5. **Scale DB** – vertical → replicas → partitioning/sharding, based on workload.

**Remember:** Measure → Optimize → Cache → Read Replicas → Scale

---

## Q16. NoSQL is faster, so why do banks still use SQL?

**Answer:** It's not about speed – it's about **correctness**.

- Transfer ₹1,00,000 from A → B needs *Debit A + Credit B* to both succeed or both fail → **ACID transactions**.
- SQL gives: strong consistency, transactions, constraints &amp; relationships, complex queries, referential integrity.
- **Use NoSQL** for massive horizontal scale, flexible schema, high-throughput access patterns: Redis (cache), MongoDB (documents), Cassandra/DynamoDB (large distributed workloads).

**Takeaway:** Database choice = workload + guarantees + scale, not hype.

---

## Q17. You have MySQL, NoSQL and Object Storage. How do you decide where data goes?

**Answer:** Understand the data and access pattern first.


| Need                                                                                                        | Choice                                    |
| ----------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| Structured data, relationships, transactions, strong consistency, complex JOINs (Users → Orders → Payments) | **MySQL**                                 |
| Flexible schema, very high scale, key-based lookups (sessions, activity, events)                            | **NoSQL**                                 |
| Large unstructured files (images, videos, PDFs, backups, 50 GB exports)                                     | **Object Storage** (never store in MySQL) |


**Remember:** Data → Access Pattern → Consistency → Scale → Database.

---

## Q18. WhatsApp Status expires in 24h; 3 billion statuses/day. A single cron DELETE would overwhelm the DB. How do you design cleanup?

**Don't:** One cron running `DELETE` on billions of rows.

**Answer:**

1. **Store expiry** – columns `status_id, user_id, created_at, expires_at`; index `expires_at`.
2. **Delete in small batches** – fetch 1K–10K expired rows, delete, repeat.
3. **Distribute cleanup** – Scheduler → Queue → multiple workers with *bounded concurrency*.
4. **Time-based partitioning** – drop/truncate old partitions instead of deleting row by row.
5. **Logical expiry** – app treats `expires_at < now` as expired; physical delete can be async.

**Remember:** Index → Batch → Distribute → Control Load → Partition

---

## Q19. How do you manage PostgreSQL, MongoDB and Redis in one Node.js app?

**Don't:** Create connections randomly inside each API.

**Answer:**

- Define responsibilities: PostgreSQL = transactions/relational, MongoDB = flexible documents, Redis = cache.
- Structure: `API → Service → Repository → DB`.
- Each DB gets its own connection pool, config, retry/reconnect strategy, health check.
- Never create a new connection per request.
- **Failure isolation:** if PostgreSQL is down, don't fail Redis/Mongo calls unnecessarily; if Redis is down, fall back to DB where acceptable.

**Takeaway:** Design for isolation, not just connectivity.

---

# 3. Caching (Redis)

## Q20. 10K RPS and cache misses hit the primary DB. How do you protect it?

**Flow:** Cache miss → DB spike → CPU/connections ↑ → latency ↑ → DB can crash.

**Answer:**

1. **Protect the primary** – rate limiting, connection pooling, bounded concurrency.
2. **Separate reads/writes** – Redis → read replica for reads; primary for writes.
3. **Prevent cache stampede** – TTL + jitter + locking/single-flight.
4. **Handle overload** – circuit breaker + load shedding (fail fast for non-critical).

**Remember:** Cache Miss → Protect DB → Read Replica → Prevent Stampede → Failure Handling

**Follow-up:** What if both Redis and the primary DB are unavailable? (Serve stale/degraded data, queue writes, fail fast with clear errors.)

---

## Q21. Redis goes down. What happens, and how do you handle it?

**Problem:** Every request falls back to DB → CPU ↑, latency ↑, connections exhausted, DB failure.

**Answer:**

1. **Controlled fallback** – fall back to DB, but never unlimited.
2. **Protect DB** – rate limiting, connection pool limits, circuit breaker, request timeouts.
3. **Redis HA** – replication + automatic failover.
4. **Gradual cache rebuild** after recovery.

**Limitation:** If Redis holds critical state (distributed locks, sessions, rate-limit counters), fallback depends on the use case.

**Remember:** Redis Failure → Controlled Fallback → Protect DB → Failover → Recovery

---

## Q22. 10M requests/day… Redis dies. Will all traffic hit the primary DB? How do you prevent a cache stampede?

**Answer:**

1. **Rate limiting** – control traffic reaching the DB.
2. **Circuit breaker** – stop excessive calls to an unhealthy DB.
3. **Stampede protection** – lock / single-flight: 1000 misses → one request hits DB → updates cache → others read from cache.
4. **TTL + jitter** – keys must not expire at the same instant.
5. **DB protection** – connection limits, bounded concurrency, load shedding.

**Interview line:** "I wouldn't allow all misses to hit the primary. I'd control traffic, protect the DB, and use single-flight with TTL jitter."

**Remember:** Redis Down → Control Traffic → Protect DB → Prevent Stampede → Recover

---

## Q23. Redis is back, but 1 million keys must be rebuilt. How do you recover without another DB outage?

**Answer:**

1. **Gradual warming** – batches (e.g. 10K at a time), paced by DB capacity.
2. **Prioritize hot keys** – frequently accessed → important business data → rest.
3. **Rate-limit rebuild** – recovery workers → rate limiter → DB; bounded concurrency.
4. **TTL jitter** – e.g. 55–65 min instead of exactly 60.
5. **Monitor** – DB CPU, connections, query latency, Redis memory, hit ratio, rebuild rate. If DB struggles, pause/slow warming (adaptive throttling).

**Interview line:** "Cache recovery is also a traffic-management problem."

---

## Q24. DB handles 500K req/s; traffic jumps to 10M req/s and Redis crashes. What do you do?

**Bottleneck chain:** Redis failure → cache misses → 10M requests → DB overload → pool exhausted → cascading failure.

**Approach:** Redis HA → circuit breaker → rate limiting → request coalescing → cache fallback → load shedding.

**Solution:** Protect the DB first – never let every request reach it. Order of defense: rate limiting / load shedding at the edge, request coalescing, circuit breaker on the DB, serve stale cache where possible.

---

# 4. Kafka, Queues &amp; Async Processing

## Q25. How does Kafka work? (Explain the flow, don't define it)

1. **Producer** – e.g. Order Service publishes `{"event":"order.created","orderId":"123"}`.
2. **Topic &amp; Partitions** – `order-events` split into P0…P3; partitions give parallelism and ordering *within a partition*.
3. **Consumer Group** – consumers in a group share partitions (P0→C1, P1→C2…). Another group can consume the same events independently.
4. **Offset** – the consumer's position in a partition.

**Interview trap:** 3 partitions + 10 consumers = max **3 active** consumers in the group.

**Remember:** Producer → Topic → Partition → Consumer Group → Offset

---

## Q26. Producer sends 100K events/sec, consumers process 50K/sec. What happens?

**Result:** Consumer lag keeps increasing.

**Don't:** Immediately add consumers.

**Check:** lag, processing time, CPU/memory, DB latency, network, partition count.

**Solutions:**

- Add consumers (up to partition count)
- Increase partitions
- Optimize consumer (batching, faster queries, async)
- Fix the real bottleneck – if DB is slow, more consumers can make it worse

**Takeaway:** Lag is a symptom, not always the root cause.

**Follow-up:** Consumers increased but lag still grows? → partition count caps parallelism (or a hot partition – Q28).

---

## Q27. Producer = 5,000 msg/s, consumers = 500 msg/s. Lag keeps growing. What do you do?

**Answer:**

1. **Identify bottleneck** – lag &amp; processing latency, partition count, consumer count, consumer CPU/memory, DB/API latency, DB CPU &amp; connection pool.
2. **Scale accordingly**
   - Partitions limit parallelism → increase partitions + consumers.
   - DB/API bottleneck → optimize queries, batch processing, bounded concurrency, protect downstream.
3. **Prevent** – alerts on consumer lag, processing rate, error rate, DB/API latency.

**Remember:** Detect → Measure → Bottleneck → Scale → Protect → Monitor

---

## Q28. 100 partitions; consumers healthy but lag rises; one partition gets 40% of traffic. How do you handle it?

**Diagnosis:** Hot partition. Adding consumers won't help – **one partition → one active consumer in a group**.

**Why it happens:** Partition key like `customerId`/`merchantId` – one heavy key → same partition.

**Solutions:**

1. Redesign the partition key.
2. Controlled key sharding – `merchant_123_1`, `merchant_123_2`, `merchant_123_3`.
3. Increase partition count if more parallelism is needed.

**Trade-off:** Changing the key affects ordering → ask "Do we really need strict ordering for this key?"

**Takeaway:** Consumer lag ≠ always a consumer problem. Check partition-level traffic and lag first.

---

## Q29. Kafka or RabbitMQ?

**Scenario:** `POST /exports` – large report; don't hold the HTTP request for 5 minutes.


| Use **RabbitMQ** when                           | Use **Kafka** when                                                           |
| ----------------------------------------------- | ---------------------------------------------------------------------------- |
| Task/job processing: 1 job → 1 worker           | Event-driven: multiple systems need the same event                           |
| Needs ack, retry, DLQ, prefetch                 | Needs partitions, consumer groups, retention, replay, ordering per partition |
| Email, PDF, image processing, report generation | Order/payment events, user activity, audit pipelines, analytics              |


**Example:** `export.created` → Exporter, Audit and Notification consumer groups all react independently → Kafka.

**Don't say:** "Kafka is faster." **Say:** "I choose based on the communication pattern."

---

## Q30. A worker processes a video but crashes before ACK. What happens?

**Result:** Message is redelivered → duplicate work, wasted resources, duplicate side effects.

**Solution:**

1. **ACK only after successful processing** – Receive → Process → Verify → ACK.
2. **Idempotency** – unique `job_id`; if already processed, skip.
3. **Retry + backoff + DLQ.**

**Limitation:** Don't assume exactly-once; design idempotent operations.

**Ask always:** "What happens if the worker crashes at this exact moment?"

---

## Q31. Worker Threads handle CPU-heavy jobs, but Node.js crashes (deploy/OOM) mid-job. What happens to an in-memory job?

**Answer:** The job is **lost**. Worker Threads solve CPU-bound work, not **job durability**.

**Fix:** `API → Durable Queue (Kafka/RabbitMQ/SQS) → Worker`. If a worker dies, the job stays in the queue and a new worker picks it up.

**Checklist:** durable queue, retries, ack after success, idempotency, DLQ, monitoring.

---

## Q32. Notification system for 10M users: Kafka down, consumers crash, SMS provider times out. How do you avoid lost/duplicate notifications?

**Flow:** `API → Outbox → Kafka → Worker → Provider`


| Failure                | Handling                            |
| ---------------------- | ----------------------------------- |
| Kafka down             | Outbox safely stores the event      |
| Consumer crashes       | Kafka redelivers                    |
| Provider timeout       | Retry with exponential backoff      |
| Processed twice        | Idempotency key prevents duplicates |
| Provider keeps failing | DLQ + monitoring                    |


**Think:** What can be lost? duplicated? delayed? What if a dependency fails?

---

## Q33. A user cancels a long-running request but the background job keeps running. How do you handle it?

**Answer:**

1. Don't keep the HTTP request open – return `{ "jobId": "123", "status": "processing" }`.
2. Process asynchronously (Queue → Worker).
3. Cancel via `POST /jobs/123/cancel` → status `CANCELLED`.
4. Worker **checks cancellation status** during execution and stops.
5. For already-processed data use idempotency + transactions + checkpoints.

**Limitation:** Cancellation isn't always immediate (non-interruptible step may finish first).

**Remember:** Cancel Request ≠ Cancel Job.

---

## Q34. A payment succeeds, but SMS/email/analytics are slow. How do you keep the payment API fast?

**Mistake:** Doing everything synchronously.

**Answer:** Separate the **critical path** from async side effects.
`Payment Service → transaction → return success → publish PaymentSuccessful → Kafka → Notification → SMS provider`

**Add:** retry + backoff, idempotency, DLQ, Outbox pattern, monitoring.

**Takeaway:** Payment success should depend on the transaction, not on whether SMS responds in 2 seconds or 2 minutes.

---

# 5. Payments, Consistency &amp; Distributed Transactions

## Q35. Payment succeeded at the gateway but the connection dropped; user sees "Failed" and retries. How do you prevent duplicate charges?

**Problem:** Ambiguous outcome – timeout ≠ failure.

**Answer:**

1. **Idempotency key** – unique ID per payment attempt, reused on retry; already processed → return existing result.
2. **Status reconciliation** – UNKNOWN → check gateway → SUCCESS/FAILED. Never blindly create a new payment.
3. **Async events** – Gateway → Payment Service → Kafka → Order/Notification/Invoice.
4. **Outbox pattern** to publish events reliably.

**Design for:** idempotency, retry, reconciliation, timeouts, duplicate prevention, event-driven processing.

---

## Q36. Customer pays ₹10,000, network failure makes it look failed, they retry. How do you prevent double charge with Kafka?

**Answer:**

1. Idempotency key per request.
2. Store transaction status against that key.
3. Retry returns the existing result.
4. Process payment events asynchronously via Kafka when needed.
5. **Commit Kafka offset only after successful processing.**
6. Idempotent consumer – ignore already-processed transactions.
7. Retry + DLQ for failing events.

**Takeaway:** Kafka gives reliable event processing; idempotency prevents double processing.

**Follow-up:** Consumer crashes before committing offset? → message is redelivered; idempotent consumer makes it safe.

---

## Q37. Payment system at 50K TPS across two regions: DB timeouts, Kafka lag, retries, one region loses provider connectivity, users see both "Failed" and "Successful". How do you recover without duplicate charges?

**Don't:** Just restart services. Multiple failures are interacting.

**Chain:** DB timeout → Kafka lag → retry storm → provider failure → duplicate payment risk.

**Answer:**

1. **Is payment actually failing?** API failure ≠ payment failure; check provider status.
2. **Stop retry storm** – exponential backoff, retry limits, circuit breaker, rate limiting.
3. **Protect DB** – reduce unnecessary writes, queue non-critical work, check pool/locks/slow queries.
4. **Payment correctness** – unique `idempotency_key`; already processed → return existing result.
5. **Recovery** – Kafka + durable payment state + reconciliation. If state is unknown, query the provider; don't retry blindly.

**Staff-level point:** You can't promise "exactly once everywhere." Aim for **exactly-once business effect** via idempotency + durable state + reconciliation.

**Remember:** Detect → Contain → Preserve correctness → Recover → Reconcile

---

## Q38. Flash sale: 100K users buy the last 1,000 items; payments succeed but inventory goes negative. How do you design and recover?

**Root cause:** Concurrency + consistency – many requests read the same stock.

**Answer:**

1. **Reserve before payment** – Request → Reserve Inventory → Payment → Confirm Order.
2. **Atomic reservation:**
   ```sql
    UPDATE inventory SET stock = stock - 1
    WHERE product_id = ? AND stock > 0
   ```

    No row updated → SOLD OUT.
3. **Payment fails** → release reservation.
4. **At scale** – order state + queue/saga + reservation timeout.
5. **Payment OK but order service crashes** – idempotency key + durable order state + reconciliation (Payment Provider ↔ Order DB ↔ Inventory).

**Interview line:** "Treat inventory as a strongly consistent resource; reserve atomically before payment; use idempotency; Saga/reconciliation for compensation."

**Ask:** "What happens if the system crashes between these two steps?"

---

## Q39. BookMyShow: 2 seats left, 1,000 users book at once, Payment Kafka topic is flooded, and Broker 2 crashes. How do you handle it?

**Bottlenecks:**

1. **Seat booking** – race condition, double booking.
2. **Payment** – partition load, consumer lag.
3. **Broker failure** – partitions on Broker 2 temporarily unavailable.

**Solutions:**

- **Seat locking** – atomic DB operation or distributed lock, short TTL reservation; only one user gets each seat.
- **Payment** – partitioned events, consumer groups, idempotent processing.
- **Broker failure** – Kafka replication → leader election → another replica becomes leader → consumers resume.

**Rule:** Never depend on a single broker.

**Remember:** Concurrency → Consistency → Failure → Recovery → Idempotency

---

## Q40. Each microservice has its own DB. What if one DB write succeeds and another fails?

**Problem:** No single SQL transaction, no global rollback, risk of inconsistency.

**Solution: Saga Pattern**

- Local transactions per service
- Compensating transactions on failure
- Eventual consistency
- Better scalability

**Takeaway:** Saga exists because distributed transactions don't work well across microservices.

---

## Q41. Order Service (50K req/s) calls Payment, Inventory, Notification. Payment slows from 200ms to 10s. How do you stop a cascading failure?

**Cascade:** Payment slow → orders wait → connections pile up → timeouts → clients retry → more load → Payment more overloaded.

**Where to use each tool:**


| Tool                         | Purpose / Where                                                              |
| ---------------------------- | ---------------------------------------------------------------------------- |
| **Timeout**                  | First line – on every outbound call to Payment so threads aren't held        |
| **Retry + Backoff + Jitter** | Prevents retry storms (limited retries only)                                 |
| **Circuit Breaker**          | Around Payment calls – stop calling when it's failing; fail fast             |
| **Rate Limiting**            | At API edge – protects your API / caps client retries                        |
| **Bounded Concurrency**      | Limits concurrent in-flight calls to Payment – protects Payment              |
| **Bulkhead**                 | Separate pools per dependency – isolates Payment from Inventory/Notification |


*(Reference answer – the original post only asked the question.)*

---

# 6. Node.js Internals

## Q42. EC2 is healthy (CPU 100%) but the Node.js API is stuck. Why?

**Cause:** A CPU-heavy **synchronous** operation blocked the **event loop** → other requests wait → latency ↑ → timeouts.

**Identify:** event-loop lag, CPU, request latency, profiling, heap/memory.
CPU high + event-loop lag rising → CPU-bound code.

**Fix (don't just restart EC2):**

- Optimize the expensive code
- Move CPU work to Worker Threads
- Use background workers/queues
- Scale horizontally
- Add monitoring + alerts

**Interview line:** "EC2 being healthy doesn't mean the Node.js app is healthy."

---

## Q43. 100,000 requests hit one Node.js server at once. Will it crash?

**Problem:** Single process → event loop bottleneck, CPU-heavy tasks, high latency, single point of failure.

**Solution:**

1. Cluster – multiple processes across CPU cores
2. Load balancer
3. Worker Threads for CPU-heavy tasks
4. Redis caching
5. DB optimization
6. Horizontal scaling

**Takeaway:** Node can handle high traffic, but you scale the *architecture*, not just the process.

---

## Q44. Node.js CPU hits 95% under 10,000 requests. What happened and what's the fix?

**Cause:** JS runs mainly on one main thread; a CPU-heavy task blocks the event loop → requests wait → p95 ↑.

**Solution – Cluster:** run multiple Node.js worker processes to use multiple CPU cores.

**Remember:** One process → mainly one core for JS. Cluster = multiple *processes*, it does not make the main thread multi-threaded.

---

## Q45. Cluster vs Worker Threads?

- **Cluster** – multiple Node.js processes across CPU cores → **scale request handling**.
- **Worker Threads** – separate threads for CPU-intensive JS → **don't block the event loop**.

**Takeaway:** Cluster = scale requests. Worker Threads = handle CPU-heavy work.

---

## Q46. Why use Streams instead of `readFile`?

**Problem:** `fs.readFileSync("5GB-file.zip")` loads everything in memory; 100 users × 5 GB = 500 GB.

**Solution:** `fs.createReadStream()` processes the file chunk by chunk.

**Interview answer:** "Streams process large data incrementally instead of loading the whole payload, reducing memory and enabling scalable pipelines."

**Takeaway:** Streams are memory-efficient pipelines: File → Transform → Compression → Network → S3.

---

## Q47. Server produces 100 MB/s but the client consumes 10 MB/s. What happens?

**Problem:** Buffer grows → memory ↑ → performance drops → OOM risk.

**Solution: Backpressure** – the consumer signals the producer to slow down. Node.js Streams handle flow control (`readable.pipe(writable)` / `pipeline()`).

**Remember:** Producer &gt; Consumer → Backpressure.

---

## Q48. Why use `Promise.all()` for 3 independent tasks?

- Without it: total time ≈ T1 + T2 + T3.
- With it: tasks run concurrently, total ≈ max(T1, T2, T3).

```js
const [users, orders, products] = await Promise.all([
  getUsers(), getOrders(), getProducts()
]);
```

**Notes:**

- It does **not** create extra threads – it starts async I/O without waiting for each other.
- If one promise rejects, `Promise.all()` rejects.

---

# 7. File Uploads

## Q49. How do you upload a 10/50/100 GB file without exhausting Node.js memory?

**Naive:** Client → Node.js → S3 (loading the file into memory → OOM).

**Answer:**

1. Never buffer the whole file in Node.js.
2. Use **streaming** with backpressure (`pipeline()`), and let the S3 SDK handle multipart uploads.
3. **Preferred for 100 GB:** S3 multipart upload with **pre-signed URLs** – the client uploads chunks directly to S3.
4. Node.js only handles **authorization + metadata**.

**Why direct upload:** avoids memory pressure, server bandwidth cost, slow uploads, crashes.

**Takeaway:** 10 GB file ≠ 10 GB RAM. Don't make Node.js the pipe.

---

## Q50. Users upload multi-GB videos; connectivity drops at 20%. How do you resume without restarting? (10M concurrent users)

**Answer:**

1. **Direct upload** – Upload API → pre-signed URL → object storage (not through Node.js).
2. **Chunked / multipart upload** – uploaded chunks stay safe.
3. **Upload session** – unique `upload_id`; metadata `upload_id + chunk_number + status`.
4. **Resume** – client/server determine missing chunks (e.g. 1–20% done, 21–100% missing) and upload only those.
5. **Durable state** – not in Node.js memory; survives API restarts.
6. **Extras** – parallel chunk uploads, checksum validation per chunk, merge chunks, background processing (thumbnails, transcoding via Kafka/FFmpeg), notify users.

**Remember:** Direct Upload → Chunk → Persist → Resume

---

# 8. Production Debugging &amp; Observability

## Q51. Users report "API is very slow" but you don't know why. How do you investigate?

**Steps:**

1. **Start with symptoms** – RPS, error rate, p95/p99, CPU, memory. What changed? When did it start?
2. **Localize** – Load Balancer → Node.js → Redis → DB → External APIs. Don't blame Node.js yet.
3. **Check Node.js** – CPU ↑ + event-loop lag ↑ → blocking/CPU-heavy code.
4. **Fix root cause** – profile, optimize blocking code, move CPU work to workers, scale only if required.

**Remember:** Observe → Compare → Localize → Confirm → Fix → Monitor

---

## Q52. After 6 years, p95/p99 latency suddenly increased with no deployment. How do you find the root cause?

**Don't start by changing code.** Ask: *where did latency increase?*

1. **Timeline** – compare before vs during: p50/p95/p99, request rate, error rate, CPU/memory, DB/Redis/external latency.
2. **Traffic** – did it spike? If normal, don't blame traffic.
3. **Node.js** – CPU, memory, event-loop lag, GC pauses, active connections.
4. **Database** – CPU, connection pool, slow queries, locks, execution plans, replication lag (a query fast at 10M rows may be slow at 500M).
5. **Redis** – hit ratio, latency, memory, evictions, connection errors. Hit ratio drop → more DB queries → higher latency.
6. **External dependencies** – latency, timeouts, error rate.

**Don't say:** "I'll optimize the query." **Say:** "I'll correlate the latency spike with traffic, app, cache, DB and dependency metrics, then identify which layer introduced it."

---

## Q53. How do you know the issue is really the database (not Node.js, Redis or network)?

**Steps:**

1. **Metrics** – API latency → Node CPU → Redis latency → DB CPU → query latency → network; find where latency starts rising.
2. **Node.js** – CPU, event loop, memory normal? Move deeper.
3. **Redis** – latency, errors, hit rate normal? Less likely the cause.
4. **Database** – CPU 95%, query latency ↑, connections ↑, slow queries ↑ → evidence.
5. **Confirm** – slow query logs, `EXPLAIN ANALYZE`, DB monitoring, compare with API latency.

**Remember:** Don't Guess → Measure → Correlate → Confirm → Fix

**Follow-up:** How do you *prove* your optimization improved the system? (Compare before/after p95/p99, DB CPU, query time and throughput under the same load test.)

---

# 9. Rate Limiting, Auth &amp; Networking

## Q54. Which rate-limiting algorithm would you choose?


| Algorithm          | Pros                                    | Cons                       |
| ------------------ | --------------------------------------- | -------------------------- |
| **Fixed Window**   | Simple, basic protection                | Boundary burst problem     |
| **Sliding Window** | More accurate, smoother                 | More memory/complexity     |
| **Token Bucket**   | Allows controlled bursts + average rate | –                          |
| **Leaky Bucket**   | Smooth, predictable output rate         | Requests may wait in queue |


**Choose by:** traffic pattern, whether bursts are allowed, how strict the limit is, need for smooth vs bursty traffic. Don't say "Token Bucket is best."

---

## Q55. You have 20+ Node.js instances (50K RPS). Where do you store the rate-limit counter?

**Problem:** Per-server counters make limits inconsistent.

**Answer:**

- Use a **shared/distributed counter – Redis** with atomic operations.
- Algorithm: **Token Bucket** if bursts are OK.
- **Redis down?** Fail-open (allow temporarily) or fail-closed (reject to protect system) – a business decision.
- **At 50K RPS** consider Redis Cluster, sharding, atomic ops/Lua, hot keys, network latency.

**Remember:** Multiple Servers → Shared State → Redis → Algorithm → Failure Handling → Scale

---

## Q56. Normal users get 5 attempts/day, VIP users 20. How do you enforce it?

1. Identify user via **JWT** → `userId`, `role`.
2. **Don't trust the client** to decide limits.
3. Backend verifies NORMAL/VIP.
4. Track usage in Redis: key `rate:userId:date`.
5. Apply limit: Normal → 5, VIP → 20.
6. Use **atomic** ops (`INCR`/Lua) so concurrent requests can't bypass it.
7. **TTL** to expire at end of daily window.
8. Exceeded → `429 Too Many Requests`.
9. **Load test** – no bypass, correct 429s, Redis latency, p95/p99, throughput, Redis-failure scenario.

**Senior question:** What if 1,000 requests from the same user arrive simultaneously? → atomic Redis operations.

---

## Q57. Session, JWT, OAuth, API Keys, mTLS – how do you choose authentication?

**Don't say** "JWT is always best." Start with the use case.


| Use case                         | Choice                     |
| -------------------------------- | -------------------------- |
| Web application                  | Session + Cookie           |
| Mobile / SPA                     | Token-based auth           |
| Third-party login                | OAuth 2.0 / OpenID Connect |
| API access                       | API Key / OAuth            |
| Service-to-service               | mTLS / OAuth               |
| Internal/controlled (with HTTPS) | Basic Auth                 |


**Ask:** Who is the client? User or service auth? What security level? How are credentials managed (expiry, rotation, revocation, storage)?

**Note:** OAuth 2.0 = delegated access; OIDC = authentication via identity providers.

**Remember:** Client → Use Case → Security → Credential Lifecycle → Authentication

---

## Q58. HTTP vs HTTPS?

- **HTTP** – no transport encryption; data can be intercepted.
- **HTTPS = HTTP + TLS**, which provides:
  1. **Encryption** – data can't be read in transit.
  2. **Integrity** – detects tampering.
  3. **Authentication** – server proves identity via TLS certificate.

**Flow:** Client → TLS handshake → certificate verification → secure session → encrypted HTTP data.

---

# 10. Architecture Components

## Q59. Load Balancer vs API Gateway – why both?


| Load Balancer                                                          | API Gateway                                                                             |
| ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Distributes traffic                                                    | Controls API traffic                                                                    |
| Health checks, removes unhealthy instances, enables horizontal scaling | Auth, rate limiting, routing, request validation, logging/observability, API versioning |


**Flow:** `Client → API Gateway → Load Balancer → Service Instances`

- Gateway decides: **WHERE** should this request go?
- Load Balancer decides: **WHICH** healthy instance handles it?

They overlap in some features; choose based on routing, security, scale and operational needs.

---

## Q60. If API Gateway routes requests, why do we need Service Discovery?

- **API Gateway** – external entry point: Client → Gateway → Order Service.
- **Service Discovery** – internal: Order Service asks "Where is a healthy Payment Service instance?" instead of hardcoding `payment-service:3001`.

**Why:** instances scale up/down, restart, change IPs, become unhealthy.

**Takeaway:** Gateway answers "Which service?"; Service Discovery answers "Which healthy instance?"

---

## Q61. How does Instagram show 100K story viewers instantly?

1. Receive story-view event
2. Publish event to Kafka
3. Store recent viewers in Redis
4. Persist permanently in PostgreSQL
5. Serve the viewer list from Redis for fast response

**Takeaway:** Event-driven architecture + caching = low-latency, scalable system.

---

## Cheat Sheet – One-Liners


| Topic         | Remember                                                  |
| ------------- | --------------------------------------------------------- |
| Scaling       | Measure → Cache → Scale → Protect → Queue → Degrade       |
| DB            | Measure → Optimize → Cache → Replicas → Shard             |
| Redis down    | Control Traffic → Protect DB → Prevent Stampede → Recover |
| Kafka lag     | Detect → Measure → Bottleneck → Scale → Protect → Monitor |
| Payments      | Idempotency + Durable State + Reconciliation              |
| Jobs          | Process → Verify → ACK                                    |
| Uploads       | Direct Upload → Chunk → Persist → Resume                  |
| Debugging     | Observe → Compare → Localize → Confirm → Fix → Monitor    |
| Staff mindset | Design for failure, not just the happy path               |


