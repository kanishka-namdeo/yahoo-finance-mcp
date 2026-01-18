# Data Verification

Comprehensive verification of data availability, quality, and known limitations.

## Table of Contents

- [Working Features](#working-features)
- [Partial Support](#partial-support)
- [Known Limitations](#known-limitations)
- [Testing Status](#testing-status)
- [API Dependencies](#api-dependencies)

---

## Working Features

The following data points and tools have been verified to work correctly through real-world testing.

### Real-Time Market Data ✅

**Tool:** [`get_quote`](TOOLS.md#get_quote)

**Available Data:**
- Stock quotes (price, change, change %, volume, market cap)
- Pre-market/after-hours prices with changes
- 52-week range (high/low)
- Key ratios (P/E trailing/forward, EPS trailing/forward, beta)
- Dividends (rate, yield, ex-dividend date)
- Trading info (open, day high/low, previous close, average volume)
- Batch processing (up to 100 symbols per request)

**Verification Status:** ✅ Fully working
- All major fields verified through real-world tests
- Data quality scoring operational
- Cache fallback working
- Batch requests functioning correctly

**Example Response:**
```json
{
  "data": [
    {
      "symbol": "AAPL",
      "price": 178.72,
      "change": 2.35,
      "changePercent": 1.33,
      "volume": 52345678,
      "marketCap": 2800000000000
    }
  ],
  "meta": {
    "fromCache": false,
    "dataAge": 150,
    "completenessScore": 95,
    "warnings": []
  }
}
```

---

### Historical Price Data ✅

**Tool:** [`get_historical_prices`](TOOLS.md#get_historical_prices)

**Available Data:**
- OHLCV data (Open, High, Low, Close, Adjusted Close, Volume)
- Customizable date ranges
- Intervals: 1d (daily), 1wk (weekly), 1mo (monthly)
- Data integrity validation (high ≥ low, close within range)
- Gap detection (missing data points)
- Split detection (stock splits based on price ratios)

**Verification Status:** ✅ Fully working
- OHLCV data verified for multiple symbols
- Integrity validation operational
- Gap detection working
- Split detection functioning

**Example Response:**
```json
{
  "data": [
    {
      "date": "2024-01-15",
      "open": 185.00,
      "high": 187.50,
      "low": 184.25,
      "close": 186.75,
      "adjClose": 186.75,
      "volume": 52345678
    }
  ],
  "meta": {
    "gaps": 0,
    "splits": 0,
    "integrityChecks": [
      {
        "type": "VALID",
        "message": "Price data is consistent"
      }
    ]
  }
}
```

---

### Company Profile ✅

**Tool:** [`get_quote_summary`](TOOLS.md#get_quote_summary)

**Available Data:**
- Business information (company name, sector, industry, long business summary)
- Contact details (city, country, website)
- Employee count (full-time employees)
- Fallback classification for missing sector/industry data

**Verification Status:** ✅ Fully working
- Company profile data verified
- Fallback classification operational
- Missing data handled gracefully

**Example Response:**
```json
{
  "data": {
    "assetProfile": {
      "companyName": "Apple Inc.",
      "sector": "Technology",
      "industry": "Consumer Electronics",
      "fullTimeEmployees": 164000
    }
  }
}
```

---

### Earnings Data ✅

**Tool:** [`get_earnings`](TOOLS.md#get_earnings)

**Available Data:**
- Quarterly earnings with actual vs estimate
- Surprise analysis (beat/miss, percentage, direction)
- Timing information (before/after/during market hours)
- Current quarter estimates with dates
- Earnings trends over time
- Next earnings date and estimate

**Verification Status:** ✅ Fully working
- Historical earnings data verified
- Surprise analysis working
- Estimates data operational
- Trends tracking functional

**Example Response:**
```json
{
  "data": {
    "quarterly": [
      {
        "date": "2024-01-25",
        "actual": 2.18,
        "estimate": 2.10,
        "surprise": 0.08,
        "surprisePercent": 3.81
      }
    ]
  }
}
```

---

### Analyst Analysis ✅

**Tool:** [`get_analysis`](TOOLS.md#get_analysis)

**Available Data:**
- Current ratings (Strong Buy, Buy, Hold, Sell, Strong Sell counts)
- Overall analyst recommendation
- Target prices (high, low, mean, median)
- Historical recommendation trends
- Earnings estimate trends

**Verification Status:** ✅ Fully working
- Analyst ratings verified
- Target prices operational
- Recommendation trends working

**Example Response:**
```json
{
  "data": {
    "recommendationTrend": [
      {
        "period": "0m",
        "strongBuy": 28,
        "buy": 15,
        "hold": 8,
        "sell": 1,
        "strongSell": 0
      }
    ],
    "targetPriceInfo": {
      "targetHigh": 220.00,
      "targetLow": 150.00,
      "targetMean": 185.00,
      "medianTarget": 180.00
    }
  }
}
```

---

### Company News ✅

**Tool:** [`get_news`](TOOLS.md#get_news)

**Available Data:**
- Articles with titles and content
- Publisher information (source/publisher name)
- Publication date (timestamp)
- URL validation (checks if links are valid)
- Related tickers (other symbols mentioned)
- Relevance filtering option

**Verification Status:** ✅ Fully working
- News articles verified
- Metadata working correctly
- URL validation operational

**Example Response:**
```json
{
  "data": {
    "news": [
      {
        "title": "Apple Reports Strong Q4 Earnings",
        "publisher": "Reuters",
        "publishDate": 1705315200000,
        "url": "https://example.com/article"
      }
    ]
  }
}
```

---

### Options Data ✅

**Tool:** [`get_options`](TOOLS.md#get_options)

**Available Data:**
- Options chain (calls and puts with strike prices)
- Greeks calculations (delta, gamma, theta, vega, rho)
- Multiple expiration dates available
- Strike filtering (ITM/OTM)
- Metadata (implied volatility, last price, bid/ask)

**Verification Status:** ✅ Fully working
- Options chains verified
- Greeks calculations accurate
- Strike filtering operational

**Example Response:**
```json
{
  "data": {
    "options": [
      {
        "strike": 180.00,
        "call": {
          "delta": 0.55,
          "gamma": 0.02,
          "theta": -0.05,
          "vega": 0.15
        }
      }
    ]
  }
}
```

---

### Major Holders ✅

**Tool:** [`get_major_holders`](TOOLS.md#get_major_holders)

**Available Data:**
- Institutional ownership breakdown
- Mutual fund holdings
- Insider transactions (buying/selling)
- Direct ownership information

**Verification Status:** ✅ Fully working
- Institutional holder data verified
- Fund holder data operational
- Insider transactions working

**Example Response:**
```json
{
  "data": {
    "holders": [
      {
        "holder": "Vanguard Group Inc",
        "shares": 1234567890,
        "percentage": 7.5
      }
    ]
  }
}
```

---

### Market Intelligence ✅

**Tools:** [`get_trending_symbols`](TOOLS.md#get_trending_symbols), [`screener`](TOOLS.md#screener)

**Available Data:**
- Trending symbols with volume metrics
- Regional filtering (US, EU, ASIA)
- Stock screener (12+ filter criteria: sector, market cap, P/E, dividend yield, beta, etc.)
- Trading volume and engagement metrics

**Verification Status:** ✅ Fully working
- Trending data verified
- Screener functional
- Multiple filter criteria working

**Example Response:**
```json
{
  "data": {
    "trending": [
      {
        "symbol": "AAPL",
        "volume": 52345678,
        "changePercent": 2.35
      }
    ]
  }
}
```

---

## Partial Support

### Financial Statements ⚠️

**Tools:** [`get_balance_sheet`](TOOLS.md#get_balance_sheet), [`get_income_statement`](TOOLS.md#get_income_statement), [`get_cash_flow_statement`](TOOLS.md#get_cash_flow_statement)

**What Works:**
- Balance sheet (assets, liabilities, equity, cash, debt)
- Income statement (revenue, expenses, net income, EPS)
- Cash flow statement (operating, investing, financing, capex)
- Annual and quarterly reports

**Known Issues:**
- Some symbols fail validation with `TYPE="UNKNOWN"` errors
- Affected stocks:
  - Smaller cap stocks with limited financial data
  - International stocks with different reporting standards
  - Recently listed companies

**Root Cause:**
The [yahoo-finance2](https://github.com/gadicc/yahoo-finance2) library's `fundamentalsTimeSeries` API has strict schema validation that rejects responses with unexpected TYPE values. Yahoo Finance's API occasionally returns data with `TYPE="UNKNOWN"` which doesn't match expected BALANCE_SHEET, INCOME_STATEMENT, or CASH_FLOW constants.

**Workarounds:**
1. Use [`get_quote_summary`](TOOLS.md#get_quote_summary) which uses the more reliable `quoteSummary` module
2. Retry the request (circuit breaker handles automatic retries)
3. Use `frequency: 'annual'` instead of `frequency: 'quarterly'`
4. Check if data is available in cached results

**Verification Status:** ⚠️ Partially working
- Works for most major stocks (AAPL, MSFT, GOOGL, etc.)
- May fail for smaller cap or international stocks
- Cache fallback helps when API fails

---

### Crypto & Forex Data ⚠️

**Tools:** [`get_crypto_quote`](TOOLS.md#get_crypto_quote), [`get_forex_quote`](TOOLS.md#get_forex_quote)

**What's Available:**
- Cryptocurrency symbols (BTC-USD, ETH-USD, SOL-USD, and 50+ other pairs)
- Forex symbols (EURUSD, GBPUSD, JPYUSD, and major currency pairs)

**Known Issues:**
- Tools exist but return placeholder data
- Crypto quotes return zeros (price: 0, volume: 0, etc.)
- Forex quotes return 1.0 (exchange rate always 1.0)

**Root Cause:**
The yahoo-finance2 library's `quote()` method, which is used by these tools, may not fully support cryptocurrency and forex data retrieval. The library's `chart()` method or Yahoo Finance's direct API endpoints may be required for accurate crypto/forex data.

**Workarounds:**
For cryptocurrency and forex exposure:
1. Use [`screener`](TOOLS.md#screener) with appropriate filters
   - Example: `sector="Technology"` for crypto-related stocks
2. Use [`get_trending_symbols`](TOOLS.md#get_trending_symbols) to identify actively traded assets
3. Look for crypto/forex ETFs and use standard stock quote tools

**Recommendation:**
For dedicated crypto/forex data, consider integrating additional APIs:
- **Crypto APIs:** CoinGecko, CoinMarketCap, Binance
- **Forex APIs:** OANDA, Fixer.io, ExchangeRate-API

**Verification Status:** ⚠️ Tools exist, data unavailable
- Tool infrastructure is in place
- API calls succeed but return placeholder data
- Requires alternative data sources

---

## Known Limitations

### Rate Limiting ⚠️

**Issue:**
Yahoo Finance has rate limits that can cause 429 errors when making too many requests.

**Symptoms:**
- 429 (Too Many Requests) errors
- Circuit breaker opening frequently
- Increased latency

**Mitigation:**
The MCP server implements:
- Token bucket rate limiting (configurable, default: 60 requests/minute)
- Exponential backoff with jitter for retries
- Circuit breaker to prevent cascading failures
- Cache fallback when APIs are rate-limited

**Best Practices:**
- Use batch requests ([`get_quote`](TOOLS.md#get_quote) with multiple symbols) instead of individual calls
- Increase cache TTL for less time-sensitive data
- Respect configured rate limit settings
- Monitor circuit breaker state via stats endpoint

**Impact:** ✅ Well-managed
- Rate limiting is working as expected
- Cache fallback provides data during rate limits
- Circuit breaker prevents cascading failures

---

### Data Freshness ⚠️

**Issue:**
Some data may be delayed or stale, especially for international markets or during market hours.

**Indicators:**
Every response includes quality metadata:
- `fromCache`: Boolean indicating if data came from cache
- `dataAge`: Milliseconds since data was fetched
- `completenessScore`: 0-100 score indicating data completeness
- `warnings`: Array of warning messages about data quality

**Recommendations:**
- Use `forceRefresh: true` when fresh data is critical
- Monitor `completenessScore` and `warnings` in responses
- Consider cached data acceptable for non-critical analysis (e.g., screening)

**Impact:** ✅ Transparent and manageable
- Data freshness is clearly indicated in responses
- Users can make informed decisions about data reliability
- Cache TTL can be configured based on needs

---

### API Reliability ⚠️

**Issue:**
Yahoo Finance API is unofficial and may experience downtime or changes.

**Symptoms:**
- Intermittent API failures
- Schema validation errors
- Breaking API changes

**Mitigation:**
The MCP server implements:
- Circuit breaker to prevent cascading failures
- Retry logic with exponential backoff
- Cache fallback when APIs are unavailable
- Chaos tests to ensure resilience

**Best Practices:**
- Monitor circuit breaker state regularly
- Check error logs for API issues
- Have fallback plans for critical applications
- Report breaking changes to yahoo-finance2 library

**Impact:** ✅ Well-managed
- Resilience patterns minimize impact
- Cache provides data during outages
- Automatic recovery when API returns

---

## Testing Status

The following test suites verify data availability and quality:

### Test Suites

- **Real-World Tests** ([`tests/e2e/real-world.test.ts`](../tests/e2e/real-world.test.ts))
  - Tests all tools with actual Yahoo Finance API calls
  - Validates data quality and completeness
  - Tests error handling and fallbacks

- **Unit Tests** ([`tests/unit/`](../tests/unit/))
  - Tests individual components in isolation
  - Validates middleware logic
  - Tests data quality algorithms

- **Integration Tests** ([`tests/integration/`](../tests/integration/))
  - Tests tool and resource workflows
  - Validates data flow through middleware
  - Tests error scenarios

- **Chaos Tests** ([`tests/chaos/`](../tests/chaos/))
  - Tests resilience to network failures
  - Tests API changes and breaking scenarios
  - Tests partial data and corrupted responses

### Current Test Results

| Data Type | Status | Coverage |
|-----------|--------|----------|
| Quote data | ✅ All major fields verified | 95%+ |
| Historical data | ✅ OHLCV with integrity validation | 95%+ |
| Company profile | ✅ Verified with fallback classification | 90%+ |
| Earnings data | ✅ Quarterly earnings and estimates | 90%+ |
| Analyst data | ✅ Ratings and target prices | 90%+ |
| News data | ✅ Articles with metadata | 85%+ |
| Holders data | ✅ Major holders information | 85%+ |
| Options data | ✅ Options chains with Greeks | 85%+ |
| Financials data | ⚠️ Works but may encounter validation errors | 70%+ |
| Crypto/Forex data | ⚠️ Tools exist but return placeholder data | 0% |

### Running Tests

```bash
npm test                          # All tests
npm run test:coverage            # With coverage report
npm run test:watch               # Watch mode
```

---

## API Dependencies

This MCP server depends on the [yahoo-finance2](https://github.com/gadicc/yahoo-finance2) library (v3.11.2).

### Library Capabilities

| Library Method | Status | MCP Tool |
|--------------|--------|------------|
| `quote()` | ✅ Fully supported | [`get_quote`](TOOLS.md#get_quote) |
| `quoteSummary()` | ✅ Fully supported | [`get_quote_summary`](TOOLS.md#get_quote_summary) |
| `historical()` | ✅ Fully supported | [`get_historical_prices`](TOOLS.md#get_historical_prices) |
| `trendingSymbols()` | ✅ Fully supported | [`get_trending_symbols`](TOOLS.md#get_trending_symbols) |
| `screener()` | ✅ Fully supported | [`screener`](TOOLS.md#screener) |
| `fundamentalsTimeSeries()` | ⚠️ Partial support (validation issues) | [`get_balance_sheet`](TOOLS.md#get_balance_sheet), [`get_income_statement`](TOOLS.md#get_income_statement), [`get_cash_flow_statement`](TOOLS.md#get_cash_flow_statement) |
| Dedicated crypto methods | ❌ Not supported | N/A |
| Dedicated forex methods | ❌ Not supported | N/A |

### Recommendations

For enhanced crypto/forex support, consider:
1. **Using dedicated APIs:**
   - Crypto: CoinGecko, CoinMarketCap, Binance
   - Forex: OANDA, Fixer.io, ExchangeRate-API

2. **Integrating as additional MCP tools:**
   - Implement crypto API client
   - Implement forex API client
   - Follow existing tool patterns

3. **Contributing to yahoo-finance2:**
   - Add crypto/forex support to library
   - Report issues with `fundamentalsTimeSeries()` validation

For more information on tools, see [TOOLS.md](TOOLS.md).
