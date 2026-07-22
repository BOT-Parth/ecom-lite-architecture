Backend → Frontend Technical Handoff Report

1. Executive Summary
   This report serves as the authoritative handoff for the Customer Accounts milestone.

The backend has completed the transition from a purely public "guest checkout" model to a fully authenticated tenant-scoped customer model. Customers can now register, securely log in to specific stores, maintain a profile, place authenticated orders, and view their historical orders.

The previous iteration allowed arbitrary tracking of orders via email and phone numbers without authentication; this feature has been entirely eliminated in favor of a secure, identity-driven checkout flow. This document details the architectural boundaries, business rules, API contracts, and exact expectations the frontend must fulfill to integrate these new capabilities.

2. Current Identity Architecture
   The system now enforces a strict, impenetrable boundary between the people who run the platform (Platform Users) and the people who shop on the platform (Customers).

Platform Users

What they are: Super Admins, Store Owners, Store Managers, and Store Staff. They manage the system and store operations.
Where they login: POST /auth/login.
JWT used: Signed with JWT_SECRET. Contains a userId.
Permissions: Governed by robust RBAC (Role-Based Access Control) across platform and specific store boundaries.
Customers

What they are: The end-users visiting the public storefront to browse products and place orders.
How they differ: Customers have no roles, no platform access, and cannot log into the admin dashboard.
Tenant scope: A Customer identity belongs exclusively to a single store. There is no concept of a "Global Platform Customer". A user shopping at Store A must create a separate account to shop at Store B.
Authentication model: Simple email/password.
JWT used: Signed with CUSTOMER_JWT_SECRET. Contains customerId, storeId, and type: 'customer'.
Store isolation: The backend mathematically prevents a token issued for Store A from authenticating against endpoints in Store B.
Why they are completely different: Attempting to merge these systems would inevitably lead to privilege escalation vulnerabilities (e.g., a customer accidentally gaining admin rights) and would break the multi-tenant architecture, which dictates that tenant A's customers are completely invisible to tenant B.

3. Database Model
   The backend relies on the following schema architecture:

Store: The root tenant entity. Every public-facing interaction is scoped under a Store.
Customer: A registered shopper. Scoped to a specific Store. Contains firstName, lastName, email, phone, password, and isActive.
Order: A finalized checkout transaction. It belongs to a Store and references a Customer. Critically, it contains a static snapshot of the customer's PII (customerName, customerEmail, customerPhone) captured at the time of checkout.
OrderItem: The line items of an order, containing snapshots of the product (productName, unitPrice) to preserve historical accuracy even if the original product changes.
Product: A sellable item within a Store.
Category: An organizational grouping for products within a Store.
Inventory: A 1:1 linked record for a product tracking its stock quantity. 4. Authentication Flow
Customer Registration: POST /stores/:storeId/customers/register creates a new customer. Returns a JWT and the profile.
Customer Login: POST /stores/:storeId/customers/login verifies credentials and issues a JWT.
Customer Logout: POST /stores/:storeId/customers/logout (Currently client-side destruction; backend simply returns 200).
Customer Session: Tracked via the JWT provided in the Authorization: Bearer <token> header.
Customer Profile: GET /stores/:storeId/customers/me returns the authenticated customer's details (name, email, phone).
Customer JWT lifecycle: Tokens expire after 24 hours. The frontend must handle dropping the session and redirecting to login.
Authentication middleware behavior: The middleware expects Authorization: Bearer <token>. It verifies the signature against CUSTOMER_JWT_SECRET, asserts the payload type is customer, and strictly asserts that the storeId in the JWT matches the storeId in the route parameters.
Unauthorized responses: Missing, malformed, or invalid tokens yield a 401 Unauthorized with the message Invalid or expired customer token.
Expired tokens: Yield a 401 Unauthorized.
Cross-store access: Providing a valid JWT for Store A to an endpoint in Store B yields a 401 Unauthorized. 5. Authorization
The frontend must adhere to the following authentication domains:

No Authentication (Public):
GET /stores (List public stores)
GET /stores/:storeId/categories
GET /stores/:storeId/products
GET /stores/:storeId/products/:id
POST /stores/:storeId/customers/register
POST /stores/:storeId/customers/login
Customer Authentication (Requires Customer JWT):
GET /stores/:storeId/customers/me
GET /stores/:storeId/customers/me/orders
GET /stores/:storeId/customers/me/orders/:id
POST /stores/:storeId/orders
Platform Authentication & Store Permissions (Requires Platform JWT):
Catalog Management, Inventory Management, Store Settings, Order Fulfillment (GET /stores/:storeId/orders, PATCH /stores/:storeId/orders/:id/status).
Super Admin (Requires Platform JWT):
Store Approvals, Global Store Listing. 6. Complete API Reference
6.1 Register Customer
Method: POST
URL: /stores/:storeId/customers/register
Auth: None
Body: { firstName, lastName, email, phone, password }
Validation: Passwords must be >= 6 characters. Valid email format. phone 10-20 chars.
Success (201): { success: true, message: "...", data: { token, customer: { id, storeId, email } } }
Error (400): Email already registered for this store.
Error (422): Validation failure.
Behavior: Hashes password, creates isolated customer record.
6.2 Login Customer
Method: POST
URL: /stores/:storeId/customers/login
Auth: None
Body: { email, password }
Validation: Must provide email and password.
Success (200): { success: true, message: "...", data: { token, customer: { id, storeId, email } } }
Error (401): Invalid credentials (generic message to prevent enumeration).
Behavior: Issues a CUSTOMER_JWT.
6.3 Customer Profile
Method: GET
URL: /stores/:storeId/customers/me
Auth: Customer JWT
Body: None
Success (200): { success: true, data: { customer: { id, storeId, firstName, lastName, email, phone } } }
Error (401): Invalid/expired token.
Error (404): Customer not found.
6.4 Place Order (Checkout)
Method: POST
URL: /stores/:storeId/orders
Auth: Customer JWT
Body: { deliveryAddress: string, items: [ { productId: uuid, quantity: int } ] }
Validation: At least 1 item. Quantities must be > 0.
Success (201): { success: true, data: { order: { ... } } }
Error (400): Insufficient inventory, Product not found.
Behavior: Atomically deducts inventory, reads req.customer to inject PII, and generates the order.
6.5 Customer Order History
Method: GET
URL: /stores/:storeId/customers/me/orders
Auth: Customer JWT
Success (200): { success: true, data: { orders: [ ... ] } }
Behavior: Lists orders belonging to the authenticated customer, sorted newest first.
6.6 Customer Order Details
Method: GET
URL: /stores/:storeId/customers/me/orders/:id
Auth: Customer JWT
Success (200): { success: true, data: { order: { ... } } }
Error (404): Order not found (or does not belong to this customer). 7. Checkout Flow
How it works: The customer logs in, adds items to their local state cart, and submits a POST request to the orders endpoint with their JWT.
Customer data injection: The frontend NO LONGER sends customerName, customerEmail, or customerPhone in the payload. The backend reads this directly from the database using the identity provided by the JWT.
Delivery Address: The only user-provided string required in the checkout payload is the deliveryAddress.
Order Snapshots: When the order is created, the current price of the product and the current name/email/phone of the customer are permanently copied into the orders and order_items tables.
Inventory Validation: The backend executes an atomic transaction. If any product in the cart lacks sufficient inventory, the entire order is rejected and no stock is deducted.
Pricing: The backend entirely ignores any price the frontend might attempt to send. Prices are calculated exclusively server-side. 8. Order History
How it works: The GET /me/orders endpoint retrieves all orders where customerId matches the logged-in user.
Sorting: Newest orders appear first (DESC by createdAt).
Ownership validation: The database query strictly filters by customerId = req.customer.id. It is mathematically impossible for Customer A to view Customer B's orders.
Individual order retrieval: Fetches the full detailed object, including order_items. 9. Error Handling
The frontend should expect standardized error responses in the format: { "success": false, "message": "Error description" } (and occasionally a errors array for validation).

401 Unauthorized: Missing JWT, expired JWT, invalid signature, or cross-store token usage. (Frontend action: Redirect to login).
403 Forbidden: (Primarily used for Platform RBAC, not typically seen by customers).
404 Not Found: Endpoint missing, or resource (like an Order ID) does not exist or belong to the user.
400 Bad Request: Business logic violations.
"Email already registered for this store"
"Insufficient inventory for product [name]"
422 Unprocessable Entity: Zod validation failures. Will include an errors array mapping fields to issues. 10. Business Rules
✓ Customers are completely separate from Platform Users.
✓ Customer identity is strictly scoped to exactly one store.
✓ The same email address is allowed across different stores (treated as entirely separate accounts).
✓ The same email address cannot exist twice inside the same store.
✓ Platform JWTs cannot authenticate customer routes.
✓ Customer JWTs cannot authenticate platform routes.
✓ Customer JWTs issued for Store A cannot access endpoints for Store B.
✓ Order placement strictly requires customer authentication.
✓ Guest checkout is completely removed from the system.
✓ Public guest order tracking via email/phone is completely removed from the system.
✓ Customer information (name, email, phone) is securely snapshotted into orders at checkout.
✓ Legacy guest orders remain valid in the database but are no longer accessible by customers. 11. Frontend State Expectations
Store JWT: Immediately upon a successful 200/201 response from /login or /register. Save it securely (e.g., localStorage or memory depending on security posture).
Clear JWT: When the user clicks Logout, or whenever an API call returns a 401 Unauthorized.
Redirects: Redirect unauthenticated users attempting to access the checkout page or the "My Orders" page back to the store's login view.
Refresh profile: Call /me on initial app load if a token exists in storage to hydrate the user's name/email in the UI header.
Checkout restriction: The frontend must proactively block checkout flows if no token is present, prompting a login/registration modal or page.
Customer info loading: Do not ask the user to type their name or email during checkout; display it statically based on their profile. 12. Removed Features
Guest Checkout: Replaced entirely by Authenticated Checkout.
Guest Order Tracking Page: The POST /stores/:storeId/orders/track endpoint no longer exists. The frontend must remove any "Track my order without an account" UI. It is replaced by the authenticated Order History view.
Customer PII in Checkout Payload: The frontend must remove customerName, customerEmail, and customerPhone from the checkout API call. 13. Security Notes
Never decode JWT for business logic: Treat the token as an opaque string. Always fetch the profile from /me to get the customer's true state.
Always trust backend authorization: If the backend says 401, the session is dead. Drop the state immediately.
Always send Authorization header: Attach Authorization: Bearer <token> to all protected endpoints.
Token Segregation: If your frontend manages both a Platform Admin dashboard and a Customer Storefront on the same domain, ensure you store their JWTs in separate keys (e.g., admin_token vs customer_token) to avoid overwriting them or sending the wrong token. 14. Documentation Verification
The backend implementation has been strictly verified against the following core documents:

backend_capability.md
module-04-public-storefront.md
module-05-orders.md
06-database-design.md
Verification Status: No differences exist. The implementation and documentation are perfectly synchronized.

15. Frontend Implementation Notes
    To align with this milestone, the frontend must now:

Remove the guest checkout form and tracking page.
Implement a Customer Registration and Login view specifically scoped to the active storefront.
Require login before the checkout button can be clicked.
Store the Customer JWT and attach it to API calls targeting /me, /me/orders, and /orders.
Fetch the customer profile via /customers/me to hydrate the UI.
Render an Order History page using /customers/me/orders.
Remove customerName, customerEmail, and customerPhone fields from the checkout submission payload. 16. Known Constraints
The following features were intentionally excluded from this MVP milestone and are not implemented:

Saved delivery addresses (Delivery address must be typed every time).
Persistent backend carts (Carts must be managed purely in frontend state).
Wishlists.
Product reviews.
Password reset / "Forgot Password" flows.
Email verification flows.
Social login (OAuth).
Refresh tokens (Sessions hard-expire after 24h).
Customer profile editing (Name/email updates are not supported).
Payment gateways (Orders bypass payment processing).
Shipping calculations.
