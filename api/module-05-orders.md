# Module 05: Orders

This module handles order placement (checkout) and order management for store owners/staff.
It integrates deeply with Customer Accounts to link orders and snaps customer PII at checkout.

## Authentication & Authorization
- **Customer endpoints** (`GET /stores/:storeId/customers/orders`, `POST /stores/:storeId/orders`) require an Authenticated Customer (`CUSTOMER_JWT`).
- **Protected store endpoints** (`GET /stores/:storeId/orders`) require a valid Platform JWT Bearer token and the `MANAGE_ORDERS` store permission for the specified `storeId`.
- **Guest endpoints** (`POST /stores/:storeId/orders/track`) are **DEPRECATED** and removed.

---

## 1. Place Order (Authenticated)
Place a new customer order.

**Endpoint:** `POST /stores/:storeId/orders`
**Access**: Authenticated Customer (`CUSTOMER_JWT`)
**Purpose**: Submits a new order, verifies customer identity via token, and atomically deducts inventory.

**Request Body:**
```json
{
  "deliveryAddress": "string (1-500 chars)",
  "items": [
    {
      "productId": "uuid",
      "quantity": "integer (positive)"
    }
  ]
}
```
*Note: `customerName`, `customerEmail`, and `customerPhone` are read directly from the `req.customer` profile and snapshotted. Duplicate `productId`s are automatically merged. If any product has insufficient inventory or does not exist, the entire request is rejected.*

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

## 2. Customer Order History (Authenticated)
List all orders placed by the authenticated customer.

**Endpoint:** `GET /stores/:storeId/customers/orders`
**Access**: Authenticated Customer (`CUSTOMER_JWT`)

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
        "items": [ ... ]
      }
    ]
  }
}
```

---

## 2.1 Customer Order Details (Authenticated)
Get full details of a specific order placed by the authenticated customer.

**Endpoint:** `GET /stores/:storeId/customers/orders/:id`
**Access**: Authenticated Customer (`CUSTOMER_JWT`)

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

## [DEPRECATED] Track Orders (Guest)
*Note: Guest checkout and guest tracking have been completely removed from the system. This section is preserved for historical context only.*

**Endpoint:** `POST /stores/:storeId/orders/track`
**Status:** ❌ Removed

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
*Last verified against code on 2026-07-23: Verified endpoints and schemas reflect Customer Authentication models. Guest ordering endpoints are removed.*
