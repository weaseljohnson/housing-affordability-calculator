# 🏡 Can I Afford This House?

A free, open-source suite of housing calculators built for everyday people, not financial professionals.

👉 **Live site:** [caniaffordthishouse.app](https://caniaffordthishouse.app)

![Calculator preview](assets/HAC-Preview.png)

---

## Tools

### 🏠 Home Affordability Calculator
**[caniaffordthishouse.app/Affordability/](https://caniaffordthishouse.app/Affordability/)**

Answers the first question most buyers ask: *Can I actually afford this home?* Enter a property and your financial details to get a clear picture of your total upfront costs, true monthly costs, and the income you'd realistically need.

### ⚖️ Rent vs. Buy Calculator
**[caniaffordthishouse.app](https://caniaffordthishouse.app)**

Goes further — puts buying and renting side by side over your chosen timeline, including breakeven analysis, investment return modeling on rent savings, and a full net financial position comparison.

---

## What Makes These Different

Most housing calculators stop at your monthly mortgage payment. These go further:

- **Full cost accounting** — mortgage P&I, property taxes, HOA, insurance, maintenance, PMI, and utilities on the buying side; rent, renters insurance, utilities, and savings contributions on the renting side
- **Long-term comparison** — cumulative spending and financial return over your chosen timeline (up to 30 years)
- **Breakeven analysis** *(Rent vs. Buy only)* — when does buying's monthly cost, cumulative spending, and net financial position cross over renting?
- **Recommended income estimates** — 30% rule and current lifestyle approaches, both using net (take-home) income as the base
- **Advanced scenarios** — mortgage buydowns (2-1, 3-2-1, permanent), extra principal paydown, capital gains tax on sale, manual PMI override
- **Shareable results** — encode your full input state into a URL, or download a summary image to share
- **Exportable data** — download year-by-year data as CSV files
- **Educational** — every input is explained; results are contextualized, not just displayed

---

## How to Use

Open the calculator of your choice, fill in your details across the input sections, and hit Calculate. That's it.

Most inputs have sensible defaults. The Affordability Calculator requires: purchase price, down payment, interest rate, and timeline. The Rent vs. Buy Calculator additionally requires monthly rent and expected rate of return.

---

## Tech Stack & Architecture

- Vanilla HTML, CSS, and JavaScript — no frameworks, no build tools, no dependencies
- Single-file architecture per tool (`index.html`) with one shared exception: JSZip v3.10.1 (`assets/jszip.min.js`), hosted locally for the CSV export feature
- 100% client-side — no backend, no tracking, no user accounts
- Hosted on GitHub Pages with a custom domain

---

## Project Documentation

The `docs/` folder contains the full technical and design record for each tool.

**Affordability Calculator (`AFB_*`)**

| File | Contents |
|---|---|
| `AFB_CALCULATION_LOGIC.md` | All formulas, data flow, input IDs, and calculation order |
| `AFB_KNOWN_ISSUES_AND_LEARNINGS.md` | Bug history, fixes, edge cases, and what not to change |
| `AFB_UX_AND_CONTENT_GUIDELINES.md` | Tone, layout rules, color palette, interaction decisions |
| `AFB_PROJECT_CONTEXT.md` | Goals, philosophy, SEO strategy, and build state |

**Rent vs. Buy Calculator (`RB_*`)**

| File | Contents |
|---|---|
| `RvB_CALCULATION_LOGIC.md` | All formulas, data flow, input IDs, and calculation order |
| `RvB_KNOWN_ISSUES_AND_LEARNINGS.md` | Bug history, fixes, edge cases, and what not to change |
| `RvB_UX_AND_CONTENT_GUIDELINES.md` | Tone, layout rules, color palette, interaction decisions |
| `RvB_PROJECT_CONTEXT.md` | Goals, philosophy, SEO strategy, and build state |

---

## Contributing & Feedback

A few hard constraints any contribution should respect:
- No JavaScript frameworks
- No backend or external API dependencies
- No monetization of any kind
- Single-file architecture per tool (keep logic in `index.html`)

For bug reports, feature ideas, or general feedback:

- **GitHub Discussions:** [Open a discussion](https://github.com/weaseljohnson/housing-affordability-calculator/discussions)
- **Google Form:** [Submit feedback](https://forms.gle/V9WyUTyqKpsAfzJr6) *(for non-GitHub users)*

---

## Disclaimer

I'm no professional finance coach — just a regular person who wanted to make informed financial decisions and is also kind of a nerd. 🤓 These are tools to help you think, not professional financial advice.

---

## License

© 2026 Joshua Johnson — [MIT License](LICENSE)