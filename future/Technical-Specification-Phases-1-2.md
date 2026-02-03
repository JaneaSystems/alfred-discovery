# AX3 Digital Transformation: Technical Specification

> **Document Type**: Technical Specification  
> **Scope**: Phase 1 (Quick-Win Automation) & Phase 2 (Architecture Evolution)  
> **Date**: February 3, 2026  
> **Status**: DRAFT — Pending Technical Discovery  
> **Revision**: 2.0 — Updated with infrastructure flexibility constraints

---

## Technical Constraints & Flexibility

| System | Change Difficulty | Notes |
|--------|-------------------|-------|
| **AS400** | 🟡 Small changes possible | Limited scope, but not blocked |
| **portal.ax3.com** | 🔴 Possible but hard | Minimize portal changes; wrap rather than modify |
| **SQL Server** | 🟢 Easy to change | New tables, stored procedures, batches all feasible |
| **New Infrastructure** | 🟢 Unlimited | New VMs, services, APIs, orchestration systems available |

**Strategic Implication**: Build new systems around legacy components rather than modifying them. Use SQL Server as the orchestration hub. Create new APIs and services to abstract legacy complexity.

---

## Document Conventions

| Marker | Meaning |
|--------|---------|
| ✅ **HIGH CONFIDENCE** | Based on confirmed information from Q&A or meeting notes |
| ⚠️ **MEDIUM CONFIDENCE** | Reasonable inference, needs validation |
| ❓ **LOW CONFIDENCE** | Assumption requiring discovery before implementation |
| 🔴 **OPEN QUESTION** | Blocks detailed design until answered |

---

## Table of Contents

1. [Phase 1: Quick-Win Automation](#phase-1-quick-win-automation)
   - [1.1 Digital Product Orchestration Service](#11-digital-product-orchestration-service)
   - [1.2 Template Auto-Generator](#12-template-auto-generator)
   - [1.3 Pre-Upload Validator](#13-pre-upload-validator)
   - [1.4 File Renamer Tool](#14-file-renamer-tool)
   - [1.5 Unified Asset Uploader](#15-unified-asset-uploader)
   - [1.6 FTP Auto-Sync](#16-ftp-auto-sync)
2. [Phase 2: Architecture Evolution](#phase-2-architecture-evolution)
   - [2.1 Digital-First SKU Path](#21-digital-first-sku-path)
   - [2.2 ax3.com-Only Distribution Flag](#22-ax3com-only-distribution-flag)
   - [2.3 New Digital Product API](#23-new-digital-product-api)
   - [2.4 Minimal Portal Modifications](#24-minimal-portal-modifications)
3. [Proposed Architecture](#proposed-architecture)
4. [Technical Dependencies](#technical-dependencies)
5. [Open Questions](#open-questions)
6. [Risk Register](#risk-register)

---

## Phase 1: Quick-Win Automation

**Objective**: Reduce manual effort in current workflow by ~60% while laying groundwork for Phase 2 architecture.

**Approach**: Build new services on SQL Server and new infrastructure. Wrap legacy systems rather than modify them. Create foundation for digital-first path.

**Estimated Duration**: 4-6 weeks

---

### 1.1 Digital Product Orchestration Service

**Purpose**: Central service that coordinates all digital product workflows, tracks state, and orchestrates automation tools.

**Why This First**: With SQL Server being easy to modify and new infrastructure available, we can build a proper orchestration layer that all other tools connect to. This becomes the foundation for Phase 2.

#### Functional Requirements

| Requirement | Description | Confidence |
|-------------|-------------|------------|
| FR-1.1.1 | Track digital product state across entire workflow | ✅ HIGH |
| FR-1.1.2 | Queue products for processing (template, validation, upload) | ✅ HIGH |
| FR-1.1.3 | Provide REST API for tool integration | ✅ HIGH |
| FR-1.1.4 | Dashboard showing pipeline status | ✅ HIGH |
| FR-1.1.5 | Audit log of all actions | ✅ HIGH |
| FR-1.1.6 | Trigger downstream actions automatically | ✅ HIGH |

#### Technical Design

```
┌─────────────────────────────────────────────────────────────────────────────┐
│              DIGITAL PRODUCT ORCHESTRATION SERVICE                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  NEW SQL SERVER TABLES                                                       │
│  ├── dpo_products          (digital product master)                         │
│  ├── dpo_workflow_state    (current state per product)                      │
│  ├── dpo_workflow_queue    (pending actions)                                │
│  ├── dpo_audit_log         (all state transitions)                          │
│  └── dpo_asset_registry    (files, locations, upload status)                │
│                                                                              │
│  NEW SERVICE (VM or Container)                                               │
│  ├── REST API (ASP.NET Core or FastAPI)                                     │
│  │   ├── POST /products              (queue new product)                    │
│  │   ├── GET  /products/{id}/status  (get workflow state)                   │
│  │   ├── POST /products/{id}/advance (move to next step)                    │
│  │   ├── GET  /queue                 (pending work items)                   │
│  │   └── GET  /dashboard             (pipeline overview)                    │
│  │                                                                           │
│  ├── Background Workers                                                      │
│  │   ├── TemplateGeneratorWorker                                            │
│  │   ├── ValidatorWorker                                                    │
│  │   ├── AssetUploaderWorker                                                │
│  │   └── FTPSyncWorker                                                      │
│  │                                                                           │
│  └── Event Bus (SQL Server Service Broker or simple polling)                │
│                                                                              │
│  WORKFLOW STATES                                                             │
│  ┌────────┐   ┌──────────┐   ┌───────────┐   ┌────────────┐   ┌──────────┐ │
│  │PENDING │──▶│TEMPLATE  │──▶│VALIDATED  │──▶│PORTAL      │──▶│ASSETS    │ │
│  │        │   │GENERATED │   │           │   │UPLOADED    │   │UPLOADED  │ │
│  └────────┘   └──────────┘   └───────────┘   └────────────┘   └──────────┘ │
│                                     │                               │       │
│                                     ▼                               ▼       │
│                              ┌───────────┐                   ┌──────────┐  │
│                              │VALIDATION │                   │   LIVE   │  │
│                              │FAILED     │                   │          │  │
│                              └───────────┘                   └──────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Infrastructure

| Component | Implementation | Confidence |
|-----------|----------------|------------|
| Database | SQL Server (new tables in existing instance) | ✅ HIGH |
| API Service | New VM or Azure App Service | ✅ HIGH |
| Workers | Windows Services or Container workloads | ✅ HIGH |
| Dashboard | Simple web UI (React/Vue or Blazor) | ✅ HIGH |

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| SQL schema design & creation | 2-3 days | ✅ HIGH |
| REST API implementation | 4-5 days | ✅ HIGH |
| Worker framework | 3-4 days | ✅ HIGH |
| Dashboard UI | 3-4 days | ✅ HIGH |
| Testing & deployment | 2-3 days | ✅ HIGH |
| **Total** | **14-19 days** | |

---

### 1.2 Template Auto-Generator

**Purpose**: Eliminate manual data entry into Excel templates by auto-populating from source systems.

**Integration**: Runs as a worker in the Orchestration Service, or callable via API.

#### Functional Requirements

| Requirement | Description | Confidence |
|-------------|-------------|------------|
| FR-1.2.1 | Generate Performance_Template.xlsx from FileMaker/SQL data | ⚠️ MEDIUM |
| FR-1.2.2 | Generate Sheets_Template.xls from FileMaker/SQL data | ⚠️ MEDIUM |
| FR-1.2.3 | Generate Choral_Template.xls from FileMaker/SQL data | ⚠️ MEDIUM |
| FR-1.2.4 | Support batch generation (multiple SKUs per template) | ✅ HIGH |
| FR-1.2.5 | Handle _CC (Condensed Score) special cases automatically | ✅ HIGH |
| FR-1.2.6 | Ensure UPC consistency across all parts of a set | ✅ HIGH |
| FR-1.2.7 | Store generated template in dpo_asset_registry | ✅ HIGH |

#### Data Sources

| Source | Data Retrieved | Access Method | Confidence |
|--------|----------------|---------------|------------|
| SQL Server (tblapproducts) | Product metadata for Performance/Choral | Direct SQL query | ✅ HIGH |
| SQL Server (nightly FM data) | All FileMaker data already synced | Direct SQL query | ✅ HIGH |
| WebCRD | Performance Music metadata | ⚠️ MEDIUM — needs discovery |

**Key Insight**: FileMaker data is already in SQL Server via the nightly CSV export. We can query SQL directly instead of accessing FileMaker. This removes a major uncertainty.

#### Technical Design

```
┌──────────────────────────────────────────────────────────────────┐
│                    TEMPLATE AUTO-GENERATOR                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  TRIGGER                                                          │
│  ├── API call: POST /templates/generate                          │
│  ├── Queue pickup from dpo_workflow_queue                        │
│  └── Manual UI trigger from dashboard                            │
│                                                                   │
│  INPUT                                                            │
│  ├── SKU list (from queue or request)                            │
│  └── Product type (auto-detected from SKU prefix)                │
│                                                                   │
│  PROCESSING                                                       │
│  ├── Query SQL Server for product metadata                       │
│  │   └── Join tblapproducts + pending tables                     │
│  ├── Apply _CC handling rules                                    │
│  ├── Validate UPC consistency                                     │
│  ├── Map fields to template columns                              │
│  └── Generate Excel file (exclude column AK)                     │
│                                                                   │
│  OUTPUT                                                           │
│  ├── Excel template saved to file share / blob storage           │
│  ├── dpo_asset_registry updated with file location               │
│  └── Workflow state advanced to TEMPLATE_GENERATED               │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

🔴 **OPEN QUESTION OQ-1**: Need access to FileMaker field spreadsheets to complete field mapping. However, we can also reverse-engineer from existing SQL tables.

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| SQL query development | 2-3 days | ✅ HIGH |
| Template generation logic | 3-4 days | ✅ HIGH |
| Field mapping implementation | 3-5 days | ⚠️ MEDIUM |
| Orchestration integration | 1-2 days | ✅ HIGH |
| Testing & validation | 2-3 days | ✅ HIGH |
| **Total** | **11-17 days** | |

---

### 1.3 Pre-Upload Validator

**Purpose**: Catch data errors before portal upload, reducing failed imports and rework.

**Integration**: Runs automatically after template generation, or on-demand via API.

#### Functional Requirements

| Requirement | Description | Confidence |
|-------------|-------------|------------|
| FR-1.3.1 | Validate completed Excel template before upload | ✅ HIGH |
| FR-1.3.2 | Check UPC consistency across set parts | ✅ HIGH |
| FR-1.3.3 | Verify _CC sort order = 1 and page count = 1 | ✅ HIGH |
| FR-1.3.4 | Validate required fields are populated | ✅ HIGH |
| FR-1.3.5 | Check for column AK presence (flag for removal) | ✅ HIGH |
| FR-1.3.6 | Verify ref_sku exists in SQL Server | ✅ HIGH |
| FR-1.3.7 | Cross-check AS400 product status (via SQL replica) | ⚠️ MEDIUM |
| FR-1.3.8 | Block workflow if validation fails | ✅ HIGH |

#### Validation Rules

| Rule | Description | Confidence |
|------|-------------|------------|
| VR-01 | All parts in a set must share the same UPC | ✅ HIGH |
| VR-02 | _CC parts must have sort order "1" | ✅ HIGH |
| VR-03 | _CC parts must have page count "1" | ✅ HIGH |
| VR-04 | Column AK causes import failure if present | ✅ HIGH |
| VR-05 | ref_sku must exist in tblapproducts or pending table | ✅ HIGH |
| VR-06 | AS400 Status must be CUR, TOP, NEW, or PND | ⚠️ MEDIUM |
| VR-07 | OutputToWeb must be JJ114 or DIGIONLY | ✅ HIGH |

**Approach for Unknown Rules**: Build extensible validation framework. Add rules incrementally as we discover them from portal failures. Log all portal rejections to identify missing rules.

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| Validation framework | 2-3 days | ✅ HIGH |
| Known rule implementation | 2-3 days | ✅ HIGH |
| SQL cross-checks | 1-2 days | ✅ HIGH |
| Orchestration integration | 1 day | ✅ HIGH |
| Error reporting UI | 1-2 days | ✅ HIGH |
| **Total** | **7-11 days** | |

---

### 1.4 File Renamer Tool

**Purpose**: Automate PDF file renaming to match digital SKU naming conventions.

**Integration**: Part of asset processing pipeline in Orchestration Service.

#### Functional Requirements

| Requirement | Description | Confidence |
|-------------|-------------|------------|
| FR-1.4.1 | Download filename mapping from report server | ✅ HIGH |
| FR-1.4.2 | Batch rename PDFs according to mapping | ✅ HIGH |
| FR-1.4.3 | Validate source files exist before rename | ✅ HIGH |
| FR-1.4.4 | Generate rename log for audit trail | ✅ HIGH |
| FR-1.4.5 | Handle conflicts (duplicate target names) | ✅ HIGH |
| FR-1.4.6 | Update dpo_asset_registry with renamed file paths | ✅ HIGH |

#### Data Source

| Source | URL Pattern | Confidence |
|--------|-------------|------------|
| Report Server | `alfredawssql06.alfredpub.com/.../Digital%20Uploads%20-%20Rename%20Files` | ✅ HIGH |

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| Report server integration | 1-2 days | ✅ HIGH |
| File matching & rename logic | 2-3 days | ✅ HIGH |
| Orchestration integration | 1 day | ✅ HIGH |
| **Total** | **4-6 days** | |

---

### 1.5 Unified Asset Uploader

**Purpose**: Single action to upload PDFs to multiple destinations (currently done 3x manually).

**Integration**: Worker in Orchestration Service, triggered after file rename.

#### Functional Requirements

| Requirement | Description | Confidence |
|-------------|-------------|------------|
| FR-1.5.1 | Upload PDFs to AWS S3 (alfred-dsm-pdfs bucket) | ✅ HIGH |
| FR-1.5.2 | Upload PDFs to Dropbox archive | ✅ HIGH |
| FR-1.5.3 | Upload cover images to AWS S3 (alfred-catfiles bucket) | ✅ HIGH |
| FR-1.5.4 | Track upload status in dpo_asset_registry | ✅ HIGH |
| FR-1.5.5 | Resume failed uploads automatically | ✅ HIGH |
| FR-1.5.6 | Update workflow state on completion | ✅ HIGH |

#### Destinations

| Destination | Protocol | Automatable | Confidence |
|-------------|----------|-------------|------------|
| AWS S3 (alfred-dsm-pdfs) | AWS SDK | ✅ Yes | ✅ HIGH |
| AWS S3 (alfred-catfiles) | AWS SDK | ✅ Yes | ✅ HIGH |
| Dropbox | Dropbox API | ✅ Yes | ✅ HIGH |
| Legato | Web portal only | ❌ No API | ✅ HIGH |

⚠️ **LIMITATION**: Legato upload cannot be automated (no API). Reduces 3x manual to 1x manual + 1x automated.

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| AWS S3 integration | 2-3 days | ✅ HIGH |
| Dropbox integration | 2-3 days | ✅ HIGH |
| Status tracking & retry | 1-2 days | ✅ HIGH |
| Orchestration integration | 1 day | ✅ HIGH |
| **Total** | **6-9 days** | |

---

### 1.6 FTP Auto-Sync

**Purpose**: Automatically push MRID files to dealer FTP after MRID upload to portal.

**Integration**: Triggered by Orchestration Service when MRID data changes in SQL.

#### Functional Requirements

| Requirement | Description | Confidence |
|-------------|-------------|------------|
| FR-1.6.1 | Detect new MRID data in SQL Server | ✅ HIGH |
| FR-1.6.2 | Generate MR_Upload_Dealer_Full.txt | ⚠️ MEDIUM |
| FR-1.6.3 | Generate MR_Upload_Dealer.txt (delta) | ⚠️ MEDIUM |
| FR-1.6.4 | Upload to ftp1.ax3.com/Dealerftp | ✅ HIGH |
| FR-1.6.5 | Log upload in audit trail | ✅ HIGH |

**New Approach**: Since we can add SQL Server triggers/tables, we can:
1. Add `dpo_mrid_changelog` table to track MRID changes
2. SQL trigger populates changelog on MRID insert/update
3. Worker polls changelog and generates files

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| SQL changelog table + trigger | 1-2 days | ✅ HIGH |
| File generation logic | 2-3 days | ⚠️ MEDIUM |
| FTP upload | 1-2 days | ✅ HIGH |
| Orchestration integration | 1 day | ✅ HIGH |
| **Total** | **5-8 days** | |

---

### Phase 1 Summary

| Component | Effort (days) | Confidence |
|-----------|---------------|------------|
| 1.1 Orchestration Service | 14-19 | ✅ HIGH |
| 1.2 Template Auto-Generator | 11-17 | ⚠️ MEDIUM |
| 1.3 Pre-Upload Validator | 7-11 | ✅ HIGH |
| 1.4 File Renamer Tool | 4-6 | ✅ HIGH |
| 1.5 Unified Asset Uploader | 6-9 | ✅ HIGH |
| 1.6 FTP Auto-Sync | 5-8 | ⚠️ MEDIUM |
| **Phase 1 Total** | **47-70 days** | |

**Recommended Team**: 2 developers for 6-8 weeks

**Key Deliverables**:
- Central orchestration service with dashboard
- End-to-end automation from SKU list to asset upload
- Only manual step remaining: Legato upload (no API)

---

## Phase 2: Architecture Evolution

**Objective**: Enable digital products as first-class entities, not derived from physical products.

**Approach**: 
- Build new APIs in SQL Server and new services
- Minimal portal changes (wrap rather than modify)
- Small AS400 changes for digital-native records
- Create new paths that coexist with legacy flows

**Estimated Duration**: 8-10 weeks

---

### 2.1 Digital-First SKU Path

**Purpose**: Allow creation of digital products without requiring a dummy physical SKU.

**Key Insight**: With SQL Server being easy to modify and new infrastructure available, we can build a parallel path that generates digital SKUs independently, then pushes minimal records to AS400 and portal.

#### Functional Requirements

| Requirement | Description | Confidence |
|-------------|-------------|------------|
| FR-2.1.1 | New SQL table: `digital_masters` (source of truth for digital-first products) | ✅ HIGH |
| FR-2.1.2 | Generate digital SKU directly in SQL without ref_sku | ✅ HIGH |
| FR-2.1.3 | Create minimal AS400 record for sales tracking | ⚠️ MEDIUM |
| FR-2.1.4 | Create minimal FileMaker/SQL record for metadata | ✅ HIGH |
| FR-2.1.5 | ax3.com displays digital-master products | ⚠️ MEDIUM |
| FR-2.1.6 | Existing ~500 dummy SKU products unchanged | ✅ HIGH |

#### Architecture: New Digital Product API

**Key Insight**: Rather than modifying portal.ax3.com extensively (hard), we build a **new API service** that:
1. Accepts digital product creation requests
2. Generates digital SKU in SQL Server
3. Pushes minimal record to AS400 (small change)
4. Generates template for portal upload OR bypasses portal entirely

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    NEW: DIGITAL PRODUCT API                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  NEW INFRASTRUCTURE (VM / Container)                                         │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                      Digital Product API                                │ │
│  │  ├── POST /digital-products          (create digital-first product)   │ │
│  │  ├── POST /digital-products/from-physical (convert existing)          │ │
│  │  ├── GET  /digital-products/{sku}    (get product details)            │ │
│  │  └── PUT  /digital-products/{sku}    (update metadata)                │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                           │                                                  │
│           ┌───────────────┼───────────────┬──────────────────┐              │
│           ▼               ▼               ▼                  ▼              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐    │
│  │ SQL Server  │  │   AS400     │  │ Royalty Sys │  │ portal.ax3.com  │    │
│  │ (new tables)│  │(min record) │  │  (rights)   │  │ (optional path) │    │
│  │             │  │             │  │             │  │                 │    │
│  │ digital_    │  │ Small chg:  │  │ Accept dig  │  │ OR bypass via   │    │
│  │ masters     │  │ accept dig  │  │ rights w/o  │  │ direct SQL      │    │
│  │ table       │  │ product type│  │ phys parent │  │ writes          │    │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────────┘    │
│                                                                              │
│  WORKFLOW OPTIONS                                                            │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                              │
│  Option A: API → Generate Template → Portal Upload (safer, uses portal)     │
│  Option B: API → Direct SQL Writes → Skip Portal (faster, more control)     │
│                                                                              │
│  Recommendation: Start with Option A, migrate to B after validation         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### New SQL Tables

```sql
-- New table: digital_masters (source of truth for digital-first products)
CREATE TABLE digital_masters (
    digital_sku         VARCHAR(20) PRIMARY KEY,  -- e.g., 00-PB-0012345
    product_type        VARCHAR(10) NOT NULL,     -- PB, PS, PK, PO, PC, PR
    title               NVARCHAR(500) NOT NULL,
    composer_artist     NVARCHAR(500),
    description         NVARCHAR(MAX),
    price               DECIMAL(10,2),
    ax3_only            BIT DEFAULT 0,            -- Secondary objective flag
    as400_sku           VARCHAR(20),              -- Minimal AS400 record ref
    royalty_configured  BIT DEFAULT 0,
    status              VARCHAR(20) DEFAULT 'DRAFT',
    created_date        DATETIME DEFAULT GETDATE(),
    created_by          VARCHAR(100),
    -- Additional metadata fields as needed
);

-- New table: digital_master_contributors
CREATE TABLE digital_master_contributors (
    id                  INT IDENTITY PRIMARY KEY,
    digital_sku         VARCHAR(20) FOREIGN KEY REFERENCES digital_masters,
    contributor_name    NVARCHAR(200),
    role                VARCHAR(50),  -- Composer, Arranger, Artist, etc.
    sort_order          INT
);

-- New table: digital_master_assets  
CREATE TABLE digital_master_assets (
    id                  INT IDENTITY PRIMARY KEY,
    digital_sku         VARCHAR(20) FOREIGN KEY REFERENCES digital_masters,
    asset_type          VARCHAR(20),  -- PDF, COVER, PREVIEW
    file_name           VARCHAR(500),
    s3_url              VARCHAR(1000),
    dropbox_path        VARCHAR(1000),
    upload_status       VARCHAR(20)
);
```

#### Minimal AS400 Record (Small Change)

🔴 **OPEN QUESTION OQ-2**: Need to confirm with Purchasing which fields are truly required. Proposed minimal record:

| Field | Value | Notes |
|-------|-------|-------|
| Pub Number | Generated | New digital series |
| Item Number | Generated | Sequential |
| Title | From API | Required |
| Product Type Code | "D" (new) | **Small AS400 change** |
| Price | From API | Required for sales |
| Royalty Flag | "Y" | Enable royalty tracking |
| Status | "CUR" | Active |
| Territory | Default or specified | |

**AS400 Change Required**: Add new Product Type Code "D" for digital-native products. This is a small configuration change, not a code change.

#### Implementation Phases

| Sub-Phase | Scope | Confidence | Duration |
|-----------|-------|------------|----------|
| 2.1.1 | SQL tables + API skeleton | ✅ HIGH | 2 weeks |
| 2.1.2 | Books/Sheets path (lowest coupling) | ⚠️ MEDIUM | 2 weeks |
| 2.1.3 | AS400 integration | ⚠️ MEDIUM | 1-2 weeks |
| 2.1.4 | ax3.com display validation | ⚠️ MEDIUM | 1-2 weeks |
| 2.1.5 | Performance/Choral paths | ⚠️ MEDIUM | 2-3 weeks |

#### Effort Estimate (Revised)

| Task | Estimate | Confidence |
|------|----------|------------|
| SQL schema design + implementation | 3-5 days | ✅ HIGH |
| Digital Product API | 8-12 days | ✅ HIGH |
| AS400 minimal integration | 5-8 days | ⚠️ MEDIUM |
| ax3.com validation/fixes | 5-10 days | ⚠️ MEDIUM |
| Royalty system integration | 3-5 days | ⚠️ MEDIUM |
| Testing & validation | 8-10 days | ✅ HIGH |
| **Total** | **32-50 days** | |

---

### 2.2 ax3.com-Only Distribution Flag

**Purpose**: Allow products to be sold only on ax3.com, not distributed to Legato/dealers.

**Simplification**: With new SQL tables, this becomes trivial—just a column.

#### Implementation

```sql
-- Already included in digital_masters table
ax3_only BIT DEFAULT 0

-- For existing products, add to existing digital product table
ALTER TABLE [existing_digital_products_table] 
ADD ax3_only BIT DEFAULT 0;
```

#### Export Logic Changes

```sql
-- Legato Export (modify existing query)
SELECT * FROM digital_products 
WHERE ax3_only = 0  -- Exclude ax3-only products

-- MRID Generation (modify existing query)  
SELECT * FROM digital_products 
WHERE ax3_only = 0 AND mrid IS NOT NULL
```

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| SQL schema change | 1 day | ✅ HIGH |
| Export query modifications | 2-3 days | ✅ HIGH |
| API support | 1-2 days | ✅ HIGH |
| Testing | 2-3 days | ✅ HIGH |
| **Total** | **6-9 days** | |

---

### 2.3 New Digital Product API

**Purpose**: Provide modern interface for digital product management, abstracting legacy system complexity.

#### API Endpoints

| Endpoint | Method | Description | Confidence |
|----------|--------|-------------|------------|
| `/digital-products` | POST | Create digital-first product | ✅ HIGH |
| `/digital-products/{sku}` | GET | Get product details | ✅ HIGH |
| `/digital-products/{sku}` | PUT | Update product metadata | ✅ HIGH |
| `/digital-products/{sku}/assets` | POST | Upload/register assets | ✅ HIGH |
| `/digital-products/{sku}/publish` | POST | Trigger go-live workflow | ✅ HIGH |
| `/digital-products/from-physical/{sku}` | POST | Convert physical to digital | ✅ HIGH |
| `/digital-products/bulk` | POST | Batch operations | ✅ HIGH |

#### Integration with Legacy Systems

```
┌────────────────────────────────────────────────────────────────────────┐
│                         API INTEGRATION LAYER                           │
├────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  API receives request                                                   │
│       │                                                                 │
│       ▼                                                                 │
│  ┌─────────────┐                                                       │
│  │ Validation  │ (replaces portal validation - we control it)          │
│  └──────┬──────┘                                                       │
│         │                                                               │
│         ▼                                                               │
│  ┌─────────────┐     ┌──────────────────────────────────────────────┐ │
│  │ SQL Server  │────▶│ Write to digital_masters + related tables    │ │
│  └──────┬──────┘     └──────────────────────────────────────────────┘ │
│         │                                                               │
│         ├──────────▶ AS400 Adapter (small changes to accept dig SKU)  │
│         │                                                               │
│         ├──────────▶ Royalty System (via existing AS400 nightly sync) │
│         │                                                               │
│         └──────────▶ Portal (Option A) OR Direct SQL (Option B)       │
│                              │                                          │
│                              ▼                                          │
│                         ┌─────────┐                                    │
│                         │ ax3.com │                                    │
│                         └─────────┘                                    │
│                                                                         │
└────────────────────────────────────────────────────────────────────────┘
```

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| API design & scaffolding | 3-4 days | ✅ HIGH |
| Core endpoints | 8-10 days | ✅ HIGH |
| AS400 adapter | 4-6 days | ⚠️ MEDIUM |
| Validation engine | 3-4 days | ✅ HIGH |
| Testing & documentation | 4-5 days | ✅ HIGH |
| **Total** | **22-29 days** | |

---

### 2.4 Minimal Portal Modifications

**Purpose**: Make smallest possible portal changes to support new workflows.

**Philosophy**: Wrap, don't rewrite. Portal changes are hard, so minimize them.

#### Required Portal Changes

| Change | Description | Effort | Confidence |
|--------|-------------|--------|------------|
| Accept null ref_sku | Allow template upload without physical parent | ⚠️ 2-4 days | ⚠️ MEDIUM |
| ax3_only checkbox | Add single checkbox to template UI | ⚠️ 1-2 days | ⚠️ MEDIUM |
| DIGITALMASTER OutputToWeb | Recognize new value | ⚠️ 1-2 days | ⚠️ MEDIUM |

#### Alternative: Bypass Portal Entirely

If portal changes prove too difficult, the new API can:
1. Write directly to the same SQL tables portal writes to
2. Trigger the same downstream processes
3. Completely bypass portal for digital-first products

**Risk**: This requires understanding all portal SQL writes. Estimated additional discovery: 3-5 days.

#### Effort Estimate

| Approach | Estimate | Confidence |
|----------|----------|------------|
| Minimal portal changes | 4-8 days | ⚠️ MEDIUM |
| OR: Portal bypass via API | 8-12 days | ⚠️ MEDIUM |

---

### Phase 2 Summary

| Component | Effort (days) | Confidence |
|-----------|---------------|------------|
| 2.1 Digital-First SKU Path | 32-50 | ⚠️ MEDIUM |
| 2.2 ax3.com-Only Distribution | 6-9 | ✅ HIGH |
| 2.3 New Digital Product API | 22-29 | ✅ HIGH |
| 2.4 Minimal Portal Modifications | 4-12 | ⚠️ MEDIUM |
| **Phase 2 Total** | **64-100 days** | |

**Recommended Team**: 2-3 developers for 8-10 weeks

**Key Deliverables**:
- New Digital Product API (primary interface for digital-first)
- SQL Server as source of truth for digital products
- Digital-first path for Books/Sheets, then Performance/Choral
- ax3.com-only distribution capability

🔴 **OPEN QUESTION OQ-11**: What technology stack is portal.ax3.com built on? (ASP.NET? Version?)

🔴 **OPEN QUESTION OQ-12**: What is the deployment process for portal changes? Who has access?

🔴 **OPEN QUESTION OQ-13**: What is Lee's availability and comfort level with making these changes?

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| Digital-master template | 5-10 days | ❓ LOW |
---

## Proposed Architecture

### Target State Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         TARGET ARCHITECTURE                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                    NEW: DIGITAL PRODUCT PLATFORM                        ││
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐         ││
│  │  │  Orchestration  │  │  Digital Prod   │  │    Dashboard    │         ││
│  │  │    Service      │  │      API        │  │       UI        │         ││
│  │  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘         ││
│  │           │                    │                    │                   ││
│  │           └────────────────────┼────────────────────┘                   ││
│  │                                │                                        ││
│  │                    ┌───────────▼───────────┐                           ││
│  │                    │      SQL SERVER       │                           ││
│  │                    │  ┌─────────────────┐  │                           ││
│  │                    │  │ digital_masters │  │                           ││
│  │                    │  │ dpo_* tables    │  │                           ││
│  │                    │  │ (new schema)    │  │                           ││
│  │                    │  └─────────────────┘  │                           ││
│  │                    └───────────┬───────────┘                           ││
│  └─────────────────────────────────┼───────────────────────────────────────┘│
│                                    │                                        │
│  ADAPTERS TO LEGACY                │                                        │
│  ──────────────────────────────────┼────────────────────────────────────── │
│           │                        │                    │                   │
│           ▼                        ▼                    ▼                   │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐        │
│  │     AS400       │    │  portal.ax3.com │    │     ax3.com     │        │
│  │  (small chgs)   │    │  (min changes)  │    │   (validate)    │        │
│  │                 │    │  OR bypass      │    │                 │        │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘        │
│           │                        │                    │                   │
│           ▼                        │                    │                   │
│  ┌─────────────────┐              │                    │                   │
│  │ Royalty System  │◄─────────────┴────────────────────┘                   │
│  │  (via AS400)    │                                                       │
│  └─────────────────┘                                                       │
│                                                                              │
│  EXTERNAL (UNCHANGED)                                                       │
│  ──────────────────────────────────────────────────────────────────────────│
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐        │
│  │    AWS S3       │    │    Dropbox      │    │     Legato      │        │
│  │  (automated)    │    │  (automated)    │    │  (manual only)  │        │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Architectural Decisions

| Decision | Rationale |
|----------|-----------|
| **SQL Server as hub** | Easy to modify, can add tables/procedures freely |
| **New API layer** | Abstracts legacy complexity, provides modern interface |
| **Wrap portal, don't rewrite** | Portal changes are hard; minimize and bypass where possible |
| **Small AS400 changes** | Required for sales tracking; keep changes minimal |
| **Coexist with legacy** | Don't break existing flows; new path runs parallel |

---

## Technical Dependencies

### Credentials & Access Required

| Resource | Purpose | Phase | Confidence |
|----------|---------|-------|------------|
| SQL Server connection (read/write) | All automation and new tables | 1, 2 | ✅ HIGH |
| AWS S3 credentials | Asset upload | 1 | ✅ HIGH |
| Dropbox API token | Asset upload | 1 | ✅ HIGH |
| FTP credentials (ftp1.ax3.com) | MRID file sync | 1 | ✅ HIGH |
| Report Server access | Filename mappings | 1 | ✅ HIGH |
| AS400 interface documentation | Minimal record creation | 2 | ⚠️ MEDIUM |
| VM/Container hosting | New services | 1, 2 | ✅ HIGH |

### System Modification Summary

| System | Phase 1 Changes | Phase 2 Changes |
|--------|-----------------|-----------------|
| **SQL Server** | Add dpo_* tables, triggers | Add digital_masters schema |
| **New Infrastructure** | Orchestration service, workers | Digital Product API |
| **AS400** | None | Small: new product type code |
| **portal.ax3.com** | None | Minimal: null ref_sku, ax3_only flag |
| **ax3.com** | None | Validate display (may need fixes) |
| **FileMaker** | None | Add DIGITALMASTER to OutputToWeb |

---

## Open Questions

### Reduced Question Set (Given New Constraints)

With SQL Server being easy and new infrastructure available, many previous questions are resolved. Remaining questions:

| ID | Question | Impact | Priority |
|----|----------|--------|----------|
| OQ-1 | FileMaker field spreadsheets access | Template field mapping | ⚠️ MEDIUM — can reverse-engineer from SQL |
| OQ-2 | Minimal AS400 record structure | Digital-first path | 🔴 HIGH — blocks Phase 2 |
| OQ-3 | ax3.com display dependencies | Website validation | ⚠️ MEDIUM — test early in Phase 2 |
| OQ-4 | Portal SQL write patterns | Bypass portal option | ⚠️ MEDIUM — discovery task |
| OQ-5 | MRID file format | FTP sync | ⚠️ LOW — can inspect existing files |

---

## Risk Register

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| **AS400 changes blocked** | Low | High | Fallback: continue dummy SKU for AS400, generate digitally elsewhere |
| **ax3.com breaks on digital-first** | Medium | Medium | Pilot with single Books/Sheets product first |
| **Portal bypass complexity** | Medium | Medium | Accept minimal portal changes as alternative |
| **Legato remains manual** | High | Low | Accepted constraint; optimize all other steps |
| **Performance/Choral coupling** | Medium | Medium | Defer to Phase 2.5; focus on Books/Sheets first |
| **Existing products break** | Low | High | Parallel paths; don't modify existing flows |

---

## Summary: Effort & Timeline

### Phase 1: Quick-Win Automation

| Component | Effort (days) | Confidence |
|-----------|---------------|------------|
| Orchestration Service | 14-19 | ✅ HIGH |
| Template Auto-Generator | 11-17 | ⚠️ MEDIUM |
| Pre-Upload Validator | 7-11 | ✅ HIGH |
| File Renamer Tool | 4-6 | ✅ HIGH |
| Unified Asset Uploader | 6-9 | ✅ HIGH |
| FTP Auto-Sync | 5-8 | ⚠️ MEDIUM |
| **Phase 1 Total** | **47-70 days** | |

**Duration**: 6-8 weeks with 2 developers

### Phase 2: Architecture Evolution

| Component | Effort (days) | Confidence |
|-----------|---------------|------------|
| Digital-First SKU Path | 32-50 | ⚠️ MEDIUM |
| ax3.com-Only Distribution | 6-9 | ✅ HIGH |
| New Digital Product API | 22-29 | ✅ HIGH |
| Minimal Portal Modifications | 4-12 | ⚠️ MEDIUM |
| **Phase 2 Total** | **64-100 days** | |

**Duration**: 8-10 weeks with 2-3 developers

### Combined Program

| Phase | Duration | Team | Investment Range |
|-------|----------|------|------------------|
| Phase 0: Discovery | 2 weeks | 1 consultant | $15-20K |
| Phase 1: Automation | 6-8 weeks | 2 developers | $60-80K |
| Phase 2: Architecture | 8-10 weeks | 2-3 developers | $100-140K |
| **Total Program** | **16-20 weeks** | | **$175-240K** |

---

*End of Technical Specification*
