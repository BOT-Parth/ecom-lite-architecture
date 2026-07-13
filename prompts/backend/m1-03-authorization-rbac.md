# M1.3 — Authorization RBAC Foundation

## Objective

Implement authorization foundation based on the approved role and permission architecture.

Authentication already exists.

This milestone adds authorization.

---

# Architecture Model

Authorization follows:

User

↓

Context

↓

Role

↓

Permission

Platform context:

UserPlatformRole
|
PlatformRole
|
PlatformPermission

Store context:

UserStoreMembership
|
StoreRole
|
StorePermission

---

# Scope

Implement:

## Permission Resolution

Create centralized permission checking logic.

The system should be able to answer:

"Does this user have this permission in this context?"

---

## Middleware

Create authorization middleware for:

Platform permissions.

Example:

requirePlatformPermission()

Store permissions.

Example:

requireStorePermission()

---

## Required Behavior

A user may have:

Platform permissions.

Store permissions.

Do not mix contexts.

---

# Requirements

Follow:

Routes
↓
Controllers
↓
Services
↓
Repositories

Controllers must not calculate permissions.

Permission logic belongs in services/utilities.

---

# Out Of Scope

Do not implement:

- Store APIs
- Store creation
- Membership management
- Frontend
- UI permissions

---

# Verification

Test:

- SUPER_ADMIN permission access.
- User without permission rejected.
- Store role permission resolution.
- Wrong context rejected.

---

# Reporting

Provide:

1. Files changed
2. Permission flow explanation
3. Middleware flow explanation
4. Verification results
5. Known limitations

Stop after completion and request approval.
