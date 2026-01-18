# Tools Reference

Complete documentation for all MCP tools available in the Yahoo Finance MCP Server.

## Table of Contents

- [Market Data](#market-data)
- [Company Intelligence](#company-intelligence)
- [Market Sentiment](#market-sentiment)
- [Cross-Asset](#cross-asset)
- [Pre-Built Prompts](#pre-built-prompts)

---

## Market Data

### get_quote

Real-time stock quotes with quality reporting.

**Location:** [`src/tools/quotes.ts`](../src/tools/quotes.ts)

**Parameters:**
```typescript
{
  symbols: string[],  // 1-100 symbols
  forceRefresh?: boolean  // Default: false
}
```

**Returns:**
```typescript
{
  data: Quote[],
  meta: {
    fromCache: boolean,
    dataAge: number,
    completenessScore: number,
    warnings: string[]
  }
}
```

**Features:**
- Batch up to 100 symbols per request
- 60-second cache TTL
- Force refresh option for real-time data
- Data quality scoring (completeness, integrity)

**Example:**
```json
{
  "symbols": ["AAPL", "MSFT", "GOOGL"],
  "forceRefresh": false
}
```

---

### get_historical_prices

Historical OHLCV data with customizable date ranges and intervals.

**Location:** [`src/tools/historical.ts`](../src/tools/historical.ts)

**Parameters:**
```typescript
{
  symbol: string,
  startDate: string,  // ISO date format
  endDate: string,  // ISO date format
  interval?: '1d' | '1wk' | '1mo'  // Default: '1d'
}
```

**Returns:**
```typescript
{
  data: HistoricalPrice[],
  meta: {
    gaps: number,
    splits: number,
    integrityChecks: IntegrityResult[]
  }
}
```

**Features:**
- Customizable date ranges
- Intervals: daily (1d), weekly (1wk), monthly (1mo)
- Gap detection
- Split detection
- Data integrity validation

**Example:**
```json
{
  "symbol": "AAPL",
  "startDate": "2023-01-01",
  "endDate": "2023-12-31",
  "interval": "1d"
}
```

---

### get_historical_prices_multi

Batch historical data for multiple symbols.

**Location:** [`src/tools/historical.ts`](../src/tools/historical.ts)

**Parameters:**
```typescript
{
  symbols: string[],  // Up to 50 symbols
  startDate: string,
  endDate: string,
  interval?: '1d' | '1wk' | '1mo'
}
```

**Returns:** Array of historical price results per symbol

**Features:**
- Up to 50 symbols per batch
- Integrity validation for each symbol
- Parallel processing for faster results

---

## Company Intelligence

### get_quote_summary

Comprehensive company overview with fallback for missing data.

**Location:** [`src/tools/summary.ts`](../src/tools/summary.ts)

**Parameters:**
```typescript
{
  symbol: string,
  modules?: string[]  // Optional: specific modules
}
```

**Returns:**
```typescript
{
  data: {
    assetProfile: AssetProfile,
    summaryDetail: SummaryDetail,
    defaultKeyStatistics: KeyStatistics
  }
}
```

**Features:**
- Company profile (sector, industry, business summary)
- Financial metrics (market cap, P/E, EPS, etc.)
- Auto-classification for missing sector/industry
- Fallback to default values

**Example:**
```json
{
  "symbol": "AAPL"
}
```

---

### get_balance_sheet

Balance sheet financial statements.

**Location:** [`src/tools/financials.ts`](../src/tools/financials.ts)

**Parameters:**
```typescript
{
  symbol: string,
  frequency?: 'annual' | 'quarterly'  // Default: 'quarterly'
}
```

**Returns:**
```typescript
{
  data: {
    balanceSheetStatements: BalanceSheet[]
  }
}
```

**Features:**
- Assets, liabilities, equity breakdown
- Annual and quarterly reports
- Data validation
- Note: May encounter validation errors for some symbols (see [Data Verification](DATA_VERIFICATION.md))

---

### get_income_statement

Income statement financial statements.

**Location:** [`src/tools/financials.ts`](../src/tools/financials.ts)

**Parameters:**
```typescript
{
  symbol: string,
  frequency?: 'annual' | 'quarterly'
}
```

**Returns:**
```typescript
{
  data: {
    incomeStatementHistory: IncomeStatement[]
  }
}
```

**Features:**
- Revenue, expenses, net income
- EPS (basic and diluted)
- Operating margins, profit margins
- Annual and quarterly reports

---

### get_cash_flow_statement

Cash flow financial statements.

**Location:** [`src/tools/financials.ts`](../src/tools/financials.ts)

**Parameters:**
```typescript
{
  symbol: string,
  frequency?: 'annual' | 'quarterly'
}
```

**Returns:**
```typescript
{
  data: {
    cashflowStatementHistory: CashFlow[]
  }
}
```

**Features:**
- Operating, investing, financing cash flows
- Capital expenditures
- Free cash flow
- Annual and quarterly reports

---

### get_earnings

Quarterly earnings with estimates and surprises.

**Location:** [`src/tools/earnings.ts`](../src/tools/earnings.ts)

**Parameters:**
```typescript
{
  symbol: string
}
```

**Returns:**
```typescript
{
  data: {
    quarterly: EarningsQuarterly[],
    estimate: EarningsEstimate
  }
}
```

**Features:**
- Historical earnings data
- Actual vs estimate comparison
- Surprise analysis (beat/miss, percentage)
- Next earnings date and estimate
- Earnings trends

---

### get_analysis

Analyst recommendations and price targets.

**Location:** [`src/tools/analysis.ts`](../src/tools/analysis.ts)

**Parameters:**
```typescript
{
  symbol: string
}
```

**Returns:**
```typescript
{
  data: {
    recommendationTrend: RecommendationTrend[],
    targetPriceInfo: TargetPriceInfo
  }
}
```

**Features:**
- Current ratings (Strong Buy, Buy, Hold, Sell, Strong Sell)
- Overall analyst recommendation
- Target prices (high, low, mean, median)
- Historical recommendation trends
- Earnings estimate trends

---

### get_major_holders

Institutional and insider ownership information.

**Location:** [`src/tools/holders.ts`](../src/tools/holders.ts)

**Parameters:**
```typescript
{
  symbol: string,
  holdersType?: 'institutional' | 'fund' | 'insider' | 'direct'
}
```

**Returns:**
```typescript
{
  data: {
    holders: Holder[],
    breakdown: HoldersBreakdown
  }
}
```

**Features:**
- Institutional ownership breakdown
- Mutual fund holdings
- Insider transactions (buying/selling)
- Direct ownership information
- Change history tracking

---

## Market Sentiment

### get_news

Latest company news with relevance scoring.

**Location:** [`src/tools/news.ts`](../src/tools/news.ts)

**Parameters:**
```typescript
{
  symbol: string,
  count?: number,  // Default: 10
  relatedTickersOnly?: boolean  // Default: false
}
```

**Returns:**
```typescript
{
  data: {
    news: NewsItem[]
  }
}
```

**Features:**
- Recent articles with titles and content
- Publisher information
- Publication date
- URL validation
- Related tickers mentioned
- Relevance filtering option

---

### get_options

Options chains with Greeks calculations.

**Location:** [`src/tools/options.ts`](../src/tools/options.ts)

**Parameters:**
```typescript
{
  symbol: string,
  expiration?: string,  // Specific expiration date
  strikeFilter?: 'ITM' | 'OTM' | 'ALL'  // Default: 'ALL'
}
```

**Returns:**
```typescript
{
  data: {
    options: OptionContract[],
    expirations: string[]
  }
}
```

**Features:**
- Calls and puts with strike prices
- Greeks calculations (delta, gamma, theta, vega, rho)
- Multiple expiration dates
- Strike filtering (ITM/OTM)
- Implied volatility, last price, bid/ask

---

### get_trending_symbols

Top trending stocks with volume metrics.

**Location:** [`src/tools/market.ts`](../src/tools/market.ts)

**Parameters:**
```typescript
{
  region?: 'US' | 'EU' | 'ASIA'  // Default: 'US'
  count?: number  // Default: 10
}
```

**Returns:**
```typescript
{
  data: {
    trending: TrendingStock[]
  }
}
```

**Features:**
- Top trending stocks
- Regional filtering (US, EU, ASIA)
- Volume metrics
- Engagement metrics
- Price change percentages

---

### screener

Filter stocks by multiple criteria.

**Location:** [`src/tools/market.ts`](../src/tools/market.ts)

**Parameters:**
```typescript
{
  filters: {
    sector?: string,
    marketCap?: { min?: number, max?: number },
    peRatio?: { min?: number, max?: number },
    dividendYield?: { min?: number },
    beta?: { min?: number, max?: number },
    region?: 'US' | 'EU' | 'ASIA'
  },
  count?: number  // Default: 20
}
```

**Returns:**
```typescript
{
  data: {
    stocks: StockQuote[]
  }
}
```

**Features:**
- 12+ filter criteria
- Sector filtering
- Market cap ranges
- P/E ratio ranges
- Dividend yield filtering
- Beta filtering
- Regional filtering

**Example:**
```json
{
  "filters": {
    "sector": "Technology",
    "marketCap": { "min": 1000000000 },
    "peRatio": { "max": 30 }
  },
  "count": 20
}
```

---

## Cross-Asset

### get_crypto_quote

Cryptocurrency prices.

**Location:** [`src/tools/quotes.ts`](../src/tools/quotes.ts)

**Parameters:**
```typescript
{
  symbols: string[]  // e.g., ["BTC-USD", "ETH-USD", "SOL-USD"]
}
```

**Returns:** Quote data (see [Data Verification](DATA_VERIFICATION.md))

**Status:** ⚠️ Returns placeholder data. See [Data Verification](DATA_VERIFICATION.md) for details and workarounds.

---

### get_forex_quote

Currency pair exchange rates.

**Location:** [`src/tools/quotes.ts`](../src/tools/quotes.ts)

**Parameters:**
```typescript
{
  pairs: string[]  // e.g., ["EURUSD", "GBPUSD", "JPYUSD"]
}
```

**Returns:** Forex data (see [Data Verification](DATA_VERIFICATION.md))

**Status:** ⚠️ Returns placeholder data. See [Data Verification](DATA_VERIFICATION.md) for details and workarounds.

---

## Pre-Built Prompts

The following pre-built prompts are available for common financial analysis tasks:

- **analyze_stock** - Comprehensive company analysis with financials, earnings, and recommendations
- **compare_stocks** - Multi-stock comparison across valuation, performance, and risk metrics
- **financial_health_check** - Liquidity, solvency, profitability, efficiency ratios
- **earnings_analysis** - Earnings trends, surprises, and quality assessment
- **market_overview** - Regional market sentiment, trending stocks, sector performance
- **portfolio_due_diligence** - Multi-stock risk assessment and diversification analysis

**Location:** [`src/prompts/`](../src/prompts/)

For more information on using prompts, see [USAGE_GUIDE.md](USAGE_GUIDE.md).
