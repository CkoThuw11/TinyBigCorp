# Learning Path: Enterprise Software Architecture

Welcome! This documentation is designed to teach you how to build maintainable, scalable software systems.

## 🎯 Who This Is For

**You should read this if you**:
- Are a junior/mid-level developer wanting to level up
- Have built apps but struggle with maintainability as they grow
- Want to understand "why" behind enterprise patterns
- Are joining a team that uses Clean Architecture

**Prerequisites**:
- Basic Python and TypeScript/JavaScript
- Understanding of functions, classes, and basic OOP
- Have written at least one web application

## 📚 Learning Path

Read these guides in order. Each builds on concepts from previous guides.

### 1. [Why This Project Exists](./01-why-this-project.md)
**Time**: 15 minutes  
**What you'll learn**: Context for why we use these patterns

Start here to understand the motivation. This isn't just "best practices" - it's solving real problems.

---

### 2. [Clean Architecture Deep Dive](./02-clean-architecture.md)
**Time**: 45 minutes  
**What you'll learn**: How to organize code into layers

The foundation of everything else. You'll trace actual code through all four layers.

**Exercise**: Walk through user creation from HTTP request to database

---

### 3. [SOLID Principles](./03-solid-principles.md)
**Time**: 60 minutes  
**What you'll learn**: Five principles that make code maintainable

Each principle has a hands-on exercise. Don't skip them!

**Exercise**: Refactor code that violates each principle

---

### 4. [OOP in Modern Development](./04-oop-in-modern-development.md)
**Time**: 30 minutes  
**What you'll learn**: When to use classes vs functions

Honest discussion about trade-offs. OOP isn't always the answer.

**Exercise**: Decide which approach fits different scenarios

---

### 5. [Data Contracts and Type Safety](./05-data-contracts.md)
**Time**: 40 minutes  
**What you'll learn**: Using Pydantic for validation and contracts

See how type safety prevents bugs before they reach production.

**Exercise**: Create a Pydantic model with custom validators

---

### 6. [Database Migrations and Team Workflows](./06-database-migrations.md)
**Time**: 35 minutes  
**What you'll learn**: How teams collaborate on database changes

Real scenarios of migration conflicts and how to resolve them.

**Exercise**: Create a migration and handle a version conflict

---

### 7. [Practical Development Workflow](./07-development-workflow.md)
**Time**: 90 minutes  
**What you'll learn**: End-to-end feature development

Guided tutorial: Add a complete feature from database to API.

**Exercise**: Add a "Product" entity following the same pattern

---

### 8. [Testing Strategies](./08-testing-strategies.md)
**Time**: 50 minutes  
**What you'll learn**: What to test and at which level

The testing pyramid and how to apply it to Clean Architecture.

**Exercise**: Write unit and integration tests

---

## 🚀 Getting Started

1. **Set up the project**: Run `docker-compose up` in the project root
2. **Start with Guide 01**: Don't skip ahead - concepts build on each other
3. **Do the exercises**: Reading isn't enough, you need to write code
4. **Take breaks**: These are dense topics, don't rush

## 💡 How to Use These Guides

### Each Guide Has:
- **Clear learning objective**: What you'll understand after reading
- **Time estimate**: Plan your learning sessions
- **Stop and Think moments**: Pause before reading the answer
- **Hands-on exercises**: Apply what you learned
- **Check your understanding**: Quiz questions

### Tips for Success:
- ✅ **Do the exercises** - This is where learning happens
- ✅ **Ask "why"** - Don't just memorize patterns
- ✅ **Experiment** - Break things to see what happens
- ✅ **Take notes** - Write down insights in your own words
- ❌ **Don't rush** - Understanding > speed
- ❌ **Don't skip exercises** - They're not optional

## 🤔 Questions?

As you read, you might wonder:
- "Isn't this over-engineered?"
- "When would I actually use this?"
- "What if I'm building a simple app?"

These guides address these questions honestly. We'll discuss trade-offs, not just benefits.

## 📖 Additional Resources

After completing these guides:
- [AGENTS.md](../AGENTS.md) - Detailed coding standards for this project
- [Backend README](../backend/README.md) - Backend-specific documentation
- [Frontend README](../frontend/README.md) - Frontend-specific documentation

## 🎓 What's Next?

After completing all guides, you should be able to:
- ✅ Explain why Clean Architecture matters
- ✅ Apply SOLID principles to your code
- ✅ Make informed decisions about OOP vs functional approaches
- ✅ Use Pydantic for type-safe APIs
- ✅ Collaborate on database changes without conflicts
- ✅ Add features following established patterns
- ✅ Write tests at the appropriate level

---

**Ready to start?** → [Begin with Guide 01: Why This Project Exists](./01-why-this-project.md)
