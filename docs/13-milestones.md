## Milestone 1 — Identity & Platform Foundation

### M1.1 Database Foundation ✅

Completed:

- Prisma database setup
- User model
- Platform roles
- Platform permissions
- Store roles
- Store permissions
- Membership relations
- Database migration
- Seed system

Verification:

- Prisma validation passed
- Migration verified
- Seed verified
- Database constraints verified

### M1.2 Authentication Foundation ✅

Completed:

- User registration
- Login
- JWT authentication
- Profile retrieval
- Password hashing
- Authentication middleware
- Error handling
- Validation layer

Verification:

- Registration tested
- Duplicate email tested
- Login tested
- Invalid password tested
- Protected route tested

### M1.3 Authorization RBAC Foundation ✅

Completed:

- Platform permission resolution
- Store permission resolution
- RBAC middleware
- Context separation

Rules implemented:

- Platform permissions cannot access store permissions
- Store permissions cannot access platform permissions
- Active user check required
- Store context uses route storeId

Verification:

- Platform permission tests passed
- Store permission tests passed
- Context isolation tested

### M1.4 Store Management Foundation ✅

Completed:

- Store request workflow
- Platform approval workflow
- Store rejection workflow
- Public store directory
- User store directory
- Atomic store approval transaction

Business Rules:

- Store ownership is represented through UserStoreMembership
- STORE_OWNER role assigned after approval
- Store approval and operational status are separated
- Operational status uses:
  - OPEN
  - CLOSED
  - SUSPENDED

Verification:

- Request creation tested
- Approval tested
- Rejection tested
- Duplicate slug protection tested
- Membership creation tested
- Public directory tested

## Milestone 2 — Catalog & Stock Foundation

### M2.1 Catalog Foundation ✅

Completed:

- Category model
- Product model
- Category CRUD endpoints
- Product CRUD endpoints
- Public catalog listings

Business Rules:
- Product category relation is optional.
- Category deletion sets category relation to null (does not delete products).

Verification:
- Category CRUD tests passed
- Product CRUD tests passed
- SetNull cascade deletion tested

### M2.2 Inventory Foundation ✅

Completed:

- Database model: Inventory
- Product one-to-one relationship
- Retrieve stock endpoints
- Update stock endpoints
- Store permission isolation (requires MANAGE_PRODUCTS)
- Non-negative integer quantity validation

Verification:
- Prisma validation passed
- Migration applied successfully
- Automated test suite passed (public retrieve, owner update, cross-store check, regular user block, negative quantity reject, DB cleanup)

## Milestone 3 — Platform Store Directory ✅ (Current Milestone Complete)

Completed:
- Platform Store Directory API endpoint (`GET /stores/platform`)
- Administrative authorization check via platform permission `APPROVE_STORE` (restricted to `SUPER_ADMIN` role)
- Decoupled logic through thin routes, controllers, services, and repository layers
- Data minimization query via explicit Prisma `select` in `listPlatformStores()`
- Multi-suite integration testing verifying access controls, returned payloads, and data minimization constraints

Verification:
- Platform directory integration tests passed (`test-stores.js`)
- Full live HTTP verification (authorized and unauthorized calls) passed (`verify-platform-directory.js`)

## Milestone 4 — Store Registration Details ✅ (Current Milestone Complete)

Completed:
- Expanded `StoreRequest` and `Store` database models with optional description and avatarUrl columns.
- Implemented optional description (max 500 characters) and avatarUrl (max 2048 characters) validators in Zod schema.
- Enforced data minimization across repository queries using explicit Prisma selects.
- Handled metadata mapping during store request approval, transferring description and avatarUrl correctly from request to store.

Verification:
- Store management integration tests passed (`test-stores.js`)
- Full live HTTP verification (payload checks, invalid inputs, and directory lists) passed (`verify-store-registration-details.js`)

## Milestone 5 — Store Settings & Metadata Management ✅ (Current Milestone Complete)

Completed:
- Added store settings endpoints (`GET /stores/:storeId/settings` and `PATCH /stores/:storeId/settings`).
- Protected setting routes using store-scoped permission `MANAGE_STORE` (assigned to `STORE_OWNER`).
- Implemented `SUPER_ADMIN` store access bypass to allow platform administrators to view/edit settings.
- Added settings validation schema (`updateSettingsSchema` in `store.validator.js`).
- Enforced data minimization via explicit select statements in service and repository queries.

Verification:
- Store settings integration verifications passed (`verify-store-settings.js`)
- All existing test suites (rbac, auth, stores, inventory, catalog) run successfully.

## Milestone 6 — Store Deletion & Recovery ⏳ (Future Milestone)

Planned Scope:
- Build Store Deletion request controls in frontend Store Settings.
- Build backend Soft Delete endpoint (`DELETE /stores/:storeId` - flags `isDeleted: true` and sets `deletedAt`).
- Implement soft-delete filters at the repository query level for public storefronts and tenant-facing listings.
- Build Platform Admin recovery dashboard and recovery endpoint to clear the deletion flag.

## Milestone 7 — Theme & Aesthetic Customisation ⏳ (Future Milestone)

Planned Scope:
- Implement client-side theme context supporting Dark Mode and Light Mode switching.
- Save user theme preferences.
- Polish layout and component styling using harmonious HSL color sets across both themes.


