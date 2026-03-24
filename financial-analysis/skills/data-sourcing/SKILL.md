## Trigger
Whenever a financial metric (revenue, R&D, market cap, price) is needed.

## Steps
1. Identify tickers and fiscal period required.
2. For fundamentals: call SEC EDGAR MCP first → extract from latest 10-K or 10-Q.
3. For market data: call Yahoo Finance / Alpha Vantage MCP → record "as of" date.
4. NEVER hard-code or estimate values. If unavailable, output N/A with reason.
5. Always record: source name, filing type, period end date, extraction timestamp.
