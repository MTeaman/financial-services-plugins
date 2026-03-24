# Data Sourcing Skill

## Purpose
Ensure every financial metric used in analysis, modeling, or output is sourced from a verified, real data provider — never estimated, fabricated, or described as "representative." This skill fires automatically whenever financial figures (revenue, R&D spend, market cap, cash, price, ratios, credit metrics) are needed.

## Trigger Conditions
Activate this skill when any of the following are required:
- Stock price or market capitalisation
- Revenue, R&D expense, operating income, net income
- Cash, cash equivalents, or short-term investments
- Balance sheet, income statement, or cash flow data
- Credit ratings, debt metrics, or bond yields
- Funding rounds, deal data, or private company information
- Macro indicators (interest rates, FX, yield curves)
- News, earnings releases, or event transcripts

## Data Source Hierarchy
Always follow this priority order. Move to the next source only if the preferred one is unavailable.

### 1. Fundamentals (Revenue, R&D, Cash, Balance Sheet)
**Primary:** SEC EDGAR MCP (`sec-edgar`)
- Use for all US-listed companies (10-K annual, 10-Q quarterly)
- Extract XBRL-tagged values only — exact figures, no rounding
- Always record: CIK, accession number, filing date, fiscal period end
- Required fields per metric: value, unit (USD thousands/millions), period type (annual/quarterly/TTM)

**Secondary:** Financial Modeling Prep MCP (`fmp`)
- Use when EDGAR data is not yet available (e.g. very recent filing lag)
- Free tier: 250 requests/day — use efficiently
- Always record: endpoint used, fiscal period, data extraction timestamp

### 2. Market Data (Price, Market Cap, Enterprise Value)
**Primary:** Yahoo Finance MCP (`yahoo-finance`)
- Fetch real-time or end-of-day quote
- Always record: ticker, exchange, price as-of date and time (UTC)
- Market cap = shares outstanding x price (verify both components)

**Secondary:** Alpha Vantage MCP (`alpha-vantage`)
- Use for historical price series or if Yahoo Finance is unavailable
- Free tier: 25 requests/day — prioritise carefully
- Always record: API endpoint, adjusted/unadjusted flag, as-of date

### 3. Ratings, Credit & Macro Data
**Primary:** FRED MCP (`frb-fred`)
- Use for yield curves, credit spreads, FX rates, macro indicators
- All data is official Federal Reserve / US government data — fully citable
- Always record: FRED series ID, observation date, units

**Note:** Private credit ratings (Moody's, S&P, Fitch) are not freely available. Use FRED credit spread proxies and note this limitation explicitly in output.

### 4. News & Earnings Releases
**Primary:** Brave Search MCP (`brave-search`)
- Use for recent news, press releases, earnings announcements
- Always record: headline, source domain, publication date

**Secondary:** SEC EDGAR 8-K filings (`sec-edgar`)
- Earnings press releases are filed as 8-K — use for official earnings data
- More reliable than news sources for exact reported figures

### 5. Private Company & Deal Data
**Primary:** Crunchbase MCP (`crunchbase-free`)
- Use for VC funding rounds, investors, board data
- Free tier: 200 records/day
- Always record: data source, last updated date from Crunchbase
- Note: valuations may be delayed 45-60 days

### 6. Ratios, Screeners & Portfolio Analytics
**Primary:** OpenBB MCP (`openbb`)
- Use for calculated ratios (P/E, P/S, EV/EBITDA), peer screeners, ETF holdings
- Always record: OpenBB endpoint, underlying data source reported by OpenBB, as-of date

## Mandatory Data Quality Rules

### Never Do
- NEVER hard-code, estimate, or assume any financial figure
- NEVER use phrases like "representative of recent trends" or "approximate"
- NEVER round figures from SEC filings — preserve exact XBRL precision
- NEVER use data from a prior model run without re-fetching and re-verifying
- NEVER proceed if a required metric is unavailable — output N/A with explanation

### Always Do
- ALWAYS call the relevant MCP tool BEFORE building any table, model, or output
- ALWAYS populate the Data Sources sheet (see below) for every numeric value used
- ALWAYS flag if any data point is older than 12 months from today's date
- ALWAYS note the fiscal period type: FY (full year), Q (quarter), TTM (trailing twelve months)
- ALWAYS convert units consistently: state whether figures are in USD thousands, millions, or billions

## Data Sources Sheet — Required Output
Every Excel or structured output produced using this skill MUST include a "Data Sources" tab/section with the following columns:

| Column | Description |
|---|---|
| Ticker | Stock ticker or company identifier |
| Metric | e.g. Revenue, R&D Expense, Market Cap, Cash |
| Value | Exact numeric value as retrieved |
| Unit | USD thousands / USD millions / USD billions |
| Period Type | FY / Q1 / Q2 / Q3 / Q4 / TTM |
| Fiscal Period End | Date (YYYY-MM-DD) |
| Source Name | e.g. SEC EDGAR 10-K, Yahoo Finance, FRED |
| Source ID | e.g. SEC accession number, FRED series ID, URL |
| Extracted On | Date and time this data was retrieved (UTC) |
| Notes | Any caveats, restatements, or unit conversions |

## Validation Checklist
Before finalising any output, verify:
- [ ] All metrics have a corresponding Data Sources entry
- [ ] No cell in the model contains a hard-coded estimate
- [ ] Fiscal periods are consistent across all companies in a comparison
- [ ] Market data as-of date is clearly stated in the model header
- [ ] Any missing data is marked N/A with a written explanation
- [ ] Unit conventions (thousands vs millions) are consistent throughout

## Error Handling
If an MCP call fails or returns no data:
1. Try the secondary source listed in the hierarchy above
2. If both fail, mark the metric as `N/A — [source name] unavailable on [date]`
3. Do NOT substitute with an estimate or a value from a prior session
4. Log the failure in the Data Sources sheet under the Notes column
5. Notify the user which metrics could not be sourced before proceeding

## Limitations to Declare in Every Output
Include the following standard disclaimer in any financial model or report produced:

> **Data Sources Note:** Fundamental data sourced from SEC EDGAR filings. Market data sourced from Yahoo Finance / Alpha Vantage. Macro data from FRED (Federal Reserve). Private company data from Crunchbase. All figures reflect the most recent available filing or quote as of the date shown. Private credit ratings are not included — FRED credit spread proxies used where applicable. This analysis does not constitute financial or investment advice.
