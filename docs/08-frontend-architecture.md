# E-Com Lite Frontend Architecture & Flow Documentation

Created on: July 16, 2026

## 1. Directory Structure & Responsibilities

- **`src/context/`**: Houses global context providers for core application state.
  - `AuthContext.jsx`: Encapsulates user session, token validation, login/logout mechanisms, and authorization metadata.
  - `ToastContext.jsx`: Provides transient overlay alert queues.
- **`src/hooks/`**: Exposes syntactic hooks (`useAuth.js`, `useToast.js`) to consume global contexts safely.
- **`src/services/`**: Configures third-party API clients and domain-level resolvers.
  - `api.js`: Axios instance configured with automatic JWT attachment and standardized response un-wrapping.
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

### Authentication (Authentication lifecycle)
1. **Bootstrap**: On startup, `AuthContext` reads the JWT token from `localStorage`.
   - If present, it executes `GET /auth/profile`. The response automatically contains the full user profile including authorization metadata.
   - If missing, or if the profile call returns `401 Unauthorized`, the session is cleared and the user is logged out.
2. **Login/Register**: Forms submit credentials to `POST /auth/login` or `POST /auth/register`. On success, tokens are saved to `localStorage`, and the user object is set in `AuthContext` memory. Includes client-side **Password visibility toggles** for improved UX.

### Authorization (Platform Roles & Membership Permissions)
- **Roles**: Mapped directly from the backend payload (`user.authorization.platformRole` and `user.authorization.storeMemberships`).
- **Route Protection**: The `ProtectedRoute` component intercepts access attempts:
  - If unauthenticated, it redirects to `/login` preserving the target path in navigation state.
  - If a route defines `allowedRoles` (e.g. `["SUPER_ADMIN"]`), and the user does not possess that platform role, they are redirected to `/profile`.

---

## 4. Routing & Landing Flows

```mermaid
graph TD
  Start[Navigating /] --> AuthCheck{Logged In?}
  AuthCheck -- No --> Login[/login]
  AuthCheck -- Yes --> RoleCheck{Platform Role?}
  RoleCheck -- SUPER_ADMIN --> Admin[/admin/requests]
  RoleCheck -- STORE_ADMIN --> StoreCheck{Has Store Slug?}
  StoreCheck -- Yes --> Dash[/stores/:slug/dashboard]
  StoreCheck -- No --> Prof[/profile]
```

- **Anonymous**: Routed to `/login` if trying to access restricted folders. Can view public storefronts (`/stores/:storeSlug`) and the `PublicMarketplace.jsx`.
- **STORE_ADMIN**:
  - Lands on `/stores/:storeSlug/dashboard` if a store is registered to their membership metadata.
  - Falls back to `/profile` if they do not yet own a store (allowing them to submit a creation request application).
- **SUPER_ADMIN**:
  - Lands directly on the approvals panel `/admin/requests` (`SuperAdminRequests.jsx`).
- **StoreResolver Behaviour**: Resolves store slugs tiered by user context (personal lists vs. platform directory vs. public fetch). Navigates correctly for closed or pending stores if authorized.

---

## 5. Navigation & Navbar Behaviour

- Dynamic rendering depending on `user.authorization` context:
  - **Anonymous**: Shows Login, Register
  - **STORE_ADMIN**: Shows Profile, Store Dashboard (only if `primaryStore` exists)
  - **SUPER_ADMIN**: Shows Browse Stores, Store Requests
- Implemented user session info dropdown display with distinct "Super Admin" vs "Merchant" labels.

---

## 6. Frontend Business Rules Enforced

1. **One User = One Store**:
   - The profile page hides the "Request New Store" CTA if the user has an active store membership.
   - Multi-store selection headers and switching controls have been completely removed from the UI.
2. **Access Control**:
   - Platform administration options (approvals queue, browse platform stores list) are only rendered and permitted for `SUPER_ADMIN` platformRole.

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
