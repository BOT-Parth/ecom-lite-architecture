# Architecture Decision Log

## ADR-001

Title

Documentation is the Source of Truth

Decision

Architecture documentation takes precedence over implementation.

Reason

Implementation can drift over time.

Documentation preserves the intended architecture.

---

## ADR-002

Title

Platform and Store Are Separate Domains

Decision

Platform responsibilities and Store responsibilities are completely separated.

Reason

Improves maintainability.

Supports multi-tenancy.

Reduces coupling.

---

## ADR-003

Title

Permission-Based Authorization

Decision

Permissions determine access.

Roles group permissions.

Users receive permissions through roles.

Reason

Supports future custom roles.

Avoids hardcoded authorization.

---

## ADR-004

Title

Context-Based Authorization

Decision

Permissions are evaluated within the active context.

Reason

The same user may simultaneously be:

- SUPER_ADMIN
- STORE_OWNER
- STORE_STAFF
- CUSTOMER

Authorization depends on the active context.

---

## ADR-005

Title

One Concern → One Home

Decision

Every responsibility has exactly one official location.

Reason

Reduces duplicated logic.

Improves readability.

Improves maintainability.

---

## ADR-006

Title

One Membership Per User Per Store

Decision

Memberships are updated rather than duplicated.

Reason

Simplifies authorization.

Improves data integrity.

---

## ADR-007

Title

Tenant Isolation

Decision

Every tenant-owned query must be scoped by store_id.

Reason

Prevents data leakage between stores.

Guarantees tenant isolation.
