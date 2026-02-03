# AX3 Digital Transformation: Technical Specification

> **Document Type**: Technical Specification  
> **Scope**: Phase 1 (Quick-Win Automation) & Phase 2 (Architecture Evolution)  
> **Date**: February 3, 2026  
> **Status**: DRAFT — Pending Technical Discovery  

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
   - [1.1 Template Auto-Generator](#11-template-auto-generator)
   - [1.2 Pre-Upload Validator](#12-pre-upload-validator)
   - [1.3 File Renamer Tool](#13-file-renamer-tool)
   - [1.4 Unified Asset Uploader](#14-unified-asset-uploader)
   - [1.5 FTP Auto-Sync](#15-ftp-auto-sync)
2. [Phase 2: Architecture Evolution](#phase-2-architecture-evolution)
   - [2.1 Digital-First SKU Path](#21-digital-first-sku-path)
   - [2.2 ax3.com-Only Distribution Flag](#22-ax3com-only-distribution-flag)
   - [2.3 Portal Modifications](#23-portal-modifications)
3. [Technical Dependencies](#technical-dependencies)
4. [Open Questions Blocking Implementation](#open-questions-blocking-implementation)
5. [Risk Register](#risk-register)

---

## Phase 1: Quick-Win Automation

**Objective**: Reduce manual effort in current workflow by ~60% without requiring changes to core systems (AS400, SQL Server schema, portal.ax3.com).

**Approach**: Standalone tools that integrate with existing data sources and outputs.

**Estimated Duration**: 4-6 weeks

---

### 1.1 Template Auto-Generator

**Purpose**: Eliminate manual data entry into Excel templates by auto-populating from source systems.

#### Functional Requirements

| Requirement | Description | Confidence |
|-------------|-------------|------------|
| FR-1.1.1 | Generate Performance_Template.xlsx from FileMaker/SQL data | ⚠️ MEDIUM |
| FR-1.1.2 | Generate Sheets_Template.xls from FileMaker/SQL data | ⚠️ MEDIUM |
| FR-1.1.3 | Generate Choral_Template.xls from FileMaker/SQL data | ⚠️ MEDIUM |
| FR-1.1.4 | Support batch generation (multiple SKUs per template) | ✅ HIGH |
| FR-1.1.5 | Handle _CC (Condensed Score) special cases automatically | ✅ HIGH |
| FR-1.1.6 | Ensure UPC consistency across all parts of a set | ✅ HIGH |

#### Data Sources

| Source | Data Retrieved | Access Method | Confidence |
|--------|----------------|---------------|------------|
| SQL Server (tblapproducts) | Product metadata for Performance/Choral | SQL query | ⚠️ MEDIUM — table structure needs confirmation |
| FileMaker | Product metadata for Books/Sheets | ❓ LOW — access method unclear |
| WebCRD | Performance Music metadata | ❓ LOW — API or export? |
| SQL Server ("pending" table) | Pre-publication data | ❓ LOW — table name/structure unknown |

#### Technical Design

```
┌──────────────────────────────────────────────────────────────────┐
│                    TEMPLATE AUTO-GENERATOR                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  INPUT                                                            │
│  ├── SKU list (manual input or file)                             │
│  ├── Product type selection (Performance/Sheets/Choral)          │
│  └── Template version selection                                   │
│                                                                   │
│  PROCESSING                                                       │
│  ├── Query SQL Server for matching records                       │
│  ├── For Performance: also query WebCRD (if accessible)          │
│  ├── Apply _CC handling rules                                    │
│  ├── Validate UPC consistency                                     │
│  ├── Map fields to template columns                              │
│  └── Exclude column AK (known import failure cause)              │
│                                                                   │
│  OUTPUT                                                           │
│  └── Completed Excel template ready for portal upload            │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

#### Field Mapping (Partial — Requires FM Field Spreadsheets)

| Template Column | Source Field | Source System | Confidence |
|-----------------|--------------|---------------|------------|
| Title | Webclean Title | FileMaker/SQL | ⚠️ MEDIUM |
| Composer | Comp arr trans artist | ContributorRole table | ⚠️ MEDIUM |
| UPC | UPC | AS400 or FileMaker | ❓ LOW |
| Page Count | (unknown) | TIFF Counter or manual | ❓ LOW |
| Sort Order | (rule-based) | Generated | ✅ HIGH |

🔴 **OPEN QUESTION OQ-1**: Need access to FileMaker field spreadsheets ("FM Product Fields (working)" and "Values for DigitalVSPhysical from PRODUCT") to complete field mapping.

🔴 **OPEN QUESTION OQ-2**: How is WebCRD accessed? API, database connection, or manual export?

#### Technology Stack (Proposed)

| Component | Technology | Rationale |
|-----------|------------|-----------|
| Runtime | Python 3.11+ or .NET 6+ | Cross-platform, easy deployment |
| Excel Generation | openpyxl (Python) or EPPlus (.NET) | Native Excel format support |
| SQL Connection | pyodbc / System.Data.SqlClient | Standard SQL Server access |
| UI | CLI first, optional desktop GUI later | Fastest to deliver |

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| SQL Server integration | 2-3 days | ⚠️ MEDIUM |
| FileMaker data access | 3-5 days | ❓ LOW — depends on access method |
| Template generation logic | 3-4 days | ✅ HIGH |
| Field mapping implementation | 2-5 days | ❓ LOW — depends on OQ-1 |
| Testing & validation | 2-3 days | ✅ HIGH |
| **Total** | **12-20 days** | |

---

### 1.2 Pre-Upload Validator

**Purpose**: Catch data errors before portal upload, reducing failed imports and rework.

#### Functional Requirements

| Requirement | Description | Confidence |
|-------------|-------------|------------|
| FR-1.2.1 | Validate completed Excel template before upload | ✅ HIGH |
| FR-1.2.2 | Check UPC consistency across set parts | ✅ HIGH |
| FR-1.2.3 | Verify _CC sort order = 1 and page count = 1 | ✅ HIGH |
| FR-1.2.4 | Validate required fields are populated | ⚠️ MEDIUM |
| FR-1.2.5 | Check for column AK presence (flag for removal) | ✅ HIGH |
| FR-1.2.6 | Verify ref_sku exists in SQL Server | ⚠️ MEDIUM |
| FR-1.2.7 | Cross-check against AS400 product status | ❓ LOW |

#### Validation Rules (Known)

| Rule | Description | Source | Confidence |
|------|-------------|--------|------------|
| VR-01 | All parts in a set must share the same UPC | Meeting notes | ✅ HIGH |
| VR-02 | _CC parts must have sort order "1" | Meeting notes | ✅ HIGH |
| VR-03 | _CC parts must have page count "1" | Meeting notes | ✅ HIGH |
| VR-04 | Column AK causes import failure if present | Q&A | ✅ HIGH |
| VR-05 | ref_sku must exist in tblapproducts or pending table | Inferred | ⚠️ MEDIUM |
| VR-06 | AS400 Status must be CUR, TOP, NEW, or PND | Q&A | ⚠️ MEDIUM |
| VR-07 | OutputToWeb must be JJ114 or DIGIONLY | Q&A | ⚠️ MEDIUM |

🔴 **OPEN QUESTION OQ-3**: What are the complete validation rules used by portal.ax3.com? Need access to validation logic documentation or code.

#### Technical Design

```
┌──────────────────────────────────────────────────────────────────┐
│                     PRE-UPLOAD VALIDATOR                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  INPUT                                                            │
│  └── Completed Excel template file                               │
│                                                                   │
│  VALIDATION LAYERS                                                │
│  ├── Layer 1: Structure validation                               │
│  │   ├── Required columns present                                │
│  │   ├── Column AK check                                         │
│  │   └── Data type validation                                    │
│  │                                                                │
│  ├── Layer 2: Business rule validation                           │
│  │   ├── UPC consistency                                         │
│  │   ├── _CC handling rules                                      │
│  │   └── Sort order logic                                        │
│  │                                                                │
│  └── Layer 3: Cross-system validation                            │
│      ├── ref_sku exists in SQL                                   │
│      ├── AS400 status check                                      │
│      └── OutputToWeb flag check                                  │
│                                                                   │
│  OUTPUT                                                           │
│  ├── Validation report (pass/fail per rule)                      │
│  ├── Error details with row/column references                    │
│  └── Suggested fixes where determinable                          │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| Template parsing | 1-2 days | ✅ HIGH |
| Structure validation | 1-2 days | ✅ HIGH |
| Business rule validation | 2-3 days | ✅ HIGH |
| Cross-system validation | 2-4 days | ⚠️ MEDIUM |
| Reporting/output | 1-2 days | ✅ HIGH |
| **Total** | **7-13 days** | |

---

### 1.3 File Renamer Tool

**Purpose**: Automate PDF file renaming to match digital SKU naming conventions.

#### Functional Requirements

| Requirement | Description | Confidence |
|-------------|-------------|------------|
| FR-1.3.1 | Download filename mapping from report server | ✅ HIGH |
| FR-1.3.2 | Batch rename PDFs according to mapping | ✅ HIGH |
| FR-1.3.3 | Validate source files exist before rename | ✅ HIGH |
| FR-1.3.4 | Generate rename log for audit trail | ✅ HIGH |
| FR-1.3.5 | Handle conflicts (duplicate target names) | ✅ HIGH |

#### Data Source

| Source | URL Pattern | Confidence |
|--------|-------------|------------|
| Report Server | `alfredawssql06.alfredpub.com/.../Digital%20Uploads%20-%20Rename%20Files` | ✅ HIGH — confirmed in workflow |

#### Technical Design

```
┌──────────────────────────────────────────────────────────────────┐
│                      FILE RENAMER TOOL                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  INPUT                                                            │
│  ├── Source folder containing PDFs (original names)             │
│  ├── Report server URL or downloaded mapping file                │
│  └── Destination folder for renamed files                        │
│                                                                   │
│  PROCESSING                                                       │
│  ├── Fetch/parse filename mapping                                │
│  ├── Match source files to mapping entries                       │
│  ├── Validate all source files present                           │
│  ├── Check for target name conflicts                             │
│  └── Execute rename (copy to new location with new name)         │
│                                                                   │
│  OUTPUT                                                           │
│  ├── Renamed files in destination folder                         │
│  ├── Rename log (old name → new name)                            │
│  └── Error report (missing files, conflicts)                     │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| Report server integration | 1-2 days | ✅ HIGH |
| File matching logic | 1-2 days | ✅ HIGH |
| Rename execution | 1 day | ✅ HIGH |
| Logging/error handling | 1 day | ✅ HIGH |
| **Total** | **4-6 days** | |

---

### 1.4 Unified Asset Uploader

**Purpose**: Single action to upload PDFs to multiple destinations (currently done 3 times manually).

#### Functional Requirements

| Requirement | Description | Confidence |
|-------------|-------------|------------|
| FR-1.4.1 | Upload PDFs to AWS S3 (alfred-dsm-pdfs bucket) | ✅ HIGH |
| FR-1.4.2 | Upload PDFs to Dropbox archive | ✅ HIGH |
| FR-1.4.3 | Upload cover images to AWS S3 (alfred-catfiles bucket) | ✅ HIGH |
| FR-1.4.4 | Track upload status per destination | ✅ HIGH |
| FR-1.4.5 | Resume failed uploads | ✅ HIGH |
| FR-1.4.6 | ~~Upload to Legato~~ | ❌ NOT FEASIBLE — no API |

#### Destinations

| Destination | Protocol | Credentials Needed | Confidence |
|-------------|----------|-------------------|------------|
| AWS S3 (alfred-dsm-pdfs) | AWS SDK / S3 API | AWS access keys | ✅ HIGH |
| AWS S3 (alfred-catfiles) | AWS SDK / S3 API | AWS access keys | ✅ HIGH |
| Dropbox | Dropbox API | OAuth token | ✅ HIGH |
| Legato | Web portal only | N/A — no automation | ✅ HIGH (confirmed no API) |

⚠️ **LIMITATION**: Legato upload cannot be automated. User must still manually upload to `app.legatomedia.com/publisher/assets/uploadWizard/`. This reduces the 3x manual upload to 1x manual + 1x automated.

#### Technical Design

```
┌──────────────────────────────────────────────────────────────────┐
│                    UNIFIED ASSET UPLOADER                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  INPUT                                                            │
│  ├── Folder containing renamed PDFs                              │
│  ├── Folder containing cover images                              │
│  ├── Destination selection (AWS, Dropbox, or both)               │
│  └── Credentials (from config file or environment)               │
│                                                                   │
│  PROCESSING                                                       │
│  ├── Validate files exist and meet requirements                  │
│  │   ├── PDF format                                              │
│  │   ├── No cover pages                                          │
│  │   ├── Fonts outlined (cannot validate, trust source)          │
│  │   └── No trailing blank pages (optional validation)           │
│  │                                                                │
│  ├── Upload to AWS S3                                            │
│  │   ├── PDFs → alfred-dsm-pdfs bucket                          │
│  │   └── Covers → alfred-catfiles bucket                        │
│  │                                                                │
│  └── Upload to Dropbox                                           │
│      └── PDFs → configured archive folder                        │
│                                                                   │
│  OUTPUT                                                           │
│  ├── Upload status report                                        │
│  ├── URLs for uploaded files                                     │
│  └── Failed upload list for retry                                │
│                                                                   │
│  ⚠️ MANUAL STEP REMAINS                                          │
│  └── User must upload to Legato separately                       │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| AWS S3 integration | 2-3 days | ✅ HIGH |
| Dropbox integration | 2-3 days | ✅ HIGH |
| File validation | 1-2 days | ✅ HIGH |
| Status tracking/retry | 1-2 days | ✅ HIGH |
| **Total** | **6-10 days** | |

---

### 1.5 FTP Auto-Sync

**Purpose**: Automatically push MRID files to dealer FTP after portal update.

#### Functional Requirements

| Requirement | Description | Confidence |
|-------------|-------------|------------|
| FR-1.5.1 | Detect new MRID data in SQL Server or report | ⚠️ MEDIUM |
| FR-1.5.2 | Generate MR_Upload_Dealer_Full.txt | ⚠️ MEDIUM |
| FR-1.5.3 | Generate MR_Upload_Dealer.txt (delta) | ⚠️ MEDIUM |
| FR-1.5.4 | Upload to ftp1.ax3.com/Dealerftp | ✅ HIGH |
| FR-1.5.5 | Log upload timestamp and file details | ✅ HIGH |

#### Data Flow

```
┌────────────────┐     ┌────────────────┐     ┌────────────────┐
│  Portal MRID   │────▶│   SQL Server   │────▶│  FTP Auto-Sync │
│    Upload      │     │   (MRIDs)      │     │     Tool       │
└────────────────┘     └────────────────┘     └───────┬────────┘
                                                      │
                                                      ▼
                                              ┌────────────────┐
                                              │ ftp1.ax3.com   │
                                              │   /Dealerftp   │
                                              └───────┬────────┘
                                                      │
                                    ┌─────────────────┼─────────────────┐
                                    ▼                 ▼                 ▼
                              ┌──────────┐     ┌──────────┐     ┌──────────┐
                              │ JW Pepper│     │Stanton's │     │  Other   │
                              └──────────┘     └──────────┘     └──────────┘
```

🔴 **OPEN QUESTION OQ-4**: What triggers the FTP upload currently? Is there a signal when MRID upload is complete, or is it time-based?

🔴 **OPEN QUESTION OQ-5**: What is the exact format of MR_Upload_Dealer_Full.txt and MR_Upload_Dealer.txt?

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| Trigger mechanism | 1-3 days | ❓ LOW |
| File generation | 2-3 days | ⚠️ MEDIUM |
| FTP upload | 1-2 days | ✅ HIGH |
| Logging | 1 day | ✅ HIGH |
| **Total** | **5-9 days** | |

---

### Phase 1 Summary

| Component | Effort (days) | Confidence | Dependencies |
|-----------|---------------|------------|--------------|
| 1.1 Template Auto-Generator | 12-20 | ⚠️ MEDIUM | OQ-1, OQ-2 |
| 1.2 Pre-Upload Validator | 7-13 | ⚠️ MEDIUM | OQ-3 |
| 1.3 File Renamer Tool | 4-6 | ✅ HIGH | None |
| 1.4 Unified Asset Uploader | 6-10 | ✅ HIGH | AWS/Dropbox credentials |
| 1.5 FTP Auto-Sync | 5-9 | ⚠️ MEDIUM | OQ-4, OQ-5 |
| **Phase 1 Total** | **34-58 days** | | |

**Recommended Team**: 2 developers for 4-6 weeks

---

## Phase 2: Architecture Evolution

**Objective**: Enable digital products as first-class entities, not derived from physical products.

**Approach**: Modify core systems (SQL Server, portal, potentially AS400 interface) to support a new "digital-master" product type.

**Estimated Duration**: 8-12 weeks

**⚠️ IMPORTANT**: Phase 2 estimates have significant uncertainty. Technical discovery (Phase 0) is required before committing to detailed plans.

---

### 2.1 Digital-First SKU Path

**Purpose**: Allow creation of digital products without requiring a dummy physical SKU.

#### Functional Requirements

| Requirement | Description | Confidence |
|-------------|-------------|------------|
| FR-2.1.1 | New product type: DIGITALMASTER (or similar) | ⚠️ MEDIUM |
| FR-2.1.2 | SQL Server generates digital SKU without ref_sku dependency | ❓ LOW |
| FR-2.1.3 | AS400 accepts minimal "digital-native" record | ❓ LOW |
| FR-2.1.4 | Royalty system accepts digital-only rights configuration | ⚠️ MEDIUM |
| FR-2.1.5 | ax3.com displays digital-master products correctly | ❓ LOW |
| FR-2.1.6 | Existing ~500 dummy SKU products continue working | ✅ HIGH |

#### Proposed Data Model Changes

```
CURRENT STATE:
═══════════════════════════════════════════════════════════════════

[Physical/Dummy SKU] ─────────┐
        │                     │
        ▼                     ▼
   [FileMaker]           [AS400]
        │                     │
        ▼                     ▼
   [SQL Server] ◄─────── (nightly)
        │
        ▼
[Digital SKU] (generated)
        │
        ▼
   [ax3.com]


PROPOSED STATE (Books/Sheets First):
═══════════════════════════════════════════════════════════════════

Option A: Digital-Master Record
───────────────────────────────
[Digital-Master SKU] ─────────┐
        │                     │
        ▼                     ▼
   [FileMaker]           [AS400]
   OutputToWeb=           (minimal
   DIGITALMASTER          record)
        │                     │
        ▼                     ▼
   [SQL Server] ◄─────── (nightly)
        │
        ▼
[Digital SKU] (generated, no ref_sku OR ref_sku = self)
        │
        ▼
   [ax3.com]


Option B: Direct Digital Creation (No Parent)
───────────────────────────────────────────────
[Digital SKU Created Directly in SQL]
        │
        ├────────▶ [AS400] (digital-specific record type)
        │
        ├────────▶ [Royalty System] (direct integration TBD)
        │
        └────────▶ [ax3.com]
```

🔴 **OPEN QUESTION OQ-6**: Which option is more feasible? Option A requires less change to current architecture. Option B is cleaner but requires more system modifications.

🔴 **OPEN QUESTION OQ-7**: What would a minimal AS400 record look like for a true digital-first product? Which of the 34+ fields can be null/defaulted?

🔴 **OPEN QUESTION OQ-8**: What does ax3.com expect from a product record? Does it require a parent physical SKU for display logic, pricing, or any other function?

#### Implementation Phases (Proposed)

| Sub-Phase | Scope | Risk Level |
|-----------|-------|------------|
| 2.1.1 | Books/Sheets digital-master (lowest coupling) | ⚠️ MEDIUM |
| 2.1.2 | Performance digital-master (higher coupling) | 🔴 HIGH |
| 2.1.3 | Choral digital-master (highest coupling) | 🔴 HIGH |

#### System Changes Required

| System | Change | Owner | Confidence |
|--------|--------|-------|------------|
| FileMaker | Add DIGITALMASTER value to OutputToWeb | Editorial Team | ✅ HIGH |
| SQL Server | Modify SKU generation to accept null/self ref_sku | Lee / Developer | ⚠️ MEDIUM |
| SQL Server | Add digital-master product table or flag | Lee / Developer | ⚠️ MEDIUM |
| portal.ax3.com | New template type for digital-master | Lee | ❓ LOW |
| portal.ax3.com | Modified validation for digital-master | Lee | ❓ LOW |
| AS400 | Accept digital-native record structure | Purchasing Team | ❓ LOW |
| ax3.com | Display logic for products without physical parent | Jon | ❓ LOW |
| Royalty System | Accept digital-only rights without physical parent | Royalty Admin | ⚠️ MEDIUM |

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| FileMaker changes | 1-2 days | ✅ HIGH |
| SQL Server schema changes | 5-10 days | ❓ LOW |
| Portal template changes | 5-10 days | ❓ LOW |
| Portal validation changes | 3-7 days | ❓ LOW |
| AS400 interface changes | 3-10 days | ❓ LOW |
| ax3.com display logic | 5-15 days | ❓ LOW |
| Testing & validation | 10-15 days | ⚠️ MEDIUM |
| **Total** | **32-69 days** | |

---

### 2.2 ax3.com-Only Distribution Flag

**Purpose**: Allow products to be sold only on ax3.com, not distributed to Legato/dealers.

#### Functional Requirements

| Requirement | Description | Confidence |
|-------------|-------------|------------|
| FR-2.2.1 | New field/flag to mark products as ax3.com-only | ⚠️ MEDIUM |
| FR-2.2.2 | Legato export excludes ax3.com-only products | ⚠️ MEDIUM |
| FR-2.2.3 | MRID files exclude ax3.com-only products | ⚠️ MEDIUM |
| FR-2.2.4 | ax3.com displays ax3.com-only products normally | ✅ HIGH |
| FR-2.2.5 | Reporting distinguishes ax3.com-only vs. full distribution | ⚠️ MEDIUM |

#### Data Model

```
PROPOSED: ax3OnlyFlag in SQL Server
═══════════════════════════════════════════════════════════════════

Digital Product Record
├── DigitalSKU: 00-PB-0012345
├── ref_sku: 47888 (or null for digital-master)
├── ax3OnlyFlag: TRUE/FALSE  ◄─── NEW FIELD
├── MRID: (null if ax3OnlyFlag = TRUE)
└── ... other fields

Export Logic Changes:
├── Legato Export: WHERE ax3OnlyFlag = FALSE
├── MRID Files: WHERE ax3OnlyFlag = FALSE
└── ax3.com Feed: No filter (all products)
```

🔴 **OPEN QUESTION OQ-9**: Where is the Legato export query defined? Is it in portal.ax3.com, SQL stored procedures, or SSRS reports?

🔴 **OPEN QUESTION OQ-10**: Are there downstream systems (reporting, analytics) that assume all digital products go to Legato?

#### System Changes Required

| System | Change | Owner | Confidence |
|--------|--------|-------|------------|
| SQL Server | Add ax3OnlyFlag column | Lee / Developer | ✅ HIGH |
| portal.ax3.com | Add ax3OnlyFlag to template/UI | Lee | ⚠️ MEDIUM |
| Legato Export | Modify query to filter by flag | Lee / Report owner | ⚠️ MEDIUM |
| MRID Generation | Modify query to filter by flag | Lee / Report owner | ⚠️ MEDIUM |
| Reporting | Update to handle new product category | Report owner | ⚠️ MEDIUM |

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| SQL schema change | 1 day | ✅ HIGH |
| Portal UI change | 2-4 days | ⚠️ MEDIUM |
| Export query changes | 2-4 days | ⚠️ MEDIUM |
| Reporting updates | 2-5 days | ⚠️ MEDIUM |
| Testing | 3-5 days | ✅ HIGH |
| **Total** | **10-19 days** | |

---

### 2.3 Portal Modifications

**Purpose**: Update portal.ax3.com to support new workflows.

#### Scope

| Modification | Description | Confidence |
|--------------|-------------|------------|
| MOD-2.3.1 | New template type for digital-master products | ❓ LOW |
| MOD-2.3.2 | Modified validation for digital-master (no ref_sku requirement) | ❓ LOW |
| MOD-2.3.3 | ax3OnlyFlag field in upload templates | ⚠️ MEDIUM |
| MOD-2.3.4 | Improved error messaging for validation failures | ⚠️ MEDIUM |
| MOD-2.3.5 | Bulk operations for territory/metadata changes | ⚠️ MEDIUM |

🔴 **OPEN QUESTION OQ-11**: What technology stack is portal.ax3.com built on? (ASP.NET? Version?)

🔴 **OPEN QUESTION OQ-12**: What is the deployment process for portal changes? Who has access?

🔴 **OPEN QUESTION OQ-13**: What is Lee's availability and comfort level with making these changes?

#### Effort Estimate

| Task | Estimate | Confidence |
|------|----------|------------|
| Digital-master template | 5-10 days | ❓ LOW |
| Validation modifications | 5-10 days | ❓ LOW |
| ax3OnlyFlag integration | 2-4 days | ⚠️ MEDIUM |
| Error messaging improvements | 3-5 days | ⚠️ MEDIUM |
| Bulk operations | 5-10 days | ⚠️ MEDIUM |
| Testing | 5-10 days | ⚠️ MEDIUM |
| **Total** | **25-49 days** | |

---

### Phase 2 Summary

| Component | Effort (days) | Confidence | Key Blockers |
|-----------|---------------|------------|--------------|
| 2.1 Digital-First SKU Path | 32-69 | ❓ LOW | OQ-6, OQ-7, OQ-8 |
| 2.2 ax3.com-Only Distribution | 10-19 | ⚠️ MEDIUM | OQ-9, OQ-10 |
| 2.3 Portal Modifications | 25-49 | ❓ LOW | OQ-11, OQ-12, OQ-13 |
| **Phase 2 Total** | **67-137 days** | | |

**Recommended Team**: 2-3 developers for 8-12 weeks

---

## Technical Dependencies

### Credentials & Access Required

| Resource | Purpose | Phase |
|----------|---------|-------|
| SQL Server connection | Read product data, potentially write | 1, 2 |
| AWS S3 credentials | Upload PDFs and cover images | 1 |
| Dropbox API token | Upload to archive | 1 |
| FTP credentials (ftp1.ax3.com) | Upload MRID files | 1 |
| Report Server access | Download filename mappings, MRID reports | 1 |
| FileMaker access | Read product metadata (method TBD) | 1 |
| portal.ax3.com code access | Modify for Phase 2 | 2 |
| AS400 interface documentation | Understand record requirements | 2 |

### System Interfaces

| Interface | Current State | Automation Potential |
|-----------|---------------|---------------------|
| SQL Server | Direct query | ✅ Full automation |
| AWS S3 | SDK access | ✅ Full automation |
| Dropbox | API access | ✅ Full automation |
| FTP | Standard FTP | ✅ Full automation |
| Report Server | HTTP/SSRS | ✅ Full automation |
| FileMaker | Unknown | ⚠️ Depends on access method |
| Legato | Web portal only | ❌ No automation (no API) |
| AS400 | Unknown interface | ❓ Needs discovery |
| portal.ax3.com | Code modification | ⚠️ Depends on access |

---

## Open Questions Blocking Implementation

### Critical (Blocks Phase 1)

| ID | Question | Impact | Who Can Answer |
|----|----------|--------|----------------|
| OQ-1 | Access to FileMaker field spreadsheets | Template field mapping | Editorial Team |
| OQ-2 | WebCRD access method (API, DB, export) | Performance template automation | IT / System Admin |
| OQ-3 | Portal validation logic documentation | Pre-upload validator completeness | Lee |

### Important (Blocks Phase 2)

| ID | Question | Impact | Who Can Answer |
|----|----------|--------|----------------|
| OQ-6 | Digital-master architecture approach | Core design decision | Technical discovery session |
| OQ-7 | Minimal AS400 record for digital-first | AS400 integration feasibility | Purchasing Team |
| OQ-8 | ax3.com display requirements | Website changes needed | Jon |
| OQ-11 | Portal technology stack | Development approach | Lee |
| OQ-12 | Portal deployment process | Change management | Lee / IT |
| OQ-13 | Lee's availability/comfort | Capacity planning | Lee / Management |

### Informational (Refines Estimates)

| ID | Question | Impact | Who Can Answer |
|----|----------|--------|----------------|
| OQ-4 | FTP upload trigger mechanism | Auto-sync design | Travis / Digital Team |
| OQ-5 | MRID file format specification | File generation logic | Travis / Digital Team |
| OQ-9 | Legato export query location | ax3-only flag implementation | Lee / Report owner |
| OQ-10 | Downstream reporting dependencies | Scope of reporting changes | Business Analyst |

---

## Risk Register

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| **Lee unavailable or uncomfortable with portal changes** | Medium | High | Identify backup developer; plan for longer timeline or external support |
| **AS400 cannot accept minimal digital records** | Medium | High | Accept that some AS400 entry will remain manual; focus on reducing other manual steps |
| **ax3.com requires significant changes for digital-master** | Medium | High | Start with Books/Sheets only; postpone Performance/Choral until website updated |
| **FileMaker access proves difficult** | Medium | Medium | Fall back to CSV export process; automate from nightly SQL data only |
| **Legato never provides API** | High | Medium | Accept manual Legato upload as permanent constraint; optimize other steps |
| **Existing 500 dummy SKU products break** | Low | High | Extensive regression testing; maintain backward compatibility |
| **Scope creep from additional product types** | Medium | Medium | Strictly phase rollout; Books/Sheets → Performance → Choral |
| **Column AK issue recurs in new templates** | Low | Low | Build permanent fix into template generator |

---

## Appendix A: Recommended Phase 0 Discovery Activities

Before committing to Phase 1-2 implementation, recommend 2-3 week discovery:

| Activity | Duration | Participants | Output |
|----------|----------|--------------|--------|
| FileMaker field documentation review | 2-3 hours | Editorial Team, Consultant | Complete field mapping |
| Portal code review with Lee | 4-8 hours | Lee, Developer | Tech stack, validation logic, change process |
| AS400 record structure workshop | 2-4 hours | Purchasing Team, Consultant | Minimal digital record spec |
| ax3.com architecture review with Jon | 2-4 hours | Jon, Developer | Website dependencies |
| Current workflow shadowing with Travis | 4-8 hours | Travis, Consultant | Detailed process documentation |
| Report Server inventory | 2-4 hours | IT, Consultant | Available data sources |

**Discovery Deliverable**: Updated technical specification with HIGH confidence estimates.

---

*End of Technical Specification*
