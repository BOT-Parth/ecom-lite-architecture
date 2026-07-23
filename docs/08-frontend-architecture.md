# E-Com Lite Frontend Architecture & Flow Documentation

Created on: July 16, 2026

## 1. Directory Structure & Responsibilities

- **`src/context/`**: Houses global context providers for core application state.
  - `AuthContext.jsx`: Encapsulates Platform User session, token validation, login/logout, and RBAC.
  - `CustomerAuthContext.jsx`: Encapsulates Storefront Customer session, token validation, login/logout scoped to a specific store.
  - `CartContext.jsx`: Manages volatile cart state, strictly scoped via React `key` to the active store.
  - `StoreContext.jsx`: Handles public store slug resolution and exposes the active `store` and `storeId` to children.
  - `ToastContext.jsx`: Provides transient overlay alert queues.
- **`src/hooks/`**: Exposes syntactic hooks (`useAuth`, `useCustomerAuth`, `useStore`, `useCart`, `useToast`) to consume global contexts safely.
- **`src/services/`**: Configures third-party API clients and domain-level resolvers.
  - `api.js`: Axios instance configured with automatic JWT attachment for Platform Users.
  - `customerApi.js`: Axios instance configured with automatic `CUSTOMER_JWT` attachment for Storefront Customers.
  - `storeResolver.js`: Tiered store slug resolution searching personal lists, admin platform directories, and public endpoints.
- **`src/pages/`**: Top-level page views mapped to path routes in the Router.
  - `PublicMarketplace.jsx`: Public landing page and marketplace directory.
  - `SuperAdminRequests.jsx`: SUPER_ADMIN approvals dashboard for store requests.
  - `BrowseStores.jsx`: SUPER_ADMIN administrative view for reviewing all tenant stores.
  - `StoreDashboard.jsx`: Protected merchant panel.
- **`src/components/`**: Modular sub-widgets divided into `auth/`, `Navbar/`, and `dashboard/` (merchant editing forms and tab sheets).
- **`src/constants/`**: Exposes configuration files like `api.js` declaring resource endpoints.
- **`src/layouts/`**: Wraps routes in shared page grids (`MainLayout.jsx`).

---

## 2. File Naming Convention

**Rule:** Avoid generic names unless the concept itself is generic within the application. A generic name is acceptable when there is exactly ONE instance of that concept in the app. A generic name is a violation when it obscures which of several similar things a file is (e.g., Dashboard, Panel, Home used for non-generic routing shells).

- **Note on `StoreDashboard.jsx`**: This name is currently deferred from renaming because there is exactly one merchant dashboard in the app. **CONDITIONAL RENAME**: Revisit and rename this file to `StoreAdminDashboard.jsx` the moment a second dashboard-type page is introduced anywhere in the application.

---

## 3. Authentication & Authorization Flows

### Platform Authentication (Admin/Merchant)
1. **Bootstrap**: On startup, `AuthContext` reads the JWT token from `localStorage`.
   - If present, it executes `GET /auth/profile`. The response automatically contains the full user profile including authorization metadata.
   - If missing, or if the profile call returns `401 Unauthorized`, the session is cleared and the user is logged out.
2. **Login/Register**: Forms submit credentials to `POST /auth/login` or `POST /auth/register`. On success, tokens are saved to `localStorage`, and the user object is set in `AuthContext` memory.
3. **Role Protection**: The `ProtectedRoute` component intercepts access attempts. Redirects unauthenticated users to `/login`. Enforces `allowedRoles` (e.g., `SUPER_ADMIN`).

### Customer Authentication (Storefront)
1. **Bootstrap**: `CustomerAuthContext` only attempts hydration (`GET /stores/:storeId/customers/me` via `customerApi`) *after* the `StoreContext` successfully resolves a valid `storeId`.
2. **Login/Register**: Customers authenticate against a specific storefront (`POST /stores/:storeId/customers/login`).
3. **Token Scoping**: `customer_token` is completely isolated from the Platform `token`. A customer token for Store A mathematically cannot authenticate against Store B.
4. **Route Protection**: The `CustomerProtectedRoute` component intercepts customer routes (like checkout or order history). Unauthenticated attempts redirect to `/stores/:storeSlug/login`.

---

## 4. Routing & Landing Flows

### Platform Routing
- **Unauthenticated**: Routed to `/login` if trying to access restricted folders.
- **STORE_ADMIN**: Lands on `/stores/:storeSlug/dashboard` if a store is registered. Falls back to `/profile` if none exists.
- **SUPER_ADMIN**: Lands directly on the approvals panel `/admin/requests`.

### Storefront Routing
- **Hierarchy**: All storefront routes are wrapped in `<PublicStoreLayout>` which provides `<StoreProvider>`, `<CustomerAuthProvider>`, and `<CartProvider>`.
- **`<StorefrontGuard>`**: Intercepts failed store resolution (invalid slugs) to globally render a "Store Not Found" view, safely preventing nested routes from mounting.
- **Customer Pages**: `/stores/:storeSlug` (Catalog), `/login`, `/register`, `/checkout`, `/orders`. Checkout and Orders are protected via `<CustomerProtectedRoute>`.

---

## 5. Navigation & Navbar Behaviour

- **Navbar**: Stripped of all links except branding, user info, and logout. It is now strictly a presentational top bar.
- **Platform Sidebar**: Primary navigation for platform administration. Dynamic rendering depending on `user.authorization` context:
  - **STORE_ADMIN**: Shows generic Dashboard, Profile, Settings.
  - **SUPER_ADMIN**: Also shows Admin section (Browse Stores, Store Requests, Platform Analytics).
- **Store Sidebar**: Primary navigation for store management contexts (Products, Categories, Orders, Inventory, Settings).
- **Customer Sidebar**: Primary navigation for the storefront, rendering categories and enabling URL-based filtering.

---

## 6. Frontend Business Rules Enforced

1. **One User = One Store**:
   - The profile page hides the "Request New Store" CTA if the user has an active store membership.
   - Multi-store selection headers and switching controls have been completely removed from the UI.
2. **Access Control**:
   - Platform administration options (approvals queue, browse platform stores list) are only rendered and permitted for `SUPER_ADMIN` platformRole.
3. **Dual Identity Segregation**:
   - Platform Users and Customers are mathematically walled off. `customerApi.js` owns the customer token, `api.js` owns the platform token.
4. **Checkout Security**:
   - Guest Checkout is officially deprecated and removed from project architecture. Customers must authenticate. PII (Name, Email, Phone) is stripped from the payload and securely hydrated natively by the backend via JWT decoding.

---

## 7. Store Settings & Branding (F3, F3.1)

### Store Settings (F3)
- **Settings API Integration**: The frontend integrates with `PATCH /stores/:storeId/settings` allowing authorized merchants (STORE_ADMIN) and SUPER_ADMINs to update store configurations.
- **Settings Tab**: A dedicated "Store Settings" tab was added to `StoreDashboard.jsx`, utilizing the existing backend contract for updates.

### Store Branding (F3.1)
- **Real-Time Updates**: Upon saving settings, `refreshProfile()` is triggered, updating the global `AuthContext` seamlessly without requiring a page reload. This propagates changes instantly to the Navbar, Profile Cards, and Store Dashboard header.
- **Branding Fields**: The public storefront (`PublicStore.jsx`) consumes `avatarUrl` and `description`.
  - **Avatar Rendering**: If an `avatarUrl` is provided, it is rendered. If absent, a stylized fallback placeholder is generated using the store's initial.
- **Product Images**: Verified that the frontend supports multiple comma-separated image URLs in the `ProductForm`, but strictly adheres to the MVP single-image behavior by only rendering the primary image (`imageUrls[0]`) in the storefront views (`PublicStore.jsx` and `ProductDetails.jsx`). No gallery or carousel components were implemented.
- **Parity Gaps**: 
  - Settings fields like `slug`, `contactEmail`, `phone`, `address`, `website`, `logoUrl`, `bannerUrl`, and `operationalStatus` are not yet supported by the settings endpoints and remain excluded from the UI.
  - Image upload is currently unsupported natively by the backend; therefore, Avatar URLs and Product Image URLs must be manually pasted as text links.
