# Catalog & Inventory Database Design

This document details the database schema layout, indexes, constraints, and relationships for the Category, Product, and Inventory models in the E-Com Lite system.

---

## Entity Relationship Diagram

```mermaid
erDiagram
    STORES ||--o{ CATEGORIES : owns }
    STORES ||--o{ PRODUCTS : owns }
    STORES ||--o{ ORDERS : receives }
    STORES ||--o{ CUSTOMERS : registers }
    CATEGORIES ||--o{ PRODUCTS : categorizes }
    PRODUCTS ||--o|{ INVENTORIES : stocks }
    CUSTOMERS ||--o{ ORDERS : places }
    ORDERS ||--|{ ORDER_ITEMS : contains }
    PRODUCTS ||--o{ ORDER_ITEMS : referenced_in }

    STORES {
        string id PK
        string name
        string slug UK
    }

    CATEGORIES {
        string id PK
        string name
        string slug
        string store_id FK
    }

    PRODUCTS {
        string id PK
        string name
        string description
        decimal price
        string_array image_urls
        string category_id FK
        string store_id FK
    }

    INVENTORIES {
        string id PK
        string product_id FK
        int quantity
    }

    CUSTOMERS {
        string id PK
        string store_id FK
        string first_name
        string last_name
        string email
        string phone
        string password
        boolean is_active
    }

    ORDERS {
        string id PK
        string order_number UK
        string store_id FK
        string customer_id FK
        string customer_name
        string customer_email
        string customer_phone
        string delivery_address
        decimal total_amount
        string status
    }

    ORDER_ITEMS {
        string id PK
        string order_id FK
        string product_id FK
        string product_name
        int quantity
        decimal unit_price
    }
```

---

## Schema Tables

### 1. `categories` Table

Stores product category structures scoped by tenant.

- **Fields**:
  - `id`: `TEXT` (UUID), Primary Key.
  - `name`: `TEXT`, Category display name.
  - `slug`: `TEXT`, URL slug.
  - `store_id`: `TEXT` (UUID), Foreign Key referencing `stores(id)`.
  - `created_at`: `TIMESTAMP`, Defaults to current time.
  - `updated_at`: `TIMESTAMP`, Automatically updates.
- **Relationships**:
  - Scoped to one `Store` (On Store deletion, Category is cascade deleted).
- **Constraints**:
  - `@@unique([store_id, slug])`: Slugs are unique per store context.

### 2. `products` Table

Stores product listings.

- **Fields**:
  - `id`: `TEXT` (UUID), Primary Key.
  - `name`: `TEXT`, Product name.
  - `description`: `TEXT` (Nullable), Optional product description.
  - `price`: `DECIMAL(10, 2)`, Monetarily precise price value.
  - `image_urls`: `TEXT[]` (Array), String array of image resource links.
  - `category_id`: `TEXT` (UUID, Nullable), Foreign Key referencing `categories(id)`.
  - `store_id`: `TEXT` (UUID), Foreign Key referencing `stores(id)`.
  - `created_at`: `TIMESTAMP`, Defaults to current time.
  - `updated_at`: `TIMESTAMP`, Automatically updates.
- **Relationships**:
  - Scoped to one `Store` (On Store deletion, Product is cascade deleted).
  - Optionally belongs to one `Category`.
- **Constraints**:
  - `onDelete: SetNull` on `category_id`: Deleting a category sets `category_id` to `NULL` for associated products, ensuring product listings remain active.

### 3. `inventories` Table

Tracks stock quantities for products.

- **Fields**:
  - `id`: `TEXT` (UUID), Primary Key.
  - `product_id`: `TEXT` (UUID), Foreign Key referencing `products(id)`, Unique.
  - `quantity`: `INTEGER`, Current stock level, Defaults to `0`.
  - `created_at`: `TIMESTAMP`, Defaults to current time.
  - `updated_at`: `TIMESTAMP`, Automatically updates.
- **Relationships**:
  - Mapped one-to-one to a `Product`.
- **Constraints**:
  - `onDelete: Cascade` on `product_id`: Deleting a product automatically deletes its inventory entry.
  - `quantity >= 0`: Enforced at runtime by the application logic and validator layers (must be a non-negative integer).
  - Scoped under the tenant's context via the product's associated `store_id` (cross-store queries are forbidden).


### 4. `customers` Table

Stores authenticated customer identities scoped to a specific store.

- **Fields**:
  - `id`: `TEXT` (UUID), Primary Key.
  - `store_id`: `TEXT` (UUID), Foreign Key referencing `stores(id)`.
  - `first_name`: `TEXT`, Customer's first name.
  - `last_name`: `TEXT`, Customer's last name.
  - `email`: `TEXT`, Customer email.
  - `phone`: `TEXT`, Customer phone number.
  - `password`: `TEXT`, Bcrypt hashed password.
  - `is_active`: `BOOLEAN`, Defaults to `true`.
  - `created_at`: `TIMESTAMP`, Defaults to current time.
  - `updated_at`: `TIMESTAMP`, Automatically updates.
- **Relationships**:
  - Scoped to one `Store` (Cascade deleted).
- **Constraints**:
  - `@@unique([store_id, email])`: Emails are unique per store context.

### 5. `orders` Table

Stores customer order records with embedded contact details.

- **Fields**:
  - `id`: `TEXT` (UUID), Primary Key.
  - `order_number`: `TEXT`, Globally unique alphanumeric string.
  - `store_id`: `TEXT` (UUID), Foreign Key referencing `stores(id)`.
  - `customer_id`: `TEXT` (UUID, Nullable), Foreign Key referencing `customers(id)`.
  - `customer_name`: `TEXT`, Snapshot of customer name.
  - `customer_email`: `TEXT`, Snapshot of customer email.
  - `customer_phone`: `TEXT`, Snapshot of customer phone.
  - `delivery_address`: `TEXT`, Delivery address.
  - `total_amount`: `DECIMAL(10, 2)`, Final computed total amount for the order.
  - `status`: `ENUM`, Defaults to `PLACED`.
  - `created_at`: `TIMESTAMP`, Defaults to current time.
  - `updated_at`: `TIMESTAMP`, Automatically updates.
- **Relationships**:
  - Scoped to one `Store` (On Store deletion, Order is cascade deleted).
  - Belongs to a `Customer` (On Customer deletion, set null).
  - Contains many `OrderItem`s (cascade deleted).

### 6. `order_items` Table

Stores historical snapshots of ordered products.

- **Fields**:
  - `id`: `TEXT` (UUID), Primary Key.
  - `order_id`: `TEXT` (UUID), Foreign Key referencing `orders(id)`.
  - `product_id`: `TEXT` (UUID, Nullable), Foreign Key referencing `products(id)`.
  - `product_name`: `TEXT`, Historical snapshot of product name.
  - `quantity`: `INTEGER`, Quantity ordered.
  - `unit_price`: `DECIMAL(10, 2)`, Historical snapshot of unit price.
  - `created_at`: `TIMESTAMP`, Defaults to current time.
  - `updated_at`: `TIMESTAMP`, Automatically updates.
- **Relationships**:
  - Belongs to an `Order` (Cascade).
  - References a `Product` (SetNull).
- **Constraints**:
  - `onDelete: SetNull` on `product_id`: Deleting a product sets `product_id` to `NULL` to preserve historical order records.

---
*Last verified against code on 2026-07-22: Verified architectural principles against current codebase.*