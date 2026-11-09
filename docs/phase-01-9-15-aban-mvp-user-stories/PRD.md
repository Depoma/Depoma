# Product Requirements Document (PRD)

**Product Name:** Depoma — Inventory & Stock Ledger Management System  
**Document Version:** 1.0  
**Status:** Approved  

---

## 1. Product Overview

**Depoma** is a web-based Inventory Management System (IMS) built for small and medium-sized enterprises (SMEs). It provides centralized control over product catalogs, multi-warehouse storage locations, stock receipts, stock issues, and historical movement auditing.

Unlike ad-hoc spreadsheet tracking or legacy tools where stock counts are overwritten in place without accountability, Depoma is architected around an **Immutable Inventory Ledger (Stock Movement)** model. Every change to stock quantity is recorded as an append-only ledger entry (`IN` or `OUT`), while materialized warehouse balances are updated atomically within the same database transaction.

Depoma is initially released as a focused single-organization system (v1.0 MVP) with a modular architecture designed to evolve into a multi-tenant Software-as-a-Service (SaaS) platform and an on-premise enterprise offering.

---

## 2. Problem Statement

Small and growing businesses—such as e-commerce merchants, distribution companies, light manufacturers, and retail chains—frequently rely on spreadsheets or disconnected paper vouchers to track warehouse stock. This creates critical operational failures:

1. **Lack of Auditability:** When stock counts are manually edited in a spreadsheet, there is no verifiable record of who changed the quantity, when the change occurred, which voucher or invoice authorized it, or why the stock moved.
2. **Stockouts and Negative Balances:** Without real-time availability validation during order fulfillment or stock issuance, operators routinely issue more items than are physically available, resulting in unfulfilled orders and negative inventory discrepancies.
3. **Unrestricted Multi-Warehouse Access:** Businesses operating multiple storage locations (e.g., a central distribution hub and regional branch warehouses) lack granular access controls, allowing operators assigned to one location to accidentally modify stock in another.
4. **Concurrent Update Conflicts (Race Conditions):** When multiple warehouse operators record movements simultaneously in shared files, concurrent edits overwrite one another and corrupt stock balances.
5. **Delayed Operational Visibility:** Managers lack an immediate, consolidated view of active SKU counts, low-stock threshold breaches, and recent warehouse activity.

---

## 3. Target Market & Localization Strategy

* **Primary Target Market:** Small and medium-sized enterprises operating in Iran with 1 to 10 warehouses and 2 to 50 staff members.
* **Localization Requirements:**
  - The user interface is localized in **Persian (Farsi)** with native **Right-to-Left (RTL)** layout support, Persian typography (`Vazirmatn`), and Persian numeral formatting.
  - All dates and timestamps are displayed in the **Jalali (Solar Hijri)** calendar in the presentation layer.
  - To guarantee data integrity, sorting accuracy, and future multi-language (LTR) extensibility, all timestamps are stored and exchanged at the database and API layers in standard **UTC (`ISO 8601` / `TIMESTAMPTZ`)**.

---

## 4. User Personas & Access Control Roles

Depoma enforces strict server-side Role-Based Access Control (RBAC) combined with resource-level warehouse ownership verification:

| Role Identifier | Role Title | Responsibilities & Access Scope |
| :--- | :--- | :--- |
| **`ADMIN`** | System Administrator | • Full administrative access across the system.<br>• Create, view, and update user accounts and role assignments.<br>• Create and manage product categories and product definitions (SKUs).<br>• Create and manage warehouse locations.<br>• **Assign Warehouse Operators to specific warehouses (`Warehouse_Operators`) and revoke access.**<br>• View balances and ledger history across all warehouses.<br>• Record stock receipts (`Stock In`) and stock issues (`Stock Out`) in any active warehouse. |
| **`OPERATOR`** | Warehouse Operator | • Read-only access to the product catalog and categories.<br>• **View and operate strictly within warehouses explicitly assigned to them by an Admin.**<br>• Record **Stock In (Receipts)** and **Stock Out (Issues)** for their assigned warehouses.<br>• View current stock balances and movement history exclusively for their assigned warehouses.<br>• No access to user management, catalog modification, warehouse configuration, or unassigned warehouses. |
| **`MANAGER`** | Operations Manager / Auditor | • Organization-wide **read-only** supervisory access.<br>• View the executive dashboard, low-stock alerts, and warehouse summaries.<br>• Inspect current stock balances across all warehouses.<br>• Audit and filter the complete stock movement ledger across all locations.<br>• Prohibited from recording stock movements or modifying master data. |

---

## 5. Product Goals & Non-Goals

### 5.1. Primary Goals (v1.0 MVP)
1. **End-to-End Operational Integrity:** Deliver a complete, production-grade workflow covering authentication, catalog setup, warehouse assignment, stock receipt/issue execution, and ledger auditing.
2. **Mathematical & Transactional Accuracy:** Guarantee that `Inventory_Balances` always equal the net sum of `Stock_Movements` using ACID database transactions and pessimistic row-level locking, completely preventing negative stock and race conditions.
3. **Server-Side Security Enforcement:** Enforce authentication, role authorization, and warehouse-level operator restrictions on every API endpoint.
4. **High-Efficiency RTL User Experience:** Provide a responsive, keyboard-friendly Persian RTL interface tailored for daily B2B back-office operations.

### 5.2. Non-Goals (Explicitly Out of Scope for v1.0)
To maintain engineering focus and release reliability, the following capabilities are excluded from v1.0 and deferred to future releases:
* Single-step inter-warehouse transfers (handled in v1.0 via a Stock Out from the source warehouse followed by a Stock In at the destination).
* Customer Relationship Management (CRM), supplier portals, purchase orders, sales invoicing, and accounting ledger integration.
* Hardware barcode/RFID scanner integration and label printing.
* Batch/Lot number tracking, serial number tracking, and expiration date management (FEFO/FIFO).
* Sub-warehouse bin, aisle, and shelf location management.
* Native iOS/Android mobile applications or desktop binaries.
* Active multi-tenant database partitioning (architected for future expansion, but deployed as single-tenant per instance in v1.0).

---

## 6. Feature Scope & Prioritization (MoSCoW)

### 6.1. Must Have (Core v1.0 Release Scope)
1. **Authentication & Session Management:**
   - Email and password sign-in (`POST /api/auth/login`), sign-out (`POST /api/auth/logout`), and current session profile retrieval (`GET /api/auth/me`).
   - JWT-based authentication and server-side RBAC/ownership guards.
2. **User Management & Warehouse Assignment:**
   - User account creation, listing, and status management by `ADMIN`.
   - Assigning and revoking `OPERATOR` access to specific warehouses (`Warehouse_Operators`).
3. **Product & Category Catalog:**
   - Category creation, listing, and editing.
   - Product creation, listing, detail inspection, and editing (including unique `SKU`, measurement unit, `minimum_stock` alert threshold, and `ACTIVE`/`ARCHIVED` status).
4. **Warehouse Management:**
   - Creating, listing, inspecting, and updating physical warehouse records (`name`, unique `code`, `address`, `status`).
5. **Inventory Balances & Stock Movement Engine:**
   - Recording **Stock In (Receipts)** to increase warehouse balance and append an `IN` ledger entry.
   - Recording **Stock Out (Issues)** with real-time balance verification, atomic deduction, and an `OUT` ledger entry.
   - Strict server-side prevention of stock issues exceeding available warehouse balance (`422 Unprocessable Entity`).
   - Viewing current stock balances per product and warehouse.
6. **Immutable Movement History (Audit Ledger):**
   - Chronological log of all stock movements displaying timestamp (Jalali in UI), product, SKU, warehouse, movement type, quantity delta, resulting balance (`balance_after`), reason, reference number, and creator identity.
   - Filtering movements by warehouse, product, and movement type.
7. **Operational Dashboard:**
   - High-level KPI summary cards (total active products, accessible active warehouses, low-stock item count, today's movement count) and a feed of recent stock movements.

### 6.2. Should Have (Secondary v1.0 Enhancements)
1. **Low-Stock Alert Filtering:** Dedicated filter on the inventory balances view to isolate items where `quantity <= minimum_stock`.
2. **Catalog Search & Pagination:** Server-side search by product name or SKU code with standardized pagination metadata.

### 6.3. Could Have (Future Roadmap Backlog — Deferred Beyond v1.0 MVP)
1. **Dedicated Stock Adjustments (`ADJUSTMENT_IN` / `ADJUSTMENT_OUT`):** Dedicated physical stocktaking reconciliation endpoint (`POST /api/inventory/adjustments`). *(Deferred out of v1.0 MVP to preserve ledger simplicity; in v1.0, stocktaking surpluses and deficits are recorded using standard `Receipt (IN)` and `Issue (OUT)` operations with dedicated adjustment reasons).*
2. **Direct Inter-Warehouse Transfers (`TRANSFER_IN` / `TRANSFER_OUT`).**
3. **CSV / Excel / PDF Report Exports.**
4. **Multi-Tenant SaaS Onboarding & Organization Isolation.**

---

## 7. Core Business Workflow (Primary User Journey)

The critical end-to-end workflow governing system design and verification is **Recording a Warehouse Stock Issue (Stock Out)**:

1. **Authentication:** A Warehouse Operator signs into Depoma using valid credentials.
2. **Context Initialization:** The system validates the operator's identity, loads their profile and assigned warehouses, and renders the RTL dashboard.
3. **Warehouse & Balance Inspection:** The operator navigates to the Inventory view, selects an assigned warehouse, and reviews the available stock balance for a target product (e.g., `12 units` available).
4. **Movement Submission:** The operator opens the **Stock Out (Issue)** form and submits:
   - Source Warehouse ID
   - Product ID
   - Issue Quantity (positive integer)
   - Movement Reason (e.g., Customer Order Fulfillment, Internal Consumption)
   - Reference Document Number and optional notes
5. **Backend Processing & Atomic Execution:**
   - **Step 5.1 (Authentication):** Validates the JWT session token and performs a live database check confirming `user.status === 'ACTIVE'` and current `user.role` (`401` / `403` if invalid or deactivated).
   - **Step 5.2 (Input Validation):** Validates DTO constraints (`quantity > 0` integer, valid UUIDs) (`400` if invalid).
   - **Step 5.3 (Interactive Database Transaction, Active-State Check & Row Lock):** Opens an interactive PostgreSQL transaction (`prisma.$transaction`) and executes all state and balance checks inside the transaction boundary so validation never becomes stale:
     - Verifies the user holds `ADMIN` or `OPERATOR` role, and if `OPERATOR`, confirms an active assignment exists in `Warehouse_Operators` for the target warehouse (`403` if unauthorized).
     - Verifies inside the transaction that both the target warehouse and product have `status === 'ACTIVE'` (`422 INACTIVE_RESOURCE` if deactivated/archived).
     - Acquires a pessimistic row lock (`SELECT id, quantity FROM inventory_balances ... FOR UPDATE`) on the product's balance row in that warehouse and evaluates `available_quantity >= requested_quantity`.
     - *Failure Path:* If available stock is `12` and requested quantity is `20`, the transaction rolls back immediately and returns `422 Unprocessable Entity` (`INSUFFICIENT_STOCK`).
     - *Success Path:* Deducts the quantity from `Inventory_Balances`, inserts an immutable `OUT` record with `balance_after` into `Stock_Movements`, and commits the transaction.
6. **UI State Synchronization:** The frontend displays a confirmation notification, invalidates cached queries, and displays the updated warehouse balance (`7 units`) alongside the new ledger entry formatted in Jalali date/time.

---

## 8. Non-Functional Requirements (NFRs)

1. **Data Integrity & Concurrency:** Stock balances must never become negative under concurrent API requests. All balance mutations must execute inside serialized or row-locked ACID transactions.
2. **Security:** Passwords must be hashed using `bcrypt`. All protected endpoints must validate JWT tokens and enforce role and warehouse ownership checks on the server. UI element hiding is treated strictly as a usability enhancement, never as a security boundary.
3. **Performance:** Standard API reads (dashboard, balance queries, catalog listings) must respond within `< 200ms` p95 under normal SME load, supported by composite B-Tree indexes on foreign keys and filter columns.
4. **Maintainability:** The backend must adhere to a strict **Modular Monolith** structure where domain modules communicate through exported services rather than cross-module table mutations.

---

## 9. Assumptions & Constraints

* **Single-Organization Deployment (v1.0):** Each deployed instance of Depoma v1.0 serves a single business entity. Multi-tenancy is deferred to the commercial SaaS expansion phase.
* **Base Unit Consistency:** Each product has a single base unit of measure (e.g., Piece, Box, Kilogram), and all movements for that product are recorded as positive integers in that base unit.
* **Managed User Provisioning:** User accounts are provisioned internally by an `ADMIN`; public self-registration is disabled by design.
