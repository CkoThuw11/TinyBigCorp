# Contributing to TinyBigCorp

Thank you for your interest in contributing! This guide will help you get started.

---

## 🚀 Quick Start

### 1. Clone the Repository

This project uses **Git submodules** for backend and frontend:

```bash
# Clone with submodules
git clone --recursive https://github.com/CkoThuw11/TinyBigCorp.git
cd TinyBigCorp

# Or if you already cloned without --recursive
git submodule update --init --recursive
```

### 2. Start Development Environment

```bash
# Start all services (database, backend, frontend)
docker-compose up
```

**Access**:
- Frontend: http://localhost:4200
- Backend API: http://localhost:8000
- API Docs: http://localhost:8000/docs

---

## 📁 Repository Structure

```
TinyBigCorp/
├── backend/          → Submodule (AdventureWork-ERP-backend)
├── frontend/         → Submodule (AdventureWork-ERP-frontend)
├── docs/             → Learning guides
├── docker-compose.yml → Orchestration
└── README.md         → Main documentation
```

**Important**: Backend and frontend are **separate Git repositories** included as submodules.

---

## 🔄 Development Workflow

### Working on Backend

```bash
cd backend

# Create feature branch
git checkout -b feature/add-products

# Make changes, following Clean Architecture
# ... edit files ...

# Run tests
pytest

# Run linting
ruff check .
black .
mypy src/

# Commit (in backend repo)
git add .
git commit -m "feat: add product entity"

# Push to backend repo
git push origin feature/add-products

# Create PR in backend repo
# → https://github.com/CkoThuw11/AdventureWork-ERP-backend
```

### Working on Frontend

```bash
cd frontend

# Create feature branch
git checkout -b feature/product-list

# Make changes, following MVVM pattern
# ... edit files ...

# Run tests
npm test

# Run linting
npm run lint

# Commit (in frontend repo)
git add .
git commit -m "feat: add product list component"

# Push to frontend repo
git push origin feature/product-list

# Create PR in frontend repo
# → https://github.com/CkoThuw11/AdventureWork-ERP-frontend
```

### Updating Main Repo

After backend/frontend PRs are merged:

```bash
cd TinyBigCorp

# Update submodules to latest
git submodule update --remote backend
git submodule update --remote frontend

# Commit the submodule updates
git add backend frontend
git commit -m "chore: update backend and frontend submodules"
git push
```

---

## 📝 Commit Message Convention

We use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code formatting (no logic change)
- `refactor`: Code restructuring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

**Examples**:
```
feat(user): add user deactivation endpoint
fix(auth): resolve token expiration issue
docs(readme): update installation instructions
refactor(user-service): extract validation logic
test(user-repo): add integration tests
```

---

## 🏗️ Architecture Guidelines

### Backend: Clean Architecture

Follow the four layers:

```
1. Domain Layer (src/domain/)
   - Pure business logic
   - No dependencies on outer layers
   - Entities, repository interfaces, exceptions

2. Application Layer (src/application/)
   - Use case orchestration
   - Services, DTOs, commands

3. Infrastructure Layer (src/infrastructure/)
   - Technical implementations
   - Database, external APIs, repositories

4. Interface Layer (src/api/)
   - External communication
   - HTTP routes, dependencies
```

**Rules**:
- ✅ Domain depends on NOTHING
- ✅ Application depends on Domain only
- ✅ Infrastructure implements Domain interfaces
- ✅ Interface depends on Application

### Frontend: MVVM + Signals

```
1. View (Template)
   - HTML only
   - No business logic

2. ViewModel (Component)
   - UI state with Signals
   - Event handlers
   - Calls services

3. Model (Service)
   - HTTP calls
   - Data transformation
```

**Rules**:
- ✅ Components don't make HTTP calls directly
- ✅ Use Signals for reactive state
- ✅ Services return Observables
- ✅ Use design system tokens (no hardcoded colors)

---

## ✅ Code Quality Standards

### Backend

**Before committing**:
```bash
# Run tests with coverage
pytest --cov=src --cov-report=term-missing

# Run linting
ruff check .
black .
mypy src/

# All must pass!
```

**Requirements**:
- ✅ Test coverage > 80%
- ✅ All linting passes
- ✅ Type hints on all functions
- ✅ Docstrings on public methods

### Frontend

**Before committing**:
```bash
# Run tests
npm test

# Run linting
npm run lint

# Build check
npm run build

# All must pass!
```

**Requirements**:
- ✅ Test coverage > 80%
- ✅ ESLint passes (no warnings)
- ✅ Prettier formatted
- ✅ No `any` types (use proper TypeScript)

---

## 🧪 Testing Requirements

### Backend Tests

**Unit Tests** (test business logic):
```python
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
```

**Integration Tests** (test with database):
```python
async def test_user_repository_create(db_session):
    repo = UserRepository(db_session)
    user = User(...)
    created = await repo.create(user)
    assert created.id is not None
```

### Frontend Tests

**Component Tests**:
```typescript
it('should create user', () => {
  const service = TestBed.inject(UserService);
  spyOn(service, 'createUser').and.returnValue(of(mockUser));
  
  component.createUser(mockCommand);
  
  expect(service.createUser).toHaveBeenCalled();
});
```

---

## 📋 Pull Request Process

### 1. Create PR

Use this template:

```markdown
## What Changed
Brief description of changes

## Why
Explain the motivation

## How to Test
1. Step 1
2. Step 2

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] Linting passes
- [ ] Follows architecture guidelines
- [ ] No breaking changes (or documented)
```

### 2. Code Review

Reviewers will check:
- ✅ **Architecture**: Follows Clean Architecture / MVVM
- ✅ **SOLID**: Adheres to SOLID principles
- ✅ **Tests**: Adequate test coverage
- ✅ **Documentation**: Code is clear, comments where needed
- ✅ **Type Safety**: Proper types (Pydantic / TypeScript)
- ✅ **Error Handling**: Proper exception handling
- ✅ **Security**: No vulnerabilities introduced

### 3. Merge

- PRs require **1 approval** minimum
- All CI checks must pass
- Squash and merge (keep history clean)

---

## 🗄️ Database Migrations

### Creating Migrations

```bash
# Create new migration file
# backend/migrations/V<number>__<description>.sql

# Example: V2__add_products.sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2) NOT NULL
);
```

**Rules**:
- ✅ Sequential version numbers
- ✅ Descriptive names
- ✅ Use transactions (BEGIN/COMMIT)
- ✅ Test locally first
- ❌ Never modify applied migrations

### Testing Migrations

```bash
# Clean restart
docker-compose down -v
docker-compose up

# Verify migrations applied
docker exec -it tinybigcorp-postgres psql -U postgres -d tinybigcorp -c "\dt"
```

---

## 🎨 Design System (Frontend)

### Use Design Tokens

**❌ Don't**:
```scss
.button {
  color: #007bff;  // Hardcoded!
  padding: 12px;   // Magic number!
}
```

**✅ Do**:
```scss
.button {
  color: var(--color-primary-500);
  padding: var(--spacing-3);
}
```

**Tokens defined in**:
- `src/app/theme/colors.ts`
- `src/app/theme/variables.scss`
- `src/app/theme/typography.scss`

---

## 🐛 Reporting Issues

### Bug Reports

Include:
1. **Description**: What happened
2. **Expected**: What should happen
3. **Steps to reproduce**: Exact steps
4. **Environment**: OS, browser, versions
5. **Screenshots**: If applicable

### Feature Requests

Include:
1. **Problem**: What problem does this solve
2. **Proposed solution**: How would it work
3. **Alternatives**: Other approaches considered
4. **Additional context**: Why this matters

---

## 📚 Learning Resources

New to the architecture patterns?

**Start here**: [`docs/README.md`](./docs/README.md)

Progressive guides covering:
- Clean Architecture
- SOLID Principles
- OOP vs Functional
- Data Contracts
- Database Migrations
- Testing Strategies

---

## 💬 Getting Help

- **Questions**: Open a GitHub Discussion
- **Bugs**: Open an Issue
- **Architecture questions**: Read `docs/` guides
- **Code review**: Tag `@anhhoangdev`

---

## 🙏 Thank You!

Every contribution helps make this a better learning resource.

**Remember**: The goal isn't just to add features - it's to demonstrate **how** to build maintainable software.

Happy coding! 🚀
