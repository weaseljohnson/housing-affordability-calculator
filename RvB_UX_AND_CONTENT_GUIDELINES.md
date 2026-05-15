# UX_AND_CONTENT_GUIDELINES.md
# Housing Affordability Calculator — UX & Content Guidelines

---

## Tone of Voice

### Core Character
Warm, conversational, honest, slightly self-deprecating. Reads like a knowledgeable friend who happens to be good with numbers — not a financial institution, not a corporate tool.

### Owner's Voice
The tool is written in first person by the creator. Key characteristics:
- Direct and unpretentious ("I'm no professional finance coach — just a regular person")
- Uses humor to soften what is otherwise a heavy, stressful topic
- Genuinely cares about helping the user make a good decision
- Does not hedge excessively or use legalese
- Allows personality to come through ("Numbers are this calculator's favorite crunchy snack")

### What to Preserve Verbatim
These lines capture the owner's voice and should not be rewritten without owner approval:
- "Hey there! If you're reading this, give yourself a pat on the back — you deserve it!"
- "Don't worry — I did my best to make this as user-friendly as possible, especially for those who consider themselves allergic to math."
- "These really are all the costs of owning a home. I know, it's a lot. 🫣"
- Any Hint box content (see below)

### What NOT to Sound Like
- Corporate financial tools ("Our proprietary algorithm calculates...")
- Overly cautious legal disclaimers
- Condescending ("Simply enter your information below")
- Clickbait or urgency-based copy ("Find out NOW if you can afford...")

---

## Writing Guidelines

### General
- Use "saving" not "investing" wherever possible — friendlier, less intimidating
- Use "saving/investing" only when the distinction genuinely matters (e.g., rate of return explanation)
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

## SEO Writing Guidelines

- Every headline should remain human-first but keyword-aware
- Use natural phrasing over forced keyword stuffing
- Prefer question-based headings where useful
- FAQ answers should target featured snippets
- Include synonyms:
  buying a house, buying a home, renting vs buying, affordability
- Maintain owner's warm tone while satisfying search intent

---

## Layout Decisions

### Single Page, Vertical Scroll
- No tabs, no multi-page navigation, no modal overlays
- Mobile-first — the majority of users are on mobile
- Desktop works and is well-supported, but mobile is the design priority

### Sticky Progress Nav Bar
- Always visible at top on desktop; replaced by floating hamburger button on mobile
- **Desktop:** Horizontal scrollable tab bar, white background, bottom-border active indicator
- **Mobile (max-width: 768px):** Nav bar container becomes transparent and non-blocking. A circular navy floating button (50×50px, drop shadow) appears in the top-right corner. Tapping it reveals a vertical dropdown card (white background, rounded corners, shadow, min-width 180px). Active state changes from bottom border to left border with background tint. Nav links get larger tap targets. Dropdown auto-closes when any link is tapped.
- Shows section names: Intro | Property | Costs | Renting | Finances | Results
- Active section highlighted with color (blue or amber matching section accent)
- Nav items are also jump links — users can jump back to any section
- Timeline section intentionally NOT in nav (it's a slim transitional section)

### Section Structure
- Each section has: section label (STEP 1 etc.) + h2 + section intro paragraph + content
- Alternating blue/amber left border accent on calculator sections
- Nav highlight color matches section accent color
- Intro section has no border accent (hero section)
- Timeline section has no border accent (slim transitional section)

### Color Assignment by Section
| Section | Accent | Nav |
|---|---|---|
| Intro | none | blue |
| Property Info (Step 1) | blue | blue |
| Ongoing Costs (Step 2) | amber | amber |
| Timeline | none | none (not in nav) |
| Renting & Saving (Step 3) | blue | blue |
| Your Financial Picture (Step 4) | amber | amber |
| Results | blue | blue |

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
- Live-updating mortgage outputs: 3-column grid (down payment, loan amount, monthly payment)
- Monthly payment card has blue highlight (most important output)
- Upfront cost outputs: 2-column grid with dark navy "Total" card spanning full width
- Output cards do NOT use color coding beyond the existing highlight — keep them clean

### Group Labels (calc-group-label)
- 13px, bold, uppercase, letter-spaced
- Color: var(--text-muted)
- Full-width bottom border in #c8bfb0
- Used to organize inputs within a section (e.g., "The Home", "Upfront Costs", "Utilities")

### How It Works Step Block (.how-it-works)
A lightweight 3-step visual block in the intro body, between the "Here's how it works:" line and the "Before you dive in" paragraph.
- Flex column layout with top/bottom border separating it from surrounding text
- Each item: numbered circle (sky blue, 22×22px) + step text
- Numbered circles use --sky-light background and --sky text
- Items separated by 0.5px --border dividers
- No card wrapper — sits open within the intro body text flow
- CSS classes: .how-it-works, .how-it-works-item, .how-it-works-num, .how-it-works-text


### Results Table — Responsive Breakpoints
The results table has two responsive breakpoints:
- **max-width: 768px** — `table-layout: fixed`, tighter padding (0.5rem/0.3rem), 0.8rem font, overflow hidden on wrapper, column widths locked 40%/30%/30%
- **max-width: 600px** — slightly relaxed padding (0.75rem/0.4rem), 0.85rem font, same fixed layout and column widths
Both breakpoints use `word-wrap: break-word` and keep the three-column buy/rent side-by-side layout intact rather than stacking.

---

## Interaction Rules

### Live Updates (Real-Time)
Only these fields update live:
- Down payment $ (derived from purchase price + down payment %)
- Loan amount (derived from purchase price + down payment %)
- Monthly mortgage payment (derived from loan amount + interest rate + term)
- Buying reserves (derived from monthly payment)
- Total upfront cash needed (derived from all upfront inputs)
- Default initial deposit display (derived from down payment)
- Default monthly contribution display (derived from owning costs vs rent)
- Negative contribution warning (appears/disappears live)
- Maintenance pre-fill (2% of purchase price — runs inside updateMortgageOutputs(), not a separate listener)

### Down Payment Dual-Mode Input (% / $ Toggle)

The down payment field (`down-payment-pct`) serves dual purpose — it stores either a percent or a dollar value depending on the current toggle mode, tracked via `dpModeToggle.dataset.mode` ('pct' or 'dollar').

**Key rules:**
- All reads of the down payment value must go through `getDpPct()`, not direct field reads. `getDpPct()` checks the current mode and converts dollar→percent on the fly if needed.
- Exception: `encodeStateToUrl()` and `decodeStateFromUrl()` intentionally read/write the raw field value. The current mode is encoded separately (key: `dpm`) and the raw field value is stored as-is (dollar or percent depending on mode). On decode, the toggle UI is restored before calculations run, and `lastDpDollarValue` is set from the restored field value if mode is dollar.
- The `dp` dollar amount in `updateMortgageOutputs()` reads the raw field directly when in dollar mode to avoid a lossy percent round-trip for display purposes. All other functions derive `dp` from `getDpPct()`.
- Dollar→percent conversion uses `toFixed(4)` to avoid floating point round-trip errors. Do not reduce this — `toFixed(2)` causes e.g. `$80,000 → 26.67% → $80,010` due to repeating decimal precision loss.
- `lastDpDollarValue` stores the exact dollar amount the user typed, so toggling back to dollar mode restores it exactly rather than reconverting from the lossy percent.

**What not to change:**
- Do not reduce `toFixed(4)` to fewer decimal places in the dollar→percent conversion.
- Do not read `down-payment-pct` value directly in calculation functions — always use `getDpPct()`.
- Do not add `down-payment-pct` mode encoding to the URL without also encoding the toggle mode (`dpm`) — a raw dollar value decoded as a percent would produce a wildly wrong down payment.
- `dpModeToggle` is scoped inside `wireUpListeners()` — any other function that needs to reference the toggle element (e.g. `decodeStateFromUrl()`) must look it up directly via `document.getElementById('dp-mode-toggle')`.

### Mobile-Specific Behavior
- Input fields use 16px font size explicitly to prevent iOS Safari auto-zoom on tap
- Viewport meta tag includes `user-scalable=0` to prevent pinch-zoom layout shifts
- The Intersection Observer uses `rootMargin: '-20% 0px -70% 0px'` rather than a threshold, so nav highlighting works correctly even when sections are taller than the viewport (a common mobile condition)
- Results section has `min-height: 80vh` to ensure the Observer can detect it even when empty (before Calculate is pressed)

### On Calculate Button
Everything else — all results table values, PTR, green highlighting.

### Validation
- Calculate button validates required fields before running calculations
- Missing fields shown in a warm amber warning card listing field names
- All other fields safely default to 0 if blank

### Toggle Switches (Default vs Manual)
- Used for Initial Deposit and Monthly Contribution
- Default ON: shows calculated default in a blue display card with explanation
- Default OFF: shows editable input field with note explaining $0 behavior
- Switching back to default restores the calculated value display

### PMI Card (Property Info section)
- Appears automatically when auto-calculated PMI > 0 (down payment under 20%)
- Hidden in auto mode when down payment ≥ 20% — no card shown for a $0 value
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

### Buydown Payment Schedule Card
- Lives inside the Advanced Settings accordion, visible only when a temporary buydown type (2-1 or 3-2-1) is selected and the buydown toggle is on
- Shows Year 1, Year 2 (and Year 3 for 3-2-1), full rate payment, and upfront cost (only when buyer is paying)
- Updates live alongside the mortgage output cards — no Calculate required
- The upfront cost cell uses the `.output-card.total` dark navy style to visually signal it as a total/summary figure

### Results Table Relabeling for Buydowns
- When a temporary buydown is active, the "Average Monthly Cost" and "Total Yearly Cost" row labels are dynamically updated to "After Buydown (Yr N+)" on Calculate
- This signals that the displayed costs reflect what the user will pay after the buydown period ends, not the subsidized early years
- When buydown is off or a permanent buydown is active, labels restore to "Year 1"
- The relabeling targets the text node inside `.tooltip-wrap` spans — do not restructure those cells without updating the targeting logic in `runCalculations()`

### Results Nav Link (.results-nav-link)
A small amber-colored navigation link that appears above the results table, inside the results-content div.
- Text: "→ How to read your results"
- Color: var(--amber), hover: var(--amber-light) with underline
- Font size: 0.88rem, font-weight: 600
- Links to #acc-results-explainer (the accordion group wrapper div)
- Scrolls to the accordion section without opening any specific accordion
- Intentionally amber rather than sky blue — signals "worth your attention" vs. a standard footnote

### Table Row Tooltips
- Results table rows 1–3 (Total Upfront Cost, Average Monthly Cost, Total Yearly Cost) use tooltip icons for brief one-sentence explanations
- Tooltips in the table use the .tooltip-box.tooltip-down modifier — opens downward rather than rightward to avoid mobile overflow in the constrained label column
- Results table rows 4–6 (Cumulative Money Spent, Savings Return, Return Minus Cumulative Spent) use footnote accordion links instead of tooltips — content is too substantial for a tooltip
- The IDs on rows 4–6 are on inner <span> elements, not the <td> itself, so JS setStr() calls update only the text node while leaving tooltip/footnote markup intact

### Share Feature

#### Share Modal Structure
- Opens via the "Share Results" button, which appears inline next to the Calculate button after results are calculated
- Modal contains three tabs: **Buying vs. Renting** (default), **Buying Only**, **Renting Only**
- Switching tabs updates the share card preview live with no loading or page navigation
- Modal closes via the ✕ button, clicking the backdrop, or pressing Escape

#### Share Card Content
The share card preview inside the modal displays:
- Navy header bar with site eyebrow label and a mode-specific title
- Abbreviated results table with four rows: Total Upfront Cost, Avg Monthly Cost — Yr 1, Financial Return, Cumulative Spending
- Recommended Gross Income row — included only if tax rates were entered. Shows a range (lower – upper) if lifestyle spending is also entered and the lifestyle result is lower than the 30% rule result. Shows just the higher number if the lifestyle result exceeds the 30% rule. Shows only the 30% rule result if no lifestyle spending was entered.
- Footer with a brief callout to the site URL
- Breakeven Analysis section — shown below the main table in all share modes. Displays all three breakeven values in a compact labeled layout. Null values display as "—". Included in both the modal preview and the canvas image.

#### Green Highlighting in Share Card
- Cost rows (lower is better): green highlights the lower of the two values
- Return rows (higher is better): green highlights the higher value
- Income range rows: green highlights the column whose upper bound is lower, since a lower recommended income means more affordable
- Green highlighting only applies in Buying vs. Renting mode. Single-column modes show no green.

#### Share Button Behavior by Platform

| | Mobile | Desktop |
|---|---|---|
| **Share Summary** | Opens native share sheet with PNG image file attached | Downloads card as PNG file |
| **Share Full Results** | Opens native share sheet with encoded URL | Copies encoded URL to clipboard with toast confirmation |

#### Encoded Share URL
- The Share Full Results URL encodes all input state into compact query parameters using a short-key dictionary (e.g. `p` for purchase price, `dp` for down payment)
- When a recipient opens the link, all inputs are pre-filled and Calculate runs automatically, scrolling to the results section
- The URL includes a `#results` hash to anchor to the results section
- URL is generated by `encodeStateToUrl()` and decoded by `decodeStateFromUrl()` on page load
- Default field values (inflation, appreciation, sale closing costs, rent increase) are included in the encoded URL so the recipient sees an identical calculation

---

## Color Palette

```
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
```

Section dividers between major sections: `#c8bfb0` (slightly darker than --border)

---

## Hint Boxes

Hint boxes are a signature element of this tool. They add personality and break up what could otherwise feel like a lecture.

**Style:** Sky blue background (--sky-light), left border (--sky), small text
**Format:** 💡 **Hint:** [content]
**Placement:** Inline within accordion content, after the relevant paragraph
**Tone:** Playful, self-referential humor about the calculator itself

**Existing Hint boxes (do not rewrite without owner approval):**
- "This calculator loves a good run, especially where numbers are involved."
- "Numbers are this calculator's favorite crunchy snack."
- "This calculator is very determined to help you determine how much it might cost you to own that home. 😄"

---

## Content That Is Educational (Not Just Functional)

These sections exist to teach, not just collect input. Treat them with full content depth:

### Intro Accordions
Three distinct accordion groups in the intro section:

**Group 1 — Things to Keep in Mind** (no label above it, first and self-explanatory)
Core mindset content — slimmed to four items only:
- The disclaimer
- Buying vs. renting is your call to make
- Renting doesn't mean you're throwing money away
- This analysis is purely financial
Do not condense further or add new items without owner approval. This is why the tool exists.

**Group 2 — Common Questions** (section-label: "Common Questions")
11 FAQ accordion items, one question per accordion item, in this order:
1. How does this calculator compare buying vs. renting?
2. How much house can I afford?
3. How much do I need for a down payment?
4. What are the hidden costs of homeownership?
5. What is PMI and when do I have to pay it?
6. What is included in a monthly mortgage payment?
7. What are closing costs when buying a home?
8. When is renting better than buying?
9. Should I save the difference if renting is cheaper?
10. Is a primary home a good investment?
11. What is a price-to-rent ratio, and what does it mean?
These questions match the FAQPage schema exactly. Do not reorder or rename without updating the schema to match.

**Group 3 — Resources & Feedback** (section-label: "Resources & Feedback")
- Additional Resources: external links organized by topic. All URLs are live.
- Feedback & Suggestions: GitHub Discussions primary, Google Form secondary.


### Help Notes on Inputs
- Property tax: explanation of gross vs. base rate, Zillow tip
- Maintenance: 1-2% rule, nudge to not underestimate
- Savings contribution: explanation of default calculation logic
- Rate of return: HYSA (3-4%) vs index funds (7%) guidance
- Rent increase YoY: historical context (4% avg 2015-2025)
- Effective income tax rate: gross vs net distinction, bracket warning

### Results Section Accordions
One accordion group with section-label "Understanding Your Results" above it.
Seven accordion items in this order:

1. **Where do I go from here?** — High-level guide to interpreting results. Covers Case 1 (buying favored), Case 2 (renting favored), investment return caveat, and the "renting now doesn't mean buying never" point.
2. **Cumulative Money Spent** — Explains what's included in each column, why principal counts, why gain on sale is excluded for buying, and why savings contributions count as spent for renting.
3. **Savings Return** — Explains what the return represents for each path (home equity net of sale costs vs. compounded savings value).
4. **Return Minus Cumulative Spent** — Explains the bottom-line calculation and the green highlighting logic. Notes this is a financial comparison only.
5. **Breakeven Analysis** — Explains all three breakeven metrics: Monthly Cost (when buying's monthly outflow drops below renting's), Cumulative Spending (when total money spent buying falls below total money spent renting), and Net Position (when buying's financial return minus cumulative spending exceeds renting's equivalent figure). Notes that "No breakeven found" means renting comes out ahead for the entire horizon modeled.6. **The 30% Rule** — Why net income is used instead of gross, lifestyle inflation concept, renting column uses 40% (30% housing + 10% savings).
6. **The Current Lifestyle Approach** — Lifestyle inflation warning, this is a MINIMUM estimate, savings contribution accounting explanation.

Anchor links from results table rows open the target accordion and scroll to it.
The nav link "→ How to read your results" above the table scrolls to the accordion group wrapper without opening any specific item.

---

### FAQ Strategy

FAQ content is not filler. It serves:

- User education
- Objection handling
- Long-tail search capture
- Structured data eligibility
- Featured snippet opportunities

---

## What Explicitly Does NOT Belong

- No color coding on output cards beyond existing highlight (no "red = bad, green = good" on inputs)
- No auto-advancing between sections
- No progress percentage or completion indicators
- No "recommended" presets or example scenarios
- No email capture or newsletter signup
- No social sharing buttons (at least in current phase)
- No advertising of any kind
- No "financial advisor" referral CTAs
- No assumptions about what the user "should" do — present data, let them decide
