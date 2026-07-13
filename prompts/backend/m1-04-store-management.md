# M1.4 — Store Management Foundation

## Objective

Implement the Store Management foundation for E-Com Lite.

This milestone introduces the first complete tenant workflow:

User
↓
Store Request
↓
Approval
↓
Store Creation
↓
Store Membership

The implementation must use the existing authentication and RBAC systems.

---

# Source of Truth

Read:

- architecture/docs/02-business-rules.md
- architecture/docs/03-actors.md
- architecture/docs/04-permission-model.md
- architecture/docs/05-workflows.md
- architecture/docs/06-database-design.md
- architecture/prompts/backend/onboarding.md

Do not redesign existing architecture.

---

# Business Rules

## Store Creation Model

Users do not directly create stores.

Flow:

User submits store request.

↓

Request waits for approval.

↓

Platform administrator approves/rejects.

↓

Approved request creates store.

↓

User receives STORE_OWNER membership.

---

# Scope

Implement:

## 1. Store Request Creation

Endpoint:

POST /store-requests

Authenticated user can request a store.

Input:

- name
- slug

Rules:

- Validate input.
- Slug must be unique.
- Create request with status:

PENDING

---

## 2. Store Request Listing

Platform administrators can view requests.

Endpoint:

GET /store-requests

Requires:

Platform permission:

APPROVE_STORE

---

## 3. Approve Store Request

Endpoint:

PATCH /store-requests/:id/approve

Requires:

Platform permission:

APPROVE_STORE

Behavior:

When approved:

1. Update request status.
2. Create Store.
3. Create UserStoreMembership.
4. Assign STORE_OWNER role.

All operations must succeed together.

Use transaction if required.

---

## 4. Reject Store Request

Endpoint:

PATCH /store-requests/:id/reject

Requires:

APPROVE_STORE

Behavior:

Update request status:

REJECTED

Do not create store.

---

## 5. User Store Listing

Endpoint:

GET /stores/my

Returns stores where:

User has UserStoreMembership.

---

## 6. Public Store Listing

Endpoint:

GET /stores

Returns:

Active approved stores.

This is the public directory concept.

---

# Architecture Requirements

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

Rules:

Controllers:

- Handle HTTP only.
- No business logic.

Services:

- Store workflows.
- Approval workflow.
- Membership creation.

Repositories:

- Database access only.

---

# Database Rules

Use existing schema.

Do not change:

- Store model.
- UserStoreMembership.
- StoreRequest.
- Roles.
- Permissions.

If schema change is required:

STOP.

Report:

1. Problem.
2. Reason.
3. Proposed change.

Wait for approval.

---

# Authorization Rules

Use existing RBAC middleware.

Examples:

Approve request:

requirePlatformPermission("APPROVE_STORE")

Store access:

requireStorePermission(permission)

---

# Out Of Scope

Do not implement:

- Product management.
- Category management.
- Inventory.
- Orders.
- Payments.
- Email verification.
- Subscription billing.

---

# Verification

Test:

## Request Creation

- Authenticated user can create request.
- Duplicate slug rejected.

## Approval

- SUPER_ADMIN can approve.
- Normal user cannot approve.
- Store is created.
- STORE_OWNER membership created.

## Rejection

- Request rejected.
- No store created.

## Store Listing

- User sees only own stores.
- Public listing only shows approved active stores.

---

# Reporting

Provide:

1. Files changed.
2. API endpoints created.
3. Store workflow explanation.
4. Authorization flow.
5. Database impact.
6. Verification results.
7. Known limitations.

Stop after completion and request approval before next milestone.
