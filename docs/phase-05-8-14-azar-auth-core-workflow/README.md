# Phase 5: Authentication, Authorization & Core Inventory Workflow

**Work Plan Window:** 8–14 Azar 1405  
**Planned Work:** Implement authentication, authorization, and the core workflow  
**Milestone:** Security Boundaries & Transactional Inventory Ledger  
**Status:** ⏳ **Planned / Not Started**  

---

## 1. Overview & Objectives

This phase delivers the core business engine of Depoma. It establishes secure user authentication, backend-enforced role and warehouse-level access guards, and implements the primary inventory journey: recording stock receipts (`Stock In`) and stock issues (`Stock Out`) backed by pessimistic row locking and atomic balance ledger updates.

---

## 2. Planned Deliverables & Engineering Tasks

| Task / Deliverable | Planned Scope & Target Outputs | Status |
| :--- | :--- | :---: |
| **1. Authentication Service (`AuthModule`)** | Real `POST /api/auth/login`, `POST /api/auth/logout`, and `GET /api/auth/me` with `bcrypt` password hashing and signed JWT issuance. | ⏳ **Planned** |
| **2. Live User Status Check & Guard Enforcement** | `JwtStrategy.validate()` performing real-time database lookups for active account status (`user.status === 'ACTIVE'`) and dynamic role enforcement via `RolesGuard`. | ⏳ **Planned** |
| **3. Warehouse-Level Operator Access Control** | Backend enforcement of `warehouse_operators` mapping, restricting operators strictly to their assigned warehouses. | ⏳ **Planned** |
| **4. Core Inventory Transaction Engine (`InventoryModule`)** | Concurrency-safe atomic `UPSERT` for `POST /api/inventory/receipts` and `SELECT ... FOR UPDATE` row locking for `POST /api/inventory/issues` with strict insufficient stock prevention (`422 INSUFFICIENT_STOCK`). | ⏳ **Planned** |
| **5. Frontend Core Workflow Integration** | Persian RTL Login page, session context provider, warehouse-filtered Inventory Balances view, and interactive Stock In/Out modals with inline error handling. | ⏳ **Planned** |
