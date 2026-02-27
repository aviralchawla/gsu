# UVM Graduate Students United — Stipend & Cost-of-Living Dashboard

An advocacy data dashboard built with [Observable Framework](https://observablehq.com/framework/) for **GEO-UAW 2322**, the graduate worker union at the University of Vermont. The dashboard documents the gap between UVM's minimum graduate stipend and the actual cost of living in Chittenden County, making the case for fair bargaining outcomes.

Live site: [uvmgsu.uaw2322.org](https://uvmgsu.uaw2322.org)

---

## What it shows

- **Stipend vs. cost-of-living benchmarks** — UVM stipend against Vermont minimum wage, MIT Living Wage, and Vermont Basic Needs Budget across three household scenarios
- **Full budget waterfall** — Where every dollar of the stipend goes across spending categories, by year and scenario (2020, 2022, 2024)
- **HUD Fair Market Rents** — Burlington-area rental costs from 2020–2024 showing the surge in housing costs
- **Inflation since 2020** — Category-by-category percent change in costs vs. stipend growth
- **Peer institution comparison** — UVM stipend vs. union contracts at Rutgers, University of Michigan, and Columbia

## Data sources

All data is hardcoded in `src/data/` as JSON files:

| File | Source |
|------|--------|
| `single-with-roommates-*.json` | Vermont Basic Needs Budget |
| `single-no-roommates-*.json` | Vermont Basic Needs Budget |
| `with-child-*.json` | Vermont Basic Needs Budget |
| `hud-fmr.json` | HUD Fair Market Rents (Chittenden County) |
| `peer-stipends.json` | Union contracts: Rutgers AAUP-AFT, UMich GEO-UAW, Columbia UAW 2710 |

## Development

```sh
npm install       # Install dependencies
npm run dev       # Start local preview at http://localhost:3000
npm run build     # Build static site to ./dist
npm run deploy    # Deploy to Observable
npm run clean     # Clear the data loader cache
```

## Project structure

```
src/
├─ index.md          # Main dashboard page (all charts and narrative)
├─ data/             # Static JSON data files
└─ public/
   └─ logo.png       # UVM GSU logo
observablehq.config.js
```

## Credits

Produced in collaboration with the **Stipends & Fees Working Group**: Baxter Worthing, Ethan Ratliff-Crain, Francis Guarascio, and Hannah Gokaslan.
