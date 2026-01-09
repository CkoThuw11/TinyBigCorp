# Practical Development Workflow

**Time**: 90 minutes  
**Prerequisites**: All previous guides  
**Goal**: Add a complete feature from database to API

---

## Guided Tutorial: Add a "Product" Entity

We'll add products to the system, following Clean Architecture.

---

## Step 1: Database Migration

**File**: `backend/migrations/V2__add_products.sql`

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    in_stock BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_products_in_stock ON products(in_stock);
```

**Run**: `docker-compose restart flyway`

---

## Step 2: Domain Entity

**File**: `backend/src/domain/entities/product.py`

```python
from dataclasses import dataclass
from datetime import datetime
from decimal import Decimal
from typing import Optional

@dataclass
class Product:
    id: Optional[int]
    name: str
    description: Optional[str]
    price: Decimal
    in_stock: bool
    created_at: datetime
    updated_at: datetime
    
    def mark_out_of_stock(self) -> None:
        """Business rule: Mark product as out of stock."""
        self.in_stock = False
    
    def update_price(self, new_price: Decimal) -> None:
        """Business rule: Update product price."""
        if new_price <= 0:
            raise ValueError("Price must be positive")
        self.price = new_price
        self.updated_at = datetime.utcnow()
```

---

## Step 3: Repository Interface

**File**: `backend/src/domain/repositories/product_repository.py`

```python
from abc import ABC, abstractmethod
from typing import List, Optional
from src.domain.entities.product import Product

class IProductRepository(ABC):
    @abstractmethod
    async def get_by_id(self, product_id: int) -> Optional[Product]:
        pass
    
    @abstractmethod
    async def list_all(self, skip: int = 0, limit: int = 100) -> List[Product]:
        pass
    
    @abstractmethod
    async def create(self, product: Product) -> Product:
        pass
    
    @abstractmethod
    async def update(self, product: Product) -> Product:
        pass
```

---

## Continue This Pattern...

Follow the same structure for:
- Application DTOs
- Application Service
- Infrastructure Repository
- API Routes

---

## Your Exercise

**Add a "Category" entity** following the same pattern:
1. Migration
2. Domain entity
3. Repository interface
4. DTOs
5. Service
6. Repository implementation
7. API routes

---

## Check Your Understanding

1. Why start with the database migration?
2. Why create the interface before the implementation?
3. How would you test the service without a database?

---

**Next**: [Guide 08: Testing Strategies](./08-testing-strategies.md)
