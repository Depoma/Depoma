# User Stories & Acceptance Criteria

**Product Name:** Depoma — Inventory & Stock Ledger Management System  
**Document Version:** 1.0  
**Format:** `As a [User Role], I want [Action/Goal], So that [Business Benefit].`  

All user stories for Depoma are prioritized using the **MoSCoW** framework (**Must Have**, **Should Have**, **Could Have**). Each story includes observable, testable acceptance criteria covering positive execution paths, validation failures, and role/ownership security boundaries.

---

## 1. Story Priority Matrix

| Story ID | Title | Primary Role | Priority | Domain Module |
| :---: | :--- | :---: | :---: | :---: |
| **US-01** | User Authentication & Session Termination | All Roles | **Must Have** | `Auth` |
| **US-02** | Product & Category Catalog Management | `ADMIN` | **Must Have** | `Catalog` |
| **US-03** | Warehouse Location Management | `ADMIN` | **Must Have** | `Warehouses` |
| **US-04** | User Provisioning & Warehouse Operator Assignment | `ADMIN` | **Must Have** | `Users` / `Warehouses` |
| **US-05** | Record Stock Receipt (Stock In) | `OPERATOR` / `ADMIN` | **Must Have** | `Inventory` |
| **US-06** | **Record Stock Issue with Balance Enforcement (Stock Out)** | `OPERATOR` / `ADMIN` | **Must Have** | `Inventory` |
| **US-07** | View Warehouse Inventory Balances | All Roles (Scoped) | **Must Have** | `Inventory` |
| **US-08** | View & Filter Immutable Stock Movement History | All Roles (Scoped) | **Must Have** | `Inventory` |
| **US-09** | Operational Summary Dashboard | All Roles (Scoped) | **Must Have** | `Dashboard` |
| **US-10** | Low-Stock Alert Filtering | `MANAGER` / `ADMIN` | **Should Have** | `Inventory` |
| **US-11** | Product Catalog Search & Filtering | All Roles | **Should Have** | `Catalog` |
| **US-12** | Dedicated Stocktaking Adjustment (`ADJUSTMENT_IN` / `ADJUSTMENT_OUT`) | `ADMIN` | **Could Have (Deferred)** | `Inventory` |
| **US-13** | Direct Single-Step Inter-Warehouse Stock Transfer | `OPERATOR` / `ADMIN` | **Could Have (Deferred)** | `Inventory` |
| **US-14** | Excel & PDF Inventory Report Exports | `MANAGER` / `ADMIN` | **Could Have (Deferred)** | `Reports` |
| **US-15** | Supplier & Customer Order Integration | `ADMIN` | **Could Have (Deferred)** | Out of v1.0 Scope |

---

## 2. Must Have User Stories (Core v1.0 Scope)

### US-01: User Authentication & Session Termination
> **User Story:**  
> **As an** authorized user (`ADMIN`, `OPERATOR`, or `MANAGER`),  
> **I want** to sign in with my email and password and sign out when my shift ends,  
> **So that** my identity and access permissions are verified and unauthorized parties cannot access company inventory records.

#### Acceptance Criteria:
* **Success Scenarios:**
  1. Submitting valid credentials for an `ACTIVE` user to `POST /api/auth/login` returns a signed JWT access token along with the user's profile, role, and assigned warehouse IDs.
  2. Authenticated requests to `GET /api/auth/me` return the current user's identity and assigned warehouses, maintaining session continuity across browser refreshes.
  3. Triggering sign-out (`POST /api/auth/logout`) clears the client session token and redirects the user to the Persian login screen.
* **Failure & Security Scenarios:**
  1. Submitting an unknown email or incorrect password returns `401 Unauthorized` with a localized Persian error message (`AUTH_INVALID_CREDENTIALS`).
  2. Attempting to sign in with an account marked `INACTIVE` returns `403 Forbidden`.
  3. Sending any request to a protected API endpoint without a valid JWT token returns `401 Unauthorized`.

---

### US-02: Product & Category Catalog Management
> **User Story:**  
> **As an** `ADMIN`,  
> **I want** to create and update product categories and product items with a unique SKU, unit of measure, and minimum stock threshold,  
> **So that** the organization maintains a standardized catalog of goods tracked across all warehouses.

#### Acceptance Criteria:
* **Success Scenarios:**
  1. An `ADMIN` can create a category (`POST /api/categories`) and edit its name or description (`PATCH /api/categories/:id`).
  2. An `ADMIN` can create a product (`POST /api/products`) specifying `name`, unique `sku`, `categoryId`, `unit` (e.g., Piece, Box, Kg), `minimumStock` (`>= 0`), and optional `description`.
  3. Created products persist in PostgreSQL and are immediately visible in the catalog list (`GET /api/products`) for all authenticated users.
  4. An `ADMIN` can update product details or transition a product's status to `ARCHIVED` (`PATCH /api/products/:id`) so it can no longer receive new stock movements while preserving historical ledger integrity.
* **Failure & Permission Scenarios:**
  1. Creating or updating a product with a duplicate `sku` is rejected with `409 Conflict` (`DUPLICATE_SKU`).
  2. Submitting a negative `minimumStock` or empty required fields is rejected with `400 Bad Request`.
  3. Requests from `OPERATOR` or `MANAGER` roles to create or modify categories or products are rejected by the backend with `403 Forbidden`.

---

### US-03: Warehouse Location Management
> **User Story:**  
> **As an** `ADMIN`,  
> **I want** to create, view, and update physical warehouse locations,  
> **So that** inventory balances and movements are isolated and tracked per storage facility.

#### Acceptance Criteria:
* **Success Scenarios:**
  1. An `ADMIN` can create a new warehouse (`POST /api/warehouses`) with a unique `name`, unique short `code` (e.g., `WH-THR-01`), physical `address`, and `description`.
  2. An `ADMIN` can update warehouse attributes or set its status to `INACTIVE` (`PATCH /api/warehouses/:id`).
  3. `ADMIN` and `MANAGER` users can view all warehouses, whereas `OPERATOR` users calling `GET /api/warehouses` only receive warehouses explicitly assigned to them.
* **Failure & Permission Scenarios:**
  1. Creating a warehouse with a duplicate `name` or `code` returns `409 Conflict` (`DUPLICATE_WAREHOUSE_CODE`).
  2. Any attempt by an `OPERATOR` or `MANAGER` to create or edit a warehouse is rejected with `403 Forbidden`.

---

### US-04: User Provisioning & Warehouse Operator Assignment
> **User Story:**  
> **As an** `ADMIN`,  
> **I want** to manage user accounts and assign specific Warehouse Operators to designated warehouses,  
> **So that** each operator can only view and execute stock operations within the facilities they are responsible for.

#### Acceptance Criteria:
* **Success Scenarios:**
  1. An `ADMIN` can create a new user (`POST /api/users`) with a designated role (`ADMIN`, `OPERATOR`, or `MANAGER`) and update their status (`PATCH /api/users/:id`).
  2. An `ADMIN` can assign an `OPERATOR` to a warehouse (`POST /api/warehouses/:id/operators`), creating a record in `warehouse_operators` capturing `warehouse_id`, `user_id`, `assigned_by`, and `assigned_at`.
  3. An `ADMIN` can revoke an operator's access to a warehouse (`DELETE /api/warehouses/:id/operators/:userId`), immediately terminating their permission to access that facility.
* **Failure & Permission Scenarios:**
  1. Assigning an operator to the same warehouse twice is prevented by a composite unique constraint and returns `409 Conflict`.
  2. Non-admin users attempting to manage users or modify warehouse operator assignments receive `403 Forbidden`.

---

### US-05: Record Stock Receipt (Stock In)
> **User Story:**  
> **As a** Warehouse Operator (or Admin),  
> **I want** to record a stock receipt (`Stock In`) for a product into an authorized warehouse,  
> **So that** incoming shipments immediately increase available warehouse stock and generate an immutable audit trail.

#### Acceptance Criteria:
* **Success Scenarios:**
  1. An authorized user submits `POST /api/inventory/receipts` with `warehouseId`, `productId`, `quantity` (positive integer `>= 1`), `reason`, and optional `referenceNo` and `description`.
  2. Within a single ACID database transaction, the backend:
     - Upserts the `inventory_balances` record for `(product_id, warehouse_id)`, incrementing `quantity` by the received amount.
     - Inserts an immutable `IN` record into `stock_movements` recording `quantity`, the resulting `balance_after`, `created_by`, and UTC `created_at`.
  3. The updated balance and movement record are immediately returned and persisted.
* **Failure & Ownership Scenarios:**
  1. **Unauthorized Warehouse Access:** If an `OPERATOR` submits a `warehouseId` not present in their `warehouse_operators` assignments, the backend rejects the request with `403 Forbidden` (`FORBIDDEN_WAREHOUSE_ACCESS`).
  2. **Role Restriction:** If a `MANAGER` attempts to record a receipt, the backend rejects the request with `403 Forbidden`.
  3. **Validation & State Check:** Submitting `quantity <= 0`, non-integer quantities, or targeting an `INACTIVE` warehouse or `ARCHIVED` product fails with `400 Bad Request` or `422 Unprocessable Entity` (`INACTIVE_RESOURCE`).

---

### US-06: Record Stock Issue with Balance Enforcement (Stock Out — Core Workflow)
> **User Story:**  
> **As a** Warehouse Operator (or Admin),  
> **I want** to record a stock issue (`Stock Out`) from an authorized warehouse with automatic availability validation,  
> **So that** outgoing inventory is accurately deducted and warehouse stock never drops below zero.

#### Acceptance Criteria:
* **Success Scenarios:**
  1. An authorized user submits `POST /api/inventory/issues` with `warehouseId`, `productId`, `quantity` (`>= 1`), `reason`, `referenceNo`, and `description`.
  2. The backend opens an interactive transaction, locks the corresponding `inventory_balances` row (`SELECT ... FOR UPDATE`), verifies `available_quantity >= requested_quantity`, deducts the quantity, and inserts an `OUT` record into `stock_movements` with the updated `balance_after`.
  3. The UI displays a confirmation banner, updates the product's balance in the table, and reflects the low-stock badge if the new balance falls at or below `minimum_stock`.
* **Failure, Concurrency & Ownership Scenarios:**
  1. **Insufficient Stock Prevention (`422`):** If a warehouse holds `12` units of a product and the user requests an issue of `20` units, the transaction rolls back completely, no movement record is created, the balance remains `12`, and the API returns `422 Unprocessable Entity` (`INSUFFICIENT_STOCK`) with the current available quantity.
  2. **Unauthorized Warehouse Access (`403`):** If an `OPERATOR` attempts to issue stock from an unassigned warehouse (even via direct API manipulation), the request is blocked with `403 Forbidden`.
  3. **Read-Only Role Rejection (`403`):** If a `MANAGER` attempts to issue stock, the request is blocked with `403 Forbidden`.
  4. **Concurrent Request Protection:** If two operators simultaneously request to issue `8` units of a product with an available balance of `10` units, row-level locking ensures the first transaction succeeds (leaving `2` units) and the second transaction fails cleanly with `422 INSUFFICIENT_STOCK`.

---

### US-07: View Warehouse Inventory Balances
> **User Story:**  
> **As a** system user (`ADMIN`, `MANAGER`, or `OPERATOR`),  
> **I want** to view current product stock balances per warehouse,  
> **So that** I have accurate visibility into available inventory across authorized storage locations.

#### Acceptance Criteria:
* **Success Scenarios:**
  1. `ADMIN` and `MANAGER` users can view balances across all warehouses and filter by `warehouseId`, `productId`, or `lowStockOnly`.
  2. `OPERATOR` users only see stock balances for warehouses assigned to them in `warehouse_operators`.
  3. Each balance item indicates whether `quantity <= minimum_stock` (`isLowStock: true`) to highlight replenishment needs.
* **Failure & Security Scenarios:**
  1. If an `OPERATOR` passes a `warehouseId` query parameter for an unassigned warehouse to `GET /api/inventory/balances`, the backend rejects the request with `403 Forbidden`.

---

### US-08: View & Filter Immutable Stock Movement History (Ledger)
> **User Story:**  
> **As a** Manager, Admin, or Operator,  
> **I want** to inspect and filter the chronological history of all stock movements,  
> **So that** every inventory addition or deduction can be audited against a user, timestamp, voucher reference, and resulting balance.

#### Acceptance Criteria:
* **Success Scenarios:**
  1. Calling `GET /api/inventory/movements` returns paginated ledger entries ordered by `created_at DESC`, including product details, warehouse name, `movementType` (`IN`, `OUT`), `quantity`, `balanceAfter`, `reason`, `referenceNo`, and creator name.
  2. Users can filter ledger records by `warehouseId`, `productId`, and `movementType`.
  3. In the frontend, UTC timestamps are formatted into Jalali date and time (`YYYY/MM/DD - HH:mm`).
* **Security & Integrity Scenarios:**
  1. `OPERATOR` queries are strictly restricted to movements occurring within their assigned warehouses (`403 Forbidden` if querying an unauthorized warehouse).
  2. Ledger entries are strictly append-only; no API endpoint exists to mutate or delete historical records in `stock_movements`.

---

### US-09: Operational Summary Dashboard
> **User Story:**  
> **As an** authenticated user,  
> **I want** to view an operational summary dashboard upon signing in,  
> **So that** I can immediately monitor key inventory metrics, low-stock warnings, and recent warehouse activity.

#### Acceptance Criteria:
* **Success Scenarios:**
  1. Calling `GET /api/dashboard/summary` returns: total active products, accessible active warehouses count, count of product-warehouse balances at or below `minimum_stock`, today's movement count, and the latest 10 stock movements.
  2. For `OPERATOR` users, warehouse counts, low-stock alerts, and recent movements are automatically scoped to their assigned warehouses.

---

## 3. Should Have User Stories (Secondary Enhancements)

### US-10: Low-Stock Alert Filtering
> **As a** `MANAGER` or `ADMIN`, **I want** to filter inventory balances to show only items at or below their minimum stock threshold, **So that** I can prioritize procurement and prevent stockouts.
* **Acceptance Criteria:** Passing `?lowStockOnly=true` to `GET /api/inventory/balances` returns only records where `inventory_balances.quantity <= products.minimum_stock`.

### US-11: Product Catalog Search & Filtering
> **As a** user, **I want** to search products by Persian name or SKU code and filter by category, **So that** I can quickly locate items in a large catalog.
* **Acceptance Criteria:** Passing `?search=<query>&categoryId=<uuid>` to `GET /api/products` performs a case-insensitive match on `name` and `sku` and returns a standardized paginated envelope `{ items, meta }`.

---

## 4. Could Have User Stories (Deferred Beyond v1.0 MVP)

### US-12: Dedicated Stocktaking Adjustment (`ADJUSTMENT_IN` / `ADJUSTMENT_OUT`) — *[Deferred from v1.0 MVP]*
> **As an** `ADMIN`, **I want** a dedicated stocktaking reconciliation endpoint when physical counts differ from system balances, **So that** stock discrepancies are categorized separately from routine receipts and issues.
* **MVP Scope Decision:** Deferred out of the v1.0 MVP to prevent directional ambiguity in the ledger. In v1.0, positive and negative stocktaking adjustments are recorded using the existing `Receipt (IN)` and `Issue (OUT)` endpoints with explicit reasons (e.g., `"اضافه انبارگردانی"` / `"کسری انبارگردانی"`). When implemented in a future release, directional movement types (`ADJUSTMENT_IN` and `ADJUSTMENT_OUT`) will be used so `CHECK (quantity > 0)` and the ledger sum formula remain unambiguous.

### US-13: Direct Single-Step Inter-Warehouse Transfer
> **As an** `ADMIN`, **I want** to transfer stock from a source warehouse to a destination warehouse in a single atomic operation, **So that** internal stock relocations require fewer manual steps.

### US-14: Excel & PDF Inventory Report Exports
> **As a** `MANAGER`, **I want** to export filtered inventory balances and movement history to Excel (`.xlsx`) and PDF formats, **So that** audit reports can be archived and shared externally.

### US-15: Supplier & Customer Order Integration
> **As an** `ADMIN`, **I want** to link stock receipts to purchase orders and stock issues to sales invoices, **So that** warehouse operations synchronize directly with commercial billing.
