# M1.1 — Database Foundation

## Role

You are implementing only the database foundation for E-Com Lite.

Read:

- Backend onboarding
- Project development rules
- Database architecture documentation

---

# Scope

Implement Prisma models for:

## User

Required fields:

- id
- email
- password
- username
- timestamps
- active status

Rules:

- Email must be unique.
- Username is not unique.

---

## Role

Represents a collection of permissions.

Examples:

- SUPER_ADMIN
- STORE_OWNER
- STORE_STAFF
- CUSTOMER

Do not hardcode authorization logic.

---

## Permission

Represents individual capabilities.

Examples:

- CREATE_STORE
- APPROVE_STORE
- MANAGE_PRODUCTS

---

## RolePermission

Maps permissions to roles.

---

## UserRole

Maps users to platform roles.

---

## Store

Tenant entity.

Requirements:

- Unique identifier
- Unique slug
- Status fields
- Ownership relationship

---

## StoreRequest

Represents a request to create a store.

Statuses:

- PENDING
- APPROVED
- REJECTED
- NEEDS_CHANGES

---

## UserStoreMembership

Maps users to stores.

Requirements:

- User
- Store
- Role

Rules:

- One membership per user per store.

---

# Constraints

Do NOT implement:

- APIs
- Controllers
- Services
- Authentication
- Authorization middleware
- Products
- Inventory
- Orders

---

# Database Rules

- Use Prisma ORM.
- Follow naming conventions.
- Avoid unnecessary future fields.
- Preserve multi-tenant isolation.

---

# Deliverables

Report:

- Schema changes
- Migration created
- Seed changes
- Design decisions
- Verification performed
