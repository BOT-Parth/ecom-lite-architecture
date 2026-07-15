# Store Dashboard API Module Documentation

This document describes the APIs for store settings, catalog, and inventory management scoped within a specific store context.

---

## Store Settings API

### `GET /stores/:storeId/settings`
* **Purpose**: Retrieve settings for a specific store.
* **Authentication**: Required (Bearer JWT).
* **Authorization**: Requires store permission `MANAGE_STORE` (Store Owner) or platform permission `APPROVE_STORE` (Super Admin).
* **Response (200 OK)**:
```json
{
  "success": true,
  "message": "Store settings retrieved successfully",
  "data": {
    "settings": {
      "name": "My Electronic Store",
      "description": "A premium electronics shop.",
      "avatarUrl": "https://example.com/assets/logo.png"
    }
  }
}
```

### `PATCH /stores/:storeId/settings`
* **Purpose**: Update settings for a specific store.
* **Authentication**: Required (Bearer JWT).
* **Authorization**: Requires store permission `MANAGE_STORE` (Store Owner) or platform permission `APPROVE_STORE` (Super Admin).
* **Request Body**:
```json
{
  "name": "My Electronic Store Updated",
  "description": "Our updated electronics shop.",
  "avatarUrl": "https://example.com/assets/logo-updated.png"
}
```
* **Validation**:
  - `name`: 3–100 characters.
  - `description`: optional string, maximum 500 characters.
  - `avatarUrl`: optional string, maximum 2048 characters.
* **Response (200 OK)**:
```json
{
  "success": true,
  "message": "Store settings updated successfully",
  "data": {
    "settings": {
      "id": "<uuid>",
      "name": "My Electronic Store Updated",
      "slug": "my-electronic-store",
      "description": "Our updated electronics shop.",
      "avatarUrl": "https://example.com/assets/logo-updated.png",
      "approvalStatus": "APPROVED",
      "operationalStatus": "OPEN",
      "createdAt": "<timestamp>",
      "updatedAt": "<timestamp>"
    }
  }
}
```
