# PROJECT_CONTEXT.md
# Home Affordability Calculator — Project Context

---

## What This Project Is

A free, single-page home affordability calculator hosted at **caniaffordthishouse.app/Affordability/**
via GitHub Pages. It helps users understand the true financial cost of buying a home — including
all upfront costs, ongoing costs, and the income required to afford it comfortably.

This calculator is a companion tool to the main Rent vs. Buy Calculator at caniaffordthishouse.app.
Both tools live on the same domain and share the same design system, tone, and philosophy. The
affordability calculator focuses exclusively on the buying side — no renting comparison, no
investment return modeling. It answers a simpler, more common first question: "Can I actually
afford this home?"

**GitHub repo:** https://github.com/weaseljohnson/housing-affordability-calculator
**Live URL:** https://caniaffordthishouse.app/Affordability/
**Parent tool:** https://caniaffordthishouse.app

---

## Core Purpose

To give everyday people — not financial professionals — a fast, transparent, educational tool
for understanding what buying a specific home would actually cost them. The tool does not give
advice. It does the math and explains what the math means.

The product functions as both:

1. A practical affordability and mortgage calculator
2. A search-discoverable educational resource for home buying costs and mortgage basics

---

## Relationship to the Rent vs. Buy Calculator

The affordability calculator is a stripped-down version of the buy side of the Rent vs. Buy
Calculator. It shares:

- The same domain and visual design system
- The same color palette, typography, and component styles
- The same tone of voice and content philosophy
- The full buy-side calculation engine (mortgage, PMI, buydowns, extra principal, capital gains)
- The same share feature (image capture + encoded URL)
- The same export feature (ZIP with CSV files)

It does not include:
- Any renting inputs or rent-side calculation logic
- The rent vs. buy comparison table (results are single-column, buying only)
- Breakeven analysis (monthly, cumulative, net position)
- Rate of return, prior savings, initial deposit, or monthly contribution inputs
- Price-to-rent ratio

The affordability calculator has its own `index.html`, its own schema markup, its own SEO
metadata, and its own sitemap entry. It is not a redirect or alias — it is a standalone page
that shares a codebase origin with the parent tool.

---

## Project Philosophy

- **Educational first, calculator second.** Explanations are part of the product, not footnotes.
- **Transparency over simplicity.** All inputs are visible and adjustable, including defaults.
- **Warm and approachable tone.** Conversational, with humor. Reads like a knowledgeable friend.
- **Mobile-first.** Designed as a single scrolling page. Desktop is supported but mobile is priority.
- **Free forever.** No monetization, no ads, no paywalls, no backend, no user accounts.
- **Open source.** Community feedback via GitHub Discussions.

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
| Domain | Porkbun (.app TLD) | ~$18/yr renewal, shared with parent tool |
| Version control | GitHub (public repo) | Open source collaboration, Issues, Discussions |
| Dev environment | VS Code + Live Server | Local preview with hot reload |

---

## SEO Strategy

- URL path chosen for searchability: `/Affordability/`
- Page title targets high-intent affordability queries:
  "Home Affordability Calculator: Can you afford to buy?"
- Meta description targets mortgage costs, hidden costs, income needed
- Substantive written content on the page (not hidden behind modals) helps organic ranking
- Fast load time (no JS frameworks, no external dependencies) is a ranking signal
- Mobile responsiveness is a ranking signal
- robots.txt, sitemap.xml, and canonical tag in place
- JSON-LD schema: WebApplication, FAQPage (13 questions matching visible accordion content),
  BreadcrumbList, WebSite, Organization
- Semantic HTML: `<main>` wrapper, `aria-labelledby` on all sections, FAQ section labeled
- FAQ accordion content structured to match schema questions for Google rich result eligibility
- Do not change page title or meta description without careful keyword research

### On-Page SEO
- H1: "Can you afford to buy?"
- Hero eyebrow: "Free Housing Calculator"
- Intro paragraph includes affordability and cost-focused keywords naturally

### Technical SEO
- Canonical tag: `https://caniaffordthishouse.app/Affordability/`
- robots meta implemented
- sitemap.xml includes this page (update `<lastmod>` on each significant push)
- Fast static site via GitHub Pages
- Mobile-first responsive layout

### Structured Data
- WebApplication
- FAQPage (13 questions — see FAQ content section below)
- BreadcrumbList (2 levels: root → Affordability)
- WebSite
- Organization

---

## Current Build State

### Completed
- Full single-page layout with sticky nav progress bar
- Intro section with accordions (mindset content, FAQ, resources, feedback)
- Step 1: Property Info (mortgage inputs + live-updating output cards)
- Step 2: Ongoing Costs (housing costs + utilities)
- Step 3: Your Financial Picture (tax rate + lifestyle spending)
- Results section (single-column summary table + explanatory accordions)
- Full buy-side calculation engine (verified against Rent vs. Buy calculator outputs)
- Validation on Calculate button
- Color system (cream/navy/amber/sky palette — shared with parent tool)
- Section accent borders (alternating blue/amber, floating style)
- Nav bar color sync with section accents
- Mobile hamburger nav (floating circular button, dropdown menu, auto-close on link tap)
- Responsive results table
- Intersection Observer nav highlighting
- iOS input zoom prevention (16px font size, user-scalable=0 viewport)
- Results section minimum height fix for Observer detection when empty
- Advanced Settings accordion: inflation, appreciation, sale closing costs, filing status,
  capital gains toggle (rate + basis adjustment), mortgage buydown (2-1, 3-2-1, permanent),
  extra principal paydown with impact output card
- PMI manual override toggle
- Share feature: image capture (canvas, Safari-compatible) + encoded URL
- Export feature: ZIP download with inputs.csv and buying-results.csv
- JSON-LD schema: WebApplication, FAQPage (13 questions), BreadcrumbList, WebSite, Organization
- Open Graph and Twitter card meta tags
- Canonical, robots, author, theme-color, application-name tags
- Favicons and web manifest

### Not Yet Built
- Year-by-year data tables (collapsed by default)
- Any additional SEO content pages

---

## FAQ Content (13 Questions — Must Match FAQPage Schema Exactly)

The FAQ accordion in the intro section contains 13 questions. The FAQPage schema in `<head>`
must match these questions exactly — do not reorder, rename, or add questions without updating
both the accordion HTML and the schema.

1. How does this calculator work?
2. How much house can I afford?
3. How much do I need for a down payment?
4. What are the hidden costs of homeownership?
5. What is PMI and when do I have to pay it?
6. What is included in a monthly mortgage payment?
7. What are closing costs when buying a home?
8. Is a primary home a good investment?
9. How do mortgage buydowns work?
10. What is a debt-to-income ratio and why does it matter?
11. How does my credit score affect my mortgage rate?
12. What is the difference between a 2-1 and a 3-2-1 mortgage buydown?
13. What are mortgage discount points and how does a permanent buydown work?
14. Who pays for a mortgage buydown — the buyer or the seller?

Note: The schema contains 14 entries (questions 9–14 cover buydown sub-topics in detail).
The accordion groups these under a single "How do mortgage buydowns work?" trigger with
sub-items inside. The schema breaks them out individually for rich result eligibility.

---

## File Structure

Files at `/Affordability/` path in repo:
- `index.html` — all HTML, CSS, and JavaScript for the affordability calculator

Shared assets (repo root `/assets/` folder):
- `jszip.min.js` — JSZip v3.10.1, hosted locally to avoid external CDN dependencies
- Icon and image assets (favicons, OG preview image, etc.)

All HTML, CSS, and JavaScript logic lives in `index.html`. No build step, no bundler,
no separate files to manage. JSZip is the only third-party dependency and is kept in
`/assets/` to avoid bloating `index.html`.

---

## Open Source / Feedback Channel

- Primary: GitHub Discussions on the repo
- Secondary: Google Form (https://forms.gle/V9WyUTyqKpsAfzJr6) for non-GitHub users
- Update notifications: Google Form (https://forms.gle/J91tRCamevEvzRqc7)