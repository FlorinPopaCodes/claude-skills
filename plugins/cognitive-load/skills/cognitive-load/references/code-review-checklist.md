# Cognitive Load Code Review Checklist

Quick reference for reviewing code through a cognitive load lens.

---

## Pre-Review Questions

Before diving into code, ask:

- [ ] **Could a newcomer understand this in <40 minutes?**
- [ ] **How many things must be held in memory simultaneously?**
- [ ] **Is the load intrinsic (problem complexity) or extraneous (presentation)?**

---

## Code-Level Checks

### Conditionals

- [ ] No more than 2-3 conditions per `if` statement
- [ ] Complex conditions extracted to named boolean variables
- [ ] Names describe intent, not mechanism (`isEligibleForDiscount` not `checkFlags`)

### Control Flow

- [ ] Maximum 2 levels of nesting (prefer 1)
- [ ] Early returns for guard clauses
- [ ] Happy path is the main flow, not buried in else branches
- [ ] No deeply nested ternaries

### Functions/Methods

- [ ] Function does one thing at appropriate abstraction level
- [ ] Parameters < 4 (consider object if more)
- [ ] No boolean flag parameters that switch behavior
- [ ] Return type is obvious from name

### Naming

- [ ] Names are self-describing (no mental mapping required)
- [ ] No abbreviations that require domain knowledge
- [ ] Consistent vocabulary (don't mix "get/fetch/retrieve")
- [ ] Plain language over jargon ("login" not "authentication flow")

### Magic Values

- [ ] No magic numbers without constants/enums
- [ ] Status codes are self-describing or well-documented
- [ ] Configuration values have meaningful names

---

## Structure-Level Checks

### Module Depth

- [ ] Interface is simpler than implementation
- [ ] No "MetricsProviderFactoryFactory" patterns
- [ ] Utility classes don't have single methods
- [ ] Related functionality is grouped, not fragmented

### Inheritance vs Composition

- [ ] Inheritance depth < 3 levels
- [ ] No "God" base classes with lots of shared state
- [ ] Composition preferred for behavior reuse
- [ ] Abstract classes have clear extension points

### Dependencies

- [ ] Direct code preferred over abstractions (when only one implementation)
- [ ] No premature "just in case" interfaces
- [ ] Dependencies are obvious, not hidden in base classes
- [ ] A little duplication is acceptable vs tight coupling

---

## Architecture-Level Checks

### Abstraction Layers

- [ ] Each layer provides real value (not just indirection)
- [ ] Changes don't ripple through all layers
- [ ] Debugging doesn't require 10+ stack frames
- [ ] Glue code isn't the majority of the codebase

### Framework Coupling

- [ ] Business logic is framework-agnostic
- [ ] Framework code wraps business logic (not vice versa)
- [ ] New contributors can understand business rules without framework expertise
- [ ] Tests don't require framework bootstrapping

### Service Boundaries

- [ ] Services are "deep" (simple interface, rich functionality)
- [ ] Feature changes don't require touching 4+ services
- [ ] Network boundaries exist for real scaling needs
- [ ] Monolith modules are well-isolated as alternative

---

## Red Flags

Immediate concerns that often indicate high cognitive load:

| Red Flag | Question to Ask |
|----------|-----------------|
| > 3 nesting levels | Can we use early returns? |
| > 80 small classes | Can we consolidate into deep modules? |
| Complex interfaces, simple implementations | Is this abstraction premature? |
| Framework decorators with business logic | Can we separate concerns? |
| "Utility" classes with one method each | Should this be inline? |
| Inheritance chain > 3 deep | Can we use composition? |
| Configuration larger than code | Is this over-engineered? |
| Tests harder than implementation | What are we really testing? |

---

## Green Flags

Signs of low cognitive load code:

- [ ] New developer can contribute within hours
- [ ] Debugging is straightforward (clear stack traces)
- [ ] Changes are localized (not rippling through layers)
- [ ] Reading order matches execution order
- [ ] Business rules are readable as almost-plain-English
- [ ] No "magic" requiring tribal knowledge
- [ ] The "boring" solution was chosen

---

## Quick Decision Matrix

| Situation | High Load Choice | Low Load Choice |
|-----------|-----------------|-----------------|
| Complex condition | Inline compound boolean | Named intermediate variables |
| Error handling | Nested if/else | Early return guards |
| Code reuse | Abstract base class | Composition/delegation |
| Flexibility | Interface + factory | Direct implementation |
| Status codes | HTTP numbers | Self-describing strings |
| Similar code | Shared abstraction | Acceptable duplication |
| Architecture | Layered (hexagonal) | Pragmatic (direct calls) |
