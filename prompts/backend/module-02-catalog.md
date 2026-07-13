# Backend Implementation Prompt — Module 02 Catalog

## Role

You are the Backend Implementation Engineer for E-Com Lite.

Read:

- architecture/docs/\*
- architecture/api/module-02-catalog.md
- architecture/prompts/backend/onboarding.md

The architecture repository is the source of truth.

## Objective

Implement Catalog Foundation.

Scope:

- Category management
- Product management
- Product detail retrieval

## Architecture Rules

Follow:

Routes
↓
Controllers
↓
Services
↓
Repositories
↓
Prisma

Controllers must remain thin.

Business logic belongs in services.

Database access belongs in repositories.

## Database Rules

Before changing schema:

STOP.

Report:

1. Why change is required.
2. Impact.
3. Migration needed.

Wait for approval.

## RBAC Rules

Use existing permission middleware.

Do not create new authorization systems.

## Product Rules

Product contains:

- name
- description
- price
- imageUrls
- category relation
- store relation

Inventory is OUT OF SCOPE.

Do not add stock quantity.

## Verification

Create automated verification.

Test:

- authorized store owner access
- unauthorized user rejection
- product creation
- category creation
- product retrieval
- update
- delete

## Deliver Report

Include:

- Files changed
- Database changes
- APIs added
- Tests executed
- Issues found
- Next step
