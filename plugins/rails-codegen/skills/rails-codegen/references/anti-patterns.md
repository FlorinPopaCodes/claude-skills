# Anti-Patterns & Checklist

## Common Anti-Patterns

### Architecture Anti-Patterns

| Anti-Pattern | Problem | Alternative |
|--------------|---------|-------------|
| Service objects | Unnecessary abstraction layer | Namespaced model classes or jobs |
| Result/Response objects | Over-engineering | Return values + exceptions |
| Context objects | Hidden dependencies | Explicit parameters |
| Interactors/Use cases | Framework overhead | Plain Ruby classes |
| Concerns for business logic | Hidden complexity | Inheritance or delegation |

### Code Organization Anti-Patterns

| Anti-Pattern | Problem | Alternative |
|--------------|---------|-------------|
| `app/services/` directory | Junk drawer | Namespaced model classes |
| `app/contexts/` directory | Unclear purpose | Form objects or jobs |
| `app/operations/` directory | Duplicate of jobs | Jobs |
| Passing hashes around | Untyped, error-prone | Return actual objects |
| God objects | Too many responsibilities | Extract to focused classes |

### Controller Anti-Patterns

| Anti-Pattern | Problem | Alternative |
|--------------|---------|-------------|
| Fat controllers | Business logic leak | Extract to models/jobs |
| Complex before_actions | Hidden flow | Explicit checks in actions |
| Concerns for business logic | Scattered code | Base controller inheritance |
| Rescue blocks everywhere | Inconsistent handling | ApplicationController rescue_from |
| Instance variables galore | Unclear dependencies | Explicit local variables |

### Model Anti-Patterns

| Anti-Pattern | Problem | Alternative |
|--------------|---------|-------------|
| Callback chains | Hard to follow | Explicit method calls |
| Fat models | Too many concerns | Extract to namespaced classes |
| Default scopes | Surprising behavior | Named scopes |
| Method missing magic | Hard to debug | Explicit methods |
| Validation callbacks | Side effects | Separate validation from logic |

### View Anti-Patterns

| Anti-Pattern | Problem | Alternative |
|--------------|---------|-------------|
| Complex logic in views | Hard to test | Query objects, presenters |
| N+1 queries in loops | Performance | Eager loading in controller |
| Helper method explosion | Unorganized | ViewComponent |
| Inline JavaScript | Unmaintainable | Stimulus controllers |
| Deeply nested partials | Hard to trace | Flat component structure |

### Database Anti-Patterns

| Anti-Pattern | Problem | Alternative |
|--------------|---------|-------------|
| Missing foreign keys | Data integrity | Always add FK constraints |
| Nullable booleans | Three-state logic | NOT NULL with default |
| No indexes on FKs | Slow joins | Index all foreign keys |
| Denormalized data | Stale data risk | Normalize, use counter cache |
| String for enums | No validation | PostgreSQL enum type |

## Detailed Examples

### Service Objects to Namespaced Classes

```ruby
# Bad - service object
# app/services/cloud_processor.rb
class CloudProcessor
  def initialize(cloud)
    @cloud = cloud
  end

  def call
    # processing logic
  end
end

# Good - namespaced model class
# app/models/cloud/processor.rb
class Cloud::Processor
  def initialize(cloud)
    @cloud = cloud
  end

  def call
    # processing logic
  end
end
```

### Result Objects to Simple Returns

```ruby
# Bad - result object
class ProcessCloud
  def call(cloud)
    result = process(cloud)
    Result.new(success: true, data: result)
  rescue => e
    Result.new(success: false, error: e.message)
  end
end

# Usage
result = ProcessCloud.new.call(cloud)
if result.success?
  use(result.data)
else
  handle_error(result.error)
end

# Good - simple return + exception
class Cloud::Processor
  def call
    process(cloud)  # Returns the processed cloud
  end
  # Let exceptions propagate naturally
end

# Usage
begin
  processed = Cloud::Processor.new(cloud).call
  use(processed)
rescue ProcessingError => e
  handle_error(e.message)
end
```

### Hash Passing to Object Returns

```ruby
# Bad - returning hash
def analyze_image(image)
  {
    width: image.width,
    height: image.height,
    format: image.format,
    colors: extract_colors(image)
  }
end

# Problems:
# - No type safety
# - Easy to misspell keys
# - No documentation

# Good - return object
ImageAnalysis = Data.define(:width, :height, :format, :colors)

def analyze_image(image)
  ImageAnalysis.new(
    width: image.width,
    height: image.height,
    format: image.format,
    colors: extract_colors(image)
  )
end

# Benefits:
# - Type safe
# - Documented structure
# - IDE autocomplete
```

### Concerns for Logic to Explicit Inheritance

```ruby
# Bad - business logic in concern
module Processable
  extend ActiveSupport::Concern

  included do
    after_create :schedule_processing
  end

  def process!
    # 50 lines of processing
  end
end

class Cloud < ApplicationRecord
  include Processable  # Hidden behavior
end

# Good - explicit inheritance or composition
class Cloud < ApplicationRecord
  after_create :schedule_processing

  def process!
    Cloud::Processor.new(self).call
  end
end
```

### Complex View Logic to Query Objects

```ruby
# Bad - logic in view
<% @participants.each do |p| %>
  <% if p.clouds.any? && p.clouds.where(state: :generated).count > 0 %>
    <% active_clouds = p.clouds.where(state: :generated).order(:created_at) %>
    <%= render active_clouds.first %>
  <% end %>
<% end %>

# Good - query object + eager loading
# Controller:
@participants = Participant::WithActiveCloudsQuery.new.call

# Query:
class Participant::WithActiveCloudsQuery
  def call
    Participant
      .joins(:clouds)
      .where(clouds: { state: :generated })
      .includes(:clouds)
      .distinct
  end
end

# View:
<% @participants.each do |p| %>
  <%= render p.clouds.first %>
<% end %>
```

## Code Style Guidelines

### Naming

```ruby
# Models: Singular nouns, domain language
class Cloud < ApplicationRecord      # Not: Image, Upload
class Participant < ApplicationRecord # Not: User

# Controllers: Plural, resource-focused
class CloudsController              # Standard
class Participant::CloudsController # Namespaced

# Jobs: Verb + noun + Job
class ProcessCloudJob
class SendWelcomeEmailJob

# Namespaced classes: Model::Operation
class Cloud::Analyzer
class Cloud::CardGenerator
class Participant::Authenticator
```

### Method Length

- Controller actions: 5-10 lines max
- Model methods: 15 lines max (extract if longer)
- Private methods: Keep focused, single responsibility

### Guard Clauses

```ruby
# Good - early returns
def process
  return if already_processed?
  return unless valid_for_processing?

  do_processing
end

# Bad - nested conditionals
def process
  if !already_processed?
    if valid_for_processing?
      do_processing
    end
  end
end
```

## Deployment Checklist

Before deploying Rails code, verify:

### Naming & Domain
- [ ] Models named after business domain concepts?
- [ ] Using domain language consistently?
- [ ] No generic names (User, Image, Post) when domain term exists?

### Models
- [ ] Model code follows organization order?
- [ ] States implemented as enums (not state machine gems)?
- [ ] Counter caches on has_many associations?
- [ ] Normalizations for data cleaning?
- [ ] Complex logic extracted to namespaced classes?

### Controllers
- [ ] Actions under 10 lines?
- [ ] No business logic in controllers?
- [ ] Using guard clauses for early returns?
- [ ] Proper namespacing for auth contexts?

### Database
- [ ] Foreign key constraints on all references?
- [ ] NOT NULL with defaults (not nullable)?
- [ ] Indexes on foreign keys and query columns?
- [ ] PostgreSQL enums for fixed value sets?
- [ ] Check constraints for data integrity?

### Jobs
- [ ] Jobs orchestrate, not execute?
- [ ] Complex logic in namespaced model classes?
- [ ] Idempotent (safe to run multiple times)?
- [ ] Proper error handling and reporting?

### Configuration
- [ ] Using Anyway Config (not Rails credentials)?
- [ ] No direct ENV access?
- [ ] Required vars documented?
- [ ] Defaults for optional settings?

### Testing
- [ ] Model specs for business logic?
- [ ] Request specs for endpoints?
- [ ] Using FactoryBot with traits?
- [ ] Not testing framework behavior?

### Architecture
- [ ] No `app/services/` directory?
- [ ] No result/context objects?
- [ ] Forms for multi-model operations?
- [ ] Query objects for complex queries?
- [ ] ViewComponent for reusable UI?

### Frontend
- [ ] Using Hotwire (Turbo + Stimulus)?
- [ ] No inline JavaScript?
- [ ] Simple view logic only?
- [ ] Proper eager loading (no N+1)?

## Quick Reference Card

```
ALWAYS                          NEVER
────────────────────────────────────────────────────
Counter cache on has_many       app/services/ directory
Foreign key constraints         Result/Context objects
Enums for state                 Devise/CanCanCan
Anyway Config                   Rails credentials
Namespaced model classes        Fat controllers
Guard clauses                   Nested conditionals
ActionPolicy                    State machine gems
RSpec + FactoryBot              Testing framework behavior
Hotwire + ViewComponent         Inline JavaScript
PostgreSQL features             Denormalized data
```
