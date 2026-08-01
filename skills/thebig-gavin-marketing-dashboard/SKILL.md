---
name: thebig-gavin-marketing-dashboard
description: Real-time financial market research cockpit aggregating CN/HK/US indices, commodities, treasury yields, sector hotspots, capital flows, 7×24 news, and AI token usage trends in a single screen dashboard.
triggers:
  - set up a financial market research dashboard
  - build a real-time stock market monitoring cockpit
  - create a multi-market data visualization dashboard
  - integrate financial data from multiple sources
  - display A-share Hong Kong and US stock indices
  - track commodity prices and treasury yields in real-time
  - build an industry chain watchlist dashboard
  - monitor AI model token usage trends
---

# theBigGavin/marketingdashboard

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive real-time market research cockpit that aggregates financial data from multiple sources into a single-screen dashboard. Built with React 19, Vite, TypeScript, and Tailwind CSS, it provides live tracking of A-share/Hong Kong/US stock indices, commodities, US treasury yields, sector hotspots, capital flows, 7×24 financial news, industry chain watchlists, and AI model token consumption trends.

## What It Does

The Marketing Dashboard is a zero-dependency data service that:

- **Multi-Market Coverage**: Real-time quotes for Shanghai/Shenzhen, Hang Seng, Dow Jones, NASDAQ, S&P 500, VIX, and USD/CNY
- **Commodities & Crypto**: Live prices for gold, silver, copper, crude oil, and Bitcoin with intraday charts
- **Treasury Monitoring**: 10Y/2Y yields, 2s10s spread, yield curve shapes with historical monthly data
- **Sector Analysis**: Industry/concept board rankings with constituent stocks, leading stocks, and capital flows
- **Capital Flow Tracking**: Top net inflow stocks, sector capital flow minute-level curves, hot/gainer/loser lists
- **Industry Chains**: Semiconductor, AI computing, new energy vehicles, robotics, pharma innovation with upstream/midstream/downstream stocks
- **AI Cockpit**: OpenRouter daily leaderboard tracking 50+ AI model providers' token consumption trends (7d~1y, 60+ day historical lookback)
- **Commodity Prices**: Precious metals, base metals, energy, agricultural products futures trends with spot price comparisons
- **News Aggregation**: 7×24 global financial news with macro keyword highlighting
- **PWA Support**: Installable as desktop application with offline caching

## Installation

### Prerequisites

- Node.js 18+
- System with `curl` available (for proxy endpoints)

### Local Development

```bash
# Clone the repository
git clone https://github.com/theBigGavin/marketingdashboard.git
cd marketingdashboard

# Install dependencies
npm install

# Start development server
npm run dev
```

- Frontend dev server: http://localhost:3000
- Data proxy service: http://localhost:3001 (Vite auto-proxies `/api` requests)

### Production Deployment

```bash
# Build for production
npm run build

# Optional: Configure OpenRouter API Key for AI dashboard
echo 'OPENROUTER_API_KEY=sk-or-v1-xxxx' > server/.env

# Start production server (serves both API and static files)
npm start
```

Access at http://localhost:3000

### Docker Deployment

```bash
docker build -t market-cockpit .
docker run -p 3000:3000 market-cockpit
```

### Environment Variables

Create `server/.env` for optional API keys:

```env
# Optional: OpenRouter API key for AI dashboard
OPENROUTER_API_KEY=sk-or-v1-your-key-here

# Optional: Custom port (default: 3000)
PORT=3000
```

## API Reference

All APIs are accessed via `/api` prefix and include built-in caching and rate limiting:

### Market Data

```typescript
// Get real-time quotes for indices/stocks
GET /api/quotes?codes=sh000001,hk.00700,us.AAPL

// Response format
{
  "sh000001": {
    "code": "sh000001",
    "name": "上证指数",
    "price": 3250.12,
    "change": 15.32,
    "percent": 0.47,
    "volume": "285.6亿",
    "open": 3245.80,
    "high": 3255.60,
    "low": 3242.10
  }
}
```

```typescript
// Get intraday minute chart
GET /api/minute?code=sh000001

// Response: array of [timestamp, price, volume]
[
  ["09:30", 3245.80, 12500000],
  ["09:31", 3246.20, 13200000],
  // ...
]
```

### Sector & Board Data

```typescript
// Get sector rankings
GET /api/boards?type=industry&dir=desc&n=20

// Get board constituent stocks
GET /api/board-stocks?code=BK0447&n=50

// Response format
{
  "stocks": [
    {
      "code": "300750",
      "name": "宁德时代",
      "price": 235.60,
      "percent": 2.35
    }
  ]
}
```

### Futures & Commodities

```typescript
// Get commodity/crypto quotes
GET /api/futures?list=GC00Y,CL00Y,BTCUSD

// Get futures intraday chart
GET /api/future-minute?code=nf_AU0

// Get futures daily K-line (up to 400 bars)
GET /api/future-daily?code=nf_AU0&n=90
```

### Capital Flow

```typescript
// Get top net inflow stocks
GET /api/moneyflow?n=50

// Get batch stock capital flows
GET /api/stock-flows?codes=300750,600519

// Get sector capital flow curves
GET /api/board-flow?n=100
```

### Treasury Data

```typescript
// Get current treasury yields
GET /api/treasuries

// Response format
{
  "2Y": 4.25,
  "10Y": 4.15,
  "spread_2s10s": -0.10,
  "timestamp": "2026-07-31T10:00:00Z"
}

// Get historical yield curves (2001-present)
GET /api/treasury-history
```

### News & Search

```typescript
// Get 7×24 financial news
GET /api/news?page=1&size=50

// Search stocks by name/pinyin
GET /api/stock-search?q=宁德

// Get stock's associated boards
GET /api/stock-boards?code=300750
```

### Industry Chain & AI

```typescript
// Query stocks by concept/industry (using Wencai)
GET /api/mystery-select?query=人工智能&limit=50

// Parse industry chain text (auto-categorize upstream/midstream/downstream)
POST /api/chain-parse
Content-Type: application/json
{
  "text": "上游：芯片设计\n中游：封装测试\n下游：终端应用"
}

// Get OpenRouter daily usage leaderboard
GET /api/openrouter-usage
```

## Code Examples

### Using the API Client

```typescript
// src/lib/api.ts - API client examples
import { fetchQuotes, fetchMinuteChart, fetchBoardStocks } from './lib/api';

// Fetch multiple quotes
const quotes = await fetchQuotes(['sh000001', 'hk.HSI', 'us.DJI']);
console.log(quotes['sh000001'].price);

// Fetch minute chart
const chart = await fetchMinuteChart('sh000001');
chart.forEach(([time, price, volume]) => {
  console.log(`${time}: ${price}`);
});

// Fetch board constituent stocks
const stocks = await fetchBoardStocks('BK0447', 30);
stocks.forEach(stock => {
  console.log(`${stock.name}: ${stock.percent}%`);
});
```

### Creating a Custom Panel Component

```typescript
// src/components/dash/CustomPanel.tsx
import React, { useEffect, useState } from 'react';
import { usePolling } from '../../hooks/usePolling';
import { fetchQuotes } from '../../lib/api';

interface Quote {
  code: string;
  name: string;
  price: number;
  percent: number;
}

export default function CustomPanel() {
  const [quotes, setQuotes] = useState<Record<string, Quote>>({});

  // Poll every 5 seconds
  usePolling(async () => {
    const data = await fetchQuotes(['sh000001', 'hk.HSI']);
    setQuotes(data);
  }, 5000);

  return (
    <div className="bg-gray-900 rounded-lg p-4">
      <h3 className="text-lg font-semibold mb-3 text-white">Market Indices</h3>
      <div className="space-y-2">
        {Object.values(quotes).map(quote => (
          <div key={quote.code} className="flex justify-between items-center">
            <span className="text-gray-300">{quote.name}</span>
            <div className="text-right">
              <div className="text-white font-mono">{quote.price.toFixed(2)}</div>
              <div className={quote.percent >= 0 ? 'text-red-500' : 'text-green-500'}>
                {quote.percent >= 0 ? '+' : ''}{quote.percent.toFixed(2)}%
              </div>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Using the Unified Market Data Center

```typescript
// src/lib/market.ts - Unified quote center
import { market } from './lib/market';

// Subscribe to market data updates
const unsubscribe = market.subscribe((quotes) => {
  console.log('Latest quotes:', quotes);
  // All panels receive the same snapshot, ensuring consistency
});

// Add codes to watch
market.addCodes(['sh000001', 'hk.00700', 'us.AAPL']);

// Get current quote
const quote = market.getQuote('sh000001');
console.log(`${quote.name}: ${quote.price} (${quote.percent}%)`);

// Cleanup
unsubscribe();
```

### Creating a Watchlist with Search

```typescript
// Using the WatchlistPanel component
import WatchlistPanel from './components/dash/WatchlistPanel';
import { fetchStockSearch } from './lib/api';

// Search for stocks
const searchStocks = async (query: string) => {
  const results = await fetchStockSearch(query);
  // results: Array<{ code: string; name: string }>
  return results;
};

// Add to watchlist (stored in localStorage)
const addToWatchlist = (code: string, name: string) => {
  const watchlist = JSON.parse(localStorage.getItem('watchlist') || '[]');
  watchlist.push({ code, name });
  localStorage.setItem('watchlist', JSON.stringify(watchlist));
};
```

### Fetching AI Model Usage Data

```typescript
// Fetch OpenRouter daily leaderboard
import { fetchOpenRouterUsage } from './lib/api';

const usageData = await fetchOpenRouterUsage();
// Response includes daily token consumption by provider
// Data is cached and accumulated over time on server

usageData.forEach(entry => {
  console.log(`${entry.date}: ${entry.provider} - ${entry.tokens} tokens`);
});
```

### Adding a New Data Source

```typescript
// server/index.cjs - Add new API endpoint
// Inside the request handler:

if (url === '/api/custom-data') {
  const cacheKey = 'custom-data';
  const cached = cache.get(cacheKey);
  if (cached) {
    sendJSON(res, cached);
    return;
  }

  try {
    const response = await fetch('https://api.example.com/data');
    const data = await response.json();
    
    cache.set(cacheKey, data, 60); // Cache for 60 seconds
    sendJSON(res, data);
  } catch (error) {
    console.error('Error fetching custom data:', error);
    sendJSON(res, { error: 'Failed to fetch data' }, 500);
  }
  return;
}
```

### Creating a Custom Spark Chart

```typescript
// src/components/dash/Spark.tsx - Mini chart component
import Spark from './components/dash/Spark';

// Usage in your panel
<Spark
  data={minuteData}  // Array of [time, price, volume]
  mode="a-stock"      // 'a-stock' | '24h' | 'daily'
  color={percent >= 0 ? 'red' : 'green'}
  height={60}
/>
```

## Configuration

### Market Indices Configuration

Edit `src/config/indices.ts` to customize tracked indices:

```typescript
export const INDICES = [
  { code: 'sh000001', name: '上证指数', market: 'CN' },
  { code: 'sz399001', name: '深证成指', market: 'CN' },
  { code: 'hk.HSI', name: '恒生指数', market: 'HK' },
  { code: 'us.DJI', name: '道琼斯', market: 'US' },
  // Add custom indices
];
```

### Commodity Configuration

Edit `src/config/commodities.ts`:

```typescript
export const COMMODITIES = [
  { code: 'GC00Y', name: '纽约黄金', unit: '美元/盎司' },
  { code: 'BTCUSD', name: '比特币', unit: 'USDT' },
  // Add custom commodities
];
```

### Industry Chain Configuration

Edit `src/config/chains.ts` to define custom industry chains:

```typescript
export const INDUSTRY_CHAINS = {
  'semiconductor': {
    name: '半导体',
    layers: {
      upstream: [
        { code: '603501', name: '韦尔股份', desc: '芯片设计' },
      ],
      midstream: [
        { code: '603160', name: '汇顶科技', desc: '芯片封装' },
      ],
      downstream: [
        { code: '002475', name: '立讯精密', desc: '终端应用' },
      ]
    }
  }
  // Add custom chains
};
```

## Common Patterns

### Polling with Shared State

```typescript
// Use useSharedPolling for data shared across components
import { useSharedPolling } from '../hooks/useSharedPolling';

const MyComponent = () => {
  const { data, loading } = useSharedPolling(
    'shared-key',
    async () => await fetchQuotes(['sh000001']),
    5000 // 5 second interval
  );

  return <div>{data?.sh000001?.price}</div>;
};
```

### Handling Market Hours

```typescript
// Check if market is open
import { isMarketOpen, getNextMarketTime } from '../lib/utils';

const isOpen = isMarketOpen('CN'); // 'CN' | 'US' | 'HK'
const nextOpen = getNextMarketTime('CN');

// Adjust polling frequency based on market hours
const interval = isOpen ? 5000 : 60000; // Fast when open, slow when closed
```

### Caching Data on Client

```typescript
// Use localStorage for persistent client-side data
const saveWatchlist = (stocks: Array<{ code: string; name: string }>) => {
  localStorage.setItem('watchlist', JSON.stringify(stocks));
};

const loadWatchlist = () => {
  const data = localStorage.getItem('watchlist');
  return data ? JSON.parse(data) : [];
};
```

### Rate Limiting Considerations

The server enforces rate limits:
- Public endpoints: 240 requests/minute per IP
- Private endpoints (mystery-select, openrouter-usage): 20 requests/minute per IP
- POST body limit: 256KB

Handle rate limits gracefully:

```typescript
const fetchWithRetry = async (url: string, maxRetries = 3) => {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url);
      if (response.status === 429) {
        // Rate limited, wait and retry
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      return await response.json();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
    }
  }
};
```

## Troubleshooting

### API Returns Empty Data

**Problem**: Endpoints return `{}` or `[]`

**Solutions**:
- Check if data source is accessible (some APIs block certain regions)
- Verify cache TTL hasn't expired aggressively
- Check server logs for upstream API errors
- For stock codes, ensure correct market prefix (`sh`, `sz`, `hk.`, `us.`)

### CORS Errors in Browser

**Problem**: Direct API calls fail with CORS error

**Solutions**:
- Use the `/api` proxy endpoints instead of direct upstream URLs
- The server only reflects CORS headers for same-origin requests
- For external use, deploy your own instance
- Check if browser is blocking mixed content (HTTP/HTTPS)

### Minute Chart Shows Gaps

**Problem**: Spark charts have missing data points

**Solutions**:
- A-share markets: Data only available during trading hours (9:30-11:30, 13:00-15:00)
- Use `mode="a-stock"` for proper time-segmented rendering
- For 24h markets (crypto, forex), use `mode="24h"`
- Check if the API returned incomplete data during non-trading hours

### PWA Not Installing

**Problem**: Browser doesn't show install prompt

**Solutions**:
- Ensure site is served over HTTPS (required for PWA)
- Check `public/manifest.json` is accessible
- Verify Service Worker registered correctly in DevTools > Application
- PWA requires at least one icon >= 192×192px
- Try clearing browser cache and reloading

### High Memory Usage

**Problem**: Server consumes excessive memory

**Solutions**:
- Server cache is bounded (LRU + periodic cleanup)
- Reduce cache size in `server/index.cjs` (`maxCacheSize` constant)
- Increase cache cleanup frequency (default: every 10 minutes)
- For large deployments, consider Redis for shared cache

### OpenRouter API Key Issues

**Problem**: AI dashboard shows no data

**Solutions**:
- Ensure `OPENROUTER_API_KEY` is set in `server/.env`
- Check API key has sufficient credits at openrouter.ai
- Verify same-origin request (endpoint blocks CORS)
- Check rate limit (20 req/min for private endpoints)

### Stock Search Not Working

**Problem**: `/api/stock-search` returns no results

**Solutions**:
- Try both full name and pinyin abbreviation (e.g., "宁德" or "nd")
- Backend uses Sina's suggest API which may have regional restrictions
- For A-shares, try with/without market prefix
- Consider implementing local stock list cache for offline search

### Futures Data Missing

**Problem**: Commodity/futures prices not updating

**Solutions**:
- Check if market is open (commodities trade on specific exchanges/hours)
- Verify correct code format: `nf_` for domestic (e.g., `nf_AU0`), `hf_` for international (e.g., `hf_CL`)
- Some futures only trade during specific sessions
- Historical K-line data requires valid contract month

### Treasury History Data Incomplete

**Problem**: `/api/treasury-history` missing recent months

**Solutions**:
- Historical data stored in `server/treasury-rates/` directory
- Current year data fetched online and merged
- Ensure server has write access to treasury-rates directory
- Check upstream data source availability
- Data accumulated monthly, gaps normal for mid-month
