# Procure-to-Pay (P2P) Process Flow — TechNova Manufacturing Pvt. Ltd.

> **Process:** Procure-to-Pay | **Modules:** MM + FI-AP
> **Company Code:** `TN01` | **Plant:** `TN01` (Bengaluru Main)

---

## 1. Process Overview

The P2P cycle covers every step from identifying a procurement need to settling the vendor's invoice. In TechNova's SAP implementation, the cycle is fully integrated — goods movements automatically generate FI accounting documents, and the Automatic Payment Program settles vendors weekly without manual journal entries.

```
[ME51N] Purchase Requisition (PR)
    ↓  Released via ME54N (Purchase Manager — L1)
[ME21N] Purchase Order ──► Vendor notified (Output NEU)
    ↓  Released via ME29N (L2 if > ₹50,000 / L3 if > ₹5,00,000)
[MIGO / 101] Goods Receipt
    ↓  FI auto-post: Dr 130100 / Cr 200200
    ↓  MM: Stock ↑ in SL01 (Raw Material Store)
[MIRO] Invoice Verification — 3-Way Match (PO + GR + Invoice)
    ↓  FI auto-post: Dr 200200 + Dr 160300 / Cr 200100
    ↓  If blocked (variance > tolerance) → MRBR to release
[F110] Automatic Payment Run (weekly — every Friday)
    ↓  FI auto-post: Dr 200100 / Cr 100200
    Output: NEFT/RTGS bank transfer advice to vendor
```

---

## 2. Step-by-Step Process

### Step 1 — Purchase Requisition (PR)

**T-Code:** `ME51N`
**Created by:** Warehouse / Production Planner or MRP (automatic)

- MRP Type `VB` triggers PR automatically when stock falls below reorder point
- Manual PR raised by store-keeper for ad-hoc requirements
- PR contains: material number, quantity, required delivery date, plant, storage location, purchasing group

**Release (T-Code `ME54N`):**
- PR value ≤ ₹50,000 → auto-approved or Purchase Manager releases
- PR value > ₹50,000 → requires Purchase Manager (L1) before PO creation

**Key Fields in PR:**
| Field | Value |
|---|---|
| Document Type | `NB` (standard) |
| Plant | `TN01` |
| Storage Location | `SL01` (raw material store) |
| Purchasing Group | `TNG1` (electronics) |
| Desired Vendor | Populated from info record |

---

### Step 2 — Purchase Order (PO)

**T-Code:** `ME21N`
**Created by:** Purchasing Team

- PO references the PR; vendor, price, and delivery terms populated from info record (`ME11`)
- GST tax code automatically determined based on vendor state vs. plant state:
  - Intra-state (Karnataka vendor, Plant TN01 Karnataka) → `I1` CGST+SGST
  - Inter-state → `I3` IGST 18%

**Release Strategy:**

| PO Net Value | Release Codes Required | Approvers |
|---|---|---|
| ≤ ₹50,000 | `L1` | Purchase Manager |
| ₹50,001 – ₹5,00,000 | `L1` + `L2` | Purchase Manager + Head of Procurement |
| > ₹5,00,000 | `L1` + `L2` + `L3` | Purchase Manager + Head of Procurement + CFO |

- PO output type `NEU` emails the PO PDF to the vendor once all release codes are applied
- Without full release, PO output cannot be triggered

**Key PO Fields:**
| Field | Value |
|---|---|
| Document Type | `NB` |
| Purchasing Org | `TN01` |
| Purchasing Group | Per material category |
| Payment Terms | Per vendor master (NET30 / NET15) |
| GR-Based IV | ✔ Activated |

---

### Step 3 — Goods Receipt (GR)

**T-Code:** `MIGO`
**Movement Type:** `101`
**Performed by:** Warehouse / Stores Team

- GR posted against PO reference
- Material received into Storage Location `SL01` (Unrestricted stock)
- Quality inspection check: if batch fails → movement type `103` (Blocked Stock) instead of `101`

**FI Auto-Posting (OBYC → BSX + WRX):**

```
Dr  130100  Raw Material Inventory    [PO quantity × MAP/std price]
Cr  200200  GR/IR Clearing Account    [same value]

FI Document Type: WE
```

**Stock Effects:**
- Unrestricted stock in `SL01` increases
- GR/IR account shows open credit (awaiting matching invoice)

**GR Document fields:**
| Field | Value |
|---|---|
| Reference | PO Number |
| Batch Number | Auto-assigned (e.g., `B-2026-04`) |
| Quality Check | Passed / Partial / Rejected |

---

### Step 4 — Invoice Verification (3-Way Match)

**T-Code:** `MIRO`
**Performed by:** Accounts Payable Team

`MIRO` automatically compares three documents before allowing posting:

```
┌─────────────────┐
│  Purchase Order  │ ← Price & quantity agreed with vendor
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Goods Receipt   │ ← Quantity actually received in warehouse
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Vendor Invoice   │ ← Price & quantity billed by vendor
└─────────────────┘
         │
         ▼
   3-Way Match Result
```

**Tolerance Keys (`OMR6`):**

| Key | Check | Limit | Result if Exceeded |
|---|---|---|---|
| `BD` | Absolute difference | ±₹500 | Auto-cleared |
| `DW` | Quantity variance | ±2% | Invoice blocked |
| `PP` | Price variance | ±2% | Invoice blocked |

**FI Auto-Posting (match passed):**

```
Dr  200200  GR/IR Clearing Account        [GR value — cleared]
Dr  160300  Input GST IGST Receivable     [18% × invoice base]
Cr  200100  Accounts Payable Control      [total invoice amount]

FI Document Type: RE
```

**If Invoice Blocked (variance > tolerance):**
- Invoice auto-blocked with block reason code
- AP Supervisor reviews via `MRBR`
- Can manually release or reject for vendor credit note

---

### Step 5 — Automatic Payment Run

**T-Code:** `F110`
**Performed by:** Finance / Treasury
**Frequency:** Weekly (every Friday)

- Payment run processes all open vendor items due by payment run date
- Payment method `T` — NEFT/RTGS bank transfer via house bank `SBI01`
- Payment advice (output) sent to vendor

**FI Auto-Posting:**

```
Dr  200100  Accounts Payable Control      [invoice total]
Cr  100200  Bank — SBI Current Account    [same amount]

FI Document Type: KZ
Payment Document: auto-assigned from range 5200000–5299999
```

---

## 3. FI Posting Summary — Full P2P Cycle

| Step | Business Event | T-Code | Dr GL | Cr GL | Amount |
|---|---|---|---|---|---|
| 1 | Purchase Order | `ME21N` | — | — | No FI |
| 2 | Goods Receipt | `MIGO` | 130100 (Stock) | 200200 (GR/IR) | PO Value |
| 3 | Invoice Verification | `MIRO` | 200200 (GR/IR) + 160300 (Tax) | 200100 (AP) | Invoice + Tax |
| 4 | Vendor Payment | `F110` | 200100 (AP) | 100200 (Bank) | Invoice Total |

---

## 4. Sample P2P Transaction

> From `06_Process_Flows/p2p_transaction_log.xlsx` — Row 1

| Document | Number | Details |
|---|---|---|
| Purchase Requisition | `PR-10000021` | 500 units ECM-500, ₹4,800 each |
| Purchase Order | `4500000085` | V-1001 Sigma Components, ₹24,00,000 + GST |
| Goods Receipt | `5000000034` | 500 EA received — SL01 — Quality: Passed |
| MIRO Invoice | `INV-SC-20260418` | ₹24,00,000 + ₹4,32,000 GST = ₹28,32,000 — 3-way matched |
| Payment | F110 run | NEFT cleared ₹28,32,000 — 18-May-2026 |

---

## 5. P2P Document Flow

```
PR (ME51N)
 └── PO (ME21N)  ─────────────────────────────► Vendor
      └── GR (MIGO / 101)
           └── MIRO Invoice Verification
                └── F110 Payment (NEFT)
                     └── Bank Debit Advice
```

Visible in system via: `ME23N → Environment → Document Flow` or `MIGO → Display → Follow-on Documents`

---

> *All transaction data is fictitious and for academic purposes only.*
