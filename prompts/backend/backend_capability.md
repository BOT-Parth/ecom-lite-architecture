Backend Capability Report
This document serves as the source of truth for the frontend team regarding the current state of the backend capabilities.

1. Platform Authentication
   Current REST Endpoints:
   POST /auth/register
   POST /auth/login
   GET /auth/profile
   Request Body:
   register: { email (string), username (string 3-50 chars), password (string min 6) }
   login: { email (string), password (string min 1) }
   Response Body:
   register/login: { success, message, data: { token, user: { id, email, username } } }
   profile: { success, message, data: { user: { id, email, username, memberships, platformRoles } } }
   Validation Rules: Valid email format. Passwords hashed with bcrypt.
   RBAC: GET /auth/profile requires a valid JWT token.
   Business Rules: Emails must be unique. The backend issues a single JWT token for all platform roles. Platform JWTs cannot be used for customer authentication.
   Unsupported fields: None.
   MVP Limitations: Password reset flows and email verification are not implemented.
2. Store Requests (Platform Administration)
   Current REST Endpoints:
   POST /store-requests
   GET /store-requests
   PATCH /store-requests/:id/approve
   PATCH /store-requests/:id/reject
   Request Body:
   POST: { name (string 3-100), slug (string 3-50, lowercase alphanumeric hyphens), description (string max 500, optional), avatarUrl (string max 2048, optional) }
   Response Body:
   Standard { success, message, data: { request: { ... } } } format.
   Validation Rules: Slug must match the slug regex pattern.
   RBAC:
   POST requires any authenticated user.
   GET, PATCH /approve, PATCH /reject strictly require the APPROVE_STORE platform permission (e.g., SUPER_ADMIN).
   Business Rules: Naming collisions on slug are prevented upfront. Approving a request runs a transaction that automatically creates the Store and assigns the STORE_OWNER membership to the requester.
   Intentional MVP Limitations: Rejections cannot currently include a "reason" field.
3. Stores & Settings
   Current REST Endpoints:
   GET /stores/my (List authenticated user's stores)
   GET /stores/platform (List all stores - Platform Admin)
   GET /stores/:storeId/settings
   PATCH /stores/:storeId/settings
   Request Body:
   PATCH /settings: { name, description, avatarUrl } (All optional, same length rules as Store Requests).
   Response Body:
   Standard lists of stores or single store objects.
   RBAC:
   GET /my requires authentication.
   GET /platform requires APPROVE_STORE platform permission.
   Settings endpoints require the MANAGE_STORE store permission for that specific storeId.
   Unsupported fields: Updating the slug after creation is not supported.
4. Public Storefront & Customer Accounts
   Current REST Endpoints:
   GET /stores (List public stores)
   GET /stores/:storeId/categories
   GET /stores/:storeId/products
   GET /stores/:storeId/products/:id
   POST /stores/:storeId/customers/register
   POST /stores/:storeId/customers/login
   POST /stores/:storeId/customers/logout
   GET /stores/:storeId/customers/me
   GET /stores/:storeId/customers/me/orders
   GET /stores/:storeId/customers/me/orders/:id
   Request Body:
   register: { firstName, lastName, email, phone, password }
   login: { email, password }
   Validation Rules: 
   Customers: email must be unique per storeId. 
   RBAC: 
   GET /stores, GET /categories, GET /products are fully public. 
   Customer register/login are public.
   Customer /me and /orders require a valid Customer JWT for the specific storeId.
   Business Rules: 
   GET /stores only returns stores where approvalStatus is APPROVED and operationalStatus is OPEN.
   Customers are strictly scoped to a single store. Customer JWTs are completely distinct from Platform JWTs.
5. Categories (Catalog)
   Current REST Endpoints:
   POST /stores/:storeId/categories
   PATCH /stores/:storeId/categories/:id
   DELETE /stores/:storeId/categories/:id
   Request Body:
   POST: { name (string 3-50), slug (string 3-50) }
   PATCH: Optional fields above.
   Validation Rules: Unique slug per store context.
   RBAC: POST, PATCH, DELETE require MANAGE_PRODUCTS store permission for the specified storeId.
   Business Rules: Deleting a category cascades by setting category_id = NULL on associated products (it does not delete the products).
6. Products & Inventory
   Current REST Endpoints:
   POST /stores/:storeId/products
   PATCH /stores/:storeId/products/:id
   DELETE /stores/:storeId/products/:id
   GET /stores/:storeId/products/:productId/inventory
   PATCH /stores/:storeId/products/:productId/inventory
   Request Body:
   Product POST: { name (3-100), description (max 500), price (positive float), imageUrls (array of valid URLs, optional), categoryId (uuid, optional) }
   Inventory PATCH: { quantity (non-negative integer) }
   Validation Rules: Price must be positive. Quantity must be >= 0 and integer. Image URLs must be valid format.
   RBAC: Modifying products or inventory requires the MANAGE_PRODUCTS store permission.
   Business Rules: Creating a product automatically creates a 1:1 linked inventory record with quantity: 0. Deleting a product cascades to delete its inventory record.
   Future Placeholders visible in code: imageUrls is just a string array; there is no backend multipart file upload handler yet.
7. Orders
   Current REST Endpoints:
   POST /stores/:storeId/orders
   GET /stores/:storeId/orders
   GET /stores/:storeId/orders/:id
   PATCH /stores/:storeId/orders/:id/status
   Request Body:
   POST /orders: { deliveryAddress (string 1-500), items (array of { productId: uuid, quantity: positive integer }, min length 1) }
   PATCH /status: { status (enum: PLACED, PROCESSING, READY, COMPLETED, CANCELLED) }
   Response Body:
   POST /orders: { success, message, data: { order: { id, orderNumber, storeId, customerId, customerName, customerEmail, customerPhone, deliveryAddress, totalAmount, status, createdAt, updatedAt, items: [{ id, productId, productName, quantity, unitPrice }] } } }
   GET /orders: { success, data: { orders: [...] } }
   GET /:id: { success, data: { order: {...} } }
   PATCH /status: { success, message, data: { order: {...} } }
   Validation Rules: Items array min 1. Product availability checked. Customer identity automatically verified via token.
   RBAC: 
   POST /orders requires a valid Customer JWT.
   GET /orders, GET /:id, PATCH /status require MANAGE_ORDERS store permission for the specified storeId (Platform User).
   Business Rules: 
   - Authenticated Checkout: Customer login is required to place orders.
   - PII Snapshots: Customer data is captured from their profile at order creation.
   - Simulated Payment: Total amount is calculated backend-side from snapshotted prices.
   - Inventory: Atomic transaction for validation, creation, and stock deduction. Whole order rejected if any item has insufficient stock.
   - Restoration: Cancelling an order restores inventory automatically.
   - Merging: Duplicate products in the request are safely merged.
   - Status Lifecycle: Forward flow (PLACED -> PROCESSING -> READY -> COMPLETED). Terminal states (COMPLETED, CANCELLED).
   - Historical Orders: Legacy guest orders without customerIds remain untouched.
8. Health
   Current REST Endpoints: GET /health
   Response Body: { "status": "ok", "message": "up and running" }
   RBAC: Public. Used for EC2 ALB target group health checks.
   Backend Features Status for Frontend Implementation
8. Backend features fully implemented but NOT consumed by the frontend: (Note: As the exact frontend state is unknown to the backend audit without inspecting frontend code, it is assumed the following recent milestones are unconsumed)

Store Requests (creation and admin approval)
Store Settings modifications
Categories & Products CRUD
Inventory patching
Orders (checkout and management)

2. Backend features partially consumed:
Authentication (login/register likely consumed, but JWT handling for tenant scoped requests /stores/:storeId/* requires frontend implementation). 

3. Backend features intentionally deferred:
Cart Persistence, Payment Gateway, Shipping, Reviews, Wishlist are intentionally deferred.
Media Uploads: avatarUrl and imageUrls expect fully qualified URLs, not binary file uploads. CDN integration is deferred.
User Roles via UI: Currently, SUPER_ADMIN is only created via prisma/seed.js and there is no UI route to promote a standard user to super admin. 

4. Backend breaking changes since the previous frontend implementation:
Environment Variables: The backend now strictly validates DATABASE_URL, JWT_SECRET, and DB_SSL at startup via src/config/env.js.
The /health endpoint logic was refactored into a HealthController, though the signature remains unchanged.
