# REST API Specification

**Product Name:** Depoma — Inventory & Stock Ledger Management System  
**Document Version:** 1.0  
**Base URL:** `/api/v1` (or `/api`)  
**Content-Type:** `application/json; charset=utf-8`  
**Authentication:** Bearer JWT (`Authorization: Bearer <access_token>`)  
**Timestamp Format:** All timestamps are transmitted in `ISO 8601 UTC` (`YYYY-MM-DDTHH:mm:ss.sssZ`). Conversion to Jalali date/time is performed by the client UI.

---

## 1. API Endpoint Summary

| Domain Module | Method & Path | Authorized Roles | Request Input | Successful Output | Primary Error Responses |
| :--- | :--- | :---: | :--- | :--- | :--- |
| **Auth** | `POST /api/auth/login` | Public | `{ email, password }` | `200 OK`: `{ accessToken, user }` | `400` Validation Error, `401` Invalid Credentials, `403` Inactive Account |
| **Auth** | `POST /api/auth/logout` | Authenticated | Active JWT Token | `200 OK`: `{ message }` | `401` Unauthorized |
| **Auth** | `GET /api/auth/me` | Authenticated | Active JWT Token | `200 OK`: Current user profile & assigned warehouses | `401` Unauthorized / Expired Token |
| **Users** | `GET /api/users` | `ADMIN` | `?role=ADMIN\|OPERATOR\|MANAGER` | `200 OK`: List of users and warehouse assignments | `401` Unauthorized, `403` Forbidden Role |
| **Users** | `POST /api/users` | `ADMIN` | `{ fullName, email, password, role }` | `201 Created`: Created user profile | `400` Validation Error, `403` Forbidden, `409` Duplicate Email |
| **Users** | `PATCH /api/users/:id` | `ADMIN` | `{ fullName?, role?, status? }` | `200 OK`: Updated user profile | `400` Invalid Data, `403` Forbidden, `404` User Not Found |
| **Categories** | `GET /api/categories` | Authenticated | None | `200 OK`: Array of categories | `401` Unauthorized |
| **Categories** | `POST /api/categories` | `ADMIN` | `{ name, description? }` | `201 Created`: Created category | `400` Validation Error, `403` Forbidden, `409` Duplicate Name |
| **Categories** | `PATCH /api/categories/:id` | `ADMIN` | `{ name?, description? }` | `200 OK`: Updated category | `400` Validation Error, `403` Forbidden, `404` Not Found, `409` Duplicate Name |
| **Products** | `GET /api/products` | Authenticated | `?categoryId=&search=&status=&page=&limit=` | `200 OK`: `{ items, meta }` paginated products list | `401` Unauthorized |
| **Products** | `POST /api/products` | `ADMIN` | `{ name, sku, categoryId, unit, minimumStock, description? }` | `201 Created`: Created product record | `400` Invalid Input, `403` Forbidden, `409` Duplicate SKU |
| **Products** | `GET /api/products/:id` | Authenticated | Path param `:id` (UUID) | `200 OK`: Product & category details *(balances fetched via `/api/inventory/balances?productId=:id`)* | `401` Unauthorized, `404` Product Not Found |
| **Products** | `PATCH /api/products/:id` | `ADMIN` | `{ name?, sku?, categoryId?, unit?, minimumStock?, status? }` | `200 OK`: Updated product record | `400` Invalid Input, `403` Forbidden, `404` Not Found, `409` Duplicate SKU |
| **Warehouses** | `GET /api/warehouses` | Authenticated (Scoped) | `?status=ACTIVE` | `200 OK`: `{ items }` accessible warehouses (`OPERATOR` sees assigned only) | `401` Unauthorized |
| **Warehouses** | `POST /api/warehouses` | `ADMIN` | `{ name, code, address?, description? }` | `201 Created`: Created warehouse record | `400` Invalid Input, `403` Forbidden, `409` Duplicate Name/Code |
| **Warehouses** | `GET /api/warehouses/:id` | Authenticated (Scoped) | Path param `:id` (UUID) | `200 OK`: Warehouse details & assigned operators | `403` Unassigned Operator, `404` Warehouse Not Found |
| **Warehouses** | `PATCH /api/warehouses/:id` | `ADMIN` | `{ name?, code?, address?, description?, status? }` | `200 OK`: Updated warehouse record | `403` Forbidden, `404` Not Found, `409` Duplicate Name/Code |
| **Warehouses** | `POST /api/warehouses/:id/operators` | `ADMIN` | `{ userId }` | `201 Created`: Operator assignment record | `400` User Not Operator, `403` Forbidden, `409` Already Assigned |
| **Warehouses** | `DELETE /api/warehouses/:id/operators/:userId` | `ADMIN` | `:id`, `:userId` (UUIDs) | `200 OK`: Assignment revoked confirmation | `403` Forbidden, `404` Assignment Not Found |
| **Inventory** | `GET /api/inventory/balances` | Authenticated (Scoped) | `?warehouseId=&productId=&lowStockOnly=` | `200 OK`: `{ items }` current stock balances per product and warehouse | `401` Unauthorized, `403` Unassigned Warehouse |
| **Inventory** | `POST /api/inventory/receipts` | `ADMIN`, `OPERATOR` | `{ warehouseId, productId, quantity, reason, referenceNo?, description? }` | `201 Created`: `IN` movement record + updated balance | `400` Quantity <= 0, `403` Unassigned Warehouse / Manager Role, `422` Inactive Resource |
| **Inventory** | `POST /api/inventory/issues` | `ADMIN`, `OPERATOR` | `{ warehouseId, productId, quantity, reason, referenceNo?, description? }` | `201 Created`: `OUT` movement record + updated balance | `400` Quantity <= 0, `403` Unassigned Warehouse, **`422` Insufficient Stock** |
| **Inventory** | `GET /api/inventory/movements` | Authenticated (Scoped) | `?warehouseId=&productId=&movementType=&page=&limit=` | `200 OK`: `{ items, meta }` paginated immutable stock movement ledger | `401` Unauthorized, `403` Unassigned Warehouse |
| **Dashboard** | `GET /api/dashboard/summary` | Authenticated (Scoped) | `?warehouseId=` (Optional) | `200 OK`: KPI counts & recent stock movements | `401` Unauthorized, `403` Unassigned Warehouse |

---

## 2. Standard List & Error Contracts

### 2.1. Standardized Collection Response Envelopes
All list endpoints across the API return deterministic object envelopes rather than top-level JSON arrays:

* **Unpaginated Collections (`GET /api/users`, `GET /api/categories`, `GET /api/warehouses`, `GET /api/inventory/balances`):**
  ```json
  {
    "items": [ ... ]
  }
  ```
* **Paginated Collections (`GET /api/products`, `GET /api/inventory/movements`):**
  ```json
  {
    "items": [ ... ],
    "meta": {
      "total": 100,
      "page": 1,
      "limit": 10,
      "totalPages": 10
    }
  }
  ```

### 2.2. Standard Error Contract

All error responses returned by the API conform to a consistent JSON envelope. The `errorCode` provides a programmatic identifier for client logic, while `message` provides a localized Persian string ready for display in the RTL user interface.

```json
{
  "statusCode": 422,
  "errorCode": "INSUFFICIENT_STOCK",
  "message": "موجودی کالا در این انبار کافی نیست. موجودی قابل دسترس: ۱۲ عدد، مقدار درخواستی: ۲۰ عدد.",
  "details": {
    "availableQuantity": 12,
    "requestedQuantity": 20,
    "productId": "p1000000-0000-4000-8000-000000000001",
    "warehouseId": "w1000000-0000-4000-8000-000000000001"
  },
  "timestamp": "2026-11-08T18:45:12.000Z",
  "path": "/api/inventory/issues"
}
```

### Standard Domain Error Codes
* `AUTH_INVALID_CREDENTIALS` (`401`): Invalid email or password.
* `AUTH_TOKEN_EXPIRED` (`401`): Missing, malformed, or expired JWT session token.
* `FORBIDDEN_ROLE` (`403`): Authenticated user's role is not permitted to perform this operation.
* `FORBIDDEN_WAREHOUSE_ACCESS` (`403`): Warehouse Operator attempted to view or mutate an unassigned warehouse.
* `VALIDATION_ERROR` (`400`): Request body or query parameters failed schema validation.
* `DUPLICATE_SKU` (`409`): A product with the specified SKU already exists.
* `DUPLICATE_WAREHOUSE_CODE` (`409`): A warehouse with the specified name or code already exists.
* `INSUFFICIENT_STOCK` (`422`): Requested stock issue quantity exceeds available warehouse balance.
* `INACTIVE_RESOURCE` (`422`): Target warehouse is `INACTIVE` or target product is `ARCHIVED`.

---

## 3. Detailed Endpoint Specifications

### 3.1. Authentication Module (`Auth`)

#### 1. User Sign-In (`POST /api/auth/login`)
* **Authorization:** Public
* **Validation Rules:**
  - `email`: Valid email format, required.
  - `password`: Non-empty string (minimum 6 characters), required.
* **Request Body:**
  ```json
  {
    "email": "operator@depoma.ir",
    "password": "Password123!"
  }
  ```
* **Successful Response (`200 OK`):**
  ```json
  {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "u1a2b3c4-d5e6-4f7a-8b9c-0d1e2f3a4b5c",
      "fullName": "علی رضایی",
      "email": "operator@depoma.ir",
      "role": "OPERATOR",
      "assignedWarehouseIds": [
        "w1000000-0000-4000-8000-000000000001"
      ]
    }
  }
  ```
* **Error Responses:** `400 Bad Request`, `401 Unauthorized` (`AUTH_INVALID_CREDENTIALS`), `403 Forbidden` (account `INACTIVE`).

#### 2. Retrieve Current Session Profile (`GET /api/auth/me`)
* **Authorization:** `ADMIN`, `OPERATOR`, `MANAGER`
* **Successful Response (`200 OK`):** Returns the authenticated user's profile and an array of `assignedWarehouses` (`[{ id, name, code }]`).

---

### 3.2. Catalog Module (`Categories` & `Products`)

#### 1. List Products (`GET /api/products`)
* **Authorization:** `ADMIN`, `OPERATOR`, `MANAGER`
* **Query Parameters:** `?categoryId=&search=&status=ACTIVE&page=1&limit=10`
* **Successful Response (`200 OK`):**
  ```json
  {
    "items": [
      {
        "id": "p1000000-0000-4000-8000-000000000001",
        "name": "مودم روتر فیبر نوری هواوی HG8245H",
        "sku": "NET-HW-8245",
        "categoryId": "c1000000-0000-4000-8000-000000000001",
        "categoryName": "تجهیزات شبکه",
        "unit": "عدد",
        "minimumStock": 10,
        "status": "ACTIVE",
        "createdAt": "2026-11-08T18:00:00.000Z"
      }
    ],
    "meta": {
      "total": 48,
      "page": 1,
      "limit": 10,
      "totalPages": 5
    }
  }
  ```

#### 2. Get Product Details (`GET /api/products/:id`)
* **Authorization:** `ADMIN`, `OPERATOR`, `MANAGER`
* **Architectural Boundary Note:** Returns only product and category master data (`CatalogModule`). To prevent circular dependencies between `CatalogModule` and `InventoryModule`, per-warehouse stock balances for this product are queried separately via `GET /api/inventory/balances?productId=:id` (which enforces warehouse-level operator access).

#### 3. Create Product (`POST /api/products`)
* **Authorization:** `ADMIN` only
* **Validation Rules:**
  - `name`: String, 2–150 characters, required.
  - `sku`: String, 2–64 characters, alphanumeric and hyphens, unique, required.
  - `categoryId`: Valid UUID referencing an existing category, required.
  - `unit`: String, 1–30 characters (e.g., `"عدد"`, `"بسته"`, `"کیلوگرم"`), required.
  - `minimumStock`: Integer `>= 0`, defaults to `0`.
* **Request Body:**
  ```json
  {
    "name": "مودم روتر فیبر نوری هواوی HG8245H",
    "sku": "NET-HW-8245",
    "categoryId": "c1000000-0000-4000-8000-000000000001",
    "unit": "عدد",
    "minimumStock": 10,
    "description": "مودم ۴ پورت گیگابیت دو آنتن"
  }
  ```
* **Successful Response (`201 Created`):**
  ```json
  {
    "id": "p1000000-0000-4000-8000-000000000001",
    "name": "مودم روتر فیبر نوری هواوی HG8245H",
    "sku": "NET-HW-8245",
    "categoryId": "c1000000-0000-4000-8000-000000000001",
    "categoryName": "تجهیزات شبکه",
    "unit": "عدد",
    "minimumStock": 10,
    "status": "ACTIVE",
    "createdAt": "2026-11-08T18:00:00.000Z"
  }
  ```

---

### 3.3. Warehouses & Operator Assignment Module (`Warehouses`)

#### 1. List Warehouses (`GET /api/warehouses`)
* **Authorization:** `ADMIN`, `MANAGER`, `OPERATOR`
* **Access Scoping Rule:**
  - `ADMIN` and `MANAGER` receive all warehouses matching the optional `?status=` filter.
  - `OPERATOR` receives **only warehouses assigned to their `userId` in `warehouse_operators`**.

#### 2. Assign Operator to Warehouse (`POST /api/warehouses/:id/operators`)
* **Authorization:** `ADMIN` only
* **Request Body:**
  ```json
  {
    "userId": "u1a2b3c4-d5e6-4f7a-8b9c-0d1e2f3a4b5c"
  }
  ```
* **Successful Response (`201 Created`):**
  ```json
  {
    "id": "wo100000-0000-4000-8000-000000000001",
    "warehouseId": "w1000000-0000-4000-8000-000000000001",
    "userId": "u1a2b3c4-d5e6-4f7a-8b9c-0d1e2f3a4b5c",
    "userFullName": "علی رضایی",
    "assignedBy": "admin000-0000-4000-8000-000000000001",
    "assignedAt": "2026-11-08T18:10:00.000Z"
  }
  ```

---

### 3.4. Inventory & Stock Ledger Module (`Inventory`)

#### 1. Query Inventory Balances (`GET /api/inventory/balances`)
* **Authorization:** `ADMIN`, `MANAGER`, `OPERATOR` (Scoped to assigned warehouses)
* **Query Parameters:**
  - `warehouseId` (UUID, optional): Filters balances for a specific warehouse. If an `OPERATOR` supplies an unassigned `warehouseId`, the server responds with `403 Forbidden`.
  - `productId` (UUID, optional): Filters balances for a specific product.
  - `lowStockOnly` (`boolean`, optional): When `true`, returns only items where `quantity <= product.minimumStock`.
* **Successful Response (`200 OK`):**
  ```json
  {
    "items": [
      {
        "balanceId": "b1000000-0000-4000-8000-000000000001",
        "warehouse": {
          "id": "w1000000-0000-4000-8000-000000000001",
          "name": "انبار مرکزی تهران",
          "code": "WH-THR-01"
        },
        "product": {
          "id": "p1000000-0000-4000-8000-000000000001",
          "name": "مودم روتر فیبر نوری هواوی HG8245H",
          "sku": "NET-HW-8245",
          "unit": "عدد",
          "minimumStock": 10
        },
        "quantity": 12,
        "isLowStock": false,
        "updatedAt": "2026-11-08T18:20:00.000Z"
      }
    ]
  }
  ```

#### 2. Record Stock Receipt — Stock In (`POST /api/inventory/receipts`)
* **Authorization:** `ADMIN`, `OPERATOR` (Scoped to assigned warehouses)
* **Validation Rules:**
  - `warehouseId`: Valid UUID, required.
  - `productId`: Valid UUID, required.
  - `quantity`: Positive integer (`>= 1`), required.
  - `reason`: String, 2–120 characters, required.
  - `referenceNo`: Optional string up to 64 characters.
  - `description`: Optional text.
* **Request Body:**
  ```json
  {
    "warehouseId": "w1000000-0000-4000-8000-000000000001",
    "productId": "p1000000-0000-4000-8000-000000000001",
    "quantity": 50,
    "reason": "خرید از تأمینکننده",
    "referenceNo": "FACTOR-405-112",
    "description": "محموله اول آبانماه تحویل گرفته شد"
  }
  ```
* **Successful Response (`201 Created`):**
  ```json
  {
    "movement": {
      "id": "m1000000-0000-4000-8000-000000000001",
      "movementType": "IN",
      "warehouseId": "w1000000-0000-4000-8000-000000000001",
      "productId": "p1000000-0000-4000-8000-000000000001",
      "quantity": 50,
      "balanceAfter": 62,
      "reason": "خرید از تأمینکننده",
      "referenceNo": "FACTOR-405-112",
      "createdBy": {
        "id": "u1a2b3c4-d5e6-4f7a-8b9c-0d1e2f3a4b5c",
        "fullName": "علی رضایی"
      },
      "createdAt": "2026-11-08T18:25:00.000Z"
    },
    "currentBalance": 62
  }
  ```

#### 3. Record Stock Issue — Stock Out (`POST /api/inventory/issues`)
* **Authorization:** `ADMIN`, `OPERATOR` (Scoped to assigned warehouses). `MANAGER` is rejected with `403 Forbidden`.
* **Business & Transactional Rules:**
  1. Opens an interactive database transaction (`prisma.$transaction`).
  2. Verifies inside the transaction that the authenticated `OPERATOR` is assigned to `warehouseId` and that both the target warehouse and product have `status === 'ACTIVE'` (`SELECT ... FOR SHARE`, throwing `422 INACTIVE_RESOURCE` if deactivated/archived).
  3. Acquires a pessimistic `SELECT ... FOR UPDATE` row lock on `inventory_balances` via `tx.$queryRaw` and verifies `availableQuantity >= quantity`.
  4. If `availableQuantity < quantity`, rolls back the transaction and returns `422 Unprocessable Entity` (`INSUFFICIENT_STOCK`).
  5. Deducts `quantity` from `inventory_balances` and appends an `OUT` row to `stock_movements` atomically.
* **Request Body:**
  ```json
  {
    "warehouseId": "w1000000-0000-4000-8000-000000000001",
    "productId": "p1000000-0000-4000-8000-000000000001",
    "quantity": 5,
    "reason": "حواله فروش به مشتری",
    "referenceNo": "OUT-405-309",
    "description": "ارسال برای سفارش شماره ۳۰۹"
  }
  ```
* **Successful Response (`201 Created`):**
  ```json
  {
    "movement": {
      "id": "m1000000-0000-4000-8000-000000000002",
      "movementType": "OUT",
      "warehouseId": "w1000000-0000-4000-8000-000000000001",
      "productId": "p1000000-0000-4000-8000-000000000001",
      "quantity": 5,
      "balanceAfter": 57,
      "reason": "حواله فروش به مشتری",
      "referenceNo": "OUT-405-309",
      "createdBy": {
        "id": "u1a2b3c4-d5e6-4f7a-8b9c-0d1e2f3a4b5c",
        "fullName": "علی رضایی"
      },
      "createdAt": "2026-11-08T18:30:00.000Z"
    },
    "currentBalance": 57
  }
  ```

#### 4. Query Stock Movement History (`GET /api/inventory/movements`)
* **Authorization:** `ADMIN`, `MANAGER`, `OPERATOR` (Scoped to assigned warehouses)
* **Query Parameters:** `?warehouseId=&productId=&movementType=IN|OUT&page=1&limit=10`
* **Successful Response (`200 OK`):**
  ```json
  {
    "items": [
      {
        "id": "m1000000-0000-4000-8000-000000000002",
        "movementType": "OUT",
        "warehouse": {
          "id": "w1000000-0000-4000-8000-000000000001",
          "name": "انبار مرکزی تهران",
          "code": "WH-THR-01"
        },
        "product": {
          "id": "p1000000-0000-4000-8000-000000000001",
          "name": "مودم روتر فیبر نوری هواوی HG8245H",
          "sku": "NET-HW-8245",
          "unit": "عدد"
        },
        "quantity": 5,
        "balanceAfter": 57,
        "reason": "حواله فروش به مشتری",
        "referenceNo": "OUT-405-309",
        "createdBy": {
          "id": "u1a2b3c4-d5e6-4f7a-8b9c-0d1e2f3a4b5c",
          "fullName": "علی رضایی"
        },
        "createdAt": "2026-11-08T18:30:00.000Z"
      }
    ],
    "meta": {
      "total": 100,
      "page": 1,
      "limit": 10,
      "totalPages": 10
    }
  }
  ```

#### 5. Duplicate Submission Protection & Deferred Adjustment Note
* **Duplicate Submission Protection (v1.0 MVP):** In the frontend, form submit buttons are immediately disabled (`mutation.isPending === true`) while a `POST /api/inventory/receipts` or `POST /api/inventory/issues` request is in flight. Because automatic network-level retries on unstable connections could theoretically submit a movement twice, automatic client retry on `POST` mutations is disabled (`retry: false` in TanStack Query). Full server-side `Idempotency-Key` header deduplication is documented as a future commercial enhancement.
* **Deferred Stock Adjustment Endpoint (`POST /api/inventory/adjustments`):** Deferred out of the v1.0 MVP scope to avoid unsigned quantity ambiguity in the ledger. In v1.0, stocktaking increases and decreases are recorded via `POST /api/inventory/receipts` (`IN`) and `POST /api/inventory/issues` (`OUT`) with explicit stocktaking reasons. Future releases introducing a dedicated adjustment endpoint will use directional movement types (`ADJUSTMENT_IN` and `ADJUSTMENT_OUT`).

---

### 3.5. Dashboard Module (`Dashboard`)

#### 1. Retrieve Operational Summary (`GET /api/dashboard/summary`)
* **Authorization:** `ADMIN`, `MANAGER`, `OPERATOR` (Scoped)
* **Successful Response (`200 OK`):**
  ```json
  {
    "totalActiveProducts": 48,
    "totalAccessibleWarehouses": 2,
    "lowStockItemsCount": 3,
    "todayMovementsCount": 14,
    "recentMovements": [
      {
        "id": "m1000000-0000-4000-8000-000000000002",
        "movementType": "OUT",
        "productName": "مودم روتر فیبر نوری هواوی HG8245H",
        "productSku": "NET-HW-8245",
        "warehouseName": "انبار مرکزی تهران",
        "quantity": 5,
        "unit": "عدد",
        "createdByFullName": "علی رضایی",
        "createdAt": "2026-11-08T18:30:00.000Z"
      }
    ]
  }
  ```
