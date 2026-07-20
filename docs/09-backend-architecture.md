# Backend Architecture & Design Flow

This document details the layered architecture structure, design guidelines, request flows, and tenant isolation policies applied within the E-Com Lite backend application.

---

## Architectural Principles

The application enforces a strictly decoupled, layered architecture:

```
  HTTP Client Request
         ↓
    Express Routes
         ↓
  Auth & RBAC Middleware
         ↓
    Zod Validator
         ↓
     Controller
         ↓
      Service
         ↓
     Repository
         ↓
    Prisma Client
         ↓
    PostgreSQL
```

### 1. Routes ([src/routes](file:///Users/bot/EC/ecom-lite-backend/src/routes/))
- **Responsibility**: Map HTTP verbs and paths to controllers.
- **Rules**: Register endpoints, route parameters, and apply middleware chains (authentication, validation, permissions). Do not place any handling logic here.

### 2. Middleware ([src/middleware](file:///Users/bot/EC/ecom-lite-backend/src/middleware/))
- **Responsibility**: Enforce cross-cutting concerns (authentication, authorization, request parsers).
- **Rules**:
  - `authenticate`: Extracts and decodes JWT, verifying `userId` and loading context.
  - `requireStorePermission(permission)`: Checks if the user holds a `UserStoreMembership` inside the target store (parsed from path parameter `storeId`) that maps to the requested permission. Enforces strict isolation boundaries.

### 3. Validators ([src/validators](file:///Users/bot/EC/ecom-lite-backend/src/validators/))
- **Responsibility**: Request input validation.
- **Rules**: Use Zod to define schema shapes. Perform type conversions and reject bad inputs before requests hit controllers.

### 4. Controllers ([src/controllers](file:///Users/bot/EC/ecom-lite-backend/src/controllers/))
- **Responsibility**: HTTP interface only.
- **Rules**: Keep controllers extremely thin. Extract parameters (`req.params`, `req.body`, `req.query`, `req.user`), invoke service methods, and delegate response formatting to `sendSuccess` helper. Avoid placing queries or business rules inside controllers.

### 5. Services ([src/services](file:///Users/bot/EC/ecom-lite-backend/src/services/))
- **Responsibility**: Own all business logic, workflow sequencing, and validations.
- **Rules**: Orchestrate data updates, check business rules (e.g. category-to-store ownership match), and raise domain specific errors (e.g. `ConflictError`, `NotFoundError`) mapping automatically to HTTP responses.

### 6. Repositories ([src/repositories](file:///Users/bot/EC/ecom-lite-backend/src/repositories/))
- **Responsibility**: Direct database persistence layer.
- **Rules**: Contain all ORM queries (`prisma`). Keep repositories completely independent of business constraints. Provide clean async interfaces for services. Use the shared `withStoreScope` helper from `src/repositories/helpers.js` to enforce tenant isolation filters without duplicating `where: { storeId }` logic.

---

## Tenant & Store Isolation Policy

E-Com Lite is a multi-tenant platform where every store must remain completely isolated from other stores:

1. **Path-based Scoping**: All catalog and inventory routes are nested under `/stores/:storeId/...`. This forces all requests to specify which store they are acting upon.
2. **Context Resolution**: The authorization middleware resolves `storeId` strictly from the route path parameter `req.params.storeId`. Supporting headers or request body contexts is forbidden to prevent context spoofing.
3. **Cross-Store Prevention**: The service layer double-checks that any referenced entity (e.g., product category inside a product creation payload) belongs to the same target `storeId` before executing mutations.

### Tenant Scoping Exceptions
While most repository queries use the `withStoreScope` helper for tenant isolation, the following are known, approved exceptions:
- `category.repository.js` `findBySlug(storeId, slug)`: Cannot use the helper because Prisma's `findUnique` strict mode restricts the `where` clause exactly to the compound unique constraint (`storeId_slug`), and throws an error if a flat `storeId` is appended.
- `inventory.repository.js`: Inventory is strictly scoped via `productId` rather than `storeId` directly, so the flat `storeId` helper does not apply mechanically.

---

## Configuration & Environment

Environment variables must be centralized, validated using Zod, and exported from `src/config/env.js`. Direct `process.env` reads throughout the application are prohibited.

### Approved Exceptions
- `prisma.config.ts`: Reads `process.env.DATABASE_URL` directly. This is an intentional exception because it is executed by the Prisma CLI entirely outside the standard application module lifecycle. See `adr/0001-prisma-config-env-exception.md`.


---
*Last verified against code on 2026-07-19: Verified architectural principles against current codebase.*