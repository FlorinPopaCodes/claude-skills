# Cognitive Load: Before & After Examples

Concrete transformations showing how to reduce extraneous cognitive load.

---

## 1. Complex Conditionals → Named Variables

### Before (High Load)
```javascript
function canAccessResource(user, resource, config) {
  if (user.isActive && !user.isBanned &&
      (user.role === 'admin' || resource.isPublic ||
       (user.permissions.includes(resource.requiredPermission) &&
        config.permissionCheckEnabled)) &&
      (!resource.requiresMFA || user.mfaVerified)) {
    return true;
  }
  return false;
}
```
**Working memory load:** 7+ conditions tracked simultaneously

### After (Low Load)
```javascript
function canAccessResource(user, resource, config) {
  const isActiveUser = user.isActive && !user.isBanned;
  const hasAdminAccess = user.role === 'admin';
  const isPublicResource = resource.isPublic;
  const hasRequiredPermission = config.permissionCheckEnabled &&
                                 user.permissions.includes(resource.requiredPermission);
  const mfaOk = !resource.requiresMFA || user.mfaVerified;

  const hasAccess = hasAdminAccess || isPublicResource || hasRequiredPermission;

  return isActiveUser && hasAccess && mfaOk;
}
```
**Working memory load:** 1-2 concepts at a time, building sequentially

---

## 2. Nested Conditionals → Early Returns

### Before (High Load)
```python
def process_order(order, user, inventory):
    if order is not None:
        if user.is_authenticated:
            if user.has_payment_method:
                if inventory.has_stock(order.items):
                    if not order.is_expired:
                        # Finally, the actual logic
                        result = execute_order(order, user)
                        return {"success": True, "result": result}
                    else:
                        return {"error": "Order expired"}
                else:
                    return {"error": "Out of stock"}
            else:
                return {"error": "No payment method"}
        else:
            return {"error": "Not authenticated"}
    else:
        return {"error": "Invalid order"}
```
**Mental overhead:** Track 5 nested contexts to reach core logic

### After (Low Load)
```python
def process_order(order, user, inventory):
    if order is None:
        return {"error": "Invalid order"}

    if not user.is_authenticated:
        return {"error": "Not authenticated"}

    if not user.has_payment_method:
        return {"error": "No payment method"}

    if not inventory.has_stock(order.items):
        return {"error": "Out of stock"}

    if order.is_expired:
        return {"error": "Order expired"}

    # Core logic is the clear focus
    result = execute_order(order, user)
    return {"success": True, "result": result}
```
**Mental overhead:** Linear flow, each guard clause is self-contained

---

## 3. Deep Inheritance → Composition

### Before (High Load)
```java
// To understand AdminUserController, must read 4 files
class AdminUserController extends UserController {
    // inherits from UserController...
}

class UserController extends AuthenticatedController {
    // inherits from AuthenticatedController...
}

class AuthenticatedController extends BaseController {
    // inherits from BaseController...
}

class BaseController {
    // Finally, the root behavior
}
```
**Navigation overhead:** 4 file jumps to understand method resolution

### After (Low Load)
```java
class AdminUserController {
    private final AuthenticationService auth;
    private final UserService users;
    private final AuditLogger audit;

    public AdminUserController(AuthenticationService auth,
                                UserService users,
                                AuditLogger audit) {
        this.auth = auth;
        this.users = users;
        this.audit = audit;
    }

    public Response handleRequest(Request req) {
        auth.requireAdmin(req);
        User user = users.getUser(req.getUserId());
        audit.log("admin_access", user);
        return Response.ok(user);
    }
}
```
**Navigation overhead:** All behavior visible in one file, dependencies explicit

---

## 4. Shallow Modules → Deep Modules

### Before (High Load)
```typescript
// 8 tiny files to track
// StringValidator.ts
export const isNotEmpty = (s: string) => s.length > 0;

// StringFormatter.ts
export const capitalize = (s: string) => s[0].toUpperCase() + s.slice(1);

// StringSanitizer.ts
export const removeWhitespace = (s: string) => s.trim();

// StringTransformer.ts
export const toLowerCase = (s: string) => s.toLowerCase();

// Usage requires importing from 4+ modules
import { isNotEmpty } from './StringValidator';
import { capitalize } from './StringFormatter';
import { removeWhitespace } from './StringSanitizer';
import { toLowerCase } from './StringTransformer';
```
**Navigation overhead:** Must know and import from many tiny modules

### After (Low Load)
```typescript
// strings.ts - one deep module with simple interface
export const Strings = {
  isNotEmpty: (s: string) => s.length > 0,
  capitalize: (s: string) => s[0].toUpperCase() + s.slice(1),
  trim: (s: string) => s.trim(),
  lower: (s: string) => s.toLowerCase(),

  // More complex composed operations
  normalize: (s: string) => Strings.trim(Strings.lower(s)),
  toTitle: (s: string) => Strings.trim(s).split(' ').map(Strings.capitalize).join(' '),
};

// Usage - one import, discoverable API
import { Strings } from './strings';
```
**Navigation overhead:** One module, complete functionality, discoverable

---

## 5. Magic Codes → Self-Describing Values

### Before (High Load)
```javascript
// Requires mental mapping: what does 401 mean here? 403? 418?
function handleAuthError(response) {
  if (response.status === 401) {
    refreshToken();
  } else if (response.status === 403) {
    showPermissionDenied();
  } else if (response.status === 418) {
    // Custom: user banned (need tribal knowledge)
    showBannedMessage();
  }
}
```

### After (Low Load)
```javascript
// Self-describing - no mental mapping needed
function handleAuthError(response) {
  switch (response.body.code) {
    case 'token_expired':
      refreshToken();
      break;
    case 'permission_denied':
      showPermissionDenied();
      break;
    case 'user_banned':
      showBannedMessage();
      break;
  }
}
```

---

## 6. Framework Coupling → Framework-Agnostic Core

### Before (High Load)
```python
# Business logic buried in framework decorators
@app.route('/orders/<order_id>/refund', methods=['POST'])
@requires_auth
@requires_permission('refund')
@validate_json_schema(RefundSchema)
@rate_limit(10, per='minute')
@audit_log('refund_attempt')
def process_refund(order_id):
    order = Order.query.get_or_404(order_id)
    if order.status != 'completed':
        abort(400, 'Cannot refund incomplete order')
    order.status = 'refunded'
    order.refunded_at = datetime.utcnow()
    db.session.commit()
    send_refund_notification.delay(order.user_id)
    return jsonify({'status': 'refunded'})
```
**Understanding requires:** Flask, SQLAlchemy, Celery, decorator semantics

### After (Low Load)
```python
# Pure business logic - no framework knowledge needed
class RefundService:
    def __init__(self, orders, notifications):
        self.orders = orders
        self.notifications = notifications

    def process_refund(self, order_id: str) -> RefundResult:
        order = self.orders.get(order_id)
        if order is None:
            return RefundResult.not_found()
        if order.status != OrderStatus.COMPLETED:
            return RefundResult.invalid_status(order.status)

        order.mark_refunded()
        self.orders.save(order)
        self.notifications.send_refund_notice(order.user_id)

        return RefundResult.success()

# Framework wiring is separate, thin layer
@app.route('/orders/<order_id>/refund', methods=['POST'])
@requires_auth
def refund_endpoint(order_id):
    result = refund_service.process_refund(order_id)
    return result.to_response()
```
**Understanding requires:** Just Python, business logic readable without framework knowledge

---

## 7. Premature Abstraction → Direct Code

### Before (High Load)
```java
// Interface with single implementation "for flexibility"
public interface UserRepository {
    User findById(Long id);
    void save(User user);
}

public class UserRepositoryImpl implements UserRepository {
    // Identical to what the interface promises
}

public interface UserService {
    User getUser(Long id);
}

public class UserServiceImpl implements UserService {
    private final UserRepository repository;

    public User getUser(Long id) {
        return repository.findById(id);  // Just delegates
    }
}

// Factory for good measure
public class UserServiceFactory {
    public static UserService create() {
        return new UserServiceImpl(new UserRepositoryImpl());
    }
}
```
**Navigation:** 5 files for simple user lookup

### After (Low Load)
```java
// Direct, no premature abstraction
public class Users {
    private final Database db;

    public Users(Database db) {
        this.db = db;
    }

    public User get(Long id) {
        return db.query("SELECT * FROM users WHERE id = ?", id)
                 .mapTo(User.class)
                 .findOne();
    }

    public void save(User user) {
        db.update("UPDATE users SET ... WHERE id = ?", user);
    }
}
```
**Navigation:** 1 file, all behavior visible

---

## Summary: Transformation Patterns

| High Load Pattern | Low Load Transformation |
|-------------------|------------------------|
| Compound boolean expression | Extract to named variables |
| Nested if/else | Early return guards |
| Deep inheritance | Composition with explicit dependencies |
| Many tiny modules | Consolidated deep modules |
| Magic numbers/codes | Self-describing values |
| Framework-entangled logic | Pure core + thin framework wrapper |
| Interface + single implementation | Direct class until extension needed |
