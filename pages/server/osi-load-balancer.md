# OSI Model — 7 Layers

**OSI = Open Systems Interconnection**


| Layer | Name             | Short Explanation                                                          | Examples              |
| ----- | ---------------- | -------------------------------------------------------------------------- | --------------------- |
| **7** | **Application**  | Provides network services directly to applications/users.                  | HTTP, HTTPS, FTP, DNS |
| **6** | **Presentation** | Handles data formatting, encryption, and compression.                      | SSL/TLS, JSON, JPEG   |
| **5** | **Session**      | Establishes, manages, and terminates communication sessions.               | Session management    |
| **4** | **Transport**    | Ensures data delivery and uses port numbers to identify applications.      | TCP, UDP              |
| **3** | **Network**      | Routes data across different networks using IP addresses.                  | IP, Router            |
| **2** | **Data Link**    | Delivers data within the same/local network using MAC addresses.           | Ethernet, MAC, Switch |
| **1** | **Physical**     | Transmits actual bits (0s and 1s) through cables, fiber, or radio signals. | Cable, Fiber, Radio   |


---

## Easy Way to Remember

**7 → Application**  
**6 → Presentation**  
**5 → Session**  
**4 → Transport**  
**3 → Network**  
**2 → Data Link**  
**1 → Physical**

### Mnemonic

**Please Do Not Throw Sausage Pizza Away**

- **P** → Physical
- **D** → Data Link
- **N** → Network
- **T** → Transport
- **S** → Session
- **P** → Presentation
- **A** → Application

## Important Interview Points

- **Layer 7:** HTTP/HTTPS
- **Layer 4:** TCP/UDP, Port numbers
- **Layer 3:** IP address, Router
- **Layer 2:** MAC address, Switch
- **Layer 1:** Cables, Fiber, Radio signals
- **Load Balancer:** Can work at **Layer 4** or **Layer 7**
- 

---

# Load Balancer

A **Load Balancer** distributes incoming traffic across multiple servers so that one server does not become overloaded.

## Which OSI Layer Does a Load Balancer Use?

### Layer 4 — Transport Layer

Works with:

- **TCP**
- **UDP**
- **IP address**
- **Port number**

It does **not need to understand the HTTP request content**.

**Example:**

```text
Client
   |
   | TCP Request
   ↓
Load Balancer
   |
   |----→ Server 1
   |----→ Server 2
   |----→ Server 3
```

The load balancer can distribute connections using methods such as:

- **Round Robin** → Server 1 → Server 2 → Server 3 → Server 1
- **Least Connections** → Sends traffic to the server with the fewest active connections
- **IP Hash** → Uses the client's IP to consistently select a server

---

### Layer 7 — Application Layer

Works with application-level information such as:

- HTTP/HTTPS
- URL/path
- HTTP headers
- Cookies
- Hostname

**Example:**

```text
Client
   |
   ↓
Load Balancer
   |
   |-- /api/*     → API Server
   |
   |-- /images/*  → Image Server
   |
   |-- /admin/*   → Admin Server
```

A Layer 7 load balancer can make routing decisions based on the **actual HTTP request**.

## L4 vs L7 Load Balancer table:


| Feature                   | L4                    | L7                                |
| ------------------------- | --------------------- | --------------------------------- |
| OSI Layer                 | Transport             | Application                       |
| Works with                | TCP/UDP               | HTTP/HTTPS                        |
| Can inspect URL?          | ❌ No                  | ✅ Yes                             |
| Can inspect HTTP headers? | ❌ No                  | ✅ Yes                             |
| Routing decision          | IP + Port             | URL, Host, Headers, Cookies, etc. |
| Processing                | Faster/lower overhead | More processing                   |
| Example                   | TCP load balancing    | HTTP reverse proxy                |


## Simple Interview Answer

> **A load balancer can work at Layer 4 or Layer 7. L4 distributes connections based mainly on IP, port, and transport-level information, while L7 can inspect HTTP information such as URL, headers, and cookies to make routing decisions.**


---


# L4 vs L7 Load Balancer

## 1. L4 Load Balancer

**L4 = Transport Layer**

L4 Load Balancer works mainly with:

- IP address
- Port number
- TCP
- UDP
- Network connections

It does **not understand the HTTP request content** such as URL path or HTTP headers.

### Simple Flow

```text
Client
   |
   ↓
L4 Load Balancer
   |
   ├──→ Server 1
   ├──→ Server 2
   └──→ Server 3
```

### Example

```text
Client → 10.0.0.10:443
             |
             ↓
       L4 Load Balancer
             |
       ┌─────┼─────┐
       ↓     ↓     ↓
     Server Server Server
       1      2      3
```

The L4 load balancer mainly uses connection-level information such as:

```text
Source IP
Destination IP
Source Port
Destination Port
TCP/UDP
```

### Common L4 Load Balancers


| Provider     | L4 Load Balancer            |
| ------------ | --------------------------- |
| AWS          | Network Load Balancer (NLB) |
| Azure        | Azure Load Balancer         |
| Google Cloud | Network Load Balancer       |
| HAProxy      | Can operate at L4           |


AWS identifies NLB as a Layer 4 load balancer, while Azure identifies Azure Load Balancer as Layer 4 TCP/UDP. Google Cloud also provides Layer 4 network load balancing.

---

# 2. When Should I Use L4?

Use **L4** when:

- You need very high throughput.
- You need low latency.
- Your application uses TCP/UDP.
- You don't need URL-based routing.
- You don't need to inspect HTTP headers/cookies.
- You are load balancing non-HTTP applications.

### Examples

```text
Gaming server
    ↓
L4 Load Balancer
    ↓
Game Server 1 / 2 / 3
```

```text
VoIP application
    ↓
L4 Load Balancer
    ↓
Voice Servers
```

```text
TCP application
    ↓
L4 Load Balancer
    ↓
Backend Servers
```

---

# 3. L7 Load Balancer

**L7 = Application Layer**

L7 Load Balancer understands application-level protocols such as:

- HTTP
- HTTPS
- HTTP headers
- URL/path
- Hostname
- Cookies
- HTTP methods

### Simple Flow

```text
Client
   |
   ↓
L7 Load Balancer
   |
   ├── /api/*     → API Servers
   ├── /admin/*   → Admin Servers
   └── /images/*  → Image Servers
```

### Example

```text
https://example.com/api/users
             |
             ↓
       L7 Load Balancer
             |
             ↓
        API Server
```

```text
https://example.com/images/a.jpg
             |
             ↓
       L7 Load Balancer
             |
             ↓
       Image Server
```

L7 can make routing decisions based on HTTP request attributes such as URL path and host headers.

---

# 4. Common L7 Load Balancers


| Provider     | L7 Load Balancer                |
| ------------ | ------------------------------- |
| AWS          | Application Load Balancer (ALB) |
| Azure        | Application Gateway             |
| Google Cloud | Application Load Balancer       |
| NGINX        | L7 reverse proxy/load balancer  |
| HAProxy      | Can operate at L7               |


AWS describes ALB as a Layer 7 load balancer. Azure Application Gateway is also Layer 7, and Google Cloud Application Load Balancers provide HTTP/HTTPS request routing.

---

# 5. When Should I Use L7?

Use **L7** when:

- Your application uses HTTP/HTTPS.
- You need URL-based routing.
- You need hostname-based routing.
- You need HTTP header-based routing.
- You need cookie-based routing/session affinity.
- You need TLS/SSL termination.
- You need Web Application Firewall (WAF) integration.
- You have microservices with different URLs.

### Example

```text
                 Internet
                    |
                    ↓
             L7 Load Balancer
                    |
       ┌────────────┼────────────┐
       ↓            ↓            ↓
    /users        /orders      /images
       ↓            ↓            ↓
   User Service  Order Service Image Service
```

---

# 6. L4 vs L7 — Easy Comparison


| Feature                   | L4                               | L7               |
| ------------------------- | -------------------------------- | ---------------- |
| OSI Layer                 | Layer 4                          | Layer 7          |
| Main protocols            | TCP/UDP                          | HTTP/HTTPS       |
| Understands URL?          | ❌ No                             | ✅ Yes            |
| Understands HTTP headers? | ❌ No                             | ✅ Yes            |
| Understands cookies?      | ❌ No                             | ✅ Yes            |
| URL-based routing         | ❌ No                             | ✅ Yes            |
| Host-based routing        | ❌ No                             | ✅ Yes            |
| TLS termination           | Depends on product/configuration | Common           |
| WAF integration           | Usually not application-aware    | Common           |
| Latency/processing        | Generally lower                  | Generally higher |
| Non-HTTP traffic          | ✅ Yes                            | ❌ Generally no   |
| Microservice path routing | ❌ No                             | ✅ Yes            |


---

# 7. Which One Is Better?

There is **no universally best choice**.

Choose based on your requirement.

### Choose L4 when:

```text
Need TCP/UDP
      +
Need high throughput
      +
Need low latency
      +
Don't need HTTP inspection
```

→ **Use L4**

### Choose L7 when:

```text
Need HTTP/HTTPS
      +
Need URL/host routing
      +
Need HTTP inspection
      +
Need TLS termination/WAF
```

→ **Use L7**

---

# 8. Real-World Example

Suppose you have an e-commerce application.

```text
                    INTERNET
                       |
                       ↓
               L7 Load Balancer
                       |
          ┌────────────┼────────────┐
          ↓            ↓            ↓
        /api         /admin       /images
          ↓            ↓            ↓
       API Pods     Admin Pods    Image Pods
```

L7 is useful because it understands:

```text
/api
/admin
/images
```

and can send each request to the appropriate backend.

---

# 9. Example Where L4 Is Better

Suppose you have a gaming server:

```text
                 Players
                    |
                    ↓
             L4 Load Balancer
                    |
          ┌─────────┼─────────┐
          ↓         ↓         ↓
       Game 1     Game 2     Game 3
```

The load balancer doesn't need to understand:

```text
/api
/admin
/images
```

It only needs to distribute TCP/UDP connections.

So **L4 is a better fit**.

---

# 10. Can We Use L4 + L7 Together?

**Yes.**

A production architecture can use both.

```text
                    INTERNET
                       |
                       ↓
                L4 Load Balancer
                       |
                       ↓
                L7 Load Balancer
                       |
          ┌────────────┼────────────┐
          ↓            ↓            ↓
        /api         /admin       /shop
          ↓            ↓            ↓
       Backend      Backend      Backend
```

For example:

- **L4** handles TCP-level connection distribution.
- **L7** handles HTTP-level routing.

Cloud providers also support architectures combining different load-balancing layers/services.

---

# 11. Kubernetes Connection

Remember:

```text
Kubernetes ≠ L4
Kubernetes ≠ L7
```

Kubernetes is a **container orchestration platform**.

It provides networking components such as:

```text
Kubernetes
    |
    ├── Service
    |      ↓
    |    Pods
    |
    └── Ingress / Gateway
           ↓
       HTTP/HTTPS routing
```

A Kubernetes **Service** can provide L4-style TCP/UDP traffic distribution.

An **Ingress Controller** can provide L7 HTTP/HTTPS routing.

Therefore:

```text
Kubernetes = Platform
L4         = Traffic handling level
L7         = Traffic handling level
Service    = Kubernetes networking component
Ingress    = Kubernetes HTTP/HTTPS routing mechanism
```

---

# 12. Interview Answer

### What is the difference between L4 and L7 Load Balancer?

> **L4 Load Balancer works at the transport layer and distributes TCP/UDP connections using information such as IP addresses and ports. L7 Load Balancer works at the application layer and can inspect HTTP/HTTPS requests to perform routing based on URL paths, hostnames, headers, and other application-level information.**

### Which one would you choose?

> **For TCP/UDP workloads where I need high throughput and low-latency connection-level load balancing, I would choose L4. For HTTP/HTTPS applications where I need URL-based routing, TLS termination, WAF, or application-aware routing, I would choose L7.**

---

# Server Scaling + Latency + Availability + Multiple Request Handling — Interview Q&A

## 1. What is Scalability?

### Answer
Scalability is the ability of a system to handle increasing traffic, users, requests, or data by adding resources.

### Example
```text
1000 RPS → 10,000 RPS
        ↓
Add more application servers
        ↓
Load Balancer distributes traffic
```

---

## 2. What is Vertical Scaling?

### Answer
Vertical scaling means increasing the resources of an existing server.

### Example
```text
8 CPU + 16 GB RAM
        ↓
32 CPU + 64 GB RAM
```

### Advantages
- Simple
- Less application complexity

### Disadvantages
- Hardware limit
- Can become expensive
- Single server can become a failure point

---

## 3. What is Horizontal Scaling?

### Answer
Horizontal scaling means adding more servers instead of increasing the size of one server.

### Example
```text
             Load Balancer
                  |
        ---------------------
        |         |         |
     Server 1  Server 2  Server 3
```

### Best for
- High traffic
- Stateless applications
- Cloud/Kubernetes environments

---

## 4. Vertical vs Horizontal Scaling

| Feature | Vertical | Horizontal |
|---|---|---|
| Meaning | Bigger server | More servers |
| Limit | Hardware limit | Can scale across many servers |
| Complexity | Lower | Higher |
| Availability | Lower | Better |
| Example | 8 → 32 CPU | 2 → 10 servers |

### Interview Answer
> For high-traffic applications, I generally prefer horizontal scaling because I can add or remove servers based on demand and avoid depending on a single large server.

---

## 5. Why do we need a Load Balancer?

### Answer
A Load Balancer distributes incoming traffic across multiple backend servers.

### Example
```text
Users
  |
  ↓
Load Balancer
  |
  ├── Server 1
  ├── Server 2
  └── Server 3
```

### Benefits
- Distributes traffic
- Prevents one server from becoming overloaded
- Improves availability
- Helps horizontal scaling

---

## 6. What happens if one server goes down?

### Answer
The Load Balancer should detect the unhealthy server using health checks and stop sending traffic to it.

```text
             Load Balancer
                /      \
               ↓        ↓
          Server 1   Server 2
             ❌          ✅
                         ↑
                   Traffic goes here
```

---

## 7. What is a Health Check?

### Answer
A health check verifies whether a backend server is healthy enough to receive traffic.

### Example
```http
GET /health
```

Expected response:
```text
200 OK
```

If the server repeatedly fails the health check, the Load Balancer removes it from rotation.

---

# L4 vs L7 Load Balancing

## 8. What is an L4 Load Balancer?

### Answer
L4 works at the Transport Layer and primarily uses:

- IP address
- Port
- TCP
- UDP

It does not need to understand HTTP URL or headers.

### Example
```text
Client
  |
  ↓
L4 Load Balancer
  |
  ├── Server 1
  ├── Server 2
  └── Server 3
```

---

## 9. What is an L7 Load Balancer?

### Answer
L7 works at the Application Layer and understands HTTP/HTTPS information such as:

- URL
- Host
- Headers
- Cookies
- HTTP methods

### Example
```text
/api/*      → API Server
/admin/*    → Admin Server
/images/*   → Image Server
```

---

## 10. L4 vs L7 — Which one should I choose?

### Choose L4 when:
- Application uses TCP/UDP
- Very high throughput is required
- Low latency is important
- HTTP inspection is not required
- URL-based routing is not required

### Choose L7 when:
- Application uses HTTP/HTTPS
- URL-based routing is required
- Host/header/cookie routing is required
- TLS termination is required
- WAF integration is required

### Interview Answer
> I would choose L4 for TCP/UDP workloads where connection-level load balancing and low overhead are important. I would choose L7 for HTTP/HTTPS applications where application-aware routing is required.

---

## 11. Can L4 and L7 be used together?

### Answer
Yes.

### Example
```text
Internet
   |
   ↓
L4 Load Balancer
   |
   ↓
L7 Load Balancer
   |
   ├── /api
   ├── /admin
   └── /shop
```

L4 handles connection-level traffic.

L7 handles HTTP-level routing.

---

# Multiple Request Handling

## 12. How does a server handle multiple requests?

### Answer
A server can handle multiple requests using:

- Multiple worker processes
- Threads/thread pools
- Async/non-blocking I/O
- Connection pools
- Multiple server instances

### Example
```text
             Load Balancer
                  |
        ---------------------
        |         |         |
      Server    Server    Server
        |
    Worker Pool
    /    |    \
 Req1  Req2  Req3
```

The exact mechanism depends on the application runtime/framework.

---

## 13. What happens if 10,000 requests arrive simultaneously?

### Answer
The system does not necessarily process all 10,000 requests at exactly the same instant.

Typical flow:

```text
10,000 requests
       ↓
Load Balancer
       ↓
Application Servers
       ↓
Workers / Async I/O
       ↓
Queue if capacity is exhausted
```

If capacity is exceeded:

- Requests wait
- Queue grows
- Latency increases
- Timeouts can occur
- Requests can eventually be rejected

---

## 14. What is a Worker Pool?

### Answer
A Worker Pool is a controlled number of workers that process incoming tasks.

### Example
```text
1000 requests
      ↓
    Queue
      ↓
Worker Pool
 ┌──┬──┬──┬──┐
 W1 W2 W3 W4
```

Instead of creating unlimited workers, the system controls concurrency.

---

## 15. What happens when all workers are busy?

### Answer
New requests wait in a queue or are rejected after a configured limit.

```text
Requests
   ↓
Queue
   ↓
Workers
 W1 W2 W3 W4
 ↑  ↑  ↑  ↑
BUSY BUSY BUSY BUSY
```

If the queue becomes full:

```text
Request → Reject / Timeout
```

---

## 16. What is a Connection Pool?

### Answer
A connection pool maintains reusable connections instead of creating a new connection for every request.

### Example
```text
100 API requests
       ↓
Connection Pool
       ↓
10 reusable DB connections
       ↓
     MySQL
```

This reduces connection creation overhead.

---

## 17. What happens if the DB connection pool is exhausted?

### Answer
New application requests wait for an available database connection.

```text
DB Pool Full
    ↓
Requests wait
    ↓
Latency increases
    ↓
Timeouts
    ↓
Possible cascading failure
```

### Possible solutions
- Correct pool sizing
- Query optimization
- Connection timeouts
- Proper connection release
- Database scaling
- Caching

---

# Latency and Throughput

## 18. What is Latency?

### Answer
Latency is the time taken to complete a request.

### Example
```text
Request sent
    ↓
100 ms
    ↓
Response received
```

Latency = 100 ms

---

## 19. What is Throughput?

### Answer
Throughput is how much work a system processes per unit of time.

### Example
```text
1000 requests/second
```

Throughput = 1000 RPS.

### Difference
```text
Latency   = How long one request takes
Throughput = How many requests are processed
```

---

## 20. What are P50, P95 and P99 latency?

### Example
```text
P50 = 100 ms
P95 = 250 ms
P99 = 500 ms
```

Meaning:

- P50 → 50% requests finish within 100 ms
- P95 → 95% requests finish within 250 ms
- P99 → 99% requests finish within 500 ms

P99 is useful for understanding slow-tail requests.

---

## 21. How do you reduce API latency?

### Answer
First identify the actual bottleneck.

### Examples

```text
Slow DB query
     ↓
Optimize query / Add index
```

```text
Repeated DB reads
     ↓
Redis / Cache
```

```text
Large static files
     ↓
CDN
```

```text
Slow external service
     ↓
Timeout + Cache + Async processing
```

```text
Users far from server
     ↓
Regional deployment / CDN
```

---

## 22. How would you debug a slow API?

### Answer
Break the request into components and measure each one.

```text
Client
  ↓
Load Balancer
  ↓
Application
  ↓
Database
  ↓
External API
```

Check:

- API latency
- CPU
- Memory
- Database query time
- Network latency
- External API latency
- Connection pool
- Queue depth
- Error rate

### Interview Answer
> I would first identify the bottleneck using metrics and tracing instead of blindly adding more servers.

---

# Availability

## 23. What is Availability?

### Answer
Availability is the percentage of time a system remains operational and accessible.

Example:

```text
99.9% Availability
```

means the system is designed to be unavailable for only a small fraction of the measurement period.

---

## 24. How do you improve availability?

### Answer
Use redundancy and remove single points of failure.

```text
             Load Balancer
                  |
        ---------------------
        |         |         |
      App 1     App 2     App 3
        |         |         |
        ---------------------
                  |
               Database
```

Use:

- Multiple application instances
- Health checks
- Database replication
- Failover
- Multiple availability zones
- Backups
- Monitoring
- Auto scaling

---

## 25. What is High Availability?

### Answer
High Availability means designing the system so failure of an individual component does not bring down the complete system.

### Example
```text
AZ-1              AZ-2

App 1              App 2
App 3              App 4
  \                 /
   \               /
     Load Balancer
```

If AZ-1 fails, AZ-2 can continue serving traffic.

---

## 26. What is a Single Point of Failure (SPOF)?

### Answer
A Single Point of Failure is a component whose failure can bring down the whole system.

### Bad Design
```text
Users
  ↓
Single Server
  ↓
Database
```

If the server fails → application unavailable.

### Better
```text
          Load Balancer
          /          \
      Server 1      Server 2
```

---

# Traffic Spikes

## 27. How would you handle a sudden 10x traffic spike?

### Answer
I would use:

```text
Traffic Spike
     ↓
Load Balancer
     ↓
Auto Scaling
     ↓
More App Instances
     ↓
Cache
     ↓
Queue for Async Work
     ↓
Database Scaling
```

Example:

```text
1000 RPS
   ↓
10,000 RPS
   ↓
Auto Scale
   ↓
10 → 50 application instances
```

---

## 28. What is Auto Scaling?

### Answer
Auto Scaling automatically adds or removes instances based on configured metrics or policies.

### Example
```text
CPU > 70%
   ↓
Add instances
```

```text
CPU < 30%
   ↓
Remove instances
```

Other scaling signals can include request count and queue depth.

---

## 29. Why can Auto Scaling fail during a sudden spike?

### Answer
Because new instances need time to start.

```text
Traffic suddenly increases
       ↓
Auto Scaling triggered
       ↓
New instances starting
       ↓
Capacity still insufficient
       ↓
Latency increases
```

### Solutions
- Keep some warm capacity
- Faster startup
- Pre-built images
- Predictive/scheduled scaling
- Proper readiness checks

---

# Stateless Applications

## 30. Why should application servers be stateless?

### Answer
If every server can process every request, horizontal scaling becomes easier.

```text
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
```

No server owns the user's session.

---

## 31. What is the problem with storing sessions in server memory?

### Answer

```text
User
 ↓
Server 1
 ↓
Session stored locally
```

Next request:

```text
User
 ↓
Server 2
```

Server 2 may not have the session.

### Solution

Store session state externally:

```text
Application Servers
       |
       ↓
     Redis
       |
    Sessions
```

---

# Database Scaling

## 32. Application servers are scaled but database is slow. What do you do?

### Answer
Scaling application servers alone will not solve a database bottleneck.

Check:

1. Slow queries
2. Missing indexes
3. Connection pool
4. Read/write ratio
5. Cache hit ratio
6. Database CPU/memory
7. Replication
8. Sharding if required

### Example

```text
100 App Servers
       ↓
    1 Database
       ↓
    Bottleneck
```

---

## 33. How do you scale read-heavy traffic?

### Answer

Use caching and read replicas.

```text
Application
    |
    ├── Redis Cache
    |
    ↓
Read Replica 1
Read Replica 2
Read Replica 3
```

---

## 34. How do you scale writes?

### Answer

Depending on the bottleneck:

- Optimize queries
- Batch writes where appropriate
- Queue asynchronous work
- Partition/shard data
- Scale database infrastructure
- Reduce unnecessary writes

The exact solution depends on workload and consistency requirements.

---

# Queue and Async Processing

## 35. Why use a Message Queue?

### Answer
A queue separates request handling from slow/background processing.

### Without Queue

```text
Client
  ↓
API
  ↓
Slow Task
  ↓
Response
```

### With Queue

```text
Client
  ↓
API
  ↓
Queue
  ↓
Worker
  ↓
Slow Task
```

The API can respond quickly while workers process the task asynchronously.

---

## 36. How does a Queue help during traffic spikes?

### Answer
A queue acts as a buffer.

```text
1000 requests/sec
       ↓
     Queue
       ↓
500 tasks/sec processing
```

Excess work waits in the queue instead of immediately overwhelming workers or downstream services.

---

## 37. What is a Dead Letter Queue (DLQ)?

### Answer
A DLQ stores messages that repeatedly fail processing.

```text
Queue
  ↓
Worker
  ↓
Failure
  ↓
Retry
  ↓
Failure
  ↓
DLQ
```

It prevents permanently failing messages from continuously blocking normal processing.

---

# Failure Handling

## 38. What is a Timeout?

### Answer
A timeout limits how long a request waits for a response.

### Example

```text
API → Payment Service
       |
       | 2 seconds
       ↓
    Timeout
```

Without timeouts, stuck requests can consume workers and connections.

---

## 39. What is a Retry?

### Answer
A retry attempts a failed operation again.

```text
Request
  ↓
Failure
  ↓
Retry
  ↓
Success
```

Retries should have limits and backoff.

---

## 40. What is Exponential Backoff?

### Answer
The waiting time increases after each failed retry.

```text
Retry 1 → 100 ms
Retry 2 → 200 ms
Retry 3 → 400 ms
Retry 4 → 800 ms
```

It helps prevent many clients from retrying simultaneously.

---

## 41. What is a Retry Storm?

### Answer
When a service fails, many clients retry at the same time, increasing load and making the failure worse.

```text
Service Failure
      ↓
1000 clients retry
      ↓
Higher traffic
      ↓
Service overloaded
      ↓
More failures
      ↓
More retries
```

### Solutions
- Exponential backoff
- Jitter
- Retry limits
- Circuit breaker

---

## 42. What is a Circuit Breaker?

### Answer
A circuit breaker stops repeatedly calling an unhealthy downstream service.

```text
Normal
  ↓
Failures increase
  ↓
Circuit OPEN
  ↓
Stop requests
  ↓
Wait
  ↓
Test request
  ↓
Recovered → CLOSED
```

It helps prevent cascading failures.

---

# Rate Limiting

## 43. What is Rate Limiting?

### Answer
Rate limiting restricts how many requests a client can make during a specific period.

### Example

```text
User → 100 requests/minute
```

Request 101:

```text
HTTP 429 Too Many Requests
```

Rate limiting protects the service from overload and abuse.

---

## 44. Where should Rate Limiting be implemented?

### Common architecture

```text
Client
  ↓
API Gateway / Load Balancer
  ↓
Rate Limiter
  ↓
Application
```

For distributed systems, a shared store such as Redis can be used depending on the design.

---

# Kubernetes

## 45. What is Kubernetes?

### Answer
Kubernetes is a container orchestration platform.

It manages:

- Containers
- Pods
- Deployments
- Services
- Scaling
- Health checks
- Scheduling

### Important

```text
Kubernetes ≠ L4
Kubernetes ≠ L7
```

Kubernetes is the platform.

L4/L7 describe traffic-handling levels.

---

## 46. How does Kubernetes help with scaling?

### Example

```text
                 Kubernetes
                      |
                  Deployment
                      |
          ---------------------
          |         |         |
        Pod 1     Pod 2     Pod 3
```

When traffic increases, Kubernetes can increase the number of pods using an autoscaling mechanism.

---

## 47. Kubernetes Service vs Ingress?

### Service

```text
Service
   ↓
Pods
```

A Service provides stable networking to a group of Pods and can handle TCP/UDP traffic.

### Ingress

```text
Ingress
   ↓
HTTP/HTTPS
   ↓
URL/Host Routing
   ↓
Services
```

Ingress is used for HTTP/HTTPS routing.

---

# Important Scenario Questions

## 48. Server CPU is 90%. What will you do?

### Answer

First identify whether CPU is actually the bottleneck.

Then:

```text
Check Metrics
     ↓
Find Expensive Operation
     ↓
Optimize
     ↓
Cache if applicable
     ↓
Horizontal Scaling
     ↓
Auto Scaling
```

Do not blindly add servers.

---

## 49. CPU is only 30%, but API latency is 5 seconds. Why?

### Answer
CPU may not be the bottleneck.

Possible causes:

- Slow DB query
- DB connection pool exhausted
- External API slow
- Network latency
- Lock contention
- Worker pool exhausted
- Queue backlog

---

## 50. Traffic doubled but latency increased 10x. Why?

### Answer
The system may have crossed a capacity threshold.

### Example

```text
Traffic increases 2x
       ↓
DB connection pool exhausted
       ↓
Requests wait
       ↓
Queue grows
       ↓
Latency increases dramatically
```

---

## 51. One backend server is slow. What should the Load Balancer do?

### Answer
If the load balancer supports appropriate health/performance-based routing, it can reduce or stop traffic to unhealthy instances.

At minimum, health checks should prevent clearly unhealthy servers from receiving traffic.

---

## 52. Round Robin vs Least Connections?

### Round Robin

```text
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
```

Useful when servers and workloads are relatively uniform.

### Least Connections

```text
Server 1 → 10 connections
Server 2 → 3 connections
Server 3 → 7 connections

New Request → Server 2
```

Useful when request duration varies significantly.

---

# Capacity Estimation

## 53. How would you estimate server capacity?

### Answer

```text
Users
 ↓
Requests/User
 ↓
Requests Per Second
 ↓
Peak Traffic
 ↓
Server Capacity
 ↓
Number of Servers
```

### Example

```text
Peak traffic = 100,000 RPS

One server = 2,000 RPS

100,000 / 2,000 = 50 servers
```

Then add appropriate capacity/headroom.

---

## 54. Average Traffic vs Peak Traffic — Which one do you use?

### Answer
For capacity planning, peak traffic is important.

### Example

```text
Average = 2,000 RPS
Peak    = 10,000 RPS
```

If you size only for 2,000 RPS, the system can fail during peak traffic.

---

# CTO-Level Scenario Questions

## 55. System handles 1,000 RPS. Tomorrow traffic will become 50,000 RPS. What will you do?

### Answer

I would approach it in layers:

```text
1. Capacity estimation
        ↓
2. Load testing
        ↓
3. Horizontal application scaling
        ↓
4. Load Balancer
        ↓
5. Redis / Cache
        ↓
6. Database optimization
        ↓
7. Read replicas / Sharding if required
        ↓
8. Queue for async workloads
        ↓
9. Auto Scaling
        ↓
10. Monitoring + Alerts
```

---

## 56. How would you handle 50,000 concurrent requests?

### Answer

I would not assume one server should process all requests.

```text
50K Requests
      ↓
Load Balancer
      ↓
Multiple App Servers
      ↓
Workers / Async Processing
      ↓
Cache
      ↓
Database
```

Then identify the bottleneck:

```text
CPU?
Memory?
Database?
Connections?
Workers?
Network?
Queue?
```

Scale the actual bottleneck.

---

## 57. How would you design a highly available API?

### Answer

```text
              Internet
                  |
                  ↓
          Load Balancer
             /      \
            ↓        ↓
         App 1     App 2
            \        /
             \      /
              Database
```

Then add:

- Multiple instances
- Health checks
- Auto scaling
- Database replication/failover
- Monitoring
- Timeouts
- Circuit breakers
- Backups

---

## 58. How would you reduce latency for users in India, Europe and the US?

### Answer

Use geographically distributed infrastructure.

```text
             Global Routing
             /      |      \
            ↓       ↓       ↓
         India   Europe    US
         Region  Region   Region
```

Use CDN for static content and route users toward an appropriate region.

---

## 59. What if the Load Balancer itself fails?

### Answer

The Load Balancer should not be a Single Point of Failure.

Use a highly available/redundant load-balancing architecture.

```text
             Traffic
                |
        ----------------
        |              |
       LB1            LB2
        |              |
        ----------------
                |
             Servers
```

---

## 60. Explain your complete scaling strategy.

### Answer

```text
                    USERS
                      |
                      ↓
              Global Routing / CDN
                      |
                      ↓
               Load Balancer
                      |
          -------------------------
          |           |           |
       App 1        App 2       App 3
          |           |           |
          -------- Cache ---------
                      |
                  Database
                 /        \
          Read Replica   Primary

        Async Work → Queue → Workers
```

### Explain it like this:

```text
More traffic
    ↓
Horizontal scaling

Higher latency
    ↓
Find the bottleneck

Read-heavy workload
    ↓
Cache + Read Replicas

Heavy background work
    ↓
Queue + Workers

Server failure
    ↓
Health Checks + Failover

Traffic spike
    ↓
Auto Scaling

Global users
    ↓
Multi-region + CDN
```

---

# Top 15 Questions to Prepare

1. What is scalability?
2. Vertical scaling vs horizontal scaling?
3. Why do we need a Load Balancer?
4. L4 vs L7 Load Balancer?
5. When would you choose L4?
6. When would you choose L7?
7. How does a server handle multiple requests?
8. How would you handle 50,000 concurrent requests?
9. Latency vs throughput?
10. How do you reduce API latency?
11. How do you achieve high availability?
12. What happens when one server fails?
13. How do you handle a sudden 10x traffic spike?
14. What happens when the database becomes the bottleneck?
15. How would you design a system that scales from 1,000 to 50,000 RPS?

# Strong Answer

> I would first identify the growth dimension: RPS, concurrent connections, data volume, or geographic traffic. For application scaling, I would keep services stateless and horizontally scale instances behind a Load Balancer. For HTTP workloads, L7 is useful for application-aware routing, while L4 is suitable for TCP/UDP connection-level workloads. I would use caching to reduce repeated reads, queues for asynchronous work, database replicas or sharding when the database becomes the bottleneck, and auto-scaling for changing traffic. For availability, I would use multiple instances, health checks, redundancy, and failover. Finally, I would monitor latency, throughput, error rate, CPU, memory, connection pools, and queue depth so that I scale the actual bottleneck instead of simply adding more servers.