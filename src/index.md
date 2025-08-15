---
toc: false
theme: [light, alt]
---

<div class="hero">
  <img src="logo.png" alt="UVM Graduate Students United Logo" style="height: 100px; margin-right: 25px;">
  <h1>UVM Graduate Students United</h1>
</div>

# Graduate Students Deserve a Living Wage!
<p>As it stands, the UVM minimum graduate student stipend of <strong>$32,000 is not enough</strong> to live in Chittenden County according to both the <a href="https://ljfo.vermont.gov/publications/report/basic-needs-budget-reports">Vermont Basic Needs Budget</a> by the State of Vermont and the <a href="https://livingwage.mit.edu/">MIT Living Wage Calculator</a>. Students are forced to compromise on necessities like housing and food in order to study at UVM. So far, the administration has failed to address these concerns.</p>
<p>Graduate students who are unable to afford food and a safe living environment are unable to fully focus on the research and teaching at hand. The administration is doing itself a disservice by neglecting to address these need. This dashboard shows what graduate students are facing when making their budgets…</p>

---

## A Breakdown of VT Basic Needs Budget
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
  currency: "USD"
});

```

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
  
  return {
    plot: Plot.plot({
      width: width,
      height: width * 0.3, // Maintain aspect ratio
      marginLeft: 60, // give space for the y-axis labels
      y: {
        grid: true,
        label: "Dollar Amount ↓",
        nice: true
      },
      x: {
        domain: transformedData.map(d => d.category),
        label: "Categories of Spending →"
      },
      marks: [
        // A horizontal rule at y=0 for reference
        Plot.ruleY([0], { stroke: "gold" }),

        // Add dotted line for total
        Plot.ruleY([total], { stroke: "black", strokeDasharray: "4,4" }),

        // Rectangles from y1 to y2
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
          text: d => d.value !== 0 ? formatUSD.format(d.value) : "",
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

<div class="card" style="padding: 10px; max-width: 100%; overflow: hidden; position: relative;">
  <div style="position: absolute; top: 10px; right: 20px; font-size: 20px; font-weight: bold;">
    Total: ${(() => {
      const result = plotSpending(data, scenario, year, 800);
      return formatUSD.format(result.total);
    })()}
  </div>
  <h3 style="text-align: center; margin: 0 auto;">Graduate Student Spending</h3> 
  ${resize((width) => plotSpending(data, scenario, year, width).plot)}
</div>

<p>As you can see, this graph shows that the current stipend does not cover the average costs of basic needs in Burlington such as rent and utilities, food, transportation, and other necessities. This means that employees are</p>

---

## Inflation has Outpaced Stipend Increases

```js
function computePercentChangeCommon(scenario, baseYear, compareYears, data) {
  const baseData = data[translateScenario(scenario, baseYear)];
  let commonCategories = new Set(baseData.map(d => d.category));
  
  // Intersect with categories from each comparison year.
  for (const year of compareYears) {
    const compareData = data[translateScenario(scenario, year)];
    const compareCategories = new Set(compareData.map(d => d.category));
    commonCategories = new Set([...commonCategories].filter(category => compareCategories.has(category)));
  }
  
  // Build a lookup map for the base year (only for common categories).
  const baseMap = new Map(
    baseData.filter(d => commonCategories.has(d.category))
            .map(d => [d.category, d.value])
  );
  
  const results = [];
  // For each comparison year, compute the percent change for each common category.
  for (const year of compareYears) {
    const compareData = data[translateScenario(scenario, year)];
    const compareMap = new Map(
      compareData.filter(d => commonCategories.has(d.category))
                 .map(d => [d.category, d.value])
    );
    
    for (const category of commonCategories) {
      const baseValue = baseMap.get(category);
      const compareValue = compareMap.get(category);
      // Only compute percent change if baseValue is non-zero and both values exist.
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

  return Plot.plot({
    width: width,
    height: width * 0.3,
    marginLeft: 60,

    x: {
      label: "Spending Category",
      domain: ["Total Salary", ...[...new Set(pctDataCommon.map(d => d.category))].sort((a, b) => {
        const aValue = pctDataCommon.find(d => d.category === a && d.year === "2024").percentChange;
        const bValue = pctDataCommon.find(d => d.category === b && d.year === "2024").percentChange;
        return bValue - aValue;
            })],
    },
    y: {
      label: "Percent Change (%)",
      grid: true,
      domain: [
        d3.min(pctDataCommon, d => d.percentChange)*2.2,
        d3.max(pctDataCommon, d => d.percentChange)*1.5
      ]
    },
    color: {
      legend: true,
      label: "Year",
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

<style>

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

@media (max-width: 640px) {
  .hero h1 {
    font-size: 70px;  // Changed from 90px
  }
}

/* Add media query for mobile responsiveness */
@media (max-width: 768px) {
  .grid {
    display: grid;
    grid-template-columns: 1fr !important;
    gap: 1rem;
  }
}

</style>
