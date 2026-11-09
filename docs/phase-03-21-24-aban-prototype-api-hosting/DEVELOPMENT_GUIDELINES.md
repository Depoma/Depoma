# Engineering & Development Guidelines

**Product Name:** Depoma — Inventory & Stock Ledger Management System  
**Document Version:** 1.0  

This document establishes the engineering standards, architectural boundary rules, coding conventions, testing requirements, and Git workflow followed by the Depoma software engineering team.

---

## 1. Engineering Team Structure & Ownership

Depoma is developed by a two-engineer core team. While each engineer leads a primary technical domain, both engineers maintain shared ownership of system architecture, API contracts, end-to-end integration, and operational reliability:

| Role | Primary Engineering Ownership | Shared Engineering Responsibilities |
| :--- | :--- | :--- |
| **Frontend Lead** | • UI/UX system design and interactive Figma prototyping.<br>• React 18, TypeScript, Tailwind CSS, and `shadcn/ui` implementation.<br>• Native Persian RTL layout architecture (`dir="rtl"`) and logical CSS styling.<br>• Client-side form state and schema validation (`React Hook Form` + `Zod`).<br>• Server-state caching and optimistic/invalidated updates (`TanStack Query`).<br>• UTC-to-Jalali calendar formatting and Persian numeral rendering.<br>• End-to-End (`Playwright`) and UI component testing. | • Collaborative requirements analysis and REST API contract design.<br>• Continuous integration between client views and backend endpoints.<br>• Peer code reviews (`Pull Requests`) and cross-stack architectural fluency. |
| **Backend & Database Lead** | • Relational data modeling (`ERD`), Prisma schema design, and SQL migrations.<br>• NestJS **Modular Monolith** architecture and domain service encapsulation.<br>• Authentication (`JWT` + `bcrypt`), RBAC (`RolesGuard`), and warehouse-level operator access enforcement (`Warehouse_Operators`).<br>• Transactional **Inventory Ledger** engine (`SELECT ... FOR UPDATE` row locking and atomic balance updates).<br>• Unit and API integration test suites (`Jest` + `Supertest`).<br>• Docker containerization, database seeding, and cloud production deployment. | • Maintaining API documentation (`API_SPECIFICATION.md`) and database specifications.<br>• Ensuring deterministic error responses and localized Persian error messages.<br>• Peer code reviews and end-to-end verification of production deployments. |

---

## 2. Modular Monolith Boundary Rules

To preserve clean domain separation and prevent architectural erosion, all backend code must comply with four non-negotiable rules:

1. **Thin Controllers, Rich Domain Services:**
   - `*.controller.ts` files are restricted to HTTP routing, parameter extraction, and delegating execution to a domain service. Business conditionals, stock calculations, and database queries must never be written inside controllers.
2. **Strict Table Ownership (No Cross-Module Writes):**
   - Each domain module has exclusive write ownership of its database tables:
     - `UsersModule` $\rightarrow$ `users`
     - `CatalogModule` $\rightarrow$ `categories`, `products`
     - `WarehousesModule` $\rightarrow$ `warehouses`, `warehouse_operators`
     - `InventoryModule` $\rightarrow$ `inventory_balances`, `stock_movements`
   - Modules must never mutate another module's tables directly. Cross-module verification (such as `InventoryService` verifying whether an operator is assigned to a warehouse) must occur through exported public service methods (`WarehousesService.assertOperatorHasAccess(...)`).
3. **Mandatory ACID Transactions & Row Locking for Stock Operations:**
   - Every mutation to `inventory_balances` and insertion into `stock_movements` **must** execute inside a single `prisma.$transaction(async (tx) => { ... })` block. Uncoordinated standalone queries against balance or movement tables are strictly prohibited.
   - Because Prisma's standard `.findUnique()` does not emit `FOR UPDATE`, `Stock Out` must acquire an exclusive row lock inside the transaction via `tx.$queryRaw` (`SELECT id, quantity FROM inventory_balances WHERE product_id = $1 AND warehouse_id = $2 FOR UPDATE`), and `Stock In` must use PostgreSQL's atomic `INSERT ... ON CONFLICT (product_id, warehouse_id) DO UPDATE`.
   - Product and warehouse `status === 'ACTIVE'` checks must also execute inside the transaction boundary (`SELECT ... FOR SHARE`) prior to balance mutation so validation cannot become stale.
4. **Immutable Ledger Enforcement:**
   - The `stock_movements` table is strictly append-only. No `update` or `delete` operations may be invoked on `prisma.stockMovement` anywhere in the codebase. Stocktaking discrepancies in v1.0 are recorded as new `IN` or `OUT` movements with explicit adjustment reasons.
5. **Acyclic Module Dependency Direction:**
   - Module imports must strictly follow `Dashboard -> Inventory -> {Catalog, Warehouses} -> Users <- Auth`. `CatalogModule` must never import `InventoryModule`.
6. **Duplicate Submission Protection:**
   - Frontend mutation submit buttons must be disabled while `mutation.isPending === true`, and TanStack Query `retry` must be set to `false` on `POST` mutations to prevent accidental duplicate ledger entries during transient network drops.

---

## 3. Coding & Naming Conventions

### 3.1. Identifier Naming Standards
* **Database Layer (PostgreSQL):** Plural `snake_case` for table names (`stock_movements`, `warehouse_operators`, `inventory_balances`) and `snake_case` for column names (`minimum_stock`, `balance_after`, `created_at`). Mapped cleanly to TypeScript `camelCase` via Prisma `@map` and `@@map`.
* **TypeScript (Backend & Frontend):**
  - Variables, functions, and class methods: `camelCase` (`recordStockIssue`, `availableQuantity`).
  - Classes, interfaces, types, enums, and React components: `PascalCase` (`InventoryService`, `StockOutModal`, `MovementType`).
  - Constants and environment keys: `UPPER_SNAKE_CASE` (`DATABASE_URL`, `JWT_SECRET`).
* **File Naming:**
  - Backend files follow NestJS `kebab-case` conventions (`inventory.controller.ts`, `inventory.service.ts`, `create-stock-issue.dto.ts`).
  - Frontend React components use `PascalCase.tsx` (`StockOutModal.tsx`, `InventoryBalancesTable.tsx`); hooks and utilities use `camelCase.ts` (`useInventoryBalances.ts`, `date.ts`).

### 3.2. Localization & Date/Time Standards
* **Storage & Transport:** All timestamps in PostgreSQL and JSON API payloads must be stored and transmitted in **UTC (`ISO 8601`)**. Storing Jalali formatted strings in the database is prohibited.
* **Presentation Layer:** The frontend converts UTC timestamps to the Jalali calendar (`YYYY/MM/DD - HH:mm`) and formats numbers into Persian digits (`۰-۹`) exclusively at render time via `src/lib/date.ts`.

---

## 4. Feature Implementation Workflow (Vertical Slice Delivery)

Features are developed iteratively in vertical slices connecting the database, backend, and frontend:

1. **Schema & Migration:** Define or update entities in `prisma/schema.prisma` and generate versioned SQL migrations (`npx prisma migrate dev`).
2. **DTO & Validation Contract:** Create request/response DTO classes in NestJS annotated with `class-validator` decorators (`@IsUUID()`, `@IsInt()`, `@Min(1)`).
3. **Service & Security Implementation:** Implement the business logic, transaction locks, and authorization checks (`@UseGuards(JwtAuthGuard, RolesGuard)`).
4. **Automated Backend Verification:** Write unit and integration tests covering both the success path and failure/authorization edge cases.
5. **Frontend Integration:** Implement the typed API client hook (`TanStack Query`) and connect the RTL React component/form.
6. **End-to-End Verification:** Verify persistence across page reloads and ensure all automated checks pass before merging.

---

## 5. Testing Strategy & Quality Gates

Quality assurance is structured across three automated testing layers:

### 5.1. Unit Tests (`Jest`)
Focus on isolated domain logic and utility functions:
* Stock availability evaluation (`available >= requested`) and `balance_after` calculation.
* Operator warehouse authorization helper logic.
* Frontend UTC-to-Jalali date formatting and Zod form validation schemas.

### 5.2. API & Database Integration Tests (`Jest` + `Supertest`)
Execute HTTP requests against the NestJS application connected to a dedicated test database:
* **Authentication:** Valid login (`200`), invalid password (`401`), unauthenticated access to protected routes (`401`).
* **Role & Warehouse Authorization:**
  - `MANAGER` attempting `POST /api/inventory/issues` $\rightarrow$ rejected with `403 Forbidden`.
  - `OPERATOR` attempting `POST /api/inventory/issues` or `GET /api/inventory/balances` on an unassigned warehouse $\rightarrow$ rejected with `403 Forbidden` (`FORBIDDEN_WAREHOUSE_ACCESS`).
* **Transactional Ledger Integrity & Negative Scenarios:**
  - Recording a `Stock In` of `50` units creates an `IN` movement and sets balance to `50`.
  - Recording a `Stock Out` of `20` units creates an `OUT` movement and sets `balance_after` and current balance to `30`.
  - Attempting a `Stock Out` of `40` units when only `30` are available fails with `422 Unprocessable Entity` (`INSUFFICIENT_STOCK`), leaves the balance at `30`, and writes no movement row.
  - Submitting `quantity: 0` or `quantity: -5` fails with `400 Bad Request`.
  - Concurrent `Stock Out` requests exceeding total available stock serialize via row locking so that stock never becomes negative.

### 5.3. End-to-End User Journey Tests (`Playwright`)
Verify the complete browser workflow:
* `Sign In as Operator -> View Dashboard -> Navigate to Inventory -> Open Stock Out Modal -> Attempt Oversell (Verify 422 Alert) -> Submit Valid Stock Out -> Verify Updated Balance and Jalali Ledger Entry`.

### 5.4. Definition of Done (DoD)
A feature is considered complete only when:
- [x] Both the primary success path and critical error/authorization paths work end-to-end.
- [x] Frontend UI, Backend API, and PostgreSQL storage are fully integrated with persistent state across browser refreshes.
- [x] TypeScript compilation (`tsc --noEmit`), ESLint, and automated tests pass with zero errors.
- [x] Technical documentation (`README.md`, `docs/phase-1/API_SPECIFICATION.md`, `docs/phase-1/DATABASE_DESIGN.md`) reflects any schema or contract updates.

---

## 6. Version Control & Git Workflow

1. **Branching Strategy:**
   - `main`: Production-ready, stable release branch.
   - `develop`: Primary integration branch for ongoing feature work.
   - Feature & Fix Branches: `feat/<scope>-<description>` (e.g., `feat/inventory-issue-transaction`, `feat/rtl-inventory-table`) and `fix/<issue>`.
2. **Commit Message Standard (Conventional Commits):**
   - `docs: standardize architecture, database, and API specifications`
   - `feat(inventory): implement atomic stock issue transaction with row locking`
   - `feat(ui): add Persian RTL stock out modal and error state handling`
   - `test(inventory): add integration tests for insufficient stock and operator access`
3. **Release Tagging:**
   - `v0.1.0`: Architecture, Database & API Specification Baseline
   - `v0.5.0`: Core Platform, Authentication & End-to-End Vertical Slice
   - `v1.0.0`: Production MVP Release
