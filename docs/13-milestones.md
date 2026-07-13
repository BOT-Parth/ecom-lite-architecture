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

