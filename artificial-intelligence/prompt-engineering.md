# Prompt Engineering for Software Development

## What is Prompt Engineering?

> "Generative AI systems are designed to produce specific responses based on the quality of the prompts provided. Prompt engineering allows these models to better understand instructions and work with a wide range of demands, from simple commands to highly technical questions."
> — IBM

Prompt Engineering is the practice of crafting instructions for AI models to get better, more predictable, and higher-quality responses. Think of it as **programming in natural language** — the quality of your prompt directly determines the quality of the output.

### Why it matters in Software Development

- Develop and maintain code faster
- Automate repetitive tasks (boilerplate, tests, docs)
- Get quick solutions to complex problems
- Assist in complex development processes
- Generate documentation and design docs
- Conduct code reviews
- Brainstorm architecture and solutions

### Why it matters when building AI-powered software

When building systems that use AI (e.g., specialized agents, copilots, chatbots):

- Define behavior and scope of the AI
- Make the AI proactive (understand user intent)
- Get specialized, domain-specific responses
- Enforce data security and privacy boundaries
- Execute extremely specific tasks reliably
- Prompt Engineering is the **"new programming language"** for AI systems

---

## Prompt Techniques

---

### 1. Role Prompting

**Definition:** Explicitly define who the model is — a professor, senior engineer, critic — to control style, tone, and consistency.

**When to use:**
- You need formal or technical writing style
- You want a specific perspective (architect, reviewer, junior dev)
- You're building an AI agent with a consistent persona
- Teaching or explaining concepts adapted to a specific audience

**Limitations:**
- The role is not always followed faithfully
- Can be overridden by user instructions in longer chains
- Difficult to measure real impact on simple tasks
- Less contrast in more advanced models

**Software Dev Use Cases:**
- Code review as a senior engineer
- Architecture review as a system architect
- Explaining concepts as a mentor to a junior developer

**Templates:**

```
You are a senior software engineer with 10 years of experience in Go and distributed systems.
Your communication is direct, technical, and you always justify trade-offs.

Task: Review the following code and point out potential issues with performance and maintainability.

[paste code here]
```

```
You are a tech lead conducting an architecture review.
Focus on scalability, fault tolerance, and observability.

Review this system design: [describe system]
```

```
You are a patient engineering mentor explaining to a junior developer.
Use simple words, analogies, and short code examples.

Explain how database transactions work and when to use them.
```

---

### 2. Zero-shot

**Definition:** Send only the task instruction — no examples, no extra context. The model relies entirely on its base knowledge.

**When to use:**
- Quick lookups and simple explanations
- Simple transformations (translate, summarize, reformat)
- High-scale systems where token economy matters
- Tasks where the model has strong base knowledge

**Limitations:**
- Can fail on complex or ambiguous problems
- Little control over output format
- Can generate hallucinations
- Output varies with small changes to the prompt
- Weak on elaborate logical reasoning

**Software Dev Use Cases:**
- Generating a quick API skeleton
- Explaining a design pattern
- Translating or clarifying error messages
- Naming variables, functions, or services

**Templates:**

```
Summarize the following pull request description in 2 sentences:
[PR description]
```

```
Explain what the Repository pattern is in the context of a Go backend API.
Keep it under 100 words.
```

```
What does HTTP status code 422 mean in the context of a REST API?
```

```
Generate a skeleton for a REST API in Go for a user management service.
Include: create, read, update, delete endpoints.
```

---

### 3. One-shot / Few-shot

**Definition:** Provide one or more input/output examples so the model learns the exact pattern, format, or style you expect.

**When to use:**
- Output format needs to be precise and consistent
- Standardizing documentation, test cases, or commit messages
- When zero-shot gives inconsistent or misformatted results
- Teaching the model your team's specific conventions

**Limitations:**
- Quality depends heavily on example quality
- Does not cover all possible variations
- Increases token cost
- Can bias the response toward the example
- May over-imitate and not innovate

**Software Dev Use Cases:**
- Generating unit tests that follow your team's pattern
- Creating API docs in a specific format
- Writing commit messages in a defined style
- Generating changelog entries

**Templates:**

```
Generate unit tests following this pattern:

Example:
Function: calculateDiscount(price, percentage)
Test: should return 90 when price is 100 and percentage is 10

Now generate tests for:
Function: validateEmail(email string) bool
```

```
Write a commit message following this style:

Example:
feat(auth): add JWT refresh token rotation
fix(api): handle null pointer in user handler

Now write a commit message for:
Change: Added pagination support to the /products endpoint
```

```
Generate API documentation following this format:

Example:
POST /users
Description: Creates a new user account
Request: { "name": string, "email": string, "password": string }
Response: { "id": string, "name": string, "email": string }
Auth: None

Now document:
PUT /users/:id
```

---

### 4. Chain of Thought (CoT)

**Definition:** Ask the model to think step by step before answering. Forces explicit reasoning rather than jumping to conclusions.

**When to use:**
- Debugging complex issues where cause is unclear
- Architecture decisions with multiple trade-offs
- Security analysis or threat modeling
- Any task where reasoning quality matters more than speed

**Limitations:**
- Generates longer responses
- Costs more tokens
- Can overthink simple problems
- May produce unnecessary reasoning steps

**Software Dev Use Cases:**
- Root cause analysis of production bugs
- Choosing between architectural approaches
- Evaluating security vulnerabilities
- Reasoning through database schema decisions

**Templates:**

```
Think step by step.

We have a Node.js API that starts returning 502 errors after 2 hours of running.
The error only happens under high load. Memory usage is stable. CPU spikes to 100%.

What could be causing this? Walk through your reasoning before giving a conclusion.
```

```
Think step by step.

I need to choose between PostgreSQL and MongoDB for a system that:
- Has 10M users
- Stores user profiles (structured) and activity logs (semi-structured)
- Needs complex relational queries for reporting
- Has a team familiar with SQL

What should I choose and why? Reason through each requirement before concluding.
```

```
Think step by step.

Analyze potential security vulnerabilities in this authentication flow:
1. User submits email + password
2. Backend queries: SELECT * FROM users WHERE email = '[input]'
3. Password compared with bcrypt
4. JWT issued with 30-day expiry, no refresh token
```

---

### 5. CoT + Self-Consistency

**Definition:** Run multiple reasoning paths independently and pick the most consistent answer. Useful when accuracy is critical.

**When to use:**
- Validating important architecture decisions
- Confirming complex logic or calculations
- Decisions where being wrong is costly
- Reviewing interpretations of ambiguous requirements

**Limitations:**
- Much higher token consumption
- Can be slower
- Does not always converge to the correct answer
- Adds little value to simple tasks
- Can repeat redundant answers

**Software Dev Use Cases:**
- Validating system design hypotheses
- Confirming security assumptions
- Reviewing requirements with multiple interpretations

**Templates:**

```
I need to decide whether to use an event-driven architecture for our checkout service.

Reason through this 3 times from different angles:
1. From a scalability and performance perspective
2. From an operational complexity perspective
3. From a team skill and maintainability perspective

Then give a final recommendation based on the most consistent conclusion.
```

```
Evaluate whether this API design follows REST best practices.
Go through it 3 times, each time focusing on a different concern:
1. Resource naming and HTTP methods
2. Status codes and error handling
3. Versioning and backward compatibility

Then summarize your findings.

API: [paste API definition]
```

---

### 6. Tree of Thought (ToT)

**Definition:** The model explores multiple reasoning paths, compares them, and selects the best one. Like a decision tree applied to thinking.

**When to use:**
- Brainstorming solutions with significant trade-offs
- Evaluating multiple implementation approaches before committing
- Exploring hypotheses before making a final decision
- Problems where the right answer is not obvious

**Limitations:**
- Produces long and complex outputs
- Spends more tokens
- Difficult to guarantee branches are truly distinct
- Can become repetitive
- Requires clear instructions to work well

**Software Dev Use Cases:**
- Choosing a rate limiting strategy
- Evaluating caching strategies
- Selecting a messaging system (Kafka vs RabbitMQ vs SQS)
- Deciding on a deployment strategy

**Templates:**

```
I need to implement a rate limiter for our API.

Explore 3 different approaches:
1. Token bucket algorithm (in-memory)
2. Sliding window counter (Redis)
3. API Gateway-level rate limiting

For each approach:
- How it works
- Pros
- Cons
- Best fit scenario

Then recommend the best one for a microservices architecture with ~50 services and 10k req/s.
```

```
I need to add full-text search to our platform.

Explore these 3 options:
1. PostgreSQL full-text search
2. Elasticsearch
3. Typesense

For each: implementation effort, performance characteristics, operational overhead, and cost.
Then recommend the best option for a startup with a small engineering team.
```

---

### 7. Skeleton of Thought (SoT)

**Definition:** First generate a skeleton/outline, then expand each section with details. Gives you structure before diving into depth.

**When to use:**
- Writing ADRs, RFCs, and design documents
- Planning API endpoints before coding
- Structuring complex requirements documents
- Organizing articles or technical guides

**Limitations:**
- A bad skeleton leads to bad expansion
- Can lose context between skeleton and expansion steps
- More tokens consumed across two phases
- Requires discipline to not skip directly to details
- Output comes in two distinct phases

**Software Dev Use Cases:**
- Writing Architecture Decision Records (ADRs)
- Planning a new microservice from scratch
- Structuring a technical specification
- Creating a migration plan

**Templates:**

```
Phase 1 - Skeleton only (no details yet):
Generate the main sections for an ADR (Architecture Decision Record)
for migrating our monolith to microservices.
List section titles and a one-line description of each.

Phase 2 - Expand:
Now expand each section with technical details, trade-offs, and decision rationale.
```

```
Phase 1 - List the main endpoints needed for a REST API for an e-commerce platform.
Only endpoint names and HTTP methods, no details yet.

Phase 2 - Expand each endpoint with:
- Full path with path params
- Request body schema
- Response schema
- Required authentication
- Possible error codes
```

---

### 8. ReAct (Reasoning + Acting)

**Definition:** The model alternates between reasoning, taking an action, and observing the result — simulating an agent cycle.

**When to use:**
- Building agents that call external tools or APIs
- Automating multi-step debugging or investigative workflows
- Simulating troubleshooting scenarios
- When the task requires decision-making at each step based on new information

**Limitations:**
- Output is long and verbose
- Difficult to control when the agent should stop
- Can simulate actions that don't actually exist
- Can invent false observations
- More complex to use reliably in production

**Software Dev Use Cases:**
- Debugging agents with access to logs and metrics
- Automated incident response workflows
- Agents that interact with external APIs
- Code generation agents that run and verify their output

**Templates:**

```
You are a debugging agent. Use the following loop: Think → Action → Observation → Repeat.

Problem: Our /checkout endpoint returns 500 errors intermittently in production.

Available actions:
- check_logs(service: string, time_range: string)
- query_db(sql: string)
- check_metrics(service: string, metric: string)

Begin debugging. Show your full reasoning at each step before taking an action.
Stop when you have identified the root cause.
```

```
You are a deployment agent. Use the loop: Think → Action → Observation → Repeat.

Task: Deploy the new version of the payments service with zero downtime.

Available actions:
- check_health(service: string)
- scale_up(service: string, instances: int)
- deploy(service: string, version: string)
- rollback(service: string)
- check_error_rate(service: string)

Execute the deployment safely. Verify health at each step before proceeding.
```

---

### 9. Prompt Chaining

**Definition:** Break a complex task into sequential steps where each output feeds into the next step as input.

**When to use:**
- Complex generation tasks that are too large for a single prompt (schema → code → tests → docs)
- When a single prompt becomes too complex or loses focus
- Building pipelines or agents with clearly defined steps
- Tasks that require validation or transformation between stages

**Limitations:**
- Higher latency due to multiple sequential calls
- Errors from one step propagate to following steps
- Increases overall token cost
- Depends on correct parsing between steps
- Adds orchestration complexity

**Software Dev Use Cases:**
- Full feature generation pipeline: schema → repository → service → tests
- Log analysis pipeline: parsing → classification → response
- Document pipeline: draft → review → polish
- CI/CD automation: analyze change → generate tests → write changelog

**Templates:**

```
[Step 1 - Data Model]
Given this user story, generate a data model in Go structs with proper field tags.

User story: As a user, I want to create a product listing with title, description,
price, category, and inventory count.

---
[Step 2 - Repository Interface] (use Step 1 output as input)
Given this data model, generate the repository interface with full CRUD methods.

[output from step 1]

---
[Step 3 - Unit Tests] (use Step 2 output as input)
Given this repository interface, generate unit tests using testify/mock.

[output from step 2]
```

```
[Step 1 - Log Parsing]
Extract all ERROR and WARN level entries from this log file and format them as JSON.
[raw logs]

---
[Step 2 - Classification] (use Step 1 output)
Classify each log entry by error type: database, network, auth, validation, or unknown.
[parsed logs from step 1]

---
[Step 3 - Response] (use Step 2 output)
For each error type found, suggest the most likely root cause and a remediation step.
[classified logs from step 2]
```

---

## Quick Reference

| Technique | Best For | Token Cost | Complexity |
|---|---|---|---|
| Role Prompting | Style, persona, tone control | Low | Low |
| Zero-shot | Simple tasks, quick answers | Low | Low |
| Few-shot | Consistent format/style output | Medium | Low |
| Chain of Thought | Reasoning, debugging, decisions | Medium | Medium |
| CoT + Self-Consistency | High-stakes decisions, validation | High | Medium |
| Tree of Thought | Trade-off analysis, brainstorming | High | Medium |
| Skeleton of Thought | Documents, structured planning | Medium | Medium |
| ReAct | Agent loops, multi-step automation | High | High |
| Prompt Chaining | Complex pipelines, feature generation | High | High |

---

## Combining Techniques

Techniques are not mutually exclusive. Real-world prompts often combine several:

```
# Role + Chain of Thought + Few-shot example:

You are a senior backend engineer specializing in Go and PostgreSQL.

Think step by step.

Here is an example of how I want you to review SQL queries:

Example:
Query: SELECT * FROM users WHERE email = '[input]'
Issue: SQL injection vulnerability — user input directly interpolated
Fix: Use parameterized queries: SELECT * FROM users WHERE email = $1

Now review this query:
SELECT * FROM orders WHERE user_id = [userID] AND status = '[status]'
```
