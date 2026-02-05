Meeting date Feb 2nd 2026

Transcript not available, these are my notes.

# Product Types
- There are 4 Product Types: performance sets, choral music, books, sheets.
- Books and sheets are almost the same, just a small difference in the SKU.
- "Performance set" is a set of music "pieces", one for instrument. Ex: for a band or an orchestra.

 # Workflow
This is the workflow that happens when new music is acquired and is put online to be sold.
- There are 2 places where music pieces can be sold: AX3 website and Legato.
- There is business desire to have the opportunity to sell only on AX3 website and not Legato. Not clear whether the current system supports it.
## Preqrequisites: steps happening before our "X" team is involved
- the Purchasing team creates a new product or item on AS400, using a sequential number approach.
- There is a nightly job running on AS400 (name of the job? Who owns the job?) that copies data with the field "Royalties (Y/N)" set to "Y" to the Royatly System.
- The Editorial Team adds data for this new product in FileMaker (FM later).
- Travis (X team) manually checks the data and sometimes fixes issues (what exactly does he check?)
- Once Travis is done (how do people know he's done? What fields does he change? CUR?) the Product (Marketing?) Manager checks everything and if it's good the product goes online.
- There is a job that exports rom FM to SQL Server every night via a CSV file. (name of the job? Who owns the job?)
- A physical-digital product is ready to be sold on AX3 website when:
  + on FM field "OutputToWeb" is "JJ114".
  + on AS400 field "Status" is "CUR".
  + Digital rights int eh Royalty System are setup for the phyisical SKU.
- A digital-only product is ready to be sold on AX3 website when:
  + on FM field "OutputToWeb" is "DIGIONLY".
  + on AS400:
    - field "Price" is "USD".
    - field "Territoris" is Value (What does value mean?).
    - field "Status" is "PND".
  + Digital rights int eh Royalty System are setup for the DUMMY SKU.

- digital-only products end up in a different table in SQL Server (what are the names? )

## Template preparation
- music submission to Amadeus (print press) requires a specific format. There's an Excel spreadsheet for that. It's called the "Template".
- it's a spreadsheet where each product is on a row, with various attributes.
- insider a "performance set", the "Form number" is like a sequence number that indicates the "position" of that part in the set.
- the spreadsheet is filled manually
- then uploaded to WebCRD.
- the Template is not kept after that.
- Template differences for digital-only:
  + not many differences
  + it's still created, but not to upload to WebCRD.
  + lots of data in the template is not needed
- I have conflicting notes on the Template:
  + upload physical product to WebCRD
  + download the Template from WebCRD
  + scrube the Template

## AS400
- there are 3 different digital SKUs in AS400
    1. entire set. Prefix PK.
    2. score digital SKU. Don't know the prefix.
    3. part SKU.  Prefix PC.
- every purchase of individual parts is tracked to the same SKU in AS400. Not optimal.
- this is due to a limitation of number of characters (10? not sure) for the item number in AS400.


## Upload to portal.ax3.com

- PT code 95 (what is this?)
- download from FM (what?)
- digital-only: rename files that Travis got via Slack from the Editorial Team.
- physical-digital: rename files that Travis gets from AWS.


## Legato export
- generate XLS file from SQL Server (what is the name of the report?)
- Remove AK column
- upload to Legato
- get MRID unique identifier from Legato



