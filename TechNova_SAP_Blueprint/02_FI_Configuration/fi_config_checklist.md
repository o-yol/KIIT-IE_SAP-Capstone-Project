# SAP FI Configuration Checklist — TechNova Manufacturing Pvt. Ltd.

> **Module:** Financial Accounting (New) — FI-GL, FI-AP, FI-AR
> **SPRO Path:** `SPRO → Financial Accounting (New) → ...`
> **Company Code:** `TN01` | **Chart of Accounts:** `TNCA`

---

## Phase 1 — Chart of Accounts & Company Code Global Settings

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 1.1 | Create Chart of Accounts | `OB13` | Key: `TNCA`, Description: TechNova COA, Maintenance Language: EN | ☐ |
| 1.2 | Assign COA to Company Code | `OB62` | TNCA → TN01 | ☐ |
| 1.3 | Define Fiscal Year Variant | `OB29` | V3 — April 1 to March 31; 12 normal + 4 special periods | ☐ |
| 1.4 | Assign Fiscal Year Variant | `OB37` | V3 → TN01 | ☐ |
| 1.5 | Define Posting Period Variant | `OBBO` | Variant: TN01 | ☐ |
| 1.6 | Assign Posting Period Variant | `OBBP` | TN01 → TN01 | ☐ |
| 1.7 | Set Company Code Global Settings | `OBY6` | Country: IN, Currency: INR, Language: EN | ☐ |
| 1.8 | Define Field Status Variant | `OBC4` | Variant: TN01 | ☐ |
| 1.9 | Assign Field Status Variant | `OBC5` | TN01 → TN01 | ☐ |

---

## Phase 2 — GL Account Groups & Number Ranges

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 2.1 | Define Account Groups | `OBD4` | ASST (100000–199999), LIAB (200000–299999), EQTY (300000–399999), REVN (400000–499999), COGS (500000–549999), EXPN (550000–699999), TAXS (700000–799999), RECON (800000–899999) | ☐ |
| 2.2 | Define Retained Earnings Account | `OB53` | COA TNCA → GL `300200` | ☐ |
| 2.3 | Create GL Account Master Records | `FS00` | 35 accounts per `chart_of_accounts.xlsx` | ☐ |

---

## Phase 3 — Document Types & Number Ranges

| # | Activity | T-Code | Doc Type | Number Range |
|---|---|---|---|---|
| 3.1 | Define Document Types | `OBA7` | `SA` — GL Document | 1000000–1999999 |
| 3.2 | Define Document Types | `OBA7` | `KR` — Vendor Invoice | 5100000–5199999 |
| 3.3 | Define Document Types | `OBA7` | `KZ` — Vendor Payment | 5200000–5299999 |
| 3.4 | Define Document Types | `OBA7` | `DR` — Customer Invoice | 1800000–1899999 |
| 3.5 | Define Document Types | `OBA7` | `DZ` — Customer Payment | 1900000–1999999 |
| 3.6 | Define Document Types | `OBA7` | `WE` — Goods Receipt (MM auto) | via MM number range |
| 3.7 | Define Document Types | `OBA7` | `RE` — Invoice Receipt (MIRO auto) | via LIV number range |
| 3.8 | Maintain FI Number Ranges | `FBN1` | All ranges for Company Code TN01 | ☐ |

**Posting Keys (`OB41`):**

| PK | D/C | Acct Type | Use |
|---|---|---|---|
| `40` | Dr | G | Debit GL account |
| `50` | Cr | G | Credit GL account |
| `01` | Dr | D | Customer invoice |
| `15` | Cr | D | Incoming customer payment |
| `31` | Cr | K | Vendor invoice |
| `25` | Dr | K | Outgoing vendor payment |

---

## Phase 4 — Accounts Payable (FI-AP)

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 4.1 | Define Vendor Account Groups | `OBD3` | `DOME` (domestic), `FORG` (foreign/import), `SERV` (services) | ☐ |
| 4.2 | Create Number Ranges for Vendors | `XKN1` | Internal + external number ranges | ☐ |
| 4.3 | Assign Number Ranges to Vendor Acct Groups | `OBAS` | Per group | ☐ |
| 4.4 | Define Reconciliation Account | `FS00` | GL `200100` — Acct type K; block direct postings | ☐ |
| 4.5 | Define Payment Terms | `OBB8` | `NET30`, `NET45`, `2/10NET30`, `IMMED` | ☐ |
| 4.6 | Configure House Banks | `FI12` | Bank: `SBI01` — SBI Current Account GL `100200` | ☐ |
| 4.7 | Configure Automatic Payment Program | `FBZP` | Paying co. code TN01, method `T` (bank transfer), house bank `SBI01`, run weekly (Friday) | ☐ |

---

## Phase 5 — Accounts Receivable (FI-AR)

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 5.1 | Define Customer Account Groups | `OBD2` | `DOME` (domestic), `EXPG` (export), `DIST` (distributor) | ☐ |
| 5.2 | Create Number Ranges for Customers | `XDN1` | Internal + external number ranges | ☐ |
| 5.3 | Assign Number Ranges to Customer Acct Groups | `OBAR` | Per group | ☐ |
| 5.4 | Define Reconciliation Account | `FS00` | GL `100300` — Acct type D; block direct postings | ☐ |
| 5.5 | Define Credit Control Area | `OB45` | Area: `TN01`, assigned to Company Code TN01 | ☐ |
| 5.6 | Configure Dunning Procedure | `FBMP` | Procedure `TN01`: Level 1 — Reminder (Day 5), Level 2 — Warning (Day 15), Level 3 — Legal Notice (Day 30) | ☐ |

---

## Phase 6 — GST / Tax Configuration

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 6.1 | Assign Tax Procedure to Country | `OBBG` | Procedure `TAXIN` → Country `IN` | ☐ |
| 6.2 | Define Tax Codes | `FTXP` | `I1` Input GST 5%, `I3` Input IGST 18%, `O3` Output IGST 18% | ☐ |
| 6.3 | Assign GL Accounts to Tax Codes | `OB40` | I1 → GL 160100/160200; I3 → GL 160300; O3 → GL 210300 | ☐ |
| 6.4 | Maintain Tax Jurisdiction | `OBYZ` | Intra-state: CGST + SGST; Inter-state / Export: IGST | ☐ |

**Tax Code Summary:**

| Tax Code | Type | Rate | Input GL | Output GL |
|---|---|---|---|---|
| `I1` | Input GST (CGST+SGST) | 5% | 160100 / 160200 | — |
| `I3` | Input GST IGST | 18% | 160300 | — |
| `O3` | Output GST IGST | 18% | — | 210300 |

---

## Phase 7 — Asset Accounting (FI-AA)

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 7.1 | Define Chart of Depreciation | `OADB` | Chart: `IN01` (India standard) | ☐ |
| 7.2 | Assign Chart of Depreciation | `OAOB` | IN01 → TN01 | ☐ |
| 7.3 | Define Asset Classes | `OAOA` | Plant & Machinery, Office Equipment | ☐ |
| 7.4 | Define Depreciation Areas | `OADB` | Area 01 — Book depreciation (SLM) | ☐ |
| 7.5 | Assign GL Accounts for Assets | `AO90` | Asset GL 110100; Accum. Depr GL 110200; Depr Exp GL 550300 | ☐ |
| 7.6 | Run Monthly Depreciation | `AFAB` | Execute each period close | ☐ |

---

## Phase 8 — Controlling Integration (CO)

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 8.1 | Create Controlling Area | `OKKP` | Area: `TN01`, Currency: INR, Fiscal Year Variant: V3 | ☐ |
| 8.2 | Assign Company Code to Controlling Area | `OX19` | TN01 → TN01 | ☐ |
| 8.3 | Create Cost Centres | `KS01` | Per Business Area: Manufacturing, Distribution, Admin | ☐ |
| 8.4 | Create Cost Elements | `KA01` | Primary cost elements linked to P&L GL accounts | ☐ |

---

## Phase 9 — Document Splitting & New GL

| # | Activity | T-Code | Value / Config | Status |
|---|---|---|---|---|
| 9.1 | Activate New GL | `FAGL_ACTIVATION` | Activate for Company Code TN01 | ☐ |
| 9.2 | Define Document Splitting Characteristics | `FAGL_SPLIT_CHAR` | Segment / Business Area as splitting characteristics | ☐ |
| 9.3 | Define Zero-Balance Clearing Account | `FAGL_FC_TRANS` | GL `800100` — Cross-segment clearing | ☐ |

---

## Phase 10 — Period-End & Year-End Activities

| # | Activity | T-Code | Frequency |
|---|---|---|---|
| 10.1 | Open / Close Posting Periods | `OB52` | Monthly |
| 10.2 | Run Asset Depreciation | `AFAB` | Monthly |
| 10.3 | Clear GR/IR Account | `MR11` | Monthly |
| 10.4 | Automatic Clearing of Open Items | `F.13` | Monthly |
| 10.5 | Foreign Currency Revaluation | `FAGL_FC_VAL` | Monthly (if USD vendors/customers) |
| 10.6 | AP Reconciliation Report | `S_ALR_87012082` | Monthly |
| 10.7 | AR Reconciliation Report | `S_ALR_87012173` | Monthly |
| 10.8 | Trial Balance | `S_ALR_87012277` | Monthly |
| 10.9 | Balance Sheet & P&L | `S_ALR_87012284` | Monthly / Annual |
| 10.10 | Year-End Closing (Balance Carry Forward) | `F.16` | Annual |

---

> *Configuration performed in DEV (Client 100) and transported to QAS → PRD via `STMS`.*
> *All data is fictitious and for academic purposes only.*
