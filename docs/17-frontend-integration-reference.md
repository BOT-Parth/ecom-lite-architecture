# Frontend Integration Reference

This document serves as the absolute source of truth for the E-Com Lite backend API. It strictly documents only the currently implemented backend capabilities. 

## 1. Overview

E-Com Lite is divided into two distinct API domains with separate identity boundaries.

- **Platform APIs**: Used by merchants and super-admins to manage stores, catalog, inventory, and orders.
- **Storefront APIs**: Used by shoppers (customers) to browse the public catalog, register, and place orders.
- **Authentication Boundaries**: The Platform and Storefront use entirely separate identities. A Platform user cannot use their token to act as a Customer, and vice versa.
- **Store-scoped APIs**: Most operational APIs require a `storeId` in the path (e.g., `/stores/:storeId/products`) to enforce tenant isolation.
- **Customer-scoped APIs**: A customer account is strictly scoped to a single `storeId`.

## 2. Authentication

The backend issues two distinct JWTs:

- **Platform JWT**: 
  - Acquired via `POST /auth/login`.
  - Sent via `Authorization: Bearer <token>`.
  - Valid for `GET /auth/profile`, `/store-requests`, and merchant actions (e.g., `/stores/:storeId/products`).
- **Customer JWT**: 
  - Acquired via `POST /stores/:storeId/customers/login`.
  - Sent via `Authorization: Bearer <token>`.
  - Signed with `CUSTOMER_JWT_SECRET`. Contains `{ type: "customer", storeId, customerId }`.
  - Required for checkout and customer order history.

**Token Ownership & Logout:**
- The frontend must never mix tokens.
- **Logout Expectations**: When a token expires (401), the frontend must globally clear the respective session (Platform or Customer). There are no refresh tokens.

## 3. Store Resolution

- **Slug Resolution**: The public storefront resolves a human-readable slug to a UUID via `GET /stores/public/:slug`.
- **storeId Usage**: The resolved UUID (`storeId`) must be injected into all subsequent Storefront API calls (e.g., `/stores/:storeId/products`). 
- **Error Cases**: If the slug is invalid, the backend returns 404. The frontend must display a "Store Not Found" view.

## 4. Customer Flow

- **Register**: `POST /stores/:storeId/customers/register` - Requires `{ firstName, lastName, email, phone, password }`. Returns Customer JWT.
- **Login**: `POST /stores/:storeId/customers/login` - Requires `{ email, password }`. Returns Customer JWT.
- **Hydration**: `GET /stores/:storeId/customers/profile` - Fetches the customer profile (requires Customer JWT).
- **Logout**: `POST /stores/:storeId/customers/logout` - Invalidates customer session (requires Customer JWT).
- **Checkout**: `POST /stores/:storeId/orders` - Places an order (requires Customer JWT).
- **Orders**: `GET /stores/:storeId/customers/orders` - Lists order history (requires Customer JWT).
- **Order Details**: `GET /stores/:storeId/customers/orders/:id` - Shows specific order details (requires Customer JWT).

## 5. Product APIs

- `GET /stores/:storeId/products`
  - **Purpose**: List all products for a store.
  - **Auth**: Public.
- `GET /stores/:storeId/products/:id`
  - **Purpose**: Get product details.
  - **Auth**: Public.
- `POST /stores/:storeId/products`
  - **Purpose**: Create a new product.
  - **Auth**: Platform JWT + `MANAGE_PRODUCTS`.
  - **Payload**: `{ name, description, price, imageUrls, categoryId }`.
- `PATCH /stores/:storeId/products/:id`
  - **Purpose**: Update a product.
  - **Auth**: Platform JWT + `MANAGE_PRODUCTS`.
- `DELETE /stores/:storeId/products/:id`
  - **Purpose**: Delete a product.
  - **Auth**: Platform JWT + `MANAGE_PRODUCTS`.

## 6. Category APIs

- `GET /stores/:storeId/categories`
  - **Purpose**: List all categories.
  - **Auth**: Public.
- `POST /stores/:storeId/categories`
  - **Purpose**: Create a category.
  - **Auth**: Platform JWT + `MANAGE_CATEGORIES`.
  - **Payload**: `{ name, slug }`.
- `PATCH /stores/:storeId/categories/:id`
  - **Purpose**: Update a category.
  - **Auth**: Platform JWT + `MANAGE_CATEGORIES`.
- `DELETE /stores/:storeId/categories/:id`
  - **Purpose**: Delete a category.
  - **Auth**: Platform JWT + `MANAGE_CATEGORIES`.

## 7. Inventory APIs

- `GET /stores/:storeId/products/:productId/inventory`
  - **Purpose**: Read absolute stock level.
  - **Auth**: Platform JWT + `MANAGE_INVENTORY`.
- `PATCH /stores/:storeId/products/:productId/inventory`
  - **Purpose**: Update absolute stock level (does not increment/decrement).
  - **Auth**: Platform JWT + `MANAGE_INVENTORY`.
  - **Payload**: `{ quantity: integer }`.

**Inventory Behavior:**
- The Backend is the sole authority on inventory.
- **Stock Validation**: During checkout, the backend natively protects against overselling. 
- **Out Of Stock**: If an order exceeds available stock, the backend rejects it with a `400 Bad Request` and returns an exact breakdown in `errors.details.items`.
- Inventory is atomically deducted upon order placement and atomically restored upon cancellation.

## 8. Order APIs

**Merchant APIs:**
- `GET /stores/:storeId/orders`
  - **Purpose**: List store orders.
  - **Auth**: Platform JWT + `MANAGE_ORDERS`.
- `GET /stores/:storeId/orders/:id`
  - **Purpose**: Get order details.
  - **Auth**: Platform JWT + `MANAGE_ORDERS`.
- `PATCH /stores/:storeId/orders/:id/status`
  - **Purpose**: Transition order state.
  - **Auth**: Platform JWT + `MANAGE_ORDERS`.
  - **Payload**: `{ status: enum }`.
  - **Status Fields**: `PLACED`, `PROCESSING`, `READY`, `COMPLETED`, `CANCELLED`.

**Customer APIs:**
- `POST /stores/:storeId/orders` (Checkout)
  - **Auth**: Customer JWT.
  - **Payload**: `{ deliveryAddress, items: [{ productId, quantity }] }`. (Note: PII is snapshotted from the JWT).

## 9. Store Requests

- `POST /store-requests`
  - **Auth**: Platform JWT.
  - **Purpose**: Submit a new store for approval.
- `GET /store-requests`
  - **Auth**: Platform JWT + `APPROVE_STORE`.
  - **Purpose**: List all requests (Super Admin).
- `PATCH /store-requests/:id/approve`
  - **Auth**: Platform JWT + `APPROVE_STORE`.
  - **Purpose**: Approve a request.
- `PATCH /store-requests/:id/reject`
  - **Auth**: Platform JWT + `APPROVE_STORE`.
  - **Purpose**: Reject a request.

*(Note: Pending and Rejected states are the only states you can act upon).*

## 10. Store Members

- **Backend capability not implemented.** Reserved for future milestone.

## 11. Platform Analytics

- **Backend capability not implemented.** Reserved for future milestone.

## 12. Error Responses

All errors use a unified envelope: `{ success: false, message: string, errors: [] }`.

- `400 Bad Request`: Validation failure (e.g., Zod validation fail, Out of stock).
- `401 Unauthorized`: Token missing, invalid, or expired.
- `403 Forbidden`: Valid token, but lacks RBAC permission (e.g., "requires APPROVE_STORE").
- `404 Not Found`: Entity does not exist.
- `409 Conflict`: Business rule violation (e.g., "User already owns a store", "Store slug already taken").
- `500 Internal Server Error`: Unhandled backend exception.

## 13. Frontend Rules

- **Pages never call repositories**: The frontend only communicates with backend HTTP endpoints.
- **Customer APIs use customer token**: Strict isolation must be maintained.
- **Platform APIs use platform token**: Strict isolation must be maintained.
- **Backend is source of truth**: The frontend must trust backend HTTP status codes (e.g., 401 triggers logout, 400 displays validation errors).
- **Frontend validation improves UX only**: The backend always performs the final, absolute validation on all payloads and permissions.

## 14. Deprecated Capabilities

The following capabilities have been permanently removed from the backend architecture:

- **Guest Checkout**: ❌ Deprecated / Do Not Use / No longer supported.
- **Guest Order Tracking**: ❌ Deprecated / Do Not Use / No longer supported. (`POST /stores/:storeId/orders/track` removed).
- **Public Order Lookup**: ❌ Deprecated / Do Not Use / No longer supported.

## 15. Capability Matrix

| Capability | Backend Status | Frontend Required |
| :--- | :--- | :--- |
| Customer Registration | ✅ | Yes |
| Customer Login | ✅ | Yes |
| Customer Profile / Hydration | ✅ | Yes |
| Customer Checkout | ✅ | Yes |
| Customer Orders | ✅ | Yes |
| Guest Checkout | ❌ Deprecated | Remove |
| Guest Tracking | ❌ Deprecated | Remove |
| Store Members | ❌ Not Implemented | Coming Soon |
| Platform Analytics | ❌ Not Implemented | Coming Soon |
