# The Simplest Thing Design Reference

## Core Philosophy

When designing software systems, do the simplest thing that could possibly work. This approach applies to fixing bugs, maintaining existing systems, and architecting new ones.

## Six Design Principles

### 1. Start with the Absolute Simplest Approach

**Principle:** Begin with what solves the current requirement. Extend only when forced by actual new requirements.

**Application:**
- Don't design for the "ideal" system with infinite scalability
- Avoid anticipating requirements six months out
- Use YAGNI (You Aren't Gonna Need It) as the ultimate design principle

**Example Decision Tree:**
```
Need rate limiting?
├─ Does edge proxy support it? → Use config file
├─ Single instance? → In-memory tracking
├─ Few replicas, can tolerate data loss on restart? → In-memory tracking
└─ Many replicas, must persist? → Add Redis
```

### 2. Deep Understanding Before Clever Solutions

**Principle:** Spend time understanding the current system thoroughly before designing. The "proper fix" requires understanding large sections of the codebase.

**Application:**
- First few solutions that come to mind are rarely the simplest
- Finding simplicity requires considering many approaches
- Understanding the system IS the engineering work

**Process:**
1. Map the current system architecture
2. Identify existing patterns and conventions
3. Look for similar problems already solved
4. Consider where new code fits naturally
5. Only then design the solution

**Warning:** Hacks are NOT simple—they add complexity by introducing "things you need to remember." The proper fix is almost always simpler than the hack.

### 3. Prefer Fewer Moving Pieces

**Principle:** Simple systems have fewer components to consider and less interconnection.

**Evaluation Criteria:**
- Fewer services/processes/threads
- Less shared state
- Clearer, more straightforward interfaces
- Less coupling between components

**Examples:**
- Unix processes > threads (no shared memory)
- Monolith > microservices (until you prove you need the split)
- Single database > distributed transactions
- Function calls > network calls

**When to Add Complexity:**
Only when you have concrete evidence that the simpler approach won't work for current (not future) requirements.

### 4. Optimize for Stability Over Scalability

**Principle:** Simple systems require less ongoing maintenance. If comparing two approaches where one requires more ongoing work with no requirement changes, the other is simpler.

**Stability Assessment:**
- Deployment complexity
- Monitoring requirements
- Incident surface area
- Operational overhead
- Environmental portability

**Example Analysis:**
```
In-memory rate limiting vs Redis:

In-memory:
+ No deployment
+ No monitoring
+ No incidents
+ Works in all environments
- Less strict guarantees across replicas

Redis:
+ Strict cross-replica guarantees
- Separate deployment
- Requires monitoring
- Can have incidents
- Needs setup in each environment

Decision: Start with in-memory unless strict guarantees are a documented requirement
```

### 5. Avoid Premature Abstraction

**Principle:** Wait until you actually need the flexibility. Most decoupling creates coordination problems without delivering benefits.

**When to Avoid Splitting:**
- "We might want to scale these independently" (but haven't needed to)
- "This will be more flexible" (but no concrete flexibility requirement)
- "This is better architecture" (but solves no current problem)

**Costs of Premature Decoupling:**
- Coordination over the wire
- Distributed transactions (genuinely hard)
- Feature implementation complexity
- Debugging across service boundaries

**When to Split:**
- Proven performance bottleneck at current scale
- Different deployment/scaling requirements NOW
- Clear ownership boundaries with minimal coordination

### 6. Accept That Great Design Looks Underwhelming

**Principle:** If it makes the problem seem easy, that's good design. Resist the temptation to add complexity to feel like "real engineering."

**Characteristics of Great Design:**
- "Oh, I didn't realize the problem was that easy"
- "You don't actually have to do anything difficult"
- Uses boring, proven approaches
- Leans on existing primitives/infrastructure
- Feels almost too simple

**Examples:**
- Unicorn web server: Just Unix sockets and forked processes
- Rails REST API: CRUD in the most boring way possible
- Edge proxy rate limiting: A few lines in a config file

## Detailed Analysis Examples

### Deep Dive: Rate Limiting Implementation Choices

**Context:** Application needs to prevent API abuse

**Option 1: Edge Proxy Configuration (Simplest)**
- Implementation: Add 5 lines to nginx/cloudflare config
- Pros: No code, no deployment, works immediately, proven reliable
- Cons: Less flexible, basic rate limiting only
- When to use: Standard rate limiting, no per-user customization needed
- Operational cost: Zero—it's already there

**Option 2: In-Memory Application Cache**
- Implementation: 50 lines of code, use existing cache library
- Pros: No new infrastructure, flexible logic, fast
- Cons: Per-instance limits (not cluster-wide), lost on restart
- When to use: <10 replicas, per-instance limiting acceptable
- Operational cost: Minimal—existing app deployment

**Option 3: Redis-Based Rate Limiting**
- Implementation: New Redis cluster, middleware, monitoring
- Pros: Cluster-wide limits, persistent, highly configurable
- Cons: New infrastructure to maintain, network latency, complexity
- When to use: >10 replicas AND need strict cluster-wide limits
- Operational cost: High—new service to deploy, monitor, maintain

**Decision Path:**
1. Check edge proxy first (5 min config change)
2. If need custom logic, use in-memory (1 day implementation)
3. Only add Redis when proven necessary (document threshold: ">10 replicas")

### Deep Dive: Monolith vs Microservices

**Scenario:** 20-developer team with growing codebase

**Option 1: Well-Structured Monolith (Start Here)**
- Implementation: Clear module boundaries, separate packages/namespaces
- Pros: Simple deployment, easy debugging, atomic transactions
- Cons: Single deployment pipeline (can become bottleneck)
- When to use: <50 developers, <10 teams, infrequent deploy conflicts
- Operational cost: Low—one deploy pipeline, one monitoring setup

**Option 2: Microservices**
- Implementation: Split into services, add service mesh, distributed tracing
- Pros: Independent deployment, tech stack flexibility, scaling
- Cons: Distributed transactions, network overhead, operational complexity
- When to use: >50 developers, proven deployment bottlenecks, clear ownership
- Operational cost: High—N services × (deploy + monitor + maintain)

**The Middle Path (Modular Monolith):**
```
Project structure:
/services
  /auth       (separate module, could become service later)
  /payments   (separate module, clear boundaries)
  /analytics  (separate module, minimal coupling)
  
All deployed together, but architecturally ready to split if needed.
```

**Thresholds to document:**
- Split when: 3+ blocked deployments per month (measured)
- Split what: Module with most deployment conflicts
- Don't split: Tightly coupled modules requiring distributed transactions

### Deep Dive: Caching Strategy Evolution

**Stage 1: No Cache (0-1K req/sec)**
- Just query the database
- Acceptable: <100ms response times
- Trigger for next stage: >100ms p95 latency

**Stage 2: HTTP Cache Headers (1K-10K req/sec)**
- Add `Cache-Control: max-age=300` headers
- CDN/browser caching
- Acceptable: Public content, 5min staleness okay
- Trigger: Need per-user caching or <5min staleness

**Stage 3: In-Memory Application Cache (10K-50K req/sec)**
- Use existing app cache (Rails.cache, Node cache)
- Acceptable: <10 replicas, per-instance okay
- Trigger: >10 replicas OR need shared cache

**Stage 4: Redis Cache (50K+ req/sec OR >10 replicas)**
- Dedicated cache layer
- Required when: Cluster-wide consistency needed
- Cost: New infrastructure

**Anti-pattern:** Jumping directly to Stage 4 "because we might need it"

## Anti-Patterns to Avoid

### Over-Engineering for Scale
**Symptom:** Designing for 100x-1000x current load
**Problem:** Can't predict bottlenecks accurately; makes codebase inflexible
**Fix:** Design for 2x-5x current load, monitor for issues

### Premature Optimization
**Symptom:** Adding caching/indexing before measuring
**Problem:** Complexity without proven benefit
**Fix:** Measure first, optimize only proven bottlenecks

### Abstraction Enthusiasm
**Symptom:** Creating frameworks for 2 use cases
**Problem:** YAGNI - abstraction adds complexity
**Fix:** Wait for 3+ concrete use cases before abstracting

### Distributed Transactions
**Symptom:** Splitting services requiring cross-service consistency
**Problem:** Distributed transactions are genuinely hard
**Fix:** Keep transactional logic in single service

## Working with This Philosophy

### In Code Review
- Challenge each new dependency/service: "Do we need this now?"
- Ask: "What's the simpler alternative?"
- Validate: "Does this work at current scale?"

### In Architecture Design
- Start with monolith on existing infrastructure
- Add complexity only for proven current needs
- Document actual requirements that drove each decision

### In Bug Fixes
- Understand the system deeply first
- The proper fix is usually simpler than the hack
- Hacks add "things to remember" = complexity

### In Feature Development
- Solve current requirement, nothing more
- Use existing patterns and infrastructure
- Extend only when new requirements force it
