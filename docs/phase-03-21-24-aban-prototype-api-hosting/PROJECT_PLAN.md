# Product Development Roadmap & Execution Plan

**Product Name:** Depoma — Inventory & Stock Ledger Management System  
**Document Version:** 1.0  
**Engineering Team:** 2 Engineers (Frontend Lead & Backend/Database Lead)  

This document outlines the engineering milestones, work breakdown structure, task dependencies, and release roadmap for delivering Depoma v1.0.

---

## 1. Milestone 1: Product Discovery, System Architecture & Specification Baseline

All foundational product requirements, domain models, database schemas, REST API contracts, and UI/UX prototype specifications have been finalized to establish a locked engineering baseline prior to implementation:

| Deliverable ID | Engineering Deliverable | Specification File | Lead Owner | Status |
| :---: | :--- | :--- | :---: | :---: |
| **M1-01** | Product Requirements & MVP Scope Definition | [`../phase-01-9-15-aban-mvp-user-stories/PRD.md`](../phase-01-9-15-aban-mvp-user-stories/PRD.md) | Product / Shared | ✅ **Completed** |
| **M1-02** | Prioritized User Stories & Testable Acceptance Criteria | [`../phase-01-9-15-aban-mvp-user-stories/USER_STORIES.md`](../phase-01-9-15-aban-mvp-user-stories/USER_STORIES.md) | Product / Shared | ✅ **Completed** |
| **M1-03** | Modular Monolith Architecture & SaaS Evolution Plan | [`../phase-02-16-20-aban-diagrams-erd/ARCHITECTURE.md`](../phase-02-16-20-aban-diagrams-erd/ARCHITECTURE.md) | Backend Lead & Frontend Lead | ✅ **Completed** |
| **M1-04** | PostgreSQL Relational Schema, `Warehouse_Operators` & Ledger Design | [`../phase-02-16-20-aban-diagrams-erd/DATABASE_DESIGN.md`](../phase-02-16-20-aban-diagrams-erd/DATABASE_DESIGN.md) | Backend & DB Lead | ✅ **Completed** |
| **M1-05** | REST API Specification & Error Contract | [`API_SPECIFICATION.md`](./API_SPECIFICATION.md) | Backend Lead & Frontend Lead | ✅ **Completed** |
| **M1-06** | UML Use Case, Activity, Architecture & ERD Models | [`../phase-02-16-20-aban-diagrams-erd/diagrams/ALL_DIAGRAMS.md`](../phase-02-16-20-aban-diagrams-erd/diagrams/ALL_DIAGRAMS.md) | Shared | ✅ **Completed** |
| **M1-07** | Persian RTL UI Design System & Interactive Prototype Spec | [`PROTOTYPE_SPECIFICATION.md`](./PROTOTYPE_SPECIFICATION.md) | Frontend Lead | ✅ **Completed** |
| **M1-08** | Cloud Infrastructure Evaluation & Production Sizing Plan | [`HOSTING_PLAN.md`](./HOSTING_PLAN.md) | Backend & DB Lead | ✅ **Completed** |
| **M1-09** | Engineering Standards, Testing Strategy & Git Workflow | [`DEVELOPMENT_GUIDELINES.md`](./DEVELOPMENT_GUIDELINES.md) | Shared | ✅ **Completed** |
| **M1-10** | Requirements Traceability & Cross-Specification Consistency Matrix | [`../phase-01-9-15-aban-mvp-user-stories/REQUIREMENTS_TRACEABILITY.md`](../phase-01-9-15-aban-mvp-user-stories/REQUIREMENTS_TRACEABILITY.md) | Shared | ✅ **Completed** |
| **M1-11** | Interactive Clickable Figma Prototype Construction | Linked in [`../../../README.md`](../../../README.md) | Frontend Lead | ⏳ **In Progress** |

---

## 2. Milestone 2: Core Platform, Authentication & End-to-End Vertical Slice (`v0.5.0`)

**Milestone Objective:** Establish the monorepo foundation, containerized PostgreSQL database, JWT authentication, role and warehouse-level authorization guards, and the first complete end-to-end vertical slice from the Persian RTL interface down to persistent database storage.

| Iteration | Backend & Database Lead Tasks | Frontend Lead Tasks | Integration & Verification Target |
| :--- | :--- | :--- | :--- |
| **Sprint 2.1:<br>Foundation & First Vertical Slice** | 1. Initialize NestJS project and `docker-compose.yml` (PostgreSQL 16).<br>2. Implement `prisma/schema.prisma`, execute initial SQL migrations, and build `seed.ts` with default users, warehouses, and catalog items.<br>3. Implement `CatalogModule` (`GET /api/products`, `POST /api/products`, `GET /api/categories`, `POST /api/categories`). | 1. Initialize React 18 + TypeScript + Vite + Tailwind CSS (`dir="rtl"`, `Vazirmatn` font, logical CSS utilities).<br>2. Build core RTL `AppShell`, `Sidebar`, `Header`, and UTC-to-Jalali date utilities (`src/lib/date.ts`).<br>3. Connect the Product Catalog view and Create Product modal to live backend endpoints. | **Deliverable:** End-to-end product creation and persistent listing across browser refreshes. |
| **Sprint 2.2:<br>Authentication, RBAC & Core Ledger Engine** | 1. Implement `AuthModule` (`POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/auth/me`) using `bcrypt` and signed JWTs.<br>2. Implement `RolesGuard` and `WarehousesService.assertOperatorHasAccess()` against `warehouse_operators`.<br>3. Implement transactional `InventoryModule` endpoints (`POST /api/inventory/receipts`, `POST /api/inventory/issues`, `GET /api/inventory/balances`) with `SELECT ... FOR UPDATE` row locking. | 1. Build the `Login` screen, session provider, protected route guards, and header identity display.<br>2. Build the `Inventory Balances` view with warehouse selector.<br>3. Implement the `Record Stock In` and `Record Stock Out` modals with Zod validation and `422` / `403` error banner rendering. | **Deliverable:** Full execution of the primary business workflow (Stock In & Stock Out with real-time balance enforcement and warehouse access control). |
| **Sprint 2.3:<br>Automated Integration Testing** | 1. Write unit and Supertest API integration tests verifying authentication (`401`), role/warehouse restrictions (`403`), insufficient stock rollback (`422`), and valid ledger commits (`201`).<br>2. Stabilize error envelopes and seed data. | 1. Write component and initial E2E tests covering sign-in and stock issue submission.<br>2. Verify RTL layout responsiveness and loading/empty/error states. | **Release Tag:** `v0.5.0` — Verified core platform and primary workflow. |

---

## 3. Milestone 3: Full MVP Completion, Production Deployment & Hardening (`v1.0.0`)

**Milestone Objective:** Complete all remaining Must-Have and prioritized Should-Have capabilities, execute comprehensive E2E and concurrency tests, and deploy the production application over HTTPS.

| Iteration | Engineering Activities (Frontend + Backend) | Completion Criteria (Definition of Done) |
| :--- | :--- | :--- |
| **Sprint 3.1:<br>Audit Ledger, Dashboard & Operator Management** | • Implement `GET /api/inventory/movements` and the `Movement History (Ledger)` view with warehouse, product, and movement-type filters.<br>• Implement `GET /api/dashboard/summary` and the `Operational Dashboard` view (KPI cards, low-stock warnings, and recent activity table).<br>• Implement `Warehouses` management view and the Admin modal for assigning/revoking `Warehouse_Operators` (`POST` & `DELETE /api/warehouses/:id/operators`).<br>• Implement `Users` management view (`GET`, `POST`, `PATCH /api/users`). | 100% of **Must Have** user stories (`US-01` through `US-09`) and low-stock filtering (`US-10`) operating end-to-end. |
| **Sprint 3.2:<br>Production Cloud Deployment & E2E Verification** | • Provision production PostgreSQL 16, NestJS API container, and React static hosting on cloud infrastructure with HTTPS.<br>• Execute production database migrations (`prisma migrate deploy`) and initial admin seeding.<br>• Run full Playwright E2E test suite and negative security/concurrency scenarios against staging and production environments. | Live HTTPS production deployment accessible and passing all functional and security test suites. |
| **Sprint 3.3:<br>Release Freeze & Documentation Sync** | • Perform UI polish, RTL alignment checks, and query performance verification.<br>• Synchronize `README.md`, `API_SPECIFICATION.md`, `DATABASE_DESIGN.md`, and architecture diagrams with the final production codebase.<br>• Tag release `v1.0.0`. | Zero open critical defects; documentation and production deployment in 100% alignment. |
