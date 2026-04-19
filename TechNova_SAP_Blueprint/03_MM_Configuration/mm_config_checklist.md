# SAP MM Configuration Checklist — TechNova Manufacturing Pvt. Ltd.

> **Module:** Materials Management — MM-PUR, MM-IM, MM-IV, MM-MRP
> **SPRO Path:** `SPRO → Materials Management → ...`
> **Company Code:** `TN01` | **Plant:** `TN01` (Bengaluru Main)

---

## Phase 1 — Enterprise Structure for MM

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 1.1 | Define Plant | `OX10` | `TN01` — Bengaluru Main Plant; `TN02` — Hosur Warehouse | ☐ |
| 1.2 | Assign Plant to Company Code | `OX18` | TN01 → TN01; TN02 → TN01 | ☐ |
| 1.3 | Define Storage Locations | `OX09` | SL01 / SL02 / SL03 / SL04 for Plant TN01 | ☐ |
| 1.4 | Define Purchasing Organisation | `OX08` | `TN01` — Central Purchasing | ☐ |
| 1.5 | Assign Purchasing Org to Company Code | `OX01` | TN01 → TN01 | ☐ |
| 1.6 | Assign Purchasing Org to Plant | `OX17` | TN01 → TN01 / TN02 | ☐ |
| 1.7 | Define Purchasing Groups | `OME4` | TNG1 (Raw Mat.), TNG2 (Packaging), TNG3 (Capital & Services) | ☐ |

---

## Phase 2 — Material Master Configuration

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 2.1 | Define Material Types | `OMS2` | ROH, HALB, FERT, HIBE — configure views, quantity/value updating | ☐ |
| 2.2 | Define Number Ranges for Materials | `MMNR` | Internal range for each material type | ☐ |
| 2.3 | Define Material Groups | `OMSK` | Electronics, Raw Metal, MRO, Sub-Assembly, Finished Goods | ☐ |
| 2.4 | Define Units of Measure | `CUNI` | EA (each), KG, LT, M | ☐ |
| 2.5 | Define Valuation Classes | `OMSK` | 3000 (ROH), 7900 (HALB), 7920 (FERT), 3010 (HIBE) | ☐ |
| 2.6 | Create Material Master Records | `MM01` | 20 materials per `material_master_data.xlsx` | ☐ |

**Material Type Summary:**

| Material Type | Description | Price Control | Valuation Class | MRP Type |
|---|---|---|---|---|
| `ROH` | Raw Materials | `V` Moving Average | `3000` | `VB` Reorder Point |
| `HALB` | Semi-Finished | `S` Standard Price | `7900` | `PD` MRP |
| `FERT` | Finished Products | `S` Standard Price | `7920` | `PD` MRP |
| `HIBE` | MRO / Consumables | `V` Moving Average | `3010` | `ND` No MRP |

---

## Phase 3 — Vendor Master & Purchasing Configuration

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 3.1 | Define Vendor Account Groups | `OBD3` | `DOME`, `FORG`, `SERV` (same as FI-AP configuration) | ☐ |
| 3.2 | Create Vendor Master Records | `XK01` | 15 vendors per `vendor_master_data.xlsx` | ☐ |
| 3.3 | Define Document Types for PO | `OMEC` | `NB` — Standard PO; `UB` — Stock Transport Order | ☐ |
| 3.4 | Define Number Ranges for PO | `OMH6` | PO range: 4500000000–4599999999 | ☐ |
| 3.5 | Define Number Ranges for PR | `OMH7` | PR range: 1000000000–1099999999 | ☐ |
| 3.6 | Create Purchasing Info Records | `ME11` | Price / vendor-material combination records | ☐ |
| 3.7 | Define Message Output Types | `MN04` | `NEU` — PO output to vendor (email/print) | ☐ |

---

## Phase 4 — Release Strategy for Purchase Orders

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 4.1 | Define Characteristics for Release | `CT04` | Characteristic `GNETWR` — Net Order Value (INR) | ☐ |
| 4.2 | Define Release Groups | `OME9` | Group `01` — Standard PO release | ☐ |
| 4.3 | Define Release Codes | `OME9` | `L1` (Purch. Manager), `L2` (Head of Procurement), `L3` (CFO) | ☐ |
| 4.4 | Define Release Strategies | `OME9` | Strategy 1: ≤₹50K → L1 only; Strategy 2: ₹50K–₹5L → L1+L2; Strategy 3: >₹5L → L1+L2+L3 | ☐ |
| 4.5 | Assign Release Strategies | `OMGQ` | Assign to PO document type NB | ☐ |

**Release Strategy Table:**

| Release Code | Value Threshold | Approver Role | T-Code |
|---|---|---|---|
| `L1` | Up to ₹50,000 | Purchase Manager | `ME54N` |
| `L2` | ₹50,001 – ₹5,00,000 | Head of Procurement | `ME29N` |
| `L3` | Above ₹5,00,000 | CFO / Director | `ME29N` |

---

## Phase 5 — Inventory Management & Automatic Account Determination

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 5.1 | Define Movement Types | `OMJJ` | 101, 103, 105, 201, 261, 311, 601, 551 | ☐ |
| 5.2 | Configure Automatic Account Determination | `OBYC` | Transaction keys: BSX, WRX, GBB/VBR, GBB/VAX, PRD, AKO | ☐ |
| 5.3 | Define Number Ranges for GR Documents | `OMBT` | GR range: 5000000000–5099999999 | ☐ |
| 5.4 | Define Tolerance Limits for GR | `OMCO` | Under-delivery / over-delivery tolerances per vendor | ☐ |

**OBYC Automatic Account Determination:**

| Transaction Key | Triggered By | Valuation Class | Debit GL | Credit GL |
|---|---|---|---|---|
| `BSX` | GR (101) | 3000 / 7920 | 130100 / 130300 | — |
| `WRX` | GR (101) | Any | — | 200200 |
| `GBB / VBR` | GI to Cost Centre (201) | 3000 | 500100 | — |
| `GBB / VAX` | GI for Delivery (601) | 7920 | 500100 | — |
| `PRD` | Price difference at GR | 3000 | 550100 | — |
| `AKO` | Scrapping (551) | Any | 550400 | 130100 |

**Movement Types:**

| Mvt Type | Description | Stock Effect | FI Posting |
|---|---|---|---|
| `101` | GR against PO → Unrestricted | ↑ Unrestricted | BSX Dr / WRX Cr |
| `103` | GR → Blocked Stock | ↑ Blocked | No FI |
| `105` | Release Blocked → Unrestricted | Reclassify | BSX Dr / WRX Cr |
| `201` | GI to Cost Centre | ↓ Unrestricted | GBB/VBR Dr / BSX Cr |
| `261` | GI to Production Order | ↓ Unrestricted | GBB/VBR Dr / BSX Cr |
| `311` | SLoc-to-SLoc Transfer | Reclassify | No FI |
| `601` | GI for Outbound Delivery (SD) | ↓ Unrestricted | GBB/VAX Dr / BSX Cr |
| `551` | Scrapping | ↓ Unrestricted | AKO |

---

## Phase 6 — Invoice Verification (MM-IV / MIRO)

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 6.1 | Configure Logistics Invoice Verification | `SPRO` | Activate GR-based invoice verification | ☐ |
| 6.2 | Define Tolerance Keys | `OMR6` | `BD` ±₹500, `DW` ±2%, `PP` ±2% | ☐ |
| 6.3 | Define Number Ranges for LIV Documents | `OMRM` | Invoice range: 5100000–5199999 | ☐ |
| 6.4 | Configure ERS (Evaluated Receipt Settlement) | `MRRL` | Optional — for GR-based auto-invoice for select vendors | ☐ |
| 6.5 | Block Invoice for Variance | `MIRO` | System auto-blocks if variance > tolerance keys | ☐ |
| 6.6 | Release Blocked Invoices | `MRBR` | AP Supervisor manually releases after review | ☐ |

**Invoice Tolerance Keys:**

| Tolerance Key | What It Checks | Configured Limit | Action if Exceeded |
|---|---|---|---|
| `BD` | Small absolute difference | ±₹500 | Auto-cleared |
| `DW` | Qty variance (invoice qty vs GR qty) | ±2% | Invoice blocked — MRBR |
| `PP` | Price variance (invoice price vs PO price) | ±2% | Invoice blocked — MRBR |

**3-Way Match Logic (MIRO):**

```
PO (ME21N)  ─┐
              ├──► MIRO compares all three ──► Match: Post invoice
GR (MIGO)  ──┤                               ──► Mismatch > tolerance: Block
              │
Invoice (Vendor) ─┘
```

---

> *Configuration performed in DEV (Client 100) and transported to QAS → PRD via `STMS`.*
> *All data is fictitious and for academic purposes only.*
