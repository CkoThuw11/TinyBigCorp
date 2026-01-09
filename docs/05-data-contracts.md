# Data Contracts and Type Safety

**Time**: 40 minutes  
**Prerequisites**: [Guide 04 - OOP in Modern Development](./04-oop-in-modern-development.md)  
**Goal**: Use Pydantic for validation and API contracts

---

## The Problem: Unvalidated Data

```python
def create_user(data: dict):
    # What if email is missing?
    # What if age is a string "twenty"?
    # What if someone sends 50 extra fields?
    user = User(
        email=data["email"],  # KeyError if missing!
        age=data["age"]       # TypeError if not int!
    )
```

**This fails at runtime.** In production. With real users.

---

## The Solution: Pydantic Models

```python
from pydantic import BaseModel, EmailStr, Field

class CreateUserCommand(BaseModel):
    model_config = ConfigDict(strict=True)
    
    email: EmailStr = Field(..., description="User email")
    age: int = Field(..., ge=18, le=120)
    username: str = Field(..., min_length=3, max_length=50)

def create_user(command: CreateUserCommand):
    # Guaranteed valid!
    user = User(
        email=command.email,  # Always valid email
        age=command.age       # Always 18-120
    )
```

**Benefits**:
- ✅ Validation at the edge
- ✅ Type safety
- ✅ Auto-generated docs
- ✅ Clear error messages

---

## Exercise

Create a Pydantic model for a Product:
- name (3-100 characters)
- price (positive float)
- category (enum: electronics, books, clothing)
- in_stock (boolean)

Add a custom validator: price must be a multiple of 0.01

<details>
<summary>Solution</summary>

```python
from enum import Enum
from pydantic import BaseModel, Field, field_validator

class Category(str, Enum):
    ELECTRONICS = "electronics"
    BOOKS = "books"
    CLOTHING = "clothing"

class CreateProductCommand(BaseModel):
    name: str = Field(..., min_length=3, max_length=100)
    price: float = Field(..., gt=0)
    category: Category
    in_stock: bool
    
    @field_validator('price')
    def validate_price(cls, v):
        if round(v, 2) != v:
            raise ValueError('Price must have at most 2 decimal places')
        return v
```

</details>

---

## Check Your Understanding

1. Why validate at the boundary (edge of system)?
2. How does Pydantic help with API documentation?
3. What's the difference between a Command and a DTO?

---

**Next**: [Guide 06: Database Migrations](./06-database-migrations.md)
