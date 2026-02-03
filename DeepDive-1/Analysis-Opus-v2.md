# AX3 Music Digital Transformation: As-Is Analysis (v2)

> **Document Purpose**: Comprehensive analysis of AX3's current operational workflow, assumptions, and open questions to support the digital transformation initiative.
>
> **Date**: February 3, 2026  
> **Based on**: Deep-dive meeting (Feb 2, 2026), Q&A responses, and pre-existing knowledge base

---

## Table of Contents

1. [Current Assumptions](#1-current-assumptions)
2. [Current Workflow (As-Is)](#2-current-workflow-as-is)
3. [Open Questions for AX3](#3-open-questions-for-ax3)
4. [Glossary](#4-glossary)

---

## 1. Current Assumptions

The following assumptions represent our current understanding of AX3's systems and processes. Each should be validated with AX3 stakeholders.

### 1.1 Architectural Assumptions

| # | Assumption | Rationale | Confidence |
|---|------------|-----------|------------|
| A1 | **A physical or dummy SKU is required before any digital product can be created** | Multiple sources confirm this constraint. The Q&A explicitly states: "The performance and choral processes are dependent upon having a physical sku or a dummy sku because portal.ax3.com is pulling the metadata." | Confirmed |
| A2 | **FileMaker is the source of truth for product metadata, but does not write to AS400** | Q&A clarifies: "FileMaker system doesn't write to the AS400. AS400 and FileMaker are not tied together in any way other than FileMaker reads data from the AS400." | Confirmed |
| A3 | **SQL Server generates digital SKUs using a serialized numbering system independent of the physical SKU's numeric portion** | Q&A confirms: "The rest of the numbers are serialized and not tied to the ref_sku." The prefix (PO, PK, PC, etc.) determines product type, but the numeric portion is sequential. | Confirmed |
| A4 | **AS400 is required for all sales transactions, including digital-only products** | Q&A states: "There would be nowhere to report the sale. An AS400 record is required currently for any product that sells, digital or physical." | Confirmed |
| A5 | **The Royalty System is updated from AS400 overnight, not directly from portal or SQL Server** | Q&A flow: "Portal.ax3.com writes to the SQL Server, and the AS400. The AS400 writes to the Royalty system overnight." | Confirmed |
| A6 | **FileMaker data is downloaded nightly to SQL Server via CSV** | Meeting notes and Q&A confirm a nightly export process that populates SQL tables (tblapproducts and a "pending" table). | Confirmed |
| A7 | **Legato has no API; all integration is via web portal upload/download** | Q&A confirms: "Legato (app.legatomedia.com) does not have an API to tie into our systems as of today." | Confirmed |
| A8 | **Choral and Performance products require metadata from the FileMaker nightly download; Books/Sheets can have metadata entered fresh** | Q&A states: "Only books and sheets require their data to be entered fresh. Choral/Performance music picks up data from the nightly data downloads into SQL from FileMaker." | Confirmed |

### 1.2 Process Assumptions

| # | Assumption | Rationale | Confidence |
|---|------------|-----------|------------|
| P1 | **The Dummy SKU requirement originated from royalty programming constraints** | Q&A states: "A dummy sku was created so that royalties could be programmed as a physical product." | Confirmed |
| P2 | **Bypassing Legato is technically possible but not operationally supported today** | Q&A confirms: "Today, the Legato process IS required for digital-only products and can't easily be skipped." However, workarounds exist for ER products. | Likely |
| P3 | **The portal was built with physical-first assumptions hard-coded** | Q&A: "The portal was built off of assumptions for physical though." | Confirmed |
| P4 | **Validation in FileMaker and portal is weak; errors propagate silently** | Q&A: "The system doesn't have good validation and doesn't have the logic to 'check'. If we skip certain fields, then the data... is blank." | Confirmed |
| P5 | **Digital-only products for Books/Sheets can technically bypass the dummy SKU, but ax3.com website logic may fail** | Q&A: "We can currently upload without a dummy sku for books/sheets, but the website logic, I believe, will have some issues." | Likely |
| P6 | **~500 digital-only products with dummy SKUs exist today; migration to a new model is not required for these** | Q&A confirms the count and states: "I believe these could stay as-is while new products use the new model." | Confirmed |
| P7 | **Dummy SKUs are only referenced internally, never in contracts or customer-facing materials** | Q&A: "No, dummy sku's are only referenced internally." | Confirmed |
| P8 | **Digital royalties are programmed separately from physical royalties; there is no dependency** | Q&A: "The digital sku is programmed separately, it is not related to the physical sku." | Confirmed |

### 1.3 Organizational Assumptions

| # | Assumption | Rationale | Confidence |
|---|------------|-----------|------------|
| O1 | **Lee (internal staff) maintains the portal code; he may have limited familiarity with it** | Q&A: "I guess now it is Lee (internal staff) who manages this... I'm not sure how comfortable Lee is with making changes to the system." | Likely |
| O2 | **Travis (X team) performs manual data validation before products go live** | Meeting notes mention Travis checking data and fixing issues. | Likely |
| O3 | **The original developers (Doug Fraser, Chris Rubeiz) are no longer actively involved** | Q&A indicates Chris was "reluctant to make changes" and is no longer the primary maintainer. | Likely |
| O4 | **The Purchasing team creates AS400 records; the Editorial team adds FileMaker data** | Meeting notes describe this division of labor as a prerequisite to digitization. | Likely |

### 1.4 Distribution Assumptions

| # | Assumption | Rationale | Confidence |
|---|------------|-----------|------------|
| D1 | **Legato only cares about the digital SKU; removing the dummy SKU would not break Legato integration** | Q&A: "Nothing would break. Legato only cares about the digital sku. The dummy sku is something internal to AX3." | Confirmed |
| D2 | **ER products (ZIP, interactive PDF) cannot be distributed through Legato** | Knowledge base and Q&A confirm Legato cannot handle these formats. | Confirmed |
| D3 | **Selling only on ax3.com (not Legato) is a business decision, not a technical limitation** | Q&A: "This is a business decision not to go through Legato. Yes a digital-only product can go through Legato." | Confirmed |
| D4 | **No financial reports, royalty calculations, or analytics require a parent physical SKU for digital products** | Q&A: "I don't believe there is any need for every SKU to have a parent physical SKU." | Likely |

---

## 2. Current Workflow (As-Is)

### 2.1 System Landscape Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           AX3 SYSTEM LANDSCAPE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌────────────────┐                              ┌────────────────┐        │
│  │  Purchasing    │──creates AS400 record──────▶│     AS400      │        │
│  │    Team        │                              │  (Inventory/   │        │
│  └────────────────┘                              │    Sales)      │        │
│                                                  └───────┬────────┘        │
│  ┌────────────────┐                                      │ nightly         │
│  │  Editorial     │──adds metadata──────────────────┐    │ sync            │
│  │    Team        │                                 │    ▼                 │
│  └────────────────┘                                 │ ┌────────────────┐   │
│                                                     │ │ Royalty System │   │
│  ┌────────────────┐                                 │ │    (Rights)    │   │
│  │   FileMaker    │◀────────────────────────────────┘ └────────────────┘   │
│  │  (Metadata)    │                                                        │
│  └───────┬────────┘                                                        │
│          │ nightly CSV export                                              │
│          ▼                                                                 │
│  ┌────────────────┐          ┌────────────────┐          ┌──────────────┐ │
│  │   SQL Server   │◀─────────│  portal.ax3    │◀─────────│    Excel     │ │
│  │ (Digital SKUs) │  writes  │     .com       │  upload  │  Templates   │ │
│  └───────┬────────┘          └────────────────┘          └──────────────┘ │
│          │                                                                 │
│          ├──────────────────┬───────────────────┬──────────────────┐      │
│          ▼                  ▼                   ▼                  ▼      │
│    ┌──────────┐      ┌──────────┐        ┌──────────┐       ┌─────────┐  │
│    │  AS400   │      │ Royalty  │        │ ax3.com  │       │ Legato  │  │
│    │(dig SKU) │      │  System  │        │          │       │(Dealers)│  │
│    └──────────┘      └──────────┘        └──────────┘       └────┬────┘  │
│                                                                   │       │
│                                                              ┌────▼────┐  │
│                                                              │  FTP +  │  │
│                                                              │ Dealers │  │
│                                                              └─────────┘  │
│                                                                           │
│  ┌──────────────────────────────────────────────────────────────────┐    │
│  │                      ASSET DISTRIBUTION                           │    │
│  │   Source PDFs  ──▶  AWS S3 (ax3.com)                             │    │
│  │                 ──▶  Legato (dealers)                             │    │
│  │                 ──▶  Dropbox (archive + non-Legato dealers)       │    │
│  └──────────────────────────────────────────────────────────────────┘    │
│                                                                           │
└───────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Product Types and Their Characteristics

| Product Type | Example SKU Types | Data Entry Method | Dummy SKU for Digital-Only? |
|--------------|-------------------|-------------------|----------------------------|
| **Performance Sets** | PK (set), PR (score), PC (parts) | Metadata pulled from FileMaker via nightly download | **Required** |
| **Choral (Octavos)** | PO | Metadata pulled from FileMaker via nightly download | **Required** |
| **Books** | PB | Data entered fresh in template | Technically optional, but ax3.com may have issues |
| **Sheets** | PS | Data entered fresh in template | Technically optional, but ax3.com may have issues |
| **Electronic Resources** | ER | Data entered via ProductAssets table | Depends on workflow |

### 2.3 Detailed Workflow by Path

#### Path A: Physical-to-Digital Conversion

**Trigger**: Decision to create digital version of an existing physical product.

```
PHASE 1: PREREQUISITES (Multiple Teams)
═══════════════════════════════════════════════════════════════════════════

Step 1.1: Purchasing Team creates AS400 record
├── System: AS400
├── Artifact: New product record with sequential SKU
├── Key Fields: Pub Number, Item Number, Price, Royalty=Y, Job Number (###95 format)
└── State: Physical product exists in AS400

Step 1.2: Nightly job copies to Royalty System
├── Trigger: Nightly batch job (owner unclear)
├── Condition: Royalty field = "Y"
├── System: AS400 → Royalty System
└── Artifact: Royalty record created

Step 1.3: Editorial Team adds metadata in FileMaker
├── System: FileMaker
├── Tables Modified: Product, ContributorRole, TOCSongSources (if multi-song)
├── Actor: Editorial Team
└── State: Product metadata populated

Step 1.4: Travis (X Team) validates data
├── Actor: Travis
├── Action: Manual review, fixes issues
├── Trigger: Unclear (how does Travis know Editorial is done?)
└── State: Data validated

Step 1.5: Product/Marketing Manager approval
├── Actor: Product Manager
├── Action: Final check, sets product ready
└── State: Ready for digitization

Step 1.6: Set readiness flags
├── System: FileMaker
│   └── Field: OutputToWeb = "JJ114"
├── System: AS400
│   └── Field: Status = "CUR" (or TOP, NEW)
├── System: Royalty System
│   └── Digital rights configured for physical SKU
└── State: READY FOR DIGITAL CONVERSION


PHASE 2: TEMPLATE PREPARATION (Travis / Digital Team)
═══════════════════════════════════════════════════════════════════════════

Step 2.1: Gather source data
├── For Performance Music: Download from WebCRD
├── For Sheets/Books: Batch search in FileMaker
├── For Choral: Gather from source PDFs + TIFF Counter
└── Artifacts: SKU list, titles, composers, page counts, UPCs

Step 2.2: Select appropriate template
├── Performance: Performance_Template.xlsx
├── Sheets/Books: Sheets_Template.xls
├── Choral: Choral_Template.xls
└── Location: portal.ax3.com/DigitalTemplates/

Step 2.3: Manually populate template
├── Action: Type data into Excel rows
├── Validation: Manual (UPC consistency, sort order, _CC handling)
├── Known Issues: Column AK can cause import failures
└── Artifact: Completed Excel template


PHASE 3: PORTAL UPLOAD & SKU GENERATION (Travis / Digital Team)
═══════════════════════════════════════════════════════════════════════════

Step 3.1: Upload template to portal
├── System: portal.ax3.com
├── Action: Select template type, upload file
└── Trigger: SQL Server processing

Step 3.2: SQL Server validation and processing
├── System: SQL Server
├── Checks: Data validation against AS400, Royalty, FileMaker (SQL tables)
├── Action: Generate digital SKU (e.g., 00-PK-0047888)
│   └── SKU format: 00-[TYPE]-[SERIAL] (serial is independent of physical SKU)
└── Artifact: Digital SKU created

Step 3.3: Propagation to downstream systems
├── SQL Server → AS400 (digital SKU registered)
├── SQL Server → Royalty System (digital SKU registered)
├── SQL Server → ax3.com (product data available)
└── State: Digital product exists in systems (not yet live)

Step 3.4: Error handling (if import fails)
├── Common Issue: Column AK in export causes failure
├── Workaround: Delete column AK, resave as XLSX
└── Action: Re-upload corrected file


PHASE 4: LEGATO INTEGRATION (Travis / Digital Team)
═══════════════════════════════════════════════════════════════════════════

Step 4.1: Download Legato export file
├── Source: alfredawssql06.alfredpub.com/reports/.../Legato
├── Action: Manual download
└── Artifact: Legato-formatted product data file

Step 4.2: Upload to Legato
├── System: app.legatomedia.com/publisher/vendors/index/
├── Action: Manual upload via web portal
├── Processing: Legato assigns MRID (external system, timing varies)
└── Latency: 5-10 minutes to process, then manual retrieval

Step 4.3: Download MRIDs from report server
├── Source: alfredawssql06.alfredpub.com/.../Dealer%20Digital%20MRID%20Feed
├── Artifacts:
│   ├── Full version (complete MRID list)
│   └── 1-month prior version (delta)
└── Action: Manual download of both versions

Step 4.4: Upload MRIDs to portal
├── System: portal.ax3.com/TSM/Digital/UpdateMRID.aspx
├── Action: Manual upload
└── Effect: MRID stored in SQL Server, linked to digital SKU

Step 4.5: Upload MRIDs to FTP for dealers
├── System: ftp1.ax3.com → Dealerftp folder
├── Files:
│   ├── MR_Upload_Dealer_Full.txt
│   └── MR_Upload_Dealer.txt
└── Effect: Integrated dealers (JW Pepper, Stanton's) can pull data


PHASE 5: ASSET PROCESSING (Travis / Digital Team)
═══════════════════════════════════════════════════════════════════════════

Step 5.1: Gather source PDFs
├── Physical-to-Digital: Files from AWS (print production archive)
├── Digital-Only: Files via Slack from Editorial Team
└── Location: Dropbox archives, AWS

Step 5.2: Prepare PDFs
├── Requirements:
│   ├── Format: PDF only
│   ├── Covers: Remove cover pages
│   ├── Fonts: Must be outlined
│   └── Blank pages: Remove trailing blank pages
└── Action: Manual verification and cleanup

Step 5.3: Rename files to match digital SKUs
├── Source: Report server for filename mapping
├── URL: alfredawssql06.alfredpub.com/.../Digital%20Uploads%20-%20Rename%20Files
└── Action: Manual rename of each PDF

Step 5.4: Upload to multiple destinations
├── Destination 1: Legato (app.legatomedia.com/publisher/assets/uploadWizard/)
│   └── Purpose: Dealer distribution
├── Destination 2: AWS S3 (alfred-dsm-pdfs bucket)
│   └── Purpose: ax3.com customer downloads
├── Destination 3: Dropbox
│   └── Purpose: Archive + non-Legato dealer access
└── ⚠️ Pain Point: Same file uploaded 3 times manually

Step 5.5: Upload cover images
├── Destination: AWS S3 (alfred-catfiles bucket)
└── Purpose: Product display on ax3.com


PHASE 6: GO-LIVE (Automated + Verification)
═══════════════════════════════════════════════════════════════════════════

Step 6.1: Product becomes available
├── ax3.com: Visible after nightly data sync + all assets uploaded
├── Legato dealers: Available after MRID cycle complete
└── State: LIVE

Step 6.2: Verification
├── Check ax3.com product page
├── Verify preview pages display correctly
├── Test download functionality
└── Confirm dealer availability (optional)
```

#### Path B: Digital-Only (with Dummy SKU)

**Trigger**: Decision to create a digital product with no corresponding physical product.

**Key Differences from Path A**:

```
PHASE 0: CREATE DUMMY SKU (Additional Step)
═══════════════════════════════════════════════════════════════════════════

Step 0.1: Create dummy record in AS400
├── System: AS400
├── Key Fields (minimum required):
│   ├── Pub Number
│   ├── Item Number (creates Dummy SKU: PUB-ITEM)
│   ├── Title
│   ├── Territory Restrictions
│   ├── Job Number (must have ###95 equivalent)
│   ├── Royalty = Y
│   ├── Price
│   ├── Product Type Code (B=Set, C=Choral)
│   ├── Company Number
│   └── Pub Date (MM/YY)
└── Note: Full record needed, though some fields differ from physical

Step 0.2: Create dummy record in FileMaker
├── System: FileMaker
├── Tables: Product, ContributorRole, TOCSongSources (if applicable)
├── Key Fields (34+ fields required):
│   ├── Webclean Title, Title for Sorting
│   ├── Comp arr trans artist (requires ContributorRole entries)
│   ├── Webdescription, Shortdescription
│   ├── Product Line, FM Category
│   ├── WebDefinition, WebFormat
│   ├── Publisher Name, Brand
│   ├── Pubnum, Xitemnum
│   ├── Various Web* fields (Instrument, Ensemble, Keywords, etc.)
│   └── NAMM Code
├── Flag: OutputToWeb = "DIGIONLY"
├── Flag: DigitalOnlyYN = "Y"
└── State: Dummy product exists

Step 0.3: Configure royalty rights for dummy SKU
├── System: Royalty System (via AS400 nightly sync)
└── Action: Digital rights attached to dummy SKU

[Then proceed with Phases 2-6 as in Path A]

DIFFERENCES IN SUBSEQUENT PHASES:
═══════════════════════════════════════════════════════════════════════════

Phase 1 (Prerequisites):
├── AS400 Status = "PND" (not CUR)
├── AS400 Price = "USD"
├── AS400 Territories = Value
└── Digital product table in SQL Server: different table (name unclear)

Phase 4 (Legato):
├── Legato integration may be limited or skipped
├── Dealers may access via Dropbox instead
└── Business decision whether to distribute through Legato

Phase 5 (Assets):
├── Source files via Slack from Editorial (not from AWS print archive)
└── File sources are different since no print production exists
```

#### Path C: Electronic Resources (ax3.com Exclusive)

**Trigger**: Product requires ZIP or interactive PDF format.

```
SIMPLIFIED WORKFLOW (Less documented, "tribal knowledge")
═══════════════════════════════════════════════════════════════════════════

Step 1: Create AS400 record for "ER" version

Step 2: Create FileMaker record for "ER" version
├── Data added via normal workflows
├── Travis is alerted (method unclear)
└── Flag: OutputToWeb = "JJ114"

Step 3: Data review
├── OutputToWeb set to "JJ114" (triggers go-live after nightly sync)
└── Review completed

Step 4: Add asset data
├── System: ProductAssets table in FileMaker
├── Fields: Asset name, file reference
└── Link PDF/ZIP to SKU

Step 5: Upload file
├── Destination: AWS bucket /alfred-sellable-digital-products-other
└── Note: NOT uploaded to Legato (incompatible format)

Step 6: Go-live (next day after nightly data download)
├── Available: ax3.com only
├── NOT available: Dealers (Legato cannot handle format)
└── State: LIVE

CURRENT VOLUME: ~5 ER products
```

### 2.4 Key System Integrations and Data Flows

#### Nightly Batch Jobs

| Job | Source | Destination | Trigger | Data Moved |
|-----|--------|-------------|---------|------------|
| AS400 → Royalty | AS400 | Royalty System | Nightly | Products with Royalty=Y |
| FileMaker → SQL Server | FileMaker | SQL Server (tblapproducts, "pending" table) | Nightly CSV export | Product metadata |
| Unknown | FileMaker | ax3.com product database | Nightly | Web-ready product data |

#### Manual Integration Points (Bottlenecks)

| Integration | Actor | Frequency | Latency Impact |
|-------------|-------|-----------|----------------|
| Template data entry | Travis/Digital Team | Per product | Hours |
| Portal upload | Travis/Digital Team | Per batch | Minutes |
| Legato upload | Travis/Digital Team | Per batch | Minutes + wait time |
| MRID download/upload | Travis/Digital Team | Per batch | 5-10 min + processing |
| Asset upload (3x) | Travis/Digital Team | Per product/batch | Minutes-hours |
| FTP upload | Travis/Digital Team | Per batch | Minutes |

### 2.5 State Transitions

```
PHYSICAL/DUMMY PRODUCT LIFECYCLE:
═════════════════════════════════════════════════════════════════════════

[AS400 Created] ─────▶ [FileMaker Populated] ─────▶ [Travis Validated]
                                │
                                ▼
                       [Manager Approved]
                                │
                                ▼
    ┌───────────────────────────┴───────────────────────────┐
    │                                                       │
    ▼                                                       ▼
[Physical Product]                              [Digital-Only Product]
OutputToWeb = JJ114                             OutputToWeb = DIGIONLY
AS400 Status = CUR/TOP/NEW                      AS400 Status = PND
    │                                                       │
    └───────────────────────────┬───────────────────────────┘
                                │
                                ▼
                    [Royalty Rights Configured]
                                │
                                ▼
                    [READY FOR DIGITIZATION]


DIGITAL PRODUCT LIFECYCLE:
═════════════════════════════════════════════════════════════════════════

[Template Uploaded] ─────▶ [SQL Server Validated] ─────▶ [Digital SKU Generated]
                                                                │
            ┌───────────────────────────────────────────────────┤
            │                                                   │
            ▼                                                   ▼
    [AS400 Updated]                                    [Royalty Updated]
            │                                                   │
            └───────────────────────────┬───────────────────────┘
                                        │
                                        ▼
                              [ax3.com Data Ready]
                                        │
                            ┌───────────┴───────────┐
                            │                       │
                            ▼                       ▼
                    [Legato Flow]             [ER Flow]
                            │                       │
                            ▼                       ▼
                    [MRID Assigned]         [AWS Upload]
                            │                       │
                            ▼                       │
                    [Assets Uploaded              │
                    to 3 destinations]             │
                            │                       │
                            └───────────┬───────────┘
                                        │
                                        ▼
                                     [LIVE]
```

### 2.6 Workflow Bottleneck Analysis

| Bottleneck | Description | Impact | Root Cause |
|------------|-------------|--------|------------|
| **Dummy SKU creation** | 34+ fields required in FileMaker + AS400 record for digital-only products | High latency, manual effort | Physical-first architecture |
| **Manual template population** | Data retyped from FileMaker/WebCRD into Excel | Error-prone, time-consuming | No direct system integration |
| **Legato round-trip** | Upload → wait → download MRID → re-upload | Adds hours-days to process | No Legato API |
| **Triple asset upload** | Same PDF uploaded to Legato, AWS, Dropbox | Redundant effort, error risk | No unified asset distribution |
| **Nightly batch dependencies** | FileMaker → SQL sync is daily | 24-hour minimum latency | Batch architecture |
| **Weak validation** | Errors discovered late in workflow | Rework required | No early validation |
| **Performance/Choral metadata dependency** | Must wait for FileMaker data in SQL | Blocks digital processing | Tightly coupled workflow |

### 2.7 Signals for Future-State Opportunities

#### Clear Candidates for Automation/Decoupling

| Signal | Observation | Implication |
|--------|-------------|-------------|
| **Books/Sheets metadata independence** | "Only books and sheets require their data to be entered fresh" — they don't depend on FileMaker nightly sync | Books/Sheets could potentially bypass dummy SKU requirement first |
| **Legato doesn't care about dummy SKU** | "Legato only cares about the digital sku. The dummy sku is something internal to AX3." | External integration is not a barrier to removing dummy requirement |
| **Digital royalties are independent** | "The digital sku is programmed separately, it is not related to the physical sku" | Royalty system may not block digital-first approach |
| **SQL SKU generation is agnostic** | "SQL will generate a digital sku regardless... it just depends on what workflow/product type path we go down" | SKU generation could accept a new "digital master" input |
| **Existing ax3.com-only path (ER)** | ER products already bypass Legato | Pattern exists for ax3.com-exclusive distribution |
| **DIGIONLY flag could be repurposed** | "We could repurpose 'DIGIONLY', we just have to be sure SQL knows to watch for the new value" | Minimal FileMaker changes needed |
| **~500 existing dummy SKUs can stay as-is** | No migration required | Reduces transition risk |

#### Structural Constraints Requiring Deeper Analysis

| Constraint | Nature | Questions |
|------------|--------|-----------|
| **AS400 required for all sales** | Every transaction must have an AS400 record | Could AS400 accept "digital-native" products? What would that record look like? |
| **ax3.com website logic** | "The website (ax3.com) may not be able to pull the appropriate data from the physical sku" | What exactly does the website expect? Who is Jon? |
| **Portal hard-coded assumptions** | "The portal was built off of assumptions for physical" | What changes would be needed? How risky are they? |
| **Performance/Choral metadata coupling** | Require FileMaker → SQL nightly sync | Can this be decoupled? What data is actually needed? |
| **Job Number requirement** | "There need to be a '###95' job number in the AS400 system" | Is this validation logic in SQL? Can it be modified? |

---

## 3. Open Questions for AX3

### 3.1 Systems & Architecture

| # | Question | Why It Matters | Priority |
|---|----------|----------------|----------|
| **S1** | What are the exact names of the nightly batch jobs (AS400 → Royalty, FileMaker → SQL)? Who owns and maintains them? | Understanding job ownership is critical for estimating change complexity | Impact: Medium, Urgency: Low, Uncertainty: High |
| **S2** | What SQL tables receive FileMaker data nightly? What are the differences between "tblapproducts" and the "pending" table mentioned? | Understanding data flow is prerequisite to designing digital-first path | Impact: High, Urgency: Medium, Uncertainty: High |
| **S3** | What specific validations does portal.ax3.com perform when a template is uploaded? Can we get the validation logic documentation or code? | Early validation is key to reducing rework; need to know current state | Impact: High, Urgency: Medium, Uncertainty: High |
| **S4** | Who is Lee, and what is his comfort level with making portal changes? What is the deployment process? | Change capacity is a key planning constraint | Impact: High, Urgency: High, Uncertainty: Medium |
| **S5** | What is the relationship between the portal and ax3.com website? Who is Jon (mentioned as contact for website logic)? | Website changes may be on critical path for digital-first | Impact: High, Urgency: Medium, Uncertainty: High |
| **S6** | What exactly does the "###95" job number requirement enforce? Is this validation in AS400 or SQL? | May be a barrier to simplifying dummy SKU creation | Impact: Medium, Urgency: Low, Uncertainty: High |
| **S7** | The meeting notes mention 3 different digital SKUs in AS400 (PK entire set, score prefix unknown, PC parts). What is the score prefix? | Completeness of SKU type understanding | Impact: Low, Urgency: Low, Uncertainty: Medium |
| **S8** | What is "PT code 95" mentioned in meeting notes? How does it relate to Job Numbers? | May be related to processing flow | Impact: Low, Urgency: Low, Uncertainty: High |

### 3.2 Data & Workflow

| # | Question | Why It Matters | Priority |
|---|----------|----------------|----------|
| **D1** | Can we get access to the FileMaker field spreadsheets mentioned ("FM Product Fields (working)" and "Values for DigitalVSPhysical from PRODUCT")? | Need to understand full field requirements for dummy vs. physical | Impact: High, Urgency: High, Uncertainty: Low |
| **D2** | For Performance/Choral products, exactly which metadata fields are pulled from the FileMaker SQL download? Could any be entered directly instead? | Determines whether metadata coupling can be loosened | Impact: High, Urgency: Medium, Uncertainty: High |
| **D3** | What triggers Travis to know Editorial is done with a product? Is there a status field, email notification, or informal process? | Understanding handoffs is key to workflow optimization | Impact: Medium, Urgency: Low, Uncertainty: High |
| **D4** | What causes the "column AK" import failure? Why hasn't it been fixed permanently? | Indicates technical debt and maintenance capacity | Impact: Low, Urgency: Low, Uncertainty: Medium |
| **D5** | When a performance set has 22 parts, and territory needs to change, why can't this be done in bulk? Is it a UI limitation or data model constraint? | Indicates operational pain point with clear automation potential | Impact: Medium, Urgency: Low, Uncertainty: Medium |
| **D6** | What is the "Ref_SKU" field in SQL Server? Is this the same as the physical/dummy SKU? | Confirm key linking mechanism | Impact: Medium, Urgency: Low, Uncertainty: Low |

### 3.3 Digital-First Feasibility

| # | Question | Why It Matters | Priority |
|---|----------|----------------|----------|
| **F1** | Has anyone tested uploading a Books/Sheets product without a dummy SKU recently? What exactly breaks on ax3.com? | Could be fastest path to digital-first for one product type | Impact: High, Urgency: High, Uncertainty: Medium |
| **F2** | The Q&A mentions products with refsku="Piano" that have no physical relation. How do those work? Can we study them as a model? | Existing precedent may inform digital-first design | Impact: High, Urgency: Medium, Uncertainty: High |
| **F3** | If we created a new value for OutputToWeb (e.g., DIGITALMASTER), what SQL changes would be needed to recognize it? | Scope technical work for new product type | Impact: High, Urgency: Medium, Uncertainty: High |
| **F4** | What would an AS400 record look like for a "true digital-first" product (no physical pretense)? Which fields could be null or have default values? | AS400 is on critical path; need to understand constraints | Impact: High, Urgency: Medium, Uncertainty: High |
| **F5** | The dummy SKU "helps us track, internally, a project." What exactly is being tracked? Could a digital-master SKU serve the same purpose? | Business process dependency on dummy SKU | Impact: Medium, Urgency: Low, Uncertainty: Medium |

### 3.4 Distribution & Channels

| # | Question | Why It Matters | Priority |
|---|----------|----------------|----------|
| **C1** | What would be required to tag a product as "AX3-only" (not distributed to Legato)? Is this the SQL database change mentioned in "Bypassing Legato" notes? | Enables secondary objective of ax3.com-only sales | Impact: High, Urgency: Medium, Uncertainty: Medium |
| **C2** | The "Bypassing Legato" section lists fields the Digital team needs to manipulate. Which of these are currently blocked, and why? | Understanding current restrictions | Impact: Medium, Urgency: Low, Uncertainty: Medium |
| **C3** | Is there demand for ER-type products (interactive PDF, ZIP) to go to dealers? If Legato can't handle them, are there alternative dealer integrations? | Scope of dealer distribution needs | Impact: Low, Urgency: Low, Uncertainty: Medium |
| **C4** | The meeting notes say "not clear whether the current system supports" ax3.com-only sales. Q&A says it's a business decision. Which is accurate for the current state? | Clarify current capability | Impact: Medium, Urgency: Medium, Uncertainty: Medium |

### 3.5 Organizational & Process

| # | Question | Why It Matters | Priority |
|---|----------|----------------|----------|
| **O1** | Who approves digital rights in the Royalty System? What is their capacity for increased volume? | Could become bottleneck if digital volume increases | Impact: Medium, Urgency: Low, Uncertainty: High |
| **O2** | Is there documentation for the "standard intake process" for digital-only metadata mentioned in Q&A? | Understand current process maturity | Impact: Medium, Urgency: Low, Uncertainty: Medium |
| **O3** | How many digital products are processed per week/month currently? What is the capacity constraint? | Baseline for improvement measurement | Impact: Medium, Urgency: Medium, Uncertainty: Medium |
| **O4** | The ER workflow is described as "needs to be documented" and "tribal knowledge." Who are the knowledge holders? | Risk of knowledge loss; documentation opportunity | Impact: Low, Urgency: Low, Uncertainty: Medium |

---

## 4. Glossary

### 4.1 Systems

| Term | Definition |
|------|------------|
| **AS400** | IBM legacy system (iSeries) used for inventory management and sales tracking. All transactions, including digital sales, must have an AS400 record. |
| **ax3.com** | AX3's direct-to-consumer e-commerce website for selling physical and digital products. |
| **Dropbox** | Cloud storage used for archiving digital assets and providing file access to dealers not using Legato. |
| **FileMaker Pro** | Desktop database application (Claris FileMaker) that serves as the source of truth for physical product metadata. |
| **FTP Server (ftp1.ax3.com)** | File transfer server hosting MRID files for integrated dealers (JW Pepper, Stanton's). |
| **Legato** | External distribution platform owned by JW Pepper for dealer integration. Assigns MRIDs and hosts digital assets for dealer distribution. Has no API. |
| **portal.ax3.com** | Internal web application for template uploads, SKU generation, and MRID management. |
| **Royalty System** | Custom database application tracking composition rights, permitted usage, and royalty payments. Updated from AS400 overnight. |
| **SQL Server** | Microsoft database storing digital product metadata and generating digital SKUs. Central hub for digital product processing. |
| **WebCRD** | Print job management system (RSA WebCRD) providing metadata for Performance Music products. |
| **AWS S3** | Amazon cloud storage hosting downloadable files for ax3.com customers. Buckets: alfred-dsm-pdfs, alfred-catfiles. |

### 4.2 Product Types

| Term | Definition |
|------|------------|
| **Books** | Full book publications (method books, folios). Use PB SKU prefix. |
| **Choral/Octavos** | Music for vocal ensembles (choirs). Standard format ~7" x 10.5". Use PO SKU prefix. |
| **Electronic Resources (ER)** | Non-standard digital products (ZIP files, interactive PDFs). Cannot be distributed through Legato. |
| **Performance Sets** | Music for ensembles (bands, orchestras). Include score + individual parts. Use PK (set), PR (score), PC (part) prefixes. |
| **Sheets** | Single sheet music or smaller publications. Use PS SKU prefix. |

### 4.3 SKU Types

| Term | Definition |
|------|------------|
| **Digital SKU** | Any SKU containing a hyphen (-). Format: 00-[TYPE]-[SERIAL]. Generated by SQL Server. |
| **Dummy SKU** | Placeholder physical product record created solely to satisfy system requirements for digital-only products. |
| **Physical SKU** | Standard product identifier for printed products. Numeric only (e.g., 47888). |
| **Ref_SKU** | SQL Server field linking digital SKUs to their parent physical or dummy SKU. |

### 4.4 SKU Type Codes

| Code | Name | Description |
|------|------|-------------|
| PB | Book | Full book publication |
| PC | Part | Individual instrument part (with suffix, e.g., _F1 for Flute 1) |
| PK | Pack/Set | Complete ensemble set (score + all parts) |
| PO | Octavo | Choral octavo |
| PR | Score | Conductor's score from ensemble set |
| PS | Sheet | Standalone sheet music |
| ER | Electronic Resource | ZIP files, interactive PDFs |

### 4.5 Status Codes and Flags

| Code | System | Meaning |
|------|--------|---------|
| CUR | AS400 | Current — active product |
| DIGIONLY | FileMaker (OutputToWeb) | Digital-only product (requires dummy SKU) |
| JJ114 | FileMaker (OutputToWeb) | Approved for digital conversion |
| NEW | AS400 | New release |
| PND | AS400 | Pending — not yet fully active (used for digital-only) |
| TOP | AS400 | Top seller |

### 4.6 Other Terms

| Term | Definition |
|------|------------|
| **_CC (Condensed Score)** | Special part suffix for condensed conductor score. Must have sort order "1" and page count "1". |
| **ContributorRole** | FileMaker table holding contributor listings (composer, arranger, etc.) per product. |
| **MRID** | Legato's unique product identifier. Required for dealer distribution. Assigned by Legato after product upload. |
| **Nightly Sync/Job** | Batch processes that run overnight to synchronize data between systems (AS400→Royalty, FileMaker→SQL). |
| **Product Table** | Main FileMaker table containing product metadata. |
| **ProductAssets** | FileMaker table mapping SKUs to their associated digital files. Used for ER products. |
| **Template** | Excel spreadsheet used to submit product data to portal.ax3.com. Three types: Performance, Sheets, Choral. |
| **TOCSongSources** | FileMaker table listing song titles within a publication (used for multi-song SKUs). |
| **UPC** | Universal Product Code. Barcode identifier. Must be consistent across all parts of a set. |

### 4.7 People & Roles

| Name/Role | Description |
|-----------|-------------|
| **Chris Rubeiz** | Former developer who built portal.ax3.com. No longer primary maintainer. |
| **Doug Fraser** | Original developer of the SQL processing system. |
| **Editorial Team** | AX3 team responsible for adding product metadata in FileMaker. |
| **Jon** | Contact for ax3.com website logic (mentioned in Q&A). |
| **Lee** | Current internal staff maintaining portal code. Comfort level with changes unknown. |
| **Purchasing Team** | AX3 team responsible for creating AS400 records. |
| **Travis** | X Team member who validates data and performs digitization workflow. |

---

*End of Document*
