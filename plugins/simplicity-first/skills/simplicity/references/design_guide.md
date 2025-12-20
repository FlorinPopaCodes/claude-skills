# Simplicity Design Guide

Detailed patterns, anti-patterns, and technology-specific guidance for applying the simplicity-first philosophy.

## The Layers of Simplicity

When building any feature, there's a natural progression from simple to complex. **Start at the top and only move down when proven necessary.**

### Data Storage

```
Level 1: In-memory (process variables, module-level state)
    ↓ When: Data must survive restarts OR shared across requests
Level 2: File system (JSON files, SQLite)
    ↓ When: Concurrent writes OR queries need indexes OR scale demands
Level 3: Single database (PostgreSQL handles almost everything)
    ↓ When: Read/write patterns genuinely diverge OR proven performance limits
Level 4: Specialized databases (Redis, Elasticsearch, time-series DBs)
```

**Example - User Sessions:**
- Blog with 100 users: In-memory hash map with cookie tokens
- App with 10K users: PostgreSQL sessions table
- App with 1M+ concurrent: Only then consider Redis

### Async Processing

```
Level 1: Inline (just do it in the request)
    ↓ When: Operation takes >100ms AND user doesn't need result
Level 2: Background thread/process (spawn and forget)
    ↓ When: Must survive crashes OR retry failed jobs
Level 3: Job queue (Sidekiq, Celery with database backend)
    ↓ When: Cross-service communication OR complex routing
Level 4: Message broker (RabbitMQ, Kafka)
```

**Example - Sending Emails:**
- Low volume: Send inline, user waits 500ms
- Medium volume: Background thread, return immediately
- High volume with retry needs: Job queue
- Multi-service event streaming: Only then Kafka

### API Design

```
Level 1: Direct function calls
    ↓ When: Separate deployable OR different languages
Level 2: REST endpoints (CRUD operations map naturally)
    ↓ When: Complex nested queries OR real-time subscriptions
Level 3: GraphQL or specialized protocols
```

### Caching

```
Level 1: No cache (databases are faster than you think)
    ↓ When: Profiling shows specific query is bottleneck
Level 2: Memoization (in-process, request-scoped)
    ↓ When: Shared across requests OR processes
Level 3: Application cache (Rails.cache, in-memory store)
    ↓ When: Shared across servers
Level 4: Distributed cache (Redis, Memcached)
```

## Anti-Pattern Catalog

### Premature Abstraction

**What it looks like:**
```python
# Over-engineered: Factory pattern for one implementation
class NotificationFactory:
    def create(self, type: str) -> Notification:
        if type == "email":
            return EmailNotification()
        raise ValueError(f"Unknown type: {type}")

# Simple: Just use the thing
notification = EmailNotification()
```

**The cost:** Every reader must understand the abstraction layer before understanding the business logic.

**When abstraction IS warranted:** When you have 3+ implementations that genuinely vary, or a clear extension point documented in requirements.

### Configuration Addiction

**What it looks like:**
```yaml
# config/notifications.yml
notifications:
  email:
    enabled: true
    provider: sendgrid
    retry_count: 3
    retry_delay_ms: 1000
    batch_size: 100
    templates_path: /templates/email
    ...
```

When in practice, only `enabled` ever changes.

**Simple alternative:**
```python
EMAIL_ENABLED = os.getenv("EMAIL_ENABLED", "true") == "true"
# Everything else is hardcoded until proven otherwise
```

**Rule:** Only configure things that actually change between environments or deployments.

### Service Layer Ceremony

**What it looks like:**
```
Controller → Service → Repository → Model → Database
     ↑          ↑           ↑
   Tests      Tests       Tests
```

For a CRUD app with minimal business logic.

**Simple alternative:**
```
Controller → Model → Database
     ↑
   Tests
```

**When layers ARE warranted:** When business logic is complex enough that testing it requires isolation from HTTP/database concerns.

### Interface Proliferation

**What it looks like:**
```typescript
interface IUserRepository {
  findById(id: string): Promise<User>;
}

class UserRepository implements IUserRepository {
  // Only implementation
}

class UserService {
  constructor(private repo: IUserRepository) {}
}
```

**Simple alternative:**
```typescript
class UserService {
  async getUser(id: string): Promise<User> {
    return await User.findById(id);
  }
}
```

**When interfaces ARE warranted:** When you actually have multiple implementations (real database vs. test fake) or cross-module boundaries.

### Microservices for Small Teams

**What it looks like:**
- 3 developers, 12 services
- Every feature requires changes to 4 services
- Half the code is service-to-service communication
- Deploys require coordination across multiple repos

**Simple alternative:** Monolith with clear module boundaries.

**When microservices ARE warranted:**
- Teams large enough for ownership boundaries (15+ engineers)
- Genuinely different scaling characteristics
- Different deployment cadences with stable interfaces

## Technology Decision Framework

### Database Selection

**Default: PostgreSQL**

PostgreSQL handles:
- Relational data (obvious)
- JSON documents (JSONB)
- Full-text search (ts_vector)
- Time-series data (with TimescaleDB extension)
- Geospatial data (PostGIS)
- Key-value patterns (hstore)

**Only reach for alternatives when:**
- Redis: Proven need for sub-millisecond reads, distributed locks, or pub/sub
- Elasticsearch: Full-text search at scale that PostgreSQL can't handle
- MongoDB: Schema genuinely unknown and evolving rapidly (rare in practice)
- Cassandra/DynamoDB: Write volume exceeds single-node PostgreSQL (very rare)

### Framework Selection

**Default: Boring mainstream framework for your language**
- Ruby: Rails
- Python: Django or FastAPI
- JavaScript/TypeScript: Express or Next.js
- Go: Standard library + minimal router

**Avoid:**
- Newest framework with best benchmarks (maintenance risk)
- Framework that requires "unlearning" (cognitive overhead)
- Framework chosen for resume appeal

### Frontend Architecture

```
Level 1: Server-rendered HTML with minimal JS
    ↓ When: Significant interactive state management needed
Level 2: Islands architecture (Astro, partial hydration)
    ↓ When: Full SPA behavior genuinely needed
Level 3: SPA (React, Vue, etc.)
```

Most CRUD apps are fine at Level 1. Admin panels, dashboards, content sites—server-rendered HTML with Turbo/HTMX for interactivity.

## Code Review Checklist

When reviewing code (or your own implementation), check for:

### Unnecessary Abstraction
- [ ] Is there a factory/builder/strategy for single implementation?
- [ ] Are there interfaces without multiple implementations?
- [ ] Are there service classes that just pass through to another layer?
- [ ] Is there an event system for what could be a function call?

### Over-Configuration
- [ ] Are there config files with values that never change?
- [ ] Are there feature flags for things that won't be toggled?
- [ ] Is there environment-specific config for universal values?

### Premature Optimization
- [ ] Is there caching without profiling data?
- [ ] Are there database indexes for unqueried columns?
- [ ] Is there pagination for lists that will never exceed 100 items?
- [ ] Are there async jobs for operations that take <100ms?

### Future-Proofing
- [ ] Are there hooks/plugins for features not yet needed?
- [ ] Is there versioning for internal APIs with one client?
- [ ] Are there abstractions "in case we switch providers"?
- [ ] Is there scale engineering for 10x current load?

## Refactoring Toward Simplicity

### Collapsing Layers

**Before:**
```ruby
# controller
def show
  @user = UserService.find(params[:id])
end

# service
class UserService
  def self.find(id)
    UserRepository.find(id)
  end
end

# repository
class UserRepository
  def self.find(id)
    User.find(id)
  end
end
```

**After:**
```ruby
def show
  @user = User.find(params[:id])
end
```

### Inlining Abstractions

**Before:**
```python
def process_order(order):
    validator = OrderValidatorFactory.create("standard")
    validator.validate(order)
    processor = PaymentProcessorFactory.create(order.payment_type)
    processor.process(order)
    notifier = NotificationFactory.create("order_confirmation")
    notifier.send(order)
```

**After:**
```python
def process_order(order):
    validate_order(order)
    process_payment(order)
    send_order_confirmation(order)
```

### Hardcoding Configuration

**Before:**
```yaml
# config/settings.yml
pagination:
  default_per_page: 25
  max_per_page: 100
```

```ruby
per_page = Settings.pagination.default_per_page
```

**After:**
```ruby
PER_PAGE = 25  # Hardcoded until we actually need to change it
```

## Decision Log Template

When complexity seems necessary, document the decision:

```markdown
## Decision: [What we're adding]

### Problem
What specific, current problem are we solving?

### Simple Alternatives Considered
1. [Simplest option] - Why it doesn't work
2. [Next simplest] - Why it doesn't work

### Chosen Approach
[The approach we're taking]

### Complexity Cost
- Lines of code added: ~X
- New dependencies: [list]
- New concepts to understand: [list]
- Ongoing maintenance: [description]

### Trigger for Removal
When could we remove this complexity?
What would need to change?
```

## The Boring Technology Advantage

Boring technology provides:

1. **Battle-tested edge cases** - Millions of users have found the bugs
2. **Abundant documentation** - Stack Overflow has the answers
3. **Hiring pool** - Engineers know it or can learn it quickly
4. **Stable APIs** - Upgrades don't break everything
5. **Predictable performance** - Known characteristics at scale

**The hidden cost of novel technology:**
- Learning curve for the team (and future hires)
- Debugging alone (small community, fewer resources)
- Migration risk (project could be abandoned)
- Unknown failure modes at scale
- Integration challenges with existing stack

**Rule of thumb:** Limit yourself to one or two "innovation tokens" per project. Everything else should be boring.

---

*Remember: The goal is working software that's easy to understand and maintain. Cleverness is a cost, not a benefit. When in doubt, do less.*
