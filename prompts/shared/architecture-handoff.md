# Frontend Integration Notes — Catalog & Inventory

This document provides frontend developers with the information needed to integrate catalog (categories and products) and inventory management features into the user interface.

---

## Catalog Integration

### 1. Categories List (`GET /stores/:storeId/categories`)
- **Required Roles**: None (Public)
- **UI Element**: Sidebar categories, navigation drop-downs.
- **Notes**: Returns a flat array of categories. Always display categories alphabetically.

### 2. Products Catalog (`GET /stores/:storeId/products`)
- **Required Roles**: None (Public)
- **Filters**: Pass `?categoryId=uuid` to filter product directories dynamically.
- **UI Element**: Storefront grid, search results.

---

## Inventory Integration

### 1. Stock Quantity Display (`GET /stores/:storeId/products/:productId/inventory`)
- **Required Roles**: None (Public)
- **UI Element**: Product details pages, storefront inventory status ("Out of stock", "In Stock (5 items)").
- **Notes**: Stock quantities are returned inside a nested object `{ data: { inventory: { quantity: number } } }`.

### 2. Stock Update Form (`PATCH /stores/:storeId/products/:productId/inventory`)
- **Required Roles**: Authenticated user with `STORE_OWNER` or `STORE_STAFF` membership role containing `MANAGE_PRODUCTS` store-level permission.
- **Request Format**:
  ```json
  {
    "quantity": 50
  }
  ```
- **UI Recommendations**:
  - **Inputs**: Number input with `min=0` and `step=1` (enforce integer check in frontend).
  - **Loading States**: Disable the "Update" button during request flight.
  - **Error Handling**: Display clear validation error messages if a negative count or decimal value is somehow submitted (expect `400 Bad Request`).
  - **Permission Check**: Hide or disable stock modification fields if the user does not have `MANAGE_PRODUCTS` permission inside the store.
