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

- ### 7. Multi-Asset Class Market Data

#### 7a. FX Rates (Foreign Exchange)

**Primary:** ECB SDW MCP (`ecb-sdw`) — for EUR-based currency pairs

- Use for official EUR exchange rates (ECB reference rates)
- All major currencies vs EUR: USD, GBP, JPY, CHF, etc.
- Daily fixings published by European Central Bank
- Always record: currency pair, exchange rate, reference date
- API endpoint: https://sdw-wsrest.ecb.europa.eu/service/data/EXR/
- Example: EUR/USD → series key `D.USD.EUR.SP00.A`

**Secondary:** FRED MCP (`frb-fred`) — for USD-based currency pairs

- Use for USD exchange rates when EUR rates not suitable
- Major series: DEXUSEU (USD/EUR), DEXJPUS (JPY/USD), DEXUSUK (USD/GBP)
- Daily data from Federal Reserve
- Always record: FRED series ID, observation date

**Tertiary:** Alpha Vantage MCP (`alpha-vantage`) — for real-time FX

- Use FX_DAILY endpoint for historical rates
- Free tier: 25 API calls/day — use sparingly
- Covers 150+ currency pairs
- Always record: from_currency, to_currency, exchange rate, timestamp

**Note on European Company Filings:**
- For EU company quarterly/annual reports, use **ESMA ESEF** (`esma-esef` already in .mcp.json)
- Covers EU-listed companies filing in iXBRL format (similar to SEC EDGAR for US)
- Access via https://filings.esma.europa.eu/ and FIRDS register

#### 7b. Commodities

**Primary:** FRED MCP (`frb-fred`) — most comprehensive free source

- **Gold:** Series GOLDAMGBD228NLBM (London Bullion Market, USD/troy oz)
- **Crude Oil (WTI):** Series DCOILWTICO ($/barrel)
- **Crude Oil (Brent):** Series DCOILBRENTEU ($/barrel)
- **Natural Gas:** Series DHHNGSP (Henry Hub, $/MMBtu)
- **Copper:** Series PCOPPUSDM ($/metric ton)
- **Silver:** Series SLVPRUSD ($/troy oz)
- Always record: commodity name, FRED series ID, price, unit, observation date
- All data is daily, from official commodity exchanges

**Secondary:** Alpha Vantage MCP (`alpha-vantage`) — for additional commodities

- Use COMMODITY endpoint (e.g., WTI, BRENT, NATURAL_GAS)
- Free tier: 25 API calls/day
- Monthly and annual intervals available
- Always record: commodity symbol, interval, price, timestamp

**Tertiary:** Yahoo Finance MCP (`yahoo-finance`) — for futures quotes

- Real-time commodity futures prices
- Tickers: GC=F (gold futures), CL=F (crude oil futures), SI=F (silver futures)
- Use for current market prices (end-of-day or delayed 15-20 min)
- Always record: ticker symbol, last price, as-of time (UTC)

#### 7c. Bonds & Fixed Income

**Primary:** FRED MCP (`frb-fred`) — US Treasury yields & corporate bonds

- **US Treasury Yields:**
  - 1-Month: DGS1MO
  - 3-Month: DGS3MO
  - 6-Month: DGS6MO
  - 1-Year: DGS1
  - 2-Year: DGS2
  - 5-Year: DGS5
  - 10-Year: DGS10
  - 30-Year: DGS30
- **Corporate Bond Indices:**
  - ICE BofA US High Yield Index Effective Yield: BAMLH0A0HYM2EY
  - Moody's Seasoned AAA Corporate Bond Yield: DAAA
  - Moody's Seasoned BAA Corporate Bond Yield: DBAA
- **TIPS (inflation-protected):** DFII10 (10-year)
- All yields are percentages (e.g., 4.25 = 4.25%)
- Always record: maturity, FRED series ID, yield, observation date

**Secondary:** ECB SDW MCP (`ecb-sdw`) — European government bonds

- German Bunds, French OATs, Italian BTPs, Spanish Bonos
- Series key format varies by country and maturity
- Example: 10-year German Bund yield via ECB Statistical Data Warehouse
- Always record: country, maturity, yield, reference date

**Note:** Individual corporate bond prices (CUSIP-level) are not freely available. Use FRED credit spread indices as proxies.

#### 7d. Interest Rates

**Primary:** FRED MCP (`frb-fred`) — US interest rate benchmarks

- **Federal Funds Rate:** DFEDTARU (target upper bound), DFEDTARL (target lower bound)
- **SOFR (Secured Overnight Financing Rate):** SOFR
- **3-Month SOFR:** SOFR3M
- **Prime Rate:** DPRIME
- **Historical LIBOR (discontinued Dec 2021):** USD3MTD156N (3-month USD LIBOR)
- Always record: rate type, FRED series ID, rate (in %), observation date

**Secondary:** ECB SDW MCP (`ecb-sdw`) — European interest rate benchmarks

- **ECB Main Refinancing Rate:** Official ECB policy rate
- **€STR (€uro Short-Term Rate):** Euro overnight rate (replaced EONIA)
- **EURIBOR:** 1-week, 1-month, 3-month, 6-month, 12-month
- Always record: rate type, tenor, rate (in %), reference date

#### 7e. Credit Risk Proxies (CDS Alternatives)

**Primary:** FRED MCP (`frb-fred`) — credit spread indices

- **Note:** Actual CDS (Credit Default Swap) spreads are proprietary data from Bloomberg/Markit/ICE
- **Use credit spreads as proxies for credit risk:**
  - ICE BofA US High Yield Spread: BAMLH0A0HYM2
  - ICE BofA BBB US Corporate Spread: BAMLC0A4CBBB
  - ICE BofA AAA US Corporate Spread: BAMLC0A1CAAA
  - Corporate Bond Spread (BAA-AAA): Calculated as DBAA minus DAAA
- **Interpretation:** Wider spreads = higher perceived credit risk
- Always record: index name, spread (in basis points), observation date
- **Always disclose:** "CDS data not available — using credit spread indices as proxy"

**Limitation:** These are market-wide indices, not entity-specific CDS spreads. For specific company credit risk, note this limitation explicitly in output.

#### 7f. Real Estate

**Primary:** FRED MCP (`frb-fred`) — US real estate price indices

- **S&P/Case-Shiller US National Home Price Index:** CSUSHPISA
- **S&P/Case-Shiller 20-City Composite:** SPCS20RSA
- **FHFA House Price Index (All-Transactions):** USSTHPI
- **Commercial Real Estate Price Index:** CPGREUSQ156N (NCREIF)
- Always record: index name, FRED series ID, index value, observation date
- **REIT Indices (publicly traded):**
  - MSCI US REIT Index: Check if available via FRED or OpenBB
  - Nareit All Equity REITs Index: Public data from Nareit.com

**Secondary:** REIT Data Market API — individual REIT property data

- Free tier: 100+ REITs, 280,000+ properties
- Property attributes: lat/long, address, size, occupancy
- API documentation: https://reitdatamarket.com
- Use for granular REIT portfolio analysis
- Always record: REIT ticker, property count, data source, as-of date

**Limitation:** Individual property valuations (Zillow, Redfin, ATTOM) require paid APIs. Use FRED indices for market-level analysis.

#### 7g. Derivatives (Options & Futures on Indices)

**Primary:** Yahoo Finance MCP (`yahoo-finance`) — index futures (limited)

- **Futures tickers:**
  - S&P 500 E-mini: ES=F
  - NASDAQ 100 E-mini: NQ=F
  - Dow Jones E-mini: YM=F
- Use for current futures prices (delayed 15-20 min)
- Always record: ticker, contract month, last price, as-of time

**Secondary:** Alpha Vantage MCP (`alpha-vantage`) — very limited options data

- Free tier: Only 5 API calls/day for OPTIONS endpoint
- Covers major US stocks and ETFs (SPY, QQQ)
- **Not suitable for comprehensive derivatives analysis**
- Always record: symbol, expiration, strike, option type (call/put), last price

**Major Limitation:**
- **No free comprehensive source for:**
  - Options chain data (Greeks, implied volatility, open interest)
  - Index options (SPX, NDX, VIX)
  - Futures options
  - OTC derivatives
- **Recommendation:** For serious derivatives analysis, note this data gap explicitly
- **Workaround:** Use Yahoo Finance web scraping for basic options chains (not via MCP — requires custom HTTP Request in N8N)

**Always Disclose:**
> "Comprehensive derivatives data (options Greeks, implied volatility surfaces, futures term structures) is not available via free APIs. Analysis limited to spot futures prices. For detailed options analysis, Bloomberg Terminal or paid data feeds required."



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
