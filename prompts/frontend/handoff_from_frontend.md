# Frontend Technical Handoff Report

## 1. Executive Summary
This report serves as the authoritative handoff for the current state of the E-Com Lite frontend application, concluding the **Customer Accounts MVP** milestone. 

The frontend has successfully transitioned from a public "guest checkout" model to a fully authenticated, dual-domain model. The architecture now strictly enforces the boundary between Platform Users (managing stores) and Customers (shopping at specific stores).

## 2. Current Frontend Architecture
The React application operates within a single SPA but maintains two completely isolated identity domains. 

**Platform Identity Domain:**
- **Users:** Super Admins, Store Owners, Store Managers, Store Staff.
- **API Client:** `src/services/api.js` (Manages `token` in `localStorage`).
- **Context:** `AuthContext.jsx` (Handles login, RBAC, and platform session hydration).
- **Routes:** `/admin/*`, `/stores`, `/stores/:storeId/dashboard/*`.

**Customer Identity Domain:**
- **Users:** Storefront Shoppers.
- **API Client:** `src/services/customerApi.js` (Manages `customer_token` in `localStorage`).
- **Context:** `CustomerAuthContext.jsx` (Handles customer login, registration, and session hydration scoped to a specific `storeId`).
- **Routes:** `/stores/:storeSlug/login`, `/stores/:storeSlug/register`, `/stores/:storeSlug/checkout`, `/stores/:storeSlug/orders/*`.

## 3. Routing & Navigation
The frontend routing hierarchy is heavily decoupled using nested layouts.

**Storefront Routing (`App.jsx` -> `PublicStoreLayout`):**
- `/stores/:storeSlug` (Public Catalog)
- `/stores/:storeSlug/products/:productId` (Public Product Details)
- `/stores/:storeSlug/login` (Public, redirects if already authenticated)
- `/stores/:storeSlug/register` (Public, redirects if already authenticated)
- `/stores/:storeSlug/checkout` (Protected via `<CustomerProtectedRoute>`)
- `/stores/:storeSlug/orders` (Protected via `<CustomerProtectedRoute>`)
- `/stores/:storeSlug/orders/:orderId` (Protected via `<CustomerProtectedRoute>`)

**Store Resolution & Guards:**
- `<PublicStoreLayout>` wraps all storefront routes.
- It instantiates `<StoreProvider>`, `<CustomerAuthProvider>`, and `<CartProvider>`.
- A nested `<StorefrontGuard>` intercepts `storeError` and `!store` (e.g., 404 Store Not Found) to safely render an error UI globally, preventing nested protected routes from prematurely mounting or redirecting unauthenticated users to `/login` for an invalid store.

## 4. Context Hierarchy & Ownership
- **`<StoreProvider>`:** Calls `storeResolver.getStoreIdBySlug(slug)`. If successful, sets `store` and `storeId`.
- **`<CustomerAuthProvider>`:** Depends on `storeId`. Hydrates `customer` state by calling `customerApi.get('/me')`. Handles login/logout logic. Silently clears token on 401s.
- **`<CartProvider>`:** Managed purely in React `useState`. Survives customer login/logout. Is explicitly scoped to the active store via `key={storeSlug}` inside `PublicStoreLayout` to prevent cross-store cart bleed.

## 5. API Ownership Rules
- **`api.js`**: Must ONLY be used for Platform Authentication, RBAC checks, Dashboard interactions, and unauthenticated public endpoints (`GET /stores`, `GET /stores/:storeId/products`). 
- **`customerApi.js`**: Must ONLY be used for endpoints requiring `CUSTOMER_JWT` (e.g. `GET /stores/:storeId/customers/me`, `POST /stores/:storeId/orders`, `GET /stores/:storeId/customers/me/orders`).
- Neither client imports the other.

## 6. Deprecated & Removed Features
- **Guest Checkout:** Deprecated and removed from project architecture. PII fields (Name, Email, Phone) were stripped from the checkout form. `CustomerAuthContext` injects the customer identity via the `CUSTOMER_JWT`.
- **Order Tracking Form:** Deprecated and removed from project architecture. Customers now view order history via the authenticated `/stores/:slug/orders` page.

## 7. Current Milestone Status
**Customer Accounts MVP (Frontend): COMPLETE BUT OUT OF SYNC.**
- Guest checkout removed and customer auth implemented.
- **KNOWN PARITY GAPS**: The frontend is currently calling incorrect legacy endpoints (`/me` instead of `/profile`, `/me/orders` instead of `/orders`), and lacks a global 401 interceptor logout handler. These were discovered in a recent Gap Analysis.

## 7.5. UI/UX Improvements (Phase X)
- `StoreDashboard` converted to nested Route components instead of local state tabs.
- `ProductForm` decomposed into a modular `ProductEditor` with a live storefront `ProductCard` preview.
- Navigation stripped from `Navbar` and moved into dedicated Sidebar components (`PlatformSidebar`, `StoreSidebar`, `CustomerSidebar`).

## 8. Known Limitations
- Cart does not persist to backend (Purely client-side session state).
- Customers cannot update their profile (Name/Email/Phone).
- Password reset is not implemented.
- Shipping calculations and payment gateways are currently simulated/bypassed.
