# System Design — Fundamentals

[← System Design Index](../README.md)

---

## What is redundancy?

Having backups of critical components so the system never depends on a single point. If one fails, another takes over without downtime.

**Example:** Two database instances running the same data. If the primary crashes, the secondary is already up-to-date and ready.

```mermaid
graph TD
    Client --> LB[Load Balancer]
    LB --> A[Server A]
    LB --> B[Server B ✦ backup]
    A --> DB1[(Primary DB)]
    B --> DB1
    DB1 -- replication --> DB2[(Secondary DB ✦ backup)]
```

---

## What is failover?

The process where the system automatically switches from a failed component to its backup, with no manual intervention required.

**Example:** Primary database crashes → health check detects failure → traffic is rerouted to the secondary in seconds.

```mermaid
sequenceDiagram
    participant App
    participant Primary DB
    participant Secondary DB
    participant Health Check

    Health Check->>Primary DB: ping
    Primary DB-->>Health Check: ✗ no response
    Health Check->>App: promote secondary
    App->>Secondary DB: now writing here
    Note over Secondary DB: Secondary becomes new Primary
```

---

## Explain load balancer

Distributes incoming traffic across multiple servers to improve scalability and availability. Prevents any single server from being overwhelmed. If a server fails, the load balancer stops sending traffic to it.

**Common algorithms:**
- **Round Robin** — each server gets requests in turn
- **Least Connections** — sends to the server with fewest active connections
- **IP Hash** — same client always hits the same server (useful for session stickiness)

```mermaid
graph LR
    Users -->|requests| LB[Load Balancer]
    LB -->|round robin| S1[Server 1]
    LB -->|round robin| S2[Server 2]
    LB -->|round robin| S3[Server 3]
    S1 & S2 & S3 --> DB[(Shared DB)]
```

---

## Vertical Scaling vs Horizontal Scaling

| | Vertical (Scale Up) | Horizontal (Scale Out) |
|-|---------------------|------------------------|
| How | Bigger machine (more CPU, RAM) | More machines |
| Limit | Hardware ceiling | Virtually unlimited |
| Cost | Expensive at high end | More predictable |
| Downtime | Often requires restart | Zero downtime possible |
| Complexity | Simple | Requires load balancer, distributed state |

```mermaid
graph TD
    subgraph Vertical
        V1[Server\n4 CPU / 16GB] --> V2[Server\n16 CPU / 64GB]
    end

    subgraph Horizontal
        H[Load Balancer] --> H1[Server 1]
        H --> H2[Server 2]
        H --> H3[Server 3]
    end
```

---

## How would you ensure high availability?

Eliminate single points of failure at every layer of the stack.

**Strategy:**
1. Multiple app instances behind a load balancer
2. Database primary/secondary replication with automatic failover
3. Multi-AZ (or multi-region) deployment for disaster recovery
4. Health checks + circuit breakers to isolate failures fast

```mermaid
graph TD
    DNS[Route 53 / DNS Failover]

    subgraph Region A - Primary
        LB_A[Load Balancer] --> App1[App Instance 1]
        LB_A --> App2[App Instance 2]
        App1 & App2 --> DB_P[(Primary DB)]
        DB_P -- replication --> DB_S[(Secondary DB)]
    end

    subgraph Region B - DR
        LB_B[Load Balancer] --> App3[App Instance]
        App3 --> DB_DR[(Replica DB)]
    end

    DNS -->|healthy| LB_A
    DNS -->|failover| LB_B
```