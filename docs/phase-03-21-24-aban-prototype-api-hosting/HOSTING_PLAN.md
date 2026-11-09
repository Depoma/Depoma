# Cloud Infrastructure, Deployment & Hosting Plan

**Product Name:** Depoma — Inventory & Stock Ledger Management System  
**Document Version:** 1.0  

This document evaluates cloud hosting providers, network accessibility constraints, resource allocations, and estimated monthly operational costs for deploying Depoma in production over **HTTPS**.

---

## 1. Infrastructure Components

A production deployment of Depoma requires four core infrastructure services:
1. **Frontend Static/SPA Hosting:** Serves the compiled React/Vite bundle over a CDN or reverse proxy with SPA fallback routing (`index.html`) and TLS termination.
2. **Backend Application Runtime:** Runs the containerized Node.js / NestJS Modular Monolith API with zero cold-start latency.
3. **Managed Relational Database:** Runs **PostgreSQL 16** with persistent SSD storage, automated daily backups, and private internal networking to the backend container.
4. **Domain / Subdomain & TLS Certificate:** Provides HTTPS encryption via automated Let's Encrypt certificates on either a platform subdomain or a custom `.ir` / `.com` domain.

---

## 2. Cloud Provider Comparison: Domestic PaaS vs. International Cloud

Because Depoma's primary initial user base operates within Iran, infrastructure selection must account for both technical capabilities and regional network reliability (latency, IP sanctions, ISP routing, and payment rails).

| Evaluation Criterion | Option A: Domestic Cloud PaaS<br>(Liara / ArvanCloud Container PaaS) | Option B: International Cloud Platforms<br>(Vercel + Render/Railway + Neon/Supabase) |
| :--- | :--- | :--- |
| **1. Estimated Monthly Cost** | **~300,000 to 450,000 IRR (Toman) / month** *(Approx.)*<br>Covers dedicated Frontend, NestJS Backend, and Managed PostgreSQL instances. | **\$0 (Free Tiers)** or **\$5 to \$15 / month** (Hobby Tiers), plus international currency exchange fees. |
| **2. Resource Guarantees** | Dedicated, always-on CPU (0.5–1 vCPU), RAM (512 MB–1 GB), and persistent SSD storage without forced sleep cycles. | Free-tier containers on Render spin down after 15 minutes of inactivity, causing **30–50 second cold-start delays** on initial API requests. Neon free tier caps storage at 0.5 GB. |
| **3. PostgreSQL Support** | Fully managed PostgreSQL 16 with automated backups and private internal network connectivity (`< 2ms` latency between API and DB). | High-quality managed PostgreSQL (Neon / Supabase), but requires external connection pooling (`PgBouncer`) if API and DB regions differ. |
| **4. Unified Deployment Experience** | Frontend, Backend, and PostgreSQL are provisioned and monitored inside a **single unified control panel**. | Typically split across 2–3 separate providers (e.g., Vercel for UI, Render for API, Neon for DB), requiring cross-origin (CORS) management across domains. |
| **5. Deployment Automation** | Automated deployments via GitHub Actions or CLI (`liara deploy`) with zero-downtime rolling updates and pre-start `prisma migrate deploy`. | Native GitHub integration, subject to regional IP restrictions on build webhooks or account verification. |
| **6. Regional Network Accessibility & Sanctions** | **Zero sanctions or ISP filtering restrictions.** Directly accessible across all domestic enterprise and mobile networks with `< 20ms` round-trip latency. | **High operational risk for Iranian users:** Default `*.vercel.app` domains face intermittent ISP filtering, and providers such as Supabase, Render, and Neon block Iranian IPs or require international credit cards. |
| **7. Production Uptime & Reliability** | Always-on containers with predictable low latency for real-time warehouse operations. | Vulnerable to cross-border routing instability and free-tier cold starts. |
| **8. Commercial Scalability** | Seamless vertical scaling of CPU/RAM/Storage and one-click custom domain binding for commercial SME rollouts. | Strong global scalability, but requires sustained international billing infrastructure. |

---

## 3. Selected Deployment Architecture

Based on the operational trade-off analysis, Depoma adopts the following two-environment strategy:

### 3.1. Local Development & CI Environment
* **Docker Compose (`docker-compose.yml`):** Provisions a local PostgreSQL 16 instance so engineers can run migrations, seed scripts, and integration tests identically across workstations without external network dependencies.

### 3.2. Production Environment
* **Primary Production Provider:** **Liara Cloud PaaS** (or ArvanCloud Cloud Container).
* **Architectural Justification:**
  1. **Uninterrupted Regional Availability:** Guarantees that warehouse operators and managers can access the application over standard connections without VPN dependencies or ISP-level domain blocks.
  2. **Zero Cold-Start Latency:** Dedicated container allocation ensures immediate execution of time-sensitive stock receipt and stock issue transactions.
  3. **Private Internal Database Networking:** Co-locating the NestJS application container and the PostgreSQL instance within the same private datacenter network minimizes lock duration during `SELECT ... FOR UPDATE` inventory transactions.
  4. **Automated HTTPS:** Native TLS 1.3 certificates are automatically provisioned and renewed on both default platform subdomains (`https://depoma-app.liara.run`) and custom domains.

*(Portability Note: Because Depoma uses standard Dockerfiles, Prisma ORM, and environment variables, the entire stack remains 100% cloud-agnostic and can be deployed to Railway, Render, AWS, or on-premise Linux servers without code changes).*

---

## 4. Production Resource Allocation, Cost Breakdown & Limits

The table below outlines the baseline production sizing and estimated monthly costs for operating Depoma:

| Infrastructure Tier | Platform Service | Resource Allocation | Estimated Monthly Cost *(Approx.)* | Operational Limits & Considerations |
| :--- | :--- | :--- | :---: | :--- |
| **Frontend SPA** | Liara Static / React Hosting | 0.25 vCPU / 256 MB RAM / 5 GB Bandwidth | ~60,000 – 90,000 Toman | Serves static compiled assets; easily handles `> 10,000` monthly sessions. |
| **Backend API** | Liara Node.js Container (NestJS) | 0.5 vCPU / 512 MB RAM / SSD | ~120,000 – 160,000 Toman | Single-instance stateless process; supports vertical scaling up to 8 vCPU / 16 GB RAM as transaction volume grows. |
| **Database** | Liara Managed PostgreSQL 16 | 0.5 vCPU / 512 MB RAM / 5 GB Persistent SSD | ~120,000 – 150,000 Toman | Default connection limit (`max_connections = 100`), managed via Prisma's connection pool (`connection_limit=10`). |
| **TLS & Domain** | Platform Subdomain (`*.liara.run`) + Let's Encrypt HTTPS | Automated SSL/TLS Certificate | **0 Toman (Included)**<br>*(Optional `.ir` domain: ~150,000 Toman/year)* | Full HTTPS enforcement with automatic HTTP-to-HTTPS redirection. |
| **Total Estimated Cost** | — | — | **~300,000 – 400,000 Toman / month** | Predictable, low-cost footprint suitable for initial production operation. |
