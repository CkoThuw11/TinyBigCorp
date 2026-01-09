# Database Migrations and Team Workflows

**Time**: 35 minutes  
**Prerequisites**: [Guide 05 - Data Contracts](./05-data-contracts.md)  
**Goal**: Collaborate on database changes without conflicts

---

## The Problem: Manual Database Changes

**Monday**: Developer A adds a column locally
```sql
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
```

**Tuesday**: Developer B pulls the code, runs it... **CRASH!**
```
Error: column "phone" does not exist
```

**Why?** Developer B's database doesn't have the column.

---

## The Solution: Flyway Migrations

### How It Works

1. **Write SQL migration files**:
```
migrations/
├── V1__initial_schema.sql
├── V2__add_phone_column.sql
└── V3__add_user_roles.sql
```

2. **Flyway tracks what's applied**:
```sql
SELECT * FROM flyway_schema_history;

| version | description      | installed_on        |
|---------|------------------|---------------------|
| 1       | initial schema   | 2026-01-09 10:00:00 |
| 2       | add phone column | 2026-01-09 11:00:00 |
```

3. **On startup, Flyway runs new migrations**

---

## Team Workflow

### Scenario: Two Developers, Same Feature

**Developer A** (adding user bio):
```sql
-- V5__add_user_bio.sql
ALTER TABLE users ADD COLUMN bio TEXT;
```

**Developer B** (adding user preferences):
```sql
-- V6__add_user_preferences.sql
CREATE TABLE user_preferences (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id)
);
```

**What happens**:
1. Both commit to Git
2. Developer B pulls A's changes
3. Flyway runs V5, then V6
4. **No conflicts!**

---

## Exercise

Create a migration that:
1. Adds a `products` table
2. Adds a foreign key from `orders` to `products`
3. Backfills existing orders with a default product

<details>
<summary>Solution</summary>

```sql
-- V7__add_products.sql
BEGIN;

-- Create products table
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Add default product
INSERT INTO products (name, price) VALUES ('Unknown Product', 0.00);

-- Add foreign key to orders
ALTER TABLE orders ADD COLUMN product_id INTEGER;

-- Backfill existing orders
UPDATE orders SET product_id = 1 WHERE product_id IS NULL;

-- Make it NOT NULL
ALTER TABLE orders ALTER COLUMN product_id SET NOT NULL;

-- Add foreign key constraint
ALTER TABLE orders ADD CONSTRAINT fk_orders_products 
    FOREIGN KEY (product_id) REFERENCES products(id);

COMMIT;
```

</details>

---

## Best Practices

1. **Never modify applied migrations** - Create new ones
2. **Use transactions** - All-or-nothing
3. **Test locally first** - Before pushing
4. **Keep migrations small** - One feature per migration

---

## Check Your Understanding

1. Why can't you just modify the database manually?
2. How does Flyway prevent conflicts between developers?
3. What happens if a migration fails halfway?

---

**Next**: [Guide 07: Development Workflow](./07-development-workflow.md)
