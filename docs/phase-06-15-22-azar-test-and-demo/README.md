# Phase 6: Integration Testing & Demo Rehearsal

**Work Plan Window:** 15–22 Azar 1405  
**Planned Work:** Test the integration and rehearse the demo  
**Submission Cutoff:** 22 Azar 1405 (23:59 IRST) | **Presentation 2:** 23–24 Azar 1405  
**Status:** ⏳ **Planned / Not Started**  

---

## 1. Overview & Objectives

This phase solidifies the working software built in Phases 4 and 5 through rigorous automated testing, negative scenario verification, and demonstration rehearsal ahead of Presentation 2 (Development Progress Review).

---

## 2. Planned Deliverables & Engineering Tasks

| Task / Deliverable | Planned Scope & Target Outputs | Status |
| :--- | :--- | :---: |
| **1. Backend Unit & Integration Tests** | Automated `Jest` + `Supertest` suites verifying authentication (`200`/`401`), role restrictions (`403`), unassigned warehouse rejection (`403`), and insufficient stock rollbacks (`422`). | ⏳ **Planned** |
| **2. Concurrency & Race-Condition Tests** | Test suites verifying that concurrent stock issue requests cannot drive stock below zero. | ⏳ **Planned** |
| **3. End-to-End User Journey Verification** | `Playwright` automated tests covering the full path: Login $\rightarrow$ Dashboard $\rightarrow$ Inventory $\rightarrow$ Stock Out $\rightarrow$ Balance Update. | ⏳ **Planned** |
| **4. Demo Rehearsal & Evidence Preparation** | Live demonstration walkthrough and test evidence compilation for Presentation 2. | ⏳ **Planned** |
| **5. Milestone Release Tag** | Git tag `v0.5.0-progress-review` representing the working software state. | ⏳ **Planned** |
