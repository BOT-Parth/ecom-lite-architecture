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

---

## ADR-008

Title

Unified Login

Decision

All platform users (SUPER_ADMIN) and tenants use a single unified authentication flow and page.

Reason

Avoids duplicate login systems and simplifies authentication routes.

---

## ADR-009

Title

Backend as Authorization Source

Decision

The backend is the sole source of truth for authorization. Both Login and Profile endpoints return the same authorization context. The frontend must never infer access rights from email addresses, usernames, IDs, or endpoint probing.

Reason

Prevents client-side authorization bypasses and secures access control logic.

---

## ADR-010

Title

Platform Store Directory

Decision

A dedicated platform store directory is provided via endpoint `GET /stores/platform` for platform administrators (SUPER_ADMIN) to browse every store on the platform (including all operationalStatus and approvalStatus states).

Reason

Enables administrative oversight and store request processing.

---

## ADR-011

Title

Public Storefront Discovery

Decision

The public storefront `GET /stores` is the only discovery mechanism for regular users. Platform-wide store directories are inaccessible to non-admin users.

Reason

Maintains multi-tenant boundary and shields internal platform directories from public view.

---

## ADR-012

Title

SUPER_ADMIN Store Access

Decision

Platform administrators (SUPER_ADMIN) can enter any store context. Within that store's context, the SUPER_ADMIN is granted the same capabilities as a Store Admin (Owner), enforced strictly at the backend level.

Reason

Allows administrators to assist tenants and manage catalog, inventory, and settings.

---

## ADR-013

Title

Store Settings Replacing Request Store

Decision

Once a merchant's store request is approved, the Request Store view converts into a Store Settings dashboard where Store Admins manage name, description, and avatar metadata.

Reason

Transitions a completed registration flow into an active management flow.

---

## ADR-014

Title

Store Registration Expansion

Decision

Store requests will expand to include Store Name, Slug, Description, and Avatar. In the UI, image upload buttons will exist as placeholders until file upload capability is implemented.

Reason

Captures necessary merchant profile details during request submission.

---

## ADR-015

Title

Store Deletion Workflow

Decision

Store Admins can request deletion via Store Settings. Deletion is executed as a backend soft-delete. Store Recovery will be implemented in a future milestone.

Reason

Prevents accidental data loss while allowing merchants to request store deactivation.

---

## ADR-016

Title

Theme Support

Decision

The frontend will support Light Mode and Dark Mode with theme preference persistence.

Reason

Improves accessibility and user experience.

---

## ADR-017

Title

Store Metadata Lifecycle and Copy Flow

Decision

Store registration metadata (name, slug, description, and avatar reference) is initially recorded on StoreRequest. Upon approval, these fields are copied into the newly created Store record. The Store record serves as the persistent long-term source of truth for settings and ongoing metadata updates.

Reason

Keeps registration workflows separate from active store management, while preserving request history.


---
*Last verified against code on 2026-07-19: Verified architectural principles against current codebase.*