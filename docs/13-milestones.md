# Implementation Milestones

## Milestone 1 — Identity & Platform Foundation

Status: **Implemented & Clean**

Completed:
- Prisma database setup
- User model, Platform roles, Platform permissions, Store roles, Store permissions, Membership relations
- Database migration and Seed system
- User registration and Login (JWT)
- Profile retrieval, password hashing
- Authentication and RBAC middlewares
- Context separation (path-based `storeId`)

Verification:
- Last verified against code on 2026-07-18: Verified presence of middlewares, route restrictions, JWT signing, and correct DB relationships in `auth.service.js` and `rbac.middleware.js`.

---

## Milestone 2 — Catalog & Stock Foundation

Status: **Implemented & Clean**

Completed:
- Category model, Product model, Inventory model
- Product CRUD, Category CRUD, Inventory stock updates
- Public catalog listings
- Store permission isolation (requires `MANAGE_PRODUCTS`)
- Non-negative integer quantity validation

Verification:
- Last verified against code on 2026-07-19: Verified CRUD endpoints. Refactored Zod validation schemas into `fields.validator.js` to resolve duplication. Implemented `withStoreScope` helper in `helpers.js` to resolve tenant scope query duplication in repositories (with `inventory.repository.js` and `category.findBySlug` documented as approved architectural exceptions in `docs/09-backend-architecture.md` and inline comments).

---

## Milestone 3 — Platform Store Directory

Status: **Implemented & Clean**

Completed:
- Platform Store Directory API endpoint (`GET /stores/platform`)
- Administrative authorization check via platform permission `APPROVE_STORE`
- Decoupled logic through thin routes, controllers, services, and repository layers
- Data minimization query via explicit Prisma `select` in `listPlatformStores()`

Verification:
- Last verified against code on 2026-07-18: Verified route, RBAC middleware wrapping, and `select` query in `store.repository.js`.

---

## Milestone 4 — Store Registration Details

Status: **Implemented & Clean**

Completed:
- Expanded `StoreRequest` and `Store` database models with optional description and avatarUrl columns.
- Implemented optional description and avatarUrl validators in Zod schema.
- Enforced data minimization across repository queries using explicit Prisma selects.
- Handled metadata mapping during store request approval, transferring description and avatarUrl correctly.

Verification:
- Last verified against code on 2026-07-18: Verified validators in `store.validator.js`, schema shapes, and mapping in `store.repository.js`.

---

## Milestone 5 — Store Settings & Metadata Management

Status: **Implemented & Clean**

Completed:
- Added store settings endpoints (`GET /stores/:storeId/settings` and `PATCH /stores/:storeId/settings`).
- Protected setting routes using store-scoped permission `MANAGE_STORE`.
- Added settings validation schema (`updateSettingsSchema`).
- Enforced data minimization via explicit select statements.

Verification:
- Last verified against code on 2026-07-18: Verified endpoints, permission checks, and `storeId` constraints.

---

## Backend Infrastructure (New/Unplanned)

Status: **Implemented & Clean**

Completed:
- Added `GET /health` endpoint for application load balancers and deployment verification.

Verification:
- Last verified against code on 2026-07-19: Verified `GET /health` endpoint now correctly routes through `health.controller.js` to resolve layering violations. Verified `process.env` reads are centralized into `src/config/env.js` and securely validated using Zod.

---

## Milestone 6 — Store Deletion & Recovery

Status: **Not Implemented**

Planned Scope:
- Build backend Soft Delete endpoint (`DELETE /stores/:storeId` - flags `isDeleted: true` and sets `deletedAt`).
- Implement soft-delete filters at the repository query level.
- Build recovery endpoint to clear the deletion flag.

---

## Milestone 7 — Theme & Aesthetic Customisation

Status: **Not Implemented**

Planned Scope:
- Mostly frontend layout and preference persistence features.

---

## Milestone 8 — Customer Orders MVP

Status: **Implemented & Clean**

Completed:
- Order and OrderItem models, tracking relationships, and cascading deletes (Store→Order Cascade, Product→OrderItem SetNull).
- `MANAGE_ORDERS` store permission.
- Customer order placement (atomic inventory deduction, item merging). *(Replaces deprecated Public order placement)*
- Customer order tracking via Order History. *(Replaces deprecated Public order tracking)*
- Protected order management APIs (status transitions, atomic inventory restoration on cancel).

Verification:
- Last verified against code on 2026-07-23: Verified tracking and placement endpoints reflect the Customer Accounts shift. Guest tracking and checkout flows are completely deprecated.
