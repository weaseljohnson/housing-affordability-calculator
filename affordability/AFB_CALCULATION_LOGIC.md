# CALCULATION_LOGIC.md
# Home Affordability Calculator — Calculation Logic

---

## Overview

All calculations are pure JavaScript, client-side only, triggered by either live input events (mortgage outputs) or the Calculate button (everything else). There is no server, no API, no state management library. All state lives in DOM input values at the time of calculation.

---

## Data Flow

USER INPUTS
│
├─► [Live] Purchase price + down payment % + interest rate + loan term
│       └─► Monthly mortgage payment, down payment $, loan amount
│               └─► [Live] Upfront cost outputs (reserves, total upfront)
│
└─► [On Calculate Button]
│
└─► All buying inputs → BUY year-by-year loop → buy summary values
└─► Summary values → Results table

---

## Order of Operations (Critical)

The following must be calculated in this order — later values depend on earlier ones:

1. **Mortgage setup** — monthly PMT, loan amount, down payment $, closing costs $, reserves $
2. **annualInterest(y) / annualPrincipal(y) / mortgageBalance(y)** — helper functions used inside the buy loop
3. **Buy loop** (years 1 → N) — accumulates buyCumCashflow, captures buyYear1Cost
4. **Buy summary values** — gainOnSale, buyNetReturn, buyCumSpent (derived after loop)
5. **Income estimates** — derived from Year 1 costs + tax rates + lifestyle spending
6. **DOM population** — all values written to table

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

### PMI — Manual Override
- The PMI manual toggle (`pmi-manual-toggle`) switches `calcPMI()` between two modes:
  - **Auto:** Tiered fixed monthly estimates based on down payment % (existing logic)
  - **Manual:** Returns the raw value from `pmi-manual-input`, clamped to 0 minimum
- In both modes, the equity drop-off check in the buy loop still applies:
  if `prevBal / prevVal < 0.8` (20% equity reached), PMI = 0 for that year regardless
  of the base value returned by `calcPMI()`
- The PMI output card is hidden when auto mode returns 0 (down payment ≥ 20%).
  In manual mode, the card is always visible so the user can edit the value.
- Manual PMI value and toggle state are included in the share URL encoding
  (`pmt` = toggle state, `pmv` = manual value)

### Extra Principal Paydown

When the extra principal toggle is enabled, a user-specified dollar amount is applied to principal each month on top of the regular mortgage payment. This changes the behavior of three core helpers:

**`mortgageBalance(afterYear)`** — When extra > 0, switches from the closed-form formula to a month-by-month loop that subtracts both the regular principal portion and the extra amount each month, floored at zero.

**`annualInterest(yearNum)`** — When extra > 0, runs from month 1 to the end of `yearNum` tracking the actual balance, accumulating only the months within that year. This correctly reflects that earlier extra payments reduce the balance that later months' interest is calculated against.

**`annualPrincipal(yearNum)`** — Unchanged in formula: `(monthlyPMT × 12) - annualInterest(yearNum)`. Because `annualInterest` now returns a lower value, `annualPrincipal` automatically reflects the accelerated paydown.

**`getExtraPrincipal()`** — Helper that returns the extra monthly amount if the toggle is on, or 0 if off. Both `mortgageBalance` and `annualInterest` call this so the toggle cleanly reverts all math to standard amortization when off.

**Buy loop** — `extraSpendYear` is added to `yearSpend` each year, capped at the remaining balance so the loop never overpays. Extra principal counts as money spent (it builds equity but is real cash out of pocket).

**PMI drop-off** — Because `mortgageBalance` now returns a lower balance in earlier years when extra payments are active, PMI drops off sooner automatically.

**Impact output card** — Calculated independently of the main Calculate button, triggered only when the extra payment field changes. Runs two separate month-by-month loops (standard and with extra) and computes:
- Interest saved over the user's chosen timeline
- Full loan payoff time reduction (total months difference between the two loops)

**Edge case — early payoff within timeline** — If extra payments pay off the loan before the timeline ends, the buy loop checks `balances[y-1] > 0` before applying extra spend, and `annualInterest`/`mortgageBalance` both break on zero balance. The user continues to own the home and pays taxes, insurance, HOA, maintenance, and utilities — just with all mortgage-related costs zeroed.

### Home Value
- Starts at purchase price
- Each year: homeValue = purchasePrice × (1 + appreciation)^y
- Default appreciation = 3% (matches inflation default)

### Property Tax
- Applied to prior year's home value each year (not original purchase price)
- Year 1 uses purchase price
- Year 2+ uses home value from prior year
- Formula: propTax = prevHomeValue × grossTaxRate

### Inflation-Adjusted Costs (Insurance, Maintenance, Utilities)
- Each year: cost × (1 + inflation)^(y-1)
- Year 1 (y=1): exponent = 0, so cost is unchanged
- Default inflation = 3%

### Capital Gains Tax on Home Sale
Only applied when the capital gains toggle is enabled in Advanced Settings.

- adjustedBasis = purchasePrice + purchaseClosingCosts + basisAdjustment
- saleCloseCost = finalHomeValue × saleCloseRate
- realizedGain = finalHomeValue - saleCloseCost - adjustedBasis
- taxableGain = max(0, realizedGain - exclusion)
  - exclusion = $250,000 (single) or $500,000 (married filing jointly)
- capGainsTax = taxableGain × capitalGainsRate (0%, 15%, or 20%)
- buyNetReturn = gainOnSale - saleCloseCost - capGainsTax

Off by default. Most users will see $0 tax since gains rarely exceed the exclusion on typical timelines and price points.

---

## Mortgage Buydown Options

Buydown settings live in the Advanced Settings accordion of Step 1. All buydown logic is optional and off by default.

### Buydown Types

**Temporary buydowns (2-1 and 3-2-1)** reduce the borrower's effective payment for the first 2–3 years via an upfront subsidy fund, then revert to the full note rate. The underlying loan rate never changes — the subsidy covers the difference.

- 2-1: Year 1 = note rate − 2%, Year 2 = note rate − 1%, Year 3+ = full note rate
- 3-2-1: Year 1 = note rate − 3%, Year 2 = note rate − 2%, Year 3 = note rate − 1%, Year 4+ = full note rate

**Permanent buydowns** use discount points paid at closing to reduce the note rate for the full loan term. Each point costs 1% of the loan amount and reduces the rate by a lender-specified amount (default 0.25% per point).

### Key Variables

| Variable | What it is |
|---|---|
| `noteRate` | The original interest rate from the input field — always the full note rate |
| `rate` / `getEffectiveRate()` | The rate used for mortgage math — equals `noteRate` for temporary buydowns, reduced by `(points × ratePerPoint)` for permanent |
| `getTemporaryBuydownSubsidy(y, loan, noteRate, term)` | Annual subsidy for year `y` = `(fullPMT - reducedPMT) × 12`. Returns 0 if no subsidy applies |
| `calcTemporaryBuydownUpfront(loan, noteRate, term)` | Total subsidy cost = sum of annual subsidies across all buydown years |
| `calcPermanentBuydownUpfront(loan)` | Point cost = `loan × (points / 100)` |

### How Buydowns Affect the Buy Loop

Inside the buy year-by-year loop, `yearSpend` is reduced by `getTemporaryBuydownSubsidy(y, ...)` for the applicable years.

`buyYearFullRateCost` is calculated at `y === 1` without the subsidy deduction. This is intentional — it represents the full-rate cost used for the monthly/yearly display rows and income estimates, labeled "After Buydown" when a temporary buydown is active.

### Upfront Cost Impact

If the payer toggle is set to "I'm paying it":
- Temporary buydown: subsidy total is added to `buyUpfront` and to the Step 1 "Total Upfront Cash Needed" card
- Permanent buydown: point cost is added to `buyUpfront` and to the Step 1 "Total Upfront Cash Needed" card

If payer is "Seller / Builder", no buydown cost is added to any upfront output.

### Results Table Behavior

- **Permanent buydown:** All results reflect the reduced rate automatically. Row labels remain "Year 1".
- **Temporary buydown:** Monthly and yearly cost rows are relabeled to "After Buydown (Yr N+)" where N is the first full-rate year (3 for 2-1, 4 for 3-2-1). Values show full-rate costs, not subsidized costs.

### Buydown Payment Schedule Card

Visible inside Advanced Settings when a temporary buydown is active. Shows the monthly payment for each subsidized year, the full note rate payment, and (if buyer is paying) the total upfront subsidy cost. Updates live as purchase price, down payment, rate, and term change.

### Helper Functions (Do Not Break)

- `getEffectiveRate()` must be used everywhere the mortgage rate is needed for calculations — not the raw `interest-rate` input value
- `getTemporaryBuydownSubsidy()` must receive `noteRate`, not `rate`, so the subsidy is always computed against the true note rate regardless of permanent buydown settings
- `calcTemporaryBuydownUpfront()` similarly must receive `noteRate` and `term`
- Do not add `buydownSubsidy` subtraction to `buyYearFullRateCost` — that variable intentionally reflects full-rate costs for the results table label rows

---

## Buy Side — Year-by-Year Loop

**Starting cashflow (year 0):**
- buyCumCashflow = -(down payment + purchase closing costs + moving & furnishing + reserves)
- Reserves = 2 months of (monthly mortgage payment + insurance/12 + monthly property tax)

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
- Buydown subsidy deducted from yearSpend for applicable years (positive offset)
- Extra principal added to yearSpend each year, capped at remaining balance (negative)

**Final year additionally:**
- Gain on sale = final home value - final mortgage balance (used for buyNetReturn calculation)
- Sale closing costs = final home value × sale closing cost % (negative, default 8%)

**buyYear1Cost** = sum of all annual costs in year 1 EXCLUDING moving costs and buydown subsidy (used for monthly/yearly cost display and income estimates)

**buyNetReturn** = gainOnSale - saleCloseCost - capGainsTax (calculated after loop using final home value and final mortgage balance)

**buyCumSpent** = Math.abs(buyCumCashflow - buyNetReturn)
This isolates total money spent from the return, so the table shows pure outflows.

---

## Summary Table Calculations

### Total Upfront Cost
Down payment + purchase closing costs + moving & furnishing + reserves (2 months PITI) + buydown cost if buyer is paying

### Average Monthly Cost — Year 1
buyYear1Cost / 12 (includes principal; excludes buydown subsidy)

### Total Yearly Cost — Year 1
buyYear1Cost (same as above × 12)

### Cumulative Spending (over N years)
Math.abs(buyCumCashflow - buyNetReturn)

### Net Proceeds on Sale (after N years)
Final home value - final mortgage balance - sale closing costs - capital gains tax (if enabled)

### Net Financial Position
Net Proceeds on Sale - Cumulative Spending
Higher is better.

### Recommended Annual Gross Income — 30% Rule
(buyYear1Cost / 0.3) / (1 - taxRateBuying)
- Uses net income as the base (30% of take-home), then back-calculates gross via tax rate
- Shows "Enter tax rate" if tax rate field is blank

### Recommended Annual Gross Income — Current Lifestyle
(buyYear1Cost + lifestyleSpending) / (1 - taxRateBuying)
- Shows "Enter tax rate" if tax rate is blank; "Enter lifestyle spending" if lifestyle is blank

---

## Default Values and Auto-Calculations

| Field | Default | Source |
|---|---|---|
| Maintenance costs | 2% of purchase price | Auto-filled when purchase price changes |
| Inflation | 3% | Long-run US average |
| Home value appreciation | 3% | Set equal to inflation |
| Sale closing costs | 8% | Realistic US market estimate |
| Purchase closing costs | 2% of loan amount | Discrete selector (2/3/4/5%) |
| Buying reserves | 2 months PITI | Calculated (mortgage + insurance/12 + monthly tax) |
| Filing status | Single | Radio toggle in Advanced Settings |
| Capital gains toggle | Off | Must be manually enabled |
| Capital gains rate | 15% | Pre-selected; 0/15/20% options |
| Basis adjustment | $0 | Manual entry when capital gains is enabled |
| Extra principal paydown | Off (toggle) | User opt-in in Advanced Settings |

---

## Live vs. On-Calculate Behavior

**Live (updates on every input keystroke):**
- Down payment amount ($)
- Loan amount
- Monthly mortgage payment
- Buying reserves
- Total upfront cash needed
- Maintenance pre-fill (2% of purchase price)
- PMI card visibility and value

**On Calculate button only:**
- All results table values
- Green highlighting (if applicable)

---

## Input ID Reference (DOM)

| Input | ID |
|---|---|
| Purchase price | purchase-price |
| Down payment % or $ | down-payment-pct |
| Down payment mode toggle | dp-mode-toggle |
| Interest rate | interest-rate |
| Loan term | loan-term (radio: term-30, term-15) |
| Moving & furnishing | moving-costs |
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
| Tax rate | tax-rate-buying |
| Lifestyle spending | lifestyle-spending |
| Extra principal toggle | extra-principal-toggle |
| Extra principal amount | extra-principal-amount |
| PMI manual toggle | pmi-manual-toggle |
| PMI manual value | pmi-manual-input |
| Buydown toggle | buydown-toggle |
| Buydown type | buydown-type (radio) |
| Buydown payer | buydown-payer (radio) |
| Discount points | buydown-points |
| Rate per point | buydown-rate-per-point |
| Filing status | filing-status (radio) |
| Capital gains toggle | capital-gains-toggle |
| Capital gains rate | cap-gains-rate (radio) |
| Basis adjustment | basis-adjustment |

Note: `getExtraPrincipal()` must be defined as a top-level function (not nested inside any init or wireup function) so it is accessible from `runCalculations()` and `calcExtraPrincipalImpact()`.

---

## Required Fields (Validated on Calculate)

- purchase-price
- down-payment-pct
- interest-rate
- years-timeline

All other fields default to 0 if blank.