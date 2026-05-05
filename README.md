# 🏡 Can I Afford This House?

A free, open-source calculator that compares the *true* financial cost of buying 
a home vs. renting and saving/investing the difference — built for everyday people, 
not financial professionals.

👉 **Live site:** [caniaffordthishouse.app](https://caniaffordthishouse.app)

![Calculator preview](assets/HAC-Preview.png)

---

## What Makes This Different

Most rent vs. buy calculators stop at your monthly mortgage payment. This one goes further:

- **Full cost accounting** — mortgage P&I, property taxes, HOA, insurance, maintenance, 
  PMI, and utilities on the buying side; rent, renters insurance, utilities, and savings 
  contributions on the renting side
- **Long-term side-by-side comparison** — cumulative spending and financial return 
  over your chosen timeline (up to 30 years)
- **Breakeven analysis** — when does buying's monthly cost, cumulative spending, 
  and net financial position cross over renting?
- **Recommended income estimates** — 30% rule and current lifestyle approaches, 
  both using net (take-home) income as the base
- **Advanced scenarios** — mortgage buydowns (2-1, 3-2-1, permanent), extra principal 
  paydown, capital gains tax on sale, manual PMI override
- **Shareable results** — encode your full input state into a URL, or download 
  a summary image to share
- **Exportable data** — download year-by-year buying and renting data as CSV files
- **Educational** — every input is explained; results are contextualized, not just 
  displayed

---

## How to Use It

Open [caniaffordthishouse.app](https://caniaffordthishouse.app), fill in your 
details across the four input sections, and hit Calculate. That's it.

Most inputs have sensible defaults. Required fields are: purchase price, down payment, 
interest rate, timeline, monthly rent, and rate of return.

---

## Tech Stack & Architecture

- Vanilla HTML, CSS, and JavaScript — no frameworks, no build tools, no dependencies
- Single-file architecture (`index.html`) with one exception: JSZip v3.10.1 
  (`assets/jszip.min.js`), hosted locally for the CSV export feature
- 100% client-side — no backend, no tracking, no user accounts
- Hosted on GitHub Pages with a custom domain

---

## Project Documentation

The `docs/` files contain the full technical and design record for this project:

| File | Contents |
|---|---|
| `CALCULATION_LOGIC.md` | All formulas, data flow, input IDs, and calculation order |
| `KNOWN_ISSUES_AND_LEARNINGS.md` | Bug history, fixes, edge cases, and what not to change |
| `UX_AND_CONTENT_GUIDELINES.md` | Tone, layout rules, color palette, interaction decisions |
| `PROJECT_CONTEXT.md` | Goals, philosophy, SEO strategy, and build state |

---

## Contributing & Feedback

This project has a few hard constraints that any contribution should respect:
- No JavaScript frameworks
- No backend or external API dependencies  
- No monetization of any kind
- Single-file architecture (keep logic in `index.html`)

For bug reports, feature ideas, or general feedback:

- **GitHub Discussions:** [Open a discussion](https://github.com/weaseljohnson/housing-affordability-calculator/discussions)
- **Google Form:** [Submit feedback](https://forms.gle/V9WyUTyqKpsAfzJr6) *(for non-GitHub users)*

---

## Disclaimer

I'm no professional finance coach — just a regular person who wanted to make informed 
financial decisions and is also kind of a nerd. 🤓 This is a tool to help you think, 
not professional financial advice.

---

## License

© 2026 Joshua Johnson — [MIT License](LICENSE)