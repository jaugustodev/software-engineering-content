# System Design Interview — Alex Xu: Study Guide Index

## Study Path

Read chapters in this order. Each chapter lists what you should study first.

| # | Chapter | Prerequisites | File |
|---|---------|--------------|------|
| 1 | Scale from Zero to Millions of Users | None | [Ch01 - Scale from Zero to Millions](chapters/01-scale.md) |
| 2 | Back-of-the-Envelope Estimation | Ch01 | [Ch02 - Back-of-the-Envelope Estimation](chapters/02-estimation.md) |
| 3 | A Framework for System Design Interviews | Ch01, Ch02 | [Ch03 - A Framework for System Design Interviews](chapters/03-framework.md) |
| 4 | Design a Rate Limiter | Ch01, Ch02, Ch03 | [Ch04 - Design a Rate Limiter](chapters/04-rate-limiter.md) |
| 5 | Design Consistent Hashing | Ch01, Ch02 | [Ch05 - Design Consistent Hashing](chapters/05-consistent-hashing.md) |
| 6 | Design a Key-Value Store | Ch05 | [Ch06 - Design a Key-Value Store](chapters/06-kv-store.md) |
| 7 | Design a Unique ID Generator in Distributed Systems | Ch01, Ch02 | [Ch07 - Design a Unique ID Generator](chapters/07-unique-id-generator.md) |
| 8 | Design a URL Shortener | Ch01, Ch05 | [Ch08 - Design a URL Shortener](chapters/08-url-shortener.md) |
| 9 | Design a Web Crawler | Ch01, Ch02 | [Ch09 - Design a Web Crawler](chapters/09-web-crawler.md) |
| 10 | Design a Notification System | Ch01, Ch02 | [Ch10 - Design a Notification System](chapters/10-notification-system.md) |
| 11 | Design a News Feed System | Ch01, Ch05 | [Ch11 - Design a News Feed System](chapters/11-news-feed.md) |
| 12 | Design a Chat System | Ch01, Ch02 | [Ch12 - Design a Chat System](chapters/12-chat-system.md) |
| 13 | Design a Search Autocomplete System | Ch01, Ch05 | [Ch13 - Design a Search Autocomplete System](chapters/13-search-autocomplete.md) |
| 14 | Design YouTube | Ch01, Ch02, Ch05 | [Ch14 - Design YouTube](chapters/14-youtube.md) |
| 15 | Design Google Drive | Ch01, Ch02, Ch05 | [Ch15 - Design Google Drive](chapters/15-google-drive.md) |
| 16 | The Learning Continues | All previous | [Ch16 - Next Steps](chapters/16-next-steps.md) |

---

## By Theme

### Foundations (Ch01–Ch03)

These three chapters are prerequisite reading for everything else. They establish the vocabulary of components, the quantitative tools for validating designs, and the interview process itself.

| Chapter | Focus |
|---------|-------|
| [Ch01 - Scale from Zero to Millions](chapters/01-scale.md) | Incremental scaling: load balancer, CDN, cache, replication, sharding, message queues |
| [Ch02 - Back-of-the-Envelope Estimation](chapters/02-estimation.md) | QPS, storage, bandwidth estimation; latency numbers; availability nines |
| [Ch03 - A Framework for System Design Interviews](chapters/03-framework.md) | 4-step interview process: requirements, high-level design, deep dive, wrap-up |

### Core Components (Ch04–Ch09)

Each chapter designs one canonical infrastructure component. Master these and you have the building blocks for every real system.

| Chapter | Component | Key Concepts |
|---------|-----------|-------------|
| [Ch04 - Design a Rate Limiter](chapters/04-rate-limiter.md) | Rate Limiter | Token bucket, leaking bucket, sliding window, Redis atomicity |
| [Ch05 - Design Consistent Hashing](chapters/05-consistent-hashing.md) | Consistent Hashing | Virtual nodes, minimal data movement during resharding |
| [Ch06 - Design a Key-Value Store](chapters/06-kv-store.md) | Key-Value Store | CAP theorem, Gossip protocol, Merkle trees, vector clocks |
| [Ch07 - Design a Unique ID Generator](chapters/07-unique-id-generator.md) | ID Generator | Snowflake, UUID, ticket server, clock synchronization |
| [Ch08 - Design a URL Shortener](chapters/08-url-shortener.md) | URL Shortener | Base62 encoding, hash collisions, redirect strategies |
| [Ch09 - Design a Web Crawler](chapters/09-web-crawler.md) | Web Crawler | BFS traversal, politeness, deduplication, distributed crawling |

### Real Systems (Ch10–Ch16)

Each chapter applies the foundation and core components to design a recognizable real-world system end-to-end.

| Chapter | System | Key Challenges |
|---------|--------|---------------|
| [Ch10 - Design a Notification System](chapters/10-notification-system.md) | Notification System | Push/SMS/email fanout, deduplication, failure handling |
| [Ch11 - Design a News Feed System](chapters/11-news-feed.md) | News Feed | Fanout on write vs. pull, cache for feeds, celebrity problem |
| [Ch12 - Design a Chat System](chapters/12-chat-system.md) | Chat System | WebSocket, online presence, group messaging, message ordering |
| [Ch13 - Design a Search Autocomplete System](chapters/13-search-autocomplete.md) | Search Autocomplete | Trie, prefix matching, caching, real-time suggestion updates |
| [Ch14 - Design YouTube](chapters/14-youtube.md) | Video Platform | Video transcoding pipeline, CDN, storage at PB scale |
| [Ch15 - Design Google Drive](chapters/15-google-drive.md) | File Sync | Block-level delta sync, conflict resolution, metadata service |
| [Ch16 - Next Steps](chapters/16-next-steps.md) | Career guidance | Further reading, real-world system references |

---

## Concept Index

Use this table to find every chapter where a concept is discussed or applied.

| Concept | Chapters |
|---------|---------|
| Cache | [Ch01](chapters/01-scale.md), [Ch04](chapters/04-rate-limiter.md), [Ch06](chapters/06-kv-store.md), [Ch08](chapters/08-url-shortener.md), [Ch11](chapters/11-news-feed.md), [Ch13](chapters/13-search-autocomplete.md), [Ch14](chapters/14-youtube.md) |
| Load Balancer | [Ch01](chapters/01-scale.md), [Ch04](chapters/04-rate-limiter.md), [Ch10](chapters/10-notification-system.md), [Ch12](chapters/12-chat-system.md) |
| CDN | [Ch01](chapters/01-scale.md), [Ch02](chapters/02-estimation.md), [Ch11](chapters/11-news-feed.md), [Ch14](chapters/14-youtube.md) |
| Database Replication | [Ch01](chapters/01-scale.md), [Ch06](chapters/06-kv-store.md), [Ch15](chapters/15-google-drive.md) |
| Consistent Hashing | [Ch01](chapters/01-scale.md), [Ch05](chapters/05-consistent-hashing.md), [Ch06](chapters/06-kv-store.md), [Ch08](chapters/08-url-shortener.md), [Ch13](chapters/13-search-autocomplete.md) |
| Message Queue | [Ch01](chapters/01-scale.md), [Ch10](chapters/10-notification-system.md), [Ch11](chapters/11-news-feed.md), [Ch14](chapters/14-youtube.md) |
| Rate Limiting | [Ch04](chapters/04-rate-limiter.md), [Ch09](chapters/09-web-crawler.md) |
| Sharding | [Ch01](chapters/01-scale.md), [Ch05](chapters/05-consistent-hashing.md), [Ch06](chapters/06-kv-store.md), [Ch12](chapters/12-chat-system.md) |
| WebSocket | [Ch12](chapters/12-chat-system.md) |
| Trie | [Ch13](chapters/13-search-autocomplete.md) |
| Back-of-Envelope Estimation | [Ch02](chapters/02-estimation.md), [Ch08](chapters/08-url-shortener.md), [Ch14](chapters/14-youtube.md), [Ch15](chapters/15-google-drive.md) |
| Stateless Design | [Ch01](chapters/01-scale.md), [Ch03](chapters/03-framework.md) |
| GeoDNS / Multi-DC | [Ch01](chapters/01-scale.md), [Ch14](chapters/14-youtube.md) |
| CAP Theorem | [Ch06](chapters/06-kv-store.md) |
| Snowflake ID | [Ch07](chapters/07-unique-id-generator.md) |
| Base62 Encoding | [Ch08](chapters/08-url-shortener.md) |
| Fanout | [Ch03](chapters/03-framework.md), [Ch11](chapters/11-news-feed.md) |
| Availability Nines | [Ch02](chapters/02-estimation.md) |
| Redis | [Ch01](chapters/01-scale.md), [Ch04](chapters/04-rate-limiter.md), [Ch11](chapters/11-news-feed.md), [Ch13](chapters/13-search-autocomplete.md) |
| Token Bucket | [Ch04](chapters/04-rate-limiter.md) |
