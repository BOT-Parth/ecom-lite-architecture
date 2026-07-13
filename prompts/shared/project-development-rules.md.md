# E-Com Lite — Project Development Rules

## Purpose

This document defines the mandatory rules for all implementation and verification work.

It complements the Architecture Documentation and Onboarding documents.

If a conflict exists:

Architecture Documentation takes precedence.

---

# Source of Truth

Priority order:

1. Architecture Documentation
2. Project Development Rules
3. Onboarding Documents
4. Milestone Prompt

Never assume missing requirements.

Never invent business rules.

---

# Scope

Only implement the approved milestone scope.

Everything not explicitly included is considered out of scope.

If additional work appears necessary:

STOP.

Report it.

Wait for architectural approval.

---

# Architecture Preservation

Never redesign:

- Business rules
- Database schema
- API contracts
- Permission model
- Folder structure
- Module boundaries

without architectural approval.

---

# Architecture Change Process

If implementation discovers an architectural issue:

1. Stop implementation.
2. Explain the issue.
3. Explain why the current architecture is insufficient.
4. Propose no implementation.
5. Wait for architecture approval.
6. Resume implementation only after approval.

Architecture changes must never happen silently.

---

# Coding Philosophy

The project values maintainability over cleverness.

Prioritize:

- Readability
- Consistency
- Centralization
- Simplicity
- Reusability

Code is written for humans first.

---

# One Concern, One Home

Every concern has one official location.

Examples:

Routes
→ routes/

Controllers
→ controllers/

Services
→ services/

Repositories
→ repositories/

Validators
→ validators/

Middleware
→ middleware/

Configuration
→ config/

Utilities
→ utils/

Never duplicate responsibilities.

---

# Layer Responsibilities

Routes

- Register endpoints only.

Controllers

- Receive request.
- Call service.
- Return response.

Services

- Business rules.
- Workflows.
- Domain logic.

Repositories

- Database access only.

Validators

- Validation only.

Middleware

- Authentication.
- Authorization.

---

# Business Logic

Business logic belongs only in services.

Never place business logic inside:

- Controllers
- Routes
- Middleware
- Repositories

---

# Database Access

Use Prisma ORM.

Repositories own all database operations.

Avoid duplicated queries.

Never scatter Prisma usage throughout the project.

---

# Validation

Validation must be centralized.

Avoid duplicated validation rules.

Use shared validators whenever possible.

---

# Permissions

Authorization is permission-based.

Never hardcode roles.

Avoid:

if (role === "STORE_OWNER")

Prefer:

Permission
↓
Role
↓
User

Permissions determine access.

Roles group permissions.

Users receive permissions through roles.

---

# Context

Always evaluate permissions within the active context.

Examples:

Platform Context

Store Context

Public Context

A user's permissions may differ between contexts.

---

# Multi-Tenancy

Tenant isolation is mandatory.

Every tenant-owned query must be scoped by:

store_id

Only SUPER_ADMIN may bypass tenant isolation.

---

# API Contracts

Do not change:

- Request format
- Response format
- Status codes

unless approved by architecture.

---

# Response Format

Successful responses:

{
"success": true,
"message": "...",
"data": {}
}

Error responses:

{
"success": false,
"message": "...",
"errors": []
}

Use the project's shared response helpers.

---

# Error Handling

Use centralized error handling.

Do not duplicate error responses.

Avoid exposing internal implementation details.

---

# Comments

Comments explain WHY.

Not WHAT.

Good:

// Store ownership is verified before updating settings.

Bad:

// Increment counter.

---

# Naming

Use meaningful names.

Avoid abbreviations unless widely understood.

Prefer explicit names over short names.

---

# Documentation

Documentation is part of implementation.

If an approved architectural change affects documentation:

Update documentation.

If it isn't documented, it isn't officially part of the architecture.

---

# Testing

Every implementation must include:

- Automated tests (when applicable)
- Manual verification
- Expected results

Never claim tests that were not executed.

---

# Reporting

Every implementation must provide:

- Summary
- Scope completed
- Files changed
- Database changes
- API changes
- Documentation updates
- Tests executed
- Manual verification
- Known limitations
- Suggested next step

Follow the Standard Reporting Format.

---

# Non-Goals

Do not:

- Refactor unrelated modules.
- Change project structure.
- Implement future features.
- Add unnecessary dependencies.
- Optimize prematurely.
- Invent APIs.
- Invent database fields.
- Invent permissions.

---

# Final Rule

Implement exactly what has been approved.

Nothing more.

Nothing less.
