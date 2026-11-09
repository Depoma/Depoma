# Phase 2: Use Case, Activity, System Diagram & Initial ERD

**Work Plan Window:** 16–20 Aban 1405  
**Planned Work:** Prepare Use Case, Activity, system diagram, and initial ERD  
**Milestone:** System Architecture & Data Modeling  
**Status:** ✅ **Completed**  

---

## 1. Overview & Objectives

This phase defines the formal architectural blueprints and persistent data models for Depoma. It establishes the system boundaries, models the primary operational workflow including all failure and authorization paths, justifies the **Modular Monolith** architecture pattern, and specifies the relational PostgreSQL database schema with its immutable ledger engine.

---

## 2. Deliverables in this Phase

| Deliverable | File | Description | Status |
| :--- | :--- | :--- | :---: |
| **System Design Diagrams (UML & Architecture)** | [`diagrams/ALL_DIAGRAMS.md`](./diagrams/ALL_DIAGRAMS.md) | Consolidated specifications for the four core design views: UML Use Case, UML Activity, System Architecture, and ERD. | ✅ **Completed** |
| **Editable Diagram Sources** | [`diagrams/`](./diagrams/) | Editable source files in Mermaid (`.mmd`) and PlantUML (`.puml`):<br>• [`use-case.mmd`](./diagrams/use-case.mmd) & [`use-case.puml`](./diagrams/use-case.puml)<br>• [`activity-stock-out.mmd`](./diagrams/activity-stock-out.mmd) & [`activity-stock-out.puml`](./diagrams/activity-stock-out.puml)<br>• [`system-architecture.mmd`](./diagrams/system-architecture.mmd)<br>• [`erd.mmd`](./diagrams/erd.mmd) | ✅ **Completed** |
| **Database Design & Inventory Ledger** | [`DATABASE_DESIGN.md`](./DATABASE_DESIGN.md) | Relational schema tables, primary/foreign keys, check constraints, composite unique indexes, concurrency-safe `UPSERT` (`Stock In`), `SELECT ... FOR UPDATE` row locking (`Stock Out`), and reference `schema.prisma`. | ✅ **Completed** |
| **Software Architecture Document** | [`ARCHITECTURE.md`](./ARCHITECTURE.md) | Modular Monolith justification, backend module boundaries, React RTL frontend architecture, multi-layer security pipeline (live JWT validation), and Multi-Tenant SaaS evolution roadmap. | ✅ **Completed** |

---

## 3. Four Core Architectural Views Summary

1. **UML Use Case View:** Models actor interactions (`Admin`, `Operator`, `Manager`) with system capabilities and `<<include>>` warehouse access verification.
2. **UML Activity View (Stock Out):** Step-by-step workflow tracing client validation, JWT authentication, role/warehouse verification, in-transaction active-state checks, `SELECT ... FOR UPDATE` row locking, insufficient stock rollback (`422`), and atomic commit.
3. **System Architecture View:** Three-tier architecture showing the Persian RTL React SPA, NestJS Modular Monolith API with bounded domain modules (`Auth`, `Users`, `Catalog`, `Warehouses`, `Inventory`, `Dashboard`), and PostgreSQL 16 database.
4. **Relational Data Model (ERD):** Tables for `users`, `warehouses`, `warehouse_operators` (join table for operator access control), `categories`, `products`, `inventory_balances`, and `stock_movements`.
