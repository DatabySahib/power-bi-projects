# Power BI Projects

A portfolio of seven end-to-end Power BI projects, each stored in **PBIP** (Power BI Project) format — the semantic model as [TMDL](https://learn.microsoft.com/power-bi/developer/projects/projects-dataset#tmdl-format) text files, the report as [PBIR](https://learn.microsoft.com/power-bi/developer/projects/projects-report) JSON, and the source data alongside it. Everything is plain text and diffable; there are no `.pbix` binaries in the repo.

Each project is self-contained: open its `.pbip` file in Power BI Desktop and you get the model, the measures, and the report page.

## Projects

| Project | Domain | Source data | Model shape | Report page |
|---|---|---|---|---|
| [Customer Lifetime Value Analysis](Customer%20Lifetime%20Value%20Analysis) | Retail / CLV | `customer_data.csv` | Star: `FactTransactions` + `DimCustomer`, `DimDate` | CLV Overview |
| [Customer Segmentation Analysis](Customer%20Segmentation%20Analysis) | Retail / RFM | `superstore.csv` (15 MB) | `Orders` + calculated `Customer RFM` dimension, `Calendar` | RFM Dashboard |
| [HR Attrition and Headcount Dashboard](HR%20Attrition%20and%20Headcount%20Dashboard) | People analytics | IBM HR Employee Attrition (`WA_Fn-UseC_-HR-Employee-Attrition.csv`) | Single flat `HR_Data` table | HR Executive Overview · Demographics & Attrition Factors |
| [Modeling Data in Power BI](Modeling%20Data%20in%20Power%20BI) | Data modeling practice | Northwind Traders CSV extracts (`Northwind Traders.zip`) | Full 7-table snowflake: `orders`, `order_details`, `products`, `categories`, `customers`, `employees`, `shippers` | Page 1 |
| [Predictive Sales Forecasting](Predictive%20Sales%20Forecasting) | Sales / forecasting | `Sample - Superstore.csv` | `Orders` + marked date table `Dim_Date` (active on Order Date, inactive on Ship Date) | Executive Sales Overview & Forecasting |
| [Social Media Sentiment Analysis](Social%20Media%20Sentiment%20Analysis) | Text / brand monitoring | Sentiment140-style tweet dataset (`train.csv`, `test.csv`) | Star: `FactSocialSentiment` + `DimGeography`, `DimAgeGroup` | Sentiment & Crisis Monitoring |
| [Visualization of Life Expectancy and GDP Variation Over Time](Visualization%20of%20Life%20Expectancy%20and%20GDP%20Variation%20Over%20Time) | Global development | Gapminder (`data/gapminder_full.csv`) | Single wide `gapminder` table | Gapminder Dashboard |

### What each one demonstrates

**Customer Lifetime Value Analysis** — Historic CLV built up from its components: AOV × purchase frequency, plus repeat-purchase rate, average customer lifespan in days, and revenue share by customer. Measures are organised into display folders (`01 Base Metrics`, `02 Customer Behavior`, `03 CLV`). Report: five KPI cards, revenue by segment and location, a daily revenue trend, and a customer-detail matrix with date/location/segment slicers.

**Customer Segmentation Analysis** — A full RFM implementation. The `Customer RFM` calculated table assigns every customer (4,873 of them) a 1–5 quintile score on Recency, Frequency and Monetary against a fixed 2014-12-31 snapshot, then maps the scores to six named segments (Champions, Loyal Customers, Potential Loyalists, Needs Attention, At Risk, Lost / Hibernating) with a sort-order key so they read best-to-worst. A parallel set of "(Dynamic)" measures recomputes R/F/M inside the current filter context, so the same model answers both "which segment is this customer in" and "what does recency look like for this slice". The At Risk KPIs deliberately ignore the segment slicer while still honouring date, region and category.

**HR Attrition and Headcount Dashboard** — Headcount, active employees, attrition count and rate, plus average income, tenure, age and time since last promotion. Two report pages: an executive overview and a demographics/attrition-driver breakdown.

**Modeling Data in Power BI** — A modeling exercise on the Northwind dataset: seven related tables, revenue computed at order-line grain (`unit_price × quantity × (1 − discount)`), and a decomposition tree over category and country. Source folder is driven by a single `NorthwindFolder` parameter.

**Predictive Sales Forecasting** — Time intelligence over a marked date table: prior-year sales via `SAMEPERIODLASTYEAR`, YoY in dollars and percent, MoM growth, and a target framework set at prior year + 10% with variance and attainment measures. Report includes Power BI's built-in forecast on the trend line, plus sales/profit by category, region and segment.

**Social Media Sentiment Analysis** — Post volume split positive/negative/neutral, a net sentiment score, a `RANKX`-based top-10 negative countries visual, and a "High Negative Risk" alert measure that flips when negative share exceeds 35%.

**Visualization of Life Expectancy and GDP Variation Over Time** — The Gapminder bubble chart, plus growth measures that respect the dataset's 5-year sampling interval (each "prior period" is resolved as the previous observed year, not the previous calendar year). Includes a population-weighted life expectancy measure alongside the naive unweighted average, and "latest year" snapshot measures so KPI cards don't sum population across all 12 sampled years.

## Repository layout

Each project folder follows the standard PBIP structure:

```
<Project Name>/
├── <Project Name>.pbip                        # open this in Power BI Desktop
├── <Project Name>.Report/
│   ├── definition.pbir
│   ├── definition/
│   │   ├── report.json
│   │   └── pages/<pageId>/
│   │       ├── page.json
│   │       └── visuals/<visualName>/visual.json
│   └── StaticResources/                       # base + custom themes
├── <Project Name>.SemanticModel/
│   ├── definition.pbism
│   └── definition/
│       ├── model.tmdl, database.tmdl, relationships.tmdl
│       ├── expressions.tmdl                   # source-folder parameter, where used
│       └── tables/*.tmdl                      # one file per table, measures in _Measures.tmdl
└── <source data>.csv
```

Conventions used throughout:

- **Measures live in a dedicated `_Measures` table** with a single hidden column, so they never hide inside a fact table and the field list has one obvious place to look.
- **Visuals are named**, not GUID'd (`kpiTotalRevenue`, `scatterLifeExpVsGdp`, `slicerRegion`), so a report diff is readable.
- **TMDL descriptions** (`///` comments) document non-obvious DAX in most models — why a measure removes a filter, why a snapshot date is fixed, why one segment rule has to be tested before another.
- All report pages are 1280×720.

## Opening a project

1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) and enable **Preview features → Power BI Project (.pbip) save option** (required on older builds; on current builds PBIP is GA).
2. Clone the repo and open `<Project Name>/<Project Name>.pbip`.
3. **Repoint the data source.** Every model references its CSV by absolute path from the machine it was built on (`C:\Users\Zenbook\Desktop\...`), so a fresh clone will fail to refresh until you update it:
   - *Customer Segmentation Analysis*, *Modeling Data in Power BI*, *Predictive Sales Forecasting*, *Visualization of Life Expectancy…* expose a single Power Query parameter (`SourceFolder` / `NorthwindFolder` / `GapminderFolder`). Change that one value in Transform data → Manage parameters, or edit `definition/expressions.tmdl` directly.
   - *Customer Lifetime Value Analysis*, *HR Attrition and Headcount Dashboard* and *Social Media Sentiment Analysis* hardcode the path inside the table partition — edit the `File.Contents(...)` line in the relevant `definition/tables/*.tmdl`.
4. Two projects ship their raw data as a zip (`Northwind Traders.zip`, `Gapminder World.zip`) — extract it before refreshing.

### Note on locale

All models carry `culture: en-US` but `sourceQueryCulture: tr-TR`, inherited from the authoring machine. If you add Power Query steps that convert text to numbers or dates, pin the culture explicitly (`Number.FromText(x, "en-US")`) — otherwise decimal separators are interpreted under Turkish conventions and values silently change.

## Data sources

| Dataset | Used by | Origin |
|---|---|---|
| Superstore | Customer Segmentation, Predictive Sales Forecasting | Tableau's Sample Superstore |
| IBM HR Analytics Employee Attrition | HR Attrition Dashboard | IBM / Kaggle |
| Northwind Traders | Modeling Data in Power BI | Classic Microsoft sample database |
| Gapminder | Life Expectancy & GDP | Gapminder Foundation |
| Sentiment140-style tweets | Social Media Sentiment Analysis | Kaggle |
| Synthetic transactions | Customer Lifetime Value Analysis | Generated sample |

All datasets are public samples or synthetic; none contain real personal data.

The largest raw file (`training.1600000.csv`, the full 1.6M-row sentiment corpus) exceeds GitHub's 100 MB limit and is excluded — see [Social Media Sentiment Analysis/.gitignore](Social%20Media%20Sentiment%20Analysis/.gitignore). The models load `train.csv` instead.

## License

No license file is present, so all rights are reserved by default. The underlying datasets remain under their own respective terms.
