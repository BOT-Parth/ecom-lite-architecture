# Module 1 — Identity & Platform API Contracts

## Authentication

### POST /auth/register

Purpose

Register a new platform user.

Authentication

None

Authorization

Public

---

### POST /auth/login

Purpose

Authenticate user.

Returns available login contexts.

Authentication

None

Authorization

Public

---

### GET /auth/profile

Purpose

Return current authenticated user.

Authentication

Required

Authorization

Authenticated User

---

## Store Requests

### POST /store-requests

Purpose

Submit a request to create a store.

Authentication

Required

Authorization

Authenticated User

Business Rules

- User must not already own a primary store.
- Only one active pending request is allowed.
- Request enters PENDING state.

---

### GET /store-requests/me

Purpose

View the authenticated user's requests.

Authentication

Required

Authorization

Authenticated User

---

### GET /admin/store-requests

Purpose

View all store requests.

Authentication

Required

Authorization

SUPER_ADMIN

---

### PATCH /admin/store-requests/:id/approve

Purpose

Approve a store request.

Business Rules

- Create Store.
- Create initial STORE_OWNER membership.
- Link requester as primary owner.
- Mark request APPROVED.

---

### PATCH /admin/store-requests/:id/reject

Purpose

Reject request.

Business Rules

Store is NOT created.

---

### PATCH /admin/store-requests/:id/needs-changes

Purpose

Return request for modification.

Business Rules

Store is NOT created.

Requester may edit and resubmit.

---

## Stores

### GET /stores/me

Purpose

Return stores available in current user contexts.

---

### GET /stores/:id

Purpose

Return store information.

Authorization

Membership OR SUPER_ADMIN

---

### PATCH /stores/:id

Purpose

Update store.

Authorization

Permission-based.

---

### DELETE /stores/:id

Purpose

Soft delete store.

Business Rules

Store enters SOFT_DELETED.

Recovery period applies.

---

## Memberships

### GET /stores/:id/members

Purpose

View store members.

---

### POST /stores/:id/members

Purpose

Invite/add member.

Business Rules

One membership per user per store.

---

### PATCH /stores/:id/members/:membershipId

Purpose

Update member role.

Business Rules

Update membership.

Do not create duplicate memberships.

---

### DELETE /stores/:id/members/:membershipId

Purpose

Remove member.

Business Rules

Follow approved membership removal workflow.

---

## SUPER_ADMIN

### GET /admin/users

### GET /admin/stores

### PATCH /admin/stores/:id/suspend

### PATCH /admin/stores/:id/restore

### DELETE /admin/stores/:id
