# TechNova Manufacturing Pvt. Ltd. — SAP Organisational Structure

> **Module:** Enterprise Structure Design | **Company Code:** `TN01` | **Client:** `100`

---

## 1. Full Organisational Hierarchy

```
CLIENT: 100
└── COMPANY: TNOV (TechNova Group)
    └── COMPANY CODE: TN01 (TechNova Manufacturing Pvt. Ltd.)
        │
        ├── [FI]  Chart of Accounts  : TNCA
        │         Fiscal Year Variant : V3 (Apr–Mar)
        │         Controlling Area    : TN01
        │         Business Areas      : TN10 / TN20 / TN30
        │
        ├── [MM]  Plant               : TN01 (Bengaluru Main Plant)
        │         Plant               : TN02 (Hosur Warehouse)
        │         Storage Locs (TN01) : SL01 / SL02 / SL03 / SL04
        │         Purchasing Org      : TN01
        │         Purchasing Groups   : TNG1 / TNG2 / TNG3
        │
        └── [SD]  Sales Org           : TN01
                  Dist. Channels      : 10 (Direct) / 20 (Dealer) / 30 (Export)
                  Divisions           : 01 (Components) / 02 (Sub-Assemblies) / 03 (Finished)
                  Shipping Point      : TNSP
```

---

## 2. FI Organisational Units

### 2.1 Company & Company Code

| Object | Key | Description | T-Code |
|---|---|---|---|
| Client | `100` | Top-level SAP instance | — |
| Company | `TNOV` | TechNova Group (consolidation entity) | `OX15` |
| Company Code | `TN01` | TechNova Manufacturing Pvt. Ltd. — legal entity for all postings | `OX02` |

**Company Code Global Settings (`OBY6`):**

| Parameter | Value |
|---|---|
| Country | `IN` |
| Currency | `INR` |
| Language | `EN` |
| Chart of Accounts | `TNCA` |
| Fiscal Year Variant | `V3` (April 1 – March 31) |
| Field Status Variant | `TN01` |

### 2.2 Controlling Area

| Object | Key | Description |
|---|---|---|
| Controlling Area | `TN01` | Linked to Company Code TN01; controls cost centre and profit centre accounting |

### 2.3 Business Areas

Business areas allow segment-level P&L reporting within a single company code.

| Business Area | Description |
|---|---|
| `TN10` | Manufacturing Division |
| `TN20` | Distribution & Logistics |
| `TN30` | Administration & Corporate |

### 2.4 Fiscal Year Variant `V3`

| Period | Dates | Type |
|---|---|---|
| Period 1–12 | April 1 – March 31 | Normal posting periods |
| Period 13–16 | March 31 (adjustment) | Special periods for year-end adjustments |

Posting periods opened/closed via `OB52`.

---

## 3. MM Organisational Units

### 3.1 Plants

| Plant | Description | Address |
|---|---|---|
| `TN01` | Bengaluru Main Plant | Electronics City, Bengaluru, Karnataka |
| `TN02` | Hosur Warehouse | Hosur Industrial Area, Tamil Nadu |

### 3.2 Storage Locations (Plant TN01)

| Storage Location | Description | Use |
|---|---|---|
| `SL01` | Raw Material Store | Incoming ROH stock after GR (Mvt 101) |
| `SL02` | Components & MRO | HIBE consumables and bulk hardware |
| `SL03` | Production / WIP Store | HALB semi-finished goods |
| `SL04` | Finished Goods Store | FERT — ready for outbound delivery |

### 3.3 Purchasing Organisation & Groups

| Object | Key | Description |
|---|---|---|
| Purchasing Org | `TN01` | Responsible for all procurement for Company Code TN01 |
| Purchasing Group | `TNG1` | Raw Materials & Electronics |
| Purchasing Group | `TNG2` | Packaging & Consumables |
| Purchasing Group | `TNG3` | Capital Goods & Services |

---

## 4. SD Organisational Units

### 4.1 Sales Organisation

| Object | Key | Description |
|---|---|---|
| Sales Org | `TN01` | Responsible for all domestic and export sales |

### 4.2 Distribution Channels

| Channel | Key | Description |
|---|---|---|
| Direct Sales | `10` | B2B direct to end customers |
| Dealer Network | `20` | Sales through distributors and dealers |
| Export | `30` | International sales — Southeast Asia |

### 4.3 Divisions

| Division | Key | Description |
|---|---|---|
| Components | `01` | Electronic components and raw sub-assemblies |
| Sub-Assemblies | `02` | Semi-finished HALB products |
| Finished Goods | `03` | Fully assembled FERT products |

### 4.4 Sales Areas

A Sales Area = Sales Org + Distribution Channel + Division. Every customer master and pricing condition is assigned to a sales area.

| Sales Area | Sales Org | D.Ch | Div | Business Use |
|---|---|---|---|---|
| `TN01-10-01` | TN01 | 10 Direct | 01 Components | Domestic B2B component sales |
| `TN01-20-01` | TN01 | 20 Dealer | 01 Components | Dealer / distributor network |
| `TN01-30-02` | TN01 | 30 Export | 02 Sub-Assy | Southeast Asia exports |
| `TN01-10-03` | TN01 | 10 Direct | 03 Finished | Direct finished product sales |

### 4.5 Shipping Point

| Object | Key | Description |
|---|---|---|
| Shipping Point | `TNSP` | Central dispatch point at Bengaluru Main Plant |

---

## 5. All Organisational Unit Assignments

Organisational units in SAP are non-functional until correctly assigned. This table documents every required assignment and the T-code used.

| # | Assignment | T-Code | Direction | Status |
|---|---|---|---|---|
| 1 | Company Code → Company | `OX16` | TN01 → TNOV | Required |
| 2 | Plant → Company Code | `OX18` | TN01 → TN01 | **Critical** — without this, MM goods movements produce no FI document |
| 3 | Plant → Company Code | `OX18` | TN02 → TN01 | Required |
| 4 | Purchasing Org → Company Code | `OX01` | TN01 → TN01 | Required |
| 5 | Purchasing Org → Plant | `OX17` | TN01 → TN01 | Required |
| 6 | Purchasing Org → Plant | `OX17` | TN01 → TN02 | Required |
| 7 | Sales Org → Company Code | `OVX3` | TN01 → TN01 | Required |
| 8 | Distribution Channel → Sales Org | `OVXK` | 10 → TN01 | Required |
| 9 | Distribution Channel → Sales Org | `OVXK` | 20 → TN01 | Required |
| 10 | Distribution Channel → Sales Org | `OVXK` | 30 → TN01 | Required |
| 11 | Division → Sales Org | `OVXA` | 01 → TN01 | Required |
| 12 | Division → Sales Org | `OVXA` | 02 → TN01 | Required |
| 13 | Division → Sales Org | `OVXA` | 03 → TN01 | Required |
| 14 | Sales Area Setup | `OVXG` | TN01 + 10 + 01 | Required |
| 15 | Sales Area Setup | `OVXG` | TN01 + 20 + 01 | Required |
| 16 | Sales Area Setup | `OVXG` | TN01 + 30 + 02 | Required |
| 17 | Sales Area Setup | `OVXG` | TN01 + 10 + 03 | Required |
| 18 | Shipping Point → Plant | `OVXD` | TNSP → TN01 | Required |
| 19 | Controlling Area → Company Code | `OKKP` | TN01 → TN01 | Required |
| 20 | Business Area → Company Code | `OX03` | TN10/20/30 → TN01 | Required |

> **Critical Note:** Assignment #2 (Plant → Company Code via `OX18`) is the most commonly missed configuration step. If omitted, `MIGO` goods receipts will not generate FI accounting documents, breaking MM-FI integration completely.

---

## 6. SAP System Landscape

All configuration in this blueprint is performed in the **DEV client (100)** and transported to QAS and PRD via TMS.

```
┌──────────────┬──────────────┬──────────────────────┐
│  DEV (100)   │  QAS (200)   │      PRD (300)        │
│ Development  │  Quality /   │   Production          │
│ & Config     │  Testing     │   (Live System)       │
│              │              │                       │
│ SPRO config  │  UAT / SIT   │  Go-Live              │
│ Sandbox      │  Regression  │  End-user operations  │
└──────────────┴──────────────┴──────────────────────┘
       │               │                 │
       └───────────────┴─────────────────┘
            Transport Management System (TMS)
            DEV → QAS → PRD via T-Code STMS
```

---

*All organisational data is fictitious and created for academic purposes only.*
