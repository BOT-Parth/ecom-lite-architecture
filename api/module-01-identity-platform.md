# Module 1 — Identity & Platform API Contracts

This module documents the **Identity & Platform** endpoints that are part of the MVP.

---

## Authentication Endpoints

### `POST /auth/register`
* **Purpose**: Register a new platform user.
* **Authentication**: None (public).
* **Authorization**: Public.
* **Request Body**:
```json
{
  "email": "user@example.com",
  "username": "user123",
  "password": "StrongPass!23"
}
```
* **Validation**:
  - `email`: must be a valid email and unique.
  - `username`: 3‑50 characters.
  - `password`: minimum 6 characters.
* **Response (201 Created)**:
```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "user": {
      "id": "<uuid>",
      "email": "user@example.com",
      "username": "user123",
      "createdAt": "<timestamp>"
    }
  }
}
```

---

### `POST /auth/login`
* **Purpose**: Authenticate a user and issue a JWT. Returns the full user authorization context in the same response, so the frontend has immediate role information without a separate follow-up request.
* **Authentication**: None (public).
* **Authorization**: Public.
* **Request Body**:
```json
{
  "email": "user@example.com",
  "password": "StrongPass!23"
}
```
* **Response (200 OK)**:
```json
{
  "success": true,
  "message": "User logged in successfully",
  "data": {
    "token": "<jwt>",
    "user": {
      "id": "<uuid>",
      "email": "user@example.com",
      "username": "user123",
      "createdAt": "<timestamp>",
      "updatedAt": "<timestamp>",
      "authorization": {
        "platformRole": null,
        "storeMemberships": []
      }
    }
  }
}
```
* **JWT Payload**: `{ "userId": "<user uuid>" }` — authorization data is never embedded in the token.

---

### `GET /auth/profile`
* **Purpose**: Retrieve the authenticated user's profile **and authorization context**. Returns the same user shape as `POST /auth/login`. Clients use this to refresh the authorization context (e.g. after store approval).
* **Authentication**: Required (Bearer JWT).
* **Authorization**: Any authenticated user (own profile only).
* **Response (200 OK)**:
```json
{
  "success": true,
  "message": "User profile retrieved successfully",
  "data": {
    "user": {
      "id": "<uuid>",
      "email": "user@example.com",
      "username": "user123",
      "createdAt": "<timestamp>",
      "updatedAt": "<timestamp>",
      "authorization": {
        "platformRole": "SUPER_ADMIN",
        "storeMemberships": [
          {
            "storeId": "<store-uuid>",
            "storeName": "My Electronics Store",
            "storeSlug": "my-electronics-store",
            "roleName": "STORE_OWNER"
          }
        ]
      }
    }
  }
}
```

#### `authorization` Object — returned by both `POST /auth/login` and `GET /auth/profile`

| Field | Type | Description |
|---|---|---|
| `platformRole` | `string \| null` | Platform-level role name (e.g. `SUPER_ADMIN`). `null` when the user has no platform role. |
| `storeMemberships` | `array` | Stores the user belongs to. `[]` when the user has no memberships. |
| `storeMemberships[].storeId` | `string` | UUID of the store. |
| `storeMemberships[].storeName` | `string` | Display name of the store. |
| `storeMemberships[].storeSlug` | `string` | URL slug of the store. |
| `storeMemberships[].roleName` | `string` | User's role in this store (e.g. `STORE_OWNER`, `STORE_STAFF`). |

#### Client Usage Rules

- Use `authorization.platformRole === 'SUPER_ADMIN'` to show/hide platform admin UI.
- Use `authorization.storeMemberships` to determine which stores the user manages.
- **Never** check `email`, `username`, or hardcoded values for authorization decisions.
- **Never** re-enforce permissions on the frontend. All permission enforcement is handled exclusively by backend RBAC middleware.

---

## Platform Admin Endpoints

### `GET /store-requests`
* **Purpose**: List all store creation requests.
* **Authentication**: Required (Bearer JWT).
* **Authorization**: Requires platform permission `APPROVE_STORE` (SUPER_ADMIN role).
* **Response (200 OK)**:
```json
{
  "success": true,
  "message": "Store creation requests retrieved successfully",
  "data": {
    "requests": [
      {
        "id": "<uuid>",
        "name": "My Electronics Store",
        "slug": "my-electronics-store",
        "status": "PENDING",
        "userId": "<uuid>",
        "storeId": null,
        "createdAt": "<timestamp>",
        "updatedAt": "<timestamp>"
      }
    ]
  }
}
```

### `POST /store-requests`
* **Purpose**: Submit a request to create a new store. Any authenticated user may submit.
* **Authentication**: Required (Bearer JWT).
* **Authorization**: Any authenticated user.
* **Request Body**:
```json
{
  "name": "My Electronics Store",
  "slug": "my-electronics-store",
  "description": "Premium electronics store with international shipping.",
  "avatarUrl": "https://example.com/images/electronics-avatar.png"
}
```
* **Validation**:
  - `name`: 3–100 characters.
  - `slug`: 3–50 characters, lowercase alphanumeric + hyphens only (`^[a-z0-9-]+$`).
  - `description`: optional string, maximum 500 characters.
  - `avatarUrl`: optional string, maximum 2048 characters.
* **Response (201 Created)**:
```json
{
  "success": true,
  "message": "Store creation request submitted successfully",
  "data": {
    "request": {
      "id": "<uuid>",
      "name": "My Electronics Store",
      "slug": "my-electronics-store",
      "description": "Premium electronics store with international shipping.",
      "avatarUrl": "https://example.com/images/electronics-avatar.png",
      "status": "PENDING",
      "userId": "<uuid>",
      "storeId": null,
      "createdAt": "<timestamp>",
      "updatedAt": "<timestamp>"
    }
  }
}
```

### `PATCH /store-requests/:id/approve`
* **Purpose**: Approve a pending store request. Creates the store and assigns STORE_OWNER membership to the requester in a single atomic transaction.
* **Authentication**: Required (Bearer JWT).
* **Authorization**: Requires platform permission `APPROVE_STORE` (SUPER_ADMIN role).
* **Response (200 OK)**:
```json
{
  "success": true,
  "message": "Store request approved and store created successfully",
  "data": {
    "store": { 
      "id": "<uuid>", 
      "name": "My Electronics Store", 
      "slug": "my-electronics-store", 
      "description": "Premium electronics store with international shipping.",
      "avatarUrl": "https://example.com/images/electronics-avatar.png",
      "approvalStatus": "APPROVED", 
      "operationalStatus": "OPEN",
      "createdAt": "<timestamp>",
      "updatedAt": "<timestamp>"
    },
    "request": { "id": "<uuid>", "status": "APPROVED", "storeId": "<uuid>" },
    "membership": { "userId": "<uuid>", "storeId": "<uuid>", "roleId": "<uuid>" }
  }
}
```

### `PATCH /store-requests/:id/reject`
* **Purpose**: Reject a pending store request.
* **Authentication**: Required (Bearer JWT).
* **Authorization**: Requires platform permission `APPROVE_STORE` (SUPER_ADMIN role).
* **Response (200 OK)**:
```json
{
  "success": true,
  "message": "Store request rejected successfully",
  "data": {
    "request": { "id": "<uuid>", "status": "REJECTED" }
  }
}
```

### `GET /stores/platform`
* **Purpose**: List all stores on the platform (Platform Store Directory).
* **Authentication**: Required (Bearer JWT).
* **Authorization**: Requires platform permission `APPROVE_STORE` (SUPER_ADMIN role).
* **Response (200 OK)**:
```json
{
  "success": true,
  "message": "Platform stores directory retrieved successfully",
  "data": {
    "stores": [
      {
        "id": "<uuid>",
        "name": "My Electronics Store",
        "slug": "my-electronics-store",
        "description": "Premium electronics store with international shipping.",
        "avatarUrl": "https://example.com/images/electronics-avatar.png",
        "approvalStatus": "APPROVED",
        "operationalStatus": "OPEN",
        "createdAt": "<timestamp>",
        "updatedAt": "<timestamp>"
      }
    ]
  }
}
```

---

## Business Rules & Validation
* Passwords are hashed with **bcrypt** (cost factor 10) before storage.
* Emails are unique (`@@unique([email])`).
* JWTs expire after `process.env.JWT_EXPIRES_IN`.
* The authentication middleware extracts `userId` from the token and attaches `req.user = { userId }` for downstream use.
* Store approval is an atomic transaction: store creation, request status update, and STORE_OWNER membership assignment all succeed or all roll back together.

---

## Layer Responsibilities
| Layer | File(s) | Responsibility |
|------|----------|----------------|
| **Routes** | `src/routes/auth.routes.js` | Declare endpoints, attach validation and auth middleware. |
| **Validators** | `src/validators/auth.validator.js` | Zod schemas for register & login payloads. |
| **Controllers** | `src/controllers/auth.controller.js` | Translate HTTP request to service calls, format responses. |
| **Services** | `src/services/auth.service.js` | Business logic: password hashing, credential verification, JWT generation, authorization context assembly. |
| **Repositories** | `src/repositories/user.repository.js` | Prisma queries for `User`. `findById` for basic lookups. `findProfileContext` for the enriched authenticated profile including platform role and store memberships. |
| **Middleware** | `src/middleware/auth.middleware.js` | JWT verification, attach `req.user`. |

---

## Verification Status
* **Unit Tests** (`test-auth.js`) cover registration, duplicate email, successful login, invalid credentials, profile access (with/without token), and `authorization` context fields in profile response.
* **Integration Tests** run the full Express stack — all pass.
* **Prisma validation** (`npx prisma validate`) succeeds.

---

**All endpoints are functional and documented.**
