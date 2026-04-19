# Master Data Governance — TechNova Manufacturing Pvt. Ltd.

> **Document:** Master Data Governance Policy
> **Company Code:** `TN01` | **ERP Platform:** SAP ECC 6.0

---

## 1. Purpose

This document defines the governance framework for all SAP master data at TechNova Manufacturing Pvt. Ltd. It establishes ownership, naming conventions, change request procedures, data quality standards, and archiving policies across the three master data domains: Financial (FI), Procurement (MM), and Sales (SD).

Poor master data is the leading cause of SAP implementation failures. A single incorrect valuation class on a material master causes every goods movement for that material to post to the wrong GL account — silently, at scale, until the error is caught in a reconciliation.

---

## 2. Master Data Domains & Ownership

| Domain | Master Data Object | Primary Owner | Secondary Owner | SAP T-Code |
|---|---|---|---|---|
| FI | GL Account Master | Finance Controller | SAP Basis / IT | `FS00` |
| FI | Vendor Master (FI views) | Accounts Payable Manager | Procurement | `XK01` / `XK02` |
| FI | Customer Master (FI views) | Accounts Receivable Manager | Sales | `XD01` / `XD02` |
| MM | Material Master | Materials Manager | Warehouse / Finance | `MM01` / `MM02` |
| MM | Vendor Master (MM/Purchasing views) | Head of Procurement | AP Manager | `XK01` / `XK02` |
| MM | Purchasing Info Record | Purchasing Team | — | `ME11` / `ME12` |
| SD | Customer Master (SD views) | Sales Manager | AR Manager | `XD01` / `XD02` |
| SD | Pricing Condition Records | Sales Pricing Analyst | Sales Manager | `VK11` / `VK12` |
| SD | Credit Limits | Credit Manager | AR Manager | `FD32` |

---

## 3. Naming Conventions

### 3.1 Material Master

| Field | Convention | Example |
|---|---|---|
| Material Number | `MAT-XXXX` (system-assigned, internal) | `MAT-5001` |
| Material Description | `[Type] [Name] [Model/Spec]` | `Electronic Control Module ECM-500` |
| Material Group | Title case, max 20 chars | `Electronics`, `Raw Metal`, `MRO` |
| Batch Number | `B-YYYY-MM` (year-month of GR) | `B-2026-04` |

### 3.2 Vendor Master

| Field | Convention | Example |
|---|---|---|
| Vendor Code | `V-XXXX` (system-assigned) | `V-1001` |
| Vendor Name | Official registered company name | `Sigma Components Ltd.` |
| GSTIN | 15-character GST number — mandatory for DOME | `36AABCS1234F1ZX` |
| Account Group | `DOME` / `FORG` / `SERV` | `DOME` |

### 3.3 Customer Master

| Field | Convention | Example |
|---|---|---|
| Customer Code | `C-XXXX` (system-assigned) | `C-1001` |
| Customer Name | Official registered company name | `Electra Automation Pvt Ltd` |
| GSTIN | 15-character GST number — mandatory for domestic | `29AABCE1111A1ZA` |
| Account Group | `DOME` / `EXPG` / `DIST` | `DOME` |
| Credit Limit | In INR, set conservatively at onboarding | `₹25,00,000` |

### 3.4 GL Account Master

| Field | Convention | Example |
|---|---|---|
| GL Number | 6-digit per account group range | `130100` |
| Short Description | Concise, uppercase | `RAW MATERIAL INVENTORY` |
| Long Description | Full name with technical note | `Raw Material Inventory — BSX Val. Class 3000` |
| Account Group | `ASST` / `LIAB` / `EQTY` / `REVN` / `COGS` / `EXPN` | `ASST` |

---

## 4. Master Data Change Request (MDCR) Process

All changes to existing master data — except minor contact detail updates — must follow the MDCR process. No direct changes in PRD without an approved MDCR.

### 4.1 Change Process Flow

```
Requestor submits MDCR form
    │
    ▼
Domain Owner reviews (SLA: 2 business days)
    │
    ├── Rejected → Requestor notified with reason
    │
    └── Approved
         │
         ▼
IT / SAP team makes change in DEV
    │
    ▼
Finance Controller / Domain Owner validates in QAS
    │
    ▼
Change transported to PRD via STMS
    │
    ▼
MDCR closed — change log updated
```

### 4.2 Change Risk Levels

| Risk Level | Example Changes | Approval Required |
|---|---|---|
| **Low** | Contact details, email, phone on vendor/customer | Domain Owner only |
| **Medium** | Payment terms, credit limit, purchasing group | Domain Owner + Finance Controller |
| **High** | Valuation class, price control, reconciliation account, account group | Domain Owner + Finance Controller + CFO |
| **Critical** | GL account type, recon-type flag, OBYC/VKOA mappings | Domain Owner + Finance Controller + CFO + SAP Architect |

> **Critical Rule:** Changing a material's valuation class or price control (e.g., from `V` MAP to `S` Standard) requires full stock to be zero-valued first. This is a disruptive change requiring a weekend maintenance window.

---

## 5. Data Quality Standards

### 5.1 Vendor Master Quality Rules

| Field | Rule | Validation |
|---|---|---|
| GSTIN | Mandatory for all DOME vendors | 15-char format: `[2-digit state][10-char PAN][1-char entity][1Z][1-char check]` |
| Bank Details | IFSC + account number mandatory before first payment | Verified against vendor's cancelled cheque |
| Payment Terms | Must match agreed contract terms | Cross-checked with procurement contract |
| Reconciliation Account | Always `200100` for all vendor groups | Finance Controller verifies at creation |
| Duplicate Check | No two vendors with same GSTIN | System enforced via duplicate check configuration |

### 5.2 Material Master Quality Rules

| Field | Rule |
|---|---|
| Valuation Class | Must match material type: ROH→3000, HALB→7900, FERT→7920, HIBE→3010 |
| Price Control | ROH and HIBE → `V` (MAP); HALB and FERT → `S` (Standard Price) |
| MRP Type | ROH → `VB`; HALB/FERT → `PD`; HIBE → `ND` |
| Reorder Point | Mandatory for MRP Type VB (ROH materials) |
| Unit of Measure | Must match physical unit; cannot be changed once goods movements exist |

### 5.3 Customer Master Quality Rules

| Field | Rule |
|---|---|
| GSTIN | Mandatory for all DOME and DIST customers |
| Credit Limit | Set at onboarding based on credit assessment; reviewed annually |
| Reconciliation Account | Always `100300` for all customer groups |
| Sales Area | Must match the sales channel the customer actually orders through |
| Dunning Procedure | `TN01` assigned to all customers |

---

## 6. Master Data Audit & Review Calendar

| Activity | Frequency | Owner | T-Code / Method |
|---|---|---|---|
| Vendor master review — payment terms, bank details | Quarterly | AP Manager | `S_ALR_87012082` + manual review |
| Customer credit limit review | Annually (or after major order) | Credit Manager | `FD32` |
| Material master — price update (MAP / std cost) | Monthly (MAP auto-updates); Std price — annually | Materials Manager / Finance | `MR21` (price change) |
| Inactive vendor / customer review | Annually | Domain Owners | Flag for blocking (`XK05` / `XD05`) |
| GL account master review | Annually | Finance Controller | `FS00` mass display |
| GSTIN validity check | Annually | Finance / Tax Team | GST portal cross-check |

---

## 7. Master Data Archiving & Blocking

### 7.1 Blocking (Soft Delete)

When a vendor, customer, or material is no longer needed but has open history:

| Object | Block T-Code | Effect |
|---|---|---|
| Vendor | `XK05` | Blocked for new POs; existing open items unaffected |
| Customer | `XD05` | Blocked for new orders; open AR unaffected |
| Material | `MM06` (deletion flag) | No new GR/GI; existing stock must be cleared first |

### 7.2 Archiving

Full archival (physical removal from database) performed via `SARA` (Archive Administration). Prerequisites:
- All open documents (POs, SOs, invoices) fully closed
- No open stock or financial balances
- Retention period expired (minimum 7 years per Indian GST and Companies Act requirements)

---

> *All policies, names, and data in this document are fictitious and created for academic purposes only.*
