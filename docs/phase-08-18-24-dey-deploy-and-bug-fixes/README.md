# Phase 8: Deploy, Fix Bugs & Test Main Scenarios

**Work Plan Window:** 18–24 Dey 1405  
**Planned Work:** Deploy, fix bugs, and test the main scenarios  
**Milestone:** Production Staging & Live Verification  
**Status:** ⏳ **Planned / Not Started**  

---

## 1. Overview & Objectives

This phase deploys the complete Depoma system to online cloud infrastructure with HTTPS, validates persistent database operation in production, and resolves functional and visual bugs identified during end-to-end scenario testing.

---

## 2. Planned Deliverables & Engineering Tasks

| Task / Deliverable | Planned Scope & Target Outputs | Status |
| :--- | :--- | :---: |
| **1. Cloud Infrastructure Provisioning** | Live Frontend static hosting, containerized NestJS backend, and managed PostgreSQL 16 provisioned according to [`HOSTING_PLAN.md`](../phase-03-21-24-aban-prototype-api-hosting/HOSTING_PLAN.md). | ⏳ **Planned** |
| **2. Production HTTPS & Domain Configuration** | Domain / approved subdomain configured with automated SSL/TLS certificates and enforced HTTPS redirection. | ⏳ **Planned** |
| **3. Production Database Migration & Seeding** | `npx prisma migrate deploy` executed on the production database with initial administrative accounts. | ⏳ **Planned** |
| **4. End-to-End Scenario Testing on Production** | Automated Playwright runs and manual testing of all primary user journeys and negative security scenarios on the live HTTPS URL. | ⏳ **Planned** |
| **5. Bug Triage & Defect Resolution** | Identification and resolution of edge-case bugs, responsive layout flaws, or API latency bottlenecks. | ⏳ **Planned** |
