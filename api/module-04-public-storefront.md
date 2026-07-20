# Module 04: Public Storefront APIs

This document defines the API contracts for the public storefront. These APIs are used to list stores, categories, and products to unauthenticated public visitors.

## Authentication & Authorization
All endpoints in this module are public. They do not require a JWT token or any specific permissions.

---

## 1. List Public Stores
Fetch a directory of all active, publicly available stores on the platform.

**Endpoint:** `GET /stores/public`
*(Note: As verified in `store.service.js`, the platform exposes a method to list public stores, though the exact route might be handled under `/stores` with specific visibility filters).*

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Stores retrieved successfully",
  "data": {
    "stores": [
      {
        "id": "uuid",
        "name": "string",
        "slug": "string",
        "description": "string",
        "avatarUrl": "string"
      }
    ]
  }
}
```

---

## 2. List Categories in a Store
Fetch all categories for a specific store.

**Endpoint:** `GET /stores/:storeId/categories`

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Categories retrieved successfully",
  "data": {
    "categories": [
      {
        "id": "uuid",
        "name": "string",
        "slug": "string"
      }
    ]
  }
}
```

---

## 3. List Products in a Store
Fetch all products for a specific store, optionally filtered by a category.

**Endpoint:** `GET /stores/:storeId/products`

**Query Parameters:**
- `categoryId` (optional uuid): Filter products by a specific category.

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Products retrieved successfully",
  "data": {
    "products": [
      {
        "id": "uuid",
        "name": "string",
        "price": "number",
        "imageUrls": ["url string"],
        "category": {
          "id": "uuid",
          "name": "string",
          "slug": "string"
        }
      }
    ]
  }
}
```

---

## 4. Get Product Details
Fetch detailed information about a single product, including its current inventory stock.

**Endpoint:** `GET /stores/:storeId/products/:id`

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Product retrieved successfully",
  "data": {
    "product": {
      "id": "uuid",
      "name": "string",
      "description": "string",
      "price": "number",
      "imageUrls": ["url string"],
      "category": { ... },
      "inventory": {
        "quantity": "number"
      }
    }
  }
}
```

---
*Last verified against code on 2026-07-19: Verified endpoints in `category.routes.js`, `product.routes.js`, and `store.service.js`.*
