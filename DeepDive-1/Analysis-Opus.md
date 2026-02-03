# AX3 Music Digital Transformation: As-Is Analysis

**Document Version:** 1.0  
**Date:** February 3, 2026  
**Purpose:** Comprehensive analysis of AX3's current digitization workflow, open questions, and observations for digital transformation initiative.

---

## Table of Contents

1. [Current Assumptions](#1-current-assumptions)
2. [Current Workflow (As-Is)](#2-current-workflow-as-is)
3. [Open Questions for AX3](#3-open-questions-for-ax3)
4. [Glossary](#4-glossary)

---

## 1. Current Assumptions

This section lists assumptions derived from the documentation that would benefit from explicit AX3 confirmation.

### 1.1 System Architecture Assumptions

| ID | Assumption | Rationale | Confidence |
|----|------------|-----------|------------|
| A1 | **FileMaker and AS400 are not directly integrated.** FileMaker reads from AS400, but AS400 does not reference FileMaker. | Explicitly stated in Q&A Section 2, Question 6: "AS400 and FileMaker are not tied together in any way other than FileMaker reads data from the AS400." | Confirmed |
| A2 | **FileMaker data is exported nightly to SQL Server via CSV.** This is a one-way batch process, not real-time sync. | Stated in Meeting Notes and Q&A Section 5, Question 20. The exact job name and owner are unknown. | Confirmed |
| A3 | **AS400 writes to the Royalty System overnight.** Records with Royalty=Y are copied. | Stated in Meeting Notes and Q&A Section 5, Question 20. | Confirmed |
| A4 | **Portal.ax3.com writes to both SQL Server and AS400 directly.** | Confirmed in Q&A Section 5, Question 20: "Portal.ax3.com writes to the SQL Server, and the AS400." | Confirmed |
| A5 | **The portal validates against AS400, Royalty System, and FileMaker (via nightly SQL download).** | Q&A Section 5, Question 18 confirms validation occurs but details are unclear. | Likely |

### 1.2 SKU and Data Model Assumptions

| ID | Assumption | Rationale | Confidence |
|----|------------|-----------|------------|
| A6 | **Digital SKU numbers are serialized and not derived from physical SKU numbers.** The numeric portion (e.g., "0047888") is a sequence number assigned by SQL Server at creation time. | Q&A Section 3, Question 11: "It's serialized. SQL picks up the next sequence when another batch of products are converted." | Confirmed |
| A7 | **The Ref_SKU field in SQL Server links digital products to their parent physical/dummy SKU.** This is the only enforced relationship. | Q&A Section 1, Question 3 confirms this. | Confirmed |
| A8 | **Dummy SKUs require nearly identical data to physical SKUs.** The only differences are `OutputtoWeb` (DIGIONLY vs JJ114) and `DigitalOnlyYN` (Y vs N). | Q&A Section 1, Question 2 provides this comparison. | Confirmed |
| A9 | **All sales transactions (digital and physical) are recorded in AS400.** There is no alternative sales recording system. | Q&A Section 2, Questions 7-8 confirm AS400 is required for all sales. | Confirmed |
| A10 | **Royalties for digital SKUs are programmed separately from physical SKUs.** The digital SKU is not automatically inheriting royalty rates from its parent. | Q&A Section 7, Question 26: "the digital sku is programmed separately, it is not related to the physical sku." | Confirmed |

### 1.3 Workflow Assumptions

| ID | Assumption | Rationale | Confidence |
|----|------------|-----------|------------|
| A11 | **Performance Music and Choral products require a physical/dummy SKU because portal.ax3.com pulls metadata from the Ref_SKU.** Books and Sheets can theoretically work without one, but website logic may have issues. | Q&A Section 9, Question 30: "We cannot upload without a refsku/dummy sku for Performance or Choral music because portal.ax3.com is pulling data based on the refsku/dummy sku." | Confirmed |
| A12 | **The Legato process is currently required for all digital products** (not just those distributed via dealers). Bypassing Legato is convoluted and manual. | Q&A Section on Legato, Question 37 confirms this. | Confirmed |
| A13 | **Legato does not have an API.** All interactions with Legato are via web portal (manual upload/download). | Q&A Path A, Question 33: "Legato (app.legatomedia.com) does not have an API to tie into our systems as of today." | Confirmed |
| A14 | **The AS400 job number must have a "###95" equivalent** for the portal to process digital SKUs for a product. | Q&A Section 2, Question 6 mentions this requirement, though the explanation is incomplete. | Likely |
| A15 | **There are approximately 500 digital-only products with dummy SKUs currently in the system.** | Q&A Section 8, Question 28. | Confirmed |

### 1.4 Distribution and Sales Assumptions

| ID | Assumption | Rationale | Confidence |
|----|------------|-----------|------------|
| A16 | **Legato only requires the digital SKU, not the physical/dummy SKU.** The dummy SKU is purely an internal AX3 construct. | Q&A Section 6, Question 24: "Nothing would break. Legato only cares about the digital sku." | Confirmed |
| A17 | **AX3-only sales (bypassing Legato) is a desired capability but not currently well-supported.** The team has identified specific fields that need to be self-serviceable to enable this. | Q&A "Bypassing Legato" section details requirements. | Confirmed |
| A18 | **ER (Electronic Resource) products bypass Legato entirely and are sold only on ax3.com.** Only 5 ER products exist today. | Q&A Path C, Questions 39-41. | Confirmed |

### 1.5 Organizational Assumptions

| ID | Assumption | Rationale | Confidence |
|----|------------|-----------|------------|
| A19 | **The portal code is maintained by internal staff (Lee), but there is reluctance to make changes due to risk.** Original developers (Doug Fraser, Chris Rubeiz) are no longer actively maintaining it. | Q&A Section 5, Question 21. | Confirmed |
| A20 | **Travis (Digital Production team) is a key bottleneck/checkpoint in the workflow.** He manually checks data, receives files via Slack, and renames files. | Meeting Notes and multiple Q&A references. | Likely |
| A21 | **The Purchasing Team creates AS400 records first (using sequential numbering), before any other system has data.** | Meeting Notes: "the Purchasing team creates a new product or item on AS400, using a sequential number approach." | Likely |

---

## 2. Current Workflow (As-Is)

This section reconstructs AX3's current operational workflow at quasi-operational detail.

### 2.1 High-Level Workflow Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        PRODUCT CREATION FLOW                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐              │
│  │ Purchasing   │───▶│  Editorial   │───▶│   Digital    │              │
│  │ Team         │    │  Team        │    │  Production  │              │
│  │ (AS400)      │    │ (FileMaker)  │    │  (Travis)    │              │
│  └──────────────┘    └──────────────┘    └──────────────┘              │
│        │                   │                    │                       │
│        │ Nightly           │ Nightly            │                       │
│        ▼ Sync              ▼ Sync               │                       │
│  ┌──────────────┐    ┌──────────────┐          │                       │
│  │   Royalty    │    │  SQL Server  │◀─────────┘                       │
│  │   System     │    │  (tblapro-   │     Template                     │
│  │              │    │   ducts)     │     Upload                       │
│  └──────────────┘    └──────────────┘                                  │
│                             │                                          │
│                             │ Generates                                │
│                             ▼ Digital SKUs                             │
│                      ┌──────────────┐                                  │
│                      │    Portal    │                                  │
│                      │ (ax3.com)    │                                  │
│                      └──────────────┘                                  │
│                             │                                          │
│              ┌──────────────┼──────────────┐                          │
│              ▼              ▼              ▼                           │
│        ┌─────────┐   ┌──────────┐   ┌─────────┐                       │
│        │ Legato  │   │  AWS S3  │   │  AS400  │                       │
│        │         │   │          │   │ (SKUs)  │                       │
│        └─────────┘   └──────────┘   └─────────┘                       │
│              │              │                                          │
│              ▼              ▼                                          │
│        ┌─────────┐   ┌──────────┐                                     │
│        │ Dealers │   │ ax3.com  │                                     │
│        └─────────┘   │ Website  │                                     │
│                      └──────────┘                                     │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Detailed Workflow by Phase

#### Phase 0: Product Acquisition and Initial Setup

**Actor:** Purchasing Team  
**System:** AS400  
**Trigger:** Decision to acquire new music IP

| Step | Action | System | Artifact Created | Notes |
|------|--------|--------|------------------|-------|
| 0.1 | Create new product record | AS400 | Physical SKU (e.g., 00-47888) | Sequential number assigned |
| 0.2 | Set Royalty flag = Y | AS400 | — | Enables nightly sync to Royalty System |
| 0.3 | Set Product Type Code | AS400 | — | B=Set, C=Choral, etc. |
| 0.4 | Set Price, Territory, Status | AS400 | — | Status varies by product type |

**Nightly Batch:** AS400 syncs records with Royalty=Y to Royalty System.

---

#### Phase 1: Metadata Entry

**Actor:** Editorial Team  
**System:** FileMaker Pro  
**Trigger:** AS400 record exists

| Step | Action | System | Artifact Created | Notes |
|------|--------|--------|------------------|-------|
| 1.1 | Create Product record | FileMaker | Product table record | Main metadata table |
| 1.2 | Create ContributorRole records | FileMaker | Contributor records | Composer, arranger, etc. |
| 1.3 | Create TOCSongSources records | FileMaker | Song listing records | Only for multi-song products |
| 1.4 | Populate ~100 required fields | FileMaker | — | See Q&A Section 36 for field list |
| 1.5 | Set OutputtoWeb flag | FileMaker | — | JJ114 (physical+digital) or DIGIONLY |

**Nightly Batch:** FileMaker exports to SQL Server tables (tblapproducts, "pending" table) via CSV.

**Key Difference for Digital-Only Products:**
- Must create a "dummy" physical SKU in AS400 (Step 0.1) even though no physical product will exist
- FileMaker field `DigitalOnlyYN` = Y
- FileMaker field `OutputtoWeb` = DIGIONLY
- AS400 Status = PND (not CUR)

---

#### Phase 2: Data Quality Review

**Actor:** Travis (Digital Production)  
**Systems:** FileMaker, AS400  
**Trigger:** Editorial team completes data entry

| Step | Action | System | Artifact Created | Notes |
|------|--------|--------|------------------|-------|
| 2.1 | Review metadata completeness | FileMaker | — | Manual inspection |
| 2.2 | Fix data issues | FileMaker | — | Corrections as needed |
| 2.3 | Verify AS400 configuration | AS400 | — | Job number "###95" requirement |
| 2.4 | Signal completion | — | — | Mechanism unclear |

**Uncertainty:** How does Travis signal completion? What specific fields or status changes indicate readiness?

---

#### Phase 3: Template Preparation

**Actor:** Travis (Digital Production)  
**Systems:** FileMaker, WebCRD, Excel  
**Trigger:** Phase 2 complete

The workflow differs by product type:

##### 3A. Performance Music (Sets)

| Step | Action | System | Artifact Created | Notes |
|------|--------|--------|------------------|-------|
| 3A.1 | Download print job data | WebCRD | — | Source of instrumentation data |
| 3A.2 | Export product data | FileMaker | — | Batch search/export |
| 3A.3 | Populate Performance_Template.xlsx | Excel | Completed template | One row per part + set + score |
| 3A.4 | Validate UPC consistency | Excel | — | All parts must share UPC |
| 3A.5 | Validate sort order | Excel | — | Correct performance sequence |
| 3A.6 | Set _CC part handling | Excel | — | Sort order=1, page count=1 |

##### 3B. Choral Music (Octavos)

| Step | Action | System | Artifact Created | Notes |
|------|--------|--------|------------------|-------|
| 3B.1 | Count pages from PDF | Source PDF | — | TIFF counter mentioned |
| 3B.2 | Export product data | FileMaker | — | — |
| 3B.3 | Populate Choral_Template.xls | Excel | Completed template | One row per octavo |

##### 3C. Books and Sheets

| Step | Action | System | Artifact Created | Notes |
|------|--------|--------|------------------|-------|
| 3C.1 | Batch search for SKUs | FileMaker | Export | — |
| 3C.2 | Populate Sheets_Template.xls | Excel | Completed template | Uses FileMaker script for formatting |

**Key Manual Steps:**
- All template population is manual data entry
- No automated validation before upload
- Templates are not retained after upload

---

#### Phase 4: Portal Upload and SKU Generation

**Actor:** Travis (Digital Production)  
**Systems:** Portal (portal.ax3.com), SQL Server, AS400  
**Trigger:** Template complete

| Step | Action | System | Artifact Created | Notes |
|------|--------|--------|------------------|-------|
| 4.1 | Navigate to portal.ax3.com | Portal | — | — |
| 4.2 | Select template type | Portal | — | Dropdown selection |
| 4.3 | Upload completed template | Portal | — | — |
| 4.4 | Diagnose errors if upload fails | Portal | — | Column AK issue common |
| 4.4a | (If failed) Delete column AK | Excel | Fixed template | Known workaround |
| 4.4b | (If failed) Re-save as XLSX | Excel | Fixed template | — |
| 4.4c | (If failed) Re-upload | Portal | — | — |
| 4.5 | SQL Server processes upload | SQL Server | — | Automated |
| 4.6 | Digital SKUs generated | SQL Server | Digital SKUs (00-PK-xxx, etc.) | Serialized numbering |
| 4.7 | SKUs propagated to AS400 | AS400 | Digital SKU records | Via Portal |
| 4.8 | SKUs propagated to Royalty System | Royalty System | Digital SKU records | Via nightly AS400 sync |

**Digital SKUs Generated for Performance Set:**
- 1 × PK (Pack/Set): e.g., 00-PK-0047888
- 1 × PR (Score): e.g., 00-PR-0047888
- N × PC (Parts): e.g., 00-PC-0047888_F1, 00-PC-0047888_CL1, etc.

---

#### Phase 5: Legato Integration (MRID Loop)

**Actor:** Travis (Digital Production)  
**Systems:** Report Server, Legato, Portal, FTP  
**Trigger:** Phase 4 complete (SKUs exist)

| Step | Action | System | Artifact Created | Notes |
|------|--------|--------|------------------|-------|
| 5.1 | Download Legato export file | Report Server | Legato data file | alfredawssql06 server |
| 5.2 | Remove column AK (if present) | Excel | Cleaned file | Manual workaround |
| 5.3 | Upload to Legato | Legato Portal | — | app.legatomedia.com |
| 5.4 | Wait for Legato processing | Legato | MRID assigned | External system |
| 5.5 | Download MRID files | Report Server | Full + Delta files | Two versions |
| 5.6 | Upload MRIDs to portal | Portal | — | portal.ax3.com/TSM/Digital/UpdateMRID.aspx |
| 5.7 | Upload MRID files to FTP | FTP | Dealer files | ftp1.alfred.com/Dealerftp |

**MRID Files:**
- MR_Upload_Dealer_Full.txt (complete list)
- MR_Upload_Dealer.txt (monthly delta)

**Timeline:** MRID round-trip takes approximately 5-10 minutes (Q&A Path A, Question 33)

---

#### Phase 6: Asset Processing

**Actor:** Travis (Digital Production)  
**Systems:** Source Archive (Dropbox/AWS), Legato, AWS S3  
**Trigger:** MRIDs acquired

##### 6A. Gather Source PDFs

| Step | Action | System | Artifact Created | Notes |
|------|--------|--------|------------------|-------|
| 6A.1 | Physical-digital: Download from AWS | AWS | Source PDFs | Print production archive |
| 6A.2 | Digital-only: Receive via Slack | Slack | Source PDFs | From Editorial team |

##### 6B. Process PDFs

| Step | Action | System | Artifact Created | Notes |
|------|--------|--------|------------------|-------|
| 6B.1 | Verify fonts outlined | — | — | Manual check |
| 6B.2 | Remove cover pages | — | — | Not included in digital |
| 6B.3 | Remove trailing blank pages | — | — | — |
| 6B.4 | Flag landscape pages | — | — | For metadata |

##### 6C. Rename and Upload

| Step | Action | System | Artifact Created | Notes |
|------|--------|--------|------------------|-------|
| 6C.1 | Download filename mapping | Report Server | Rename mapping | Digital Uploads - Rename Files report |
| 6C.2 | Rename PDFs to match SKUs | — | Renamed files | Must match exactly |
| 6C.3 | Upload to Legato | Legato | — | Dealer distribution |
| 6C.4 | Upload to AWS S3 | AWS S3 | — | alfred-dsm-pdfs bucket |
| 6C.5 | Upload to Dropbox | Dropbox | — | Archive |
| 6C.6 | Upload cover image | AWS S3 | — | alfred-catfiles bucket |

**Key Issue:** Same file uploaded to 3 destinations manually.

---

#### Phase 7: Go-Live and Verification

**Actor:** Product Manager (confirmation unclear)  
**Systems:** ax3.com, Legato dealers  
**Trigger:** Phase 6 complete

| Step | Action | System | Artifact Created | Notes |
|------|--------|--------|------------------|-------|
| 7.1 | Verify product on ax3.com | ax3.com | — | — |
| 7.2 | Verify Legato dealer access | — | — | — |
| 7.3 | Test preview pages | ax3.com | — | — |
| 7.4 | Test download | ax3.com | — | — |

---

### 2.3 Workflow Variant: Electronic Resources (ER) — Path C

**Distinct from standard flow.** Used for ZIP files, interactive PDFs.

| Step | Action | System | Notes |
|------|--------|--------|-------|
| C.1 | Create AS400 record for ER version | AS400 | ER item number prefix |
| C.2 | Create FileMaker record for ER version | FileMaker | Standard metadata flow |
| C.3 | Set OutputtoWeb = JJ114 | FileMaker | Go-live trigger |
| C.4 | Add asset data to ProductAssets table | FileMaker | Extra table for ER |
| C.5 | Upload PDF/ZIP to AWS bucket | AWS S3 | /alfred-sellable-digital-products-other |
| C.6 | Nightly sync enables sale | — | Next day availability |

**Key Difference:** No Legato integration. Sold only on ax3.com.

---

### 2.4 State Transitions

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PRODUCT STATE DIAGRAM                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  [Created in AS400]                                                 │
│         │                                                           │
│         ▼                                                           │
│  [Metadata Entry in FileMaker]                                      │
│         │                                                           │
│         │ Nightly export to SQL                                     │
│         ▼                                                           │
│  [Available in SQL Server]                                          │
│         │                                                           │
│         │ Template upload                                           │
│         ▼                                                           │
│  [Digital SKUs Generated]                                           │
│         │                                                           │
│         │ MRID acquired                                             │
│         ▼                                                           │
│  [Legato Ready]                                                     │
│         │                                                           │
│         │ Assets uploaded                                           │
│         ▼                                                           │
│  [LIVE FOR SALE]                                                    │
│                                                                     │
│  States are implicit. There is no formal status field tracking      │
│  progress through these stages.                                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

### 2.5 Summary of Manual vs. Automated Steps

| Category | Manual Steps | Automated Steps |
|----------|--------------|-----------------|
| **Data Entry** | AS400 record creation, FileMaker metadata entry, template population | — |
| **Data Sync** | — | AS400→Royalty (nightly), FileMaker→SQL (nightly) |
| **SKU Generation** | Template upload to portal | SKU generation by SQL Server |
| **Legato** | Download, clean, upload, download MRIDs, upload MRIDs | Legato MRID assignment |
| **Assets** | Gather, verify, rename, upload (3×) | — |
| **Validation** | All validation is manual | Portal upload validation (limited) |

---

## 3. Open Questions for AX3

Questions are grouped by theme. Each includes:
- The question
- Why it matters
- Priority assessment (Impact / Urgency / Uncertainty)

### 3.1 System Integration and Data Flow

| # | Question | Why It Matters | Impact | Urgency | Uncertainty |
|---|----------|----------------|--------|---------|-------------|
| Q1 | **What is the name and owner of the nightly job that exports FileMaker to SQL Server?** | Understanding ownership is critical for any changes to this integration point. | Medium | Low | High |
| Q2 | **What is the name and owner of the nightly job that syncs AS400 to Royalty System?** | Same rationale as Q1. | Medium | Low | High |
| Q3 | **What specific validations does the portal perform on template upload?** We know it checks AS400, Royalty, and FileMaker data, but what exactly? | To understand what breaks if we change the data model. | High | Medium | High |
| Q4 | **What does the "###95" job number requirement mean exactly?** The Q&A answer was incomplete. | This is mentioned as a failure point for portal processing. | High | Medium | High |
| Q5 | **What are the names of the SQL Server tables for digital-only products?** Meeting notes mention "tblapproducts" and a "pending" table. | Needed to understand data model for digital-first architecture. | Medium | Low | Medium |

### 3.2 SKU and Data Model

| # | Question | Why It Matters | Impact | Urgency | Uncertainty |
|---|----------|----------------|--------|---------|-------------|
| Q6 | **Can we access the spreadsheets referenced in Q&A Question 2?** ("FM Product Fields (working)" and "Values for DigitalVSPhysical from PRODUCT") | These detail required vs. optional fields, critical for simplification. | High | High | Low |
| Q7 | **What exactly does portal.ax3.com pull from the Ref_SKU for Performance and Choral products?** | Understanding this dependency is key to enabling digital-first. | High | High | Medium |
| Q8 | **What would break on ax3.com website if a digital product had no Ref_SKU?** Q&A says "the website logic, I believe, will have some issues." | Need concrete understanding of website dependencies. | High | Medium | High |
| Q9 | **What is a "Web SKU"?** The Q&A glossary says "Unsure." | Unclear if this is a real artifact or obsolete term. | Low | Low | High |

### 3.3 Legato and Distribution

| # | Question | Why It Matters | Impact | Urgency | Uncertainty |
|---|----------|----------------|--------|---------|-------------|
| Q10 | **Why is the Legato process currently required even for AX3-only products?** Is it purely historical, or are there technical dependencies? | Critical for enabling AX3-only distribution path. | High | High | Medium |
| Q11 | **What is the full list of "lower priority fields" mentioned in the Bypassing Legato section?** | To scope the self-service capability properly. | Medium | Medium | Medium |
| Q12 | **Who is the contact at Legato/JW Pepper for technical questions?** | May need to coordinate if any integration changes are considered. | Low | Low | Medium |

### 3.4 Workflow and Process

| # | Question | Why It Matters | Impact | Urgency | Uncertainty |
|---|----------|----------------|--------|---------|-------------|
| Q13 | **How does Travis signal that Phase 2 (data quality review) is complete?** What fields change, or what notification is sent? | Understanding handoffs between phases. | Medium | Medium | High |
| Q14 | **Why is the template not retained after upload?** Is there a reason, or just historical practice? | Retaining templates could aid troubleshooting and audit. | Low | Low | Medium |
| Q15 | **What is PT code 95?** Referenced in meeting notes. | Unknown term that may be relevant. | Low | Low | High |
| Q16 | **What is the actual role of the "Product Manager" in the go-live process?** | Unclear who approves final publication. | Medium | Low | Medium |

### 3.5 Technical Debt and Known Issues

| # | Question | Why It Matters | Impact | Urgency | Uncertainty |
|---|----------|----------------|--------|---------|-------------|
| Q17 | **Why hasn't the Column AK issue been fixed?** Q&A says reports "aren't being maintained." | Symptom of broader maintenance issues. | Medium | Low | Low |
| Q18 | **What is the "Dealer Data file not being formatted in tab delimited anymore" issue?** | Another maintenance issue mentioned but not explained. | Medium | Medium | High |
| Q19 | **Are reports no longer being automatically uploaded?** Q&A mentions this. | May indicate degradation of automation over time. | Medium | Medium | Medium |
| Q20 | **What is Lee's comfort level with making portal changes?** | Affects what changes are feasible in the short term. | High | High | High |

### 3.6 Business and Legal

| # | Question | Why It Matters | Impact | Urgency | Uncertainty |
|---|----------|----------------|--------|---------|-------------|
| Q21 | **Is there business demand to distribute ER products through dealers?** Q&A says "Not sure." | Affects whether ER path needs Legato integration. | Medium | Low | High |
| Q22 | **Are there any contractual obligations with Legato/JW Pepper that require sending all products through their system?** | Legal constraints on bypassing Legato. | High | Medium | High |
| Q23 | **What is the long-term relationship strategy with Legato?** | Informs whether to invest in better Legato integration or decouple. | High | Low | High |

### 3.7 Reconciliation: Conflicts Between Documents

| # | Conflict | Document A | Document B | Recommended Resolution |
|---|----------|------------|------------|------------------------|
| C1 | **Digital SKU derivation**: Knowledge Base says digital SKU numbers derive from physical SKU. Q&A says they are serialized sequences. | Knowledge Base Section 5.4 | Q&A Section 3, Q11 | **Q&A is authoritative.** Digital SKU numbers are serialized, not derived from physical. The Ref_SKU field links them, but the number itself is independent. |
| C2 | **Template workflow**: Meeting notes have conflicting statements about uploading to WebCRD vs. downloading from WebCRD. | Meeting Notes | — | **Need clarification.** Likely context-dependent (Performance uses WebCRD data; digital-only does not). |
| C3 | **Portal domain**: Knowledge Base references portal.alfred.com; Q&A references portal.ax3.com. | Knowledge Base | Q&A | **Likely both valid.** May be aliased domains or documentation inconsistency. Recommend confirming. |

---

## 4. Glossary

### 4.1 Product Types

| Term | Definition |
|------|------------|
| **Performance Set (Ensemble)** | Music for groups of musicians (band, orchestra, jazz band). Includes a conductor's score and multiple individual instrument parts. |
| **Choral Music (Octavo)** | Music for vocal ensembles (choirs). Formatted as tall, narrow booklets (~7" × 10.5"). |
| **Book** | Full publication (method book, folio, songbook). |
| **Sheet** | Standalone single piece of sheet music. |
| **Electronic Resource (ER)** | ZIP files, interactive PDFs, or other non-standard digital formats. Sold only on ax3.com. |

### 4.2 SKU Types

| Code | Name | Description |
|------|------|-------------|
| **PK** | Pack/Set | Complete ensemble set (score + all parts) |
| **PR** | Score | Conductor's score from ensemble set |
| **PC** | Part | Individual instrument part (with suffix like _F1 for Flute 1) |
| **PO** | Octavo | Choral octavo |
| **PS** | Sheet | Standalone sheet music (current standard) |
| **PB** | Book | Full book publication |
| **ER** | Electronic Resource | ZIP files, interactive PDFs |
| **A** | Audio (single) | Single audio track |
| **AA** | Audio (album) | Full album |
| PIP, PSG, PSP, PSL | Legacy Sheet codes | No longer used for new products |
| XIP, XS, XSP | Legacy XML codes | MusicXML files (rarely used) |
| VAP | Video/Audio Package | Multimedia product (rarely used) |

### 4.3 Systems

| Term | Definition |
|------|------------|
| **AS400** | IBM iSeries legacy system. Source of truth for inventory, sales transactions, and SKU creation. All sales (digital and physical) are recorded here. |
| **FileMaker Pro** | Desktop database application. Source of truth for product metadata (title, composer, description, etc.). |
| **SQL Server** | Microsoft database. Stores digital SKU metadata. Receives nightly exports from FileMaker. Generates digital SKUs. |
| **Royalty System** | Database tracking composition rights and royalty payments. Receives nightly sync from AS400 for records with Royalty=Y. |
| **Portal (portal.ax3.com)** | Internal web application for template uploads and workflow management. Writes to SQL Server and AS400. |
| **Legato** | External distribution platform owned by JW Pepper. Assigns MRIDs and enables dealer distribution. No API; web portal only. |
| **WebCRD** | Print production management system (RSA software). Source of print job data for Performance Music. |
| **AWS S3** | Cloud storage. Buckets include alfred-dsm-pdfs (digital files) and alfred-catfiles (cover images). |
| **FTP Server** | ftp1.alfred.com. Hosts MRID files for integrated dealers. |
| **Dropbox** | Archive storage and non-Legato dealer file access. |
| **ax3.com** | Consumer-facing e-commerce website. |

### 4.4 Fields and Flags

| Term | System | Values/Meaning |
|------|--------|----------------|
| **OutputtoWeb** | FileMaker | JJ114 = approved for digital (physical+digital); DIGIONLY = digital-only product |
| **DigitalOnlyYN** | FileMaker | Y = digital-only (dummy SKU); N = has physical product |
| **Ref_SKU** | SQL Server | Links digital SKU to its parent physical/dummy SKU |
| **Status** | AS400 | CUR = Current (active); TOP = Top seller; NEW = New release; PND = Pending |
| **Royalty** | AS400 | Y/N flag controlling sync to Royalty System |
| **Product Type** | AS400 | B = Set; C = Choral; etc. |

### 4.5 Identifiers

| Term | Definition |
|------|------------|
| **SKU (Stock Keeping Unit)** | Unique product identifier. Physical SKUs are numeric only (e.g., 47888). Digital SKUs contain hyphens (e.g., 00-PK-0047888). |
| **Dummy SKU** | Placeholder physical SKU created to satisfy system requirements when no physical product exists. Required for digital-only products. |
| **MRID** | Legato's unique product identifier. Required for dealer distribution. Assigned by Legato after data upload. |
| **UPC** | Universal Product Code. Barcode identifier. Must be consistent across all parts of a set. |
| **Ref_SKU / Parent SKU / Reference SKU** | The physical or dummy SKU that a digital product is linked to. |

### 4.6 Processes and Artifacts

| Term | Definition |
|------|------------|
| **Template** | Excel spreadsheet used to prepare product data for upload to portal. Three variants: Performance_Template.xlsx, Sheets_Template.xls, Choral_Template.xls. |
| **MRID Loop** | The process of uploading data to Legato, receiving MRIDs, and distributing MRIDs to portal and FTP. |
| **Nightly Batch/Sync** | Automated jobs that run overnight: FileMaker→SQL Server export; AS400→Royalty System sync. |

### 4.7 People and Roles

| Term | Definition |
|------|------------|
| **Travis** | Digital Production team member. Key role in data quality review, file processing, and template management. |
| **Lee** | Current maintainer of portal code. Internal staff. |
| **Doug Fraser** | Original developer of backend systems. |
| **Chris Rubeiz** | Built portal.ax3.com. Previously maintained the system. |
| **Editorial Team** | Responsible for entering product metadata in FileMaker. |
| **Purchasing Team** | Responsible for creating initial AS400 records. |

---

## Appendix: Observations (Non-Prescriptive)

The following observations flag potential bottlenecks, structural constraints, and automation candidates. These are framed as observations, not recommendations.

### A.1 Workflow Bottlenecks

| Observation | Evidence |
|-------------|----------|
| **Manual template population is labor-intensive.** Every product requires manual data entry from multiple sources (WebCRD, FileMaker) into Excel templates. | Q&A Question 34 notes "Most are already" auto-populated via FileMaker script for books/sheets, but Performance/Choral still require significant manual work. |
| **Triple asset upload creates redundant work.** The same PDF must be uploaded to Legato, AWS S3, and Dropbox separately. | Knowledge Base Section 7, Phase 6. |
| **Lack of global edit capability for performance sets.** Changing territory restrictions requires editing 20+ records individually. | Q&A "Bypassing Legato" section explicitly describes this pain point. |
| **Legato round-trip adds latency even for AX3-only products.** The Legato process is currently required for all digital products. | Q&A Question 37. |

### A.2 Structural Constraints

| Observation | Evidence |
|-------------|----------|
| **The dummy SKU requirement is historical, not technically necessary for Legato or downstream systems.** | Q&A confirms Legato only cares about digital SKU (Q24), royalties are programmed separately (Q26), and no financial systems require physical SKU (Q27). |
| **The portal was built with physical-first assumptions.** Removing the dummy SKU requirement would require portal modifications. | Q&A Question 19: "The portal was built off of assumptions for physical though." |
| **Performance and Choral products have stronger dependencies on Ref_SKU than Books/Sheets.** | Q&A Question 30: Books/Sheets can work without dummy SKU (with possible website issues); Performance/Choral cannot. |

### A.3 Automation Candidates

| Area | Current State | Automation Potential |
|------|---------------|---------------------|
| Template population | Manual data entry | High — data exists in FileMaker and WebCRD |
| UPC/sort order validation | Manual checklist | High — rules are well-defined |
| Legato file cleanup (Column AK) | Manual Excel edit | High — predictable transformation |
| MRID distribution | Manual upload to portal + FTP | High — file copy operations |
| Asset upload (3 destinations) | Manual upload × 3 | High — parallel upload to known destinations |
| Global field updates (territory, etc.) | Edit records one-by-one | High — bulk update capability |

### A.4 Data Model Observations

| Observation | Implication |
|-------------|-------------|
| Digital SKU numbers are serialized sequences, not derived from physical SKU. | The Ref_SKU relationship is a soft link, not a naming convention dependency. |
| The Ref_SKU field is the only enforced link between digital and physical products. | Removing dummy SKU means eliminating or replacing this link. |
| FileMaker validation is weak ("accepts it, and that field will be blank"). | Data quality depends on process discipline, not system enforcement. |

---

*End of Document*
