# UX_AND_CONTENT_GUIDELINES.md
# Home Affordability Calculator — UX & Content Guidelines

---

## Tone of Voice

### Core Character
Warm, conversational, honest, slightly self-deprecating. Reads like a knowledgeable friend who
happens to be good with numbers — not a financial institution, not a corporate tool.

### Owner's Voice
The tool is written in first person by the creator. Key characteristics:
- Direct and unpretentious ("I'm no professional finance coach — just a regular person")
- Uses humor to soften what is otherwise a heavy, stressful topic
- Genuinely cares about helping the user make a good decision
- Does not hedge excessively or use legalese
- Allows personality to come through

### What to Preserve Verbatim
These lines capture the owner's voice and should not be rewritten without owner approval:
- "Hey there! If you're reading this, give yourself a pat on the back — you deserve it!"
- "Don't worry — I did my best to make this as user-friendly as possible, especially for
  those who consider themselves allergic to math."
- Any Hint box content (see below)

### What NOT to Sound Like
- Corporate financial tools ("Our proprietary algorithm calculates...")
- Overly cautious legal disclaimers
- Condescending ("Simply enter your information below")
- Clickbait or urgency-based copy ("Find out NOW if you can afford...")

---

## Writing Guidelines

### General
- Avoid jargon without explanation. When jargon is necessary, explain it inline.
- Contractions are fine and preferred (it's, you'll, you've, that's)
- Oxford comma is used consistently
- Em dashes (—) used for emphasis and asides, not hyphens

### Educational Content Philosophy
- Explanations are part of the product, not optional extras
- The goal is to teach, not just calculate
- All assumptions should be surfaced and explained
- Users should understand what every input does and why it matters
- Educational depth is opt-in (accordions) — the main flow should not be blocked by long text

### Humor
- Hint boxes are the primary vehicle for humor
- Humor should be gentle and self-aware, never at the user's expense
- Emoji use is intentional and sparse — 🤓, 🫣, 😄 are established in the codebase

### SEO Writing Guidelines
- Every headline should remain human-first but keyword-aware
- Use natural phrasing over forced keyword stuffing
- Prefer question-based headings where useful
- FAQ answers should target featured snippets
- Include synonyms: buying a house, buying a home, home affordability, mortgage costs
- Maintain owner's warm tone while satisfying search intent

---

## Layout Decisions

### Single Page, Vertical Scroll
- No tabs, no multi-page navigation, no modal overlays for content
- Mobile-first — the majority of users are on mobile
- Desktop works and is well-supported, but mobile is the design priority

### Sticky Progress Nav Bar
- Always visible at top on desktop; replaced by floating hamburger button on mobile
- **Desktop:** Horizontal scrollable tab bar, white background, bottom-border active indicator
- **Mobile (max-width: 768px):** Nav bar container becomes transparent and non-blocking.
  A circular navy floating button (50×50px, drop shadow) appears in the top-right corner.
  Tapping it reveals a vertical dropdown card (white background, rounded corners, shadow,
  min-width 180px). Active state changes from bottom border to left border with background
  tint. Nav links get larger tap targets. Dropdown auto-closes when any link is tapped.
- Shows section names: Intro | Property | Costs | Finances | Results
- Active section highlighted with color (blue or amber matching section accent)
- Nav items are also jump links — users can jump back to any section
- Timeline is not a separate nav item — Step 3 (Your Financial Picture) follows directly
  after Step 2 (Ongoing Costs) with no intermediate nav stop

### Section Structure
- Each section has: section label (STEP 1 etc.) + h2 + section intro paragraph + content
- Alternating blue/amber left border accent on calculator sections
- Nav highlight color matches section accent color
- Intro section has no border accent (hero section)

### Color Assignment by Section

| Section | Accent | Nav |
|---|---|---|
| Intro | none | blue |
| Property Info (Step 1) | blue | blue |
| Ongoing Costs (Step 2) | amber | amber |
| Your Financial Picture (Step 3) | blue | blue |
| Results | amber | amber |

### Section Accent Border Style
- Thin (2px) rounded line
- Floats away from section edges (inset top and bottom by 2.5rem)
- Positioned at left: 0, outside the content padding
- Feels like a typographic accent, not a structural border

### Input Groups
- Each input has: label + input field + optional help note + optional help links
- Help links use arrow prefix (→) and sky blue color
- Input fields use prefix/suffix pattern for $ and % symbols
- Focus state changes border to sky blue
- 44px height on all inputs (mobile touch target)

### Output Cards
- Live-updating mortgage outputs: 2-column grid (down payment, loan amount)
- Monthly payment card spans full width with blue highlight (most important output)
- Upfront cost outputs: reserves card + dark navy "Total Upfront Cash Needed" card
- Output cards do NOT use color coding beyond the existing highlight — keep them clean

### Group Labels (calc-group-label)
- 13px, bold, uppercase, letter-spaced
- Color: var(--text-muted)
- Full-width bottom border in #c8bfb0
- Used to organize inputs within a section (e.g., "The Home", "Upfront Costs", "Utilities")

### How It Works Step Block (.how-it-works)
A lightweight 2-step visual block in the intro body, between the "Here's how it works:"
line and the "Before you dive in" paragraph.
- Flex column layout with top/bottom border separating it from surrounding text
- Each item: numbered circle (sky blue, 22×22px) + step text
- Items separated by 0.5px --border dividers
- No card wrapper — sits open within the intro body text flow
- CSS classes: .how-it-works, .how-it-works-item, .how-it-works-num, .how-it-works-text
- Step 1: "Enter your property details and ongoing costs"
- Step 2: "Hit Calculate to see a side-by-side comparison"

Note: The step 2 label says "side-by-side comparison" but the results table is
single-column (buying only). This should be updated to something like "Hit Calculate
to see your full cost breakdown" if the how-it-works block is ever revised.

### Results Table
- Single-column layout: row label | Buying
- Navy header row
- Alternating cream/warm-white row backgrounds
- Rows 1–3 use tooltip icons for brief inline explanations (opens downward on mobile)
- Rows 4–6 use footnote accordion links instead of tooltips (content too long for tooltip)
- The IDs on rows 4–6 are on inner `<span>` elements, not the `<td>` itself, so JS
  `setStr()` calls update only the text node while leaving footnote markup intact

### Results Table — Responsive Behavior
- `max-width: 768px`: `table-layout: fixed`, tighter padding, 0.8rem font,
  column widths 60% label / 40% value
- Both breakpoints use `word-wrap: break-word`

---

## Interaction Rules

### Live Updates (Real-Time)
Only these fields update live:
- Down payment $ (derived from purchase price + down payment %)
- Loan amount (derived from purchase price + down payment %)
- Monthly mortgage payment (derived from loan amount + interest rate + term)
- Buying reserves (derived from monthly payment + insurance + tax)
- Total upfront cash needed (derived from all upfront inputs)
- Maintenance pre-fill (2% of purchase price — runs inside updateMortgageOutputs())
- PMI card visibility and auto value

### Down Payment Dual-Mode Input (% / $ Toggle)

The down payment field (`down-payment-pct`) serves dual purpose — it stores either a percent
or a dollar value depending on the current toggle mode, tracked via
`dpModeToggle.dataset.mode` ('pct' or 'dollar').

**Key rules:**
- All reads of the down payment value must go through `getDpPct()`, not direct field reads.
  `getDpPct()` checks the current mode and converts dollar→percent on the fly if needed.
- Exception: `encodeStateToUrl()` and `decodeStateFromUrl()` intentionally read/write the
  raw field value. The current mode is encoded separately (key: `dpm`) and the raw field
  value is stored as-is. On decode, the toggle UI is restored before calculations run.
- Dollar→percent conversion uses `toFixed(4)` to avoid floating point round-trip errors.
  Do not reduce this — `toFixed(2)` causes e.g. `$80,000 → 26.67% → $80,010`.
- `dpModeToggle` is scoped inside `wireUpListeners()` — any other function that needs to
  reference the toggle element must look it up directly via
  `document.getElementById('dp-mode-toggle')`.

**What not to change:**
- Do not reduce `toFixed(4)` to fewer decimal places in the dollar→percent conversion.
- Do not read `down-payment-pct` value directly in calculation functions — always use
  `getDpPct()`.
- Do not encode the raw dollar value in the URL without also encoding the toggle mode
  (`dpm`) — a raw dollar value decoded as a percent would produce a wildly wrong result.

### On Calculate Button
All results table values and highlighting. Validation runs first — missing required fields
shown in an amber warning card listing field names.

### Validation
- Required fields: purchase price, down payment, interest rate, timeline
- Missing fields shown in a warm amber warning card
- All other fields safely default to 0 if blank

### PMI Card (Property Info section)
- Appears automatically when auto-calculated PMI > 0 (down payment under 20%)
- Hidden in auto mode when down payment ≥ 20%
- A Manual toggle in the card label row switches between auto display and an editable input
- In manual mode, the card is always visible regardless of down payment percentage
- Use case: VA loans, other PMI-exempt financing, or custom PMI amounts
- The equity auto-drop at 22% applies in both modes — manual mode does not disable it
- Input is clamped to 0 minimum on blur

### Accordions
- Used throughout for educational content and advanced settings
- Closed by default everywhere
- Triggered by clicking header button
- Chevron rotates 180° when open
- Anchor links from results table open the target accordion and scroll to it

### Advanced Settings
- Single accordion in Step 1 containing all advanced inputs
- Inflation rate, home appreciation, sale closing costs
- Filing status radio (Single / Married Filing Jointly)
- Capital gains toggle → reveals rate selector + basis adjustment field
- Mortgage buydown toggle → reveals type (2-1 / 3-2-1 / Permanent) + payer + inputs
- Extra principal toggle → reveals amount input + impact output cards
- Auto-opens on page load if a shared URL encoded any advanced toggle as active

### Buydown Payment Schedule Card
- Lives inside the Advanced Settings accordion
- Visible only when a temporary buydown type (2-1 or 3-2-1) is selected and active
- Shows Year 1, Year 2 (and Year 3 for 3-2-1), full rate payment
- Shows upfront cost only when buyer is paying — uses `.output-card.total` dark navy style
- Updates live alongside the mortgage output cards — no Calculate required

### Results Table Relabeling for Buydowns
- When a temporary buydown is active, the "Average Monthly Cost" and "Total Yearly Cost"
  row labels are dynamically updated to "After Buydown (Yr N+)" on Calculate
- When buydown is off or permanent, labels restore to "Year 1"
- The relabeling targets the text node inside `.tooltip-wrap` spans — do not restructure
  those cells without updating the targeting logic in `runCalculations()`

### Results Nav Link (.results-nav-link)
A small amber-colored navigation link that appears above the results table.
- Text: "→ How to read your results"
- Color: var(--amber), hover: var(--amber-light) with underline
- Font size: 0.88rem, font-weight: 600
- Links to #acc-results-explainer (the accordion group wrapper div)
- Scrolls to the accordion section without opening any specific accordion

### Share Feature

#### Share Modal Structure
- Opens via the "Share Results" button, which appears inline next to the Calculate and
  Export buttons after results are calculated
- Single-column only — no tabs (no Buying vs. Renting / Buying Only / Renting Only modes)
- Modal closes via the ✕ button, clicking the backdrop, or pressing Escape

#### Share Card Content
The share card preview inside the modal displays:
- Navy header bar with site eyebrow label and title "My Home Affordability Results"
- Abbreviated results table with four rows:
  Total Upfront Cost, Avg Monthly Cost — Yr 1, Net Proceeds on Sale, Cumulative Spending
- Recommended Gross Income row — included only if tax rate was entered. Shows a range
  (lower – upper) if lifestyle spending is also entered and the lifestyle result is lower
  than the 30% rule result. Shows just the higher number if the lifestyle result exceeds
  the 30% rule. Shows only the 30% rule result if no lifestyle spending was entered.
- Footer with a brief callout to the site URL

#### Share Button Behavior by Platform

| | Mobile | Desktop |
|---|---|---|
| **Share Summary** | Opens native share sheet with PNG image attached | Downloads card as PNG file |
| **Share Full Results** | Opens native share sheet with encoded URL | Copies encoded URL to clipboard with toast |

#### Encoded Share URL
- Encodes all input state into compact query parameters using a short-key dictionary
- When a recipient opens the link, all inputs are pre-filled and Calculate runs automatically,
  scrolling to the results section
- URL includes a `#results` hash to anchor to the results section
- Generated by `encodeStateToUrl()` and decoded by `decodeStateFromUrl()` on page load

### Export Feature
- ZIP download containing two CSV files: `inputs.csv` and `buying-results.csv`
- `inputs.csv` — all input field values at time of export, organized by section
- `buying-results.csv` — year-by-year data from the buy loop (home value, mortgage balance,
  equity, interest, principal, taxes, insurance, HOA, maintenance, PMI, utilities,
  buydown subsidy, extra principal, total year cost, cumulative spent, net position)
- Triggered by the Export Data button (shown after Calculate is pressed)
- Uses JSZip v3.10.1, hosted locally at `/assets/jszip.min.js`

### Mobile-Specific Behavior
- Input fields use 16px font size explicitly to prevent iOS Safari auto-zoom on tap
- Viewport meta tag includes `maximum-scale=1.0, user-scalable=0`
- The Intersection Observer uses `rootMargin: '-20% 0px -70% 0px'` rather than a threshold
- Results section has `min-height: 80vh` to ensure the Observer can detect it when empty

---

## Color Palette

--cream:       #fdf8f2   (page background)
--warm-white:  #fffcf8   (card/input backgrounds)
--navy:        #1e2d40   (primary text, headings, dark cards)
--navy-light:  #2e4260   (secondary navy)
--amber:       #c47b2b   (accent color — buttons, amber sections)
--amber-light: #f5a940   (amber hover state)
--amber-bg:    #fef3e2   (amber tint backgrounds)
--sky:         #2a7ab8   (primary accent — links, blue sections, focus)
--sky-light:   #e6f2fa   (sky tint backgrounds, hint boxes)
--text-main:   #1e2d40   (body text)
--text-muted:  #5a6a7a   (secondary text, section intros)
--text-light:  #8a9aaa   (helper text, notes, placeholders)
--border:      #e8e0d5   (input borders, dividers)

Section dividers between major sections: `#c8bfb0` (slightly darker than --border)

---

## Hint Boxes

Hint boxes are a signature element of this tool. They add personality and break up what
could otherwise feel like a lecture.

**Style:** Sky blue background (--sky-light), left border (--sky), small text
**Format:** 💡 **Hint:** [content]
**Placement:** Inline within accordion content, after the relevant paragraph
**Tone:** Playful, self-referential humor about the calculator itself

**Existing Hint boxes (do not rewrite without owner approval):**
- "This calculator loves a good run, especially where numbers are involved."
- "Numbers are this calculator's favorite crunchy snack."

---

## Content That Is Educational (Not Just Functional)

### Intro Accordions
Three distinct accordion groups in the intro section:

**Group 1 — Things to Keep in Mind** (no label above it)
Core mindset content — three items:
- The disclaimer
- Buying a home is your call to make
- This analysis is purely financial
Do not condense further or add new items without owner approval.

**Group 2 — Common Questions** (section-label: "Common Questions")
11 FAQ accordion items matching the schema, in this order:
1. How does this calculator work?
2. How much house can I afford?
3. How much do I need for a down payment?
4. What are the hidden costs of homeownership?
5. What is PMI and when do I have to pay it?
6. What is included in a monthly mortgage payment?
7. What are closing costs when buying a home?
8. Is a primary home a good investment?
9. How do mortgage buydowns work? (contains sub-items covering 2-1 vs 3-2-1,
   discount points, and who pays)
10. What is a debt-to-income ratio and why does it matter?
11. How does my credit score affect my mortgage rate?

These questions align with the FAQPage schema. Do not reorder or rename without
updating the schema to match.

**Group 3 — Resources & Feedback** (section-label: "Resources & Feedback")
- Additional Resources: external links organized by topic. All URLs are live.
- Feedback & Suggestions: GitHub Discussions primary, Google Form secondary.

### Help Notes on Inputs
- Property tax: explanation of effective vs. base rate, with a toggle-on-click help box
  showing the calculation method (tax history ÷ purchase price × 100)
- Maintenance: 1-2% rule, nudge to not underestimate, auto-prefilled at 2%
- Lifestyle spending: collapsible "How do I calculate this?" accordion with step-by-step
  instructions

### Results Section Accordions
One accordion group with section-label "Understanding Your Results" above it.
Six accordion items in this order:

1. **Where do I go from here?** — High-level guide to interpreting results. Home appreciation
   caveat, flexibility of savings vs. real estate, renting now doesn't mean buying never.
2. **Cumulative Spending** — What's included (all upfront + all ongoing costs over N years),
   why principal counts, why gain on sale is excluded.
3. **Financial Return** — What the Net Proceeds on Sale figure represents: home value minus
   remaining mortgage balance minus sale closing costs.
4. **Net Proceeds vs Cumulative Spending** — The bottom-line calculation. Positive = came out
   ahead financially. Notes this is financial only, not a life decision.
5. **The 30% Rule** — Why net income is used instead of gross. Lifestyle inflation concept.
6. **The Current Lifestyle Approach** — Lifestyle inflation warning, this is a MINIMUM
   estimate, how the calculation works.

Anchor links from results table rows open the target accordion and scroll to it.
The nav link "→ How to read your results" above the table scrolls to the accordion
group wrapper without opening any specific item.

---

## What Explicitly Does NOT Belong

- No renting inputs, renting math, or rent vs. buy comparison of any kind
- No breakeven analysis
- No rate of return, prior savings, initial deposit, or savings contribution inputs
- No color coding on output cards beyond existing highlight
- No auto-advancing between sections
- No progress percentage or completion indicators
- No "recommended" presets or example scenarios
- No email capture or newsletter signup
- No advertising of any kind
- No "financial advisor" referral CTAs
- No assumptions about what the user "should" do — present data, let them decide


