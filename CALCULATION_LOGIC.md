# CALCULATION_LOGIC.md
# Housing Affordability Calculator — Calculation Logic

---

## Overview

All calculations are pure JavaScript, client-side only, triggered by either live input events (mortgage outputs) or the Calculate button (everything else). There is no server, no API, no state management library. All state lives in DOM input values at the time of calculation.

---

## Data Flow

```
USER INPUTS
    │
    ├─► [Live] Purchase price + down payment % + interest rate + loan term
    │       └─► Monthly mortgage payment, down payment $, loan amount
    │               └─► [Live] Upfront cost outputs (reserves, total upfront)
    │
    ├─► [Live] Any buying cost input change
    │       └─► Default monthly savings contribution recalculates
    │
    └─► [On Calculate Button]
            │
            ├─► All buying inputs → BUY year-by-year loop → buy summary values
            ├─► All renting inputs → RENT year-by-year loop → rent summary values
            └─► Summary values → Results table + PTR display
```

---

## Order of Operations (Critical)

The following must be calculated in this order — later values depend on earlier ones:

1. **Mortgage setup** — monthly PMT, loan amount, down payment $, closing costs $, reserves $
2. **annualInterest(y) / annualPrincipal(y) / mortgageBalance(y)** — helper functions used inside the buy loop
3. **Initial deposit & monthly contribution** — resolved from toggle state (default vs manual)
4. **Buy loop** (years 1 → N) — accumulates buyCumCashflow, captures buyYear1Cost
5. **Buy summary values** — gainOnSale, buyNetReturn, buyCumSpent (derived after loop)
6. **Rent loop** (years 1 → N) — accumulates rentCumCashflow, savingsValue, captures rentYear1Cost
7. **Rent summary values** — rentCumSpent (derived after loop)
8. **Income estimates** — derived from Year 1 costs + tax rates + lifestyle spending
9. **PTR** — derived from price, closing, reserves, moving, monthly rent
10. **DOM population** — all values written to table + green highlighting applied

---

## Key Formulas

### Mortgage Payment (Monthly PMT)
Standard amortization formula using monthly compounding:
- Monthly rate = annual rate / 12
- Number of payments = loan term in years × 12
- PMT = loan × (monthlyRate × (1 + monthlyRate)^n) / ((1 + monthlyRate)^n - 1)
- If rate = 0: PMT = loan / n

### Annual Interest for Year Y
Sum of 12 monthly interest payments for that year:
- For each month m in [((y-1)×12)+1 ... y×12]:
  - Balance at start of month = loan × (1+r)^(m-1) - PMT × ((1+r)^(m-1) - 1) / r
  - Interest for month = balance × monthlyRate
- Annual interest = sum of all 12 monthly interest values

This matches Excel/Numbers behavior where IPMT with annual periods internally converts to monthly compounding.

### Annual Principal for Year Y
= (Monthly PMT × 12) - Annual Interest for Year Y

### Mortgage Balance After Year Y
Running balance after all payments through month (y × 12):
- balance = loan × (1+r)^(y×12) - PMT × ((1+r)^(y×12) - 1) / r
- Floored at 0

### PMI (Private Mortgage Insurance)
Fixed monthly dollar estimates based on down payment %:
- DP ≤ 5%: $200/month
- 5% < DP < 10%: $146/month
- 10% ≤ DP < 20%: $60/month
- DP ≥ 20%: $0 (no PMI)

PMI drops off when (mortgage balance / home value) < 0.8 (i.e. 20% equity reached).
Checked each year using prior year's balance and prior year's home value.

### Home Value
- Starts at purchase price
- Each year: homeValue = purchasePrice × (1 + appreciation)^y
- Default appreciation = 3% (matches inflation default)

### Property Tax
- Applied to prior year's home value each year (not original purchase price)
- Year 1 uses purchase price
- Year 2+ uses home value from prior year
- Formula: propTax = prevHomeValue × grossTaxRate

### Inflation-Adjusted Costs (Insurance, Maintenance, Utilities, Renters Insurance)
- Each year: cost × (1 + inflation)^(y-1)
- Year 1 (y=1): exponent = 0, so cost is unchanged
- Default inflation = 3%

### Capital Gains Tax
- Capital gains exclusion amount based on tax filing status toggle (married vs single)
- User chooses capital gains tax rate from radio (default 15%)
- User also input basis adjustment
- adjusted basis = closing costs + user input basis adjustment
- realizedGain = finalHomeValue - saleCloseCost - adjustedBasis;
- taxable gain derived from difference between realized gain and exclusion amount
- total capital gains tax = taxable gain * CG tax rate
- Total CG tax subtracted from the net return on selling the home.

---

## Buy Side — Year-by-Year Loop

**Starting cashflow (year 0):**
- buyCumCashflow = -(down payment + purchase closing costs + moving & furnishing + reserves)
- Reserves = 2 months of monthly mortgage payment

**Each year adds:**
- Annual interest (negative cashflow)
- Annual principal (negative cashflow — real money out even though it builds equity)
- Property tax (negative, applied to prior year home value)
- HOA × 12 (negative, no inflation adjustment)
- Homeowner's insurance (negative, inflation-adjusted)
- Maintenance (negative, inflation-adjusted)
- PMI if applicable (negative)
- Utilities × 12 (negative, inflation-adjusted)
- Moving & furnishing again in the FINAL year only (negative)

**Final year additionally adds:**
- Gain on sale = final home value - final mortgage balance (positive)
- Sale closing costs = final home value × sale closing cost % (negative, default 8%)

**buyYear1Cost** = sum of all annual costs in year 1 EXCLUDING moving costs (used for monthly/yearly cost display and income estimates)

**buyNetReturn** = gain on sale - sale closing costs (calculated after loop using final home value and final mortgage balance)

**buyCumSpent** = Math.abs(buyCumCashflow - buyNetReturn)
This isolates total money spent from the return, so the table shows pure outflows.

---

## Rent Side — Year-by-Year Loop

**Starting cashflow (year 0):**
- rentCumCashflow = -(security deposit + moving costs + rentReserves + initialDeposit)
- rentReserves = 1 month's rent (NOT 2 months mortgage — different from buying side)
- initialDeposit = down payment amount by default, or manual override

**Savings starting value:**
- savingsValue = initialDeposit + priorSavings

**Each year adds to cashflow:**
- Annual rent (negative, increases by rentIncreaseYoY% each year)
- Renters insurance (negative, inflation-adjusted)
- Utilities × 12 (negative, inflation-adjusted)
- Monthly savings contribution × 12 (negative cashflow — money going to savings)
- Moving costs in FINAL year only (negative)

**Savings value — updated each year (annuity due — beginning of period):**
- savingsValue = (savingsValue + contribYear) × (1 + ror)
- Contributions are added BEFORE applying growth (type=1 in Excel FV terminology)
- This matches original Numbers workbook behavior

**rentYear1Cost** = rent + insurance + utilities + savings contribution (year 1 values)
Note: savings contribution IS included in Year 1 cost for display and income estimate purposes.

**rentCumSpent** = Math.abs(rentCumCashflow)
Note: savingsValue is NOT subtracted here — rentCumCashflow already only contains outflows.
The savings return is shown separately in the results table.

---

## Summary Table Calculations

### Total Upfront Cost
- Buying: down payment + purchase closing costs + moving & furnishing + reserves (2 months PMT)
- Renting: security deposit + moving costs + rent reserves (1 month rent) + initial deposit

### Average Monthly Cost — Year 1
- Buying: buyYear1Cost / 12 (includes principal)
- Renting: rentYear1Cost / 12 (includes savings contribution)

### Total Yearly Cost — Year 1
- Buying: buyYear1Cost
- Renting: rentYear1Cost

### Cumulative Money Spent (over N years)
- Buying: Math.abs(buyCumCashflow - buyNetReturn)
- Renting: Math.abs(rentCumCashflow)

### Savings Return (after N years)
- Buying: home value at year N - mortgage balance at year N - sale closing costs
- Renting: final savingsValue (compounded savings including all contributions)

### Return Minus Cumulative Spent
- Each column: savings return - cumulative money spent
- Higher is better — green highlight on the larger value

### Recommended Annual Gross Income — 30% Rule
- Buying: (buyYear1Cost / 0.3) / (1 - taxRateBuying)
- Renting: (rentYear1Cost / 0.4) / (1 - taxRateRenting)
  - Uses 0.4 (not 0.3) for renting because the comparison includes savings (30% housing + 10% savings = 40%)
- Uses NET income as the base, then back-calculates gross via tax rate
- Shows "Enter tax rate" if tax rate fields are blank

### Recommended Annual Gross Income — Current Lifestyle
- Buying: (buyYear1Cost + lifestyleSpending) / (1 - taxRateBuying)
- Renting: (rentYear1Cost - monthlyContrib×12 + lifestyleSpending) / (1 - taxRateRenting)
  - Savings contribution is SUBTRACTED from rentYear1Cost because lifestyle spending is assumed to already include it
  - This avoids double-counting savings in the income estimate

### Price-to-Rent Ratio (PTR)
- (purchasePrice + purchaseClosingCosts + reserves + movingCosts) / (12 × monthlyRent)
- Color scale: <10 blue, 10-15 green, 15-20 yellow, 20-25 orange, >25 red

---

## Default Values and Auto-Calculations

| Field | Default | Source |
|---|---|---|
| Maintenance costs | 2% of purchase price | Auto-filled when purchase price changes |
| Inflation | 3% | Long-run US average |
| Home value appreciation | 3% | Set equal to inflation |
| Sale closing costs | 8% | Realistic US market estimate |
| Rent increase YoY | 4% | US average 2015–2025 |
| Initial deposit (renting) | Equal to down payment | Toggle default, overridable |
| Monthly savings contribution | Monthly owning cost - monthly rent | Toggle default, overridable |
| Purchase closing costs | 2% of loan amount | Discrete selector (2/3/4/5%) |
| Buying reserves | 2 months mortgage payment | Calculated |
| Renting reserves | 1 month's rent | Calculated |

---

## Live vs. On-Calculate Behavior

**Live (updates on every input keystroke):**
- Down payment amount ($)
- Loan amount
- Monthly mortgage payment
- Buying reserves
- Total upfront cash needed
- Default initial deposit display
- Default monthly contribution display
- Negative contribution warning

**On Calculate button only:**
- All results table values
- PTR
- Green highlighting
- Year-by-year data (not yet built)

---

## Input ID Reference (DOM)

| Input | ID |
|---|---|
| Purchase price | purchase-price |
| Down payment % | down-payment-pct |
| Interest rate | interest-rate |
| Loan term | loan-term (radio: term-30, term-15) |
| Moving & furnishing (buying) | moving-costs |
| Purchase closing costs | closing-pct (radio: close-2/3/4/5) |
| Property tax rate | property-tax-rate |
| HOA fees | hoa-fees |
| HO insurance | ho-insurance |
| Maintenance | maintenance-costs |
| Electric (buying) | electric-buy |
| Water (buying) | water-buy |
| Other utilities (buying) | other-util-buy |
| Inflation | inflation-rate |
| Home appreciation | home-appreciation |
| Sale closing costs | sale-closing-costs |
| Timeline | years-timeline |
| Monthly rent | monthly-rent |
| Security deposit | security-deposit |
| Renters insurance | renters-insurance |
| Moving (renting) | moving-costs-rent |
| Electric (renting) | electric-rent |
| Water (renting) | water-rent |
| Other utilities (renting) | other-util-rent |
| Prior savings | prior-savings |
| Initial deposit toggle | initial-deposit-toggle |
| Initial deposit manual | initial-deposit-manual-input |
| Monthly contrib toggle | monthly-contrib-toggle |
| Monthly contrib manual | monthly-contrib-manual-input |
| Rate of return | rate-of-return |
| Rent increase YoY | rent-increase-yoy |
| Tax rate (buying) | tax-rate-buying |
| Tax rate (renting) | tax-rate-renting |
| Lifestyle spending | lifestyle-spending |

---

## Required Fields (Validated on Calculate)

- purchase-price
- down-payment-pct
- interest-rate
- years-timeline
- monthly-rent
- rate-of-return

All other fields default to 0 if blank.
