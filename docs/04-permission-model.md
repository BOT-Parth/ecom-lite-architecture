# Permission Model & RBAC

This document outlines the factual implementation of Role-Based Access Control (RBAC) currently present in the codebase.

## Core Concepts

The platform separates authorization into two distinct boundaries:
1. **Platform Permissions**: Governs global, platform-wide administrative actions.
2. **Store Permissions**: Governs tenant-specific actions inside a single store.

Permissions are granted to Roles. Users are granted Roles. All authorization in the codebase checks for *permissions*, not *roles* directly (to prevent hardcoding role logic into routes).

---

## 1. Platform Permissions

Platform permissions are defined in `src/permissions/platform.permissions.js`.
They are checked when users attempt to perform administrative operations across the entire E-Com Lite platform.

**Current Permissions:**
- `APPROVE_STORE`: Allows the user to view, approve, and reject incoming store requests.

**Associated Roles:**
Currently, this permission is granted to the `SUPER_ADMIN` platform role.

---

## 2. Store Permissions

Store permissions are defined in `src/permissions/store.permissions.js`.
They are checked when a user attempts to mutate data within a specific store tenant (`storeId`). A user's access is determined by their `UserStoreMembership` in that specific store.

**Current Permissions:**
- `MANAGE_STORE`: Allows modifying store-level settings (name, description, avatar).
- `MANAGE_PRODUCTS`: Allows creating, updating, and deleting categories and products, as well as managing inventory stock.
- `MANAGE_ORDERS`: Allows managing customer orders (view, list, update status).

**Associated Roles:**
These permissions are mapped to store roles (e.g., `STORE_OWNER`, `STORE_MANAGER`, `STORE_STAFF`) via the database seed during tenant creation.

---

## 3. Middleware Enforcement

Permissions are strictly enforced at the routing layer using `src/middleware/rbac.middleware.js`:
- `requirePlatformPermission(permission)`: Verifies the authenticated user holds a platform role mapping to the required permission.
- `requireStorePermission(permission)`: Verifies the authenticated user holds a membership in `req.params.storeId` with a role that maps to the required permission.

*Note: For the philosophical reasons behind why roles are decoupled from permissions, or why tenant scoping is path-based, see the planned but unwritten `02-business-rules.md` and `12-project-philosophy.md` documents.*

---
*Last verified against code on 2026-07-19: Verified permission constants in `src/permissions/*`, middleware in `src/middleware/rbac.middleware.js`, and usage in routes.*
