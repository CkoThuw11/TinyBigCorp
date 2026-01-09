# OOP in Modern Development: When and Why

**Time**: 30 minutes  
**Prerequisites**: [Guide 03 - SOLID Principles](./03-solid-principles.md)  
**Goal**: Understand when to use classes vs functions

---

## The Debate

**Junior Developer**: "Why use classes? Functions are simpler!"

**Senior Developer**: "Sometimes. Let me show you when each approach shines."

---

## When Functions Are Better

### Scenario: Utility Operations

```python
# ✅ GOOD - Pure functions for utilities
def format_currency(amount: float) -> str:
    return f"${amount:,.2f}"

def calculate_tax(amount: float, rate: float) -> float:
    return amount * rate

def validate_email(email: str) -> bool:
    return '@' in email and '.' in email
```

**Why functions work here**:
- No state to manage
- Simple input → output
- Easy to test
- Easy to understand

**Don't use classes for this!**

---

## When Classes Are Better

### Scenario: Managing State

```python
# ❌ BAD - Functions with global state
_users = []

def add_user(user):
    _users.append(user)

def get_user(user_id):
    return next(u for u in _users if u.id == user_id)

# ✅ GOOD - Class encapsulates state
class UserRepository:
    def __init__(self):
        self._users = []
    
    def add(self, user: User) -> None:
        self._users.append(user)
    
    def get(self, user_id: int) -> Optional[User]:
        return next((u for u in self._users if u.id == user_id), None)
```

**Why classes work here**:
- State is encapsulated
- Multiple instances possible
- Clear ownership of data

---

## The Real Question

**Not**: "Should I use OOP?"  
**But**: "What problem am I solving?"

### Use Functions When:
- ✅ Stateless operations
- ✅ Pure transformations
- ✅ Utility helpers
- ✅ Simple scripts

### Use Classes When:
- ✅ Managing state
- ✅ Need polymorphism
- ✅ Grouping related behavior
- ✅ Building reusable components

---

## Exercise

Look at this project. Find examples of:
1. Pure functions (hint: check DTOs)
2. Classes with state (hint: repositories)
3. Classes for polymorphism (hint: interfaces)

---

## Check Your Understanding

1. When would you choose a function over a class?
2. What problems does encapsulation solve?
3. Is OOP always better? When is it worse?

---

**Next**: [Guide 05: Data Contracts](./05-data-contracts.md)
