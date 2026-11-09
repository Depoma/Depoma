# Phase 1: Define the MVP and Prioritize User Stories

**Work Plan Window:** 9–15 Aban 1405  
**Planned Work:** Define the MVP and prioritize user stories  
**Milestone:** Initial Product Definition & Requirements Specification  
**Status:** ✅ **Completed**  

---

## 1. Overview & Objectives

This phase establishes the foundational scope and user requirements for **Depoma (Inventory & Stock Ledger Management System)**. It identifies the target SME operational challenges, defines the core MVP boundaries, sets non-goals, and formalizes user stories with observable and testable acceptance criteria.

---

## 2. Deliverables in this Phase

| Deliverable | File | Description | Status |
| :--- | :--- | :--- | :---: |
| **Product Requirements Document (PRD)** | [`PRD.md`](./PRD.md) | Business problem statement, target users, system roles (`ADMIN`, `OPERATOR`, `MANAGER`), MoSCoW MVP boundaries, non-goals, and the primary business workflow. | ✅ **Completed** |
| **User Stories & Acceptance Criteria** | [`USER_STORIES.md`](./USER_STORIES.md) | Standard user stories (`As a [role], I want [action], so that [benefit]`) across Must Have, Should Have, and Could Have priorities with positive, error, and authorization acceptance criteria. | ✅ **Completed** |
| **Requirements Traceability Matrix** | [`REQUIREMENTS_TRACEABILITY.md`](./REQUIREMENTS_TRACEABILITY.md) | Cross-functional traceability matrix mapping product requirements and user stories to API endpoints, database entities, and UI prototype screens. | ✅ **Completed** |

---

## 3. Key Decisions & Outputs

1. **Product Name & Positioning:** **Depoma**, a web-based inventory management system tailored for Iranian SMEs with native Persian (RTL) interface and Jalali calendar display.
2. **Access Control Roles:**
   - **`ADMIN`:** Full configuration, user management, warehouse assignment, catalog, and inventory operations.
   - **`OPERATOR`:** Operations restricted strictly to assigned warehouses (`Warehouse_Operators`).
   - **`MANAGER`:** Organization-wide read-only access for auditing and executive visibility.
3. **Core MVP Workflow:** Recording a warehouse stock issue (`Stock Out`) with real-time balance validation, row-level locking, and immutable ledger recording.
