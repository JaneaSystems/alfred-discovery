# Alfred Music Digitization Process: Complete Knowledge Base

> **Document Purpose**: This document provides comprehensive information about Alfred Music's digitization process for use by AI language models. It contains business context, technical architecture, workflows, terminology, and operational details sufficient to discuss, analyze, and reason about the digitization process without additional context.

---

## Table of Contents

1. [Company Overview](#1-company-overview)
2. [Business Context](#2-business-context)
3. [System Architecture](#3-system-architecture)
4. [Product Taxonomy](#4-product-taxonomy)
5. [SKU System](#5-sku-system)
6. [Digitization Paths](#6-digitization-paths)
7. [Detailed Process Workflows](#7-detailed-process-workflows)
8. [Data Flows](#8-data-flows)
9. [Architectural Constraints](#9-architectural-constraints)
10. [Known Pain Points](#10-known-pain-points)
11. [Glossary](#11-glossary)
12. [Reference Tables](#12-reference-tables)

---

## 1. Company Overview

### 1.1 About Alfred Music

Alfred Music is a music publishing company that produces and distributes sheet music for musicians, educators, and institutions worldwide. The company is owned by Peaksware, a technology and media company focused on the music and endurance sports markets.

Alfred Music's catalog spans multiple genres and formats:
- Concert band and orchestra music
- Choral music for school, church, and community choirs
- Solo instrumental and vocal sheet music
- Educational method books for learning instruments
- Play-along products with audio accompaniment
- Interactive educational materials

### 1.2 Sales Channels

Alfred Music sells products through two primary channels:

**Direct Sales (Alfred.com)**
- Company-owned e-commerce website
- Sells both physical and digital products
- Full catalog availability
- Direct customer relationship

**Dealer Distribution (via Legato)**
- Partnership with music retailers
- Primary partner: JW Pepper (owns the Legato platform)
- Other dealers: Stanton's, and various music stores
- Dealers sell Alfred products through their own storefronts
- Digital distribution requires MRID (Legato's product identifier)

### 1.3 Product Formats

**Physical Products**
- Printed sheet music shipped to customers
- Inventory tracked in warehouse
- Sold through Alfred.com and dealers

**Digital Products**
- Downloadable PDF files
- No physical inventory
- Instant delivery
- Sold through Alfred.com and Legato-integrated dealers

**Electronic Resources (ER)**
- ZIP files containing multiple assets
- Interactive PDFs with embedded audio/video
- Sold exclusively on Alfred.com (dealers cannot process these formats)

---

## 2. Business Context

### 2.1 Historical Context

Alfred Music's systems and processes were designed when physical print was the only product format. The company's infrastructure—databases, inventory systems, royalty tracking, and workflows—all assume physical products as the primary entity.

When digital products emerged as a market opportunity, Alfred adapted by extending the physical product workflow rather than building a native digital-first system. This created the current architecture where digital products are derived from physical products.

### 2.2 The Core Constraint

**Digital products cannot be created without a physical product (or a "dummy" physical product) existing first in the system.**

This constraint permeates the entire architecture:
- FileMaker (product database) requires a physical SKU record
- Digital SKU generation derives from physical SKU identifiers
- Royalty System attaches rights to physical/dummy SKUs
- AS400 (inventory system) expects product records even for digital-only items

For digital-only products, staff must create a "dummy" physical SKU—a placeholder record with minimal data—solely to satisfy system requirements before creating the actual digital product.

### 2.3 Strategic Goals

Alfred Music's modernization initiative aims to:
1. **Reduce latency**: Decrease time from "ready to digitize" to "live for sale"
2. **Increase automation**: Minimize manual data entry and file handling
3. **Enable digital-first**: Allow digital products without requiring physical/dummy SKUs
4. **Unify workflows**: Consolidate fragmented processes and templates
5. **Improve quality**: Reduce errors through validation and automation

### 2.4 Future Vision: Content Hub

Alfred's long-term aspiration (2026 initiative) is a "Content Hub" architecture where:
- Single source of truth for all product metadata
- Parallel workflows for different teams (art, audio, metadata)
- Production Owner dashboard with gate/checkpoint visibility
- Automated distribution to all sales channels
- Print-on-demand capability for physical products

---

## 3. System Architecture

### 3.1 System Overview

Alfred Music's digitization workflow spans ten interconnected systems. Each system serves a specific purpose, and they communicate through automated syncs, manual file transfers, and batch processes.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        ALFRED MUSIC SYSTEMS                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐       │
│  │  FileMaker   │     │    AS400     │     │   Royalty    │       │
│  │   (Product   │────▶│  (Inventory  │     │   System     │       │
│  │   Metadata)  │     │    Sales)    │     │   (Rights)   │       │
│  └──────┬───────┘     └──────────────┘     └──────────────┘       │
│         │                    ▲                    ▲                 │
│         ▼                    │                    │                 │
│  ┌──────────────┐           │                    │                 │
│  │   WebCRD     │           │                    │                 │
│  │   (Print     │           │                    │                 │
│  │    Jobs)     │           │                    │                 │
│  └──────┬───────┘           │                    │                 │
│         │                    │                    │                 │
│         ▼                    │                    │                 │
│  ┌──────────────┐     ┌─────┴────────┐          │                 │
│  │    Excel     │     │              │          │                 │
│  │  Templates   │────▶│  SQL Server  │──────────┘                 │
│  │  (Manual)    │     │  (Digital    │                            │
│  └──────────────┘     │   SKUs)      │                            │
│                       └──────┬───────┘                            │
│                              │                                     │
│         ┌────────────────────┼────────────────────┐               │
│         ▼                    ▼                    ▼               │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐      │
│  │   Legato     │     │  Alfred.com  │     │     AWS      │      │
│  │  (Dealers)   │     │  (Direct)    │     │  (Storage)   │      │
│  └──────────────┘     └──────────────┘     └──────────────┘      │
│         │                                         ▲               │
│         ▼                                         │               │
│  ┌──────────────┐     ┌──────────────┐           │               │
│  │     FTP      │     │   Dropbox    │───────────┘               │
│  │  (Dealers)   │     │  (Archive)   │                           │
│  └──────────────┘     └──────────────┘                           │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

### 3.2 Core Systems Detail

#### 3.2.1 FileMaker Pro

**Role**: Source of truth for physical product metadata

**Type**: Desktop database application (Claris FileMaker)

**Function**:
- Stores comprehensive product information
- Manages catalog data for physical products
- Provides data for digitization process
- Source for batch searches and data exports

**Key Data Elements**:
| Field | Description |
|-------|-------------|
| SKU | Unique product identifier |
| Title | Product name |
| Composer | Original composer name |
| Arranger | Arranger name (if applicable) |
| Instrumentation | Instruments/voices required |
| Price | Retail price |
| UPC | Universal Product Code (barcode) |
| Outputtoweb | Digital eligibility flag |

**Key Flags**:
| Value | Meaning |
|-------|---------|
| `JJ114` | Approved for digital (physical-to-digital conversion) |
| `DIGIONLY` | Digital-only product (requires dummy physical SKU) |

**Limitations**:
- Desktop application not designed for API access
- Data extraction requires manual export or batch search
- No native integration with web services
- Changes require manual sync to other systems

#### 3.2.2 AS400

**Role**: Inventory management and sales tracking

**Type**: IBM AS/400 (iSeries) legacy system

**Function**:
- Tracks physical inventory quantities
- Records sales transactions
- Manages product status lifecycle
- Calculates reorder points

**Key Data Elements**:
| Field | Description |
|-------|-------------|
| SKU | Product identifier |
| Status | Product lifecycle state |
| Price | Pricing by currency |
| Territories | Geographic availability |
| Inventory | Quantity on hand |

**Status Codes**:
| Code | Meaning | Description |
|------|---------|-------------|
| `CUR` | Current | Active product in catalog |
| `TOP` | Top Seller | High-volume product |
| `NEW` | New Release | Recently published |
| `PND` | Pending | Not yet fully active |

**Limitations**:
- Legacy system with limited modern integration capabilities
- Designed for physical inventory—digital products require placeholder records
- Changes may require specialized AS400 programming skills

#### 3.2.3 SQL Server

**Role**: Digital product metadata and SKU generation

**Type**: Microsoft SQL Server database

**Function**:
- Stores digital-specific product metadata
- Generates digital SKUs based on business rules
- Processes template uploads from portal.alfred.com
- Triggers synchronization to other systems

**Key Operations**:
- Digital SKU creation (applies naming conventions)
- Metadata storage and retrieval
- Data propagation to AS400, Royalty System, Alfred.com
- Report generation for Legato integration

**Relationship to Other Systems**:
- Receives data from portal template uploads
- Syncs digital SKU data to AS400
- Updates Royalty System with new digital products
- Feeds product data to Alfred.com storefront

#### 3.2.4 Royalty System

**Role**: Rights management and royalty tracking

**Type**: Custom database application

**Function**:
- Tracks composition ownership and rights holders
- Defines permitted usage (print, digital, territories)
- Calculates royalty payments based on sales
- Validates digital distribution eligibility

**Key Concepts**:
| Concept | Description |
|---------|-------------|
| Rights Holder | Entity entitled to royalty payments |
| Territory | Geographic region where sales are permitted |
| Usage Rights | Permitted formats (print, digital, streaming) |
| Royalty Rate | Percentage or fixed amount per sale |

**Requirement**: Digital rights must be explicitly configured for a SKU before digital sales are permitted. Rights are attached to SKU records (physical or dummy).

**Limitation**: Rights configuration is a manual process performed by authorized staff.

#### 3.2.5 Portal (portal.alfred.com)

**Role**: Data ingestion and workflow management

**Type**: Internal web application

**Function**:
- Receives Excel template uploads from staff
- Validates uploaded data
- Triggers SQL Server processing
- Manages MRID upload/download workflow
- Provides template downloads

**Key URLs and Functions**:
| Function | Endpoint |
|----------|----------|
| Template upload | portal.alfred.com (template dropdown) |
| MRID upload | portal.alfred.com/TSM/Digital/UpdateMRID.aspx |
| Template downloads | portal.alfred.com/DigitalTemplates/ |

**Templates Available**:
- Performance_Template.xlsx (ensemble music)
- Sheets_Template.xls (sheet music and books)
- Choral_Template.xls (choral octavos)

**Known Issues**:
- Column AK in some exports causes import failures
- Workaround: manually delete column AK and resave as XLSX

#### 3.2.6 Legato

**Role**: External distributor integration platform

**Owner**: JW Pepper (third-party music retailer)

**Function**:
- Enables music dealers to access and sell Alfred's digital products
- Assigns MRIDs (unique product identifiers for dealer systems)
- Hosts digital assets for dealer distribution
- Processes product metadata uploads

**Key URLs**:
| Function | URL |
|----------|-----|
| Data upload | app.legatomedia.com/publisher/vendors/index/ |
| Asset upload | app.legatomedia.com/publisher/assets/uploadWizard/ |

**MRID Concept**:
- MRID = unique identifier assigned by Legato
- Required for dealer distribution
- Must be retrieved from Legato after product upload
- Must be re-uploaded to Alfred portal and FTP for dealers

**Workflow**:
1. Alfred uploads product data to Legato
2. Legato processes and assigns MRID
3. Alfred downloads MRID files (full + monthly delta)
4. Alfred uploads MRIDs to portal.alfred.com
5. Alfred uploads MRID files to FTP for integrated dealers

**Limitations**:
- Web portal interface only (no API documented)
- Cannot handle ZIP files or interactive PDFs
- Upload/download cycle adds latency
- External system outside Alfred's control

#### 3.2.7 AWS S3

**Role**: Cloud storage for digital assets

**Provider**: Amazon Web Services

**Buckets**:
| Bucket | Purpose |
|--------|---------|
| `alfred-dsm-pdfs` | Digital sheet music PDF files |
| `alfred-catfiles` | Cover images and catalog assets |

**Function**:
- Hosts downloadable files for Alfred.com customers
- Serves assets when customers complete purchases
- Provides reliable, scalable file delivery

**Access Pattern**: Files uploaded during digitization process; served to customers via Alfred.com

#### 3.2.8 Dropbox

**Role**: Archive storage and dealer file access

**Function**:
- Stores archival copies of all digital assets
- Provides file access for dealers not using Legato
- Backup repository for source files

**Usage**:
- Digital-only products: dealers access files via Dropbox
- Archive: all digitized products stored for recovery
- Source files: print production archives

#### 3.2.9 FTP Server (ftp1.alfred.com)

**Role**: Dealer data distribution

**Function**:
- Hosts MRID files for fully integrated dealers
- Dealers pull product data from designated folders

**Key Path**: "Dealerftp" folder

**Files Distributed**:
- MR_Upload_Dealer_Full.txt (complete MRID list)
- MR_Upload_Dealer.txt (monthly delta)

**Integrated Dealers**: JW Pepper, Stanton's, and others with direct system integration

#### 3.2.10 WebCRD

**Role**: Print job management

**Type**: Rochester Software Associates (RSA) WebCRD

**Function**:
- Manages print production workflow
- Tracks print jobs and specifications
- Source of metadata for Performance Music products

**Usage in Digitization**:
- Data downloaded for Performance Music template population
- Provides print specifications that inform digital product setup

---

## 4. Product Taxonomy

### 4.1 Music Product Categories

#### 4.1.1 Performance Music (Ensemble Sets)

**Description**: Music for groups of musicians playing together, such as bands, orchestras, and jazz ensembles.

**Components**:
| Component | Description |
|-----------|-------------|
| Score | Master view showing all parts; used by conductor |
| Parts | Individual booklets for each instrument |
| Set | Complete package (score + all parts) |

**Ensemble Types**:
| Type | Description |
|------|-------------|
| Concert Band | Wind and percussion ensemble (woodwinds, brass, percussion) |
| Orchestra | Strings, woodwinds, brass, and percussion |
| Marching Band | Mobile ensemble for parades and sporting events |
| Jazz Band | Saxophones, trumpets, trombones, rhythm section |
| String Orchestra | Strings only (violin, viola, cello, bass) |
| Chamber Ensemble | Small groups (quartet, quintet, etc.) |

**Typical Part Count**: 15-40+ individual parts per set

**Digital Consideration**: Each part becomes a separate digital file; complete set plus individual parts sold separately.

#### 4.1.2 Choral Music (Octavos)

**Description**: Music for vocal ensembles (choirs).

**Format**: Octavo—a tall, narrow booklet approximately 7" x 10.5"

**Voice Parts (SATB)**:
| Part | Description |
|------|-------------|
| Soprano | Highest female voice |
| Alto | Lower female voice |
| Tenor | Higher male voice |
| Bass | Lower male voice |

**Variations**: SAB (3-part), SSA (women's choir), TTBB (men's choir), unison

**Digital Consideration**: Single PDF per choral piece (unlike ensemble sets with multiple parts)

#### 4.1.3 Sheet Music (Sheets/Books)

**Description**: Music for solo instruments or small combinations.

**Subcategories**:
| Type | Description |
|------|-------------|
| Single Sheet | One song/piece |
| Folio | Collection of songs (songbook) |
| Method Book | Educational progressive instruction |
| Fake Book | Lead sheets with melody and chords |

**Formats**: Piano/vocal, guitar tablature, lead sheet, instrumental solo

**Digital Consideration**: May be sold as complete books or individual songs extracted from larger collections.

#### 4.1.4 Electronic Resources

**Description**: Non-standard digital products that don't fit traditional sheet music categories.

**Types**:
| Type | Description |
|------|-------------|
| Interactive PDF | PDF with embedded audio, video, or clickable elements |
| ZIP Package | Bundle of multiple files (PDFs, audio, video) |
| Play-Along | Sheet music with backing track audio |
| Curriculum Kit | Educational materials with multiple components |

**Digital Consideration**: Cannot be distributed through Legato; Alfred.com exclusive.

### 4.2 Product Relationships

```
Physical Product (in FileMaker)
       │
       ├── Has physical SKU (e.g., 47888)
       │
       └── Can generate Digital Products:
              │
              ├── Digital Set (00-PK-0047888)
              │      └── Contains: Score + All Parts
              │
              ├── Digital Score (00-PR-0047888)
              │      └── Conductor's score only
              │
              └── Digital Parts (00-PC-0047888_XX)
                     ├── 00-PC-0047888_F1 (Flute 1)
                     ├── 00-PC-0047888_F2 (Flute 2)
                     ├── 00-PC-0047888_CL1 (Clarinet 1)
                     └── ... (additional parts)
```

---

## 5. SKU System

### 5.1 SKU Fundamentals

**SKU** (Stock Keeping Unit): Unique identifier for each product in Alfred's catalog.

**Key Rule**: Any SKU containing a hyphen (-) is a digital SKU.

**Physical SKU Format**: Numeric only (e.g., `47888`)

**Digital SKU Format**: `00-XX-0000000` where XX is the product type code

### 5.2 Digital SKU Type Codes

| Code | Name | Description | Example |
|------|------|-------------|---------|
| PK | Pack/Set | Complete ensemble set (score + all parts) | 00-PK-0047888 |
| PR | Score | Conductor's score from ensemble set | 00-PR-0047888 |
| PC | Part | Individual instrument part | 00-PC-0047888_F1 |
| PO | Octavo | Choral octavo | 00-PO-0012345 |
| PS | Sheet | Standalone sheet music (current standard) | 00-PS-0098765 |
| PB | Book | Full book publication | 00-PB-0054321 |
| ER | Electronic Resource | ZIP files, interactive PDFs | 00-ER-0011111 |
| PIP | Sheet (legacy) | Legacy code, no longer used for new products | 00-PIP-000123 |
| PSG | Sheet (legacy) | Legacy code, no longer used for new products | 00-PSG-000123 |
| PSP | Sheet (legacy) | Legacy code, no longer used for new products | 00-PSP-000123 |
| PSL | Sheet (legacy) | Legacy code, no longer used for new products | 00-PSL-000123 |
| XIP | XML (legacy) | MusicXML file | 00-XIP-000123 |
| XS | XML (legacy) | MusicXML file | 00-XS-0000123 |
| XSP | XML (legacy) | MusicXML file | 00-XSP-000123 |
| VAP | Video/Audio Package | Multimedia with video, audio, and sheet music | 00-VAP-000123 |

### 5.3 Part Suffixes

For ensemble parts (PC SKUs), suffixes indicate the specific instrument:

| Suffix | Instrument |
|--------|------------|
| _F1, _F2 | Flute 1, Flute 2 |
| _OB | Oboe |
| _CL1, _CL2, _CL3 | Clarinet 1, 2, 3 |
| _BCL | Bass Clarinet |
| _BSN | Bassoon |
| _AS1, _AS2 | Alto Saxophone 1, 2 |
| _TS | Tenor Saxophone |
| _BS | Baritone Saxophone |
| _T1, _T2, _T3 | Trumpet 1, 2, 3 |
| _HN1, _HN2 | Horn 1, 2 |
| _TBN1, _TBN2, _TBN3 | Trombone 1, 2, 3 |
| _BTBN | Bass Trombone |
| _BAR | Baritone/Euphonium |
| _TU | Tuba |
| _PERC1, _PERC2 | Percussion 1, 2 |
| _TIMP | Timpani |
| _MLT | Mallet Percussion |
| _DB | Double Bass |
| _CC | Condensed/Conductor (special: sort order "1", page count "1") |

**Example Full SKU**: `00-PC-0047888_F1` = Flute 1 part for product 47888

### 5.4 SKU Relationships

```
Physical SKU: 47888
       │
       └── Digital SKUs derived:
              │
              ├── 00-PK-0047888 (Complete Set)
              ├── 00-PR-0047888 (Score)
              ├── 00-PC-0047888_CC (Condensed Score)
              ├── 00-PC-0047888_F1 (Flute 1)
              ├── 00-PC-0047888_F2 (Flute 2)
              ├── 00-PC-0047888_CL1 (Clarinet 1)
              ├── 00-PC-0047888_CL2 (Clarinet 2)
              ├── 00-PC-0047888_CL3 (Clarinet 3)
              └── ... (20+ additional parts)
```

---

## 6. Digitization Paths

Alfred Music has three distinct paths for creating digital products, determined by whether a physical product exists and the file format of the digital deliverable.

### 6.1 Path A: Physical-to-Digital

**Definition**: Converting an existing physical product to digital format.

**Trigger Conditions**:
- Physical product exists in FileMaker with complete metadata
- Physical product has inventory in AS400
- Decision made to offer digital version

**Characteristics**:
| Attribute | Value |
|-----------|-------|
| Physical product exists | Yes |
| FileMaker flag | `Outputtoweb = JJ114` |
| AS400 status | `CUR`, `TOP`, `NEW`, or `PND` |
| Dummy SKU required | No (real physical SKU exists) |
| Legato integration | Full (MRID assigned) |
| Distribution | Alfred.com + all Legato dealers |

**Asset Sources**:
- PDFs from print production archive
- Cover images from catalog assets
- Metadata from FileMaker and WebCRD

**This is the most common path** — converting existing print catalog to digital.

### 6.2 Path B: Digital-Only (with Dummy SKU)

**Definition**: Creating a digital product when no physical version exists or will be created.

**Trigger Conditions**:
- No physical product exists
- Product is standard PDF format
- Decision made to publish digital-only

**Characteristics**:
| Attribute | Value |
|-----------|-------|
| Physical product exists | No |
| Dummy SKU required | Yes |
| FileMaker flag | `Outputtoweb = DIGIONLY` |
| AS400 status | `PND` |
| Legato integration | Limited or none |
| Distribution | Alfred.com; dealers via Dropbox |

**Use Cases**:
- Niche arrangements not viable for print
- Test releases to gauge market demand
- Third-party publisher content
- Low-volume specialty products

**Key Constraint**: Despite being digital-only, the system requires creating a "dummy" physical SKU in FileMaker before the digital product can be created. This dummy exists solely to satisfy system requirements.

### 6.3 Path C: Alfred.com Exclusive (Electronic Resources)

**Definition**: Creating non-standard digital products that cannot be distributed through dealer systems.

**Trigger Conditions**:
- Product is ZIP file or Interactive PDF
- Dealers cannot process the file format
- Alfred.com-only distribution acceptable

**Characteristics**:
| Attribute | Value |
|-----------|-------|
| File format | ZIP or Interactive PDF |
| SKU type | ER (Electronic Resource) |
| Legato integration | None (incompatible format) |
| Distribution | Alfred.com only |
| Dealer access | Not available |

**Sub-Variants**:

| Variant | Description | Records Created |
|---------|-------------|-----------------|
| Standalone ER | No physical companion | 1 ER SKU → 1 Alfred.com record |
| ER with Companion | Has related physical product | 1 ER SKU + 1 Physical SKU → 2 Alfred.com records |

**Use Cases**:
- Interactive method books with embedded audio/video
- Curriculum kits with multiple file types
- Play-along packages (sheet music + backing tracks)
- Multimedia educational materials

### 6.4 Path Comparison Matrix

| Attribute | Path A | Path B | Path C |
|-----------|--------|--------|--------|
| Physical product exists | Yes | No | No (maybe companion) |
| Dummy SKU required | No | Yes | Depends |
| File format | Standard PDF | Standard PDF | ZIP / Interactive PDF |
| SKU type | PK/PR/PC/PO/PS/PB | PK/PR/PC/PO/PS/PB | ER |
| FileMaker flag | JJ114 | DIGIONLY | JJ114 |
| Legato integration | Full | Limited/None | None |
| MRID assigned | Yes | Maybe | No |
| Alfred.com | Yes | Yes | Yes |
| Dealer distribution | Yes (Legato) | Via Dropbox | No |
| Process complexity | High | High | Medium |
| Distribution reach | Maximum | Limited | Minimal |

---

## 7. Detailed Process Workflows

### 7.1 Path A Workflow: Physical-to-Digital

#### Step 1: Prerequisites Verification

**Systems Involved**: FileMaker, AS400, Royalty System

**Actions**:
1. Verify physical product exists in FileMaker with complete metadata
2. Confirm AS400 status is appropriate (CUR, TOP, NEW, or PND)
3. Set FileMaker flag: `Outputtoweb = JJ114`
4. Configure digital rights in Royalty System for the physical SKU

**Validation Points**:
- Product metadata is complete and accurate
- Pricing is set
- Rights holder information is on file
- Digital distribution is permitted by rights agreement

#### Step 2: Data Gathering

**Systems Involved**: WebCRD, FileMaker, Source PDFs

**Actions by Product Type**:

| Product Type | Data Source | Actions |
|--------------|-------------|---------|
| Performance Music | WebCRD | Download print job data |
| Sheets/Books | FileMaker | Batch search for physical SKUs |
| Choral | Source PDFs + TIFF Counter | Gather page counts and metadata |

**Data Elements Gathered**:
- Physical SKU
- Title and composer/arranger
- Instrumentation or voicing
- Page counts
- Preview page count (pages visible before purchase)
- UPC code
- Price

#### Step 3: Template Population

**Systems Involved**: Excel templates

**Actions**:
1. Select appropriate template based on product type:
   - Performance Music: `Performance_Template.xlsx`
   - Sheets/Books: `Sheets_Template.xls`
   - Choral: `Choral_Template.xls`

2. Manually enter data into template fields

**Validation Rules (Must Verify)**:
| Rule | Description |
|------|-------------|
| UPC Consistency | UPC must be identical across all parts, score, and set |
| Sort Order | Parts must be in correct performance order |
| _CC Handling | Condensed score must have sort order "1" and page count "1" |
| Orientation | Flag any landscape-oriented pages |

#### Step 4: Portal Upload and SKU Generation

**Systems Involved**: portal.alfred.com, SQL Server, AS400, Royalty System

**Actions**:
1. Upload completed template to portal.alfred.com
2. Select appropriate template type from dropdown
3. Submit for processing

**Automated Processing**:
1. SQL Server validates uploaded data
2. Digital SKU generated (e.g., 00-PK-0047888)
3. Metadata stored in SQL Server digital SKU table
4. SKU propagated to AS400
5. SKU propagated to Royalty System
6. Product data sent to Alfred.com

**Error Handling**:
- If import fails, diagnose error message
- Common fix: delete column AK from Excel file, resave as XLSX
- Re-upload corrected file

#### Step 5: Legato Integration (MRID Loop)

**Systems Involved**: Internal report server, Legato, portal.alfred.com, FTP

**Actions**:

1. **Download Legato file**:
   - URL: `alfredawssql06.alfredpub.com/reports/report/Digital_Reports/Legato`
   - Contains product data formatted for Legato

2. **Upload to Legato**:
   - URL: `app.legatomedia.com/publisher/vendors/index/`
   - Submit product data
   - Wait for Legato processing

3. **Download MRIDs**:
   - URL: `alfredawssql06.alfredpub.com/reports/report/Digital_Reports/Dealer%20Digital%20MRID%20Feed`
   - Download two versions:
     - Full version (complete MRID list)
     - 1-month prior version (delta)

4. **Upload MRIDs to Portal**:
   - URL: `portal.alfred.com/TSM/Digital/UpdateMRID.aspx`
   - Upload MRID files

5. **Upload to FTP for Dealers**:
   - Server: `ftp1.alfred.com`
   - Folder: "Dealerftp"
   - Files: `MR_Upload_Dealer_Full.txt`, `MR_Upload_Dealer.txt`

**Failure Handling**:
- If Legato upload fails, diagnose issues
- May require data correction and re-upload
- Loop until successful

#### Step 6: Asset Processing

**Systems Involved**: Source archive, Legato, AWS S3, Dropbox

**Actions**:

1. **Gather Source PDFs**:
   - Location: Print production archive (Dropbox)
   - Verify files exist for all parts

2. **Verify Asset Specifications**:
   | Requirement | Description |
   |-------------|-------------|
   | Format | PDF only |
   | Covers | Remove cover pages |
   | Fonts | Must be outlined (converted to shapes) |
   | Blank Pages | Remove trailing blank pages |

3. **Rename Files**:
   - Download filename mapping from report server
   - URL: `alfredawssql06.alfredpub.com/reports/report/Digital_Reports/Digital%20Uploads%20-%20Rename%20Files`
   - Rename each PDF to match digital SKU

4. **Upload Assets**:
   | Destination | Purpose |
   |-------------|---------|
   | Legato | Dealer distribution |
   | AWS S3 (`alfred-dsm-pdfs`) | Alfred.com customer downloads |
   | Dropbox | Archive |

5. **Cover Processing**:
   - Upload cover image to AWS (`alfred-catfiles`)

#### Step 7: Verification and Go-Live

**Actions**:
- Verify product appears on Alfred.com
- Verify product accessible through Legato dealers
- Confirm preview pages display correctly
- Test download functionality

### 7.2 Path B Workflow: Digital-Only

#### Step 1: Create Dummy SKU

**Systems Involved**: FileMaker, AS400, Royalty System

**Actions**:
1. Create new record in FileMaker
2. Enter minimal required metadata
3. Set flag: `Outputtoweb = DIGIONLY`
4. Create corresponding record in AS400:
   - Price = USD value
   - Territories = applicable regions
   - Status = PND
5. Configure digital rights in Royalty System for dummy SKU

**Key Difference from Path A**: Creating a placeholder record for a product that will never be printed.

#### Steps 2-6: Same as Path A

The remaining workflow follows Path A with these differences:
- Data comes from source files rather than print archives
- No WebCRD data available
- Legato integration may be limited or skipped
- Dealers may access via Dropbox instead of Legato

#### Distribution Outcome

- Alfred.com: Full availability
- Dealers: Dropbox access (not Legato integration)

### 7.3 Path C Workflow: Electronic Resources

#### Step 1: Determine Sub-Variant

**Decision Point**: Does a related physical product exist?

| Scenario | Action |
|----------|--------|
| Standalone ER | Create ER SKU only |
| ER with Physical Companion | Create ER SKU + Physical/Dummy SKU |

#### Step 2: Create ER SKU

**Systems Involved**: FileMaker, AS400

**Actions**:
1. Create ER record in FileMaker
2. Mark with flag `JJ114`
3. Set up in AS400 as needed

#### Step 3: Asset Processing

**Systems Involved**: ProdAssets table, AWS S3

**Actions**:
1. Prepare ZIP file or Interactive PDF
2. Link file to SKU in ProdAssets table
3. Upload to AWS S3

**Note**: No Legato upload (incompatible format)

#### Step 4: Go-Live

**Outcome**:
- Product available on Alfred.com only
- No dealer distribution
- 1 or 2 Alfred.com records depending on sub-variant

---

## 8. Data Flows

### 8.1 SKU Creation Data Flow

```
FileMaker (Physical/Dummy SKU)
       │
       │ Manual: Flag set (JJ114 or DIGIONLY)
       ▼
Excel Template (Manual data entry)
       │
       │ Manual: Upload to portal
       ▼
portal.alfred.com
       │
       │ Automated: Validation + Processing
       ▼
SQL Server
       │
       │ Automated: Generate Digital SKU
       │
       ├──────────────────────────────────────┐
       │                                      │
       ▼                                      ▼
    AS400                              Royalty System
(Digital SKU registered)           (Digital SKU registered)
       │                                      │
       │                                      │
       └──────────────┬───────────────────────┘
                      │
                      ▼
                 Alfred.com
            (Product available)
```

### 8.2 Legato MRID Data Flow

```
SQL Server (Digital products ready)
       │
       │ Report generation
       ▼
Internal Report Server
(alfredawssql06.alfredpub.com)
       │
       │ Manual: Download Legato file
       ▼
Staff Workstation
       │
       │ Manual: Upload to Legato
       ▼
Legato (app.legatomedia.com)
       │
       │ Legato processing (assigns MRID)
       ▼
Staff Workstation
       │
       │ Manual: Download MRID files
       │
       ├─────────────────────────────────┐
       │                                 │
       │ Manual: Upload                  │ Manual: Upload
       ▼                                 ▼
portal.alfred.com                 FTP Server (ftp1.alfred.com)
       │                                 │
       │                                 │
       ▼                                 ▼
   SQL Server                      Integrated Dealers
(MRID stored)                   (Pepper, Stanton's, etc.)
```

### 8.3 Asset Distribution Data Flow

```
Source PDFs (Print Archive / Dropbox)
       │
       │ Manual: Gather files
       ▼
Staff Workstation
       │
       │ Manual: Verify specs (fonts, pages, covers)
       │
       │ Manual: Rename to match SKUs
       │
       ├────────────────────────────────────────────┐
       │                    │                       │
       │ Manual upload      │ Manual upload         │ Manual upload
       ▼                    ▼                       ▼
    Legato              AWS S3                  Dropbox
(Dealer distribution)  (alfred-dsm-pdfs)       (Archive)
       │                    │
       │                    │
       ▼                    ▼
   Dealers             Alfred.com
(via Legato)         (Customer downloads)
```

### 8.4 Complete System Integration Map

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DATA SOURCES                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│   WebCRD          FileMaker         Source PDFs         Royalty System      │
│   (Print data)    (Product data)    (Assets)            (Rights data)       │
└───────┬───────────────┬─────────────────┬─────────────────────┬─────────────┘
        │               │                 │                     │
        └───────────────┼─────────────────┘                     │
                        ▼                                       │
              ┌─────────────────┐                               │
              │  Excel Template │                               │
              │  (Manual entry) │                               │
              └────────┬────────┘                               │
                       │                                        │
                       ▼                                        │
              ┌─────────────────┐                               │
              │ portal.alfred   │                               │
              │     .com        │◄──────────────────────────────┘
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │   SQL Server    │
              │ (Digital SKUs)  │
              └────────┬────────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   ┌─────────┐   ┌──────────┐   ┌──────────┐
   │  AS400  │   │ Royalty  │   │Alfred.com│
   │         │   │  System  │   │          │
   └─────────┘   └──────────┘   └──────────┘

              ┌─────────────────┐
              │  Report Server  │
              │(alfredawssql06) │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │     Legato      │
              │  (JW Pepper)    │
              └────────┬────────┘
                       │
        ┌──────────────┴──────────────┐
        ▼                             ▼
   ┌─────────┐                   ┌─────────┐
   │   FTP   │                   │ Dealers │
   │ Server  │                   │         │
   └─────────┘                   └─────────┘

              ┌─────────────────┐
              │   Source PDFs   │
              └────────┬────────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   ┌─────────┐   ┌──────────┐   ┌──────────┐
   │  Legato │   │  AWS S3  │   │ Dropbox  │
   │ (Assets)│   │          │   │(Archive) │
   └─────────┘   └──────────┘   └──────────┘
```

---

## 9. Architectural Constraints

### 9.1 Physical-First Dependency

**Constraint**: Digital products cannot exist without a physical product (or dummy) existing first.

**Manifestations**:
- FileMaker requires a physical SKU record before digital processing
- Digital SKU naming derives from physical SKU numbers
- Royalty System attaches rights to physical/dummy SKUs
- AS400 expects product records even for digital-only items

**Impact**:
- Extra work creating dummy SKUs for digital-only products
- Conceptual confusion (fake physical products in the system)
- Cannot do true "digital-first" publishing

**Root Cause**: Systems designed when physical was the only format; digital added as extension.

### 9.2 Template Fragmentation

**Constraint**: Three separate Excel templates exist for different product types.

**Templates**:
| Template | Product Type | Unique Fields |
|----------|--------------|---------------|
| Performance_Template.xlsx | Ensemble music | Part suffixes, score/parts relationships |
| Sheets_Template.xls | Sheet music, books | imSKU, imFilename, imPages, imPreview |
| Choral_Template.xls | Choral octavos | Voice part specifications |

**Impact**:
- Staff must know which template to use
- Validation rules differ per template
- No single unified data entry process
- Training complexity increased

### 9.3 External System Dependency (Legato)

**Constraint**: Dealer distribution depends on Legato, a system owned by JW Pepper.

**Characteristics**:
- Web portal interface only (no documented API)
- Upload/download cycle required for MRID acquisition
- Cannot handle non-standard file formats (ZIP, interactive PDF)
- Processing time outside Alfred's control

**Impact**:
- Latency added by upload/wait/download cycle
- Manual intervention required for data exchange
- File format limitations restrict product types
- Dependency on third-party system availability

### 9.4 Multi-Destination Asset Distribution

**Constraint**: Digital assets must be uploaded to multiple locations separately.

**Destinations**:
| Destination | Purpose | Manual Upload Required |
|-------------|---------|------------------------|
| Legato | Dealer distribution | Yes |
| AWS S3 | Alfred.com downloads | Yes |
| Dropbox | Archive + non-Legato dealers | Yes |
| FTP | Integrated dealer MRID files | Yes |

**Impact**:
- Same file uploaded 3+ times
- Error risk (forgetting a destination, uploading wrong file)
- Time consumed by repetitive uploads
- No single "publish" action

### 9.5 Manual Data Entry Dependency

**Constraint**: Data must be manually entered into templates despite existing in source systems.

**Data Re-Entry Points**:
- WebCRD data → manually typed into template
- FileMaker data → manually typed into template
- Page counts → manually counted and entered
- Filenames → manually looked up and entered

**Impact**:
- High labor cost per product
- Error risk from manual transcription
- Latency from human processing time
- Inconsistent data quality

### 9.6 Validation Gap

**Constraint**: Validation is manual and happens late in the process.

**Current Validation**:
- Template validation: manual checklist
- UPC consistency: manual verification
- Sort order: manual verification
- Portal upload: automated but limited error messages

**Impact**:
- Errors discovered late in process
- Rework required when issues found at upload
- No early warning system
- Quality dependent on individual diligence

---

## 10. Known Pain Points

### 10.1 Manual Steps Inventory

| # | Manual Step | Frequency | Automation Potential |
|---|-------------|-----------|---------------------|
| 1 | Download WebCRD data | Per product | High |
| 2 | Batch search FileMaker for SKUs | Per batch | High |
| 3 | Manual data entry into template | Per product | High |
| 4 | Validate UPC, sort order, page counts | Per product | High |
| 5 | Upload template to portal | Per batch | Medium |
| 6 | Diagnose import failures, fix column AK | On failure | High |
| 7 | Download Legato file from report server | Per batch | High |
| 8 | Upload to Legato portal | Per batch | Medium |
| 9 | Download MRIDs (2 versions) | Per batch | High |
| 10 | Upload MRIDs to portal + FTP | Per batch | High |
| 11 | Rename PDF files to match SKUs | Per product | High |
| 12 | Upload assets to Legato, AWS, Dropbox | Per batch | High |

### 10.2 Common Errors

| Error | Description | Frequency | Impact |
|-------|-------------|-----------|--------|
| Column AK issue | Extra column causes portal import failure | Common | Re-work required |
| UPC mismatch | Parts have different UPCs than set | Occasional | Data inconsistency |
| Sort order wrong | Parts in incorrect performance order | Occasional | Poor customer experience |
| Missing pages | Page count doesn't match actual PDF | Occasional | Customer complaints |
| Wrong filename | File renamed incorrectly | Occasional | Download failures |
| Legato rejection | Data format not accepted | Occasional | Delays, diagnosis needed |

### 10.3 Latency Sources

| Source | Estimated Impact | Notes |
|--------|------------------|-------|
| Manual data entry | Hours | Varies by product complexity |
| Template validation | Minutes-Hours | Depends on error count |
| Portal processing | Minutes | Automated |
| Legato round-trip | Hours-Days | External system, variable |
| Asset uploads (3x) | Minutes-Hours | Depends on file sizes |
| Approval waits | Variable | Not fully documented |

### 10.4 Data Duplication

| Data Element | Source | Duplicated To |
|--------------|--------|---------------|
| Product metadata | FileMaker | SQL Server, Excel template |
| SKU | FileMaker | AS400, SQL Server, Royalty System |
| MRID | Legato | Portal, FTP, SQL Server |
| PDF files | Source archive | Legato, AWS, Dropbox |

---

## 11. Glossary

### 11.1 Music Industry Terms

| Term | Definition |
|------|------------|
| Arranger | Person who adapts a musical composition for different instruments or ensembles |
| Composer | Person who wrote the original musical composition |
| Concert Band | Wind and percussion ensemble typically found in schools (woodwinds, brass, percussion) |
| Conductor | Person who directs a musical ensemble using gestures (typically with a baton) |
| Ensemble | Group of musicians performing together |
| Fake Book | Collection of lead sheets with melody and chord symbols |
| Folio | Collection of songs bound together (songbook) |
| Instrumentation | The specific instruments required to perform a piece |
| Jazz Band | Ensemble featuring saxophones, trumpets, trombones, and rhythm section |
| Lead Sheet | Single-page music showing melody, lyrics, and chord symbols |
| Marching Band | Mobile ensemble performing at sporting events and parades |
| Method Book | Educational publication teaching instrumental or vocal technique progressively |
| Octavo | Standard format for choral music; tall narrow booklet approximately 7" x 10.5" |
| Orchestra | Large ensemble including string, woodwind, brass, and percussion sections |
| Part | Individual instrument's music extracted from a larger work |
| Play-Along | Product combining sheet music with audio backing tracks for practice |
| SATB | Soprano, Alto, Tenor, Bass — the four standard voice parts in choral music |
| Score | Master view showing all instrument parts simultaneously; used by conductor |
| Set | Complete package of ensemble music (score + all parts) |
| Sheet Music | Printed or digital music notation for performance |
| String Orchestra | Ensemble of string instruments only (violin, viola, cello, bass) |
| Tablature | Music notation showing finger positions rather than standard notation (common for guitar) |
| Voicing | The specific voice parts required (e.g., SATB, SSA, TTBB) |

### 11.2 System and Technical Terms

| Term | Definition |
|------|------------|
| AS400 | IBM legacy system used for inventory and sales tracking |
| AWS S3 | Amazon Web Services Simple Storage Service; cloud file storage |
| Dummy SKU | Placeholder physical product record created to enable digital-only products |
| FileMaker Pro | Desktop database application; source of truth for physical product metadata |
| FTP | File Transfer Protocol; method for transferring files to servers |
| Legato | External distribution platform owned by JW Pepper for dealer integration |
| MRID | Legato's unique product identifier required for dealer distribution |
| Outputtoweb | FileMaker flag indicating digital eligibility |
| Portal | Internal web application (portal.alfred.com) for template uploads and workflow |
| ProdAssets | Database table mapping SKUs to their associated digital files |
| Royalty System | Database tracking composition rights, permitted usage, and royalty payments |
| SKU | Stock Keeping Unit; unique product identifier |
| SQL Server | Microsoft database storing digital product metadata |
| UPC | Universal Product Code; barcode identifier for retail sales |
| WebCRD | Print production management system by RSA |

### 11.3 SKU Type Codes

| Code | Full Name | Description |
|------|-----------|-------------|
| PK | Pack | Complete ensemble set (score + all parts) |
| PR | Score | Conductor's score from ensemble set |
| PC | Part | Individual instrument part |
| PO | Octavo | Choral octavo |
| PS | Sheet | Standalone sheet music |
| PB | Book | Full book publication |
| ER | Electronic Resource | ZIP files, interactive PDFs |
| PIP | Legacy Sheet | Legacy code, no longer used |
| PSG | Legacy Sheet | Legacy code, no longer used |
| PSP | Legacy Sheet | Legacy code, no longer used |
| PSL | Legacy Sheet | Legacy code, no longer used |
| XIP | XML | MusicXML file (legacy) |
| XS | XML | MusicXML file (legacy) |
| XSP | XML | MusicXML file (legacy) |
| VAP | Video/Audio Package | Multimedia product |

### 11.4 Status Codes and Flags

| Code/Flag | System | Meaning |
|-----------|--------|---------|
| JJ114 | FileMaker | Approved for digital conversion |
| DIGIONLY | FileMaker | Digital-only product (requires dummy SKU) |
| CUR | AS400 | Current — active product |
| TOP | AS400 | Top seller |
| NEW | AS400 | New release |
| PND | AS400 | Pending — not yet fully active |

---

## 12. Reference Tables

### 12.1 System Access Points

| System | Type | Access URL/Method |
|--------|------|-------------------|
| FileMaker Pro | Desktop App | Local installation |
| AS400 | Terminal | Internal terminal access |
| SQL Server | Database | Internal connection |
| Royalty System | Application | Internal application |
| portal.alfred.com | Web App | https://portal.alfred.com |
| Legato (data) | Web App | https://app.legatomedia.com/publisher/vendors/index/ |
| Legato (assets) | Web App | https://app.legatomedia.com/publisher/assets/uploadWizard/ |
| Report Server | Web | http://alfredawssql06.alfredpub.com/reports/ |
| AWS S3 | Cloud Storage | s3://alfred-dsm-pdfs, s3://alfred-catfiles |
| Dropbox | Cloud Storage | Shared folders |
| FTP | File Server | ftp1.alfred.com |
| Alfred.com | E-commerce | https://www.alfred.com |

### 12.2 Report URLs

| Report | URL |
|--------|-----|
| Legato Export | http://alfredawssql06.alfredpub.com/reports/report/Digital_Reports/Legato |
| MRID Feed | http://alfredawssql06.alfredpub.com/reports/report/Digital_Reports/Dealer%20Digital%20MRID%20Feed |
| Rename Files | http://alfredawssql06.alfredpub.com/reports/report/Digital_Reports/Digital%20Uploads%20-%20Rename%20Files |

### 12.3 Template Downloads

| Template | URL |
|----------|-----|
| Performance Music | https://portal.alfred.com/DigitalTemplates/Performance_Template.xlsx |
| Sheets/Books | https://portal.alfred.com/DigitalTemplates/Sheets_Template.xls |
| Choral | https://portal.alfred.com/DigitalTemplates/Choral_Template.xls |

### 12.4 Path Decision Matrix

```
START
  │
  ▼
Does physical product exist?
  │
  ├── YES ──► Path A: Physical-to-Digital
  │           • Full Legato integration
  │           • Maximum distribution
  │
  └── NO
       │
       ▼
     Is file format standard PDF?
       │
       ├── YES ──► Path B: Digital-Only
       │           • Requires dummy SKU
       │           • Limited Legato integration
       │           • Dealers via Dropbox
       │
       └── NO (ZIP or Interactive PDF)
                  │
                  ▼
                Path C: Alfred.com Exclusive
                  • ER SKU type
                  • No Legato integration
                  • No dealer distribution
```

### 12.5 Asset Specification Requirements

| Requirement | Specification |
|-------------|---------------|
| File Format | PDF only |
| Cover Pages | Remove (not included in digital) |
| Fonts | Must be outlined (converted to shapes) |
| Blank Pages | Remove trailing blank pages |
| Orientation | Flag landscape pages |
| File Naming | Must match digital SKU exactly |

### 12.6 Ensemble Part Suffix Reference

| Instrument Family | Suffix Examples |
|-------------------|-----------------|
| Flutes | _F1, _F2, _PICC |
| Oboes | _OB, _OB2, _EH |
| Clarinets | _CL1, _CL2, _CL3, _BCL, _CBCL |
| Bassoons | _BSN, _BSN2, _CBSN |
| Saxophones | _AS1, _AS2, _TS, _BS |
| Trumpets | _T1, _T2, _T3 |
| Horns | _HN1, _HN2, _HN3, _HN4 |
| Trombones | _TBN1, _TBN2, _TBN3, _BTBN |
| Low Brass | _BAR, _TU |
| Percussion | _PERC1, _PERC2, _TIMP, _MLT |
| Strings | _VN1, _VN2, _VA, _VC, _DB |
| Special | _CC (condensed), _PNO (piano) |

---

## Document Metadata

| Attribute | Value |
|-----------|-------|
| Subject | Alfred Music Digitization Process |
| Version | 1.0 |
| Created | January 2026 |
| Purpose | AI LLM Knowledge Base |
| Scope | Complete process documentation |
| Sources | 10 internal Alfred Music documents |

---

*End of Document*
