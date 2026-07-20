# M1.3 — Authorization RBAC Foundation

## Objective

Implement the authorization foundation for E-Com Lite.

Authentication already exists.

This milestone adds permission-based authorization using the approved RBAC architecture.

---

# Source of Truth

Follow:

- architecture/docs/03-actors.md (Planned — not yet written)
- architecture/docs/04-permission-model.md
- architecture/docs/06-database-design.md
- architecture/prompts/backend/onboarding.md

Do not redesign the permission model.

---

# Authorization Model

Authorization is based on:

User

↓

Context

↓

Role

↓

Permission

There are two separate contexts.

---

# Platform Context

Relationship:

User

↓

UserPlatformRole

↓

PlatformRole

↓

PlatformPermission

Example:

SUPER_ADMIN

Permissions:

- CREATE_STORE
- APPROVE_STORE

---

# Store Context

Relationship:

User

↓

UserStoreMembership

↓

StoreRole

↓

StorePermission

Example:

STORE_OWNER

STORE_STAFF

CUSTOMER

---

# Important Rule

Do not mix platform permissions and store permissions.

A user permission depends on the context being checked.

---

# Scope

Implement:

## 1. Permission Resolution Service

Create centralized permission checking logic.

The system must answer:

"Does this user have this permission in this context?"

Examples:

Platform:

Does user have APPROVE_STORE?

Store:

Does user have MANAGE_PRODUCTS for Store A?

---

## 2. Authorization Middleware

Create reusable middleware.

Examples:

requirePlatformPermission(permission)

requireStorePermission(permission)

Middleware should:

- receive required permission
- identify authenticated user
- check permission
- allow or reject request

---

## 3. Permission Utilities

Create reusable helpers if required.

Permission logic must not be duplicated.

---

# Architecture Rules

Follow:

Routes

↓

Controllers

↓

Services

↓

Repositories

Rules:

Controllers must not:

- query roles
- query permissions
- calculate access

Services/utilities own permission logic.

---

# Out Of Scope

Do not implement:

- Store APIs
- Store creation
- Store approval
- Membership APIs
- Product APIs
- Frontend changes

Only implement authorization foundation.

---

# Verification

Perform verification for:

## Platform Permission

Test:

- SUPER_ADMIN with permission succeeds.
- User without permission fails.

---

## Store Permission

Test:

- STORE_OWNER permission succeeds.
- STORE_STAFF permission succeeds only for assigned permissions.
- User without store membership fails.

---

## Context Isolation

Verify:

- Platform permission cannot be used as store permission.
- Store permission cannot be used as platform permission.

---

# Documentation

If architecture-relevant decisions are introduced:

Update documentation.

Do not modify architecture without approval.

---

# Final Report

Provide:

1. Files changed.
2. Permission flow explanation.
3. Middleware flow explanation.
4. Verification performed.
5. Results.
6. Known limitations.
7. Code walkthrough.

After completion:

Stop and request approval before moving to the next milestone.
