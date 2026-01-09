# TinyBigCorp: A Learning Journey

> **By anhhoangdev** - Teaching software architecture through intentional design

---

## 💭 The Story Behind This Project

I've seen too many developers struggle with the same problem: they can build features quickly, but their codebases become unmaintainable nightmares within months.

**This project exists to change that.**

It's not about building the next startup. It's about teaching the **principles** that separate junior developers from senior engineers - the patterns used at Google, Amazon, and Netflix to build software that lasts.

---

## 🎯 What I Want You to Learn

### 1. **Architecture Isn't About Being Clever**

It's about solving real problems:
- How do you change databases without rewriting everything?
- How do you test business logic without external dependencies?
- How do you onboard new developers in days, not months?

**Clean Architecture** answers these questions.

### 2. **Patterns Have Purpose**

Every pattern in this project solves a specific pain point:
- **SOLID principles** → Make code flexible and maintainable
- **Dependency Injection** → Enable testing and swapping implementations
- **Repository Pattern** → Decouple business logic from databases
- **Data Contracts** → Catch errors before production

**I don't use patterns because they're "best practices."**  
**I use them because they solve problems I've faced in production.**

### 3. **Trade-offs Are Real**

This project is **intentionally over-engineered** for a simple CRUD app.

Why? Because I want you to see the **full pattern** in a safe environment where you can:
- Break things without consequences
- Experiment with violations
- Understand the "why" behind each decision

**In real projects, you'll make different trade-offs.** That's fine. The goal is to make **informed** decisions, not follow dogma.

---

## 🚀 How to Use This Project

### For Learners

1. **Start with the guides**: [`docs/README.md`](./docs/README.md)
2. **Follow the learning path**: 8 progressive guides (6 hours total)
3. **Do the exercises**: Reading isn't enough - you need to write code
4. **Break things**: Violate the patterns and see what breaks

### For Teachers

1. **Use as curriculum**: Each guide is 15-60 minutes of focused learning
2. **Assign exercises**: Hands-on tasks with solutions provided
3. **Review code**: Use SOLID principles in code reviews
4. **Discuss trade-offs**: When to use these patterns vs when to skip them

### For Teams

1. **Onboarding material**: New hires learn your architecture patterns
2. **Reference implementation**: "This is how we structure code"
3. **Discussion starter**: "Should we use this pattern here?"

---

## 🎓 The Learning Path

**→ [Start Here: docs/README.md](./docs/README.md) ←**

Progressive guides covering:

1. **Why This Project Exists** - The journey from junior to senior (story-based)
2. **Clean Architecture** - Trace code through all 4 layers (hands-on)
3. **SOLID Principles** - Each principle with exercises (problem→solution)
4. **OOP in Modern Development** - When to use classes vs functions (honest discussion)
5. **Data Contracts** - Pydantic and type safety (practical examples)
6. **Database Migrations** - Team collaboration without conflicts (real scenarios)
7. **Development Workflow** - Add a feature end-to-end (guided tutorial)
8. **Testing Strategies** - Unit/Integration/E2E pyramid (with examples)

Each guide:
- ✅ Builds on previous concepts
- ✅ Includes "Stop and Think" moments
- ✅ Has hands-on exercises
- ✅ Uses real code from this project
- ✅ Discusses trade-offs honestly

---

## 🛠️ Quick Start

### Clone with Submodules

This project uses **Git submodules** for backend and frontend:

```bash
# Clone with submodules
git clone --recursive https://github.com/CkoThuw11/TinyBigCorp.git
cd TinyBigCorp

# Or if you already cloned without --recursive
git submodule update --init --recursive
```

**Repository Structure**:
- **Main repo**: Documentation, Docker orchestration, learning guides
- **Backend submodule**: [AdventureWork-ERP-backend](https://github.com/CkoThuw11/AdventureWork-ERP-backend)
- **Frontend submodule**: [AdventureWork-ERP-frontend](https://github.com/CkoThuw11/AdventureWork-ERP-frontend)

---

### Run Everything

**One command to run everything:**

```bash
docker-compose up
```

This automatically:
- ✅ Starts PostgreSQL database
- ✅ Runs Flyway migrations
- ✅ Starts backend API with hot-reload (http://localhost:8000)
- ✅ Starts frontend with hot-reload (http://localhost:4200)

**Edit any file** → It auto-reloads. No manual setup needed.

---

## 💡 My Philosophy

### Maintainability > Speed

I'd rather spend 2 hours designing a feature that's easy to change than 30 minutes building something that becomes technical debt.

### Explicitness > "Magic"

I prefer verbose, clear code over clever abstractions. Future developers (including me) will thank me.

### Interfaces > Implementation

I design contracts first, implementations second. This forces me to think about "what" before "how."

### Teaching > Documenting

I don't just show you code - I explain **why** it's structured this way and **when** you should use it differently.

---

## 🎯 Success Metrics

**You've learned what I wanted to teach if you can**:

1. **Explain the "why"** - Not just "this is how we do it" but "this solves X problem"
2. **Make trade-offs** - Know when to use these patterns and when to skip them
3. **Apply to new contexts** - Take these principles to your own projects
4. **Teach others** - Explain Clean Architecture to a junior developer

---

## 🤝 Contributing

Want to contribute? Awesome! Here's how:

### Quick Start

1. **Clone with submodules**: `git clone --recursive https://github.com/CkoThuw11/TinyBigCorp.git`
2. **Read the guide**: [CONTRIBUTING.md](./CONTRIBUTING.md)
3. **Pick a repo**: Work on [backend](https://github.com/CkoThuw11/AdventureWork-ERP-backend) or [frontend](https://github.com/CkoThuw11/AdventureWork-ERP-frontend)
4. **Follow the patterns**: Clean Architecture for backend, MVVM for frontend

### What We Look For

- ✅ Follows architectural patterns (Clean Architecture / MVVM)
- ✅ Adheres to SOLID principles
- ✅ Includes tests (coverage > 80%)
- ✅ Passes linting and type checks
- ✅ Updates documentation

### Resources

- [Main Contributing Guide](./CONTRIBUTING.md) - Multi-repo workflow
- [Backend Contributing](./backend/CONTRIBUTING.md) - Clean Architecture details
- [Frontend Contributing](./frontend/CONTRIBUTING.md) - MVVM and design system
- [Git Submodules Reference](./docs/git-submodules-reference.md) - Quick commands

---

## 📖 Technical Details

<details>
<summary>Click to see architecture diagrams and tech stack</summary>

### Backend: Clean Architecture (Onion Architecture)
```
┌─────────────────────────────────────┐
│     Interface Layer (API)           │  ← FastAPI Routes
├─────────────────────────────────────┤
│     Application Layer               │  ← Services, DTOs
├─────────────────────────────────────┤
│     Domain Layer                    │  ← Entities, Interfaces
├─────────────────────────────────────┤
│     Infrastructure Layer            │  ← Database, External APIs
└─────────────────────────────────────┘
```

### Frontend: MVVM + Signal Architecture
```
┌─────────────────────────────────────┐
│     View (Template)                 │  ← HTML Templates
├─────────────────────────────────────┤
│     ViewModel (Component)           │  ← Signals, UI Logic
├─────────────────────────────────────┤
│     Model (Service)                 │  ← HTTP, Data Transform
└─────────────────────────────────────┘
```

### Tech Stack
- **Backend**: Python 3.11+, FastAPI, Pydantic V2, SQLAlchemy (async)
- **Frontend**: Angular 21, Standalone Components, Signals, SCSS
- **Database**: PostgreSQL 16, Flyway migrations
- **DevOps**: Docker Compose, hot-reload for both backend and frontend

</details>

---

## 🤝 Contributing

This is a teaching project. If you want to contribute:

1. **Keep the teaching focus** - Every change should make learning easier
2. **Follow the patterns** - Demonstrate the principles we're teaching
3. **Update the guides** - If you change code, update relevant documentation
4. **Be honest about trade-offs** - Don't just show benefits, show costs too

---

## 📚 Additional Resources

- [AGENTS.md](./AGENTS.md) - Detailed coding standards and architectural rules
- [Backend README](./backend/README.md) - Backend-specific documentation
- [Frontend README](./frontend/README.md) - Frontend-specific documentation

---

## 🙏 Final Thoughts

**I built this because I wish I had it when I was learning.**

I spent years making the same mistakes:
- Putting business logic in controllers
- Tightly coupling code to databases
- Writing untestable code
- Creating technical debt

**You don't have to make those mistakes.**

Learn from this project. Break things. Ask questions. Understand the "why."

Then go build something amazing.

**— anhhoangdev**

---

## 📄 License

MIT License - Use this however helps you learn and grow.
