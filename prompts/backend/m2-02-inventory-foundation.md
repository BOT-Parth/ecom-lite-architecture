# Milestone M2.2 — Inventory Foundation

## Objective

Implement inventory management for products inside stores.

This milestone introduces stock tracking functionality while maintaining tenant isolation and existing RBAC rules.

## Scope

The module will support:

- Inventory record creation
- Inventory retrieval
- Stock quantity updates
- Stock availability checking
- Store-level permission protection

## Business Rules

- Every product belongs to exactly one store.
- Inventory belongs to a product.
- Only users with STORE-level inventory management permission can modify inventory.
- Users cannot access inventory from another store.
- Public users cannot modify inventory.
- Inventory quantity cannot become negative.

## Expected Architecture

Follow existing project architecture:

Routes
↓
Controllers
↓
Services
↓
Repositories
↓
Prisma Database

Controllers must remain thin.

Business logic belongs inside services.

Database access belongs inside repositories.

## Database Changes

Expected new model:

Inventory

Fields:

- id
- productId
- quantity
- createdAt
- updatedAt

Relationship:

Product 1 ---- 1 Inventory

## Verification Requirements

Implementation is not complete until:

- Prisma migration succeeds
- Database schema is verified
- API tests pass
- Tenant isolation is verified
- RBAC behavior is verified
- Documentation is updated
- Git commit is created
