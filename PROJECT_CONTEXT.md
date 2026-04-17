# PROJECT_CONTEXT.md
# Housing Affordability Calculator — Project Context

---

## What This Project Is

A free, single-page housing affordability calculator hosted at **caniaffordthishouse.app** via GitHub Pages. It helps users compare the true financial cost of buying a home vs. renting and saving/investing the difference. The tool originated as an Apple Numbers workbook and was rebuilt as a website to reach a broader audience.

**GitHub repo:** https://github.com/weaseljohnson/housing-affordability-calculator
**Live URL:** https://caniaffordthishouse.app

---

## Core Purpose

To give everyday people — not financial professionals — a transparent, educational tool for making the biggest financial decision of their lives. The tool does not give advice. It does the math and explains what the math means.

Two use cases:
1. Compare buying vs. renting side by side over a defined time horizon
2. Determine what a specific property (bought or rented) would actually cost

---

## Project Philosophy

- **Educational first, calculator second.** Explanations are part of the product, not footnotes. Most competing tools hide assumptions. This one surfaces them.
- **Transparency over simplicity.** All inputs are visible and adjustable, including defaults. Users should understand what every number means.
- **Warm and approachable tone.** Written in first person, conversational, with humor. Reads like a knowledgeable friend, not a financial institution.
- **Mobile-first.** Designed as a single scrolling page. Desktop is supported but mobile is the priority.
- **Free forever.** No monetization, no ads, no paywalls, no backend, no user accounts. Hosted on GitHub Pages with a custom domain (~$18/yr).
- **Open source.** Community feedback via GitHub Discussions, with a Google Form as a fallback for non-GitHub users.

---

## Hard Constraints — What This Project Is NOT

- No frameworks (no React, no Vue, no Angular)
- No backend, no server, no database
- No user accounts or saved sessions
- No monetization of any kind
- No financial advice tone — disclaim clearly, inform thoroughly
- No over-engineering — plain HTML/CSS/JS only
- No dark patterns, no email capture, no tracking

---

## Tech Stack

| Layer | Choice | Reason |
|---|---|---|
| Language | Vanilla HTML/CSS/JS | No build tools, no dependencies, fast, SEO-friendly |
| Hosting | GitHub Pages | Free forever, tied to open source workflow |
| Domain | Porkbun (.app TLD) | ~$18/yr renewal, transparent pricing |
| Version control | GitHub (public repo) | Open source collaboration, Issues, Discussions |
| Dev environment | VS Code + Live Server | Local preview with hot reload |

---

## SEO Strategy

- Domain chosen for searchability: `caniaffordthishouse.app`
- Page title: "Can I Afford This House? | Housing Affordability Calculator"
- Meta description targets "housing affordability calculator", "rent vs buy", "can I afford a home"
- Substantive written content on the page (not hidden behind modals) helps organic ranking
- Fast load time (no JS frameworks, no external dependencies) is a ranking signal
- Mobile responsiveness is a ranking signal

---

## Current Build State (as of first publish)

### Completed
- Full single-page layout with sticky nav progress bar
- Intro section with accordions (mindset content, resources, feedback)
- Step 1: Property Info (mortgage inputs + live-updating output cards)
- Step 2: Ongoing Costs (housing costs + utilities + advanced settings)
- Timeline section (single shared timeline input)
- Step 3: Renting & Saving (full renting inputs + savings parameters with toggle defaults)
- Step 4: Your Financial Picture (tax rates + lifestyle spending)
- Results section (summary table, PTR display, explanatory accordions)
- Full calculation engine (verified against original Numbers workbook)
- Validation on Calculate button
- Color system (cream/navy/amber/sky palette)
- Section accent borders (alternating blue/amber, floating style)
- Nav bar color sync with section accents
- GitHub Discussions link live
- Custom domain connected
- Mobile hamburger nav (floating circular button, dropdown menu, auto-close on link tap)
- Responsive results table (dual breakpoints: 768px and 600px)
- Intersection Observer nav highlighting rebuilt for mobile reliability (rootMargin-based, single unified observer)
- iOS input zoom prevention (16px input font size, user-scalable=0 viewport)
- Results section minimum height fix for Observer detection when empty

### Not Yet Built
- Year-by-year data tables (collapsed by default, one for buying, one for renting)
- README for the GitHub repo
- Any additional SEO content pages
- Any future features from community feedback

---

## File Structure

Single file: `index.html`
Everything — HTML, CSS, JavaScript — lives in this one file. This is intentional. No build step, no bundler, no separate files to manage.

Mobile-first design is fully implemented. The desktop nav (horizontal sticky tab bar) transforms on mobile (max-width: 768px) into a floating circular hamburger button in the top-right corner that reveals a vertical dropdown menu. The nav is completely non-blocking on mobile — the progress bar container itself becomes transparent and pointer-events: none, with only the button re-enabling pointer events.

---

## Open Source / Feedback Channel

- Primary: GitHub Discussions on the repo
- Secondary: Google Form (https://forms.gle/V9WyUTyqKpsAfzJr6) for non-GitHub users
- Update notifications: Google Form (https://forms.gle/J91tRCamevEvzRqc7)
