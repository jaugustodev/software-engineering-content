# System Design Interview: An Insider's Guide — Study Repository

**Author:** Alex Xu  
**Start here:** [INDEX.md](INDEX.md)

---

## Book Overview

"System Design Interview: An Insider's Guide" by Alex Xu is a practical guide designed to help software engineers ace system design interviews. It covers foundational scaling concepts, estimation techniques, a structured interview framework, and deep dives into 12 real-world system designs — from rate limiters and consistent hashing to YouTube and Google Drive.

The book bridges the gap between theoretical distributed systems knowledge and the practical communication skills needed to perform well in a 45–60 minute system design interview.

---

## Learning Goals

By the end of this study repository, you will be able to:

- Scale a system from a single server to millions of users
- Perform quick back-of-the-envelope capacity estimations
- Follow a structured 4-step framework during any system design interview
- Design core infrastructure components (rate limiters, consistent hashing, key-value stores)
- Architect real-world systems (URL shortener, web crawler, notification system, news feed, chat, autocomplete, YouTube, Google Drive)
- Articulate trade-offs clearly and confidently under interview pressure

---

## Table of Contents

| Chapter | Title | Key Topics |
|---------|-------|------------|
| [Chapter 01](./chapters/01-scale.md) | Scale from Zero to Millions of Users | DNS, load balancers, databases, caching, CDN, sharding, stateless design |
| [Chapter 02](./chapters/02-estimation.md) | Back-of-the-Envelope Estimation | Powers of two, latency numbers, QPS/storage calculations |
| [Chapter 03](./chapters/03-framework.md) | A Framework for System Design Interviews | 4-step process, scoping, high-level design, deep dive, wrap-up |
| [Chapter 04](./chapters/04-rate-limiter.md) | Design a Rate Limiter | Token bucket, leaky bucket, fixed/sliding window, distributed rate limiting |
| [Chapter 05](./chapters/05-consistent-hashing.md) | Design Consistent Hashing | Virtual nodes, hash rings, data replication across nodes |
| [Chapter 06](./chapters/06-kv-store.md) | Design a Key-Value Store | Partitioning, replication, consistency, Dynamo-style architecture |
| [Chapter 07](./chapters/07-unique-id-generator.md) | Design a Unique ID Generator | UUID, Snowflake, multi-master replication, ticket servers |
| [Chapter 08](./chapters/08-url-shortener.md) | Design a URL Shortener | Hashing, base62, 301 vs 302 redirects, rate limiting |
| [Chapter 09](./chapters/09-web-crawler.md) | Design a Web Crawler | BFS, URL frontier, DNS caching, politeness, content deduplication |
| [Chapter 10](./chapters/10-notification-system.md) | Design a Notification System | APNs, FCM, SMS, email, templates, event tracking, retry, dedup |
| [Chapter 11](./chapters/11-news-feed.md) | Design a News Feed System | Fan-out on write vs read, graph DB, hybrid approach |
| [Chapter 12](./chapters/12-chat-system.md) | Design a Chat System | WebSocket, presence service, message sync, group chat |
| [Chapter 13](./chapters/13-search-autocomplete.md) | Design a Search Autocomplete System | Trie, top-k at each node, AJAX, data gathering vs query service |
| [Chapter 14](./chapters/14-youtube.md) | Design YouTube | 6-component DAG transcoding, adaptive bitrate, CDN cost optimization |
| [Chapter 15](./chapters/15-google-drive.md) | Design Google Drive | Block server pipeline, delta sync, conflict resolution, metadata schema |
| [Chapter 16](./chapters/16-next-steps.md) | The Learning Continues | Real-world companies, foundational papers, advanced distributed systems topics |

---

## Suggested Study Path

### Phase 1: Foundation (Days 1–3)
Start with the building blocks — these chapters underpin everything else.
1. **Chapter 01** — Understand how systems scale incrementally
2. **Chapter 02** — Practice estimation until it becomes natural
3. **Chapter 03** — Internalize the 4-step interview framework

### Phase 2: Core Components (Days 4–7)
Learn the reusable design patterns that appear in almost every system.
4. **Chapter 04** — Rate Limiter (algorithms, distributed enforcement)
5. **Chapter 05** — Consistent Hashing (used in caches, databases, load balancers)
6. **Chapter 06** — Key-Value Store (the anatomy of systems like Dynamo/Redis)
7. **Chapter 07** — Unique ID Generator (Snowflake, distributed coordination)

### Phase 3: Classic System Designs (Days 8–12)
Apply foundations to well-known problems.
8. **Chapter 08** — URL Shortener (hash functions, redirects, rate limits)
9. **Chapter 09** — Web Crawler (BFS, deduplication, politeness)
10. **Chapter 10** — Notification System (multi-channel delivery, fan-out)
11. **Chapter 11** — News Feed System (fan-out trade-offs, real-time delivery)

### Phase 4: Advanced Systems (Days 13–16)
Tackle the most complex, large-scale designs.
12. **Chapter 12** — Chat System (WebSocket, presence, group chat)
13. **Chapter 13** — Search Autocomplete (trie, distributed caching)
14. **Chapter 14** — YouTube (video pipeline, CDN, adaptive streaming)
15. **Chapter 15** — Google Drive (chunking, sync, conflict resolution)

### Phase 5: Review & Mock Interviews (Days 17–21)
- Review flashcards for all chapters
- Do timed mock sessions using each chapter's interview notes
- Re-read cheatsheets the day before interviews

---

## Key Concepts Covered Throughout the Book

### Scaling Patterns
- Vertical vs. horizontal scaling
- Stateless vs. stateful architecture
- Sharding and consistent hashing
- Replication (master-slave, multi-master)
- CDN and caching layers

### Communication & Messaging
- HTTP polling vs. long polling vs. WebSocket vs. Server-Sent Events
- Message queues (async decoupling)
- Fan-out on write vs. fan-out on read

### Storage
- SQL vs. NoSQL trade-offs
- Key-value stores, document stores, graph databases
- Blob storage for media
- Block-level storage for files

### Reliability & Availability
- Single points of failure
- Replication and failover
- Data center redundancy
- CAP theorem trade-offs

### Performance
- Caching strategies (write-through, write-back, write-around, read-through)
- Cache eviction (LRU, LFU, FIFO)
- Back-of-the-envelope estimation
- Latency numbers every engineer should know

---

## Common Interview Patterns

| Pattern | When to Apply | Example Systems |
|---------|--------------|-----------------|
| Fan-out | Delivering content to many users | News feed, notifications |
| Rate limiting | Preventing abuse | API gateways, login systems |
| Consistent hashing | Distributing load evenly | Distributed caches, databases |
| Snowflake ID | Unique IDs at scale | Tweets, messages, events |
| Two-phase commit | Distributed transactions | Payment systems |
| CQRS | Read/write separation | News feed, dashboards |
| Sharding | Horizontal database scaling | User data, message history |
| Bloom filter | Fast set membership checks | Web crawlers, caches |
| Trie | Prefix-based search | Autocomplete, spell check |
| Merkle tree | Data consistency verification | Distributed KV stores |

---

## Quick Navigation

- **Estimation shortcuts** → [Chapter 02: Back-of-the-Envelope Estimation](./chapters/02-estimation.md)
- **Interview framework** → [Chapter 03: System Design Framework](./chapters/03-framework.md)
- **Rate limiting algorithms** → [Chapter 04: Rate Limiter](./chapters/04-rate-limiter.md)
- **Consistent hashing & virtual nodes** → [Chapter 05: Consistent Hashing](./chapters/05-consistent-hashing.md)
- **Latency numbers** → [Chapter 02: Back-of-the-Envelope Estimation](./chapters/02-estimation.md)
- **Fan-out trade-offs** → [Chapter 11: News Feed System](./chapters/11-news-feed.md)
- **WebSocket vs polling** → [Chapter 12: Chat System](./chapters/12-chat-system.md)
- **Video transcoding pipeline (6 components)** → [Chapter 14: YouTube](./chapters/14-youtube.md)
- **Block-level delta sync** → [Chapter 15: Google Drive](./chapters/15-google-drive.md)
- **Notification channels (APNs/FCM/SMS/Email)** → [Chapter 10: Notification System](./chapters/10-notification-system.md)
