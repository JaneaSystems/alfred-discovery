# Alfred Music Digital Product Management System - Project Summary

> **Purpose**: This document provides a comprehensive overview of the Alfred Music digitization project for AI language models. It summarizes the business context, technical architecture, workflows, and modernization initiatives.
>
> **Last Updated**: February 5, 2026

---

## Table of Contents

1. [Executive Overview](#executive-overview)
2. [Business Context](#business-context)
3. [System Architecture](#system-architecture)
4. [Product Types and Workflows](#product-types-and-workflows)
5. [Current Pain Points](#current-pain-points)
6. [Modernization Initiative](#modernization-initiative)
7. [Key Systems and Data Flow](#key-systems-and-data-flow)
8. [Important Terminology](#important-terminology)
9. [Project Documentation Structure](#project-documentation-structure)

---

## Executive Overview

### What Is This Project?

Alfred Music is a music publishing company owned by Peaksware that produces and distributes sheet music for musicians, educators, and institutions worldwide. This project focuses on **modernizing the digital product creation and distribution workflow** for sheet music.

### The Core Challenge

Alfred Music's systems were originally designed for physical print products. The current architecture requires every digital product to have a corresponding physical SKU (or a "dummy" physical SKU placeholder), creating significant inefficiencies and manual work.

### Strategic Goals

1. **Reduce latency**: Decrease time from "ready to digitize" to "live for sale"
2. **Increase automation**: Minimize manual data entry and file handling
3. **Enable digital-first**: Allow digital products without requiring physical/dummy SKUs
4. **Unify workflows**: Consolidate fragmented processes and templates
5. **Improve quality**: Reduce errors through validation and automation

### Current State

- **10+ interconnected systems** (FileMaker, AS400, SQL Server, Royalty System, etc.)
- **3 digitization paths** (Physical-to-Digital, Digital-Only, Electronic Resources)
- **4 product types** (Performance Sets, Choral, Books, Sheets)
- **Highly manual process** requiring template creation, data entry, file renaming, and multi-system updates
- **Sales channels**: Alfred.com (direct) and Legato platform (dealers like JW Pepper)

---

## Business Context

### Company Overview

- **Name**: Alfred Music
- **Parent**: Peaksware (technology and media company)
- **Industry**: Music publishing
- **Products**: Sheet music across multiple genres and formats
- **Distribution**: Direct sales (Alfred.com) and dealer network (via Legato platform)

### Product Formats

| Format | Description | Distribution |
|--------|-------------|--------------|
| **Physical** | Printed sheet music shipped to customers | Alfred.com + Dealers |
| **Digital** | Downloadable PDF files, instant delivery | Alfred.com + Dealers |
| **Electronic Resources (ER)** | ZIP files with multiple assets, interactive PDFs | Alfred.com only |

### Historical Context

Alfred's infrastructure was built when physical print was the only product format. When digital products emerged, the company **extended the physical product workflow** rather than building a native digital-first system. This created the current constraint:

> **Digital products cannot be created without a physical product (or "dummy" physical product) existing first in the system.**

This constraint affects:
- Product database (FileMaker) requires a physical SKU record
- Digital SKU generation derives from physical SKU identifiers
- Royalty System attaches rights to physical/dummy SKUs
- AS400 inventory system expects product records even for digital-only items

---

## System Architecture

### Core Systems Overview

Alfred Music's digitization workflow spans **10 interconnected systems**:

```
┌─────────────────────────────────────────────────┐
│            ALFRED MUSIC SYSTEMS                 │
├─────────────────────────────────────────────────┤
│                                                 │
│  FileMaker → AS400 → SQL Server → Alfred.com   │
│     ↓          ↑          ↓                     │
│  WebCRD    Royalty     Legato                   │
│     ↓        System      ↓                      │
│   Excel                FTP                      │
│  Templates              ↓                       │
│                      Dealers                    │
│     ↓                                           │
│    AWS/Dropbox (File Storage)                   │
│                                                 │
└─────────────────────────────────────────────────┘
```

### System Details

| System | Role | Type | Key Function |
|--------|------|------|--------------|
| **FileMaker** | Product metadata database | Desktop database | Source of truth for physical product information |
| **AS400** | Inventory & sales tracking | IBM legacy system | Tracks inventory quantities and sales transactions |
| **SQL Server** | Digital product metadata | Microsoft database | Generates digital SKUs, stores digital-specific data |
| **Royalty System** | Rights management | Custom database | Tracks composition ownership and royalty payments |
| **portal.alfred.com** | Data ingestion | Internal web app | Receives template uploads, triggers processing |
| **WebCRD** | Print job management | Print workflow system | Stores print production data |
| **Legato** | Dealer distribution | External platform | JW Pepper's platform for music retailers |
| **Alfred.com** | E-commerce | Public website | Direct-to-consumer sales |
| **AWS** | File storage | Cloud storage | Stores PDF assets |
| **Dropbox** | File archive & sharing | Cloud storage | Archive and dealer file distribution |

### Critical System Constraints

| System | Modification Difficulty | Notes |
|--------|------------------------|-------|
| AS400 | 🟡 Small changes possible | Limited scope, legacy platform |
| portal.alfred.com | 🔴 Possible but hard | Minimize changes; wrap rather than modify |
| SQL Server | 🟢 Easy to change | New tables, stored procedures feasible |
| New Infrastructure | 🟢 Unlimited | New VMs, services, APIs all available |

---

## Product Types and Workflows

### Four Main Product Types

1. **Performance Sets (Ensembles)**
   - Band or orchestra music with multiple instrument parts
   - Most complex: one product creates 20+ digital SKUs (score, parts, set)
   - Example: Concert band piece with score + 25 instrument parts

2. **Choral Music**
   - Choral octavos for school, church, community choirs
   - Typically single-part or SATB arrangements
   - Simpler structure than performance sets

3. **Books**
   - Full method books for learning instruments
   - Educational materials with multiple pages/songs
   - Almost identical to sheets (minor SKU differences)

4. **Sheets**
   - Individual standalone pieces
   - Solo instrumental or vocal music
   - Simplest product type

### Three Digitization Paths

#### Path A: Physical-to-Digital

- **Purpose**: Converting existing physical products to digital format
- **Trigger**: Physical product exists, decision made to digitize
- **FileMaker flag**: `Outputtoweb = JJ114`
- **AS400 status**: `CUR`, `TOP`, `NEW`, or `PND`
- **Distribution**: Alfred.com + all Legato dealers
- **Dummy SKU required**: No (real physical SKU exists)
- **Most common path** - converting existing print catalog

#### Path B: Digital-Only (with Dummy SKU)

- **Purpose**: Creating digital product without physical version
- **Trigger**: No physical product exists, standard PDF format
- **FileMaker flag**: `Outputtoweb = DIGIONLY`
- **AS400 status**: `PND`
- **Distribution**: Alfred.com; dealers via Dropbox
- **Dummy SKU required**: Yes (placeholder to satisfy system requirements)
- **Use cases**: Niche arrangements, test releases, low-volume specialty products

#### Path C: Electronic Resources (Alfred.com Exclusive)

- **Purpose**: Non-standard digital products (ZIP, interactive PDFs)
- **File format**: ZIP files or Interactive PDFs with embedded media
- **SKU type**: ER (Electronic Resource)
- **Distribution**: Alfred.com only (dealers cannot process format)
- **Legato integration**: None (incompatible format)
- **Use cases**: Interactive method books, curriculum kits, multimedia materials

---

## Current Pain Points

### Manual and Time-Consuming Processes

1. **Template Creation**
   - Staff manually creates Excel templates for each product
   - Three different template types (Performance, Sheets/Books, Choral)
   - Data must be manually entered from multiple sources
   - Templates are one-time use (not retained after upload)

2. **Multi-System Data Entry**
   - Same information entered in multiple systems
   - No automated synchronization
   - High risk of inconsistencies

3. **File Management**
   - Files received via Slack, AWS, Dropbox
   - Manual renaming required (different conventions for each product type)
   - Asset uploads to portal.alfred.com are manual
   - FTP uploads to dealers are manual

4. **Validation Gaps**
   - Limited automated validation of data completeness
   - Errors discovered late in process
   - Manual quality checks required

### Architectural Constraints

5. **Dummy SKU Requirement**
   - Digital-only products must create placeholder physical SKU
   - Adds complexity and time
   - Clutters product database with non-physical records

6. **No Alfred.com-Only Distribution Flag**
   - Cannot easily restrict products to Alfred.com only
   - Dealer distribution is all-or-nothing
   - Manual workarounds required

7. **Legato Integration Complexity**
   - Multi-step MRID (Music Retailer ID) loop:
     1. Generate export file from SQL Server
     2. Upload to Legato
     3. Download MRIDs from Legato
     4. Upload MRIDs back to portal
   - Manual column deletion required (remove "AK" column)
   - No automated synchronization

8. **Limited Workflow Visibility**
   - No central dashboard showing product status
   - Difficult to track where products are in the pipeline
   - No audit trail of actions

---

## Modernization Initiative

### Phase 1: Quick-Win Automation (4-6 weeks)

**Objective**: Reduce manual effort by ~60% while laying groundwork for future architecture

**Components**:

1. **Digital Product Orchestration Service**
   - Central service coordinating all digital product workflows
   - Tracks product state across entire pipeline
   - REST API for tool integration
   - Dashboard showing pipeline status
   - New SQL Server tables for state management

2. **Template Auto-Generator**
   - Automatically generates Excel templates from FileMaker/WebCRD data
   - Eliminates manual data entry
   - Pre-validates data before template creation

3. **Pre-Upload Validator**
   - Validates template data before portal upload
   - Checks UPC consistency, sort order, required fields
   - Reduces import failures

4. **File Renamer Tool**
   - Automatically renames PDFs to portal.alfred.com conventions
   - Handles all product types
   - Eliminates manual renaming errors

5. **Unified Asset Uploader**
   - Single interface for uploading PDFs to portal
   - Batch processing capability
   - Error handling and retry logic

6. **FTP Auto-Sync**
   - Automated upload to dealer FTP sites
   - Scheduled synchronization
   - Eliminates manual FTP steps

### Phase 2: Architecture Evolution

**Objective**: Enable digital-first product creation without dummy SKUs

**Components**:

1. **Digital-First SKU Path**
   - Create digital products without physical/dummy SKU requirement
   - New workflow bypassing FileMaker for digital masters
   - Metadata stored directly in SQL Server

2. **Alfred.com-Only Distribution Flag**
   - New field indicating products should not go to Legato
   - Bypass MRID workflow for Alfred-exclusive products

3. **New Digital Product API**
   - REST API for creating digital products programmatically
   - Reduces dependency on portal.alfred.com
   - Foundation for future integrations

4. **Minimal Portal Modifications**
   - Wrapper approach: new API layer around existing portal
   - Avoid deep portal changes (difficult to modify)

### Future Vision: Content Hub (2026 Initiative)

- Single source of truth for all product metadata
- Parallel workflows for different teams (art, audio, metadata)
- Production Owner dashboard with gate/checkpoint visibility
- Automated distribution to all sales channels
- Print-on-demand capability for physical products

---

## Key Systems and Data Flow

### SKU System

**SKU** (Stock Keeping Unit): Unique identifier for each product

**Key Rule**: Any SKU containing a hyphen (-) is a digital SKU

**Digital SKU Format**: `00-XX-0000000` where XX is the product type code

| Code | Name | Description | Example |
|------|------|-------------|---------|
| PK | Pack/Set | Complete ensemble set (score + all parts) | 00-PK-0047888 |
| PR | Score | Conductor's score from ensemble | 00-PR-0047888 |
| PC | Part | Individual instrument part | 00-PC-0047888_F1 |
| PO | Octavo | Choral octavo | 00-PO-0012345 |
| PS | Sheet | Standalone sheet music | 00-PS-0098765 |
| PB | Book | Full book publication | 00-PB-0054321 |
| ER | Electronic Resource | ZIP files, interactive PDFs | 00-ER-0011111 |

**Example Relationship**:
```
Physical SKU: 47888
    └── Digital SKUs:
        ├── 00-PK-0047888 (Complete Set)
        ├── 00-PR-0047888 (Score)
        ├── 00-PC-0047888_F1 (Flute 1)
        ├── 00-PC-0047888_CL1 (Clarinet 1)
        └── ... (20+ additional parts)
```

### Data Flow Overview

```
1. Purchasing Team creates product in AS400 (sequential number)
   ↓
2. Nightly job copies data (Royalty=Y) to Royalty System
   ↓
3. Editorial Team adds metadata in FileMaker
   ↓
4. Travis (X team) manually checks and fixes data
   ↓
5. Product Manager reviews and approves
   ↓
6. Nightly export from FileMaker to SQL Server (CSV)
   ↓
7. Travis creates Excel template (manual)
   ↓
8. Travis uploads template to portal.alfred.com
   ↓
9. SQL Server generates digital SKUs
   ↓
10. SKUs propagated to AS400, Royalty System, Alfred.com
    ↓
11. (Optional) Legato integration for dealer distribution
    ↓
12. Asset files uploaded to portal and AWS
    ↓
13. (Optional) FTP upload to dealers
```

### Prerequisites for Product to Go Live

**Physical-Digital Product (Path A)**:
- FileMaker: `Outputtoweb = JJ114`
- AS400: `Status = CUR`
- Royalty System: Digital rights setup for physical SKU

**Digital-Only Product (Path B)**:
- FileMaker: `Outputtoweb = DIGIONLY`
- AS400: `Price = USD`, `Territories = Value`, `Status = PND`
- Royalty System: Digital rights setup for dummy SKU

---

## Important Terminology

| Term | Definition |
|------|------------|
| **Dummy SKU** | Placeholder physical SKU created solely to satisfy system requirements for digital-only products |
| **Outputtoweb** | FileMaker field indicating digital eligibility (`JJ114` or `DIGIONLY`) |
| **MRID** | Music Retailer ID - Legato's unique identifier for products |
| **Template** | Excel spreadsheet with product metadata for portal upload |
| **PT code 95** | Job number designation in AS400 required for digital processing |
| **Form number** | Sequence number indicating position of a part within a performance set |
| **_CC** | Condensed/Conductor score (special handling: sort order "1", page count "1") |
| **UPC** | Universal Product Code - must be identical across all parts in a set |
| **Ref_SKU** | SQL Server field linking digital SKUs to their physical/dummy SKU origin |
| **WebCRD** | Print workflow system storing production job data |
| **Legato** | JW Pepper's platform for digital distribution to music retailers |

---

## Project Documentation Structure

### Core Documents

1. **AX3_Music_Complete_Knowledge_Base.md** (1,446 lines)
   - Comprehensive reference document for AI models
   - Detailed system architecture and workflows
   - Product taxonomy and SKU system
   - Business context and pain points
   - Reference tables and glossary

2. **Meeting-Deep-Dive1.md**
   - Notes from February 2, 2026 meeting
   - Product types and workflow details
   - Preliminary setup requirements
   - Template preparation process
   - AS400 and Legato export details

3. **questions-answers.md** (352 lines)
   - Q&A about data model and system structure
   - FileMaker database structure details
   - AS400 integration specifics
   - Field requirements and differences
   - Side-by-side comparisons of dummy vs. real SKUs

4. **Technical-Specification-Phases-1-2.md** (884 lines)
   - Technical specification for modernization project
   - Phase 1 quick-win automation components
   - Phase 2 architecture evolution plans
   - Infrastructure constraints and flexibility
   - Effort estimates and risk register

5. **Full Process Digital to Alfred.com Workflow.drawio**
   - Visual workflow diagram
   - Shows product types and digitization paths
   - System interactions and data flow
   - Template creation and upload process

### Supporting Directories

- **DeepDive-1/** - Analysis documents and workflow diagrams
- **future/** - Future plans including Python script for discovery questions
- **Images/** - Supporting visual assets

---

## Current Team and Responsibilities

- **Travis** (X team): Manual data checking, template creation, file management
- **Purchasing Team**: Creates new products in AS400
- **Editorial Team**: Adds metadata in FileMaker
- **Product Manager**: Final review and approval before products go live

---

## Key Insights for AI Models

### Understanding the Constraint

The single most important constraint to understand: **Alfred Music's systems require a physical SKU to exist before any digital product can be created**. This is not a business requirement but a technical limitation of the legacy architecture.

### Why Dummy SKUs Exist

Dummy SKUs are a workaround for the physical-first architecture:
- They have no inventory
- They will never be printed or shipped
- They exist solely to satisfy database foreign key relationships and system expectations
- They contain minimal data compared to real physical products

### Three Paths, Same Underlying Systems

While there are three distinct digitization paths, they all flow through the same core systems (FileMaker, AS400, SQL Server). The paths differ in:
- Data sources
- Required fields
- Distribution channels
- Process complexity

### Modernization Strategy

The modernization approach is **pragmatic and incremental**:
- **Phase 1**: Automate current processes without major architectural changes
- **Phase 2**: Introduce digital-first capabilities while maintaining backward compatibility
- **Future**: Complete content hub transformation

This approach recognizes that:
- Legacy systems (AS400, portal) are difficult to change
- SQL Server is flexible and easy to extend
- New infrastructure can wrap and orchestrate legacy systems
- Business operations must continue during transformation

---

## Success Metrics

### Phase 1 Goals

- Reduce manual template creation time by 90%
- Eliminate manual file renaming errors
- Provide real-time workflow visibility
- Reduce time-to-market by 40-60%

### Phase 2 Goals

- Enable digital-first products without dummy SKUs
- Support Alfred.com-exclusive distribution
- Provide API access for future integrations
- Further reduce time-to-market by 30-40%

---

## Questions and Discovery Areas

### Open Questions (as of Feb 2026)

1. **Nightly Jobs**: 
   - What are the exact names of the AS400 and FileMaker sync jobs?
   - Who owns and maintains these jobs?

2. **FileMaker Completion Signal**:
   - How do downstream systems know when Travis completes data checking?
   - What field changes indicate completion?

3. **SQL Server Tables**:
   - What are the exact table names for digital-only vs. physical-digital products?

4. **PT Code 95**:
   - What is the significance of this code?
   - How is it used in processing?

5. **Template Retention**:
   - Why aren't templates kept after upload?
   - Could retaining them provide value?

### Areas Requiring Further Investigation

- Detailed portal.alfred.com API capabilities
- AS400 modification procedures and constraints
- Royalty System integration points
- Error handling and recovery procedures
- Current performance bottlenecks
- User pain points beyond automation opportunities

---

## Conclusion

The Alfred Music digitization project represents a **systematic modernization of a legacy physical-first architecture** to support digital-first workflows. The project balances pragmatic automation of current processes (Phase 1) with strategic architectural evolution (Phase 2), recognizing the constraints of legacy systems while building toward a future content hub vision.

The key to understanding this project is recognizing that **technical debt and architectural constraints are the primary drivers of complexity**, not business requirements. The actual business need (sell digital sheet music) is straightforward; the implementation complexity stems from extending a physical product infrastructure to support digital products.

Success will be measured not just in automation and time savings, but in **architectural flexibility** - the ability to create digital products natively without physical product dependencies, enabling Alfred Music to respond rapidly to market opportunities and customer needs.

---

**For AI Models**: This summary provides sufficient context to discuss, analyze, and reason about the Alfred Music digitization process. Refer to the detailed knowledge base documents for specific system behaviors, data schemas, and workflow steps.
