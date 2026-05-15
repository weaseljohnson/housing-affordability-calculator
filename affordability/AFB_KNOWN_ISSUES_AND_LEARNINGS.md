# KNOWN_ISSUES_AND_LEARNINGS.md
# Home Affordability Calculator — Known Issues & Learnings

---

## Verify Calculation Accuracy

Verify any new changes against the following inputs (all unspecified are blank or use
the automatic value):
- Purchase price: $300,000 | Down payment: 20% | Interest rate: 6% | Term: 30yr
- Property tax: 2% | Insurance: $1,800/yr | Maintenance: $6,000/yr
- Purchase closing costs: 2% | Timeline: 10 years
- Defaults: inflation 3%, appreciation 3%, sale closing costs 8%
- Tax rate: 20% | Lifestyle spending: $60,000

Results/Outputs based on those inputs:
- Monthly P&I: $1,439
- Reserves: $4,178
- Total Upfront Cost: $68,978
- Average Monthly Cost: $2,589
- Total Yearly Cost: $31,067
- Cumulative Spending: $399,850
- Net Proceeds on Sale: $170,075
- Net Financial Position: -$229,775
- 30% Rule Income: $129,446
- Lifestyle Rule Income: $113,834

Note: These are the buying-side values from the Rent vs. Buy calculator verification
inputs. The affordability calculator produces identical buy-side outputs for the same
inputs, which is the intended behavior — the two tools share the same buy-side engine.

---

## Debugging History

This calculator shares its full buy-side calculation engine with the Rent vs. Buy
Calculator. All bugs discovered and fixed during that tool's development apply here.
The following entries are carried forward from that history with notes on relevance.

---

### Bug 1 — Monthly/Yearly Cost Too Low
**Issue:** Average monthly cost for owning was off by ~$253/month
**Cause:** Principal paydown was excluded from Year 1 cost calculation
**Fix:** Include `annualPrincipal(1)` in `buyYear1Cost` accumulation
**Lesson:** Principal is real money out of pocket even though it builds equity.
Include it in cost display.

---

### Bug 2 — IPMT/PPMT Implementation Mismatch
**Issue:** Annual interest and principal figures did not match the original Numbers workbook
**Cause:** Initial implementation used annual-period math (rate=6%, nper=30) but Numbers
internally converts to monthly compounding even when annual periods are input
**Fix:** Replaced with monthly compounding summed annually:
- Loop through 12 months for each year
- Calculate balance at start of each month using standard amortization formula
- Sum 12 monthly interest charges to get annual interest
- Annual principal = (monthlyPMT × 12) - annual interest
**Lesson:** Never assume annual period formulas match spreadsheet behavior. Apple Numbers
and Excel IPMT with annual inputs use monthly compounding internally.

---

### Bug 3 — Cumulative Money Spent (Buying) — Double Subtraction
**Issue:** buyCumSpent was double-subtracting the gain on sale
**Cause:** The gain on sale was being added to buyCumCashflow inside the final year loop,
then subtracted again in `Math.abs(buyCumCashflow - buyNetReturn)`
**Fix:** Keep gain on sale out of buyCumCashflow accumulation, calculate buyNetReturn
separately, then: `buyCumSpent = Math.abs(buyCumCashflow - buyNetReturn)`
**Lesson:** Be explicit about what buyCumCashflow represents. It should be pure outflows
only during the loop, with the gain handled separately.

---

### Bug 4 — Property Tax Applied to Fixed Purchase Price
**Issue:** Property tax was not appreciating with home value
**Cause:** Tax calculation used original purchase price every year
**Fix:** Apply tax rate to prior year's home value each year. Year 1 uses purchase price.
Year 2+ uses `price × (1+appreciation)^(y-1)`
**Lesson:** Property taxes are reassessed. Using purchase price understates taxes in
later years.

---

### Bug 5 — clearErrors() Deleting Static DOM Elements
**Issue:** clearErrors() used `document.querySelectorAll(".warning-card")` which matched
all elements with that class — including the static `#validation-msg` div. On the second
Calculate press, clearErrors() removed `#validation-msg` from the DOM entirely. The click
handler then crashed on `validationMsg.style.display = 'none'` (null reference), preventing
`handleCalculate()` from ever being called.
**Fix:** Changed `showError()` to use className `"warning-card dynamic-error"` and
`clearErrors()` to `querySelectorAll(".dynamic-error")` — scoping cleanup to only
dynamically injected errors.
**Lesson:** Never use a shared visual class as a DOM selector for programmatic removal.
Dynamic elements need their own distinct class for lifecycle management.

---

### Bug 6 — Anchor Links to Non-Accordion Elements Blocked by Click Handler
**Issue:** The results nav link ("→ How to read your results") uses
`href="#acc-results-explainer"` which targets the accordion group wrapper div, not an
accordion body. The existing `a[href^="#acc-"]` click handler called `openAccordionById()`
and `e.preventDefault()` on all matching links, silently blocking the scroll without
opening anything.
**Fix:** Added a classList check before calling `openAccordionById()`. The handler now
only attempts to open an accordion if the target element has the class `"accordion-body"`.
Non-accordion targets fall through to scroll-only behavior.
**Lesson:** When intercepting anchor clicks by href pattern, always verify the target
element type before assuming it's the expected component. Pattern matching on href prefix
is too broad when the same prefix is used for both component targets and structural anchors.

---

### Bug 7 — Share Modal Listeners Silently Failing
**Issue:** Clicking the Share Results button did nothing. Close button also unresponsive.
**Cause:** `initShareModal()` was called outside any DOM-ready wrapper. The script ran
before the modal HTML was fully parsed, so all `getElementById` calls inside
`initShareModal()` returned null and listener attachment silently failed.
**Fix:** Wrapped the entire initialize block in
`document.addEventListener('DOMContentLoaded', () => { ... })`.
**Lesson:** Any init function that attaches listeners to DOM elements must run after the
DOM is ready. Silent failure with no console error is the typical symptom when listeners
are attached to null.

---

### Bug 8 — Safari Canvas Capture Blank Image
**Issue:** Share Summary produced a blank downloaded PNG in Safari.
**Cause:** The SVG foreignObject approach to canvas capture is not supported in Safari.
Safari silently renders a blank image rather than throwing an error.
**Fix:** Replaced with a fully manual canvas painting approach — each element of the share
card is drawn directly using Canvas 2D API calls. No HTML-to-canvas library or SVG
serialization involved.
**Lesson:** SVG foreignObject canvas capture is unreliable across browsers. For guaranteed
cross-browser image generation, paint manually to canvas.

---

### Bug 9 — Upfront Cost Card and Results Table Showed Different Values for Buydown
**Issue:** The Step 1 "Total Upfront Cash Needed" card did not include the buydown cost
while the results table `r-upfront-buy` cell did.
**Cause:** `updateUpfrontOutputs()` originally had no awareness of buydown cost.
`runCalculations()` correctly added `buydownUpfront` to `buyUpfront`, but the live card
used a separate code path.
**Fix:** Added buydown cost calculation to `updateUpfrontOutputs()` using the same IIFE
pattern as `runCalculations()`. Added local `noteRate` and `term` reads inside
`updateUpfrontOutputs()` since those weren't previously in scope.
**Lesson:** Any time a cost component is added to the results table upfront cell, it must
also be added to the live Step 1 upfront card. These are two separate code paths that
must stay in sync.

---

### Bug 10 — annualPrincipal Returning Phantom Value After Loan Payoff
**Issue:** After the loan balance reached zero (early payoff via extra principal),
`annualPrincipal` continued returning the full annual payment amount rather than 0.
**Cause:** Formula `(monthlyPMT × 12) - annualInterest(yearNum)` has no awareness of
whether the loan is already paid off — when annualInterest returns 0, it returns the
full payment as principal.
**Fix:** Cap the return value at `mortgageBalance(yearNum - 1)`. If the prior year's
balance is 0, return 0 immediately.
**Lesson:** Any derived formula that depends on a running balance needs a zero-balance
guard, not just the functions that track the balance directly.

---

### Bug 11 — getExtraPrincipal Nested Inside an Init Function
**Issue:** Calculator threw "Can't find variable: getExtraPrincipal" on Calculate.
**Cause:** `getExtraPrincipal()` was nested inside an init function during implementation,
making it invisible to `runCalculations()` and `calcExtraPrincipalImpact()`.
**Fix:** Moved `getExtraPrincipal()` to be a top-level helper alongside `calcPMI()`,
`getBuydownType()`, and similar helpers.
**Lesson:** Any helper function called from `runCalculations()` or from other top-level
calculation functions must itself be defined at the top level. Nesting during
implementation is easy to do accidentally — always check the closing braces.

---

### Bug 12 — CSP Blocking Locally-Hosted Script Files
**Issue:** JSZip failed to load with "Can't find variable: JSZip" despite the file being
correctly downloaded and referenced in a script tag.
**Cause:** The Content Security Policy header had `script-src 'unsafe-inline'` which only
permits inline scripts. It does not permit external script files — even when hosted on the
same domain.
**Fix:** Added `'self'` to the `script-src` directive:
`script-src 'self' 'unsafe-inline'`
Then changed the script tag to a relative path: `assets/jszip.min.js`
**Lesson:** `'unsafe-inline'` and `'self'` are independent CSP directives. Inline scripts
and local script files both need to be explicitly permitted. If you ever add another
locally-hosted script file in the future, `'self'` in `script-src` already covers it —
no CSP change needed.

---

## Known Issues


---

## Mobile Share — navigator.canShare File Guard Required
When passing image files to the Web Share API on mobile, always check
`navigator.canShare({ files: [fileToShare] })` before setting `shareData.files`. Not all
mobile browsers support file sharing via the Share API even if they support
`navigator.share`. Skipping this check causes a silent failure or uncaught error on
unsupported browsers.

---

## What NOT to Change Without Re-Testing

These areas are interdependent and easy to break:

1. The order of `buyCumCashflow` accumulation relative to `buyNetReturn` calculation
2. The `annualPrincipal` zero-balance guard — do not remove the `mortgageBalance(yearNum - 1)`
   cap
3. The maintenance prefill logic — lives inside `updateMortgageOutputs()`. Do not
   re-separate it into its own listener on `purchase-price`.
4. The `purchase-price` field has two input listeners (mortgage outputs and savings
   defaults). This is intentional. Do not consolidate them into one.
5. Do not add `maintenance-costs` to the clampOnBlur dollar inputs list — it will
   conflict with the auto-fill behavior in `updateMortgageOutputs()`.
6. Do not change `clearErrors()` to select by `".warning-card"` — it must select only
   `".dynamic-error"` to avoid deleting static DOM elements.
7. The `a[href^="#acc-"]` click handler checks `classList.contains('accordion-body')`
   before calling `openAccordionById()`. Do not remove this check.
8. Do not move the initialize block outside of
   `document.addEventListener('DOMContentLoaded', ...)` — share modal listeners and all
   other init functions will silently fail to attach.
9. `getEffectiveRate()` must be called wherever the mortgage interest rate is consumed —
   not the raw `interest-rate` input value. This includes `updateMortgageOutputs()`,
   `updateUpfrontOutputs()`, and `runCalculations()`.
10. `getTemporaryBuydownSubsidy()` and `calcTemporaryBuydownUpfront()` must receive
    `noteRate` (the raw input rate), not `rate` (the effective rate). Passing `rate` would
    cause permanent and temporary buydown logic to interfere with each other.
11. Do not subtract `buydownSubsidy` from `buyYearFullRateCost`. That variable must always
    represent full-rate year 1 costs for the results table monthly/yearly rows and income
    estimates.
12. The `updateBuydownScheduleCard()` call inside `updateMortgageOutputs()` is intentional.
    Removing it would cause the payment schedule card to go stale when the user changes
    purchase price, rate, or term.
13. The `pmi-manual-toggle` checkbox controls both the visibility of the PMI card and which
    branch of `calcPMI()` is used. Do not remove the `isManual` guard from `calcPMI()`.
    The equity drop-off check in the buy loop applies in both modes and must not be
    conditioned on the toggle.
14. `getExtraPrincipal()` is called inside both `mortgageBalance` and `annualInterest`. Do
    not inline the toggle check — keeping it in one helper ensures both functions stay in
    sync when the toggle state changes.
15. The extra principal buy loop cap (`Math.min(getExtraPrincipal() * 12, prevBalForExtra)`)
    prevents overpayment in the final year of the loan. Do not remove this cap.
16. Do not read `down-payment-pct` value directly in calculation functions — always use
    `getDpPct()`. Do not encode the raw dollar value in the URL without also encoding the
    toggle mode (`dpm`).
17. Do not reduce `toFixed(4)` to fewer decimal places in the dollar→percent down payment
    conversion. `toFixed(2)` causes floating point round-trip errors.

---

## Edge Cases to Be Aware Of

### Zero interest rate
- PMT formula would divide by zero. Handled: `if (r === 0) return loan / n`
- annualInterest would also divide by zero. Handled via same guard

### Blank required fields
- Validation on Calculate button catches: purchase price, down payment, interest rate,
  timeline
- All other blank fields default to 0 via `|| 0` pattern

### Extra principal payoff before timeline ends
- If extra payments pay off the loan before the user's chosen timeline, `mortgageBalance`
  returns 0 and the buy loop applies zero extra spend for remaining years
- `annualInterest` breaks on zero balance, returning only interest accrued up to payoff
- All non-mortgage costs (taxes, insurance, HOA, maintenance, utilities) continue
  accumulating correctly for the full timeline

### Extra principal with buydown active
- Temporary buydowns reduce the effective payment via subsidy — extra principal stacks on
  top of this and is not affected by the subsidy logic
- Permanent buydowns reduce `getEffectiveRate()`, which flows into all amortization math
  including `calcExtraPrincipalImpact`

### PMI drop-off
- Checked each year using PRIOR year's balance and PRIOR year's home value
- If prevBal / prevVal >= 0.8, PMI applies. Below 0.8 (20% equity), PMI = 0

### Timeline = 0 or blank
- Loop runs 0 times, all values = 0
- Caught by validation before reaching calculation

---

## Performance Notes

- `mortgageBalance(y)` loops through all months up to year y when extra principal is active.
  Called once per year in the buy loop. For a 30-year timeline, this means up to
  30 × 360 = 10,800 iterations. Acceptable for a calculator.
- `annualInterest(y)` similarly loops through months per call.
- No performance issues observed in practice with timelines up to 30 years.

---

## Mobile Implementation Notes

### Nav Observer — Threshold vs. rootMargin
The Intersection Observer uses `rootMargin: '-20% 0px -70% 0px'` with `threshold: 0`.
This defines a trigger band in the upper-middle of the screen and fires as soon as any
part of a section enters it, regardless of section height. This is required for reliable
nav highlighting on mobile where sections are taller than the viewport.

### Results Section Observer Detection
With an empty results section (before Calculate is pressed), the Intersection Observer
couldn't detect the section reliably because it had almost no height. Fixed by giving
`#results` a `min-height: 80vh`.

### iOS Input Auto-Zoom
iOS Safari auto-zooms into any input field with font-size below 16px. All input fields,
prefixes, and suffixes are set to exactly `font-size: 16px`. The viewport meta tag also
includes `maximum-scale=1.0, user-scalable=0`.

---

### Note — JSZip Dependency
JSZip v3.10.1 is hosted locally at `assets/jszip.min.js`. It was deliberately NOT loaded
from a CDN to keep the site free of external dependencies, rate limits, and third-party
accounts. To update in the future, download the new minified distribution from the JSZip
GitHub releases page and replace the file in `assets/`. No other changes required.