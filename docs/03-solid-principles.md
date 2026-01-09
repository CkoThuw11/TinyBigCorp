# SOLID Principles: The Foundation of Maintainable Code

**Time**: 60 minutes  
**Prerequisites**: [Guide 02 - Clean Architecture](./02-clean-architecture.md)  
**Goal**: Master the five principles that make code flexible and maintainable

---

## What is SOLID?

SOLID is an acronym for five principles:
- **S**ingle Responsibility Principle
- **O**pen/Closed Principle
- **L**iskov Substitution Principle
- **I**nterface Segregation Principle
- **D**ependency Inversion Principle

**These aren't just "best practices" - they solve specific problems.**

Let's learn each one by seeing the problem first, then the solution.

---

## S - Single Responsibility Principle

### The Problem

```python
class UserService:
    def create_user(self, email: str, name: str):
        # Validate email
        if '@' not in email:
            raise ValueError("Invalid email")
        
        # Save to database
        conn = psycopg2.connect(DATABASE_URL)
        conn.execute("INSERT INTO users VALUES (?, ?)", (email, name))
        conn.commit()
        
        # Send welcome email
        smtp = smtplib.SMTP('smtp.gmail.com')
        smtp.send(email, "Welcome!", f"Hello {name}")
        
        # Log to analytics
        requests.post('https://analytics.com/track', {
            'event': 'user_created',
            'email': email
        })
        
        # Generate PDF report
        pdf = FPDF()
        pdf.add_page()
        pdf.cell(200, 10, f"User: {name}")
        pdf.output(f"{email}.pdf")
```

**Stop and Think**: How many reasons could this class change?

<details>
<summary>Answer</summary>

This class has **FIVE** reasons to change:
1. Email validation rules change
2. Database schema changes
3. Email provider changes (Gmail → SendGrid)
4. Analytics service changes
5. PDF format changes

**Each change risks breaking the others.**

</details>

### The Solution

**One class = One responsibility**

```python
# 1. Validation responsibility
class EmailValidator:
    def validate(self, email: str) -> bool:
        return '@' in email and '.' in email

# 2. Database responsibility
class UserRepository:
    async def save(self, user: User) -> User:
        # Database logic only
        pass

# 3. Email responsibility
class EmailService:
    async def send_welcome(self, user: User) -> None:
        # Email logic only
        pass

# 4. Analytics responsibility
class AnalyticsService:
    async def track_user_created(self, user: User) -> None:
        # Analytics logic only
        pass

# 5. Orchestration responsibility
class UserService:
    def __init__(
        self,
        user_repo: UserRepository,
        email_service: EmailService,
        analytics: AnalyticsService
    ):
        self._user_repo = user_repo
        self._email_service = email_service
        self._analytics = analytics
    
    async def create_user(self, command: CreateUserCommand) -> UserDto:
        # Orchestrate the workflow
        user = await self._user_repo.save(User(...))
        await self._email_service.send_welcome(user)
        await self._analytics.track_user_created(user)
        return UserDto.model_validate(user)
```

**Now**: Each class has ONE reason to change.

### Exercise

Look at [`src/application/services/user_service.py`](file:///c:/Workspace/Training/TinyBigCorp/backend/src/application/services/user_service.py).

**Question**: Does it follow SRP? What is its single responsibility?

<details>
<summary>Answer</summary>

**Yes, it follows SRP.**

Its single responsibility: **Orchestrate user-related use cases**.

It doesn't:
- ❌ Validate email format (Pydantic does that)
- ❌ Execute SQL (Repository does that)
- ❌ Handle HTTP (Routes do that)

It only:
- ✅ Coordinates domain entities
- ✅ Enforces business rules
- ✅ Calls repositories

</details>

---

## O - Open/Closed Principle

### The Problem

```python
class ReportGenerator:
    def generate(self, data: dict, format: str) -> bytes:
        if format == "pdf":
            # PDF generation code
            return self._generate_pdf(data)
        elif format == "excel":
            # Excel generation code
            return self._generate_excel(data)
        elif format == "csv":
            # CSV generation code
            return self._generate_csv(data)
        # Adding new format requires modifying this class!
```

**Stop and Think**: What happens when you need to add JSON format?

<details>
<summary>Answer</summary>

You have to:
1. Modify this class (add another `elif`)
2. Risk breaking existing formats
3. Retest everything

**This violates OCP**: You're modifying existing code to add new functionality.

</details>

### The Solution

**Open for extension, closed for modification**

```python
# Abstract interface
class IReportGenerator(ABC):
    @abstractmethod
    def generate(self, data: dict) -> bytes:
        pass

# Concrete implementations
class PdfReportGenerator(IReportGenerator):
    def generate(self, data: dict) -> bytes:
        # PDF logic
        pass

class ExcelReportGenerator(IReportGenerator):
    def generate(self, data: dict) -> bytes:
        # Excel logic
        pass

# Adding new format doesn't modify existing code!
class JsonReportGenerator(IReportGenerator):
    def generate(self, data: dict) -> bytes:
        return json.dumps(data).encode()

# Usage
class ReportService:
    def __init__(self, generators: dict[str, IReportGenerator]):
        self._generators = generators
    
    def generate(self, data: dict, format: str) -> bytes:
        generator = self._generators.get(format)
        if not generator:
            raise ValueError(f"Unknown format: {format}")
        return generator.generate(data)
```

**Now**: Add new formats without touching existing code.

### Exercise

How does this project use OCP?

**Hint**: Look at [`src/domain/repositories/user_repository.py`](file:///c:/Workspace/Training/TinyBigCorp/backend/src/domain/repositories/user_repository.py) (interface) and [`src/infrastructure/repositories/user_repository.py`](file:///c:/Workspace/Training/TinyBigCorp/backend/src/infrastructure/repositories/user_repository.py) (implementation).

<details>
<summary>Answer</summary>

The `IUserRepository` interface is **closed for modification**.

But you can **extend** it with new implementations:
- `PostgresUserRepository` (current)
- `MongoUserRepository` (future)
- `InMemoryUserRepository` (testing)

**No changes to the interface or existing implementations needed.**

</details>

---

## D - Dependency Inversion Principle

**(Jumping to D because it's crucial for understanding the rest)**

### The Problem

```python
class UserService:
    def __init__(self):
        # Depends on concrete PostgreSQL implementation
        self._repo = PostgresUserRepository()
    
    async def create_user(self, user: User) -> User:
        return await self._repo.save(user)
```

**Stop and Think**: Can you test this without PostgreSQL?

<details>
<summary>Answer</summary>

**No!** Every test needs:
- PostgreSQL running
- Database migrations applied
- Test data setup
- Cleanup after tests

**Tests are slow, brittle, and hard to write.**

</details>

### The Solution

**Depend on abstractions, not concretions**

```python
class UserService:
    def __init__(self, user_repo: IUserRepository):  # Abstract!
        self._repo = user_repo
    
    async def create_user(self, user: User) -> User:
        return await self._repo.save(user)

# In production
service = UserService(PostgresUserRepository(session))

# In tests
service = UserService(InMemoryUserRepository())

# Future: Switch to MongoDB
service = UserService(MongoUserRepository(client))
```

**Now**: Service doesn't know or care about the database.

### The Dependency Rule

```
High-level modules should not depend on low-level modules.
Both should depend on abstractions.
```

**Translation**:
- Business logic (high-level) shouldn't depend on database (low-level)
- Both should depend on interfaces (abstractions)

### Exercise

Look at the constructor of [`UserService`](file:///c:/Workspace/Training/TinyBigCorp/backend/src/application/services/user_service.py).

**Question**: What does it depend on? Concrete or abstract?

<details>
<summary>Answer</summary>

```python
def __init__(self, user_repository: IUserRepository):
    self._user_repo = user_repository
```

It depends on `IUserRepository` (abstract interface), not `PostgresUserRepository` (concrete implementation).

**This is DIP in action.**

</details>

---

## L - Liskov Substitution Principle

### The Problem

```python
class Bird:
    def fly(self):
        print("Flying!")

class Sparrow(Bird):
    def fly(self):
        print("Sparrow flying!")  # ✅ Works

class Penguin(Bird):
    def fly(self):
        raise Exception("Penguins can't fly!")  # ❌ Breaks!

# Code expecting a Bird
def make_bird_fly(bird: Bird):
    bird.fly()  # Crashes if bird is a Penguin!

make_bird_fly(Sparrow())  # ✅ Works
make_bird_fly(Penguin())  # ❌ Crashes
```

**The problem**: Penguin violates the contract of Bird.

### The Solution

**Subtypes must be substitutable for their base types**

```python
class Bird(ABC):
    @abstractmethod
    def move(self):
        pass

class FlyingBird(Bird):
    def move(self):
        print("Flying!")

class Sparrow(FlyingBird):
    def move(self):
        print("Sparrow flying!")

class Penguin(Bird):
    def move(self):
        print("Swimming!")

# Now this works with ANY bird
def make_bird_move(bird: Bird):
    bird.move()  # Always works!

make_bird_move(Sparrow())  # ✅ Flying!
make_bird_move(Penguin())  # ✅ Swimming!
```

**Key**: The abstraction (`Bird.move()`) works for all subtypes.

### Exercise

Could you create a `CachedUserRepository` that implements `IUserRepository`?

```python
class CachedUserRepository(IUserRepository):
    def __init__(self, repo: IUserRepository, cache: Cache):
        self._repo = repo
        self._cache = cache
    
    async def get_by_id(self, user_id: int) -> Optional[User]:
        # Check cache first
        cached = self._cache.get(f"user:{user_id}")
        if cached:
            return cached
        
        # Fall back to repository
        user = await self._repo.get_by_id(user_id)
        if user:
            self._cache.set(f"user:{user_id}", user)
        return user
```

**Question**: Would `UserService` work with this? Why?

<details>
<summary>Answer</summary>

**Yes!** Because `CachedUserRepository` implements `IUserRepository`.

From `UserService`'s perspective:
- It calls `get_by_id()`
- It gets back a `User` or `None`
- It doesn't know or care about caching

**This is LSP**: You can substitute `CachedUserRepository` for any `IUserRepository`.

</details>

---

## I - Interface Segregation Principle

### The Problem

```python
class IWorker(ABC):
    @abstractmethod
    def work(self):
        pass
    
    @abstractmethod
    def eat(self):
        pass
    
    @abstractmethod
    def sleep(self):
        pass

class Human(IWorker):
    def work(self): print("Working")
    def eat(self): print("Eating")
    def sleep(self): print("Sleeping")

class Robot(IWorker):
    def work(self): print("Working")
    def eat(self): raise NotImplementedError("Robots don't eat!")
    def sleep(self): raise NotImplementedError("Robots don't sleep!")
```

**Problem**: Robot is forced to implement methods it doesn't need.

### The Solution

**Many specific interfaces > One general interface**

```python
class IWorkable(ABC):
    @abstractmethod
    def work(self):
        pass

class IFeedable(ABC):
    @abstractmethod
    def eat(self):
        pass

class ISleepable(ABC):
    @abstractmethod
    def sleep(self):
        pass

class Human(IWorkable, IFeedable, ISleepable):
    def work(self): print("Working")
    def eat(self): print("Eating")
    def sleep(self): print("Sleeping")

class Robot(IWorkable):  # Only implements what it needs!
    def work(self): print("Working")
```

**Now**: Each class implements only what it needs.

### Exercise

Look at [`IUserRepository`](file:///c:/Workspace/Training/TinyBigCorp/backend/src/domain/repositories/user_repository.py).

**Question**: Does it follow ISP? Are there methods a repository might not need?

<details>
<summary>Answer</summary>

Currently, `IUserRepository` has 7 methods:
- `get_by_id`, `get_by_email`, `get_by_username`
- `list_all`, `create`, `update`, `delete`

**For a read-only repository** (like for analytics), you might want:

```python
class IReadOnlyUserRepository(ABC):
    @abstractmethod
    async def get_by_id(self, user_id: int) -> Optional[User]:
        pass
    
    @abstractmethod
    async def list_all(self, skip: int, limit: int) -> List[User]:
        pass

class IUserRepository(IReadOnlyUserRepository):
    @abstractmethod
    async def create(self, user: User) -> User:
        pass
    
    @abstractmethod
    async def update(self, user: User) -> User:
        pass
```

**This follows ISP**: Read-only clients don't see write methods.

</details>

---

## Putting It All Together

Let's see how SOLID principles work together in this project:

```python
# S - Single Responsibility
# UserService: Orchestrate use cases (one job)
# UserRepository: Data access (one job)
# EmailService: Send emails (one job)

# O - Open/Closed
# IUserRepository: Interface (closed)
# PostgresUserRepository, MongoUserRepository: Extensions (open)

# L - Liskov Substitution
# Any IUserRepository can be used interchangeably
# UserService doesn't break with different implementations

# I - Interface Segregation
# IUserRepository: Only user-related methods
# Not a giant IRepository with methods for all entities

# D - Dependency Inversion
# UserService depends on IUserRepository (abstraction)
# Not on PostgresUserRepository (concretion)
```

---

## Final Exercise: Refactor This Code

Here's code that violates multiple SOLID principles:

```python
class OrderProcessor:
    def process_order(self, order_data: dict):
        # Validate
        if not order_data.get('customer_email'):
            raise ValueError("Email required")
        
        # Calculate total
        total = 0
        for item in order_data['items']:
            if item['type'] == 'book':
                total += item['price'] * 0.9  # 10% discount
            elif item['type'] == 'electronics':
                total += item['price'] * 0.95  # 5% discount
            else:
                total += item['price']
        
        # Save to database
        conn = psycopg2.connect(DATABASE_URL)
        conn.execute("INSERT INTO orders ...", ...)
        
        # Send email
        smtp.send(order_data['customer_email'], f"Total: ${total}")
        
        # Process payment
        if order_data['payment_method'] == 'credit_card':
            stripe.charge(total)
        elif order_data['payment_method'] == 'paypal':
            paypal.charge(total)
```

**Your Task**: Refactor this to follow SOLID principles.

**Hints**:
1. What are the different responsibilities?
2. What might change independently?
3. How would you test each part?
4. What abstractions are missing?

<details>
<summary>Solution</summary>

```python
# S - Separate responsibilities
class OrderValidator:
    def validate(self, order: Order) -> bool:
        return bool(order.customer_email)

class PricingService:
    def calculate_total(self, items: List[OrderItem]) -> float:
        return sum(self._calculate_item_price(item) for item in items)
    
    def _calculate_item_price(self, item: OrderItem) -> float:
        # Pricing logic
        pass

class IOrderRepository(ABC):
    @abstractmethod
    async def save(self, order: Order) -> Order:
        pass

class IEmailService(ABC):
    @abstractmethod
    async def send_order_confirmation(self, order: Order) -> None:
        pass

# O - Open for extension
class IPaymentProcessor(ABC):
    @abstractmethod
    async def charge(self, amount: float) -> PaymentResult:
        pass

class StripePaymentProcessor(IPaymentProcessor):
    async def charge(self, amount: float) -> PaymentResult:
        # Stripe logic
        pass

class PayPalPaymentProcessor(IPaymentProcessor):
    async def charge(self, amount: float) -> PaymentResult:
        # PayPal logic
        pass

# D - Depend on abstractions
class OrderService:
    def __init__(
        self,
        validator: OrderValidator,
        pricing: PricingService,
        repository: IOrderRepository,
        email: IEmailService,
        payment_processors: dict[str, IPaymentProcessor]
    ):
        self._validator = validator
        self._pricing = pricing
        self._repository = repository
        self._email = email
        self._payment_processors = payment_processors
    
    async def process_order(self, command: CreateOrderCommand) -> OrderDto:
        # Orchestrate
        if not self._validator.validate(order):
            raise ValidationError()
        
        total = self._pricing.calculate_total(order.items)
        order.total = total
        
        saved_order = await self._repository.save(order)
        
        processor = self._payment_processors[order.payment_method]
        await processor.charge(total)
        
        await self._email.send_order_confirmation(saved_order)
        
        return OrderDto.model_validate(saved_order)
```

**Now**:
- ✅ Each class has one responsibility (S)
- ✅ Can add new payment methods without modifying OrderService (O)
- ✅ Can swap implementations (L)
- ✅ Interfaces are focused (I)
- ✅ Depends on abstractions (D)

</details>

---

## Check Your Understanding

1. What is the Single Responsibility Principle? Give an example from this project.
2. How does Open/Closed Principle enable adding features without breaking existing code?
3. Why does Dependency Inversion make testing easier?
4. What's the difference between LSP and OCP?
5. When might you violate these principles? (Hint: Sometimes you should!)

---

## Next Steps

You now understand the principles that make Clean Architecture work.

**Next**: [Guide 04: OOP in Modern Development](./04-oop-in-modern-development.md)

Learn when to use classes vs functions, and why OOP still matters in 2026.
