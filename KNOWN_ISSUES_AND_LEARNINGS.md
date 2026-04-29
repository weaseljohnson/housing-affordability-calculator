# KNOWN_ISSUES_AND_LEARNINGS.md
# Housing Affordability Calculator — Known Issues & Learnings

---

## Verified Calculation Accuracy

All calculations have been verified against the original Apple Numbers workbook using these test inputs:
- Purchase price: $300,000 | Down payment: 20% | Interest rate: 6% | Term: 30yr
- Property tax: 2.2% | HOA: $25/mo | Insurance: $1,750/yr | Maintenance: $6,000/yr
- Utilities (buying): $300/mo total | Moving (buying): $5,000
- Purchase closing costs: 2% | Timeline: 10 years
- Defaults: inflation 3%, appreciation 3%, sale closing costs 8%
- Monthly rent: $1,740 | Security deposit: $0 | Renters insurance: $150/yr
- Moving (renting): $0 | Utilities (renting): $200/mo total
- Prior savings: $16,000 | Initial deposit: manual $0 | Monthly contrib: manual $500
- Rate of return: 3.75% | Rent increase YoY: 4%
- Tax rates: 22% buying, 14% renting | Lifestyle spending: $62,400

Results match the workbook within acceptable rounding tolerance on all owning values. Renting values match after fixing known bugs documented below.

---

## Debugging History

### Bug 1 — Monthly/Yearly Cost Too Low
**Issue:** Average monthly cost for owning was off by ~$253/month
**Cause:** Principal paydown was excluded from Year 1 cost calculation
**Fix:** Include `annualPrincipal(1)` in `buyYear1Cost` accumulation
**Lesson:** Principal is real money out of pocket even though it builds equity. Include it in cost display.

---

### Bug 2 — IPMT/PPMT Implementation Mismatch
**Issue:** Annual interest and principal figures did not match Numbers workbook
**Cause:** Initial implementation used annual-period math (rate=6%, nper=30) but Numbers internally converts to monthly compounding even when annual periods are input
**Fix:** Replaced with monthly compounding summed annually:
- Loop through 12 months for each year
- Calculate balance at start of each month using standard amortization formula
- Sum 12 monthly interest charges to get annual interest
- Annual principal = (monthlyPMT × 12) - annual interest
**Lesson:** Never assume annual period formulas match spreadsheet behavior. Apple Numbers and Excel IPMT with annual inputs use monthly compounding internally.

---

### Bug 3 — Cumulative Money Spent (Buying) — Double Subtraction
**Issue:** buyCumSpent was double-subtracting the gain on sale
**Cause:** The gain on sale was being added to buyCumCashflow inside the final year loop, then subtracted again in `Math.abs(buyCumCashflow - buyNetReturn)`
**Fix:** Keep gain on sale out of buyCumCashflow accumulation, calculate buyNetReturn separately, then: `buyCumSpent = Math.abs(buyCumCashflow - buyNetReturn)`
**Lesson:** Be explicit about what buyCumCashflow represents. It should be pure outflows only during the loop, with the gain handled separately.

---

### Bug 4 — Cumulative Money Spent (Renting) — Double Subtraction
**Issue:** rentCumSpent was ~$95k too high
**Cause:** `Math.abs(rentCumCashflow - savingsValue)` was double-subtracting savings. rentCumCashflow already only contains outflows (savings contributions are treated as negative cashflow, i.e. money leaving the user's pocket). savingsValue is tracked separately and shown in its own results row.
**Fix:** `rentCumSpent = Math.abs(rentCumCashflow)` — no savings value subtraction needed
**Lesson:** Keep cumulative cashflow and savings value as completely separate accumulators. Don't mix them.

---

### Bug 5 — Savings Value Compounding (Annuity Due vs Ordinary Annuity)
**Issue:** Renting savings return was off by ~$3,000 over 10 years
**Cause:** Implementation used end-of-period compounding: `savingsValue = savingsValue * (1+ror) + contribYear` (ordinary annuity)
**Fix:** Use beginning-of-period compounding: `savingsValue = (savingsValue + contribYear) * (1+ror)` (annuity due, type=1)
**Lesson:** Always check FV formula type. The original workbook used FV with type=1 (beginning of period). This produces meaningfully different results over 10+ years.

---

### Bug 6 — Renting Reserves Used Wrong Value
**Issue:** Renting cumulative cashflow was off by ~$1,138
**Cause:** Renting reserves were calculated as 2 months of mortgage payment (same as buying), but the workbook uses 1 month's rent
**Fix:** `const rentReserves = rent` (1 month's rent, not mortgage-based)
**Lesson:** Buying reserves (lender requirement) and renting reserves (first/last month convention) are different things. Do not share the same variable.

---

### Bug 7 — Property Tax Applied to Fixed Purchase Price
**Issue:** Property tax was not appreciating with home value
**Cause:** Tax calculation used original purchase price every year
**Fix:** Apply tax rate to prior year's home value each year. Year 1 uses purchase price. Year 2+ uses `price × (1+appreciation)^(y-1)`
**Lesson:** Property taxes are reassessed. Using purchase price understates taxes in later years.

---

### Bug 8 — Renting Lifestyle Income Estimate Double-Counting Savings
**Issue:** Renting lifestyle income estimate was ~$14k too high
**Cause:** Formula was `(rentYear1Cost + lifestyle + monthlyContrib×12) / (1-taxRate)` — adding savings contribution twice, since rentYear1Cost already includes savings contributions
**Fix:** `(rentYear1Cost - monthlyContrib×12 + lifestyle) / (1-taxRate)` — subtract savings from rentYear1Cost before adding lifestyle, since lifestyle spending is assumed to already account for the user's savings habit
**Lesson:** The lifestyle spending input explicitly includes the user's savings in their mental model of "what I spend." Don't add savings separately on top of it.

---

### Bug 9 — Wrong Tax Rate Used for Renting 30% Rule (Workbook Bug)
**Issue:** Original workbook was using the buying tax rate for the renting 30% rule calculation
**Cause:** Copy/paste error in the original Numbers workbook, not in the web implementation
**Resolution:** Web implementation uses the correct renting tax rate. This is actually more accurate than the original workbook.

---

### Bug 10 — clearErrors() Deleting Static DOM Elements
Issue: clearErrors() used document.querySelectorAll(".warning-card") which matched
all elements with that class — including the static #validation-msg div and
#negative-contrib-warning. On the second Calculate press, clearErrors() removed
#validation-msg from the DOM entirely. The click handler then crashed on
validationMsg.style.display = 'none' (null reference), preventing handleCalculate()
from ever being called.
Fix: Changed showError() to use className "warning-card dynamic-error" and
clearErrors() to querySelectorAll(".dynamic-error") — scoping cleanup to only
dynamically injected errors.
Lesson: Never use a shared visual class as a DOM selector for programmatic
removal. Dynamic elements need their own distinct class for lifecycle management.

### Bug 11 — Anchor Links to Non-Accordion Elements Blocked by Click Handler
Issue: The results nav link ("→ How to read your results") uses href="#acc-results-explainer"
which targets the accordion group wrapper div, not an accordion body. The existing
a[href^="#acc-"] click handler called openAccordionById() and e.preventDefault() on
all matching links, silently blocking the scroll without opening anything.
Fix: Added a classList check before calling openAccordionById(). The handler now
only attempts to open an accordion if the target element has the class "accordion-body".
Non-accordion targets fall through to scroll-only behavior.
Lesson: When intercepting anchor clicks by href pattern, always verify the target
element type before assuming it's the expected component. Pattern matching on href
prefix is too broad when the same prefix is used for both component targets and
structural anchors.

### Bug 12 — Share Modal Listeners Silently Failing
**Issue:** Clicking the Share Results button did nothing. Close button and tabs also unresponsive.
**Cause:** `initShareModal()` was called at the bottom of the `<script>` block, outside any DOM-ready wrapper. The script ran before the modal HTML was fully parsed, so all `getElementById` calls inside `initShareModal()` returned null and listener attachment silently failed.
**Fix:** Wrapped the entire initialize block in `document.addEventListener('DOMContentLoaded', () => { ... })`.
**Lesson:** Any init function that attaches listeners to DOM elements must run after the DOM is ready. Silent failure with no console error is the typical symptom when listeners are attached to null.

### Bug 13 — sc-url-display Null Reference in populateShareCard
**Issue:** If the `id="sc-url-display"` element is removed from the modal HTML, `populateShareCard` throws a null reference error on every tab switch and modal open.
**Cause:** After removing the URL display element from the modal HTML, the JS still referenced it with `document.getElementById('sc-url-display').textContent = url`.
**Fix:** Remove these two lines from `populateShareCard` if the element is not present in the HTML:
```javascript
const url = encodeStateToUrl();
document.getElementById('sc-url-display').textContent = url;
```
**Lesson:** When removing HTML elements, always search the JS for any references to their IDs and remove those too.

### Bug 14 — Safari Canvas Capture Blank Image
**Issue:** Share Summary produced a blank downloaded PNG in Safari.
**Cause:** The SVG foreignObject approach to canvas capture is not supported in Safari. Safari silently renders a blank image rather than throwing an error.
**Fix:** Replaced with a fully manual canvas painting approach — each element of the share card (header, table rows, PTR badge, footer) is drawn directly using Canvas 2D API calls. No HTML-to-canvas library or SVG serialization involved.
**Lesson:** SVG foreignObject canvas capture is unreliable across browsers. For guaranteed cross-browser image generation, paint manually to canvas. It is more code but the only approach that works everywhere.

### Bug 15 — PTR Badge Rendered as Hourglass Shape on Canvas
**Issue:** The PTR badge on the downloaded PNG appeared as two squished, wide, pointy hourglass shapes rather than a pill shape.
**Cause:** The custom `roundRect` helper function conflicted with Safari's internal `ctx.roundRect` implementation, producing unexpected path geometry.
**Fix:** Replaced the `roundRect` call for the badge with explicit `ctx.arc` calls to draw two semicircular end caps connected by straight lines — a true pill shape that does not rely on `roundRect`.
**Lesson:** Do not use custom `roundRect` helpers for pill/capsule shapes. Use paired `ctx.arc` calls instead. They are unambiguous and work identically across all browsers.

---

### Known Issue — Dead Code in shareSummary
The following line in `shareSummary` is a no-op property access that does nothing and should be removed:
```javascript
shareData.text;
```
It is harmless but misleading.

---

### Mobile Share — navigator.canShare File Guard Required
When passing image files to the Web Share API on mobile, always check `navigator.canShare({ files: [fileToShare] })` before setting `shareData.files`. Not all mobile browsers support file sharing via the Share API even if they support `navigator.share`. Skipping this check causes a silent failure or uncaught error on unsupported browsers.


### Known Issue — clampOnBlur conflicts with maintenance-costs auto-fill
If maintenance-costs is included in the dollar clamp list, the blur handler's
synthetic input event dispatch triggers updateMortgageOutputs(), which overwrites
the user's manually entered value with 2% of purchase price.
Fix: maintenance-costs must not be in the clampOnBlur dollar inputs list.

## Known Remaining Differences from Workbook

- Renters insurance is now inflation-adjusted in the web version. The original workbook used a flat value. This is intentional — it's more realistic.
- Mortgage payment is perfectly constant in the web version (standard fixed-rate amortization). The original workbook had slight drift due to an implementation quirk. Web version is more accurate.
- Small floating-point differences (~$1-5) may exist due to JavaScript floating-point arithmetic vs. Numbers' internal precision. These are not logic errors.

---

## Edge Cases to Be Aware Of

### Zero interest rate
- PMT formula would divide by zero. Handled: `if (r === 0) return loan / n`
- annualInterest would also divide by zero. Handled in mortgageBalance: uses same guard

### Blank required fields
- Validation on Calculate button catches: purchase price, down payment, interest rate, timeline, monthly rent, rate of return
- All other blank fields default to 0 via `|| 0` pattern

### Monthly rent = 0
- PTR would divide by zero. Handled: `ptr = rent > 0 ? ... : null`
- Shows "—" in results if null

### Initial deposit manual = 0
- Must be explicitly respected as zero, not fall back to default
- Handled via: `isNaN(manualDepositInput) ? 0 : manualDepositInput`
- The NaN check distinguishes "user typed 0" from "field is blank"

### Negative default monthly contribution
- Occurs when rent > monthly ownership cost
- Displayed as $0 with a warning card explaining the situation
- Warning: "Your estimated rent is higher than the projected cost of owning..."

### PMI drop-off
- Checked each year using PRIOR year's balance and PRIOR year's home value
- If prevBal / prevVal >= 0.8, PMI applies. Below 0.8 (i.e. 20% equity), PMI = 0

### Timeline = 0 or blank
- Loop runs 0 times, all values = 0
- Caught by validation before reaching calculation

---

## Performance Notes

- The `mortgageBalance(y)` function loops through all months up to year y. Called once per year in the buy loop. For a 30-year timeline, this means up to 30 × 360 = 10,800 iterations. Acceptable for a calculator, but would need optimization if called more frequently.
- The `annualInterest(y)` function similarly loops through 12 months per call. Both are called inside the main buy loop, making the effective complexity O(years × months).
- No performance issues observed in practice with timelines up to 30 years.

---

## What NOT to Change Without Re-Testing

These areas are interdependent and easy to break:

1. The order of `buyCumCashflow` accumulation relative to `buyNetReturn` calculation
2. The `rentCumSpent` formula — do not add savingsValue subtraction back
3. The annuity due formula for savings — do not change to end-of-period
4. The `rentReserves = rent` assignment — do not use mortgage reserves here
5. The renting lifestyle formula — do not add `monthlyContrib×12` back
6. The renting 30% rule divisor — must be 0.4, not 0.3
7. The maintenance prefill logic — now lives inside `updateMortgageOutputs()`. Do not re-separate it into its own listener on `purchase-price`.
8. The `purchase-price` field has two input listeners (mortgage outputs and savings defaults). This is intentional. Do not consolidate them into one — they serve different update chains.
9. Do not add maintenance-costs to the clampOnBlur dollar inputs list — it will
   conflict with the auto-fill behavior in updateMortgageOutputs().
10. Do not change clearErrors() to select by ".warning-card" — it must select
   only ".dynamic-error" to avoid deleting static DOM elements.
11. The a[href^="#acc-"] click handler checks classList.contains('accordion-body') before
    calling openAccordionById(). Do not remove this check — it allows the results nav link
    to scroll without attempting to open a non-existent accordion.
12. Do not move the initialize block outside of `document.addEventListener('DOMContentLoaded', ...)` — share modal listeners and all other init functions will silently fail to attach.


---

## Mobile Implementation Notes

### Nav Observer — Threshold vs. rootMargin
The original Intersection Observer used `threshold: 0.3`, meaning 30% of a section had to be visible to trigger the active nav highlight. This works on desktop but fails on mobile when sections are taller than the viewport (30% of a tall section may never enter view at once). The fix: use `rootMargin: '-20% 0px -70% 0px'` with `threshold: 0`. This defines a trigger band in the upper-middle of the screen and fires as soon as any part of a section enters it, regardless of section height. The two previously separate observers (one for highlighting, one for amber/blue color) were also consolidated into one.

### Results Section Observer Detection
With an empty results section (before Calculate is pressed), the Intersection Observer couldn't detect the section reliably because it had almost no height. Fixed by giving `#results` a `min-height: 80vh`. The Results nav item now highlights correctly when the user scrolls to the bottom.

### iOS Input Auto-Zoom
iOS Safari auto-zooms into any input field with font-size below 16px. All input fields, prefixes, and suffixes were set to exactly `font-size: 16px` (not rem) to prevent this. The viewport meta tag also includes `maximum-scale=1.0, user-scalable=0`.

### Dual Results Table Breakpoints
The results table has overlapping responsive rules at both 768px and 600px. The 768px block (main mobile override) applies the tightest settings; the 600px block was added earlier and applies slightly looser values. Both are intentional — they produce slightly different behavior at tablet vs. phone widths — but if the table is ever restyled, both blocks need to be updated together.
