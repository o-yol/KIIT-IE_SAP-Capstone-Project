# Order-to-Cash (O2C) Process Flow — TechNova Manufacturing Pvt. Ltd.

> **Process:** Order-to-Cash | **Modules:** SD + FI-AR
> **Company Code:** `TN01` | **Sales Org:** `TN01`

---

## 1. Process Overview

The O2C cycle covers the complete journey from customer inquiry to cash collection. TechNova's SAP implementation integrates SD and FI fully — PGI (goods issue) automatically posts COGS, and billing (`VF01`) automatically posts AR and revenue without manual journal entries.

```
[VA21] Quotation ──► Customer (Output AN00, 30-day validity)
    ↓  Customer confirms
[VA01] Sales Order ──► Auto: Availability Check + Credit Check (FD32)
    Output BA00: Order Confirmation emailed to customer
    ↓  Availability confirmed
[VL01N] Outbound Delivery document created
    ↓
[VL02N — Picking] Pick from SL04 (Finished Goods Store)
    ↓
[VL02N — Post Goods Issue] Movement Type 601
    FI auto-post: Dr 500100 / Cr 130300
    MM: FG Stock ↓
    ↓
[VF01] Billing Document (Type F2 — GST Tax Invoice)
    FI auto-post: Dr 100300 / Cr 400100 + Cr 210300
    Output RD00: GST tax invoice emailed to customer
    ↓
[F-28] Incoming Payment posted
    FI: Dr 100200 / Cr 100300 — AR open item cleared
```

---

## 2. Step-by-Step Process

### Step 1 — Quotation (Optional)

**T-Code:** `VA21`
**Created by:** Sales Team

- Quotation validity: 30 days
- Pricing determined automatically via pricing procedure `TNPRC`
- Output type `AN00` — quotation PDF emailed to customer
- Converted to Sales Order via `VA01` → `Create with Reference`

---

### Step 2 — Sales Order

**T-Code:** `VA01`
**Created by:** Sales / Order Management Team

At sales order save, SAP runs two automatic checks:

**a) Availability Check (ATP):**
- Checks unrestricted stock in Plant TN01, Storage Location SL04
- If stock available → confirmed delivery date populated
- If stock insufficient → delivery date pushed out; MRP triggered for planned order

**b) Credit Check (`OVA8`):**
- SAP compares: Customer open AR + current order value vs. credit limit (`FD32`)
- If within limit → order confirmed
- If limit exceeded → delivery block `01` applied; sales order saved but cannot be delivered
- Credit manager reviews and releases via `VKM1`

**Pricing (Procedure `TNPRC`):**

| Step | Condition | Amount |
|---|---|---|
| Base Price (PR00) | ₹45,000 per unit | ₹4,50,000 (10 units) |
| Customer Discount (K004) | –5% | –₹22,500 |
| **Net Price** | | ₹4,27,500 |
| Output GST 18% (MWST) | 18% × net | ₹76,950 |
| **Grand Total** | | ₹5,04,450 |
| Cost (VPRS — statistical) | Std price ₹35,000 | ₹3,50,000 |

**Key Sales Order Fields:**

| Field | Value |
|---|---|
| Document Type | `OR` (Standard Order) |
| Sales Area | `TN01-10-01` (Direct, Components) |
| Pricing Procedure | `TNPRC` |
| Payment Terms | NET30 / NET45 per customer master |
| Shipping Point | `TNSP` |

---

### Step 3 — Outbound Delivery

**T-Code:** `VL01N`
**Created by:** Warehouse / Logistics Team

- Delivery document created with reference to Sales Order
- System proposes picking quantity from Storage Location `SL04` (Finished Goods Store)
- Picking confirmed in `VL02N` → transfer order created

**Key Delivery Fields:**

| Field | Value |
|---|---|
| Document Type | `LF` |
| Shipping Point | `TNSP` |
| Pick from Storage Loc | `SL04` |
| Delivery Date | Per sales order confirmed date |

---

### Step 4 — Post Goods Issue (PGI)

**T-Code:** `VL02N → Post Goods Issue`
**Movement Type:** `601`
**Performed by:** Warehouse Team

This is the critical stock-reducing event. On PGI:
- Finished goods stock in `SL04` decreases
- FI accounting document auto-posted via `OBYC → GBB/VAX`

**FI Auto-Posting:**

```
Dr  500100  Cost of Goods Sold           [std cost × qty shipped]
Cr  130300  Finished Goods Inventory     [same value]

FI Document Type: WA (auto-generated)
OBYC Key: GBB/VAX (Dr) + BSX (Cr)
```

> After PGI, the goods legally belong to the customer. Billing can now proceed.

---

### Step 5 — Billing

**T-Code:** `VF01` (individual) or `VF04` (billing due list — bulk)
**Billing Type:** `F2` (Tax Invoice — GST compliant)
**Performed by:** Finance / Billing Team

`VF01` creates the customer-facing GST tax invoice and simultaneously posts the FI revenue document automatically via `VKOA`.

**FI Auto-Posting (VKOA):**

```
Dr  100300  Accounts Receivable Control  [net + GST]
Cr  400100  Domestic Sales Revenue       [net value]
Cr  210300  GST Output IGST Payable      [18% GST]

FI Document Type: DR (Customer Invoice)
VKOA Key: TN01 + D.Ch 10 + Div 01 + Cust. AAG 01 + Mat. AAG 01
```

**GST on Billing:**
- Intra-state customer (same state as plant) → CGST + SGST (split 9%+9%)
- Inter-state customer → IGST 18% single line
- Export customer → IGST applicable (or zero-rated under LUT)

**Output:** `RD00` — GST-compliant tax invoice PDF emailed to customer

---

### Step 6 — Incoming Payment

**T-Code:** `F-28`
**Performed by:** Accounts Receivable / Treasury Team

- Payment received from customer (NEFT / cheque)
- AR open item cleared against billing document

**FI Posting:**

```
Dr  100200  Bank — SBI Current Account   [amount received]
Cr  100300  Accounts Receivable Control  [AR open item cleared]

FI Document Type: DZ (Customer Payment)
```

**Dunning (if payment overdue — `FBMP`):**
| Dunning Level | Day | Action |
|---|---|---|
| Level 1 | Day 5 after due | Reminder email |
| Level 2 | Day 15 after due | Warning with interest |
| Level 3 | Day 30 after due | Legal notice |

---

## 3. FI Posting Summary — Full O2C Cycle

| Step | Business Event | T-Code | Dr GL | Cr GL |
|---|---|---|---|---|
| 1 | Sales Order | `VA01` | — | — (no FI) |
| 2 | Post Goods Issue | `VL02N` | 500100 (COGS) | 130300 (FG Stock) |
| 3 | Billing | `VF01` | 100300 (AR) | 400100 (Revenue) + 210300 (GST) |
| 4 | Customer Payment | `F-28` | 100200 (Bank) | 100300 (AR cleared) |

---

## 4. SAP Document Flow

The complete document chain is visible in `VA03 → Environment → Document Flow`:

```
Quotation            20000001  (VA21)
  └── Sales Order    30000001  (VA01)
        └── Delivery      80000001  (VL01N)
              └── Billing Doc   90000001  (VF01)
                    └── FI-DR Doc     1800000001  (Auto)
                          └── FI-DZ Payment  1900000001  (F-28)
```

---

## 5. Sample O2C Transaction

> From `06_Process_Flows/o2c_transaction_log.xlsx` — Row 1

| Document | Number | Details |
|---|---|---|
| Sales Order | `SO-80000011` | C-1001 Electra Automation, 10 × MAT-7001 @ ₹45,000 |
| Outbound Delivery | `LF-90000011` | Picked from SL04, shipped via TNSP |
| Post Goods Issue | Movement 601 | Dr 500100 ₹35,000 / Cr 130300 ₹35,000 (std cost) |
| Billing | `INV-90000011` | Net ₹4,50,000 + CGST ₹40,500 + SGST ₹40,500 = ₹5,31,000 |
| Customer Payment | F-28 | ₹5,31,000 cleared — 02-May-2026 |

---

> *All transaction data is fictitious and for academic purposes only.*
