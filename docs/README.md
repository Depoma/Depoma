# Depoma Engineering Documentation Index

Welcome to the central documentation directory for **Depoma**, a web-based Inventory and Stock Ledger Management System built for small and medium-sized enterprises (SMEs).

This documentation directory is organized into dedicated phase directories mapped directly to the official semester work plan (`Software_Engineering_Semester_Project_Final.pdf` Section 7). Non-presentation rows are designated as individual engineering phases with their corresponding deliverables and documentation.

---

## 1. Project Phases Overview & Schedule

| Phase | Directory | Schedule (1405) | Planned Work Description | Status |
| :---: | :--- | :--- | :--- | :---: |
| **Phase 01** | 📁 **[`phase-01-9-15-aban-mvp-user-stories/`](./phase-01-9-15-aban-mvp-user-stories/README.md)** | **9–15 Aban** | Define the MVP and prioritize user stories. | ✅ **Completed** |
| **Phase 02** | 📁 **[`phase-02-16-20-aban-diagrams-erd/`](./phase-02-16-20-aban-diagrams-erd/README.md)** | **16–20 Aban** | Prepare Use Case, Activity, system diagram, and initial ERD. | ✅ **Completed** |
| **Phase 03** | 📁 **[`phase-03-21-24-aban-prototype-api-hosting/`](./phase-03-21-24-aban-prototype-api-hosting/README.md)** | **21–24 Aban** | Prepare the prototype, API list, technology choices, and hosting plan. *(Phase 1 Submission Cutoff: 24 Aban at 23:59)* | ✅ **Specs Completed**<br>*(⏳ Figma Link Pending)* |
| — | *[Presentation 1: Analysis and Design Review]* | *25–26 Aban* | *Formal 8-minute presentation + 4-minute defense (20 points)* | — |
| **Phase 04** | 📁 **[`phase-04-27-aban-7-azar-setup-integration/`](./phase-04-27-aban-7-azar-setup-integration/README.md)** | **27 Aban–7 Azar** | Set up code, database, API, and the first UI-to-API integration. | ⏳ **Planned** |
| **Phase 05** | 📁 **[`phase-05-8-14-azar-auth-core-workflow/`](./phase-05-8-14-azar-auth-core-workflow/README.md)** | **8–14 Azar** | Implement authentication, authorization, and the core workflow. | ⏳ **Planned** |
| **Phase 06** | 📁 **[`phase-06-15-22-azar-test-and-demo/`](./phase-06-15-22-azar-test-and-demo/README.md)** | **15–22 Azar** | Test the integration and rehearse the demo. *(Phase 2 Submission Cutoff: 22 Azar at 23:59)* | ⏳ **Planned** |
| — | *[Presentation 2: Development Progress Review]* | *23–24 Azar* | *Formal 7-minute demo + 5-minute defense (25 points)* | — |
| **Phase 07** | 📁 **[`phase-07-25-azar-17-dey-must-have-features/`](./phase-07-25-azar-17-dey-must-have-features/README.md)** | **25 Azar–17 Dey** | Complete approved Must-Have features; target completion by 17 Dey. | ⏳ **Planned** |
| **Phase 08** | 📁 **[`phase-08-18-24-dey-deploy-and-bug-fixes/`](./phase-08-18-24-dey-deploy-and-bug-fixes/README.md)** | **18–24 Dey** | Deploy, fix bugs, and test the main scenarios. | ⏳ **Planned** |
| **Phase 09** | 📁 **[`phase-09-25-30-dey-code-freeze-docs/`](./phase-09-25-30-dey-code-freeze-docs/README.md)** | **25–30 Dey** | Finish fixes and documentation; code freeze by 30 Dey. | ⏳ **Planned** |
| **Phase 10** | 📁 **[`phase-10-1-3-bahman-final-verification/`](./phase-10-1-3-bahman-final-verification/README.md)** | **1–3 Bahman** | Verify the final version; submit by 3 Bahman at 18:00. | ⏳ **Planned** |
| — | *[Presentation 3: Final MVP and Engineering Defense]* | *5–6 Bahman* | *Formal 12-minute demo + 6-minute defense (55 points)* | — |

---

## 2. Directory Structure of Completed Phases (Phases 1–3)

### Phase 01: 9–15 Aban — MVP Definition & User Stories
* 📄 **[`phase-01-9-15-aban-mvp-user-stories/PRD.md`](./phase-01-9-15-aban-mvp-user-stories/PRD.md)** — Problem statement, target users, roles, MoSCoW MVP boundaries, and non-goals.
* 📋 **[`phase-01-9-15-aban-mvp-user-stories/USER_STORIES.md`](./phase-01-9-15-aban-mvp-user-stories/USER_STORIES.md)** — Prioritized user stories with observable and testable acceptance criteria.
* 🔍 **[`phase-01-9-15-aban-mvp-user-stories/REQUIREMENTS_TRACEABILITY.md`](./phase-01-9-15-aban-mvp-user-stories/REQUIREMENTS_TRACEABILITY.md)** — Traceability matrix connecting requirements to stories, APIs, schema, and UI views.

### Phase 02: 16–20 Aban — System Diagrams & Initial ERD
* 📊 **[`phase-02-16-20-aban-diagrams-erd/diagrams/ALL_DIAGRAMS.md`](./phase-02-16-20-aban-diagrams-erd/diagrams/ALL_DIAGRAMS.md)** — 4 required design views (Use Case, Activity, Architecture, ERD).
* 📁 **[`phase-02-16-20-aban-diagrams-erd/diagrams/`](./phase-02-16-20-aban-diagrams-erd/diagrams/)** — Editable diagram sources (`.mmd` and `.puml`).
* 🗄️ **[`phase-02-16-20-aban-diagrams-erd/DATABASE_DESIGN.md`](./phase-02-16-20-aban-diagrams-erd/DATABASE_DESIGN.md)** — PostgreSQL schema, constraints, indexes, transactional locking, and `schema.prisma`.
* 🏗️ **[`phase-02-16-20-aban-diagrams-erd/ARCHITECTURE.md`](./phase-02-16-20-aban-diagrams-erd/ARCHITECTURE.md)** — Modular Monolith architecture, DAG module rules, live JWT check, and SaaS roadmap.

### Phase 03: 21–24 Aban — Prototype, API Contracts & Hosting Plan
* 🎨 **[`phase-03-21-24-aban-prototype-api-hosting/PROTOTYPE_SPECIFICATION.md`](./phase-03-21-24-aban-prototype-api-hosting/PROTOTYPE_SPECIFICATION.md)** — Persian RTL visual design system and 8-frame clickable prototype blueprint.
* 🔌 **[`phase-03-21-24-aban-prototype-api-hosting/API_SPECIFICATION.md`](./phase-03-21-24-aban-prototype-api-hosting/API_SPECIFICATION.md)** — Standardized REST API contracts, request/response envelopes, and error codes.
* ☁️ **[`phase-03-21-24-aban-prototype-api-hosting/HOSTING_PLAN.md`](./phase-03-21-24-aban-prototype-api-hosting/HOSTING_PLAN.md)** — Cloud provider evaluation, resource limits, and monthly cost estimates.
* 🛠️ **[`phase-03-21-24-aban-prototype-api-hosting/DEVELOPMENT_GUIDELINES.md`](./phase-03-21-24-aban-prototype-api-hosting/DEVELOPMENT_GUIDELINES.md)** — Coding standards, testing strategy, and Git workflow.
* 📅 **[`phase-03-21-24-aban-prototype-api-hosting/PROJECT_PLAN.md`](./phase-03-21-24-aban-prototype-api-hosting/PROJECT_PLAN.md)** — Engineering milestones and roadmap execution plan.
