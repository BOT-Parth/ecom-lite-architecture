# Module 05: Customer Orders APIs

This document defines the API contracts for the Customer Orders module. These APIs handle order placement, guest tracking, and store staff order management.

## Authentication & Authorization
- **Public endpoints** (`POST /stores/:storeId/orders`, `POST /stores/:storeId/orders/track`) require no authentication.
- **Protected endpoints** require a valid JWT Bearer token and the `MANAGE_ORDERS` store permission for the specified `storeId`.

---

## 1. Place Order (Public)
Place a new customer order.

**Endpoint:** `POST /stores/:storeId/orders`

**Request Body:**
```json
{
  "customerName": "string (3-100 chars)",
  "customerEmail": "string (valid email)",
  "customerPhone": "string (min 10 chars)",
  "deliveryAddress": "string (1-500 chars)",
  "items": [
    {
      "productId": "uuid",
      "quantity": "integer (positive)"
    }
  ]
}
```
*Note: Duplicate `productId`s are automatically merged. If any product has insufficient inventory or does not exist, the entire request is rejected.*

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Order placed successfully",
  "data": {
    "order": {
      "id": "uuid",
      "orderNumber": "string",
      "storeId": "uuid",
      "customerName": "string",
      "customerEmail": "string",
      "customerPhone": "string",
      "deliveryAddress": "string",
      "totalAmount": "number",
      "status": "PLACED",
      "createdAt": "timestamp",
      "updatedAt": "timestamp",
      "items": [
        {
          "id": "uuid",
          "productId": "uuid",
          "productName": "string",
          "quantity": "integer",
          "unitPrice": "number"
        }
      ]
    }
  }
}
```

---

## 2. Track Orders (Public)
Track existing guest orders for a specific store. Orders are returned newest first.

**Endpoint:** `POST /stores/:storeId/orders/track`

**Request Body:**
```json
{
  "email": "string",
  "phone": "string"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "orders": [
      {
        "id": "uuid",
        "orderNumber": "string",
        "storeId": "uuid",
        "totalAmount": "number",
        "status": "string",
        "createdAt": "timestamp",
        "updatedAt": "timestamp",
        "maskedDeliveryAddress": "string (e.g., 'Tech City, CA' or '***')",
        "items": [ ... ]
      }
    ]
  }
}
```
*Note: The response intentionally omits `customerEmail`, `customerPhone`, and `customerName`, and masks the `deliveryAddress` to protect PII.*

---

## 3. List Orders (Protected)
List all orders for a store.

**Endpoint:** `GET /stores/:storeId/orders`

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "orders": [
      {
        "id": "uuid",
        "orderNumber": "string",
        "status": "string",
        "totalAmount": "number",
        "createdAt": "timestamp",
        "items": [ ... ],
        ...
      }
    ]
  }
}
```

---

## 4. Get Order Details (Protected)
Get full details of a specific order.

**Endpoint:** `GET /stores/:storeId/orders/:id`

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "order": { ... }
  }
}
```

---

## 5. Update Order Status (Protected)
Update the status of an existing order.

**Endpoint:** `PATCH /stores/:storeId/orders/:id/status`

**Request Body:**
```json
{
  "status": "string (PLACED, PROCESSING, READY, COMPLETED, CANCELLED)"
}
```

**Status Transition Rules:**
- Valid forward flows: `PLACED` → `PROCESSING` → `READY` → `COMPLETED`.
- Orders can transition to `CANCELLED` from `PLACED`, `PROCESSING`, or `READY`.
- `COMPLETED` and `CANCELLED` are terminal states.
- If an order is transitioned to `CANCELLED`, its inventory is automatically restored. Idempotent cancellation requests are rejected.

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Order status updated successfully",
  "data": {
    "order": { ... }
  }
}
```

---
*Last verified against code on 2026-07-20: Verified endpoints, Zod schemas, and masking logic in `order.routes.js`, `order.controller.js` and `order.validator.js`.*
