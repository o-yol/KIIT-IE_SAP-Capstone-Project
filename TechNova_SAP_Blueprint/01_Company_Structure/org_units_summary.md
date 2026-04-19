# TechNova — Organisational Units Quick Reference

> All 27 SAP organisational units for TechNova Manufacturing Pvt. Ltd. in one table.
> **Company Code:** `TN01` | **Client:** `100`

---

## All 27 Organisational Units

| # | Unit Type | Key | Description | Module | Creation T-Code |
|---|---|---|---|---|---|
| 1 | Client | `100` | SAP Instance — TechNova Group | Cross | — |
| 2 | Company | `TNOV` | TechNova Group (consolidation entity) | FI | `OX15` |
| 3 | Company Code | `TN01` | TechNova Manufacturing Pvt. Ltd. | FI | `OX02` |
| 4 | Chart of Accounts | `TNCA` | 6-digit GL account plan | FI | `OB13` |
| 5 | Fiscal Year Variant | `V3` | April 1 – March 31 (Indian FY) | FI | `OB29` |
| 6 | Controlling Area | `TN01` | Cost & profit centre accounting | CO | `OKKP` |
| 7 | Business Area | `TN10` | Manufacturing Division | FI | `OX03` |
| 8 | Business Area | `TN20` | Distribution & Logistics | FI | `OX03` |
| 9 | Business Area | `TN30` | Administration & Corporate | FI | `OX03` |
| 10 | Plant | `TN01` | Bengaluru Main Plant | MM | `OX10` |
| 11 | Plant | `TN02` | Hosur Warehouse | MM | `OX10` |
| 12 | Storage Location | `SL01` | Raw Material Store (Plant TN01) | MM | `OX09` |
| 13 | Storage Location | `SL02` | Components & MRO Store (Plant TN01) | MM | `OX09` |
| 14 | Storage Location | `SL03` | Production / WIP Store (Plant TN01) | MM | `OX09` |
| 15 | Storage Location | `SL04` | Finished Goods Store (Plant TN01) | MM | `OX09` |
| 16 | Purchasing Org | `TN01` | Central Purchasing — all procurement | MM | `OX08` |
| 17 | Purchasing Group | `TNG1` | Raw Materials & Electronics | MM | `OME4` |
| 18 | Purchasing Group | `TNG2` | Packaging & Consumables | MM | `OME4` |
| 19 | Purchasing Group | `TNG3` | Capital Goods & Services | MM | `OME4` |
| 20 | Sales Organisation | `TN01` | All domestic and export sales | SD | `OVX5` |
| 21 | Distribution Channel | `10` | Direct Sales (B2B) | SD | `OVXI` |
| 22 | Distribution Channel | `20` | Dealer / Distributor Network | SD | `OVXI` |
| 23 | Distribution Channel | `30` | Export — Southeast Asia | SD | `OVXI` |
| 24 | Division | `01` | Electronic Components | SD | `OVXB` |
| 25 | Division | `02` | Sub-Assemblies (HALB) | SD | `OVXB` |
| 26 | Division | `03` | Finished Products (FERT) | SD | `OVXB` |
| 27 | Shipping Point | `TNSP` | Central Dispatch — Bengaluru Plant | SD | `OVXD` |

---

## Assignment Summary

| Assignment | T-Code | From → To |
|---|---|---|
| Company Code → Company | `OX16` | TN01 → TNOV |
| Plant → Company Code | `OX18` | TN01, TN02 → TN01 |
| Purchasing Org → Company Code | `OX01` | TN01 → TN01 |
| Purchasing Org → Plant | `OX17` | TN01 → TN01 / TN02 |
| Sales Org → Company Code | `OVX3` | TN01 → TN01 |
| Distribution Channel → Sales Org | `OVXK` | 10, 20, 30 → TN01 |
| Division → Sales Org | `OVXA` | 01, 02, 03 → TN01 |
| Sales Area Setup | `OVXG` | All combinations |
| Shipping Point → Plant | `OVXD` | TNSP → TN01 |
| Controlling Area → Company Code | `OKKP` | TN01 → TN01 |

---

## Sales Areas (Combinations)

| Sales Area | Sales Org | D.Ch | Division |
|---|---|---|---|
| `TN01-10-01` | TN01 | 10 Direct | 01 Components |
| `TN01-20-01` | TN01 | 20 Dealer | 01 Components |
| `TN01-30-02` | TN01 | 30 Export | 02 Sub-Assy |
| `TN01-10-03` | TN01 | 10 Direct | 03 Finished |

---

*All data is fictitious and for academic use only.*
