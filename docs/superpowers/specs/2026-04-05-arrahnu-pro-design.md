# Ar-Rahnu Pro — Design Spec

## Overview

A single-file web application for calculating, tracking, and simulating Bank Rakyat Ar-Rahnu (Pajak Gadai Islam-i) pawn broking costs. Built as a personal tool with no backend — all data persists in localStorage.

## Tech Stack

- Single HTML file (no build tools)
- Vue.js 3 via CDN (reactivity, component structure)
- Tailwind CSS via CDN (styling)
- localStorage for data persistence
- Gold price API for live pricing

## UI Structure

### Header
- App title: "Ar-Rahnu Pro"
- Live gold price badge (spot price in MYR/gm, last-updated timestamp)
- Language toggle: EN / BM (persisted in localStorage)

### Color Scheme
- Dark neutral background (#1a1a2e or similar)
- Gold/amber accent (#d4a843) for highlights, buttons, active states
- Clean card-based layout with subtle shadows

### Navigation
- 3 tabs: Calculator | Tracker | Simulator
- Active tab persisted in localStorage
- Responsive: comparison mode stacks vertically on mobile screens

---

## Module 1: Calculator (Kalkulator)

### Inputs
- Gold weight (grams) — number input
- Gold karat — dropdown: 16K, 18K, 20K, 21K, 22K, 24K
- Gold price per gram — auto-filled from API, editable for manual override
- Financing percentage — slider 50%–80% (default 80%)

### Karat Purity Multipliers
| Karat | Purity |
|-------|--------|
| 24K   | 1.000  |
| 22K   | 0.916  |
| 21K   | 0.875  |
| 20K   | 0.833  |
| 18K   | 0.750  |
| 16K   | 0.667  |

### Calculations
1. **Marhun Value** = weight (gm) x gold price per gm x purity multiplier
2. **Max Financing** = marhun value x financing percentage
3. **Profit Rate Tier** (based on marhun value):
   - RM0.00–RM499.99: RM0.60 per RM100/month
   - RM500.00–RM4,999.99: RM0.75 per RM100/month
   - RM5,000.00–RM9,999.99: RM0.80 per RM100/month
   - RM10,000.00 and above: RM0.85 per RM100/month
4. **Monthly Profit** = marhun value / 100 x rate
   _(Note: Bank Rakyat charges profit based on marhun value, not financing amount. Verified from actual SAG documents where profit is calculated on the full marhun value regardless of financing percentage.)_
5. **Per 6-month cycle** = monthly profit x 6
6. **Total** = monthly profit x tenure months
7. **Total Repayment** = financing amount + total profit
8. **Annualized Profit Rate** = (total profit / financing amount) x (12 / tenure months) x 100
   _(Simple annualized rate, not compounding EAR)_

### Output Display
- Summary card with all calculated values
- Cost breakdown table: monthly, 6-month, 12-month, 18-month
- Tenure toggle: 6 / 12 / 18 months (consistent with Simulator)
- Visual indicator for profit rate tier

---

## Module 2: Tracker (Penjejak)

### Dashboard (top section)
- Total marhun value across all accounts
- Total financing outstanding
- Total monthly profit cost
- Next payment due date (earliest across accounts)
- Number of active accounts

### Account List
Each account displayed as a card showing:
- Account number and status badge (active/matured/redeemed/rolled over)
- Marhun summary (total weight, total value)
- Financing amount
- Next payment due date with countdown (days remaining)
- Monthly profit cost

### Account Data Model
```
{
  id: string (auto-generated),
  accountNo: string,
  financingAccountNo: string,
  branch: string,
  status: "active" | "matured" | "redeemed" | "rolled_over",
  createdDate: date,
  maturityDate: date,
  items: [
    {
      description: string,
      karat: number,
      weightGm: number,
      valueMYR: number
    }
  ],
  totalMarhunValue: number (computed),
  financingAmount: number,
  marginOfFinance: number (computed %),
  profitRate: number (per RM100/month),
  paymentSchedule: [
    { cycle: 1|2|3, dueDate: date, amount: number, paid: boolean }
  ],
  rolloverFrom: string|null (previous account id),
  rolloverTo: string|null (next account id, for forward traversal),
  notes: string
}
```

### Validation Rules
- **Required fields**: accountNo, createdDate, maturityDate, at least 1 item
- **Optional fields**: financingAccountNo, branch, notes
- **Items**: must have description, karat (16-24), weightGm (> 0), valueMYR (> 0)
- **Dates**: maturityDate must be after createdDate
- **Financing**: must be > 0 and <= 80% of total marhun value

### Add/Edit Account Form
- Account details fields
- Dynamic item list (add/remove marhun items)
- Auto-calculated fields (marhun value, profit rate, payment amounts)
- Payment schedule auto-generated based on dates

### Rollover Chain
- Simple timeline showing linked accounts: Account A -> Account B -> Account C
- Each node shows date and financing amount

### Data Management
- Export all accounts as JSON file download
- Import from JSON file
- Clear all data option (with confirmation)

---

## Module 3: Simulator (Simulasi)

### Scenario Builder
Inputs with sliders and number fields:
- Gold price per gram (pre-filled with current, adjustable with slider)
- Gold weight (grams)
- Gold karat (dropdown)
- Financing percentage (50%–80% slider)
- Tenure (6 / 12 / 18 months toggle)

### Live Results Panel
Updates instantly as inputs change:
- Marhun value
- Financing amount
- Monthly profit
- Total profit for selected tenure
- Total repayment
- Annualized profit rate

### Comparison Mode
- Side-by-side layout: Scenario A vs Scenario B
- Each side has independent inputs
- Difference row at bottom highlighting savings/costs

### Simulation Scenarios (preset buttons)
- **Gold price change**: "What if gold price drops 10%?" — shows impact on marhun value and financing headroom
- **Early redemption**: Enter months held, see profit paid vs remaining obligation
- **Rollover cost**: Compare cost of rolling over vs redeeming and re-pledging
- **Add more gold**: Enter additional weight, see new combined financing available

### Early Redemption Calculator
- Input: months held (1–18)
- Output: profit paid so far, profit saved vs full tenure, net cost

---

## Gold Price Integration

### Auto-Fetch
- On app load, fetch spot gold price in MYR per gram
- Primary API: frankfurter.app (free, no key, CORS-enabled) for USD/MYR rate combined with a gold spot price from a CORS-friendly source
- Fallback: If all APIs fail, use cached price from localStorage with "stale" warning badge
- If no cache exists and API fails, default to manual-entry-only mode with a prompt to enter price
- Cache response in localStorage with timestamp
- Show "last updated" badge in header

### Manual Override
- Editable gold price field in header and in each module
- Override does not persist (resets on reload unless user explicitly saves)

### Note to User
- Displayed disclaimer: Bank Rakyat's valuation rate may differ from spot price. For accurate calculations, use the bank's quoted rate.

---

## Internationalization (i18n)

### Implementation
- All UI strings stored in a translation object with `en` and `bm` keys
- Language toggle in header switches all text instantly (Vue reactivity)
- Selected language persisted in localStorage
- Number formatting: Malaysian convention (comma for thousands, dot for decimals)
- Date formatting: DD/MM/YYYY

### Scope
- All labels, headings, buttons, placeholders, tooltips, disclaimers
- Profit rate table headers
- Status badges
- Error messages

---

## Data Persistence

### localStorage Keys
- `arrahnu_accounts` — JSON array of account objects
- `arrahnu_goldPrice` — cached gold price with timestamp
- `arrahnu_language` — "en" or "bm"
- `arrahnu_simulatorState` — last used simulator inputs
- `arrahnu_activeTab` — last active tab

### Export/Import
- Export: download all localStorage data as single JSON file
- Import: upload JSON file, validate structure, then prompt user to choose:
  - "Replace all" — clear existing data, load imported data
  - "Merge (skip duplicates)" — add new accounts, skip if account ID already exists
- Validation on import: check required fields per validation rules, reject invalid entries with error message

---

## Non-Goals (out of scope)
- User authentication / multi-user
- Server-side storage or database
- Integration with Bank Rakyat systems
- Push notifications for payment reminders
- Mobile app (responsive web is sufficient)
- Other bank's Ar-Rahnu rates (Bank Rakyat only for now)
