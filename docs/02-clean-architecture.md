# Clean Architecture: Building Software That Lasts

**Time**: 45 minutes  
**Prerequisites**: [Guide 01 - Why This Project Exists](./01-why-this-project.md)  
**Goal**: Understand how to organize code into layers and trace execution through them

---

## Let's Trace Real Code

We're going to follow what happens when a user is created, from HTTP request to database.

Open your project and let's walk through it together.

---

## Step 1: The HTTP Request Arrives

**File**: [`src/api/routes/users.py`](file:///c:/Workspace/Training/TinyBigCorp/backend/src/api/routes/users.py)

```python
@router.post("/", response_model=UserDto)
async def create_user(
    command: CreateUserCommand,
    user_service: Annotated[UserService, Depends(get_user_service)]
):
    try:
        return await user_service.create_user(command)
    except DomainValidationError as e:
        raise HTTPException(status_code=400, detail=e.message)
```

**Stop and Think**: What is this function's ONLY job?

<details>
<summary>Answer</summary>

This function does THREE things:
1. **Parse** the HTTP request into a `CreateUserCommand`
2. **Call** the service to do the work
3. **Handle** errors and convert them to HTTP responses

Notice what it does NOT do:
- ❌ No validation logic
- ❌ No business rules
- ❌ No database queries
- ❌ No email sending

**This is the Interface Layer** - it translates between HTTP and your application.

</details>

---

## Step 2: Validate the Input

**File**: [`src/application/dtos/user_dto.py`](file:///c:/Workspace/Training/TinyBigCorp/backend/src/application/dtos/user_dto.py)

```python
class CreateUserCommand(BaseModel):
    model_config = ConfigDict(strict=True)
    
    email: EmailStr = Field(..., description="User email address")
    username: str = Field(..., min_length=3, max_length=50)
    full_name: str = Field(..., min_length=1, max_length=100)
```

**Before the route function even runs**, Pydantic validates:
- Email is a valid email format
- Username is 3-50 characters
- Full name is 1-100 characters

**Stop and Think**: Why validate here instead of in the service?

<details>
<summary>Answer</summary>

**Validate at the boundary** (edge of your system).

Benefits:
- ✅ Invalid data never enters your system
- ✅ Business logic doesn't need to check format
- ✅ Same validation for HTTP, CLI, CSV import
- ✅ Auto-generated API documentation

This is called **"Parse, don't validate"** - once parsed, you KNOW it's valid.

</details>

---

## Step 3: Execute Business Logic

**File**: [`src/application/services/user_service.py`](file:///c:/Workspace/Training/TinyBigCorp/backend/src/application/services/user_service.py)

```python
class UserService:
    def __init__(self, user_repository: IUserRepository):
        self._user_repo = user_repository
    
    async def create_user(self, command: CreateUserCommand) -> UserDto:
        # 1. Check business constraints
        existing_user = await self._user_repo.get_by_email(command.email)
        if existing_user:
            raise EntityAlreadyExistsError("User", command.email)
        
        # 2. Create domain entity
        user = User(
            id=None,
            email=command.email,
            username=command.username,
            full_name=command.full_name,
            is_active=True,
            created_at=datetime.utcnow(),
            updated_at=datetime.utcnow()
        )
        
        # 3. Persist via repository
        created_user = await self._user_repo.create(user)
        
        # 4. Return DTO
        return UserDto.model_validate(created_user)
```

**This is the Application Layer** - orchestrating the use case.

**Stop and Think**: Why does this service take `IUserRepository` instead of `PostgresUserRepository`?

<details>
<summary>Answer</summary>

**Dependency Inversion Principle** (the "D" in SOLID).

```python
# BAD - Depends on concrete implementation
def __init__(self):
    self._user_repo = PostgresUserRepository()

# GOOD - Depends on abstraction
def __init__(self, user_repository: IUserRepository):
    self._user_repo = user_repository
```

Benefits:
- ✅ Can test with `InMemoryUserRepository` (no database needed)
- ✅ Can swap to `MongoUserRepository` without changing this code
- ✅ Service doesn't know or care about database details

</details>

---

## Step 4: Domain Entity

**File**: [`src/domain/entities/user.py`](file:///c:/Workspace/Training/TinyBigCorp/backend/src/domain/entities/user.py)

```python
@dataclass
class User:
    id: Optional[int]
    email: str
    username: str
    full_name: str
    is_active: bool
    created_at: datetime
    updated_at: datetime
    
    def deactivate(self) -> None:
        """Business rule: Users can be deactivated but not deleted."""
        self.is_active = False
    
    def activate(self) -> None:
        """Activate the user account."""
        self.is_active = True
```

**This is the Domain Layer** - pure business logic.

**Stop and Think**: What imports does this file have?

<details>
<summary>Answer</summary>

Look at the imports:
```python
from dataclasses import dataclass
from datetime import datetime
from typing import Optional
```

**Only Python standard library!**

No imports from:
- ❌ Infrastructure (no SQLAlchemy)
- ❌ Application (no DTOs)
- ❌ Interface (no FastAPI)

**This is the golden rule of Clean Architecture**: Domain has ZERO dependencies on outer layers.

Why? So business logic can be understood without knowing about databases, APIs, or frameworks.

</details>

---

## Step 5: Repository Interface

**File**: [`src/domain/repositories/user_repository.py`](file:///c:/Workspace/Training/TinyBigCorp/backend/src/domain/repositories/user_repository.py)

```python
class IUserRepository(ABC):
    @abstractmethod
    async def create(self, user: User) -> User:
        pass
    
    @abstractmethod
    async def get_by_email(self, email: str) -> Optional[User]:
        pass
```

**This is still Domain Layer** - defining WHAT operations we need, not HOW to do them.

**Stop and Think**: Why define an interface instead of just implementing the repository?

<details>
<summary>Answer</summary>

**Interface Segregation** + **Dependency Inversion**.

The interface defines the **contract**:
- "I need to be able to create users"
- "I need to be able to find users by email"

The implementation is in Infrastructure layer:
- "I'll use PostgreSQL to do that"
- "Or I'll use MongoDB"
- "Or I'll use in-memory for testing"

**The business logic doesn't care HOW**, only WHAT.

</details>

---

## Step 6: Repository Implementation

**File**: [`src/infrastructure/repositories/user_repository.py`](file:///c:/Workspace/Training/TinyBigCorp/backend/src/infrastructure/repositories/user_repository.py)

```python
class UserRepository(IUserRepository):
    def __init__(self, session: AsyncSession):
        self._session = session
    
    async def create(self, user: User) -> User:
        # Map domain entity to ORM model
        model = UserModel(
            email=user.email,
            username=user.username,
            full_name=user.full_name,
            is_active=user.is_active,
            created_at=user.created_at,
            updated_at=user.updated_at
        )
        
        self._session.add(model)
        await self._session.commit()
        await self._session.refresh(model)
        
        # Map ORM model back to domain entity
        return self._to_entity(model)
```

**This is the Infrastructure Layer** - implementing technical details.

**Stop and Think**: Why map between `User` (domain) and `UserModel` (ORM)?

<details>
<summary>Answer</summary>

**Separation of concerns**.

`UserModel` (SQLAlchemy):
- Has database-specific annotations
- Knows about tables, columns, relationships
- Tied to PostgreSQL

`User` (domain entity):
- Pure Python dataclass
- No database knowledge
- Can be used without any database

**Why bother?**
- ✅ Can test domain logic without database
- ✅ Can change database without changing domain
- ✅ Domain entities are simple and clear

**The trade-off**: More code (mapping layer).

</details>

---

## The Complete Flow

Let's trace the entire journey:

```
1. HTTP POST /api/v1/users
   ↓
2. FastAPI validates CreateUserCommand (Pydantic)
   ↓
3. Route calls UserService.create_user()
   ↓
4. UserService checks business rules
   ↓
5. UserService creates User entity (domain)
   ↓
6. UserService calls IUserRepository.create()
   ↓
7. PostgresUserRepository maps User → UserModel
   ↓
8. SQLAlchemy saves to PostgreSQL
   ↓
9. PostgresUserRepository maps UserModel → User
   ↓
10. UserService maps User → UserDto
    ↓
11. Route returns UserDto as JSON
```

**Exercise**: Open each file and trace this flow yourself. Put a print statement in each layer and watch the execution.

---

## The Four Layers Explained

### Layer 1: Interface (Outer)

**Location**: `src/api/`

**Purpose**: Translate between external world and your application

**Examples**:
- HTTP routes (FastAPI)
- CLI commands
- GraphQL resolvers

**Rules**:
- ✅ Parse requests
- ✅ Call services
- ✅ Format responses
- ❌ No business logic
- ❌ No database queries

---

### Layer 2: Application (Orchestration)

**Location**: `src/application/`

**Purpose**: Coordinate use cases and workflows

**Contains**:
- Services (orchestrate domain entities)
- DTOs (data transfer objects)
- Commands/Queries

**Rules**:
- ✅ Orchestrate domain entities
- ✅ Call repositories
- ✅ Handle transactions
- ❌ No HTTP knowledge
- ❌ No database knowledge

---

### Layer 3: Domain (Core)

**Location**: `src/domain/`

**Purpose**: Business logic and rules

**Contains**:
- Entities (business objects)
- Repository interfaces (contracts)
- Domain exceptions

**Rules**:
- ✅ Pure business logic
- ✅ No external dependencies
- ❌ No framework code
- ❌ No infrastructure code

**The Golden Rule**: Domain depends on NOTHING.

---

### Layer 4: Infrastructure (Technical)

**Location**: `src/infrastructure/`

**Purpose**: Implement technical details

**Contains**:
- Repository implementations
- Database models (ORM)
- External API clients

**Rules**:
- ✅ Implement interfaces from Domain
- ✅ Handle technical details
- ❌ No business logic

---

## Why This Structure?

### The "Swap Test"

**Question**: Can you swap PostgreSQL for MongoDB?

**Answer**: Yes! Change only:
1. `src/infrastructure/repositories/user_repository.py` (new implementation)
2. `src/api/dependencies.py` (inject new repository)

**Everything else stays the same**:
- ✅ Domain entities
- ✅ Business logic
- ✅ API routes
- ✅ DTOs

---

### The "Test Test"

**Question**: Can you test business logic without a database?

**Answer**: Yes!

```python
# In-memory repository for testing
class InMemoryUserRepository(IUserRepository):
    def __init__(self):
        self.users = []
    
    async def create(self, user: User) -> User:
        user.id = len(self.users) + 1
        self.users.append(user)
        return user

# Test the service
async def test_create_user():
    repo = InMemoryUserRepository()
    service = UserService(repo)
    
    command = CreateUserCommand(
        email="test@example.com",
        username="testuser",
        full_name="Test User"
    )
    
    result = await service.create_user(command)
    
    assert result.email == "test@example.com"
    assert len(repo.users) == 1
```

**No database, no HTTP, just pure logic testing.**

---

### The "Junior Test"

**Question**: Can a new developer understand the business?

**Answer**: Yes! Tell them to read `src/domain/` only.

They'll see:
- What a User is
- What operations are possible
- What business rules exist

**No SQL, no HTTP, no framework noise.**

---

## Common Questions

### "Isn't this over-engineered?"

For a 100-line script? Yes.  
For a 100,000-line application? No.

**The question isn't "Is this more code?"**  
**The question is "What does this buy us?"**

It buys us:
- Testability
- Flexibility
- Clarity
- Maintainability

---

### "When should I use this?"

**Use Clean Architecture when**:
- Project will last > 6 months
- Multiple developers
- Requirements will change
- Need comprehensive testing

**Skip it when**:
- Prototyping
- Proof of concept
- Personal scripts
- Throwaway code

---

## Hands-On Exercise

### Task: Trace User Deactivation

Follow the flow when a user is deactivated:

1. Find the HTTP route: `POST /users/{id}/deactivate`
2. Find the service method
3. Find the domain entity method
4. Find the repository method

**Questions**:
1. Which layer contains the business rule "users can be deactivated"?
2. Which layer knows about HTTP status codes?
3. Which layer talks to the database?
4. Could you test deactivation without a database?

<details>
<summary>Answers</summary>

1. **Domain layer** - `User.deactivate()` method
2. **Interface layer** - The route handles HTTP 404, 200, etc.
3. **Infrastructure layer** - `UserRepository.update()`
4. **Yes** - Use `InMemoryUserRepository` in tests

</details>

---

## Check Your Understanding

Before moving on, make sure you can answer:

1. What is the purpose of each of the four layers?
2. Why does Domain layer have no dependencies?
3. How does dependency injection enable testing?
4. What's the difference between `User` (entity) and `UserModel` (ORM)?
5. When would you skip Clean Architecture?

---

## Next Steps

You now understand HOW Clean Architecture works.

**Next**: [Guide 03: SOLID Principles](./03-solid-principles.md)

Learn the five principles that make this architecture maintainable.
