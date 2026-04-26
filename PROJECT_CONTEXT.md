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

The product functions as both:

1. A powerful rent vs buy calculator
2. A search-discoverable educational resource for housing affordability

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
- Page title: "Rent vs Buy Calculator: Can You Afford to Buy — Is Renting Smarter?" — chosen to capture high search-intent queries (rent vs buy, can you afford to buy, is renting smarter) while maintaining human appeal
- Meta description targets "buying vs. renting", "mortgage", "taxes", "insurance", "can I afford a home", and "income needed"
- Substantive written content on the page (not hidden behind modals) helps organic ranking
- Fast load time (no JS frameworks, no external dependencies) is a ranking signal
- Mobile responsiveness is a ranking signal
- robots.txt, sitemap.xml, and canonical tag in place at repo root
- JSON-LD schema: WebApplication, FAQPage (11 questions matching visible accordion content), BreadcrumbList, WebSite, Organization
- Semantic HTML landmarks: <main> wrapper, aria-labelledby on all sections, FAQ section wrapper
- FAQ accordion content structured to match schema questions for Google rich result eligibility
- Do not change page title or meta description without careful keyword research — both were deliberately crafted


- ### On-Page SEO
- Search-focused H1:
  "Can you afford to buy — or is renting smarter?"
- Optimized title tag targeting:
  rent vs buy calculator, can afford home, renting smarter
  "Rent vs Buy Calculator: Can You Afford to Buy — Is Renting Smarter?"
- Enhanced meta description targeting affordability + ownership costs
- Intro paragraph includes high-intent keywords naturally

### Technical SEO
- Canonical tag implemented
- robots meta implemented
- robots.txt deployed
- sitemap.xml deployed
- Fast static site performance via GitHub Pages
- Mobile-first responsive layout

### Structured Data
- WebApplication
- FAQPage
- BreadcrumbList
- WebSite
- Organization

### Content SEO
- Search-intent FAQ library
- Educational explanations throughout page
- "What’s included" ownership-cost content

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
- robots.txt and sitemap.xml added to repo root (update <lastmod> in sitemap on each significant push)
- Canonical tag, meta robots, meta author, theme-color, application-name added to <head>
- JSON-LD schema: WebApplication, FAQPage (11 questions), BreadcrumbList, WebSite, Organization
- Semantic HTML: <main> wrapper, aria-labelledby on all sections, FAQ section labeled
- Hero eyebrow upgraded to keyword-rich: "Free Home Affordability & Rent vs. Buy Calculator"
- Tagline rewritten for SEO keyword density, font size reduced to 0.92rem and color lightened to --text-light to de-emphasize visually
- "How it works" step block added to intro body (between "Here's how it works:" and "Before you dive in")
- Intro accordions restructured into three groups: Things to Keep in Mind (slimmed to 4 items), Common Questions (11 FAQ items), Resources & Feedback (separated)
- Results table: tooltips on rows 1–3, footnote accordion links on rows 4–6
- Results nav link "→ How to read your results" added above results table in amber
- Results accordions expanded to 7 items under "Understanding Your Results" label: Where do I go from here?, Cumulative Money Spent, Savings Return, Return Minus Cumulative Spent, PTR explainer, 30% Rule, Current Lifestyle Approach
- README for the GitHub repo

### Not Yet Built
- Year-by-year data tables (collapsed by default, one for buying, one for renting)
- Any additional SEO content pages
- Any future features from community feedback

---

## File Structure

Three files at repo root:
- `index.html` — all HTML, CSS, and JavaScript
- `robots.txt` — crawler access rules and sitemap pointer
- `sitemap.xml` — page manifest for Google indexing. Update <lastmod> date on every significant push.

Everything — HTML, CSS, JavaScript — lives in index.html. This is intentional. No build step, no bundler, no separate files to manage.


Mobile-first design is fully implemented. The desktop nav (horizontal sticky tab bar) transforms on mobile (max-width: 768px) into a floating circular hamburger button in the top-right corner that reveals a vertical dropdown menu. The nav is completely non-blocking on mobile — the progress bar container itself becomes transparent and pointer-events: none, with only the button re-enabling pointer events.

---

## Open Source / Feedback Channel

- Primary: GitHub Discussions on the repo
- Secondary: Google Form (https://forms.gle/V9WyUTyqKpsAfzJr6) for non-GitHub users
- Update notifications: Google Form (https://forms.gle/J91tRCamevEvzRqc7)
