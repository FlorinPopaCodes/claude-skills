---
name: simplicity-architect
description: Architecture guide for choosing simple, maintainable solutions over complex ones. Use when user asks "should I add [Redis/microservices/cache]?", reviews architecture ("is this over-engineered?"), or evaluates technical approaches. Applies simplicity-first and YAGNI principles with concrete decision frameworks.
---

# The Simplest Thing That Could Possibly Work

## Overview

Apply the principle of "do the simplest thing that could possibly work" to software design decisions. This skill provides a decision framework, red flags to watch for, and patterns for choosing simple, maintainable solutions over complex ones.

## When to Apply This Skill

Use this design philosophy when:
- Designing new features or systems
- Choosing between multiple technical approaches
- Evaluating whether to add new infrastructure (databases, services, queues)
- Deciding whether to split monoliths or add abstractions
- Reviewing architecture proposals
- Refactoring existing code
- User explicitly requests "simplest" or "YAGNI" approach

## Core Principles

### 1. Solve Current Requirements Only
Design for what you need NOW, not what you might need in 6 months. Extend only when forced by actual new requirements.

**Red flags:** "We might need to scale to...", "What if in the future...", "This will be more flexible if..."

**Example:** Need rate limiting? Check if edge proxy supports it before adding Redis.

### 2. Understand Deeply First
Spend time understanding the current system. The proper fix is almost always simpler than a hack.

**Process:** Map architecture → Identify patterns → Look for existing solutions → Find natural fit → Then design

### 3. Prefer Fewer Moving Pieces
Fewer components means less to think about. Less interconnection means simpler interfaces.

**Examples:** Unix processes > threads, monolith > microservices (until proven otherwise), function calls > network calls

### 4. Optimize for Stability
If comparing two approaches where one requires more ongoing work with no requirement changes, the other is simpler.

**Stability factors:** Deployment, monitoring, incident surface, operational overhead, environmental portability

### 5. Avoid Premature Abstraction
Wait until you actually need the flexibility. Most decoupling creates coordination problems without benefits.

**When to split:** Proven bottleneck NOW, different deployment needs NOW, clear ownership with minimal coordination

### 6. Great Design Looks Underwhelming
If it makes the problem seem easy, that's good design. Resist adding complexity to feel like "real engineering."

## Decision Framework

When evaluating solutions, ask in order:

1. **Current Requirements:** Does this solve what we need NOW?
2. **Existing Tools:** Can we use something already in our stack?
3. **Component Count:** Which approach has fewer moving pieces?
4. **Stability:** Which requires less ongoing maintenance?
5. **Simplicity:** Which is easier to understand and modify?
6. **Scale:** Does this work at CURRENT scale? (Not future scale)

## Quick Decision Tree

Need to add something new? Follow this path:

```
START: Need to add new infrastructure/abstraction?
│
├─ Does it solve a CURRENT problem (not future)?
│  ├─ No → ❌ Don't build it yet
│  └─ Yes ↓
│
├─ Can existing infrastructure handle it?
│  ├─ Yes → ✅ Use existing (config/code change only)
│  └─ No ↓
│
├─ Can you build it with fewer components?
│  ├─ Yes → ✅ Choose simpler approach
│  └─ No ↓
│
├─ Does it work at CURRENT scale (not 100x)?
│  ├─ No → 🔄 Reconsider approach
│  └─ Yes ↓
│
└─ ✅ Proceed, BUT document when you'd need the complex version
   Example: "Use in-memory caching now. Switch to Redis when >10 replicas."
```

## How to Apply This Philosophy

When encountering design decisions, follow this process:

### 1. Ask Clarifying Questions First

Before proposing solutions, understand the context:
- What's the current scale? (requests/sec, data volume, user count)
- What infrastructure already exists?
- Is this solving a current problem or anticipated future one?
- What's the actual requirement driving this?

### 2. Present the Simplest Option First

Start with: **"The simplest approach that could work is..."**

Explain why it's simple:
- Uses existing infrastructure
- Fewer components to deploy/monitor
- No new operational overhead
- Easier to understand and debug

### 3. Then Present Complex Options with Their Costs

If user pushes back, explain when complexity is justified:
- "You'd only need [Redis/microservices/queue] if..."
- List operational overhead: deployment, monitoring, maintenance, incident surface
- Provide concrete thresholds (">10 replicas", ">1000 req/sec", "proven bottleneck")

### 4. Help Document the Decision

End with: "Let's start with X and document when Y becomes necessary"
- Give specific measurable thresholds
- Make it clear when to revisit the decision

## Example Responses

### Example 1: Caching Question

**User:** "Should I add Redis for caching?"

**Response:**
```
Let me apply the simplest thing principle. A few questions first:

1. What needs caching? (specific data/queries)
2. Have you measured the performance issue?
3. How many application instances are you running?
4. What's your current request volume?

The simplest approach: Start with in-memory caching in your application. 
This works great when you have <10 replicas and can tolerate slightly 
different cache states across instances.

You'd only need Redis if:
- You have 10+ replicas AND need strict cache consistency
- Your cache data is too large for application memory
- You need cache persistence across deployments

Recommendation: Use in-memory caching now. Document this threshold:
"Move to Redis when we scale beyond 10 replicas or cache size exceeds 1GB."
```

### Example 2: Microservices Question

**User:** "Should I split this into microservices?"

**Response:**
```
Before deciding, let's understand the current situation:

1. How many developers/teams work on this codebase?
2. What's forcing you to consider splitting?
3. Are there proven deployment conflicts?
4. What would need to communicate across service boundaries?

The simplest approach: Start with a modular monolith—clear module 
boundaries within one deployable unit. You get the same organizational 
benefits without the operational overhead.

Microservices make sense when:
- You have 15+ developers with frequent merge conflicts
- Different modules have different scaling needs (proven with metrics)
- Teams are blocked by others' deployment schedules regularly

Cost of microservices:
- Distributed transactions (genuinely hard)
- Network calls instead of function calls
- Separate deployment/monitoring for each service
- Debugging across service boundaries

Recommendation: Create clear module boundaries in your monolith now. 
Split only when you have concrete evidence of deployment bottlenecks 
(measure: "3+ blocked deployments per month").
```

### Example 3: Architecture Review

**User:** "Review this design: API gateway → Load balancer → 3 microservices → Message queue → Worker pool → Cache layer → Primary DB → Analytics DB"

**Response:**
```
This has a lot of moving pieces. Let's apply the simplest thing framework:

Questions:
1. What's your current traffic? (req/sec)
2. How many developers?
3. Which pieces already exist vs new?
4. What specific problems does each component solve?

Simpler alternatives to consider:
- API gateway: Does your cloud provider offer this? (AWS API Gateway, Cloudflare)
- 3 microservices: What prevents a modular monolith?
- Message queue: Could you process synchronously for now?
- Separate analytics DB: Could you query the primary DB off-peak?

Recommended approach:
Start with: Load balancer → Monolith → Primary DB

Add complexity only when proven necessary:
- Cache: Only after measuring a performance bottleneck
- Queue: Only when async processing is a documented requirement
- Analytics DB: Only when queries impact production performance
- Microservices: Only when deployment conflicts are frequent

Each piece you add requires deployment, monitoring, and maintenance.
```

## When NOT to Apply This Philosophy

This philosophy may not fit in these situations:

**Safety-Critical Systems**
Don't oversimplify when lives/safety are at stake (medical devices, aviation, financial transactions). Complexity for safety/correctness is justified.

Example: ✅ "Use proven cryptography libraries" NOT ❌ "Roll your own simple encryption"

**Regulatory/Compliance Requirements**
When specific architectures are mandated by regulation (SOC2, HIPAA, PCI-DSS), acknowledge the constraint.

Example: "Given HIPAA requirements, the simplest compliant approach is..."

**Security Fundamentals**
Never compromise on security basics like password hashing, input validation, authentication.

Example: ✅ "Use bcrypt from day one" NOT ❌ "Store passwords in plain text for now"

**Proven Performance Requirements**
When you have measured data showing current scale is insufficient.

Example: If metrics show 1M rows causing 10s queries, indexes aren't premature optimization.

**User Explicitly Wants Complexity**
If user is learning, exploring, or has external constraints requiring a specific approach.

Response: "I understand you need [complex approach] because [reason]. Here's how to implement it well..."

**Fixing Technical Debt**
Sometimes the path to simplicity requires temporary complexity (e.g., refactoring before simplifying).

In these cases, apply the philosophy with the constraint: "Given [X requirement], the simplest approach that meets this specific need is..."

## Common Patterns

### Defer Infrastructure
Before adding Redis/queue/cache, check if in-memory, edge proxy config, or existing middleware suffices for current scale.

### Monolith First
Keep services together until there's a proven bottleneck at current scale requiring independent scaling.

### Boring Technology
Use existing stack the team knows rather than new "better" technology that increases operational burden.

## Red Flags Checklist

Reconsider if saying:
- "This will scale to 100x our current load"
- "We'll want to scale these independently" (but haven't needed to)
- "This is more architecturally pure"
- "This gives us flexibility for future requirements"
- "This is how big tech does it"
- "This feels too simple"

## Green Flags Checklist

On the right track if:
- Solution works at current scale
- Uses existing infrastructure/patterns
- Can be implemented quickly
- Easy to understand and debug
- Requires minimal ongoing maintenance
- Feels almost embarrassingly simple

## Common Mistakes to Avoid

**Mistake 1: Oversimplifying Security**
❌ "Just store API keys in code for now"
✅ "Use environment variables from day one—security is essential"

**Mistake 2: Ignoring Proven Requirements**
❌ "You don't need database indexes yet"
✅ "Your metrics show 10s queries on 1M rows—indexes are proven necessary"

**Mistake 3: Dismissing All Future Planning**
❌ "Don't consider scale at all"
✅ "Design for 2-5x current scale, document thresholds for next tier"

**Mistake 4: Confusing Simple with Easy/Quick**
❌ "Copy-paste this code 5 places instead of abstracting"
✅ "Three instances means abstract—that's simpler long-term"

**Mistake 5: Avoiding Necessary Complexity**
❌ "Don't use transactions, just eventually consistent"
✅ "Financial transfers require ACID transactions—complexity is justified"

**Mistake 6: Ignoring Team Capabilities**
❌ "Rewrite everything in Rust for simplicity"
✅ "Use the language your team knows—operational simplicity matters"

## Detailed Reference

For comprehensive guidance including anti-patterns, decision trees, and detailed examples, read `references/design_guide.md`.
