# TechNova Manufacturing Pvt. Ltd. — SAP ERP Implementation Blueprint

**Capstone Project | KIIT**

> A full-scope SAP ERP implementation blueprint for a fictitious Indian electronics manufacturer — covering enterprise structure design, module-wise SPRO configuration, master data, automatic account determination, and end-to-end business process documentation across SAP FI, MM, and SD.

---

## Table of Contents

1. [Company Profile](#1-company-profile)
2. [SAP System Landscape](#2-sap-system-landscape)
3. [Enterprise Structure Design](#3-enterprise-structure-design)
4. [SAP FI — Financial Accounting Configuration](#4-sap-fi--financial-accounting-configuration)
5. [SAP MM — Materials Management Configuration](#5-sap-mm--materials-management-configuration)
6. [SAP SD — Sales & Distribution Configuration](#6-sap-sd--sales--distribution-configuration)
7. [Automatic Account Determination (OBYC & VKOA)](#7-automatic-account-determination-obyc--vkoa)
8. [End-to-End Process Flows](#8-end-to-end-process-flows)
9. [Master Data](#9-master-data)
10. [Repository Structure](#10-repository-structure)
11. [Key T-Code Reference](#11-key-t-code-reference)
12. [Student Details](#12-student-details)

---

## 1. Company Profile

**TechNova Manufacturing Pvt. Ltd.** is a mid-sized Indian electronics components manufacturer headquartered in Bengaluru, Karnataka. It procures raw materials, manufactures electronic sub-assemblies, and distributes finished products to domestic B2B clients and Southeast Asian export markets.

| Attribute | Value |
|---|---|
| Company Name | TechNova Manufacturing Pvt. Ltd. |
| Industry | Electronics Manufacturing |
| Headquarters | Bengaluru, Karnataka, India |
| Annual Turnover | ₹120 Crore (approx.) |
| Employees | ~850 |
| ERP Platform | SAP ECC 6.0 (S/4HANA-compatible blueprint) |
| Company Code | `TN01` |
| Chart of Accounts | `TNCA` (6-digit GL account numbering) |
| Fiscal Year | `V3` — April 1 to March 31 (Indian FY) |
| Base Currency | INR |
| Tax Framework | GST (TAXIN procedure) — CGST, SGST, IGST |

---

## 2. SAP System Landscape

This blueprint is designed for a **three-tier SAP landscape**, which is the industry standard for any production SAP implementation:

```
┌─────────────────────────────────────────────────────────┐
│                  SAP SYSTEM LANDSCAPE                   │
├────────────────┬────────────────┬───────────────────────┤
│   DEV (100)    │   QAS (200)    │     PRD (300)         │
│  Development   │  Quality /     │   Production          │
│  & Config      │  Testing       │   (Live System)       │
│                │                │                       │
│  SPRO config   │  UAT / SIT     │  Go-Live              │
│  Sandbox       │  Regression    │  End-user operations  │
└────────────────┴────────────────┴───────────────────────┘
         │                │                  │
         └────────────────┴──────────────────┘
              Transport Management System (TMS)
              Transports move DEV → QAS → PRD
              via T-Code STMS
```

All configuration documented in this blueprint is performed in the **DEV client** via transaction `SPRO` (Implementation Guide / IMG), then transported to QAS for testing and finally to PRD.

---

## 3. Enterprise Structure Design

The SAP enterprise structure defines the legal, logistical, and commercial hierarchy of TechNova within the system. All subsequent configuration and master data depends on this foundation being set up first and in the correct sequence.

### 3.1 Complete Organizational Hierarchy

```
CLIENT: 100
└── COMPANY: TNOV (TechNova Group)
    └── COMPANY CODE: TN01 (TechNova Manufacturing Pvt. Ltd.)
        │
        ├── [FI]  Chart of Accounts : TNCA
        │         Fiscal Year Variant: V3 (Apr–Mar)
        │         Controlling Area  : TN01
        │         Business Areas    : TN10 / TN20 / TN30
        │
        ├── [MM]  Plant             : TN01 (Bengaluru Main)
        │         Plant             : TN02 (Hosur Warehouse)
        │         Storage Locs (TN01): SL01 / SL02 / SL03 / SL04
        │         Purchasing Org    : TN01
        │         Purchasing Groups : TNG1 / TNG2 / TNG3
        │
        └── [SD]  Sales Org         : TN01
                  Dist. Channels    : 10 (Direct) / 20 (Dealer) / 30 (Export)
                  Divisions         : 01 / 02 / 03
                  Shipping Point    : TNSP
```

### 3.2 Key Assignment T-Codes

Organizational units in SAP are non-functional until correctly assigned to each other. Wrong assignments cause master data and posting failures downstream.

| Assignment | T-Code | Direction |
|---|---|---|
| Company Code → Company | `OX16` | TN01 → TNOV |
| Plant → Company Code | `OX18` | TN01 → TN01 |
| Purchasing Org → Company Code | `OX01` | TN01 → TN01 |
| Purchasing Org → Plant | `OX17` | TN01 → TN01 |
| Sales Org → Company Code | `OVX3` | TN01 → TN01 |
| Distribution Channel → Sales Org | `OVXK` | 10/20/30 → TN01 |
| Division → Sales Org | `OVXA` | 01/02/03 → TN01 |
| Sales Area setup | `OVXG` | Combine all three |
| Shipping Point → Plant | `OVXD` | TNSP → TN01 |

> **Why this matters technically:** If Plant TN01 is not assigned to Company Code TN01 via `OX18`, goods movements posted in MM will fail to generate FI accounting documents — completely breaking the MM-FI integration at runtime.

---

## 4. SAP FI — Financial Accounting Configuration

All FI configuration is performed via `SPRO → Financial Accounting (New) → ...`

### 4.1 Company Code Global Settings (`OBY6`)

| Parameter | Value | Significance |
|---|---|---|
| Company Code | `TN01` | All FI documents post under this legal entity |
| Country | `IN` | Determines tax procedure (TAXIN) and date format |
| Currency | `INR` | All documents default to Indian Rupee |
| Chart of Accounts | `TNCA` | Links GL account master to this company code |
| Fiscal Year Variant | `V3` | April–March; 12 normal + 4 special periods |
| Field Status Variant | `TN01` | Controls required/optional fields on every posting |

### 4.2 Chart of Accounts — Account Groups (`OBD4`)

Account groups control the number range and field status for GL account creation via `FS00`.

| Account Group | From | To | Account Type |
|---|---|---|---|
| `ASST` | 100000 | 199999 | Assets |
| `LIAB` | 200000 | 299999 | Liabilities |
| `EQTY` | 300000 | 399999 | Equity / Capital |
| `REVN` | 400000 | 499999 | Revenue |
| `COGS` | 500000 | 549999 | Cost of Goods Sold |
| `EXPN` | 550000 | 699999 | Expenses |
| `TAXS` | 700000 | 799999 | Tax Accounts |
| `RECON` | 800000 | 899999 | Reconciliation / Clearing |

Retained Earnings account set via `OB53` → GL `300200`.

### 4.3 Document Types & Number Ranges (`OBA7`, `FBN1`)

Document types determine which account types can be combined in a posting and define the number range used for each document class — critical for audit trail integrity.

| Doc Type | Description | Account Types Allowed | Number Range |
|---|---|---|---|
| `SA` | GL Account Document | G | 1000000–1999999 |
| `KR` | Vendor Invoice | K, G | 5100000–5199999 |
| `KZ` | Vendor Payment | K, G | 5200000–5299999 |
| `DR` | Customer Invoice | D, G | 1800000–1899999 |
| `DZ` | Customer Payment | D, G | 1900000–1999999 |
| `WE` | Goods Receipt (auto from MM) | G | via MM number range |
| `RE` | Invoice Receipt (auto from MIRO) | K, G | via LIV number range |

### 4.4 Posting Keys (`OB41`)

Posting keys determine debit/credit direction and the account type for each line item in a document. Used in every manual FI transaction.

| PK | Debit/Credit | Account Type | Use Case |
|---|---|---|---|
| `40` | Debit | G (GL) | Debit a GL account |
| `50` | Credit | G (GL) | Credit a GL account |
| `01` | Debit | D (Customer) | Customer invoice line |
| `15` | Credit | D (Customer) | Incoming customer payment |
| `31` | Credit | K (Vendor) | Vendor invoice line |
| `25` | Debit | K (Vendor) | Outgoing vendor payment |

### 4.5 Accounts Payable (`OBD3`, `FBZP`)

- Vendor account groups: `DOME` (domestic), `FORG` (foreign/import), `SERV` (services) — defined via `OBD3`
- Reconciliation account: GL `200100` — Accounts Payable Control (account type `K`; system prevents direct manual postings)
- Payment terms via `OBB8`: `NET30`, `NET45`, `2/10NET30`, `IMMED`
- Automatic Payment Program (`FBZP`): paying company code TN01, payment method `T` (bank transfer), house bank `SBI01`, next payment run scheduled weekly (every Friday)

### 4.6 Accounts Receivable (`OBD2`, `FBMP`, `OB45`)

- Customer account groups: `DOME`, `EXPG`, `DIST` — defined via `OBD2`
- Reconciliation account: GL `100300` — Accounts Receivable Control (account type `D`)
- Dunning procedure `TN01` via `FBMP`: 3 levels — Reminder (Day 5) → Warning (Day 15) → Legal Notice (Day 30)
- Credit Control Area `TN01` linked to Company Code via `OB45`

### 4.7 GST Configuration (`OBYZ`, `OBBG`)

India uses the `TAXIN` tax procedure (condition-based). Assigned to country `IN` via `OBBG`.

| Tax Code | Type | Rate | Input GL | Output GL |
|---|---|---|---|---|
| `I1` | Input GST | 5% | 160100 / 160200 | — |
| `I3` | Input GST (IGST) | 18% | 160300 | — |
| `O3` | Output GST (IGST) | 18% | — | 210300 |

Tax jurisdiction is automatic: intra-state transactions → CGST + SGST; inter-state / export → IGST.

---

## 5. SAP MM — Materials Management Configuration

### 5.1 Material Types & Price Control (`OMS2`)

Material type controls which master data views are available, which movement types are permitted, and how stock value is managed.

| Material Type | Description | Price Control | Valuation Class | Rationale |
|---|---|---|---|---|
| `ROH` | Raw Materials | `V` Moving Average | `3000` | Prices fluctuate with market; MAP self-adjusts |
| `HALB` | Semi-Finished | `S` Standard Price | `7900` | Stable internal cost; variances go to PRD account |
| `FERT` | Finished Products | `S` Standard Price | `7920` | COGS must be predictable for margin analysis |
| `HIBE` | MRO / Consumables | `V` Moving Average | `3010` | Low-value, no standard cost needed |

### 5.2 Automatic Account Determination — `OBYC`

This is the most technically critical MM configuration step. Every goods movement posts to FI automatically — the GL account is determined by the combination of **transaction key + valuation class** set up in `OBYC`.

| Transaction Key | Triggered By | Debit Account | Credit Account |
|---|---|---|---|
| `BSX` | GR (101), GI reversal | Inventory (130100) | — |
| `WRX` | GR (101) | — | GR/IR Clearing (200200) |
| `GBB / VBR` | GI to Cost Center (201) | Consumption (500100) | — |
| `GBB / VAX` | GI for Delivery (601) | COGS (500100) | — |
| `PRD` | Price difference (std vs. actual) | Price Diff. (550100) | — |
| `AKO` | Scrapping (551) | Scrap Loss | Inventory |

> **Technically:** When MIGO posts GR with movement type `101` for material `TN-RM-001` (valuation class `3000`), SAP looks up `OBYC → BSX → Chart of Accounts TNCA → Valuation Class 3000 → GL 130100`. This happens in milliseconds with no human intervention.

### 5.3 MRP Types

| MRP Type | Planning Logic | Materials |
|---|---|---|
| `VB` | Reorder Point — triggers PR when stock < reorder point | ROH (raw materials) |
| `PD` | MRP — demand-driven from production/sales orders | HALB, FERT |
| `ND` | No MRP — manual procurement only | MRO/HIBE |

MRP run via `MD01` (plant-wide) or `MD02` (single item). Output: planned orders → converted to PRs (external procurement) or production orders (in-house).

### 5.4 Purchase Order Release Strategy (`OME9`, `OMGQ`)

Release strategies enforce approval workflows before a PO can be transmitted to a vendor.

| Release Code | Value Threshold | Approver Role |
|---|---|---|
| `L1` | Up to ₹50,000 | Purchase Manager |
| `L2` | ₹50,001 – ₹5,00,000 | Head of Procurement |
| `L3` | Above ₹5,00,000 | CFO / Director |

Configured using characteristic `GNETWR` (Net Order Value). Without all required release codes, the PO output (NEU) cannot be triggered and the vendor is not notified.

### 5.5 Movement Types (`OMJJ`)

| Mvt Type | Description | Stock Effect | FI Posting Keys |
|---|---|---|---|
| `101` | GR against PO → Unrestricted | ↑ Unrestricted | BSX Dr / WRX Cr |
| `103` | GR → Blocked Stock | ↑ Blocked | No FI |
| `105` | Release Blocked → Unrestricted | Reclassify | BSX Dr / WRX Cr |
| `201` | GI to Cost Center | ↓ Unrestricted | GBB/VBR Dr / BSX Cr |
| `261` | GI to Production Order | ↓ Unrestricted | GBB/VBR Dr / BSX Cr |
| `311` | SLoc-to-SLoc Transfer | Reclassify | No FI |
| `601` | GI for Outbound Delivery (SD) | ↓ Unrestricted | GBB/VAX Dr / BSX Cr |
| `551` | Scrapping | ↓ Unrestricted | AKO |

### 5.6 3-Way Match in MIRO

`MIRO` compares three documents before allowing invoice posting. Tolerance keys defined in `OMR6`:

| Tolerance Key | What It Checks | Configured Limit |
|---|---|---|
| `BD` | Small absolute differences (auto-clear) | ±₹500 |
| `DW` | Quantity variance (invoice qty vs GR qty) | ±2% |
| `PP` | Price variance (invoice price vs PO price) | ±2% |

Variance beyond tolerance → invoice auto-blocked → must be manually released via `MRBR` by AP supervisor.

---

## 6. SAP SD — Sales & Distribution Configuration

### 6.1 Sales Areas

A Sales Area is the unique combination of Sales Org + Distribution Channel + Division. Every customer master record, pricing condition, and sales document is tied to a specific sales area.

| Sales Area | Sales Org | D.Ch | Div | Business Use |
|---|---|---|---|---|
| `TN01-10-01` | TN01 | 10 Direct | 01 Components | Domestic B2B component sales |
| `TN01-20-01` | TN01 | 20 Dealer | 01 Components | Dealer/distributor network |
| `TN01-30-02` | TN01 | 30 Export | 02 Sub-Assy | Southeast Asia exports |
| `TN01-10-03` | TN01 | 10 Direct | 03 Finished | Direct finished product sales |

### 6.2 Pricing Procedure `TNPRC` (`V/08`)

The pricing procedure defines the exact calculation sequence from base price to final invoice total.

| Step | Condition Type | Description | Base | Calculation |
|---|---|---|---|---|
| 10 | `PR00` | Base Price | — | Manual / info record / price list |
| 20 | `K004` | Customer Discount | Step 10 | % deduction |
| 30 | `K007` | Customer Group Discount | Step 10 | % deduction |
| 40 | — | **Net Price** (subtotal) | — | Steps 10–30 |
| 50 | `MWST` | Output GST 18% | Step 40 | % on net |
| 60 | `VPRS` | Cost (Std Price / MAP) | — | **Statistical only** — not in invoice |
| 70 | — | **Grand Total** | — | Step 40 + Step 50 |

`VPRS` is statistical — it pulls the material's standard or moving average cost into the order for margin visibility without affecting the customer-facing price. Step 60 (VPRS) subtracted from Step 40 (Net Price) gives the **contribution margin** per order line.

Pricing procedure is determined via `OVKK` by the combination of: Sales Area + Customer Pricing Procedure (customer master field) + Document Pricing Procedure (sales doc type field).

### 6.3 Revenue Account Determination — `VKOA`

`VKOA` is the SD equivalent of `OBYC`. It maps billing line items to specific GL revenue accounts automatically when `VF01` is run.

```
Key: Application V + Table 001
     Sales Org (TN01) + Dist. Channel + Div + Cust. AAG + Mat. AAG
     → GL Account
```

| Customer AAG | Material AAG | GL | Description |
|---|---|---|---|
| `01` Domestic | `01` Components | `400100` | Domestic Sales Revenue |
| `02` Export | `01` Components | `400200` | Export Sales Revenue |
| `01` Domestic | `02` Sub-Assy | `400100` | Domestic Sales Revenue |

### 6.4 Credit Management (`OB45`, `FD32`, `OVA8`)

- Credit Control Area `TN01` assigned to Company Code TN01 via `OB45`
- Static credit check at sales order save (`OVA8`): if open AR + current order value > credit limit → delivery block `01` applied automatically
- Individual credit limits managed in `FD32` per customer
- Blocked orders reviewed and released by credit manager via `VKM1`

---

## 7. Automatic Account Determination (OBYC & VKOA)

This is the core of SAP's integration power. When configured correctly, **no manual journal entries** are needed for any routine goods movement or billing transaction.

### Complete GL Posting Map

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PROCUREMENT (MM → FI)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Business Event          Dr GL              Cr GL
────────────────────────────────────────────────────────────
GR (Mvt 101)            130100 (Stock)     200200 (GR/IR)
Invoice MIRO            200200 (GR/IR)     200100 (AP)
                        160300 (Tax In)
Vendor Payment F110     200100 (AP)        100200 (Bank)
GI to Production        500100 (COGS)      130100 (Stock)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SALES (SD → FI)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Business Event          Dr GL              Cr GL
────────────────────────────────────────────────────────────
PGI Mvt 601             500100 (COGS)      130300 (FG Stock)
Billing VF01            100300 (AR)        400100 (Revenue)
                                           210300 (GST Out)
Customer Payment F-28   100200 (Bank)      100300 (AR)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### How the GR/IR Clearing Account Works

GL `200200` (GR/IR) is a **bridging liability** that reconciles the timing gap between goods receipt and invoice receipt:

```
Step 1 — Goods Received (MIGO):
  Dr 130100 Raw Material Inventory    ₹29,000
  Cr 200200 GR/IR Clearing            ₹29,000
  (Stock is in the warehouse; no invoice yet)

Step 2 — Invoice Received (MIRO):
  Dr 200200 GR/IR Clearing            ₹29,000  ← cleared
  Dr 160300 GST Input IGST            ₹5,220
  Cr 200100 Accounts Payable Control  ₹34,220
  (Liability transferred to vendor; GR/IR = zero)

Step 3 — Payment (F110):
  Dr 200100 Accounts Payable Control  ₹34,220
  Cr 100200 Bank — SBI Current        ₹34,220
  (Vendor fully settled)
```

Any remaining balance in `200200` at month-end = unmatched GR or invoice. Cleared and analyzed via `MR11`.

---

## 8. End-to-End Process Flows

### 8.1 Procure-to-Pay (P2P)

```
[ME51N] Purchase Requisition (PR)
    ↓  Released via ME54N (Purchase Manager)
[ME21N] Purchase Order ──→ Vendor notified (Output type NEU)
    ↓  Released via ME29N (Head of Procurement if > ₹50,000)
[MIGO / 101] Goods Receipt
    FI auto-post: Dr 130100 / Cr 200200
    MM: Stock ↑ in SL01 (Raw Material Store)
    ↓
[MIRO] Invoice Verification — 3-Way Match: PO + GR + Invoice
    FI auto-post: Dr 200200 + Dr 160300 / Cr 200100
    ↓  If blocked (variance > tolerance) → MRBR to release
[F110] Automatic Payment Run (weekly)
    FI auto-post: Dr 200100 / Cr 100200
    Output: NEFT/RTGS bank transfer advice
```

**Sample transaction** (from `06_Process_Flows/p2p_transaction_log.xlsx`):

| Document | Number | Value |
|---|---|---|
| Purchase Order | `4500000001` | ₹34,220 incl. GST (200 PCB boards from Arya Electronics) |
| Goods Receipt | `5000000001` | ₹29,000 stock value posted |
| Invoice | `5100000001` | ₹34,220 — 3-way matched and posted |
| Payment | `P-2025-0301` | ₹34,220 NEFT cleared |

### 8.2 Order-to-Cash (O2C)

```
[VA21] Quotation ──→ Customer (Output AN00, 30-day validity)
    ↓  Customer confirms
[VA01] Sales Order ──→ Auto: Availability Check + Credit Check (FD32)
    Output BA00: Order Confirmation emailed to customer
    ↓  Availability confirmed
[VL01N] Outbound Delivery document created
    ↓
[LT0A / VL02N] Picking from SL03 (Finished Goods Store)
    ↓
[VL02N → Post Goods Issue] Movement Type 601
    FI auto-post: Dr 500100 / Cr 130300
    MM: FG Stock ↓
    ↓
[VF01] Billing Document (Type F2 — Tax Invoice)
    FI auto-post: Dr 100300 / Cr 400100 + Cr 210300
    Output RD00: GST-compliant tax invoice emailed to customer
    ↓
[F-28] Incoming Payment posted
    FI: Dr 100200 / Cr 100300 — AR open item cleared
```

**SAP Document Flow** (visible via `VA03 → Environment → Document Flow`):

```
Quotation       20000001
  └── Sales Order     30000001
        └── Delivery        80000001
              └── Billing Document  90000001
                    └── FI-DR Document   1800000001
                          └── FI-DZ Payment  1900000001
```

### 8.3 Record-to-Report (R2R) — Month-End Close Sequence

| Day | Activity | T-Code | Owner |
|---|---|---|---|
| D-5 | Freeze MM — complete all GRs for the period | `OB52` | MM Team |
| D-4 | Complete all SD billing — run billing due list | `VF04` | SD / Finance |
| D-3 | Run asset depreciation | `AFAB` | Fixed Assets |
| D-2 | Post accruals (salary, utilities, warranty) | `FB50` | GL Team |
| D-2 | Clear GR/IR account, investigate aged items | `MR11` / `F.13` | AP Team |
| D-1 | Foreign currency revaluation (USD vendors/customers) | `FAGL_FC_VAL` | Treasury |
| D-1 | Reconcile AP/AR subledgers vs GL control accounts | `S_ALR_87012082` | Finance |
| D-0 | Close posting period for account types A/D/K/S | `OB52` | Controller |
| D+1 | Review Trial Balance — investigate unusual balances | `S_ALR_87012277` | CFO |
| D+3 | Issue P&L and Balance Sheet | `S_ALR_87012284` | Finance |

---

## 9. Master Data

### 9.1 Chart of Accounts — Key GL Accounts (`02_FI_Configuration/chart_of_accounts.xlsx`)

35 accounts across 8 groups. Highlighted accounts and their technical role:

| GL | Description | Technical Role |
|---|---|---|
| `100300` | Accounts Receivable Control | Recon. type D — fed by SD billing via VKOA |
| `130100` | Raw Material Inventory | BSX target for ROH valuation class 3000 |
| `130300` | Finished Goods Inventory | BSX target for FERT valuation class 7920 |
| `200100` | Accounts Payable Control | Recon. type K — fed by MIRO and F110 |
| `200200` | GR/IR Clearing | WRX — must net to zero by month-end |
| `210300` | GST Output IGST Payable | Settled monthly via GSTR-3B filing |
| `400100` | Domestic Sales Revenue | VKOA target for domestic customer AAG `01` |
| `500100` | Cost of Goods Sold | GBB/VAX at PGI (601) and GBB/VBR at GI (261) |
| `550100` | Price Difference | PRD — captures std cost vs. actual GR variance |

### 9.2 Vendor Master — 15 Vendors (`03_MM_Configuration/vendor_master_data.xlsx`)

- `DOME` (10 domestic): all with valid GSTIN, payment terms NET30/NET45, reconciliation account `200100`
- `FORG` (3 import): USD currency, import pricing, IGST applicable at GR
- `SERV` (2 service): no GR-based IV; direct invoice posting

### 9.3 Material Master — 20 Materials (`03_MM_Configuration/material_master_data.xlsx`)

| Type | Count | Key Fields Configured |
|---|---|---|
| ROH | 10 | Valuation class 3000, price control V, MRP type VB, reorder points, GR processing time |
| HALB | 4 | Valuation class 7900, standard price S, MRP type PD, in-house production time |
| FERT | 4 | Valuation class 7920, standard price S, MRP type PD, sales views configured |
| HIBE | 2 | Valuation class 3010, price control V, MRP type ND |

### 9.4 Customer Master — 15 Customers (`04_SD_Configuration/customer_master_data.xlsx`)

- `DOME` (10): credit limits ₹8L–₹50L, payment terms NET30/NET45, customer AAG `01`
- `EXPG` (3): Singapore, Indonesia, Malaysia — IGST applicable, USD pricing in some cases
- `DIST` (2): National distributors with highest credit limits (₹55L–₹60L), dealer discount group

---

## 10. Repository Structure

```
TechNova-SAP-Blueprint/
│
├── 01_Company_Structure/
│   ├── org_structure.md              # Full SAP org hierarchy with assignment T-codes
│   └── org_units_summary.md          # Quick-reference: all 27 org units in one table
│
├── 02_FI_Configuration/
│   ├── fi_config_checklist.md        # 10-phase FI SPRO configuration guide
│   └── chart_of_accounts.xlsx        # 35 GL accounts — color-coded by account type
│
├── 03_MM_Configuration/
│   ├── mm_config_checklist.md        # 6-phase MM SPRO configuration guide
│   ├── material_master_data.xlsx     # 20 materials — all views, MRP, valuation
│   └── vendor_master_data.xlsx       # 15 vendors — GSTIN, payment terms, account groups
│
├── 04_SD_Configuration/
│   ├── sd_config_checklist.md        # 8-phase SD SPRO configuration guide
│   └── customer_master_data.xlsx     # 15 customers — credit limits, sales areas
│
├── 05_Integration/
│   └── cross_module_integration.md   # OBYC + VKOA mapping; full GL posting reference
│
├── 06_Process_Flows/
│   ├── p2p_process_flow.md           # P2P — step-by-step with FI postings & 3-way match
│   ├── o2c_process_flow.md           # O2C — document flow chain with T-codes
│   ├── r2r_process_flow.md           # R2R — month-end close calendar & year-end steps
│   ├── p2p_transaction_log.xlsx      # 7 sample P2P transactions with status tracking
│   └── o2c_transaction_log.xlsx      # 7 sample O2C transactions with GST breakdown
│
├── 07_Master_Data/
│   └── master_data_governance.md     # Ownership, naming conventions, change process
│
└── README.md
```

---

## 11. Key T-Code Reference

### Global / SPRO
| T-Code | Purpose |
|---|---|
| `SPRO` | SAP Implementation Guide — all configuration entry point |
| `STMS` | Transport Management System — move config DEV → QAS → PRD |
| `OB52` | Open / Close Posting Periods |
| `OBY6` | Company Code Global Settings |

### SAP FI
| T-Code | Purpose |
|---|---|
| `OB13` | Create Chart of Accounts |
| `OBD4` | Define Account Groups |
| `FS00` | Create / Maintain GL Account Master |
| `OB29` | Define Fiscal Year Variant |
| `OBA7` | Define Document Types |
| `FBN1` | Maintain FI Document Number Ranges |
| `OB41` | Define Posting Keys |
| `OBD3` | Define Vendor Account Groups |
| `OBD2` | Define Customer Account Groups |
| `FBZP` | Configure Automatic Payment Program |
| `FBMP` | Configure Dunning |
| `OB45` | Define Credit Control Area |
| `OBYZ` | Maintain Tax Procedure |
| `FB50` | Enter GL Document (manual journal) |
| `F110` | Execute Automatic Payment Run |
| `F-28` | Post Incoming Customer Payment |
| `AFAB` | Run Asset Depreciation |
| `FAGL_FC_VAL` | Foreign Currency Revaluation |
| `F.13` | Automatic Clearing of Open Items |
| `MR11` | Maintain / Clear GR/IR Account |
| `S_ALR_87012277` | Trial Balance Report |
| `S_ALR_87012284` | Balance Sheet & P&L Statement |
| `S_ALR_87012082` | Vendor Balance List (AP Reconciliation) |
| `S_ALR_87012173` | Customer Balance List (AR Reconciliation) |

### SAP MM
| T-Code | Purpose |
|---|---|
| `MM01` | Create Material Master |
| `XK01` | Create Vendor Master (Centrally) |
| `ME11` | Create Purchasing Info Record |
| `OBYC` | Configure Automatic Account Determination |
| `OMR6` | Define Invoice Verification Tolerances |
| `ME51N` | Create Purchase Requisition |
| `ME54N` | Release Purchase Requisition |
| `ME21N` | Create Purchase Order |
| `ME29N` | Release Purchase Order |
| `MIGO` | Goods Receipt / Issue / Transfer Posting |
| `MIRO` | Logistics Invoice Verification (3-way match) |
| `MRBR` | Release Payment-Blocked Invoices |
| `MD01` | MRP — Total Planning Run |
| `MMBE` | Stock Overview by Material/Plant/SLoc |
| `MB52` | Warehouse Stocks Report |

### SAP SD

| T-Code | Purpose |
|---|---|
| `XD01` | Create Customer Master (Centrally) |
| `VK11` | Create Pricing Condition Records |
| `V/08` | Maintain Pricing Procedure |
| `OVKK` | Assign Pricing Procedure to Sales Area |
| `VKOA` | Revenue Account Determination |
| `VA01` | Create Sales Order |
| `VA03` | Display Sales Order + Document Flow |
| `VL01N` | Create Outbound Delivery |
| `VL02N` | Change Delivery / Post Goods Issue |
| `VF01` | Create Billing Document |
| `VF04` | Billing Due List (bulk billing) |
| `FD32` | Maintain Customer Credit Limit |
| `VKM1` | Release Credit-Blocked Sales Orders |
| `VA05` | List of Open Sales Orders |

---

> *All company names, GSTIN numbers, vendor/customer records, and financial figures in this repository are entirely fictitious and created solely for academic purposes.*