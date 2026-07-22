# Module 04: Public Storefront APIs

This document defines the API contracts for the public storefront. These APIs are used to list stores, categories, and products to unauthenticated public visitors.

## Authentication & Authorization
- **Public endpoints** (e.g. List Stores, List Products, Customer Registration/Login) do not require a JWT token.
- **Customer endpoints** (e.g. `/me`, `/me/orders`) require a valid Customer JWT for the specific store context.

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
}
```

---

## 5. Register Customer
Register a new customer account for a specific store.

**Endpoint:** `POST /stores/:storeId/customers/register`

**Request Body:**
```json
{
  "firstName": "string (1-100 chars)",
  "lastName": "string (1-100 chars)",
  "email": "string (valid email)",
  "phone": "string (10-20 chars)",
  "password": "string (min 6 chars)"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Customer registered successfully",
  "data": {
    "token": "string (JWT)",
    "customer": {
      "id": "uuid",
      "storeId": "uuid",
      "email": "string"
    }
  }
}
```

---

## 6. Login Customer
Authenticate a customer for a specific store.

**Endpoint:** `POST /stores/:storeId/customers/login`

**Request Body:**
```json
{
  "email": "string",
  "password": "string"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "token": "string (JWT)",
    "customer": {
      "id": "uuid",
      "storeId": "uuid",
      "email": "string"
    }
  }
}
```

---

## 7. Get Customer Profile
Retrieve the authenticated customer's profile details.

**Endpoint:** `GET /stores/:storeId/customers/me`
**Access**: Authenticated Customer (`CUSTOMER_JWT`)

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Customer profile retrieved successfully",
  "data": {
    "customer": {
      "id": "uuid",
      "storeId": "uuid",
      "firstName": "string",
      "lastName": "string",
      "email": "string",
      "phone": "string"
    }
  }
}
```

---

## 8. Get Customer Order History
Retrieve the authenticated customer's past orders.

**Endpoint:** `GET /stores/:storeId/customers/me/orders`
**Access**: Authenticated Customer (`CUSTOMER_JWT`)

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Orders retrieved successfully",
  "data": {
    "orders": [
      {
        "id": "uuid",
        "orderNumber": "string",
        "totalAmount": "number",
        "status": "string",
        "createdAt": "timestamp"
      }
    ]
  }
}
```

---

## 9. Get Customer Order Details
Retrieve detailed information about a specific past order for the customer.

**Endpoint:** `GET /stores/:storeId/customers/me/orders/:id`
**Access**: Authenticated Customer (`CUSTOMER_JWT`)

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Order retrieved successfully",
  "data": {
    "order": { ... }
  }
}
```

---
*Last verified against code on 2026-07-22: Verified customer and order endpoints.*
