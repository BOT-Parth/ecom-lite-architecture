# Module 1 — Identity & Platform Foundation

## Read First

Before implementation, read and follow all permanent project documentation.

This includes (but is not limited to):

- Architecture documentation
- Backend onboarding
- Project development rules
- Standard reporting format

These documents are the source of truth.

---

# Role

You are the Backend Implementation Engineer.

Implement only the approved architecture.

Do not redesign the architecture.

Do not invent requirements.

If architecture is insufficient:

STOP.

Report the issue.

Wait for approval.

---

# Objective

Implement the Identity & Platform Foundation.

This module establishes the core entities and workflows that all future modules depend on.

The implementation should prioritize correctness, readability, maintainability, and centralized business logic.

---

# Scope

Implement the following:

## Authentication

- Register
- Login
- Get current user profile

Requirements:

- JWT authentication
- Password hashing using bcrypt
- Email uniqueness validation
- Username support (non-unique)
- Persistent login until logout
- Multiple device login allowed
- Prevent duplicate email registration

---

## Identity

Implement:

- Users
- Roles
- Permissions
- Role Permissions
- Platform User Roles

Authorization must be permission-based.

Do not hardcode role names in business logic.

---

## Store Requests

Implement:

- Create store request
- View own requests
- SUPER_ADMIN view all requests
- Approve request
- Reject request
- Needs Changes

Business rules must follow the approved architecture.

Store creation happens only after approval.

---

## Stores

Implement:

- Store entity
- Store lifecycle
- Store operational status
- Store visibility
- Slug uniqueness

Business rules:

- One user owns one primary store (current scope)
- Future expansion must remain possible
- Store created only after approved request

---

## Memberships

Implement:

user_store_memberships

Business rules:

- One membership per user per store
- Role changes update membership
- Membership removal follows approved architecture
- Tenant isolation applies

---

## SUPER_ADMIN

Implement:

- View users
- View stores
- Approve stores
- Reject stores
- Needs Changes
- Suspend stores
- Soft delete stores

Future platform functionality is out of scope.

---

# Excluded

Do NOT implement:

- Products
- Categories
- Inventory
- Orders
- Customers
- Analytics
- Reports
- Billing
- Email verification
- Password reset
- Notifications
- Ownership transfer
- Custom roles
- Subscription enforcement

---

# Architecture Requirements

Follow:

One Concern → One Home

Thin Controllers

Service Layer

Repository Layer

Validator Layer

Centralized Permission Logic

Centralized Validation

No duplicated business logic

Readable code

Professional comments

---

# Database

Implement the approved Prisma schema for this module only.

Create migrations.

Create seed data where appropriate.

Do not create tables for future modules.

---

# API

Implement only the APIs required for this module.

Follow the approved response format.

Use proper HTTP status codes.

---

# Documentation

Update documentation if implementation requires approved architectural updates.

Otherwise preserve documentation.

---

# Testing

Verify:

Authentication

Authorization

Store Request workflow

Store Approval workflow

Membership workflow

Tenant isolation

---

# Deliverables

Provide the Standard Reporting Format including:

- Summary
- Scope completed
- Files changed
- Database changes
- API changes
- Documentation changes
- Tests executed
- Manual verification
- Known limitations
- Suggested next milestone

Do not claim work that was not completed.
