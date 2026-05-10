<p align="center">
  <img src="https://img.shields.io/badge/MPERS-Compliant-0d6efd?style=for-the-badge" alt="MPERS">
  <img src="https://img.shields.io/badge/MFRS%2FIFRS-Compliant-198754?style=for-the-badge" alt="MFRS">
  <img src="https://img.shields.io/badge/ITA%201967-Compliant-6f42c1?style=for-the-badge" alt="ITA 1967">
  <img src="https://img.shields.io/badge/Claude%20Code-Skill-f97316?style=for-the-badge" alt="Claude Code Skill">
</p>

<h1 align="center">Malaysian Accounting Workflow</h1>

<p align="center">
  <strong>AI-powered accounting engagement skill for Claude Code</strong><br>
  Produces financial statements, tax computations, and compilation reports<br>
  fully compliant with Malaysian standards.
</p>

<p align="center">
  <a href="#quick-start">Quick Start</a> &bull;
  <a href="#features">Features</a> &bull;
  <a href="#architecture">Architecture</a> &bull;
  <a href="#supported-entities">Entities</a> &bull;
  <a href="#white-labelling">White-Label</a> &bull;
  <a href="#benchmarks">Benchmarks</a>
</p>

---

## Quick Start

### Install as Claude Code Skill

```bash
# Clone the repo
git clone https://github.com/AA-Amjad/Accounting-skills.git

# Install as a skill in Claude Code
claude skill install ./Accounting-skills
```

### Or use directly

Point Claude Code at the `SKILL.md` file in your project, or copy the `accounting-workflow/` directory into your `.claude/skills/` folder.

### Trigger the skill

The skill activates when you:
- Ask to prepare financial statements for a Malaysian entity
- Provide bank statements or payslips to process
- Ask about MPERS, MFRS, capital allowances, or statutory deductions
- Need a tax computation (Form B/C/P/PT/TF/TP)
- Want to classify transactions against a chart of accounts

---

## Features

### Compliance Coverage

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   ACCOUNTING STANDARDS          TAX LAW                         │
│   ──────────────────           ────────                         │
│   ✓ MFRS (= IFRS)             ✓ Form C (Companies)             │
│   ✓ MPERS (Private)           ✓ Form B (Individuals)           │
│   ✓ MCA (Cooperatives)        ✓ Form P (Partnerships)          │
│                                ✓ Form PT (LLP/PLT)              │
│   ENTITY TYPES                 ✓ Form TF (Koperasi)             │
│   ────────────                 ✓ Form TP (Trusts)               │
│   ✓ Berhad (Public Listed)                                      │
│   ✓ Sdn Bhd (Private)         TAX INCENTIVES                   │
│   ✓ PLT / LLP                 ──────────────                   │
│   ✓ Sole Proprietor           ✓ Pioneer Status                  │
│   ✓ Partnership               ✓ Investment Tax Allowance        │
│   ✓ Koperasi                  ✓ Reinvestment Allowance          │
│   ✓ Society / Foundation      ✓ MSC Malaysia                    │
│   ✓ Trust                     ✓ Green Tech (GITA/GITE)          │
│                                ✓ Transfer Pricing (S140A)        │
│                                ✓ Earnings Stripping (S140C)      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Workflow Pipeline

```
 ┌──────────┐    ┌──────────────┐    ┌────────────────┐    ┌──────────┐
 │  PHASE 0 │───▶│   PHASE 1    │───▶│   PHASE 2      │───▶│ PHASE 2A │
 │  Setup   │    │  Knowledge   │    │ Auto-Classify  │    │   ASK    │
 │          │    │   Base       │    │                │    │ Employee │
 └──────────┘    └──────────────┘    └────────────────┘    └────┬─────┘
                                                                 │
      ┌──────────────────────────────────────────────────────────┘
      ▼
 ┌──────────┐    ┌──────────────┐    ┌────────────────┐    ┌──────────┐
 │ PHASE 3  │───▶│   PHASE 4    │───▶│   PHASE 5      │───▶│ PHASE 6  │
 │ Journals │    │  Workbook    │    │    Tax         │    │Compilatn │
 │ (Batch)  │    │ (Single Pass)│    │  Computation   │    │  Report  │
 └──────────┘    └──────────────┘    └────────────────┘    └────┬─────┘
                                                                 │
      ┌──────────────────────────────────────────────────────────┘
      ▼
 ┌──────────┐    ┌──────────────┐
 │ PHASE 7  │───▶│   PHASE 8    │
 │    QC    │    │   Output     │
 │ (42 pts) │    │  Excel + PDF │
 └──────────┘    └──────────────┘
```

### Interactive Classification

The skill doesn't just dump unknowns to Suspense. It actively resolves ambiguity:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. AUTO-CLASSIFY          Prior year mapping + patterns        │
│     ────────────           (handles ~80% of transactions)       │
│                                                                 │
│  2. ASK EMPLOYEE           Batch unclear items via              │
│     ────────────           AskUserQuestion (resolves ~18%)      │
│                                                                 │
│  3. SUSPENSE               Only genuinely unresolvable          │
│     ────────               items reach here (~2%)               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Prior Year Intelligence

For returning clients, the skill:
- Auto-reuses last year's payee classifications
- Flags if amounts changed >30% ("ALI was RM2k/month, now RM6k — confirm?")
- Flags if frequency changed (quarterly → monthly)
- Records all decisions for next year's auto-classification

---

## Architecture

```
accounting-workflow/
│
├── SKILL.md                          ← Core instructions (325 lines)
│                                       Workflow phases, decision trees,
│                                       triggers, interactive classification
│
├── references/                       ← Loaded on-demand by entity type
│   ├── mfrs.md                        MFRS/IFRS framework (Berhad)
│   ├── mpers.md                       MPERS framework (Sdn Bhd, PLT)
│   ├── tax_malaysia.md                Comprehensive tax law
│   ├── entity_types.md                All 8 entity types + selection tree
│   ├── statutory_deductions.md        EPF/SOCSO/EIS/PCB/HRDF rates
│   ├── qc_checklist.md                42-point quality control
│   ├── compilation_report.md          ISRS 4410 compilation engagement
│   ├── client_readme_template.md      Engagement memory template
│   └── coa_templates/                 Chart of Accounts per entity
│       ├── coa_sdn_bhd.json            (93 accounts)
│       ├── coa_bhd.json                (98 accounts, MFRS-specific)
│       ├── coa_plt.json                (76 accounts)
│       ├── coa_sole_prop.json          (74 accounts)
│       ├── coa_partnership.json        (76 accounts)
│       └── coa_koperasi.json           (62 accounts)
│
└── scripts/                          ← Executable tools
    ├── generate_workbook.py            Batch Excel generation (openpyxl)
    └── generate_pdf_report.py          PDF financial statements (reportlab)
```

### Progressive Disclosure

The skill loads only what's needed:

| Trigger | Files Loaded |
|---|---|
| Any engagement | `SKILL.md` (always in context) |
| Sdn Bhd / PLT | + `mpers.md` + `coa_sdn_bhd.json` or `coa_plt.json` |
| Berhad (public) | + `mfrs.md` + `coa_bhd.json` |
| Tax computation | + `tax_malaysia.md` |
| Payroll processing | + `statutory_deductions.md` |
| Quality control | + `qc_checklist.md` |

---

## Supported Entities

| Entity | Framework | Tax Form | Depth |
|---|---|---|---|
| Berhad (Public) | MFRS | Form C | Full |
| Sdn Bhd (Private) | MPERS | Form C | Full |
| PLT / LLP | MPERS | Form PT | Full |
| Sole Proprietor | S21A ITA | Form B | Full |
| Partnership | S21A ITA | Form P | Full |
| Koperasi | MCA | Form TF | Lite |
| Society / Foundation | Applicable | Form TF | Lite |
| Trust | MFRS/MPERS | Form TP | Lite |

**Full** = complete COA, workflow, tax computation, all edge cases handled.
**Lite** = key differences documented, COA template provided, core workflow applies.

---

## Outputs

### Excel Working Papers (12 sheets)

```
┌─── Company Info
├─── Chart of Accounts
├─── Bank Transactions (classified, running balance)
├─── Payroll Summary
├─── Fixed Asset Register (with depreciation schedule)
├─── Journal Entries (all JEs with narrations)
├─── General Ledger (per-account, running balances)
├─── Trial Balance (opening → movement → closing)
├─── Income Statement
├─── Balance Sheet (classified current/non-current)
├─── Tax Computation
└─── Queries & Notes
```

### PDF Financial Statements

- Cover page with firm branding
- Table of Contents (page-numbered)
- Statement of Comprehensive Income
- Statement of Financial Position
- Trial Balance
- General Ledger (detailed)
- Notes (if applicable)

Format: Black & white, Helvetica, minimal professional design.

---

## White-Labelling

To use this skill for your own firm, update the config block at the top of `SKILL.md`:

```yaml
firm_name: "Your Firm Name"
registration: "Your Registration (e.g., CA12345)"
contact: "your@email.com"
```

All outputs (PDF footers, working paper headers, compilation reports) will use your firm's details.

---

## Benchmarks

Tested against Claude without the skill on 3 scenarios (30 assertions total):

| Metric | With Skill | Without Skill |
|---|---|---|
| **Pass Rate** | **100%** | 73.3% |
| Critical Errors | 0 | 3 |
| Compliance Failures | 0 | 8 |

### Critical error prevented

Without the skill, Claude treated a PLT (Limited Liability Partnership) as a traditional partnership — adding back RM120,000 in partners' remuneration, stating SME rates don't apply, and using the wrong tax form. This single error would cause **RM18,000+ in excess tax**.

With the skill: zero compliance errors across all entity types.

---

## Dependencies

| Script | Requires |
|---|---|
| `generate_workbook.py` | `openpyxl` |
| `generate_pdf_report.py` | `reportlab`, `openpyxl` |

```bash
pip install openpyxl reportlab
```

---

## Key Principles

| # | Principle |
|---|---|
| 1 | Every number traces to a source document |
| 2 | Check documents before asking questions |
| 3 | Standards compliance is non-negotiable |
| 4 | Zero tolerance for imbalance (TB, BS, bank rec) |
| 5 | Document everything in working papers |

---

## License

MIT

---

<p align="center">
  <sub>Built for <strong>Hazli Johar & Co.</strong> Chartered Accountants (NF1932)</sub><br>
  <sub>Designed for Claude Code &bull; Works with Claude Opus, Sonnet & Haiku</sub>
</p>
