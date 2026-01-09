# Testing Strategies

**Time**: 50 minutes  
**Prerequisites**: [Guide 07 - Development Workflow](./07-development-workflow.md)  
**Goal**: Test at the right level with the right tools

---

## The Testing Pyramid

```
        /\
       /E2E\      Few, slow, expensive
      /------\
     /  INT   \   Some, medium speed
    /----------\
   /   UNIT     \ Many, fast, cheap
  /--------------\
```

---

## Unit Tests: Test Business Logic

**What**: Test a single unit (class/function) in isolation

**Example**: Test `UserService` without database

```python
class InMemoryUserRepository(IUserRepository):
    def __init__(self):
        self.users = []
    
    async def create(self, user: User) -> User:
        user.id = len(self.users) + 1
        self.users.append(user)
        return user

async def test_create_user():
    # Arrange
    repo = InMemoryUserRepository()
    service = UserService(repo)
    command = CreateUserCommand(
        email="test@example.com",
        username="testuser",
        full_name="Test User"
    )
    
    # Act
    result = await service.create_user(command)
    
    # Assert
    assert result.email == "test@example.com"
    assert len(repo.users) == 1
```

**Benefits**:
- ✅ Fast (milliseconds)
- ✅ No external dependencies
- ✅ Easy to write
- ✅ Tests business logic

---

## Integration Tests: Test Infrastructure

**What**: Test with real external dependencies

**Example**: Test `UserRepository` with real database

```python
async def test_user_repository_create(db_session):
    # Arrange
    repo = UserRepository(db_session)
    user = User(
        id=None,
        email="test@example.com",
        username="testuser",
        full_name="Test User",
        is_active=True,
        created_at=datetime.utcnow(),
        updated_at=datetime.utcnow()
    )
    
    # Act
    created = await repo.create(user)
    
    # Assert
    assert created.id is not None
    
    # Verify in database
    result = await db_session.execute(
        select(UserModel).where(UserModel.id == created.id)
    )
    assert result.scalar_one() is not None
```

**Benefits**:
- ✅ Tests actual database interactions
- ✅ Catches SQL errors
- ✅ Verifies mappings

**Trade-offs**:
- ⚠️ Slower (seconds)
- ⚠️ Requires database setup
- ⚠️ More complex

---

## E2E Tests: Test Full Stack

**What**: Test complete user flows

**Example**: Test HTTP → Service → Database → HTTP

```python
async def test_create_user_endpoint(client):
    # Act
    response = await client.post(
        "/api/v1/users",
        json={
            "email": "test@example.com",
            "username": "testuser",
            "full_name": "Test User"
        }
    )
    
    # Assert
    assert response.status_code == 201
    data = response.json()
    assert data["email"] == "test@example.com"
    assert "id" in data
```

**Benefits**:
- ✅ Tests real user scenarios
- ✅ Catches integration issues

**Trade-offs**:
- ⚠️ Slowest (seconds to minutes)
- ⚠️ Brittle (many failure points)
- ⚠️ Hard to debug

---

## What to Test at Each Level

### Unit Tests (70%)
- Business logic
- Validation rules
- Domain entity methods
- Service orchestration

### Integration Tests (20%)
- Database queries
- Repository implementations
- External API calls

### E2E Tests (10%)
- Critical user flows
- Happy paths
- Key error scenarios

---

## Exercise

Write tests for the Product feature you added:

1. **Unit test**: `ProductService.create_product()`
2. **Integration test**: `ProductRepository.create()`
3. **E2E test**: `POST /api/v1/products`

---

## Check Your Understanding

1. Why test business logic without a database?
2. When do you need integration tests?
3. Why have fewer E2E tests than unit tests?

---

## Congratulations!

You've completed all 8 guides. You now understand:
- ✅ Why these patterns matter
- ✅ Clean Architecture layers
- ✅ SOLID principles
- ✅ OOP vs functional approaches
- ✅ Data contracts with Pydantic
- ✅ Database migrations
- ✅ Development workflows
- ✅ Testing strategies

**Next**: Apply these to your own projects!
