---
name: accounting-workflow
description: "Malaysian accounting engagement workflow — produces financial statements (MFRS/MPERS), tax computations, and compilation reports from source documents. Use this skill whenever the user needs to: prepare financial statements for any Malaysian entity (Sdn Bhd, Bhd, PLT, Sole Prop, Partnership, Koperasi, Society, Trust, Foundation), process bank statements into working papers, compute Malaysian tax (Form B/C/P/PT/TF/TP), generate trial balance or general ledger, classify transactions against a chart of accounts, run QC on accounting work, or produce PDF/Excel financial reports. Also triggers when: user mentions MPERS, MFRS, IFRS in a Malaysian context; user asks about capital allowances, tax adjustments, or statutory deductions (EPF/SOCSO/EIS); user has bank statements or payslips to process; user asks about compilation reports or management accounts. Even if the user doesn't say 'accounting' explicitly — if they have financial source documents and need structured output, this skill applies."
---

# Malaysian Accounting Engagement Workflow

## Firm Configuration

```yaml
firm_name: "Hazli Johar & Co."
registration: "Chartered Accountants (NF1932)"
contact: "hazli@hazlijohar.my"
# To white-label: replace the above with your firm's details
```

## Core Principles

1. **Every number traces to a source document.** Never fabricate, estimate, or hallucinate.
2. **Check documents before asking questions.** Only ask when genuinely missing or contradictory.
3. **Standards compliance is non-negotiable.** MFRS, MPERS, ITA 1967 — no shortcuts.
4. **Zero tolerance for imbalance.** TB must balance. BS must balance. Bank must reconcile exactly.
5. **Document everything.** Every decision, assumption, and classification goes in working papers.

---

## Framework Selection Matrix

| Entity Type | Framework | Tax Filing | Basis | Reference |
|---|---|---|---|---|
| Berhad (Public Listed) | MFRS (= IFRS) | Form C | Accrual | `references/mfrs.md` |
| Sdn Bhd (Private Limited) | MPERS | Form C | Accrual | `references/mpers.md` |
| PLT (LLP) | MPERS | Form PT | Accrual | `references/entity_types.md` |
| Sole Proprietor | Accrual per S21A ITA 1967 | Form B | Accrual | `references/entity_types.md` |
| Partnership | Accrual per S21A ITA 1967 | Form P + B/BE | Accrual | `references/entity_types.md` |
| Koperasi | MCA Standards | Form TF | Accrual | `references/entity_types.md` |
| Society/Foundation | Applicable framework | Form TF | Accrual | `references/entity_types.md` |
| Trust | MFRS/MPERS (varies) | Form TP | Accrual | `references/entity_types.md` |

Read the relevant reference file based on entity type identified.

---

## Workflow — Streamlined Pipeline

The workflow operates as a **batch pipeline** for speed. Instead of processing transactions one-by-one, classify everything first, then generate outputs in a single pass.

### Phase 0: Engagement Setup (< 2 minutes)

1. Scan the client folder — identify all available documents
2. Determine entity type from registration documents (SSM extract, Form 9/13/49, LLP registration)
3. Identify financial year (FY start and end dates)
4. Check for prior year data (audited accounts, prior working papers, tax computation b/f)
5. Assess document completeness:
   - **Mandatory**: Full-year bank statements + entity registration
   - **Required for payroll**: Payslips or EA forms
   - **Required for assets**: Fixed asset register or purchase invoices > RM2,000
   - **Required for MFRS**: Prior year signed accounts (for comparatives)
6. Select framework from matrix above
7. Load appropriate COA template from `references/coa_templates/`
8. Generate client README from `references/client_readme_template.md`

If mandatory documents are missing → ASK immediately. This is a blocker.

### Phase 1: Knowledge Base Ingestion

Before classifying transactions, build context from all available sources:

**Prior Year Data** (scan client folder for these files):
- Prior year working papers (.xlsx) → extract closing balances, classifications, payee mappings
- Audit report / management accounts (PDF) → extract AJEs, closing balances
- Prior year tax computation → extract b/f losses, unabsorbed CA, S108 balance
- Client README from prior engagement → extract decisions made, special policies

**Verbal Context** (from employee during conversation):
- Accept and incorporate any context provided in chat (e.g., "Ali is a freelance videographer")
- Record all verbal context in the client README for future reference
- Verbal context from the employee is authoritative — it overrides guesses

**Prior Year Payee Mapping** (for returning clients):
Build a lookup table from prior year classifications:
```
{payee_pattern: account_code, last_amount, last_classification_note}
```
This becomes the primary classification tool for recurring transactions.

### Phase 2: Document Extraction & Auto-Classification (Batch)

Process ALL source documents in one pass:

**Bank Statements:**
- Extract every transaction: date, description, amount (DR/CR), running balance
- Verify opening balance matches prior year closing (or zero for new entities)
- Verify closing balance matches last statement
- Flag any breaks in statement continuity

**Classification Rules** (apply to ALL transactions in order of priority):

**Priority 1 — Prior year match** (returning clients):
- Same payee description → same classification as last year
- **Flag if amount changed >30%** from prior year average ("PAYMENT TO ALI was RM2,000/month last year, now RM6,000 — confirm still subcontractor?")
- **Flag if frequency changed** (was quarterly, now monthly — confirm)

**Priority 2 — Pattern matching** (all clients):
- Salary patterns → 5000 Salary & Wages
- EPF/KWSP → 2110 EPF Payable (if paying liability) or 5100 (employer portion)
- SOCSO/PERKESO → 2120 SOCSO Payable or 5110
- LHDN/IRB → 2100 Tax Payable or 9000
- Rental payments (recurring, same amount) → 5200 Rental
- Utility providers (TNB, SAJ, IWK, TM, Maxis, Celcom, Digi, U Mobile) → 5210 Utilities
- Bank charges / service charge → 5980
- Interest charges → 5990
- Adobe, Google, Microsoft, Canva, Zoom (software) → 5400

**Priority 3 — Invoice matching**:
- Match against invoices (if provided) by amount and approximate date

**Priority 4 — Unclassified** (batch for asking):
- Collect all unmatched items into a pending list (DO NOT book to Suspense yet)

### Phase 2A: Interactive Classification (AskUserQuestion)

After auto-classification is complete, present ALL unclassified items to the employee in a single batch using AskUserQuestion. This resolves ambiguity in real-time rather than generating a passive query sheet.

**How to batch questions:**

Group unclassified items by payee/pattern. For each group, ask ONE question with classification options:

```
Example AskUserQuestion:
"I found 4 monthly payments of RM3,000 to 'ALI BIN HASSAN'. How should I classify these?"
Options:
- Subcontractor / Freelancer (5030)
- Director Remuneration (5035)
- Salary & Wages - undocumented employee (5000)
- Loan repayment / Related party (1110)
```

**Rules for batching:**
- Group same-payee transactions into one question (don't ask 12 times for monthly salary)
- Present amounts and dates so the employee has context
- Offer the most likely classifications as options (based on amount, frequency, description)
- Include "I need to check with the client" as an option (→ Suspense + query sheet)
- Max 4 questions per AskUserQuestion call (tool limitation). If more, do multiple rounds.
- Use technical accounting language — the audience is firm employees

**After answers received:**
- Apply classifications to all transactions in the group
- Record decisions in client README (for next year's auto-classification)
- Any items the employee couldn't resolve → Suspense (2900) + add to Queries & Notes

**Change-flagged items** (prior year reuse with significant changes):
- Present separately: "These are classified same as last year but amounts changed significantly. Confirm or reclassify?"
- If confirmed → proceed with same classification
- If reclassified → update mapping for future years

**Payroll** (if payslips provided):
- Extract: gross salary, allowances, deductions (EPF employee, SOCSO, EIS, PCB), net pay
- Verify: does net pay match bank payment? If not, flag.
- Determine: is employer absorbing employee statutory? (net = gross means yes)

**Fixed Assets** (if invoices/register provided):
- Extract: description, date acquired, cost, category
- Apply capitalisation threshold: > RM2,000 → capitalise; ≤ RM2,000 → expense (or 100% CA)
- Assign depreciation rates per `references/depreciation_rates.md` section in entity framework

### Phase 3: Batch Journal Generation

Generate ALL journal entries as a structured dataset (not one-by-one):

1. **Opening Balance JE** (JE-001): All opening balances from prior year
2. **Bank Transaction JEs** (JE-002 to JE-NNN): One JE per classified bank transaction
3. **Payroll Accrual JEs** (monthly): Gross salary + employer statutory → payable accounts
4. **Depreciation JE**: Annual depreciation for all fixed assets (pro-rated for acquisitions)
5. **Year-End Adjustments**: Accruals, prepayments, bad debts, provisions
6. **Tax Provision JE**: Current tax expense based on computation

Output format for each JE:
```
{date, je_number, description, account_code, account_name, debit, credit}
```

### Phase 4: Workbook Generation (Single Pass)

Run the workbook generator script (`scripts/generate_workbook.py`) with the classified data to produce the complete Excel working papers in one execution:

**Sheets generated:**
1. Company Info
2. Chart of Accounts (used accounts only)
3. Bank Transactions (with classification column and running balance)
4. Payroll Summary (monthly breakdown if applicable)
5. Fixed Asset Register (with depreciation schedule)
6. Journal Entries (all JEs with narrations)
7. General Ledger (per-account with running balances)
8. Trial Balance (with opening, movement, closing columns)
9. Income Statement (grouped by category)
10. Balance Sheet (classified: current/non-current)
11. Tax Computation (adjustments, CA, tax payable)
12. Queries & Notes (all suspense items and open questions)

### Phase 5: Tax Computation

Read `references/tax_malaysia.md` for comprehensive tax rules. Compute:

1. Adjusted income (add back non-deductible, deduct non-taxable)
2. Capital allowances (IA + AA, with b/f and c/f)
3. Chargeable income
4. Tax payable (apply correct rates for entity type)
5. Tax incentives (if applicable — pioneer, ITA, RA, etc.)
6. Instalment payments comparison (CP204/CP500)

### Phase 6: Compilation Report

For compilation engagements (not audit/review):
- Prepare accountant's compilation report per ISRS 4410
- State: no assurance expressed, management responsibility, accountant's responsibility
- Attach: financial statements + notes

### Phase 7: Quality Control

Run checks from `references/qc_checklist.md`. All items in Section A (Mathematical Integrity) are **blockers** — cannot proceed to output if any fail.

Key automated checks:
- TB: Total DR = Total CR (tolerance: RM0.00)
- BS: Assets = Liabilities + Equity (tolerance: RM0.00)
- Bank: GL closing balance = bank statement closing balance
- P&L net profit ties to BS retained earnings movement
- All JEs balance (DR = CR per entry)
- No account with impossible balance (e.g., negative cash, CR receivable without credit note)

### Phase 8: Output Generation

1. **Excel Working Papers** — already generated in Phase 3
2. **PDF Financial Statements** — run `scripts/generate_pdf_report.py`
   - Cover page, TOC, Income Statement, Balance Sheet, Notes, GL, TB
   - Black & white, Helvetica, minimal professional design
   - Footer: firm name + page number
3. **Tax Computation** — included in working papers Sheet 11
4. **Client README** — update with engagement summary, decisions made, notes for next year

---

## Materiality & Precision

| Item | Threshold |
|---|---|
| Line item materiality | RM100 (below this may be grouped) |
| Rounding | 2 decimal places (sen) throughout |
| Suspense tolerance | RM0.00 |
| Bank reconciliation tolerance | RM0.00 |
| TB balance tolerance | RM0.00 |

---

## Suspense Account Policy

Suspense is the **last resort**, not the first response. The classification flow is:

1. **Auto-classify** from prior year mapping + pattern matching (Phase 2)
2. **Ask the employee** via AskUserQuestion for remaining items (Phase 2A)
3. **Only then** — items the employee couldn't resolve → 2900 Suspense
4. Add unresolved items to Queries & Notes sheet (for client follow-up)
5. Continue processing — do not halt for suspense items
6. All suspense must resolve before finalising

A well-processed engagement should have minimal or zero Suspense items after Phase 2A.
The Queries & Notes sheet is reserved for items that require CLIENT input (not employee).

---

## Communication Standards

**To employee** (technical): specific account references, debit/credit analysis, what's wrong and what's needed.

**To client** (plain English): numbered list, one question per item, actionable ("Please provide invoice for RM3,800 payment to KENZ DIGITAL on 15/06/2024").

---

## Edge Cases — Quick Reference

| Situation | Treatment |
|---|---|
| Employer absorbs employee EPF/SOCSO/EIS | Full amount to employer expense accounts |
| Director vs Employee | Directors: 5035, no SOCSO/EIS if not under contract of service |
| Related party transactions | Flag, disclose under MPERS S33 / MFRS 124 |
| Capital vs Revenue (> RM2,000) | Capitalise to PPE |
| Capital vs Revenue (≤ RM2,000) | Expense or 100% small value asset CA |
| Personal expense in business (Sole Prop) | DR 3400 Drawings |
| Foreign currency | Transaction date rate; retranslate monetary items at closing rate |
| Missing bank statements | BLOCKER — cannot proceed |
| Incomplete payroll | Use bank payments as basis, flag limitation |

---

## Reference Files

Load these as needed based on engagement requirements:

| File | When to load |
|---|---|
| `references/mfrs.md` | Entity is Berhad (public listed) |
| `references/mpers.md` | Entity is Sdn Bhd or PLT |
| `references/tax_malaysia.md` | Always (tax computation phase) |
| `references/entity_types.md` | Entity identification / framework selection |
| `references/qc_checklist.md` | Phase 6 quality control |
| `references/compilation_report.md` | Compilation engagement scope |
| `references/statutory_deductions.md` | Payroll processing |

## Scripts

| Script | Purpose |
|---|---|
| `scripts/generate_workbook.py` | Batch Excel workbook generation from classified data |
| `scripts/generate_pdf_report.py` | PDF financial statements from Excel workbook |

---

## What This Skill Never Does

1. Fabricates or estimates numbers without source documents
2. Produces an unbalanced trial balance or balance sheet
3. Skips bank reconciliation
4. Makes business decisions for the client
5. Assumes tax treatment without verification
6. Produces reports in non-standard format (always B&W, minimal, consistent)
