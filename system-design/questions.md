# System Design — Questions

## Scalability

- How do you scale a system from 1k to 10M users?
- What is horizontal vs vertical scaling?
- How does load balancing work and what algorithms exist?
- What is sharding and how do you choose a shard key?
- How do you handle hotspots in a distributed system?

## Databases

- SQL vs NoSQL — when to choose each?
- What is database replication and what are its trade-offs?
- How do you handle eventual consistency?
- What is a write-ahead log (WAL)?
- How do you design a schema for high-write throughput?

## Caching

- Where do you put a cache (client, CDN, application, database)?
- What are cache eviction policies (LRU, LFU, TTL)?
- How do you handle cache invalidation?
- What is cache stampede and how do you prevent it?
- Redis vs Memcached — when to use each?

## Messaging & Queues

- When do you use a message queue vs direct API calls?
- What is the difference between a queue and a pub/sub topic?
- How do you guarantee at-least-once vs exactly-once delivery?
- How do you handle consumer lag?
- Kafka vs SQS — when to use each?

## Streaming

- How does event streaming differ from batch processing?
- How do you design a real-time feed (e.g., Twitter timeline)?
- How do you stream LLM responses to the client?

## Vector Databases

- What is a vector database and when do you need one?
- How does approximate nearest neighbor (ANN) search work?
- Pinecone vs Weaviate vs pgvector — trade-offs?

## Reliability

- What is the difference between availability and reliability?
- How do you design for fault tolerance?
- What are SLOs, SLAs, and SLIs?
- What is a circuit breaker pattern?
- How do you design a retry strategy?

## API Design

- REST vs GraphQL vs gRPC — when to use each?
- How do you version an API without breaking clients?
- How do you rate-limit an API?
- How do you design idempotent APIs?
