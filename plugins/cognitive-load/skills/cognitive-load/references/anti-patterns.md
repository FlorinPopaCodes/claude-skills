# Cognitive Load Anti-Patterns

A comprehensive catalog of patterns that increase extraneous cognitive load.

---

## Code-Level Anti-Patterns

### Complex Conditionals

**What it looks like:**
```javascript
if (user.isActive && (user.role === 'admin' || user.permissions.includes('write')) &&
    !user.isBanned && (config.allowGuests || user.isAuthenticated)) {
  // ...
}
```

**Why it's harmful:** Forces tracking 5+ logical states simultaneously. Exceeds working memory (~4 chunks).

**Load level:** Extreme

---

### Nested If Statements

**What it looks like:**
```python
if is_valid:
    if has_permission:
        if not is_rate_limited:
            if connection.is_open:
                process_request()
```

**Why it's harmful:** Each nesting level adds preconditions to track. By level 4, you're holding 4 contexts plus the actual logic.

**Load level:** Extreme

---

### Deep Inheritance Hierarchies

**What it looks like:**
```
AdminController
    ↳ extends UserController
        ↳ extends GuestController
            ↳ extends BaseController
                ↳ extends AbstractController
```

**Why it's harmful:** Understanding any method requires jumping through 5 files. Method resolution is non-obvious.

**Load level:** Extreme

---

### Shallow Module Proliferation

**What it looks like:**
- 80+ classes with single methods
- Every utility has its own file
- `StringHelper`, `StringFormatter`, `StringValidator`, `StringSanitizer`...

**Why it's harmful:** You must remember ALL modules AND all their interactions. The navigation overhead exceeds the benefit of "small classes."

**Load level:** Extreme

---

### Magic Numbers and Status Codes

**What it looks like:**
```javascript
if (response.status === 401) { /* expired token */ }
if (response.status === 403) { /* no permission */ }
if (response.status === 418) { /* banned user - custom */ }
```

**Why it's harmful:** Requires mental mapping. Every reader must recreate the same mental model.

**Load level:** High

---

## Architecture-Level Anti-Patterns

### Distributed Monolith (Microservice Proliferation)

**What it looks like:**
- 17 microservices for 5 developers
- Every feature change touches 4+ services
- Debugging requires correlating logs across services

**Why it's harmful:** Network boundaries add failure modes without reducing coupling. Every request involves mental map of service topology.

**Load level:** Extreme

---

### Excessive Abstraction Layers

**What it looks like:**
```
Controller → Service → Repository → DataMapper → EntityManager → Connection
```

**Why it's harmful:**
- Each layer adds indirection
- Debugging traces explode exponentially
- Changes ripple through all layers
- Glue code proliferates

**False promise:** "Easy to swap databases" - this addresses ~10% of migration pain. Real pain is data model incompatibilities, protocols, distributed system semantics.

**Load level:** Extreme

---

### Framework-Coupled Business Logic

**What it looks like:**
```python
@app.route('/users')
@requires_auth
@validate_schema(UserSchema)
@cache(timeout=300)
def get_users():
    # Business logic buried in framework decorators
    return db.query(User).filter(User.active == True).all()
```

**Why it's harmful:** Understanding business logic requires understanding framework internals. New contributors need weeks of framework training.

**Load level:** High

---

### DRY Abuse (Premature Abstraction)

**What it looks like:**
```python
# "Common" utility extracted from 2 slightly similar use cases
def process_data(data, mode='default', transform=None, validate=True,
                 strict=False, fallback=None, retry_count=3):
    # 200 lines handling all variations
```

**Why it's harmful:**
- Tight coupling between unrelated components
- 10+ stack trace levels when debugging
- Changes affect unexpected consumers
- Interface more complex than duplicated code would be

**Load level:** High

---

### DDD Folder Structure Worship

**What it looks like:**
```
src/
  domain/
    aggregates/
    entities/
    value-objects/
    repositories/
    services/
  application/
    commands/
    queries/
    handlers/
  infrastructure/
    persistence/
    messaging/
```

**Why it's harmful:**
- Every team interprets DDD differently
- Structure becomes battleground for debate
- New developers must learn custom mental model
- DDD is about problem space, not folder organization

**Load level:** Medium-High

---

## Language/Feature Anti-Patterns

### Feature Overuse

**What it looks like:**
```cpp
// C++ - which initialization syntax? Why this one?
auto x = 5;
auto y{5};
auto z = {5};
int w(5);
int v = int{5};
```

**Why it's harmful:** Readers must recreate the decision rationale. "Why did they choose this approach from available features?"

**Load level:** Medium

---

### Clever Tricks

**What it looks like:**
```javascript
// Bitwise for floor
const floor = ~~value;

// Boolean coercion
const exists = !!obj;

// Comma operator abuse
const result = (validate(), transform(), compute());
```

**Why it's harmful:**
- Original author added incrementally (familiar to them)
- Newcomers encounter entire mess at once
- Each trick requires mental pause to decode

**Load level:** Medium-High

---

## Summary Table

| Anti-Pattern | Load Level | Working Memory Chunks Required |
|--------------|-----------|-------------------------------|
| Complex conditionals | Extreme | 5+ |
| Deep nesting | Extreme | 4+ (compounding) |
| Deep inheritance | Extreme | 5+ files |
| Shallow modules (80+) | Extreme | All modules + interactions |
| Distributed monolith | Extreme | Service topology + protocols |
| Excessive layers | Extreme | All layers + glue code |
| Framework coupling | High | Framework internals |
| DRY abuse | High | All abstraction consumers |
| Magic numbers | High | Mental mapping |
| Clever tricks | Medium-High | Language edge cases |
| Feature overuse | Medium | Decision rationale |
| DDD structure worship | Medium-High | Team-specific patterns |
