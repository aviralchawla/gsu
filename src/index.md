---
toc: false
theme: [light, alt]
---

<div class="hero">
  <img src="logo.png" alt="UVM Graduate Students United Logo" style="height: 100px; margin-right: 25px;">
  <h1>UVM Graduate Students United</h1>
</div>

# Graduate Students Deserve a Living Wage!

<p>UVM's minimum graduate student stipend is <strong>$34,375</strong>. The actual cost of living for a single adult in Chittenden County, according to MIT, is <strong>$56,036</strong>. Even sharing a home with a roommate costs over <strong>$40,000</strong> a year. The numbers don't add up. They haven't for years. And they're getting worse.</p>

<p>This dashboard documents the gap between what graduate workers are paid and what it costs to live here — and why we are at the bargaining table fighting to close it.</p>

---

```js
const single_with_room_2024 = FileAttachment("data/single-with-roommates-2024.json").json()
const single_no_room_2024 = FileAttachment("data/single-no-roommates-2024.json").json()
const with_child_2024 = FileAttachment("data/with-child-2024.json").json()
const single_with_room_2022 = FileAttachment("data/single-with-roommates-2022.json").json()
const single_no_room_2022 = FileAttachment("data/single-no-roommates-2022.json").json()
const with_child_2022 = FileAttachment("data/with-child-2022.json").json()
const single_with_room_2020 = FileAttachment("data/single-with-roommates-2020.json").json()
const single_no_room_2020 = FileAttachment("data/single-no-roommates-2020.json").json()
const with_child_2020 = FileAttachment("data/with-child-2020.json").json()
const hudFmr = FileAttachment("data/hud-fmr.json").json()
const peerStipends = FileAttachment("data/peer-stipends.json").json()
```

```js
const data = {
  "single_with_room_2024": single_with_room_2024,
  "single_no_room_2024": single_no_room_2024,
  "with_child_2024": with_child_2024,
  "single_with_room_2022": single_with_room_2022,
  "single_no_room_2022": single_no_room_2022,
  "with_child_2022": with_child_2022,
  "single_with_room_2020": single_with_room_2020,
  "single_no_room_2020": single_no_room_2020,
  "with_child_2020": with_child_2020
}
```

```js
const transformData = (data) => {
  let cumulative = 0;
  return data.map(d => {
    const y1 = cumulative;
    const y2 = y1 + d.value;
    cumulative = y2;
    return { ...d, y1, y2 };
  });
};

const formatUSD = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD",
  maximumFractionDigits: 0
});
```

---

## The Gap Is Not a Rounding Error

```js
function totalCosts(scenData) {
  return scenData.filter(d => d.value < 0).reduce((sum, d) => sum + Math.abs(d.value), 0);
}

const benchmarkData = [
  { label: "Vermont Min. Wage (annualized)", shortLabel: "VT Min. Wage", value: 29994, type: "Reference" },
  { label: "UVM Graduate Stipend", shortLabel: "UVM Stipend", value: 34375, type: "Stipend" },
  { label: "Basic Needs: Single + Roommate", shortLabel: "BN: w/ Roommate", value: totalCosts(single_with_room_2024), type: "Basic Needs" },
  { label: "Basic Needs: Single, No Roommate", shortLabel: "BN: No Roommate", value: totalCosts(single_no_room_2024), type: "Basic Needs" },
  { label: "MIT Living Wage (single adult)", shortLabel: "MIT Living Wage", value: 56036, type: "Reference" },
  { label: "Basic Needs: Single Parent", shortLabel: "BN: Single Parent", value: totalCosts(with_child_2024), type: "Basic Needs" },
];

function plotBenchmarks(width) {
  const isMobile = width < 500;
  const ml = isMobile ? Math.min(120, Math.floor(width * 0.38)) : 245;
  const mr = isMobile ? 50 : 90;
  const displayData = benchmarkData.map(d => ({ ...d, label: isMobile ? d.shortLabel : d.label }));
  return Plot.plot({
    width,
    height: isMobile ? 200 : 260,
    marginLeft: ml,
    marginRight: mr,
    x: {
      label: "Annual Amount",
      tickFormat: d => `$${Math.round(d / 1000)}k`,
      grid: true
    },
    color: {
      domain: ["Stipend", "Reference", "Basic Needs"],
      range: ["#0b5032", "#aaa", "#ef5350"],
      legend: true,
      label: ""
    },
    marks: [
      Plot.barX(displayData, {
        y: "label",
        x: "value",
        fill: "type",
        sort: { y: "x" },
        rx: 3
      }),
      Plot.ruleX([34375], {
        stroke: "#0b5032",
        strokeWidth: 2,
        strokeDasharray: "6,3"
      }),
      Plot.text(displayData, {
        y: "label",
        x: "value",
        text: d => isMobile ? `$${Math.round(d.value / 1000)}k` : formatUSD.format(d.value),
        dx: 4,
        textAnchor: "start",
        fontSize: isMobile ? 10 : 12
      })
    ]
  });
}
```

<p>The UVM stipend sits just <strong>$4,381 above Vermont's minimum wage</strong> and tens of thousands below what it actually costs to live in Burlington. Even under the most generous setting, graduate workers end the year more than $6,200 in the hole. Without a roommate: about $15,500 short. Supporting a child? Over $40,000!</p>

<div class="card" style="padding: 10px; max-width: 100%; overflow: hidden;">
  <h3 style="text-align: center; margin: 0 auto 10px;">Stipend vs. Cost of Living Benchmarks (Chittenden County, 2024)</h3>
  ${resize((width) => plotBenchmarks(width))}
</div>

---

## Here's Where Every Dollar Goes

<p>The chart below traces the full budget under each scenario. Every dollar of income is committed even before the month begins.</p>

```js
function translateScenario(scenario, year) {
  if (scenario === "Single with a Roommate") {
    return `single_with_room_${year}`;
  } else if (scenario === "Single without a Roommate") {
    return `single_no_room_${year}`;
  } else if (scenario === "With a Child") {
    return `with_child_${year}`;
  }
}

function plotSpending(data, scenario, year, width) {
  const scen_data = data[translateScenario(scenario, year)];
  const transformedData = transformData(scen_data);
  const total = transformedData[transformedData.length - 1].y2;
  const triangle = d3.symbol().type(d3.symbolTriangle).size(100);

  const isMobile = width < 500;
  return {
    plot: Plot.plot({
      width: width,
      height: Math.max(width * 0.3, isMobile ? 200 : 160),
      marginLeft: 60,
      marginBottom: isMobile ? 75 : 30,
      y: {
        grid: true,
        label: "Dollar Amount ↓",
        nice: true
      },
      x: {
        domain: transformedData.map(d => d.category),
        label: "Categories of Spending →",
        tickRotate: isMobile ? -45 : 0
      },
      marks: [
        Plot.ruleY([0], { stroke: "gold" }),
        Plot.ruleY([total], { stroke: "black", strokeDasharray: "4,4" }),
        Plot.rectY(transformedData, {
          x: "category",
          y1: "y1",
          y2: "y2",
          fill: d => d.value >= 0 ? "#63D453" : "#ef5350"
        }),
        Plot.text(transformedData, {
          x: "category",
          y: d => (d.y1 + d.y2) / 2,
          dy: 5,
          text: d => !isMobile && d.value !== 0 ? formatUSD.format(d.value) : "",
          fill: "black",
          textAnchor: "middle"
        })
      ]
    }),
    total: total
  };
}
```

```js
const scenarioInput = Inputs.radio(["Single with a Roommate", "Single without a Roommate", "With a Child"], {value: "Single with a Roommate", label: "Scenario:"});
const scenario = view(scenarioInput)
```
```js
const yearInput = Inputs.radio(["2020", "2022", "2024"], {value: "2024", label: "Year:"});
const year = view(yearInput)
```

<div class="card" style="padding: 10px; max-width: 100%; overflow: hidden;">
  <h3 style="text-align: center; margin: 0 auto 2px;">Graduate Student Spending</h3>
  <p style="text-align: center; margin: 0 0 6px; font-size: 1.05rem; font-weight: bold;">
    Total: ${(() => {
      const result = plotSpending(data, scenario, year, 800);
      return formatUSD.format(result.total);
    })()}
  </p>
  ${resize((width) => plotSpending(data, scenario, year, width).plot)}
</div>

<p>Housing alone consumes a third of the stipend before food, transportation, or healthcare are counted.</p>

---

## Burlington's Rents Have Surged

<p>Housing costs have not held steady. Burlington, and Vermont at large, has seen an big surge in housing costs. HUD's Fair Market Rent — the federal government's own benchmark for what rental housing should cost — has risen sharply across Chittenden County. A one-bedroom that cost <strong>$1,223/month</strong> in 2020 costs <strong>$1,595</strong> in 2024 — a <strong>30% increase in four years</strong>, most of it concentrated in a single year. The stipend grew 18.5% over the same period.</p>

```js
function plotHudFmr(data, width) {
  return Plot.plot({
    width,
    height: Math.max(width * 0.3, 160),
    marginLeft: 60,
    marginTop: 25,
    marginRight: 75,
    x: {
      label: "Year",
      ticks: [2020, 2023, 2024],
      tickFormat: d => String(d)
    },
    y: {
      label: "Monthly Rent ($)",
      grid: true,
      tickFormat: d => `$${d.toLocaleString()}`
    },
    color: {
      legend: true,
      label: "Unit Type",
      domain: ["1-Bedroom", "2-Bedroom"],
      range: ["#e57373", "#c62828"]
    },
    marks: [
      Plot.lineY(data, {
        x: "year",
        y: "monthly",
        stroke: "type",
        strokeWidth: 2.5,
        marker: "dot"
      }),
      Plot.text(data.filter(d => d.type === "1-Bedroom"), {
        x: "year",
        y: "monthly",
        text: d => `$${d.monthly.toLocaleString()}`,
        textAnchor: "start",
        dx: 5,
        dy: -14,
        fontSize: 11
      }),
      Plot.text(data.filter(d => d.type === "2-Bedroom"), {
        x: "year",
        y: "monthly",
        text: d => `$${d.monthly.toLocaleString()}`,
        textAnchor: "start",
        dx: 5,
        dy: -14,
        fontSize: 11
      })
    ]
  });
}
```

<div class="card" style="padding: 10px; max-width: 100%; overflow: hidden;">
  <h3 style="text-align: center; margin: 0 auto;">HUD Fair Market Rents — Chittenden County (2020–2025)</h3>
  ${resize((width) => plotHudFmr(hudFmr, width))}
</div>

<p>A two-bedroom apartment — the basis for the most affordable roommate scenario — now costs <strong>$1,887/month</strong>: <strong>$22,644 annually</strong>, or 71% of the entire stipend, before any other expense.</p>

---

## It's Not Just Rent

<p>Between 2020 and 2024, the stipend grew <strong>27%</strong>. Food costs rose <strong>43%</strong>. Healthcare climbed <strong>more than 50%</strong>. What looks like a raise is, in real terms, a pay cut.</p>

```js
function computePercentChangeCommon(scenario, baseYear, compareYears, data) {
  const baseData = data[translateScenario(scenario, baseYear)];
  let commonCategories = new Set(baseData.map(d => d.category));

  for (const year of compareYears) {
    const compareData = data[translateScenario(scenario, year)];
    const compareCategories = new Set(compareData.map(d => d.category));
    commonCategories = new Set([...commonCategories].filter(category => compareCategories.has(category)));
  }

  const baseMap = new Map(
    baseData.filter(d => commonCategories.has(d.category))
            .map(d => [d.category, d.value])
  );

  const results = [];
  for (const year of compareYears) {
    const compareData = data[translateScenario(scenario, year)];
    const compareMap = new Map(
      compareData.filter(d => commonCategories.has(d.category))
                 .map(d => [d.category, d.value])
    );

    for (const category of commonCategories) {
      const baseValue = baseMap.get(category);
      const compareValue = compareMap.get(category);
      if (baseValue !== 0 && baseValue != null && compareValue != null) {
        const pctChange = ((compareValue - baseValue) / baseValue) * 100;
        if (pctChange !== 0) {
          results.push({
            category,
            year,
            percentChange: pctChange
          });
        }
      }
    }
  }
  return results;
}

function plotPercentChange(data, scenario, base_year, compare_years, width) {
  const pctDataCommon = computePercentChangeCommon(scenario, base_year, compare_years, data);

  const isMobilePct = width < 500;
  return Plot.plot({
    width: width,
    height: Math.max(width * 0.3, isMobilePct ? 200 : 160),
    marginLeft: 60,
    marginBottom: isMobilePct ? 75 : 30,
    x: {
      label: "Spending Category",
      tickRotate: isMobilePct ? -45 : 0,
      domain: ["Total Salary", ...[...new Set(pctDataCommon.map(d => d.category))].filter(c => c !== "Total Salary").sort((a, b) => {
        const aValue = (pctDataCommon.find(d => d.category === a && d.year === "2024") || { percentChange: 0 }).percentChange;
        const bValue = (pctDataCommon.find(d => d.category === b && d.year === "2024") || { percentChange: 0 }).percentChange;
        return bValue - aValue;
      })],
    },
    y: {
      label: "Percent Change (%)",
      grid: true,
      domain: [
        Math.min(0, (d3.min(pctDataCommon, d => d.percentChange) || 0) * 2.2),
        Math.max(10, (d3.max(pctDataCommon, d => d.percentChange) || 10) * 1.5)
      ]
    },
    color: {
      legend: true,
      label: "Year",
      type: "ordinal",
      domain: ["2022", "2024"],
      range: ["#4c78a8", "#f58518"]
    },
    marks: [
      Plot.ruleY([0], { stroke: "black"}),
      Plot.barY(pctDataCommon, {
        x: "category",
        y: "percentChange",
        fill: "year",
        dodge: true,
        title: d => `${d.category} (${d.year})\n${d.percentChange.toFixed(1)}%`
      })
    ]
  })
}
```

```js
const scenarioInput_inflation = Inputs.radio(["Single with a Roommate", "Single without a Roommate", "With a Child"], {value: "Single with a Roommate", label: "Scenario:"});
const scenario_inflation = view(scenarioInput_inflation)
```

<div class="card" style="padding: 10px; max-width: 100%; overflow: hidden; position: relative;">
  <h3 style="text-align: center; margin: 0 auto;">% Price Changes Since 2020</h3>
  ${resize((width) => plotPercentChange(data, scenario_inflation, "2020", ["2022", "2024"], width))}
</div>

---

## Other Universities Are Paying More

<p>UVM is far from the only institution that employs graduate workers. But it ranks among the lowest payers of universities where graduate workers have organized. The gap is not a function of what universities can afford, but a function of what they have been made to offer at the bargaining table.</p>

```js
const shortInstitution = {
  "UVM (current)": "UVM (current)",
  "Rutgers (AAUP-AFT)": "Rutgers",
  "GEO-UAW 2322 demand": "Our Demand",
  "Univ. of Michigan (GEO)": "U. Michigan",
  "Columbia University": "Columbia"
};

function plotPeerStipends(data, width) {
  const isMobile = width < 500;
  const ml = isMobile ? Math.min(100, Math.floor(width * 0.32)) : 240;
  const mr = isMobile ? 50 : 90;
  const displayData = data.map(d => ({ ...d, institution: isMobile ? (shortInstitution[d.institution] || d.institution) : d.institution }));
  return Plot.plot({
    width,
    height: isMobile ? 200 : 270,
    marginLeft: ml,
    marginRight: mr,
    x: {
      label: "Annual Stipend",
      tickFormat: d => `$${Math.round(d / 1000)}k`,
      grid: true
    },
    color: {
      domain: ["UVM", "Peer institution", "Union demand"],
      range: ["#ef5350", "#aaa", "#0b5032"],
      legend: true,
      label: ""
    },
    marks: [
      Plot.barX(displayData, {
        y: "institution",
        x: "annual",
        fill: "type",
        sort: { y: "x" },
        rx: 3
      }),
      Plot.text(displayData, {
        y: "institution",
        x: "annual",
        text: d => isMobile ? `$${Math.round(d.annual / 1000)}k` : formatUSD.format(d.annual),
        dx: 4,
        textAnchor: "start",
        fontSize: isMobile ? 10 : 12
      })
    ]
  });
}
```

<div class="card" style="padding: 10px; max-width: 100%; overflow: hidden;">
  <h3 style="text-align: center; margin: 0 auto;">Annual Doctoral Stipend: UVM vs. Peer Institutions</h3>
  ${resize((width) => plotPeerStipends(peerStipends, width))}
</div>

<p>GEO-UAW 2322 is planning to propose a minimum demand of <strong>$40,000</strong>, This still falls below what graduate workers earn at the University of Michigan and Columbia under their existing union contracts. Rutgers, also in active bargaining, already exceeds UVM's current stipend. Good bargaining has moved the needle at public universities across the country. It can at UVM too.</p>

---

## What We're Asking For

<p>Graduate workers are employees. We teach undergraduate courses, staff research labs, and generate the scholarship that defines this university. We are not asking for extraordinary treatment, we are asking for the bare minimum. We are asking to be paid enough to live in the county where we work, and for that floor to keep pace with rising costs so we are not back making this same argument in two years.</p>

<p>UVM has the resources. What has been lacking is the will. We are at the bargaining table to change that.</p>

---

## We Need Your Support

<p>Data doesn't win contracts, people do. This is not only a graduate worker issue. When the people who teach your courses, staff the labs your research depends on, and mentor the next generation of scholars cannot afford to live in Burlington, the entire university community bears the cost. A fair contract at UVM sets a precedent that matters well beyond this campus.</p>

<div class="support-grid">
  <div class="support-card">
    <strong>Graduate workers</strong>
    <p>Show up. Attend open bargaining sessions. Stay engaged and make your voice heard — the strength of this contract is a direct function of the number of people behind it.</p>
  </div>
  <div class="support-card">
    <strong>Faculty and staff</strong>
    <p>A public statement of solidarity carries real weight at the bargaining table. Speak up — to your departments, your deans, and the administration directly.</p>
  </div>
  <div class="support-card">
    <strong>Undergraduate students</strong>
    <p>The graduate workers who teach your courses are being paid poverty wages in real terms. Contact the administration. Make some noise about it.</p>
  </div>
  <div class="support-card">
    <strong>Everyone else</strong>
    <p>Share this dashboard. Follow our bargaining updates. Show up to solidarity events when they happen.</p>
  </div>
</div>

<div class="cta-buttons" style="margin-top: 1.5rem;">
  <a href="https://uvmgsu.uaw2322.org" target="_blank" class="cta-btn">More from UVM GSU</a>
  <a href="https://uvmgsu.uaw2322.org/bargaining/" target="_blank" class="cta-btn cta-btn-outline">Follow our Bargaining</a>
</div>

---

## Acknowledgements ✊❤️

This work was produced in collaboration with the Stipends & Fees Working Group: **Baxter Worthing**, **Ethan Ratliff-Crain**, **Francis Guarascio**, and **Hannah Gokaslan**.

This dashboard, like so much of what this union produces, rests on the sustained effort and dedication of graduate students at the University of Vermont — past and present. We are deeply grateful for that collective labor.

<style>

.cta-buttons {
  display: flex;
  gap: 1rem;
  margin: 1.5rem 0;
  flex-wrap: wrap;
}

.cta-btn {
  display: inline-block;
  padding: 0.6rem 1.4rem;
  border-radius: 6px;
  font-weight: 600;
  font-size: 0.95rem;
  text-decoration: none !important;
  transition: background 0.2s, color 0.2s, border-color 0.2s;
  background: #0b5032;
  color: #fff !important;
  border: 2px solid #0b5032;
}

.cta-btn:hover {
  background: #063a23;
  border-color: #063a23;
  color: #fff !important;
}

.cta-btn-outline {
  background: transparent;
  color: #0b5032 !important;
}

.cta-btn-outline:hover {
  background: #0b5032;
  color: #fff !important;
}

.support-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  margin: 1.5rem 0;
}

.support-card {
  background: var(--theme-background-alt, #f4f4f4);
  border-left: 4px solid #0b5032;
  border-radius: 6px;
  padding: 1rem 1.2rem;
}

.support-card strong {
  display: block;
  font-size: 1rem;
  color: #0b5032;
  margin-bottom: 0.4rem;
}

.support-card p {
  margin: 0;
  font-size: 0.9rem;
  line-height: 1.5;
}

#observablehq-footer {
  display: none;
}

.hero {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  font-family: var(--sans-serif);
  margin: 0rem 0 2rem;
  text-wrap: balance;
  text-align: center;
}

.hero h1 {
  margin: 1rem 0;
  padding: 1rem 0;
  max-width: none;
  font-size: 65px;
  font-weight: 700;
  line-height: 1;
  background: linear-gradient(30deg, #0b5032,rgb(17, 109, 62));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.hero h2 {
  margin: 0;
  max-width: 34em;
  font-size: 20px;
  font-style: initial;
  font-weight: 500;
  line-height: 1.5;
  color: var(--theme-foreground-muted);
}

@media (max-width: 768px) {
  .hero h1 {
    font-size: 42px;
  }
  .grid {
    display: grid;
    grid-template-columns: 1fr !important;
    gap: 1rem;
  }
}

@media (max-width: 640px) {
  .hero {
    flex-direction: column;
    margin: 0 0 1rem;
  }
  .hero h1 {
    font-size: 32px;
  }
  .support-grid {
    grid-template-columns: 1fr;
  }
  .cta-buttons {
    flex-direction: column;
    align-items: stretch;
  }
  .cta-btn {
    text-align: center;
  }
  .card {
    padding: 8px !important;
  }
}

</style>
