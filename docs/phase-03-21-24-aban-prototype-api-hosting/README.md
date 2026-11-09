# Phase 3: Prototype, API List, Technology Choices & Hosting Plan

**Work Plan Window:** 21–24 Aban 1405  
**Planned Work:** Prepare the prototype, API list, technology choices, and hosting plan  
**Submission Cutoff:** 24 Aban 1405 (23:59 IRST) | **Presentation 1:** 25–26 Aban 1405  
**Status:** ✅ **Specifications Completed** *(⏳ External Figma URL Pending)*  

---

## 1. Overview & Objectives

This phase delivers the interface specifications, API contracts, infrastructure hosting strategy, and engineering guidelines required to conclude the initial Analysis & Design milestone before code implementation begins.

---

## 2. Deliverables in this Phase

| Deliverable | File | Description | Status |
| :--- | :--- | :--- | :---: |
| **UI/UX & Prototype Specification** | [`PROTOTYPE_SPECIFICATION.md`](./PROTOTYPE_SPECIFICATION.md) | Comprehensive Persian RTL visual design system (`1440×900` frames, `Vazirmatn` typography, color tokens, component library) and an 8-frame interaction wiring blueprint for the clickable prototype in Figma. | ✅ **Completed** |
| **REST API Contract Specification** | [`API_SPECIFICATION.md`](./API_SPECIFICATION.md) | Contract specifications for 24 REST endpoints across all domain modules with method, path, role requirements, standardized `{ items }` / `{ items, meta }` envelopes, validation rules, and error codes. | ✅ **Completed** |
| **Cloud Deployment & Hosting Plan** | [`HOSTING_PLAN.md`](./HOSTING_PLAN.md) | Analysis of hosting choices (Domestic PaaS vs. International Cloud), regional network reliability, HTTPS strategy, production resource allocations, and estimated monthly costs (~300k–400k Toman/month). | ✅ **Completed** |
| **Engineering & Development Guidelines** | [`DEVELOPMENT_GUIDELINES.md`](./DEVELOPMENT_GUIDELINES.md) | Team ownership, Modular Monolith rules, coding conventions, three-tier testing strategy (`Jest`, `Supertest`, `Playwright`), and Git workflow. | ✅ **Completed** |
| **Project Roadmap & Work Plan** | [`PROJECT_PLAN.md`](./PROJECT_PLAN.md) | Semester work plan mapping, sprint execution targets, and delivery milestones. | ✅ **Completed** |

---

## 3. Pending External Item for Presentation 1

* **Clickable Prototype URL (Figma):**  
  The complete geometry, typography, color codes, component states, and click wiring are fully specified in [`PROTOTYPE_SPECIFICATION.md`](./PROTOTYPE_SPECIFICATION.md). The interactive prototype is to be assembled in Figma using these specifications, and its public link placed in the root [`README.md`](../../README.md).
