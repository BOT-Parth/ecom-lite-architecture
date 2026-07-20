# Module 02: Catalog APIs

This document defines the API contracts for the Catalog Management module. These APIs are used by authenticated store staff to manage their inventory and product listings.

## Authentication & Authorization
All endpoints in this module require:
- A valid JWT Bearer token in the `Authorization` header.
- The `MANAGE_PRODUCTS` permission for the specific `storeId` provided in the path.

---

## 1. Create Category
Create a new product category inside a store.

**Endpoint:** `POST /stores/:storeId/categories`

**Request Body:**
```json
{
  "name": "string (3-50 chars)",
  "slug": "string (3-50 chars, lowercase alphanumeric and hyphens)"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Category created successfully",
  "data": {
    "category": {
      "id": "uuid",
      "name": "string",
      "slug": "string",
      "storeId": "uuid"
    }
  }
}
```

---

## 2. Update Category
Update an existing product category.

**Endpoint:** `PATCH /stores/:storeId/categories/:id`

**Request Body:** (All fields optional)
```json
{
  "name": "string (3-50 chars)",
  "slug": "string (3-50 chars)"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Category updated successfully",
  "data": {
    "category": { ... }
  }
}
```

---

## 3. Delete Category
Delete an existing product category.

**Endpoint:** `DELETE /stores/:storeId/categories/:id`

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Category deleted successfully",
  "data": null
}
```

---

## 4. Create Product
Create a new product inside a store.

**Endpoint:** `POST /stores/:storeId/products`

**Request Body:**
```json
{
  "name": "string (3-100 chars)",
  "description": "string (optional)",
  "price": "number (positive float)",
  "imageUrls": ["url string"] (optional),
  "categoryId": "uuid" (optional)
}
```
*Note: An associated inventory record is automatically created with quantity 0.*

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Product created successfully",
  "data": {
    "product": {
      "id": "uuid",
      "name": "string",
      "price": "number",
      "storeId": "uuid",
      "categoryId": "uuid",
      ...
    }
  }
}
```

---

## 5. Update Product
Update an existing product.

**Endpoint:** `PATCH /stores/:storeId/products/:id`

**Request Body:** (All fields optional)
```json
{
  "name": "string",
  "description": "string",
  "price": "number",
  "imageUrls": ["url string"],
  "categoryId": "uuid"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Product updated successfully",
  "data": {
    "product": { ... }
  }
}
```

---

## 6. Delete Product
Delete an existing product.

**Endpoint:** `DELETE /stores/:storeId/products/:id`

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Product deleted successfully",
  "data": null
}
```

---
*Last verified against code on 2026-07-19: Verified endpoints and Zod schemas in `category.routes.js`, `product.routes.js` and `catalog.validator.js`.*
