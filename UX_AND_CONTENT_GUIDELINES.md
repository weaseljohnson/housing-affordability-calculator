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
- Maintenance pre-fill (2% of purchase price, fills on purchase price change)

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

### Accordions
- Used throughout for educational content and advanced settings
- Closed by default everywhere
- Triggered by clicking header button
- Chevron rotates 180° when open
- Anchor links from results table open the target accordion and scroll to it

### Advanced Settings Toggle
- Visually distinct from content accordions (muted label color, different styling)
- Used for: inflation rate, home appreciation, sale closing costs
- Also used for: "How do I calculate my lifestyle spending?" in Financial Picture section
- Collapsed by default — power users only

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
- **Things to Keep in Mind — The Big Picture:** Core mindset content. Do not condense. This is why the tool exists.
- **Additional Resources:** External links organized by topic. All URLs are live.
- **Feedback & Suggestions:** GitHub Discussions primary, Google Form secondary.

### Help Notes on Inputs
- Property tax: explanation of gross vs. base rate, Zillow tip
- Maintenance: 1-2% rule, nudge to not underestimate
- Savings contribution: explanation of default calculation logic
- Rate of return: HYSA (3-4%) vs index funds (7%) guidance
- Rent increase YoY: historical context (4% avg 2015-2025)
- Effective income tax rate: gross vs net distinction, bracket warning

### Results Section Accordions
- **The 30% Rule:** Why net income is used instead of gross, lifestyle inflation concept, renting column uses 40% (30% housing + 10% savings)
- **The Current Lifestyle Approach:** Lifestyle inflation warning, this is a MINIMUM estimate, savings contribution accounting explanation

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
