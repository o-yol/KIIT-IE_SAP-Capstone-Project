# Cross-Module Integration — FI, MM & SD

> **TechNova Manufacturing Pvt. Ltd.**
> This document covers the automatic account determination configuration (`OBYC` for MM and `VKOA` for SD) and the complete GL posting reference for every business event across the P2P and O2C cycles.

---

## 1. Overview — How SAP Integration Works

SAP's integration power comes from **automatic document generation**: every goods movement in MM and every billing event in SD triggers a real-time FI accounting document with zero manual journal entries. This is achieved through two configuration tables:

| Config Table | Module | Purpose | T-Code |
|---|---|---|---|
| `OBYC` | MM → FI | Maps goods movement + valuation class → GL accounts | `OBYC` |
| `VKOA` | SD → FI | Maps billing + customer/material account assignment group → GL revenue account | `VKOA` |

---

## 2. OBYC — MM Automatic Account Determination

### 2.1 How SAP Determines the GL Account at MIGO

When a goods movement is posted in `MIGO`, SAP executes this lookup in milliseconds:

```
MIGO Post (Mvt 101)
    │
    ▼
What is the Transaction Key?  ──► BSX (stock posting)
    │
    ▼
What is the Chart of Accounts? ──► TNCA
    │
    ▼
What is the Valuation Class?   ──► 3000 (ROH)
    │
    ▼
OBYC Table Lookup: BSX + TNCA + 3000 ──► GL 130100 (Raw Material Inventory)
    │
    ▼
FI Document auto-posted: Dr 130100 / Cr 200200
```

### 2.2 Complete OBYC Configuration

| Transaction Key | Business Event | Valuation Class | Debit GL | Debit Description | Credit GL | Credit Description |
|---|---|---|---|---|---|---|
| `BSX` | GR against PO (Mvt 101) — ROH | 3000 | 130100 | Raw Material Inventory | — | — |
| `BSX` | GR against PO (Mvt 101) — HALB | 7900 | 130200 | Semi-Finished Inventory | — | — |
| `BSX` | GR against PO (Mvt 101) — FERT | 7920 | 130300 | Finished Goods Inventory | — | — |
| `BSX` | GR against PO (Mvt 101) — HIBE | 3010 | 130400 | MRO Inventory | — | — |
| `WRX` | GR against PO (Mvt 101) — GR/IR offset | Any | — | — | 200200 | GR/IR Clearing |
| `GBB / VBR` | GI to Cost Centre (Mvt 201) | 3000 | 500100 | Consumption / COGS | — | — |
| `GBB / VAX` | GI for Outbound Delivery (Mvt 601) | 7920 | 500100 | Cost of Goods Sold | — | — |
| `PRD` | Price difference at GR (std cost vs actual) | Any | 550100 | Price Difference | — | — |
| `AKO` | Scrapping (Mvt 551) | Any | 550400 | Scrap Loss | 130100 | Raw Material Inventory |

### 2.3 Valuation Class Reference

| Material Type | Valuation Class | Key | Inventory GL |
|---|---|---|---|
| ROH — Raw Materials | 3000 | Moving average price | 130100 |
| HALB — Semi-Finished | 7900 | Standard price | 130200 |
| FERT — Finished Goods | 7920 | Standard price | 130300 |
| HIBE — MRO | 3010 | Moving average price | 130400 |

---

## 3. VKOA — SD Revenue Account Determination

### 3.1 How SAP Determines the Revenue GL at VF01

When `VF01` (billing) is run:

```
VF01 — Create Billing Document
    │
    ▼
VKOA Table Lookup:
    Application = V
    Table       = 001
    Sales Org   = TN01
    D.Channel   = 10 / 20 / 30
    Division    = 01 / 02 / 03
    Cust. AAG   = 01 (Domestic) / 02 (Export) / 03 (Distributor)
    Mat. AAG    = 01 (Components) / 02 (Sub-Assy)
    │
    ▼
GL Revenue Account ──► 400100 / 400200 / 400300
    │
    ▼
FI Document auto-posted:
    Dr 100300 (AR Control)
    Cr 400100 (Revenue)
    Cr 210300 (GST Output IGST)
```

### 3.2 Complete VKOA Configuration Table

| Sales Org | D.Ch | Div | Customer AAG | Material AAG | Revenue GL | GL Description |
|---|---|---|---|---|---|---|
| TN01 | 10 | 01 | `01` Domestic | `01` Components | `400100` | Domestic Sales Revenue |
| TN01 | 20 | 01 | `01` Domestic | `01` Components | `400100` | Domestic Sales Revenue |
| TN01 | 10 | 03 | `01` Domestic | `02` Sub-Assy | `400100` | Domestic Sales Revenue |
| TN01 | 30 | 02 | `02` Export | `01` Components | `400200` | Export Sales Revenue |
| TN01 | 20 | 01 | `03` Distributor | `01` Components | `400300` | Dealer Sales Revenue |

---

## 4. Complete GL Posting Reference

### 4.1 Procure-to-Pay (MM → FI)

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EVENT: Goods Receipt — MIGO (Movement Type 101)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Dr  130100  Raw Material Inventory        ₹29,000
Cr  200200  GR/IR Clearing Account        ₹29,000

OBYC Key: BSX (Dr) + WRX (Cr)
FI Document Type: WE (auto-generated)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EVENT: Invoice Verification — MIRO (3-Way Match Passed)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Dr  200200  GR/IR Clearing Account        ₹29,000  (cleared)
Dr  160300  Input GST IGST Receivable     ₹5,220
Cr  200100  Accounts Payable Control      ₹34,220

FI Document Type: RE (auto-generated)
Tax Code: I3 (IGST 18%)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EVENT: Vendor Payment — F110 Automatic Payment Run
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Dr  200100  Accounts Payable Control      ₹34,220
Cr  100200  Bank — SBI Current Account    ₹34,220

Payment Method: T (NEFT/RTGS bank transfer)
FI Document Type: KZ

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EVENT: Goods Issue to Production — MIGO (Movement Type 261)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Dr  500100  Cost of Goods Sold            ₹29,000
Cr  130100  Raw Material Inventory        ₹29,000

OBYC Key: GBB/VBR (Dr) + BSX (Cr)
```

### 4.2 Order-to-Cash (SD → FI)

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EVENT: Post Goods Issue — VL02N (Movement Type 601)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Dr  500100  Cost of Goods Sold            ₹28,000
Cr  130300  Finished Goods Inventory      ₹28,000

OBYC Key: GBB/VAX (Dr) + BSX (Cr)
FI Document Type: WA (auto-generated)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EVENT: Billing — VF01 (Billing Type F2 — Tax Invoice)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Dr  100300  Accounts Receivable Control   ₹53,100
Cr  400100  Domestic Sales Revenue        ₹45,000
Cr  210300  GST Output IGST Payable       ₹8,100

VKOA Key: TN01 + 10 + 01 + Customer AAG 01 + Mat. AAG 01
FI Document Type: DR (auto-generated)
Tax Code: O3 (IGST 18%)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EVENT: Customer Payment — F-28 (Incoming Payment)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Dr  100200  Bank — SBI Current Account    ₹53,100
Cr  100300  Accounts Receivable Control   ₹53,100

FI Document Type: DZ (AR cleared)
```

---

## 5. GR/IR Clearing Account — Month-End Reconciliation

GL `200200` bridges the timing gap between goods receipt and invoice receipt. It must net to zero at month-end.

| Scenario | GR/IR Balance | Action |
|---|---|---|
| GR posted, invoice not yet received | Credit balance (vendor owes) | Normal — wait for invoice |
| Invoice posted, GR not yet done | Debit balance (goods not received) | Investigate — possible duplicate invoice |
| Balance remains at period-end | Either direction | Clear via `MR11`; investigate aged items |

Run `MR11` at every period close to maintain the account clean.

---

## 6. Module Integration Summary

```
┌──────────────────────────────────────────────────────────────┐
│                   SAP MODULE INTEGRATION MAP                  │
├────────────┬────────────────────────────┬────────────────────┤
│   MM       │   Integration Point         │   FI               │
├────────────┼────────────────────────────┼────────────────────┤
│ MIGO (GR)  │ OBYC BSX + WRX             │ Dr Stock / Cr GR/IR│
│ MIRO (LIV) │ 3-way match + OBYC WRX     │ Dr GR/IR / Cr AP   │
│ F110 (Pay) │ Automatic Payment Program  │ Dr AP / Cr Bank    │
├────────────┼────────────────────────────┼────────────────────┤
│   SD       │   Integration Point         │   FI               │
├────────────┼────────────────────────────┼────────────────────┤
│ VL02N PGI  │ OBYC GBB/VAX               │ Dr COGS / Cr Stock │
│ VF01 Bill  │ VKOA Revenue Determination │ Dr AR / Cr Revenue │
│ F-28 (Pay) │ Open Item Clearing         │ Dr Bank / Cr AR    │
└────────────┴────────────────────────────┴────────────────────┘
```

---

> *All GL accounts, values, and business scenarios are fictitious and created for academic purposes only.*
