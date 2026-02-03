# AX3 Digitization Initiative — As-Is Workflow Reconstruction & Open Questions

## 1. Current Assumptions

> Assumptions are listed because the source set contains gaps, internal inconsistencies, and a mix of “knowledge base” statements vs. client-confirmed answers. Conflicts are explicitly flagged.

1. **FileMaker is the source of truth for core product metadata used to create/maintain digital sellables.**  
   *Evidence / rationale:* AX3 states print metadata must exist first and FileMaker Pro is the source of truth for that metadata; FileMaker is exported nightly into SQL for downstream use. :contentReference[oaicite:0]{index=0} :contentReference[oaicite:1]{index=1} :contentReference[oaicite:2]{index=2}  
   *Confidence:* **High**

2. **AS400 is the system of record for sales transactions for both physical and digital products.**  
   *Evidence / rationale:* AX3 explicitly states AS400 tracks every transactional sale and a record is required for any product that sells; without an AS400 record, sales would have “nowhere to report.” :contentReference[oaicite:3]{index=3}  
   *Confidence:* **High**

3. **Digital SKUs are generated in SQL Server and are linked back to a physical/dummy SKU via a “Ref_SKU” field (at least for major workflows).**  
   *Evidence / rationale:* AX3 explicitly describes Ref_SKU as the linkage and that SQL auto-generates digital SKUs based on the physical/dummy SKU in Ref_SKU. :contentReference[oaicite:4]{index=4}  
   *Confidence:* **High**

4. **A “dummy SKU” is a required workaround for digital-only products in the current operational model for at least some product types.**  
   *Evidence / rationale:* Multiple sources state digital products cannot exist without a print (or dummy) product first; Path B describes creation of dummy SKU in FileMaker + AS400 + royalties as prerequisite. However, AX3 also notes some historical exceptions and partial capability for books/sheets. :contentReference[oaicite:5]{index=5} :contentReference[oaicite:6]{index=6} :contentReference[oaicite:7]{index=7}  
   *Confidence:* **High** (requirement exists broadly), **Medium** (universality across all product types)

5. **FileMaker flags drive “go-live eligibility” and differ between physical+digital vs digital-only.**  
   *Evidence / rationale:* Notes list `OutputToWeb = JJ114` for physical+digital and `OutputToWeb = DIGIONLY` for digital-only; Q&A supports DIGIONLY as the digital-only flag and suggests it could be repurposed. :contentReference[oaicite:8]{index=8} :contentReference[oaicite:9]{index=9}  
   *Confidence:* **High**

6. **A nightly batch export exists from FileMaker to SQL Server (CSV-based), and portal processing relies on those SQL tables.**  
   *Evidence / rationale:* Meeting notes and Q&A describe nightly FileMaker download into SQL tables (e.g., tblapproducts / pending) and portal pulling metadata from nightly downloads. :contentReference[oaicite:10]{index=10} :contentReference[oaicite:11]{index=11}  
   *Confidence:* **High**

7. **Portal (portal.ax3.com) writes to SQL Server and AS400 and performs validation checks across AS400, royalties, and FileMaker-derived SQL tables.**  
   *Evidence / rationale:* AX3 states portal writes to SQL and AS400; validation checks exist but details are not enumerated (Lee “might know”). :contentReference[oaicite:12]{index=12}  
   *Confidence:* **Medium**

8. **Legato MRID acquisition is a manual round-trip with no API integration.**  
   *Evidence / rationale:* AX3 states Legato does not have an API; MRID download/upload loop is manual. :contentReference[oaicite:13]{index=13} :contentReference[oaicite:14]{index=14}  
   *Confidence:* **High**

9. **Digital SKU numbering is serialized and not mathematically derived from the physical SKU value.**  
   *Evidence / rationale:* AX3 explicitly states the numeric portion is serialized; only the prefix indicates type and depends on workflow. This conflicts with some older diagrammatic examples that depict a more direct “derived from physical” relationship. **Conflict flagged:** “derived from physical SKU” is described conceptually in some materials, but the client answer states serialization is independent. :contentReference[oaicite:15]{index=15} :contentReference[oaicite:16]{index=16} :contentReference[oaicite:17]{index=17}  
   *Confidence:* **High** (serialization), **Medium** (how often legacy “derived” framing still appears operationally)

10. **Digital-only distribution through Legato is possible, but current practice and/or process may treat Legato as required.**  
   *Evidence / rationale:* **Direct conflict:**  
   - Q23/Q24: digital-only can go through Legato; Legato only cares about digital SKU; dummy SKU is internal. :contentReference[oaicite:18]{index=18}  
   - Q37 statement: “Today, the Legato process IS required for digital-only products and can’t easily be skipped.” :contentReference[oaicite:19]{index=19}  
   - Knowledge base Path B: Legato integration “limited or none” and dealers via Dropbox. :contentReference[oaicite:20]{index=20}  
   *Confidence:* **Low** until AX3 reconciles definitions of “required” (technical vs business vs “required for dealer reach”).

11. **Digital SKU type codes and formats follow the documented conventions (PK/PR/PC/PO/PS/PB/etc.).**  
   *Evidence / rationale:* Digital SKU explanation document enumerates formats and rules (hyphen implies digital SKU). Knowledge base mirrors these definitions. :contentReference[oaicite:21]{index=21} :contentReference[oaicite:22]{index=22}  
   *Confidence:* **High**

12. **AS400 has an item-number length constraint that may force aggregation of individual part purchases under a shared identifier.**  
   *Evidence / rationale:* Meeting notes state individual parts are tracked to the same SKU due to character limits, but this is not corroborated elsewhere. :contentReference[oaicite:23]{index=23}  
   *Confidence:* **Low–Medium** (needs confirmation)

---

## 2. Current Workflow (As-Is)

> Scope: end-to-end operational workflow from “new music acquired” to “digital product sellable,” including physical+digital and digital-only variants, as currently executed. Observational bottlenecks are included inline and labeled as **Observation** (non-prescriptive).

### 2.1 Intake and prerequisite records (common foundation)

1. **Create initial product/item record in AS400 (physical or dummy anchor).**  
   1.1 **Actor:** Purchasing team  
   1.2 **System:** AS400  
   1.3 **Input artifact:** Acquisition decision / new title details (not fully specified)  
   1.4 **Output artifact:** AS400 product record (SKU/pub-item), using sequential numbering approach (per notes)  
   1.5 **State change:** AS400 product enters an initial lifecycle status (varies by path; see §2.4/2.5)  
   1.6 **Manual vs automated:** Manual creation :contentReference[oaicite:24]{index=24}  

2. **Overnight propagation to Royalty System for SKUs marked royalty-eligible.**  
   2.1 **Actor:** System job (owner/name unknown)  
   2.2 **System(s):** AS400 → Royalty System  
   2.3 **Trigger/condition:** AS400 “Royalties (Y/N)” = “Y” (per notes)  
   2.4 **Output artifact:** Royalty System record(s) / availability for rights setup  
   2.5 **Manual vs automated:** Automated nightly job :contentReference[oaicite:25]{index=25}  

3. **Create/maintain product metadata in FileMaker.**  
   3.1 **Actor:** Editorial team  
   3.2 **System:** FileMaker (Product + related tables)  
   3.3 **Input artifact:** Title metadata, contributors, and (optionally) TOC song list  
   3.4 **Output artifacts:**  
   - `Product` table record (core metadata)  
   - `ContributorRole` rows (contributors per product)  
   - `TOCSongSources` rows when multi-song publication :contentReference[oaicite:26]{index=26}  
   3.5 **Manual vs automated:** Manual entry :contentReference[oaicite:27]{index=27}  

4. **Pre-validation / data QC before downstream digital processing.**  
   4.1 **Actor:** Travis / “X team” (Digital Production function inferred)  
   4.2 **System(s):** FileMaker (and likely AS400/Royalty references)  
   4.3 **Input artifact:** Newly created FileMaker product data  
   4.4 **Output artifact:** Corrected/normalized FileMaker product record (exact QC checklist unknown)  
   4.5 **State change:** “Ready for marketing check” (informal; exact field/marker unknown)  
   4.6 **Manual vs automated:** Manual QC :contentReference[oaicite:28]{index=28}  
   **Observation:** QC responsibility appears concentrated in an individual role with unclear completion signaling, increasing cycle-time variability and rework risk. :contentReference[oaicite:29]{index=29}  

5. **Marketing/Product Manager review and publish-to-web gating (for AX3.com eligibility).**  
   5.1 **Actor:** Product (Marketing) Manager (as noted)  
   5.2 **Systems:** FileMaker + AS400 + Royalty System (gating conditions vary by path)  
   5.3 **Input artifact:** “QC’d” FileMaker record  
   5.4 **Output artifacts:** Field updates that enable downstream jobs (see §2.4/2.5)  
   5.5 **Manual vs automated:** Manual decision/review :contentReference[oaicite:30]{index=30}  

6. **Nightly export of FileMaker data into SQL Server tables for portal/processing.**  
   6.1 **Actor:** System job (owner/name unknown)  
   6.2 **Systems:** FileMaker → SQL Server (via CSV export per notes)  
   6.3 **Output artifact:** SQL tables used by portal processes (examples mentioned: `tblapproducts`, “pending table”)  
   6.4 **Manual vs automated:** Automated nightly export :contentReference[oaicite:31]{index=31} :contentReference[oaicite:32]{index=32}  
   **Observation:** Nightly batching introduces inherent latency and creates “next day” dependency for error discovery and correction. :contentReference[oaicite:33]{index=33} :contentReference[oaicite:34]{index=34}  

---

### 2.2 Template preparation and data assembly (bridge from metadata to digital SKU creation)

7. **Assemble data required to create digital SKUs and digital sellable records.**  
   7.1 **Actors:** Digital Production (Travis) with inputs from Editorial and system exports  
   7.2 **Systems / sources referenced:** WebCRD, FileMaker, existing PDFs, and/or reports; plus “Template” spreadsheets. :contentReference[oaicite:35]{index=35} :contentReference[oaicite:36]{index=36} :contentReference[oaicite:37]{index=37}  
   7.3 **Input artifacts:**  
   - Product metadata (FileMaker; via direct use and/or nightly-exported SQL representation)  
   - Print job details (WebCRD) for some product types (performance music emphasized) :contentReference[oaicite:38]{index=38}  
   - PDF assets (print archive/AWS/Slack depending on path) :contentReference[oaicite:39]{index=39} :contentReference[oaicite:40]{index=40}  
   7.4 **Output artifact:** Completed Excel template rows (one row per sellable item/part, depending on product type) :contentReference[oaicite:41]{index=41} :contentReference[oaicite:42]{index=42}  
   7.5 **Manual vs automated:** Largely manual assembly and formatting :contentReference[oaicite:43]{index=43} :contentReference[oaicite:44]{index=44}  
   **Observation:** Multiple-source data gathering (WebCRD + FileMaker + PDFs) and manual “format into template” is explicitly called the major bottleneck. :contentReference[oaicite:45]{index=45}  

8. **Create and manage the Excel “Template” file used for downstream processing.**  
   8.1 **Actor:** Digital Production (Travis)  
   8.2 **System:** Excel (template artifacts)  
   8.3 **Input artifact:** Assembled metadata and part/score/set structure rules  
   8.4 **Output artifact:** Template file ready for upload to portal (and, for print, potentially to WebCRD)  
   8.5 **State change:** “Data package ready for ingestion”  
   8.6 **Manual vs automated:** Manual creation; unclear retention policy (“Template is not kept after upload” in notes) :contentReference[oaicite:46]{index=46}  
   **Conflict flagged:** Notes indicate conflicting sequences: “upload physical product to WebCRD,” “download the Template from WebCRD,” and “scrub the Template.” The knowledge base describes templates as a manual artifact selected by product type, not downloaded from WebCRD. :contentReference[oaicite:47]{index=47} :contentReference[oaicite:48]{index=48}  

---

### 2.3 Portal ingestion and digital SKU generation (systemic creation step)

9. **Upload template to portal.ax3.com to trigger validation and creation steps.**  
   9.1 **Actor:** Digital Production (Travis)  
   9.2 **System:** portal.ax3.com (portal)  
   9.3 **Input artifact:** Completed Excel template  
   9.4 **Output artifacts:**  
   - Validated ingestion results (success or error messages)  
   - New digital SKU records in SQL Server  
   - New digital SKU records in AS400 (for sales reporting)  
   - (Potentially) new digital SKU records in Royalty System / royalty eligibility chain  
   9.5 **State change:** “Digital SKU(s) created” and “digital products registered for downstream distribution”  
   9.6 **Manual vs automated:** Manual upload + automated processing :contentReference[oaicite:49]{index=49} :contentReference[oaicite:50]{index=50}  

10. **Portal validation checks across systems (partially known).**  
   10.1 **Actor:** Portal automation (code maintained internally, “Lee” inferred)  
   10.2 **Systems referenced:** SQL Server (nightly FileMaker-derived tables), AS400, Royalty System, FileMaker-derived metadata  
   10.3 **Inputs:** Template row values + system-of-record references (Ref_SKU, etc.)  
   10.4 **Outputs:**  
   - Errors when required data is blank/misaligned  
   - Successful creation when checks pass  
   10.5 **Manual vs automated:** Automated checks; details not enumerated :contentReference[oaicite:51]{index=51}  
   **Observation:** Validation logic is opaque to operators (“no list of checks”), increasing troubleshooting time and reliance on specific individuals with backend access. :contentReference[oaicite:52]{index=52}  

11. **Digital SKU format assignment (type-code selection + serialized numbering).**  
   11.1 **Actor:** SQL Server process (stored procedure/code; not directly accessible to business users)  
   11.2 **System:** SQL Server  
   11.3 **Inputs:** Workflow path/type selection + Ref_SKU + product-type conventions  
   11.4 **Outputs:** Digital SKUs in standardized formats (examples):  
   - Performance set family: `00-PK-...` (set), `00-PR-...` (score), `00-PC-...` (parts) :contentReference[oaicite:53]{index=53}  
   - Choral octavo: `00-PO-...` :contentReference[oaicite:54]{index=54}  
   - Sheets/Books: `00-PS-...`, `00-PB-...` :contentReference[oaicite:55]{index=55}  
   11.5 **Manual vs automated:** Automated SKU generation :contentReference[oaicite:56]{index=56}  

---

### 2.4 Go-live gating conditions (AX3.com sellability signals)

12. **Physical + digital product go-live gating (as captured in meeting notes).**  
   12.1 **Actor:** Cross-system gating (Marketing/Product Manager + batch jobs)  
   12.2 **Systems:** FileMaker + AS400 + Royalty System  
   12.3 **Required conditions (per notes):**  
   - FileMaker `OutputToWeb = JJ114`  
   - AS400 `Status = CUR`  
   - Digital rights set up in Royalty System for the *physical* SKU :contentReference[oaicite:57]{index=57}  
   12.4 **Output artifact:** Product becomes eligible to appear on AX3.com (as described)  
   12.5 **Manual vs automated:** Field-setting manual; effect realized via nightly jobs and downstream processes (implied) :contentReference[oaicite:58]{index=58}  

13. **Digital-only product go-live gating (as captured in meeting notes + Q&A).**  
   13.1 **Actor:** Cross-system gating (Editorial/Travis/Marketing + batch jobs)  
   13.2 **Systems:** FileMaker + AS400 + Royalty System  
   13.3 **Required conditions (per notes):**  
   - FileMaker `OutputToWeb = DIGIONLY`  
   - AS400: `Status = PND` and other fields (Price/Territories) set appropriately (some values unclear in notes)  
   - Digital rights set up in Royalty System for the *dummy* SKU :contentReference[oaicite:59]{index=59}  
   13.4 **Dummy record content expectations (partial, client-provided):**  
   - FileMaker: many metadata fields are still required; validation is weak (skipped fields become blanks in SQL nightly download and then portal pulls blanks) :contentReference[oaicite:60]{index=60}  
   - AS400: must have price, Royalty=Y, appropriate product type codes, etc. :contentReference[oaicite:61]{index=61}  
   13.5 **Manual vs automated:** Manual record creation and rights setup :contentReference[oaicite:62]{index=62}  
   **Observation:** Digital-only still inherits physical-first overhead via duplicate record creation (FileMaker + AS400 + royalties), creating structural cycle-time and error surface area. :contentReference[oaicite:63]{index=63} :contentReference[oaicite:64]{index=64}  

---

### 2.5 Legato distribution and MRID loop (dealer distribution)

14. **Generate and upload Legato metadata export.**  
   14.1 **Actor:** Digital Production  
   14.2 **Systems:** SQL Server reporting → Legato web portal  
   14.3 **Input artifact:** Legato upload file (XLS/XLSX) generated from SQL Server report (exact report name not confirmed in notes) :contentReference[oaicite:65]{index=65} :contentReference[oaicite:66]{index=66}  
   14.4 **Manual step noted:** Remove “AK” column to avoid import failures (known issue) :contentReference[oaicite:67]{index=67} :contentReference[oaicite:68]{index=68} :contentReference[oaicite:69]{index=69}  
   14.5 **Output artifact:** Uploaded metadata in Legato :contentReference[oaicite:70]{index=70}  
   **Observation:** Manual “fix-up” of export files (AK column) indicates brittle coupling between export format and receiver validation; recurring operational tax. :contentReference[oaicite:71]{index=71} :contentReference[oaicite:72]{index=72}  

15. **Receive MRID(s) from Legato and propagate back into AX3 ecosystem.**  
   15.1 **Actor:** Digital Production  
   15.2 **Systems:** Legato → portal → SQL Server; plus FTP for dealers  
   15.3 **Input artifact:** MRID file(s) downloaded from Legato / report server outputs  
   15.4 **Output artifacts:**  
   - MRIDs uploaded into portal to update SQL-based tables  
   - MRID feed files placed on FTP for integrated dealers :contentReference[oaicite:73]{index=73}  
   15.5 **Manual vs automated:** Manual download/upload; no Legato API :contentReference[oaicite:74]{index=74} :contentReference[oaicite:75]{index=75}  

16. **Determine whether a given digital product is distributed via Legato vs AX3-only / alternate methods.**  
   16.1 **Actor:** Digital Production + Business decision owners  
   16.2 **Systems impacted:** Legato, AX3.com, FTP, Dropbox/AWS  
   16.3 **Conflict flagged (needs resolution):**  
   - Some materials state digital-only may be “limited/none” in Legato and dealers use Dropbox. :contentReference[oaicite:76]{index=76}  
   - Q23/Q24 say digital-only can go through Legato; nothing breaks without dummy SKU; business decision drives Legato usage. :contentReference[oaicite:77]{index=77}  
   - Another section asserts Legato is “required” today for digital-only and cannot easily be skipped. :contentReference[oaicite:78]{index=78}  
   16.4 **Manual vs automated:** Mostly manual decision + manual operational routing (as described)

---

### 2.6 Asset acquisition, renaming, and uploads (file operations)

17. **Acquire PDF (or ZIP) assets and validate readiness.**  
   17.1 **Actor:** Digital Production  
   17.2 **Systems/sources:** Print archive/Dropbox, AWS buckets, Slack from Editorial (depending on path) :contentReference[oaicite:79]{index=79} :contentReference[oaicite:80]{index=80}  
   17.3 **Input artifact:** Source PDF(s) / ZIP(s)  
   17.4 **Output artifact:** “Approved” files ready for naming/upload (criteria include covers/fonts/blank pages per KB) :contentReference[oaicite:81]{index=81}  
   17.5 **Manual vs automated:** Manual inspection/handling :contentReference[oaicite:82]{index=82}  

18. **Rename files to match digital SKU naming expectations.**  
   18.1 **Actor:** Digital Production  
   18.2 **Systems:** Local workstation + mapping output (report)  
   18.3 **Input artifact:** PDF files + SKU mapping (SQL-assigned digital SKUs) :contentReference[oaicite:83]{index=83} :contentReference[oaicite:84]{index=84}  
   18.4 **Output artifact:** Correctly named files (exact match to digital SKU) :contentReference[oaicite:85]{index=85} :contentReference[oaicite:86]{index=86}  
   18.5 **Manual vs automated:** Manual renaming described :contentReference[oaicite:87]{index=87} :contentReference[oaicite:88]{index=88}  
   **Observation:** File renaming is a high-frequency manual step tightly coupled to SKU generation timing; errors directly cause customer-facing delivery failures. :contentReference[oaicite:89]{index=89} :contentReference[oaicite:90]{index=90}  

19. **Upload files to required destinations (multi-target publishing).**  
   19.1 **Actor:** Digital Production  
   19.2 **Systems / destinations:**  
   - Legato (dealer distribution assets)  
   - AWS (AX3.com downloads)  
   - Dropbox (archive and/or non-Legato distributors)  
   - FTP (MRID feeds, not the PDFs themselves, for integrated dealers) :contentReference[oaicite:91]{index=91} :contentReference[oaicite:92]{index=92}  
   19.3 **Input artifact:** Renamed digital assets  
   19.4 **Output artifact:** Assets available in each channel’s expected location  
   19.5 **Manual vs automated:** Manual uploads described across destinations :contentReference[oaicite:93]{index=93} :contentReference[oaicite:94]{index=94}  
   **Observation:** Duplicate uploads to multiple repositories increase time cost and risk of partial publication (one destination missing or outdated). :contentReference[oaicite:95]{index=95} :contentReference[oaicite:96]{index=96}  

---

### 2.7 SKU structures and dependencies (how physical/dummy relates to digital)

20. **Establish and manage SKU relationships (conceptual and operational).**  
   20.1 **Actor:** System processes + Digital Production  
   20.2 **Systems:** FileMaker, SQL Server, AS400  
   20.3 **Artifacts:**  
   - Physical SKU (numeric) or dummy physical SKU (internal placeholder)  
   - Digital SKUs (hyphenated PUB-ITEM formats) such as PK/PR/PC/PO/PS/PB :contentReference[oaicite:97]{index=97} :contentReference[oaicite:98]{index=98}  
   20.4 **Dependency:**  
   - Operational linkage commonly expressed via Ref_SKU (physical/dummy anchor) :contentReference[oaicite:99]{index=99}  
   - Digital SKU numbering serialized; not inherently derived from physical numeric SKU per client answer :contentReference[oaicite:100]{index=100}  
   20.5 **Manual vs automated:** Link establishment automated during portal/SQL processing, but requires correct upstream setup (dummy/physical + royalty + AS400 fields) :contentReference[oaicite:101]{index=101}  

---

## 3. Open Questions for AX3

> Questions include (a) explicit questions in meeting notes, (b) gaps implied by incomplete answers, and (c) conflicts across sources.  
> Priority scoring is provided per question: **Impact / Urgency / Uncertainty**.

### Theme A — System-of-record boundaries & data flows

1. **What are the exact SQL Server table names and schemas used for: (a) physical-derived digital products, and (b) digital-only products (noted as “different table”)?**  
   *Why it matters:* Required to trace where digital-only becomes “first-class” vs “pending,” and to quantify change surface and operational dependencies. :contentReference[oaicite:102]{index=102}  
   *Priority:* Impact **High** / Urgency **High** / Uncertainty **High**

2. **What is the authoritative, current integration diagram for portal.ax3.com: which systems it reads vs writes, and through what mechanisms (direct connections vs nightly tables)?**  
   *Why it matters:* Confirms which dependencies are hard constraints vs replaceable conventions; needed to assess workflow coupling without proposing designs. :contentReference[oaicite:103]{index=103}  
   *Priority:* Impact **High** / Urgency **High** / Uncertainty **Medium**

3. **What are the names, owners, schedules, and failure modes of the nightly jobs: (a) AS400 → Royalty System, and (b) FileMaker → SQL export?**  
   *Why it matters:* These jobs appear to be critical path gates and latency sources; ownership and monitoring determine operational reliability. :contentReference[oaicite:104]{index=104} :contentReference[oaicite:105]{index=105}  
   *Priority:* Impact **High** / Urgency **Medium** / Uncertainty **High**

4. **How does ax3.com pull product data at runtime (from SQL tables, from FileMaker exports, from AS400, or a dedicated “product DB”)?**  
   *Why it matters:* Several statements imply the website “pulls appropriate data from the physical SKU”; this is central to digital-only feasibility and to “AX3-only” selling. :contentReference[oaicite:106]{index=106}  
   *Priority:* Impact **High** / Urgency **High** / Uncertainty **High**

---

### Theme B — Go-live gating rules & validation logic

5. **What is the definitive checklist of portal.ax3.com validations per workflow (performance/choral/books/sheets), including required fields and cross-system checks?**  
   *Why it matters:* Validation opacity drives rework; it also defines the minimal viable “digital master” data set (even if AX3 reuses DIGIONLY). :contentReference[oaicite:107]{index=107}  
   *Priority:* Impact **High** / Urgency **High** / Uncertainty **High**

6. **What field(s) or state(s) indicate “Travis is done” and that a title is ready for manager review? Is there a formal state machine in FileMaker or elsewhere?**  
   *Why it matters:* Current handoff appears informal; without an explicit state transition, cycle time and accountability are hard to measure. :contentReference[oaicite:108]{index=108}  
   *Priority:* Impact **Medium** / Urgency **Medium** / Uncertainty **High**

7. **Confirm the exact AS400 field values required for digital-only readiness (notes mention Price=USD, Territories=“Value”, Status=PND). What are the valid values and meanings?**  
   *Why it matters:* Digital-only gating depends on correct AS400 setup; ambiguous values cause processing failures and delayed launches. :contentReference[oaicite:109]{index=109} :contentReference[oaicite:110]{index=110}  
   *Priority:* Impact **High** / Urgency **High** / Uncertainty **High**

8. **What does “PT code 95” refer to in portal processing (and is it related to the AS400 job-number requirement of “###95” mentioned elsewhere)?**  
   *Why it matters:* Appears to be a hidden prerequisite that can block processing; unclear whether it is a code, a job type, or a data linkage convention. :contentReference[oaicite:111]{index=111} :contentReference[oaicite:112]{index=112}  
   *Priority:* Impact **High** / Urgency **Medium** / Uncertainty **High**

---

### Theme C — SKU semantics, relationships, and constraints

9. **Resolve the contradiction: Are digital SKU numeric portions serialized independent of the physical SKU, or are they derived/mirrored from physical SKUs in some workflows?**  
   *Why it matters:* Determines whether “physical-first” is a true identifier dependency or primarily a record-presence dependency; also impacts traceability of parent/child relationships. **Conflict exists across sources.** :contentReference[oaicite:113]{index=113} :contentReference[oaicite:114]{index=114} :contentReference[oaicite:115]{index=115}  
   *Priority:* Impact **High** / Urgency **High** / Uncertainty **High**

10. **Confirm AS400 item-number length constraints and current practice for tracking part-level sales (notes say parts map to same SKU). Is this true, and if so, where is the mapping stored?**  
   *Why it matters:* Affects reporting granularity, royalty allocation assumptions, and whether “part SKUs” exist as distinct sellables in AS400. :contentReference[oaicite:116]{index=116}  
   *Priority:* Impact **High** / Urgency **Medium** / Uncertainty **High**

11. **What is the definition of “Web SKU” (explicitly called out as “Unsure” in Q&A notes)?**  
   *Why it matters:* Ambiguity suggests a parallel identifier used in ax3.com or downstream tables; resolving prevents mis-modeling identifiers. :contentReference[oaicite:117]{index=117}  
   *Priority:* Impact **Medium** / Urgency **Low** / Uncertainty **High**

---

### Theme D — Legato usage, AX3-only selling, and channel routing

12. **Resolve the contradiction: Is Legato required today for digital-only products, or can digital-only products be sold without Legato (AX3.com only, and/or dealers via Dropbox)? What does “required” mean operationally (dealer reach vs technical gating)?**  
   *Why it matters:* Directly impacts the secondary objective (AX3-only selling) and the scope of “digital-only as first-class.” **Conflict exists within Q&A and between Q&A and KB.** :contentReference[oaicite:118]{index=118} :contentReference[oaicite:119]{index=119}  
   *Priority:* Impact **High** / Urgency **High** / Uncertainty **High**

13. **If AX3 wants “sell on ax3.com but not Legato,” what is the current mechanism (if any) to tag a product as AX3-only, and where is that tag stored/read?**  
   *Why it matters:* A “tag AX3-only or Legato-or-not in SQL + downstream logic” is described as a need; current-state support is unclear. :contentReference[oaicite:120]{index=120}  
   *Priority:* Impact **High** / Urgency **High** / Uncertainty **High**

14. **For non-Legato distributors (FTP / alternate dealers), what is the current authoritative distribution process and artifact set (metadata + assets)?**  
   *Why it matters:* Some flows cite FTP for MRID feeds and Dropbox for dealer access; clarifying actual operational routing prevents incorrect assumptions about channel obligations. :contentReference[oaicite:121]{index=121} :contentReference[oaicite:122]{index=122} :contentReference[oaicite:123]{index=123}  
   *Priority:* Impact **Medium** / Urgency **Medium** / Uncertainty **High**

---

### Theme E — Templates, WebCRD, and print/digital coupling

15. **Is the Excel “Template” ever generated from WebCRD (downloaded), or always manually created? What is the exact sequence for performance sets?**  
   *Why it matters:* Conflicting notes suggest different operational realities; this determines whether the “template bottleneck” is data entry vs data transformation. :contentReference[oaicite:124]{index=124} :contentReference[oaicite:125]{index=125}  
   *Priority:* Impact **High** / Urgency **Medium** / Uncertainty **High**

16. **Where is the template stored (if at all) after upload, and what is the audit trail for what was uploaded and when?**  
   *Why it matters:* Notes say the template is not kept; lack of retention limits traceability, debugging, and post-incident analysis. :contentReference[oaicite:126]{index=126}  
   *Priority:* Impact **Medium** / Urgency **Medium** / Uncertainty **High**

---

### Theme F — Ownership, maintainability, and operational support

17. **Who currently maintains portal.ax3.com code and SQL stored procedures, and what is the deployment/change process?**  
   *Why it matters:* Change feasibility and risk are driven by ownership and release process; current answer is partially speculative (“I guess now it is Lee”). :contentReference[oaicite:127]{index=127}  
   *Priority:* Impact **High** / Urgency **Medium** / Uncertainty **Medium**

18. **What “self-serve” edits does Digital Production need to make in the MR Upload (or equivalent) SQL table today, and what is the current access/control model?**  
   *Why it matters:* AX3 describes a set of essential fields requiring data engineer intervention; confirms where operational friction concentrates. :contentReference[oaicite:128]{index=128}  
   *Priority:* Impact **High** / Urgency **Medium** / Uncertainty **Medium**

---

## 4. Glossary

**AA- / A- SKU** — Digital audio SKUs: “A” for single track, “AA” for album; distributed through channels including Naxos; sometimes linked to physical products. :contentReference[oaicite:129]{index=129}  

**AS400** — Legacy IBM system used by AX3 for product records and transactional sales reporting for physical and digital products. :contentReference[oaicite:130]{index=130} :contentReference[oaicite:131]{index=131}  

**AX3.com** — AX3 direct-to-consumer ecommerce channel (referred to as ax3.com in the materials). :contentReference[oaicite:132]{index=132} :contentReference[oaicite:133]{index=133}  

**ContributorRole (FileMaker table)** — Stores contributor name/role/sort-order for a product. :contentReference[oaicite:134]{index=134}  

**Digital SKU** — A PUB-ITEM identifier that represents a digital sellable; any SKU containing “-” is digital in AX3’s conventions. Examples: `00-PK-0000123`, `00-PO-0000123`. :contentReference[oaicite:135]{index=135} :contentReference[oaicite:136]{index=136}  

**Dummy SKU** — A physical-placeholder SKU/record created to satisfy physical-first system constraints for digital-only products; exists internally and is not customer-facing per AX3. :contentReference[oaicite:137]{index=137} :contentReference[oaicite:138]{index=138}  

**ER SKU** — “Electronic Resource” SKU type used for non-standard digital assets (ZIPs, interactive PDFs). Limited volume and process described as new/under-documented. :contentReference[oaicite:139]{index=139} :contentReference[oaicite:140]{index=140}  

**FileMaker (FM)** — Product metadata database; source of truth for print metadata and upstream metadata for digitization; contains `Product`, `ContributorRole`, and sometimes `TOCSongSources`. :contentReference[oaicite:141]{index=141} :contentReference[oaicite:142]{index=142}  

**Legato** — External distributor platform owned by JW Pepper; used for dealer distribution of digital products and assigns MRIDs; no API per AX3. :contentReference[oaicite:143]{index=143} :contentReference[oaicite:144]{index=144}  

**MRID** — Legato-assigned unique identifier required by Legato-integrated dealers to access product data/assets. :contentReference[oaicite:145]{index=145} :contentReference[oaicite:146]{index=146}  

**OutputToWeb / Outputtoweb** — FileMaker field used as an eligibility flag (examples: `JJ114` for approved digital conversion; `DIGIONLY` for digital-only). :contentReference[oaicite:147]{index=147} :contentReference[oaicite:148]{index=148} :contentReference[oaicite:149]{index=149}  

**PC / PK / PR** — Digital SKU type codes for performance music: PC=Part, PK=Set/Pack, PR=Score. :contentReference[oaicite:150]{index=150}  

**PO** — Digital SKU type code for choral octavos. :contentReference[oaicite:151]{index=151}  

**PS / PB (and legacy PIP/PSG/PSP/PSL)** — Digital SKU type codes for sheets (PS current) and books (PB); legacy sheet prefixes exist but not used for new products. :contentReference[oaicite:152]{index=152}  

**Portal (portal.ax3.com)** — Internal AX3 web application used to upload templates, trigger SQL processing, and write to SQL Server and AS400. :contentReference[oaicite:153]{index=153}  

**Ref_SKU** — SQL field described by AX3 as the linkage between physical/dummy SKUs and generated digital SKUs. :contentReference[oaicite:154]{index=154}  

**Royalty System** — System that stores rights/royalty configuration; digital rights must be set up for a SKU before digital sale eligibility (physical SKU for physical+digital; dummy SKU for digital-only per notes). :contentReference[oaicite:155]{index=155} :contentReference[oaicite:156]{index=156} :contentReference[oaicite:157]{index=157}  

**SQL Server** — Stores digital product metadata and generates digital SKUs; receives nightly FileMaker-derived data and portal writes. :contentReference[oaicite:158]{index=158} :contentReference[oaicite:159]{index=159}  

**Template (Excel)** — Spreadsheet format used to package product rows for ingestion; ensemble-specific and used for digital SKU creation workflows; described as major bottleneck and often manually prepared. :contentReference[oaicite:160]{index=160} :contentReference[oaicite:161]{index=161} :contentReference[oaicite:162]{index=162}  

**TOCSongSources (FileMaker table)** — Optional table storing song titles within a multi-song publication. :contentReference[oaicite:163]{index=163}  

**WebCRD** — Print job management system and a data source for some template population steps (especially performance music). :contentReference[oaicite:164]{index=164} :contentReference[oaicite:165]{index=165}  
