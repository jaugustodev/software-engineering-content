# CAP Theorem

The CAP Theorem states that a distributed system can only fully guarantee **two** of these three properties at the same time:

| Property | Definition |
|---|---|
| **C — Consistency** | Every read receives the most recent write (or an error) |
| **A — Availability** | Every request receives a response — never a timeout error |
| **P — Partition Tolerance** | The system keeps operating even when network failures split nodes |

---

## Why P is not optional

In real distributed systems, network partitions are inevitable. Switches fail, datacenters lose connectivity, latencies spike. Therefore, **P is mandatory** — the real choice is between C and A *when a partition occurs*.

```
         Network partition
              │
    ┌─────────┴──────────┐
    │                    │
    ↓                    ↓
Node A (write)        Node B (read)
    │                    │
    X ── network down ── X
```

When the network fails, the system must choose:
- **CP**: block reads/writes on B until sync is restored → consistent, but unavailable
- **AP**: allow reads on B with potentially stale data → available, but inconsistent

---

## CP vs AP in practice

### CP — Consistency + Partition Tolerance

The system refuses to respond if it cannot guarantee up-to-date data.

**When to use:** financial transactions, flight reservations, inventory management, systems where wrong data causes real harm.

**Trade-off:** during a partition, part of the system becomes unavailable.

**Real-world examples:**

| System | Why CP? |
|---|---|
| HBase | blocks operations during region server re-election |
| Zookeeper | quorum required for writes; without quorum, refuses |
| etcd / Consul | cluster coordination; stale data would break the cluster |
| MongoDB (write concern majority) | only confirms write after majority of replicas acknowledge |

```
Scenario: bank payment

Client → Node A (withdraw $100)
           │
           X ← partition
           │
         Node B (stale balance)

CP: Node B returns 503 error until network recovers
AP: Node B approves withdrawal → negative balance
```

### AP — Availability + Partition Tolerance

The system always responds, but may return stale data (eventual consistency).

**When to use:** social media feeds, shopping carts, product catalogs, systems where degraded experience is better than an error.

**Trade-off:** clients may see stale data for a few seconds or minutes.

**Real-world examples:**

| System | Why AP? |
|---|---|
| Cassandra | always accepts writes; reconciles conflicts later (LWW or CRDT) |
| DynamoDB | eventually consistent reads by default |
| CouchDB | multi-master; resolves conflicts with versioning |
| DNS | propagates changes over minutes/hours; always responds |

```
Scenario: Twitter timeline

User publishes tweet → Node A
           │
           X ← partition
           │
         Node B (replica)

AP: followers on Node B see tweet with 30s delay → acceptable
CP: everyone's timeline is unavailable until network recovers → unacceptable
```

---

## Databases by category

```
         CONSISTENCY
              ▲
              │
    CP        │        (CA - only exists without P)
  HBase  ◄───┼───► PostgreSQL (single node)
  etcd        │     MySQL (single node)
  Zookeeper   │
              │
──────────────┼──────────────► AVAILABILITY
              │
    AP        │
 Cassandra ◄──┤
  DynamoDB    │
  CouchDB     │
              │
         PARTITION
         (always present in distributed systems)
```

**Do CA databases exist?** Only in single-node setups. PostgreSQL, MySQL, and Oracle run as CA on a single server — consistent and available, but no partition tolerance because there's no network between nodes.

---

## Beyond CAP: the PACELC model

CAP only describes behavior *during a partition*. The **PACELC** model also covers the normal case:

```
if Partition → choose between Availability vs Consistency
Else (normal operation) → choose between Latency vs Consistency
```

| System | During partition | Normal operation |
|---|---|---|
| DynamoDB | AP | EL (favors latency) |
| Cassandra | AP | EL (favors latency) |
| MongoDB | CP | EC (favors consistency) |
| Spanner (Google) | CP | EC (favors consistency) |

PACELC is more useful for system design because most decisions happen during *normal operation*, not during failures.

---

## Decision framework for system design

```
Question 1: does wrong data cause financial or integrity damage?
├── Yes → CP required
│         (payments, inventory, reservations, auth tokens)
└── No  → AP is acceptable
          (feeds, catalogs, counters, recommendations)

Question 2: does the system need coordination between nodes?
├── Yes → CP (distributed locks, leader election, consensus)
└── No  → AP may be sufficient

Question 3: what is the cost of a wrong answer?
├── High (money lost, data corrupted) → CP
└── Low (user sees a post 1s late)    → AP
```

### Example: e-commerce system design

| Component | Choice | Reason |
|---|---|---|
| Inventory (stock levels) | CP | selling out-of-stock items has real cost |
| Shopping cart | AP | phantom item in cart is tolerable |
| Product catalog | AP | stale price for 30s is acceptable |
| Payment processing | CP | financial consistency is mandatory |
| Order history | AP | user seeing order with 5s delay is fine |
| User session | CP | stale session can allow unauthorized access |

---

## Tunable Consistency

Cassandra and DynamoDB let you adjust consistency per operation — it's not always one or the other:

```
Cassandra write/read levels:
  ONE    → fastest, least consistent   (AP)
  QUORUM → balanced                    (between CP and AP)
  ALL    → slowest, most consistent    (CP)

Rule: write QUORUM + read QUORUM = guaranteed consistency
      (majority of nodes must confirm)
```

This lets different components of the same system operate at different consistency levels.

---

## Key points for interviews

- **P is not optional** in distributed systems — the real choice is C vs A during a partition
- **CA databases** are single-node; in multi-node setups, you always have P
- **Eventual consistency ≠ no consistency** — it's consistency with a delay
- **PACELC** is more complete than CAP for architecture decisions
- **Tunable consistency** (Cassandra, DynamoDB) allows per-operation granularity
- In interviews, justify CP vs AP choices based on **business impact**, not technical preference
