# Phase 4: Code, Database, API Setup & Initial UI Integration

**Work Plan Window:** 27 Aban – 7 Azar 1405  
**Planned Work:** Set up code, database, API, and the first UI-to-API integration  
**Milestone:** Codebase Initialization & First End-to-End Vertical Slice  
**Status:** ⏳ **Planned / Not Started**  

---

## 1. Overview & Objectives

This phase kicks off active software development following Presentation 1. The team initializes the shared repository codebase, configures containerized PostgreSQL 16 via Docker, applies initial Prisma database migrations, seeds foundational records, and implements the first complete vertical slice from the React frontend to the database.

---

## 2. Planned Deliverables & Engineering Tasks

| Task / Deliverable | Planned Scope & Target Outputs | Status |
| :--- | :--- | :---: |
| **1. Development Environment & Docker Setup** | Working `docker-compose.yml` for local PostgreSQL 16, `.env.example`, and Node.js workspace configuration. | ⏳ **Planned** |
| **2. Database Schema & Migration Execution** | Initialized `backend/prisma/schema.prisma` matching [`DATABASE_DESIGN.md`](../phase-02-16-20-aban-diagrams-erd/DATABASE_DESIGN.md), migration scripts generated, and database seeding script (`prisma/seed.ts`). | ⏳ **Planned** |
| **3. NestJS Backend Scaffolding** | Base NestJS Modular Monolith application structure, global exception filters, validation pipes, and Prisma service module. | ⏳ **Planned** |
| **4. React Frontend Scaffolding** | React 18 + TypeScript + Vite project initialized with Tailwind CSS RTL support, `Vazirmatn` Persian font, base layout shells (`AppShell`, `Sidebar`, `Header`), and TanStack Query setup. | ⏳ **Planned** |
| **5. First Vertical Slice: Product Catalog** | `CatalogModule` (`POST /api/products`, `GET /api/products`) connected to PostgreSQL and integrated with the React Product Catalog view and creation modal. | ⏳ **Planned** |
| **6. Integration Verification** | Verified data persistence across page reloads and database queries. | ⏳ **Planned** |
