# Catalog and Inventory API Contracts

This document contains the backend API contracts for the Category, Product, and Inventory modules.

---

## Category APIs

### POST /stores/:storeId/categories
* **Purpose**: Create a product category for a store.
* **Authentication**: Required (JWT Bearer Token)
* **Authorization**: Requires store permission `MANAGE_PRODUCTS` in the target store.
* **Request Body**:
  ```json
  {
    "name": "Laptops",
    "slug": "laptops"
  }
  ```
* **Validation**:
  - `name`: string (min 3, max 50)
  - `slug`: string (min 3, max 50, lowercase alphanumeric + hyphens only: `^[a-z0-9-]+$`)
* **Response (201 Created)**:
  ```json
  {
    "success": true,
    "message": "Category created successfully",
    "data": {
      "category": {
        "id": "uuid-string",
        "name": "Laptops",
        "slug": "laptops",
        "storeId": "store-uuid-string",
        "createdAt": "timestamp",
        "updatedAt": "timestamp"
      }
    }
  }
  ```

### GET /stores/:storeId/categories
* **Purpose**: List all categories within a store.
* **Authentication**: None (Public)
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "message": "Categories retrieved successfully",
    "data": {
      "categories": [
        {
          "id": "uuid-string",
          "name": "Laptops",
          "slug": "laptops",
          "storeId": "store-uuid-string",
          "createdAt": "timestamp",
          "updatedAt": "timestamp"
        }
      ]
    }
  }
  ```

### PATCH /stores/:storeId/categories/:id
* **Purpose**: Update a category.
* **Authentication**: Required (JWT Bearer Token)
* **Authorization**: Requires store permission `MANAGE_PRODUCTS` in the target store.
* **Request Body**: (At least one of the fields is optional)
  ```json
  {
    "name": "Laptops & Notebooks",
    "slug": "laptops-notebooks"
  }
  ```
* **Response (200 OK)**: Returns updated category details.

### DELETE /stores/:storeId/categories/:id
* **Purpose**: Delete a category.
* **Authentication**: Required (JWT Bearer Token)
* **Authorization**: Requires store permission `MANAGE_PRODUCTS` in the target store.
* **Business Rules**: Products referencing this category are NOT deleted. Their category relation is updated to `NULL` (SetNull).
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "message": "Category deleted successfully",
    "data": null
  }
  ```

---

## Product APIs

### POST /stores/:storeId/products
* **Purpose**: Create a product inside a store.
* **Authentication**: Required (JWT Bearer Token)
* **Authorization**: Requires store permission `MANAGE_PRODUCTS` in the target store.
* **Request Body**:
  ```json
  {
    "name": "MacBook Pro M3",
    "description": "Powerful Apple Silicon Laptop",
    "price": 1999.99,
    "imageUrls": ["https://example.com/macbook.jpg"],
    "categoryId": "category-uuid-string"
  }
  ```
* **Validation**:
  - `name`: string (min 3, max 100)
  - `description`: string (optional)
  - `price`: positive number (decimal formatted)
  - `imageUrls`: array of valid URLs (optional)
  - `categoryId`: UUID of a category belonging to the store (optional)
* **Business Rules**: Creates a matching `Inventory` record with quantity `0` automatically.
* **Response (201 Created)**: Returns created product and linked category details.

### GET /stores/:storeId/products
* **Purpose**: List all products in a store. Optionally filter by category.
* **Authentication**: None (Public)
* **Query Parameters**:
  - `categoryId` (optional): Filter list to only show products in this category.
* **Response (200 OK)**: Returns list of products.

### GET /stores/:storeId/products/:id
* **Purpose**: Retrieve detailed information for a specific product.
* **Authentication**: None (Public)
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "message": "Product details retrieved successfully",
    "data": {
      "product": {
        "id": "uuid-string",
        "name": "MacBook Pro M3",
        "description": "Powerful Apple Silicon Laptop",
        "price": "1999.99",
        "imageUrls": ["..."],
        "categoryId": "...",
        "storeId": "...",
        "createdAt": "...",
        "updatedAt": "...",
        "category": { ... },
        "inventory": { ... }
      }
    }
  }
  ```

### PATCH /stores/:storeId/products/:id
* **Purpose**: Update product details.
* **Authentication**: Required (JWT Bearer Token)
* **Authorization**: Requires store permission `MANAGE_PRODUCTS`.
* **Request Body**: (All fields are optional, `categoryId` can be set to `null` to clear relation)
* **Response (200 OK)**: Returns updated product payload.

### DELETE /stores/:storeId/products/:id
* **Purpose**: Delete a product.
* **Authentication**: Required (JWT Bearer Token)
* **Authorization**: Requires store permission `MANAGE_PRODUCTS`.
* **Response (200 OK)**

---

## Inventory APIs

### GET /stores/:storeId/products/:productId/inventory
* **Purpose**: Retrieve product stock quantity.
* **Authentication**: None (Public)
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "message": "Inventory retrieved successfully",
    "data": {
      "inventory": {
        "id": "uuid-string",
        "productId": "uuid-string",
        "quantity": 0,
        "createdAt": "timestamp",
        "updatedAt": "timestamp"
      }
    }
  }
  ```

### PATCH /stores/:storeId/products/:productId/inventory
* **Purpose**: Update inventory quantity.
* **Authentication**: Required (JWT Bearer Token)
* **Authorization**: Requires store permission `MANAGE_PRODUCTS`.
* **Request Body**:
  ```json
  {
    "quantity": 50
  }
  ```
* **Validation**:
  - `quantity`: non-negative integer (`>= 0`)
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "message": "Inventory updated successfully",
    "data": {
      "inventory": {
        "id": "uuid-string",
        "productId": "uuid-string",
        "quantity": 50,
        "createdAt": "timestamp",
        "updatedAt": "timestamp"
      }
    }
  }
  ```
