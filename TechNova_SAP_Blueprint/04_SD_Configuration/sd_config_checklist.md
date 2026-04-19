# SAP SD Configuration Checklist — TechNova Manufacturing Pvt. Ltd.

> **Module:** Sales & Distribution — SD-SLS, SD-SHP, SD-BIL, SD-CAS
> **SPRO Path:** `SPRO → Sales and Distribution → ...`
> **Company Code:** `TN01` | **Sales Org:** `TN01`

---

## Phase 1 — Enterprise Structure for SD

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 1.1 | Define Sales Organisation | `OVX5` | `TN01` — TechNova Sales Org | ☐ |
| 1.2 | Assign Sales Org to Company Code | `OVX3` | TN01 → TN01 | ☐ |
| 1.3 | Define Distribution Channels | `OVXI` | `10` Direct, `20` Dealer, `30` Export | ☐ |
| 1.4 | Assign Distribution Channels to Sales Org | `OVXK` | 10, 20, 30 → TN01 | ☐ |
| 1.5 | Define Divisions | `OVXB` | `01` Components, `02` Sub-Assemblies, `03` Finished Goods | ☐ |
| 1.6 | Assign Divisions to Sales Org | `OVXA` | 01, 02, 03 → TN01 | ☐ |
| 1.7 | Set Up Sales Areas | `OVXG` | TN01+10+01, TN01+20+01, TN01+30+02, TN01+10+03 | ☐ |
| 1.8 | Define Shipping Point | `OVXD` | `TNSP` — Central Dispatch, Bengaluru; assign to Plant TN01 | ☐ |

---

## Phase 2 — Customer Master Configuration

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 2.1 | Define Customer Account Groups | `OBD2` | `DOME` (domestic B2B), `EXPG` (export), `DIST` (distributor) | ☐ |
| 2.2 | Define Number Ranges for Customers | `XDN1` | Internal number ranges per account group | ☐ |
| 2.3 | Assign Number Ranges to Account Groups | `OBAR` | Per group | ☐ |
| 2.4 | Define Customer Pricing Procedure | `OVKP` | Field in customer master that feeds pricing procedure determination | ☐ |
| 2.5 | Create Customer Master Records | `XD01` | 15 customers per `customer_master_data.xlsx` | ☐ |
| 2.6 | Maintain Customer Credit Limits | `FD32` | Per customer — DOME: ₹8L–₹50L; DIST: ₹55L–₹60L | ☐ |

---

## Phase 3 — Sales Document Types

| # | Activity | T-Code | Doc Type | Description | Status |
|---|---|---|---|---|---|
| 3.1 | Define Sales Document Types | `VOV8` | `OR` | Standard Order | ☐ |
| 3.2 | Define Sales Document Types | `VOV8` | `RE` | Returns Order | ☐ |
| 3.3 | Define Sales Document Types | `VOV8` | `QT` | Quotation | ☐ |
| 3.4 | Define Number Ranges for Sales Docs | `VN01` | OR: 30000000–39999999; QT: 20000000–29999999 | ☐ |
| 3.5 | Define Item Categories | `VOV7` | `TAN` (standard item), `TANN` (free item) | ☐ |
| 3.6 | Define Schedule Line Categories | `VOV6` | `CP` (standard — MRP relevant) | ☐ |

---

## Phase 4 — Pricing Procedure `TNPRC`

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 4.1 | Define Pricing Procedure | `V/08` | Procedure: `TNPRC` — TechNova Standard Pricing | ☐ |
| 4.2 | Define Condition Types | `V/06` | PR00 (Base Price), K004 (Customer Disc.), K007 (Cust. Group Disc.), MWST (GST), VPRS (Cost) | ☐ |
| 4.3 | Define Access Sequences | `V/07` | For each condition type | ☐ |
| 4.4 | Assign Pricing Procedure | `OVKK` | Sales Area + Customer Pricing Procedure + Doc Pricing Procedure → TNPRC | ☐ |
| 4.5 | Create Pricing Condition Records | `VK11` | Base prices, discounts per customer/material | ☐ |

**Pricing Procedure `TNPRC` — Step Sequence:**

| Step | Condition Type | Description | Base | Calculation |
|---|---|---|---|---|
| 10 | `PR00` | Base Price | — | Manual / info record |
| 20 | `K004` | Customer Discount | Step 10 | % deduction |
| 30 | `K007` | Customer Group Discount | Step 10 | % deduction |
| 40 | — | **Net Price** (subtotal) | — | Steps 10–30 |
| 50 | `MWST` | Output GST 18% | Step 40 | % on net |
| 60 | `VPRS` | Cost (MAP / Std Price) | — | Statistical only |
| 70 | — | **Grand Total** | — | Step 40 + Step 50 |

> `VPRS` is statistical — feeds margin visibility in the order but is NOT charged to the customer.

---

## Phase 5 — Availability Check & Transfer of Requirements

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 5.1 | Define Checking Groups | `OVZ2` | `02` — Individual requirements check | ☐ |
| 5.2 | Define Checking Rules | `OVZ9` | Rule A — sales order; Rule B — delivery | ☐ |
| 5.3 | Configure ATP Check Scope | `OVZ9` | Include: unrestricted stock, purchase orders, production orders | ☐ |
| 5.4 | Activate Transfer of Requirements | `OVZ1` | Link SD requirements to MRP | ☐ |

---

## Phase 6 — Delivery & Shipping

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 6.1 | Define Delivery Document Types | `0VLK` | `LF` — Outbound Delivery | ☐ |
| 6.2 | Define Number Ranges for Deliveries | `VN01` | LF: 80000000–89999999 | ☐ |
| 6.3 | Define Shipping Conditions | `OVST` | Conditions linked to customer master | ☐ |
| 6.4 | Configure Goods Issue (PGI) | `VL02N` | Movement type `601` — triggers FI auto-posting Dr 500100 / Cr 130300 | ☐ |
| 6.5 | Define Routes | `OVR1` | Bengaluru → Delhi, Bengaluru → Chennai, Bengaluru → Singapore | ☐ |

---

## Phase 7 — Billing

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 7.1 | Define Billing Document Types | `VOFA` | `F2` — Tax Invoice (GST-compliant); `RE` — Credit Memo | ☐ |
| 7.2 | Define Number Ranges for Billing Docs | `VN01` | F2: 90000000–99999999 | ☐ |
| 7.3 | Configure Revenue Account Determination | `VKOA` | Application V, Table 001 — Cust. AAG + Mat. AAG → GL account | ☐ |
| 7.4 | Define Billing Output Type | `V/30` | `RD00` — Tax Invoice PDF email to customer | ☐ |
| 7.5 | Run Billing Due List | `VF04` | Bulk billing run — executed at period-end | ☐ |

**VKOA Revenue Account Determination:**

| Customer AAG | Material AAG | GL | Description |
|---|---|---|---|
| `01` Domestic | `01` Components | `400100` | Domestic Sales Revenue |
| `02` Export | `01` Components | `400200` | Export Sales Revenue |
| `01` Domestic | `02` Sub-Assy | `400100` | Domestic Sales Revenue |

**Billing FI Posting (auto via VKOA):**

```
Dr 100300  Accounts Receivable Control
   Cr 400100  Sales Revenue
   Cr 210300  GST Output IGST Payable
```

---

## Phase 8 — Credit Management

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 8.1 | Define Credit Control Area | `OB45` | Area: `TN01`, Currency: INR, linked to Company Code TN01 | ☐ |
| 8.2 | Define Credit Check at Sales Order | `OVA8` | Static credit check — block delivery if AR + order > credit limit | ☐ |
| 8.3 | Assign Credit Control Area to Company Code | `OB45` | TN01 → TN01 | ☐ |
| 8.4 | Set Credit Limits per Customer | `FD32` | DOME: ₹8L–₹50L; EXPG: ₹30L–₹40L (USD equiv.); DIST: ₹55L–₹60L | ☐ |
| 8.5 | Review and Release Credit-Blocked Orders | `VKM1` | Credit manager releases after review | ☐ |

---

> *Configuration performed in DEV (Client 100) and transported to QAS → PRD via `STMS`.*
> *All data is fictitious and for academic purposes only.*
