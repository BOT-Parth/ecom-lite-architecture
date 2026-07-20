# Core Business Workflows

This document explicitly defines the step-by-step business workflows implemented in the platform.

## Store Request & Approval Workflow

This is the primary workflow for onboarding a new store tenant onto the platform. It involves a transition from a user-submitted request to a fully operational store with associated permissions.

### 1. Request Submission
1. An authenticated user submits a payload containing `name`, `slug`, `description`, and `avatarUrl` to `POST /store-requests`.
2. The service checks the database for any existing approved store with the same `slug` to prevent naming collisions upfront.
3. A new `StoreRequest` entity is created with the status `PENDING`.

### 2. Administrative Review
1. A platform administrator (user with the `APPROVE_STORE` platform permission) fetches the pending requests via `GET /store-requests`.
2. The administrator can either Reject or Approve the request.
3. If Rejected, the `StoreRequest` status is updated to `REJECTED` and the workflow terminates.

### 3. Approval Transaction (The Store Creation)
When a platform administrator approves a `PENDING` request, the repository layer executes a strict ACID transaction (`prisma.$transaction`) with the following steps:
1. **Verification**: Verify the request exists and is still `PENDING`.
2. **Collision Check**: Perform a final check to ensure no `Store` currently exists with the requested `slug`.
3. **Store Creation**: Create the actual `Store` entity. The `name`, `slug`, `description`, and `avatarUrl` are mapped over from the request. The store's `approvalStatus` is set to `APPROVED` and `operationalStatus` to `OPEN`.
4. **Request Resolution**: Update the original `StoreRequest` status to `APPROVED` and link it to the newly created `Store` ID.
5. **Membership Assignment**: The system looks up the system `STORE_OWNER` role ID and creates a `UserStoreMembership` record linking the user who submitted the request to the newly created store with the `STORE_OWNER` role. This instantly grants them full management permissions over the store.

---
*Last verified against code on 2026-07-19: Verified workflow logic and transactional steps in `store-request.service.js` and `store.repository.js`.*
