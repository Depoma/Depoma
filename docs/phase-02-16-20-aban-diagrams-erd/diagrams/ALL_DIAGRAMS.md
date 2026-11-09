# System Design & Architecture Diagrams

**Product Name:** Depoma — Inventory & Stock Ledger Management System  
**Document Version:** 1.0  

This document consolidates the four core architectural views of the Depoma platform. Editable Mermaid (`.mmd`) and PlantUML (`.puml`) source files are maintained alongside this document in `docs/phase-02-16-20-aban-diagrams-erd/diagrams/`:
1. **UML Use Case Diagram** (Actor roles and system capabilities)
2. **UML Activity Diagram** (End-to-end Stock Out workflow with authorization, row locking, and error branches)
3. **System Architecture Diagram** (Client, Modular Monolith Backend, and PostgreSQL Database tiers)
4. **Entity-Relationship Diagram (ERD)** (Persistent relational schema, keys, and constraints)

---

## 1. UML Use Case Diagram

This view illustrates the interactions of the three system roles (`Admin`, `Warehouse Operator`, `Manager`) with Depoma's functional boundary.

### 1.1. Mermaid Diagram
```mermaid
flowchart LR
    Admin(["👤 System Administrator\n(Admin)"])
    Operator(["👷 Warehouse Operator\n(Operator)"])
    Manager(["📊 Operations Manager\n(Manager)"])

    subgraph Depoma_System ["Depoma Inventory Management System Boundary"]
        direction TB
        UC_Auth("Authenticate & Manage Session\n(Login / Logout)")
        UC_Dash("View Operational Dashboard\n(View Dashboard)")
        
        subgraph Admin_Capabilities ["Master Data & Access Administration (Admin Only)"]
            UC_Users("Manage User Accounts & Roles\n(Manage Users)")
            UC_Assign("Assign Operators to Warehouses\n(Manage Warehouse_Operators)")
            UC_Warehouses("Create & Update Warehouses\n(Manage Warehouses)")
            UC_Catalog("Manage Products & Categories\n(Manage Catalog)")
        end

        subgraph Inventory_Operations ["Core Inventory & Ledger Operations (Warehouse-Scoped)"]
            UC_StockIn("Record Stock Receipt\n(Stock In / Receipt)")
            UC_StockOut("Record Stock Issue\n(Stock Out / Issue)")
            UC_CheckAccess{{"«include»\nVerify Warehouse Access & Stock Availability"}}
        end

        subgraph Audit_Views ["Inventory Visibility & Auditing (Warehouse-Scoped)"]
            UC_ViewBal("View Current Warehouse Balances\n(View Inventory Balances)")
            UC_ViewHist("Inspect & Filter Movement History\n(View Immutable Ledger)")
        end
    end

    Admin --> UC_Auth
    Admin --> UC_Dash
    Admin --> UC_Users
    Admin --> UC_Assign
    Admin --> UC_Warehouses
    Admin --> UC_Catalog
    Admin --> UC_StockIn
    Admin --> UC_StockOut
    Admin --> UC_ViewBal
    Admin --> UC_ViewHist

    Operator --> UC_Auth
    Operator --> UC_Dash
    Operator --> UC_StockIn
    Operator --> UC_StockOut
    Operator --> UC_ViewBal
    Operator --> UC_ViewHist

    Manager --> UC_Auth
    Manager --> UC_Dash
    Manager --> UC_ViewBal
    Manager --> UC_ViewHist

    UC_StockIn -.->|include| UC_CheckAccess
    UC_StockOut -.->|include| UC_CheckAccess
    UC_ViewBal -.->|include| UC_CheckAccess
    UC_ViewHist -.->|include| UC_CheckAccess
```

### 1.2. PlantUML Specification (`use-case.puml`)
```plantuml
@startuml Depoma_Use_Case_Diagram
left to right direction
skinparam packageStyle rectangle
skinparam actorStyle awesome

actor "System Administrator (Admin)" as Admin
actor "Warehouse Operator (Operator)" as Operator
actor "Operations Manager (Manager)" as Manager

rectangle "Depoma Inventory Management System Boundary" {
  usecase (UC-01: Authenticate & Manage Session) as UC_Auth
  usecase (UC-02: View Operational Dashboard) as UC_Dash
  usecase (UC-03: Manage Users & Roles) as UC_Users
  usecase (UC-04: Assign Operators to Warehouses) as UC_Assign
  usecase (UC-05: Manage Warehouses) as UC_Warehouses
  usecase (UC-06: Manage Products & Categories) as UC_Catalog
  usecase (UC-07: Record Stock Receipt - Stock In) as UC_StockIn
  usecase (UC-08: Record Stock Issue - Stock Out) as UC_StockOut
  usecase (UC-09: View Inventory Balances) as UC_ViewBal
  usecase (UC-10: View Movement History Ledger) as UC_ViewHist
  usecase (Verify Operator Warehouse Access) as UC_VerifyAccess
  usecase (Verify Available Balance & Lock Row) as UC_VerifyStock
}

Admin --> UC_Auth
Admin --> UC_Dash
Admin --> UC_Users
Admin --> UC_Assign
Admin --> UC_Warehouses
Admin --> UC_Catalog
Admin --> UC_StockIn
Admin --> UC_StockOut
Admin --> UC_ViewBal
Admin --> UC_ViewHist

Operator --> UC_Auth
Operator --> UC_Dash
Operator --> UC_StockIn
Operator --> UC_StockOut
Operator --> UC_ViewBal
Operator --> UC_ViewHist

Manager --> UC_Auth
Manager --> UC_Dash
Manager --> UC_ViewBal
Manager --> UC_ViewHist

UC_StockIn ..> UC_VerifyAccess : <<include>>
UC_StockOut ..> UC_VerifyAccess : <<include>>
UC_StockOut ..> UC_VerifyStock : <<include>>
UC_ViewBal ..> UC_VerifyAccess : <<include>>
UC_ViewHist ..> UC_VerifyAccess : <<include>>
@enduml
```

---

## 2. UML Activity Diagram — Record Stock Issue (Stock Out Workflow)

This diagram traces the primary business workflow (**Recording a Stock Issue / Stock Out**) across the client, backend security pipeline, and PostgreSQL transactional ledger, explicitly modeling all validation, permission, insufficient stock (`422`), and database failure paths.

```mermaid
flowchart TD
    Start([Start: User on Inventory Balances Screen]) --> SelectAction[Select warehouse, product, and click 'Record Stock Out']
    SelectAction --> FillForm[Enter issue quantity, movement reason, reference number, and notes]
    FillForm --> ClientValidation{Client-Side Form Validation\nQuantity > 0 and required fields valid?}

    ClientValidation -- "No (Form Error)" --> ShowClientErr[Render inline Persian validation errors under fields]
    ShowClientErr --> FillForm

    ClientValidation -- "Yes" --> SendAPI["Dispatch POST /api/inventory/issues to Backend"]
    SendAPI --> CheckAuth{1. JWT Authentication Check\nIs Bearer token valid and active?}

    CheckAuth -- "No (401 Unauthorized)" --> Err401[Reject request and redirect client to Login screen]
    Err401 --> EndNode([End])

    CheckAuth -- "Yes" --> CheckRole{2. Role & Warehouse Ownership Check\nIs user ADMIN, or OPERATOR assigned to\ntarget warehouse in Warehouse_Operators?}

    CheckRole -- "No (403 Forbidden)" --> Err403["Reject request: FORBIDDEN_WAREHOUSE_ACCESS / FORBIDDEN_ROLE"]
    Err403 --> ShowServerErr[Display error alert banner in UI]
    ShowServerErr --> EndNode

    CheckRole -- "Yes (Authorized)" --> ServerVal{3. DTO Schema Validation\nIs quantity > 0 integer and UUIDs valid?}

    ServerVal -- "No (400 Bad Request)" --> Err400["Reject request: VALIDATION_ERROR"]
    Err400 --> ShowServerErr

    ServerVal -- "Yes" --> BeginTx["4. Open Interactive Database Transaction (BEGIN)\nVerify Warehouse & Product status == ACTIVE (FOR SHARE)\nand Lock Balance Row (SELECT ... FOR UPDATE)"]
    BeginTx --> CheckActive{5. Are Warehouse & Product ACTIVE?}

    CheckActive -- "No (422 Inactive)" --> RollbackInactive["Rollback Transaction (ROLLBACK)\nReturn 422 INACTIVE_RESOURCE"]
    RollbackInactive --> ShowServerErr

    CheckActive -- "Yes" --> CheckStock{6. Availability Verification\nIs current_quantity >= requested_quantity?}

    CheckStock -- "No: Insufficient Stock (422)" --> RollbackStock["Rollback Transaction (ROLLBACK)\nReturn 422 INSUFFICIENT_STOCK"]
    RollbackStock --> Err422["Display error alert in modal with current available balance"]
    Err422 --> EndNode

    CheckStock -- "Yes: Sufficient Stock" --> UpdateDB["7. Deduct quantity in Inventory_Balances\nand insert immutable OUT row in Stock_Movements"]
    UpdateDB --> CheckDBCommit{8. Did database write succeed?}

    CheckDBCommit -- "No (500 Database Error)" --> RollbackDB["Rollback Transaction (ROLLBACK)\nPreserve data consistency"]
    RollbackDB --> Err500["Display server error notification to user"]
    Err500 --> EndNode

    CheckDBCommit -- "Yes" --> CommitTx["Commit Transaction (COMMIT)\nReturn HTTP 201 with movement and new balance"]
    CommitTx --> UpdateUI["Display confirmation banner in RTL UI\nRefresh balance table and Jalali movement history"]
    UpdateUI --> EndNode
```

---

## 3. System Architecture Diagram

```mermaid
flowchart TB
    User(["👤 System User\n(Admin / Operator / Manager)"])

    subgraph Browser ["Web Browser (Persian RTL Client SPA)"]
        ReactUI["React 18 + TypeScript UI\n(Tailwind CSS + shadcn/ui)"]
        QueryCache["Server State & API Client\n(TanStack Query + Axios)"]
        FormEngine["Form Validation & Jalali Date Formatter\n(React Hook Form + Zod + Intl/Jalali)"]
        ReactUI --> FormEngine
        ReactUI --> QueryCache
    end

    subgraph Server ["Application Server (NestJS Modular Monolith)"]
        REST_API["REST API Controllers\n(/api/auth, /api/products, /api/warehouses, /api/inventory)"]
        SecPipeline["Security & Validation Pipeline\n(JwtAuthGuard -> RolesGuard -> ValidationPipe)"]

        subgraph Modules ["Encapsulated Domain Modules"]
            M_Auth["Auth & Users Module"]
            M_Catalog["Catalog Module\n(Products & Categories)"]
            M_Warehouse["Warehouses Module\n(Warehouses & Operator Assignments)"]
            M_Inventory["Inventory Module\n(Ledger & Balance Engine)"]
            M_Dashboard["Dashboard Module"]
        end

        PrismaORM["Persistence & Transaction Manager\n(Prisma Client)"]

        REST_API --> SecPipeline
        SecPipeline --> Modules
        M_Inventory -.->|"Verify operator warehouse assignment"| M_Warehouse
        M_Inventory -.->|"Verify active product status"| M_Catalog
        M_Dashboard -.->|"Aggregate KPI metrics"| M_Inventory
        Modules --> PrismaORM
    end

    subgraph Database ["Persistent Data Tier"]
        Postgres[("PostgreSQL 16 Database\n(ACID Transactions, Row Locks, FK/Unique/Check Constraints)")]
    end

    User <-->|"Interacts with Persian RTL Views"| ReactUI
    QueryCache <-->|"HTTPS / JSON + Bearer JWT"| REST_API
    PrismaORM <-->|"SQL / TCP Connection Pool"| Postgres
```

---

## 4. Entity-Relationship Diagram (ERD)

```mermaid
erDiagram
    USERS {
        uuid id PK
        varchar full_name
        varchar email UK
        varchar password_hash
        enum role "ADMIN | OPERATOR | MANAGER"
        enum status "ACTIVE | INACTIVE"
        timestamptz created_at
        timestamptz updated_at
    }

    WAREHOUSES {
        uuid id PK
        varchar name UK
        varchar code UK
        text address
        text description
        enum status "ACTIVE | INACTIVE"
        timestamptz created_at
        timestamptz updated_at
    }

    WAREHOUSE_OPERATORS {
        uuid id PK
        uuid warehouse_id FK "UNIQUE(warehouse_id, user_id)"
        uuid user_id FK "UNIQUE(warehouse_id, user_id)"
        uuid assigned_by FK
        timestamptz assigned_at
    }

    CATEGORIES {
        uuid id PK
        varchar name UK
        text description
        timestamptz created_at
        timestamptz updated_at
    }

    PRODUCTS {
        uuid id PK
        varchar name
        varchar sku UK
        uuid category_id FK
        varchar unit
        integer minimum_stock "CHECK >= 0"
        text description
        enum status "ACTIVE | ARCHIVED"
        timestamptz created_at
        timestamptz updated_at
    }

    INVENTORY_BALANCES {
        uuid id PK
        uuid product_id FK "UNIQUE(product_id, warehouse_id)"
        uuid warehouse_id FK "UNIQUE(product_id, warehouse_id)"
        integer quantity "CHECK >= 0"
        timestamptz updated_at
    }

    STOCK_MOVEMENTS {
        uuid id PK
        uuid product_id FK
        uuid warehouse_id FK
        enum movement_type "IN | OUT"
        integer quantity "CHECK > 0"
        integer balance_after "CHECK >= 0"
        varchar reference_no
        varchar reason
        text description
        uuid created_by FK
        timestamptz created_at
    }

    USERS ||--o{ WAREHOUSE_OPERATORS : "assigned to"
    USERS ||--o{ WAREHOUSE_OPERATORS : "granted by"
    WAREHOUSES ||--o{ WAREHOUSE_OPERATORS : "managed by"
    CATEGORIES ||--o{ PRODUCTS : "categorizes"
    PRODUCTS ||--o{ INVENTORY_BALANCES : "has balance"
    WAREHOUSES ||--o{ INVENTORY_BALANCES : "stores"
    PRODUCTS ||--o{ STOCK_MOVEMENTS : "recorded in"
    WAREHOUSES ||--o{ STOCK_MOVEMENTS : "logged at"
    USERS ||--o{ STOCK_MOVEMENTS : "created by"
```
