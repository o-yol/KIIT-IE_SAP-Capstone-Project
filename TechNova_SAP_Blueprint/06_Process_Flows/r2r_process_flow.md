# Record-to-Report (R2R) Process Flow — TechNova Manufacturing Pvt. Ltd.

> **Process:** Record-to-Report (Month-End & Year-End Close)
> **Module:** FI-GL, FI-AP, FI-AR, FI-AA, CO
> **Company Code:** `TN01` | **Fiscal Year:** V3 (April 1 – March 31)

---

## 1. Overview

The R2R cycle encompasses all activities from daily transaction recording through to the issuance of financial statements. For TechNova, the month-end close follows a structured 10-step sequence across D-5 to D+3 (where D = last calendar day of the period). The year-end close in March carries additional steps for balance carry-forward and statutory filing.

---

## 2. Month-End Close Calendar

| Day | Activity | T-Code | Owner | Notes |
|---|---|---|---|---|
| D-5 | Freeze MM — complete all GRs for the period | `OB52` | MM Team | Close MM posting period for period M; no backdated GRs after this |
| D-4 | Complete all SD billing — run billing due list | `VF04` | SD / Finance | Process all pending deliveries into invoices before period close |
| D-3 | Run asset depreciation | `AFAB` | Fixed Assets | Planned run — posts depreciation Dr 550300 / Cr 110200 |
| D-2 | Post accruals (salary, utilities, warranty provisions) | `FB50` | GL Team | Manual journal using doc type SA |
| D-2 | Clear GR/IR account; investigate aged items | `MR11` / `F.13` | AP Team | GL 200200 must net to zero; aged items escalated |
| D-1 | Foreign currency revaluation (USD vendors / customers) | `FAGL_FC_VAL` | Treasury | Posts FX gain/loss for open USD items at month-end rate |
| D-1 | Reconcile AP/AR subledgers vs GL control accounts | `S_ALR_87012082` / `S_ALR_87012173` | Finance | Vendor sub-ledger total = GL 200100; Customer sub-ledger total = GL 100300 |
| D-0 | Close posting period for account types A/D/K/S/M | `OB52` | Controller | Locks the closed period; only special periods 13–16 remain open for adjustments |
| D+1 | Review Trial Balance — investigate unusual balances | `S_ALR_87012277` | CFO | Sign-off required before financial statements |
| D+3 | Issue P&L and Balance Sheet | `S_ALR_87012284` | Finance | Distributed to management and board |

---

## 3. Step-by-Step Month-End Activities

### 3.1 MM Period Freeze (D-5)

**T-Code:** `OB52` (close MM posting period for prior-period GRs)

- Inform all plants to complete pending GRs at least 5 days before period end
- Any GR posted after freeze must be backdated — requires Finance approval
- Run `MB52` (Warehouse Stocks Report) to confirm all expected deliveries are received

**Checklist:**
- [ ] All POs with expected delivery by period-end have GR posted
- [ ] GR/IR account reviewed for unmatched items (`MB5S`)
- [ ] No open blocked-stock items pending quality decision

---

### 3.2 SD Billing Completion (D-4)

**T-Code:** `VF04` (Billing Due List)

- Run billing due list to capture all deliveries pending billing
- Ensure all PGI documents from the period are billed
- AR opens once billing is posted — critical for period-end AR balance accuracy

**Checklist:**
- [ ] `VF04` run — zero items remaining in billing due list
- [ ] All credit memos processed for returns
- [ ] Customer statements sent for reconciliation

---

### 3.3 Asset Depreciation (D-3)

**T-Code:** `AFAB` (Depreciation Run)

- Planned run for the current period
- Posts depreciation for all active assets under Chart of Depreciation `IN01`

**FI Posting:**
```
Dr  550300  Depreciation Expense          [calculated per asset]
Cr  110200  Accumulated Depreciation      [same amount]
```

**Checklist:**
- [ ] `AFAB` planned run executed — no errors
- [ ] Asset addition / disposal transactions completed before depreciation run
- [ ] `AW01N` — Asset Explorer used to verify per-asset depreciation

---

### 3.4 Accruals & Provisions (D-2)

**T-Code:** `FB50` (GL Document Entry)

Manual journal entries for expenses incurred but not yet invoiced:

| Accrual | Dr GL | Cr GL | Amount |
|---|---|---|---|
| Salary payable | 550XXX (Salary Expense) | 200400 (Salary Payable) | Per payroll |
| Utilities provision | 550XXX (Utilities Expense) | 200XXX (Utilities Payable) | Estimated |
| Warranty provision | 550XXX (Warranty Expense) | 200XXX (Warranty Reserve) | % of sales |

Document Type: `SA` | Must be reversed in the next period via `F.81` (mass reversal)

---

### 3.5 GR/IR Clearing (D-2)

**T-Code:** `MR11` (GR/IR Account Maintenance) and `F.13` (Automatic Clearing)

- GL `200200` (GR/IR Clearing) must net to zero at period-end
- Run `F.13` to auto-clear matched GR/IR pairs
- `MR11` handles unmatched items:
  - GR posted, no invoice → leave open if invoice expected within 30 days; accrue if older
  - Invoice posted, no GR → investigate with warehouse; reverse if duplicate

**Checklist:**
- [ ] `F.13` automatic clearing run — matched pairs cleared
- [ ] `MR11` — residual items reviewed and actioned
- [ ] GL 200200 balance = ₹0 (or explained in close notes)

---

### 3.6 Foreign Currency Revaluation (D-1)

**T-Code:** `FAGL_FC_VAL`

- Applies to open USD items: import vendor invoices (V-2001, V-2002, V-2003) and export customer AR (C-2001, C-2002, C-2003)
- SAP revalues open items at the period-end exchange rate
- Posts unrealised FX gain/loss to designated GL accounts

**FI Posting (example — USD vendor invoice, INR strengthened):**
```
Dr  200100  Accounts Payable Control      [FX gain — liability reduced]
Cr  XXXXXX  Unrealised FX Gain            [P&L]
```

---

### 3.7 AP / AR Subledger Reconciliation (D-1)

**T-Codes:** `S_ALR_87012082` (Vendor Balance List) | `S_ALR_87012173` (Customer Balance List)

- Vendor subledger total must equal GL `200100` balance
- Customer subledger total must equal GL `100300` balance
- Any discrepancy = data integrity issue requiring investigation before close

---

### 3.8 Period Close (D-0)

**T-Code:** `OB52`

Close posting periods for:
- Account type `A` (Assets)
- Account type `D` (Customers)
- Account type `K` (Vendors)
- Account type `S` (GL Accounts)
- Account type `M` (Materials)

Special periods 13–16 remain open for year-end adjustments only.

---

### 3.9 Trial Balance Review (D+1)

**T-Code:** `S_ALR_87012277`

- CFO and Controller review all GL balances
- Flag any unusual movements for explanation
- Confirm all accruals are posted
- Sign-off required before P&L issuance

---

### 3.10 Financial Statements (D+3)

**T-Code:** `S_ALR_87012284` (Balance Sheet & P&L Statement)

- Balance Sheet: Assets, Liabilities, Equity as at period-end
- P&L: Revenue, COGS, Gross Margin, Expenses, Net Profit for the period
- Distributed to: MD, CFO, Board of Directors, Statutory Auditors (at year-end)

---

## 4. Year-End Close (March — Period 12 / Special Periods 13–16)

| Step | Activity | T-Code | Notes |
|---|---|---|---|
| YE-1 | Complete all month-end steps for March | Per above | Full month-end sequence |
| YE-2 | Post year-end audit adjustments | `FB50` | Use special periods 13–16; doc type SA |
| YE-3 | Run final depreciation (period 12) | `AFAB` | Confirm no assets added/disposed after this |
| YE-4 | Perform balance carry-forward — GL | `F.16` | Carries P&L balance to retained earnings GL 300200 |
| YE-5 | Perform balance carry-forward — AR/AP | `F.07` | Carries open items to new fiscal year |
| YE-6 | Open new fiscal year posting periods | `OB52` | Open April (Period 1) of new FY |
| YE-7 | Run GST annual reconciliation | External | Reconcile GSTR-1 / GSTR-3B with GL tax accounts |
| YE-8 | Issue annual financial statements | `S_ALR_87012284` | For statutory filing and board approval |

---

## 5. Key Reports Used in R2R

| Report | T-Code | Purpose | Frequency |
|---|---|---|---|
| Trial Balance | `S_ALR_87012277` | All GL balances | Monthly |
| Balance Sheet & P&L | `S_ALR_87012284` | Financial statements | Monthly / Annual |
| Vendor Balance List | `S_ALR_87012082` | AP subledger vs GL 200100 | Monthly |
| Customer Balance List | `S_ALR_87012173` | AR subledger vs GL 100300 | Monthly |
| GR/IR Analysis | `MB5S` | Open GR/IR items | Monthly |
| Stock Overview | `MMBE` | Inventory balances by SLoc | Monthly |
| Asset Explorer | `AW01N` | Per-asset depreciation review | Monthly |
| Open Item Report | `FBL1N` / `FBL5N` | Vendor/Customer open items | Monthly |

---

> *All dates, amounts, and data in this document are fictitious and for academic purposes only.*
