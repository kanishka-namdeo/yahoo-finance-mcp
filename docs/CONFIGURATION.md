# Configuration Guide

Complete guide to configuring the Yahoo Finance MCP Server for your needs.

## Table of Contents

- [Overview](#overview)
- [Quick Configuration](#quick-configuration)
- [Rate Limiting](#rate-limiting)
- [Caching Strategy](#caching-strategy)
- [Circuit Breaker](#circuit-breaker)
- [Retry Logic](#retry-logic)
- [Server Settings](#server-settings)
- [Environment Variables](#environment-variables)
- [Best Practices](#best-practices)

---

## Overview

Configuration is done through a `config.json` file in the project root. All settings are validated at startup using Zod schemas.

**Configuration loading order:**
1. Default values (in code)
2. `config.json` file (if present)
3. Environment variables (override file settings)

---

## Quick Configuration

### Minimal Configuration

For most use cases, start with this minimal config:

```json
{
  "server": {
    "transport": "stdio"
  }
}
```

### Recommended Configuration

For production use:

```json
{
  "server": {
    "transport": "stdio",
    "logLevel": "info"
  },
  "rateLimit": {
    "requestsPerMinute": 60,
    "requestsPerHour": 1500
  },
  "cache": {
    "maxCacheSize": 1000
  },
  "circuitBreaker": {
    "failureThreshold": 5,
    "monitoringWindow": 60000,
    "successThreshold": 3
  }
}
```

---

## Rate Limiting

### Overview

The rate limiter uses three strategies to prevent overwhelming the Yahoo Finance API:
1. **Token Bucket**: Smooth burst handling
2. **Adaptive Throttling**: Adjusts based on API responses
3. **Per-Endpoint Tracking**: Prevents hotspot abuse

### Configuration Options

```json
{
  "rateLimit": {
    "requestsPerMinute": 60,
    "requestsPerHour": 1500,
    "tokenRefillRate": 1.0,
    "burstLimit": 5
  }
}
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `requestsPerMinute` | number | 60 | Maximum requests per minute |
| `requestsPerHour` | number | 1500 | Maximum requests per hour |
| `tokenRefillRate` | number | 1.0 | Tokens added per millisecond |
| `burstLimit` | number | 5 | Maximum burst requests |

### Recommended Settings

**Development:**
```json
{
  "rateLimit": {
    "requestsPerMinute": 30,
    "requestsPerHour": 500
  }
}
```

**Production:**
```json
{
  "rateLimit": {
    "requestsPerMinute": 60,
    "requestsPerHour": 1500
  }
}
```

**High-Volume:**
```json
{
  "rateLimit": {
    "requestsPerMinute": 120,
    "requestsPerHour": 3000
  }
}
```

### How Token Bucket Works

1. **Tokens refilled** at `tokenRefillRate` per millisecond
2. **Each request** consumes one token
3. **Burst allowed** up to `burstLimit` tokens
4. **After burst**, requests throttled until tokens refill

**Example** (requestsPerMinute=60, burstLimit=5):
- 60 tokens refill per minute (1 token/second)
- Can burst 5 requests instantly
- After burst, wait 5 seconds for full refill
- Sustained rate: 60 requests/minute

---

## Caching Strategy

### Overview

Cache reduces API calls and improves performance with tiered TTL strategy.

### Configuration Options

```json
{
  "cache": {
    "ttlQuotes": 60000,
    "ttlHistorical": 3600000,
    "ttlFinancials": 86400000,
    "ttlNews": 300000,
    "maxCacheSize": 1000
  }
}
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `ttlQuotes` | number | 60000 | Time-to-live for quotes (1 minute) |
| `ttlHistorical` | number | 3600000 | Time-to-live for historical data (1 hour) |
| `ttlFinancials` | number | 86400000 | Time-to-live for financial statements (24 hours) |
| `ttlNews` | number | 300000 | Time-to-live for news (5 minutes) |
| `maxCacheSize` | number | 1000 | Maximum number of cache entries |

### Recommended Settings

**Real-Time Trading** (minimal caching):
```json
{
  "cache": {
    "ttlQuotes": 10000,
    "ttlHistorical": 600000,
    "maxCacheSize": 500
  }
}
```

**Analysis & Research** (standard caching):
```json
{
  "cache": {
    "ttlQuotes": 60000,
    "ttlHistorical": 3600000,
    "ttlFinancials": 86400000,
    "maxCacheSize": 1000
  }
}
```

**High-Volume Screening** (aggressive caching):
```json
{
  "cache": {
    "ttlQuotes": 120000,
    "ttlHistorical": 7200000,
    "ttlFinancials": 172800000,
    "maxCacheSize": 5000
  }
}
```

### Cache Memory Usage

Approximate memory usage per cache entry:
- Quote: ~500 bytes
- Historical data: ~2 KB per symbol
- Financials: ~5 KB per symbol
- News: ~1 KB per article

**Example** (1000 entries):
- Mixed cache: ~1-2 MB memory

---

## Circuit Breaker

### Overview

Circuit breaker prevents cascading failures by stopping requests when API is unstable.

### Configuration Options

```json
{
  "circuitBreaker": {
    "errorThresholdPercentage": 50,
    "failureThreshold": 5,
    "successThreshold": 3,
    "monitoringWindow": 60000,
    "resetTimeoutMs": 60000
  }
}
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `errorThresholdPercentage` | number | 50 | Error percentage to trigger opening |
| `failureThreshold` | number | 5 | Number of failures to trigger opening |
| `successThreshold` | number | 3 | Number of successes to close circuit |
| `monitoringWindow` | number | 60000 | Time window for failure counting (ms) |
| `resetTimeoutMs` | number | 60000 | Time to wait before testing recovery (ms) |

### How It Works

```
CLOSED State (Normal)
  ├─ Success: Reset failure count
  └─ Failure: Increment count
      └─ If count >= threshold → OPEN

OPEN State (Fail Fast)
  ├─ Reject all requests
  └─ Wait for resetTimeout
      └─ After timeout → HALF-OPEN

HALF-OPEN State (Testing)
  ├─ Allow limited requests (test probes)
  ├─ Success: Count successes
  │   └─ If successes >= threshold → CLOSED
  └─ Failure: Back to OPEN
```

### Recommended Settings

**Lenient** (more tolerant of failures):
```json
{
  "circuitBreaker": {
    "failureThreshold": 10,
    "successThreshold": 5,
    "monitoringWindow": 120000,
    "resetTimeoutMs": 30000
  }
}
```

**Standard** (balanced):
```json
{
  "circuitBreaker": {
    "failureThreshold": 5,
    "successThreshold": 3,
    "monitoringWindow": 60000,
    "resetTimeoutMs": 60000
  }
}
```

**Strict** (fail fast on errors):
```json
{
  "circuitBreaker": {
    "failureThreshold": 3,
    "successThreshold": 2,
    "monitoringWindow": 30000,
    "resetTimeoutMs": 120000
  }
}
```

---

## Retry Logic

### Overview

Exponential backoff with jitter handles transient failures.

### Configuration Options

```json
{
  "retry": {
    "maxRetries": 3,
    "baseDelay": 1000,
    "maxDelay": 10000,
    "jitter": 0.1,
    "retryableStatusCodes": [429, 500, 502, 503, 504],
    "retryableErrors": ["ECONNRESET", "ETIMEDOUT", "ECONNREFUSED"]
  }
}
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `maxRetries` | number | 3 | Maximum number of retry attempts |
| `baseDelay` | number | 1000 | Base delay between retries (ms) |
| `maxDelay` | number | 10000 | Maximum delay between retries (ms) |
| `jitter` | number | 0.1 | Random jitter factor (0-1) |
| `retryableStatusCodes` | number[] | [429, 500, 502, 503, 504] | HTTP codes to retry |
| `retryableErrors` | string[] | ["ECONNRESET", "ETIMEDOUT", "ECONNREFUSED"] | Error types to retry |

### How Exponential Backoff Works

**Delay calculation:**
```typescript
delay = Math.min(
  baseDelay * Math.pow(2, attempt) * (1 + Math.random() * jitter),
  maxDelay
);
```

**Example** (baseDelay=1000ms, maxDelay=10000ms, jitter=0.1):

| Attempt | Delay Range | Reason |
|---------|--------------|---------|
| 1 | 1000-1100ms | Initial retry |
| 2 | 2000-2200ms | Exponential backoff |
| 3 | 4000-4400ms | Longer backoff |
| 4+ | 10000ms (max) | Hit max delay |

### Why Jitter?

Jitter prevents thundering herd problems:
- **Without jitter**: All retries happen at same time → API overload
- **With jitter**: Retries spread randomly → Smoother load

### Recommended Settings

**Fast Failures** (quick retries):
```json
{
  "retry": {
    "maxRetries": 2,
    "baseDelay": 500,
    "maxDelay": 5000,
    "jitter": 0.2
  }
}
```

**Standard** (balanced):
```json
{
  "retry": {
    "maxRetries": 3,
    "baseDelay": 1000,
    "maxDelay": 10000,
    "jitter": 0.1
  }
}
```

**Persistent** (more retries):
```json
{
  "retry": {
    "maxRetries": 5,
    "baseDelay": 1000,
    "maxDelay": 30000,
    "jitter": 0.1
  }
}
```

---

## Server Settings

### Configuration Options

```json
{
  "server": {
    "transport": "stdio",
    "logLevel": "info"
  }
}
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `transport` | string | "stdio" | MCP transport protocol ("stdio" or "http") |
| `logLevel` | string | "info" | Logging level ("error", "warn", "info", "debug") |

### Transport Options

**stdio** (default):
- Standard MCP transport for Claude Desktop
- Direct communication via stdin/stdout
- Recommended for most use cases

**http** (advanced):
- HTTP transport for custom integrations
- Requires additional setup
- Not recommended for Claude Desktop

### Log Levels

| Level | Use Case |
|-------|-----------|
| `error` | Critical errors only |
| `warn` | Warnings and errors |
| `info` | Standard operation (recommended) |
| `debug` | Detailed debugging (verbose) |

---

## Environment Variables

Environment variables override config file settings:

| Variable | Format | Example |
|----------|---------|---------|
| `RATE_LIMIT_RPM` | number | `60` |
| `RATE_LIMIT_RPH` | number | `1500` |
| `CACHE_TTL_QUOTES` | number | `60000` |
| `CACHE_MAX_SIZE` | number | `1000` |
| `CIRCUIT_BREAKER_FAILURE_THRESHOLD` | number | `5` |
| `SERVER_LOG_LEVEL` | string | `info` |
| `SERVER_TRANSPORT` | string | `stdio` |

### Example Usage

**Linux/Mac:**
```bash
export RATE_LIMIT_RPM=60
export SERVER_LOG_LEVEL=debug
npm start
```

**Windows (PowerShell):**
```powershell
$env:RATE_LIMIT_RPM = 60
$env:SERVER_LOG_LEVEL = "debug"
npm start
```

**Windows (CMD):**
```cmd
set RATE_LIMIT_RPM=60
set SERVER_LOG_LEVEL=debug
npm start
```

---

## Best Practices

### Rate Limiting

1. **Start conservative** and increase gradually
2. **Monitor 429 errors** and adjust accordingly
3. **Use batch requests** instead of individual calls
4. **Cache aggressively** for less time-sensitive data

### Caching

1. **Shorter TTL** for real-time data (quotes)
2. **Longer TTL** for static data (financials)
3. **Monitor cache hit ratio** (target: 70-90%)
4. **Adjust cache size** based on memory constraints

### Circuit Breaker

1. **Tune based on your API reliability**
2. **Monitor circuit state** via stats endpoint
3. **Adjust thresholds** based on failure patterns
4. **Don't set too strict** - prevents false positives

### Retry Logic

1. **Keep retry attempts reasonable** (3-5)
2. **Always use jitter** to prevent thundering herd
3. **Set max delay** to prevent excessive waiting
4. **Monitor retry success rate** (target: 85-95%)

### General

1. **Start with defaults** and adjust based on needs
2. **Monitor metrics** regularly
3. **Test configuration** in development before production
4. **Document custom settings** for team members

For more information on architecture, see [ARCHITECTURE.md](ARCHITECTURE.md).
