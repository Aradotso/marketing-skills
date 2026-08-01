---
name: marketingdashboard-financial-cockpit
description: Real-time financial market research dashboard aggregating CN/HK/US indices, commodities, treasury yields, sector flows, news, and AI token usage
triggers:
  - create a financial market dashboard
  - build a stock market monitoring screen
  - set up real-time market data display
  - implement financial data visualization
  - create a trading cockpit interface
  - build a market research dashboard
  - set up commodity price tracking
  - create AI token usage dashboard
---

# marketingdashboard Financial Cockpit

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive real-time market research dashboard built with React 19, Vite, and TypeScript that aggregates A-share/Hong Kong/US stock indices, commodities, treasury yields, sector hotspots, capital flows, 7×24 news, industry chain watchlists, and AI model token usage trends on a single screen.

## What It Does

This project provides a zero-dependency data service with built-in Node.js proxy that aggregates public market data APIs. It features:

- **Global market indices** with minute-level charts (SSE, SZSE, HSI, DJIA, NDX, SPX, VIX, USD/CNY)
- **Commodities & crypto** (gold, silver, copper, crude oil, BTC) with real-time prices
- **US Treasury yields** monitoring (10Y, 2Y, spread, yield curve history)
- **Sector analysis** with industry/concept board rankings and constituent stocks
- **Capital flow tracking** with real-time money flow charts and stock rankings
- **Industry chain visualization** (semiconductors, AI compute, EVs, robotics, biotech)
- **AI Dashboard** tracking 50+ LLM providers' token consumption via OpenRouter API
- **Goods price page** with futures trends and spot-futures basis tables
- **7×24 financial news** aggregation with keyword highlighting
- **PWA support** for desktop installation

## Installation

### Prerequisites

- Node.js 18+
- `curl` command available (for some proxy endpoints)

### Local Development

```bash
# Clone the repository
git clone https://github.com/theBigGavin/marketingdashboard.git
cd marketingdashboard

# Install dependencies
npm install
# or
pnpm install

# Start development servers
npm run dev
```

- Frontend dev server: http://localhost:3000
- Data proxy service: http://localhost:3001 (Vite auto-proxies `/api`)

### Production Build

```bash
# Build frontend
npm run build

# (Optional) Configure OpenRouter API Key for AI Dashboard
echo 'OPENROUTER_API_KEY=sk-or-v1-your-key-here' > server/.env

# Start production server (serves both API and static files)
npm start
```

Visit http://localhost:3000

### Docker Deployment

```bash
docker build -t market-cockpit .
docker run -p 3000:3000 market-cockpit
```

## Architecture & Data Flow

The project uses a unified quote center (`src/lib/market.ts`) that batch-fetches prices every 5s and distributes snapshots to all components. Server-side caching reduces API pressure:

```typescript
// Frontend unified quote center
import { quoteCenter } from '@/lib/market';

// Subscribe to real-time quotes
useEffect(() => {
  const codes = ['sh000001', 'sz399001', 'hk.HSI'];
  quoteCenter.subscribe(codes);
  
  return () => quoteCenter.unsubscribe(codes);
}, []);

// Get latest quote data
const quote = quoteCenter.getQuote('sh000001');
// { code, name, price, change, changePercent, ... }
```

## Key API Endpoints

All endpoints are proxied through `/api/*`:

### Real-time Quotes

```typescript
// Fetch multiple stock/index quotes
const response = await fetch('/api/quotes?codes=sh000001,sz399001,hk.HSI');
const quotes = await response.json();
// Returns array of { code, name, price, change, changePercent, open, high, low, volume, ... }
```

### Minute Charts

```typescript
// Get intraday minute-level data
const response = await fetch('/api/minute?code=sh000001');
const data = await response.json();
// { times: ['09:31', '09:32', ...], prices: [3250.5, 3251.2, ...] }
```

### Sector Boards

```typescript
// Get top industry/concept boards
const response = await fetch('/api/boards?type=industry&dir=desc&n=20');
const boards = await response.json();
// [{ code, name, changePercent, leader, leaderPercent, ... }, ...]
```

### Board Constituent Stocks

```typescript
// Get stocks in a specific board
const response = await fetch('/api/board-stocks?code=BK0477&n=50');
const stocks = await response.json();
// [{ code, name, price, changePercent, ... }, ...]
```

### Commodities & Futures

```typescript
// Get futures/commodity quotes
const response = await fetch('/api/futures?list=GC00,CL00,BTC');
const futures = await response.json();
// [{ code, name, price, change, changePercent, ... }, ...]

// Get futures daily K-line (up to 400 bars)
const response = await fetch('/api/future-daily?code=nf_AU0&n=90');
const dailyData = await response.json();
// { dates: ['2024-01-01', ...], open: [...], high: [...], low: [...], close: [...] }
```

### Capital Flow

```typescript
// Get top stocks by net capital inflow
const response = await fetch('/api/moneyflow?n=30');
const flows = await response.json();
// [{ code, name, netInflow, netInflowPercent, ... }, ...]

// Get board-level capital flow curves
const response = await fetch('/api/board-flow?n=10');
const boardFlows = await response.json();
// [{ name, data: [{ time, value }, ...] }, ...]
```

### Stock Rankings

```typescript
// Get stock rankings by various metrics
const response = await fetch('/api/rank?sort=amount&n=50');
// sort: 'rise' (gainers) | 'fall' (losers) | 'amount' (volume) | 'turnover'
const ranks = await response.json();
```

### US Treasury Yields

```typescript
// Get current treasury yields
const response = await fetch('/api/treasuries');
const yields = await response.json();
// { '2Y': 4.25, '10Y': 4.15, spread: -0.10, ... }

// Get historical yield curve data (monthly snapshots since 2001)
const response = await fetch('/api/treasury-history');
const history = await response.json();
// [{ date: '2024-01', '1M': 5.2, '3M': 5.3, '6M': 5.1, '1Y': 4.8, '2Y': 4.5, ... }, ...]
```

### Industry Chain & Stock Search

```typescript
// Search stocks by concept/industry (uses mystery select API)
const response = await fetch('/api/mystery-select?query=半导体&limit=50');
const stocks = await response.json();
// [{ code: '600584', name: '长电科技', ... }, ...]

// Parse industry chain text into upstream/midstream/downstream
const response = await fetch('/api/chain-parse', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ text: '上游：硅片\n中游：芯片设计\n下游：封装测试' })
});
const parsed = await response.json();
// { upstream: [...], midstream: [...], downstream: [...] }

// Stock search by name/pinyin
const response = await fetch('/api/stock-search?q=贵州茅台');
const results = await response.json();
// [{ code: '600519', name: '贵州茅台', market: 'sh' }, ...]
```

### AI Dashboard (OpenRouter)

```typescript
// Get daily LLM token usage data
const response = await fetch('/api/openrouter-usage');
const usage = await response.json();
// { dates: ['2024-01-01', ...], providers: { 'openai': [1000, 1200, ...], 'anthropic': [...], ... } }
```

### Financial News

```typescript
// Get 7×24 financial news feed
const response = await fetch('/api/news?page=1&size=50');
const news = await response.json();
// { items: [{ id, title, content, time, source, ... }, ...], total: 1000 }
```

## Component Examples

### Using the Polling Hook

```typescript
import { usePolling } from '@/hooks/usePolling';

function MarketPanel() {
  const { data, loading, error } = usePolling<QuoteData[]>(
    async () => {
      const res = await fetch('/api/quotes?codes=sh000001,sz399001');
      return res.json();
    },
    5000 // Poll every 5 seconds
  );

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      {data?.map(quote => (
        <div key={quote.code}>
          {quote.name}: {quote.price} ({quote.changePercent}%)
        </div>
      ))}
    </div>
  );
}
```

### Creating a Mini Sparkline Chart

```typescript
import { Spark } from '@/components/dash/Spark';

function IndexCard() {
  const [minuteData, setMinuteData] = useState<number[]>([]);
  
  useEffect(() => {
    fetch('/api/minute?code=sh000001')
      .then(res => res.json())
      .then(data => setMinuteData(data.prices));
  }, []);

  return (
    <div className="p-4 bg-slate-800 rounded">
      <h3>上证指数</h3>
      <Spark 
        data={minuteData} 
        width={200} 
        height={60} 
        color="#10b981"
        strokeWidth={1.5}
      />
    </div>
  );
}
```

### Building a Sector Panel

```typescript
import { usePolling } from '@/hooks/usePolling';

function SectorPanel() {
  const { data: boards } = usePolling(
    async () => {
      const res = await fetch('/api/boards?type=industry&dir=desc&n=10');
      return res.json();
    },
    10000
  );

  const [selectedBoard, setSelectedBoard] = useState<string | null>(null);
  const { data: stocks } = usePolling(
    async () => {
      if (!selectedBoard) return [];
      const res = await fetch(`/api/board-stocks?code=${selectedBoard}&n=20`);
      return res.json();
    },
    5000,
    [selectedBoard]
  );

  return (
    <div className="grid grid-cols-2 gap-4">
      <div>
        <h3>行业板块</h3>
        {boards?.map(board => (
          <div 
            key={board.code}
            onClick={() => setSelectedBoard(board.code)}
            className="cursor-pointer hover:bg-slate-700 p-2"
          >
            {board.name} {board.changePercent}%
          </div>
        ))}
      </div>
      <div>
        <h3>成分股</h3>
        {stocks?.map(stock => (
          <div key={stock.code}>
            {stock.name} {stock.price} ({stock.changePercent}%)
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Watchlist with Local Storage

```typescript
import { useState, useEffect } from 'react';
import { quoteCenter } from '@/lib/market';

function WatchlistPanel() {
  const [watchlist, setWatchlist] = useState<string[]>(() => {
    const saved = localStorage.getItem('watchlist');
    return saved ? JSON.parse(saved) : [];
  });

  useEffect(() => {
    localStorage.setItem('watchlist', JSON.stringify(watchlist));
    quoteCenter.subscribe(watchlist);
    return () => quoteCenter.unsubscribe(watchlist);
  }, [watchlist]);

  const addStock = (code: string) => {
    setWatchlist(prev => [...new Set([...prev, code])]);
  };

  const removeStock = (code: string) => {
    setWatchlist(prev => prev.filter(c => c !== code));
  };

  return (
    <div>
      {watchlist.map(code => {
        const quote = quoteCenter.getQuote(code);
        return (
          <div key={code}>
            {quote?.name || code}: {quote?.price}
            <button onClick={() => removeStock(code)}>Remove</button>
          </div>
        );
      })}
    </div>
  );
}
```

## Configuration

### Environment Variables

Create `server/.env` for server-side configuration:

```bash
# Optional: OpenRouter API Key for AI Dashboard
OPENROUTER_API_KEY=sk-or-v1-your-key-here

# Optional: Custom port (default: 3000 in production, 3001 for dev proxy)
PORT=3000
```

### Static Configuration Files

Key configuration files in `src/config/`:

```typescript
// src/config/indices.ts - Index definitions
export const INDICES = [
  { code: 'sh000001', name: '上证指数', color: '#ef4444' },
  { code: 'sz399001', name: '深证成指', color: '#10b981' },
  // ...
];

// src/config/commodities.ts - Commodity definitions
export const COMMODITIES = [
  { code: 'GC00', name: '纽约金', unit: '美元/盎司' },
  { code: 'CL00', name: 'WTI原油', unit: '美元/桶' },
  // ...
];

// src/config/chains.ts - Industry chain definitions
export const CHAINS = [
  {
    name: '半导体产业链',
    segments: [
      { name: '上游', stocks: ['600584', '688981', ...] },
      { name: '中游', stocks: ['688396', '688256', ...] },
      { name: '下游', stocks: ['002371', '603986', ...] }
    ]
  },
  // ...
];
```

## Common Patterns

### Multi-Panel Dashboard Layout

```typescript
function Dashboard() {
  return (
    <div className="grid grid-cols-12 gap-4 p-4 bg-slate-900 min-h-screen">
      {/* Left column: Indices + Commodities */}
      <div className="col-span-3 space-y-4">
        <IndicesPanel />
        <CommoditiesPanel />
      </div>
      
      {/* Center column: Sectors + Capital Flow */}
      <div className="col-span-6 space-y-4">
        <SectorPanel />
        <CapitalFlowPanel />
      </div>
      
      {/* Right column: News + Watchlist */}
      <div className="col-span-3 space-y-4">
        <NewsPanel />
        <WatchlistPanel />
      </div>
    </div>
  );
}
```

### Shared Polling for Multiple Components

```typescript
import { useSharedPolling } from '@/hooks/useSharedPolling';

// Multiple components can share the same polling instance
function Component1() {
  const { data } = useSharedPolling('sector-data', 
    () => fetch('/api/boards?type=industry&n=10').then(r => r.json()),
    10000
  );
  // ...
}

function Component2() {
  const { data } = useSharedPolling('sector-data', 
    () => fetch('/api/boards?type=industry&n=10').then(r => r.json()),
    10000
  );
  // Both components share the same data and polling timer
}
```

### Error Handling with Fallback

```typescript
function RobustPanel() {
  const { data, error, retry } = usePolling(
    async () => {
      try {
        const res = await fetch('/api/quotes?codes=sh000001');
        if (!res.ok) throw new Error('API error');
        return res.json();
      } catch (err) {
        // Fallback to direct Tencent API
        const res = await fetch('https://qt.gtimg.cn/q=sh000001');
        return parseDirectResponse(res);
      }
    },
    5000
  );

  if (error) {
    return (
      <div>
        Error loading data. <button onClick={retry}>Retry</button>
      </div>
    );
  }
  // ...
}
```

## Troubleshooting

### API Rate Limiting

The server enforces rate limits:
- Public endpoints: 240 requests/minute per IP
- Private endpoints (mystery-select, openrouter): 20 requests/minute per IP
- Response: 429 Too Many Requests

**Solution**: Increase polling intervals or implement request batching.

### CORS Issues

APIs only reflect CORS headers for same-origin requests. Cross-origin browser requests are blocked.

**Solution**: Always use the `/api/*` proxy, not direct external API URLs.

### Memory Cache Growing

Server uses LRU cache with automatic cleanup. If memory issues occur:

**Solution**: Restart the server or reduce cache TTL in `server/index.cjs`:

```javascript
// Reduce cache TTL (milliseconds)
const CACHE_TTL = {
  quotes: 1500,    // 1.5s (was default)
  boards: 5000,    // 5s → reduce to 3000
  news: 30000,     // 30s → reduce to 15000
};
```

### Missing Historical Data

Spot price history is accumulated by server-side scheduled tasks (every 4 hours).

**Solution**: Run the server continuously to build history, or manually trigger:

```bash
# Inside server directory
node -e "require('./index.cjs').collectSpotPrices()"
```

### OpenRouter API Not Working

AI Dashboard requires `OPENROUTER_API_KEY` environment variable.

**Solution**:

```bash
# In server/.env
OPENROUTER_API_KEY=sk-or-v1-your-actual-key

# Restart server
npm start
```

### PWA Not Installing

PWA requires HTTPS or localhost.

**Solution**: Deploy behind a reverse proxy with SSL (nginx, Cloudflare Tunnel) or use localhost for testing.

### Development Proxy Not Working

Frontend dev server (Vite) should auto-proxy `/api` to port 3001.

**Solution**: Check `vite.config.ts`:

```typescript
export default defineConfig({
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true
      }
    }
  }
});
```

If port 3001 is in use, change it in `server/dev.cjs`:

```javascript
const PORT = process.env.PROXY_PORT || 3001;
```

## Advanced Usage

### Adding Custom Industry Chains

Edit `src/config/chains.ts`:

```typescript
export const CHAINS = [
  // ... existing chains
  {
    name: 'My Custom Chain',
    segments: [
      { 
        name: 'Upstream', 
        stocks: ['600000', '600001'] // Add stock codes
      },
      { 
        name: 'Midstream', 
        stocks: ['600002', '600003'] 
      },
      { 
        name: 'Downstream', 
        stocks: ['600004'] 
      }
    ]
  }
];
```

### Extending API Endpoints

Add new endpoints in `server/index.cjs`:

```javascript
function handleRequest(req, res) {
  const url = new URL(req.url, `http://${req.headers.host}`);
  
  if (url.pathname === '/api/my-endpoint') {
    const data = await fetchMyData();
    sendJSON(res, data);
    return;
  }
  
  // ... existing handlers
}
```

### Custom Market Hours

Trading hours are defined in `src/lib/market.ts`:

```typescript
export function isInTradingHours(market: 'cn' | 'us' | 'hk'): boolean {
  const now = new Date();
  const hour = now.getHours();
  const minute = now.getMinutes();
  
  if (market === 'cn') {
    return (hour === 9 && minute >= 30) || 
           (hour >= 10 && hour < 11) || 
           (hour === 11 && minute < 30) ||
           (hour >= 13 && hour < 15);
  }
  // ... customize as needed
}
```

This skill covers the essential usage patterns for building financial market dashboards with the marketingdashboard project.
