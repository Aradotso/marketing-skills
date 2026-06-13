---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, script generation, and video creation with Claude/OpenAI
triggers:
  - automate content creation with AI research
  - generate marketing content from trending topics
  - create videos from written content automatically
  - build content pipeline with Claude and OpenAI
  - scrape news and generate social media posts
  - automate video rendering with Remotion
  - set up AI content automation workflow
  - generate multilingual marketing content
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **Marketing Pipeline Share** - an end-to-end automated content creation system that researches trending topics, generates scripts in multiple formats and languages, and renders videos automatically using Claude 3, OpenAI, and Remotion.

## What This Project Does

Marketing Pipeline Share is a TypeScript-based content automation pipeline that:

- **Auto-scans research sources** (TechCrunch, a16z, Twitter/X, LinkedIn) for trending topics in the last 24 hours
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Supports bilingual output** (English & Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for Reels, TikTok, Shorts
- **Provides Next.js interface** for scheduling and managing content

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
# or
pnpm install

# Copy environment template
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Social Media APIs
TWITTER_API_KEY=your_twitter_key
LINKEDIN_API_KEY=your_linkedin_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Visit `http://localhost:3000` to access the interface.

## Key Components & Usage

### 1. Research Module - Auto-Scan Trending Topics

```typescript
import { ResearchEngine } from '@/lib/research/engine';

// Initialize research engine
const research = new ResearchEngine({
  apiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h'
});

// Scan for trending topics
const trends = await research.scanTrends({
  keyword: 'AI automation',
  limit: 10,
  language: 'en'
});

console.log(trends);
// Returns: Array of trending articles with metadata
```

### 2. Content Generation - AI Writer

```typescript
import { ContentGenerator } from '@/lib/ai/generator';
import Anthropic from '@anthropic-ai/sdk';

// Initialize AI client
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const generator = new ContentGenerator({
  client: anthropic,
  model: 'claude-3-5-sonnet-20241022'
});

// Generate content in multiple formats
const content = await generator.create({
  topic: 'AI Content Automation',
  format: 'toplist', // or 'pov', 'case-study', 'how-to'
  language: 'en', // or 'vi' for Vietnamese
  tone: 'professional', // or 'friendly', 'humorous'
  sources: trends, // from research module
  includeStats: true
});

console.log(content.title);
console.log(content.body);
console.log(content.metadata);
```

### 3. Bilingual Content Generation

```typescript
import { BilingualGenerator } from '@/lib/ai/bilingual';

const bilingualGen = new BilingualGenerator({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  openaiKey: process.env.OPENAI_API_KEY
});

// Generate content in both languages simultaneously
const dualContent = await bilingualGen.generatePair({
  topic: 'Marketing Automation Trends 2026',
  format: 'case-study',
  primaryLang: 'en',
  secondaryLang: 'vi',
  tone: 'expert'
});

console.log(dualContent.english);
console.log(dualContent.vietnamese);
```

### 4. Video Rendering with Remotion

```typescript
import { VideoRenderer } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

const renderer = new VideoRenderer({
  compositionId: 'ContentVideo',
  outputFormat: 'mp4'
});

// Prepare video from content
const videoConfig = await renderer.prepare({
  content: content.body,
  title: content.title,
  style: 'minimal', // or 'dynamic', 'infographic'
  aspectRatio: '9:16', // for Reels/TikTok/Shorts
  duration: 30 // seconds
});

// Render video
const output = await renderer.render({
  config: videoConfig,
  outputPath: './output/video.mp4',
  quality: 'high'
});

console.log(`Video rendered: ${output.path}`);
```

### 5. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  openaiKey: process.env.OPENAI_API_KEY,
  rapidApiKey: process.env.RAPIDAPI_KEY
});

// Run full automation pipeline
const result = await pipeline.execute({
  keyword: 'AI automation',
  formats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  generateVideo: true,
  videoAspectRatio: '9:16',
  schedule: {
    platform: 'facebook',
    time: new Date('2026-06-15T10:00:00')
  }
});

console.log(`Generated ${result.articles.length} articles`);
console.log(`Rendered ${result.videos.length} videos`);
console.log(`Scheduled ${result.scheduled.length} posts`);
```

## API Routes (Next.js)

### Generate Content API

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentPipeline } from '@/lib/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, language } = req.body;

  try {
    const pipeline = new ContentPipeline({
      anthropicKey: process.env.ANTHROPIC_API_KEY,
      openaiKey: process.env.OPENAI_API_KEY,
      rapidApiKey: process.env.RAPIDAPI_KEY
    });

    const result = await pipeline.execute({
      keyword,
      formats: [format],
      languages: [language],
      generateVideo: false
    });

    res.status(200).json(result);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Research Trends API

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ResearchEngine } from '@/lib/research/engine';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keyword, sources, limit = 10 } = req.query;

  const research = new ResearchEngine({
    apiKey: process.env.RAPIDAPI_KEY,
    sources: (sources as string)?.split(',') || ['techcrunch', 'a16z'],
    timeRange: '24h'
  });

  const trends = await research.scanTrends({
    keyword: keyword as string,
    limit: parseInt(limit as string)
  });

  res.status(200).json(trends);
}
```

## Configuration

### Content Formats

Available content formats in the system:

```typescript
type ContentFormat = 
  | 'toplist'      // Top 10, Top 5 style lists
  | 'pov'          // Point of view / opinion pieces
  | 'case-study'   // In-depth case analysis
  | 'how-to';      // Tutorial style content

type ContentTone = 
  | 'professional' // Expert, authoritative
  | 'friendly'     // Casual, approachable
  | 'humorous';    // Light, entertaining
```

### Research Sources Configuration

```typescript
// lib/config/sources.ts
export const RESEARCH_SOURCES = {
  techcrunch: {
    endpoint: 'https://techcrunch.com/wp-json/wp/v2/posts',
    rateLimit: 100, // requests per hour
    parser: 'wordpress'
  },
  a16z: {
    endpoint: 'https://a16z.com/feed',
    rateLimit: 50,
    parser: 'rss'
  },
  twitter: {
    endpoint: 'https://api.twitter.com/2/tweets/search/recent',
    rateLimit: 180,
    parser: 'twitter-api-v2'
  },
  linkedin: {
    endpoint: 'https://api.linkedin.com/v2/posts',
    rateLimit: 100,
    parser: 'linkedin'
  }
};
```

### Video Templates

```typescript
// lib/video/templates.ts
export const VIDEO_TEMPLATES = {
  minimal: {
    backgroundColor: '#ffffff',
    textColor: '#000000',
    font: 'Inter',
    animation: 'fade'
  },
  dynamic: {
    backgroundColor: '#0066ff',
    textColor: '#ffffff',
    font: 'Poppins',
    animation: 'slide'
  },
  infographic: {
    backgroundColor: '#f5f5f5',
    textColor: '#333333',
    font: 'Roboto',
    animation: 'zoom',
    includeCharts: true
  }
};
```

## Common Patterns

### Pattern 1: Daily Trend Scanner

```typescript
// scripts/daily-scan.ts
import { ContentPipeline } from '@/lib/pipeline';
import cron from 'node-cron';

const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  openaiKey: process.env.OPENAI_API_KEY,
  rapidApiKey: process.env.RAPIDAPI_KEY
});

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('Starting daily trend scan...');
  
  const keywords = ['AI', 'Marketing', 'Automation', 'SaaS'];
  
  for (const keyword of keywords) {
    const result = await pipeline.execute({
      keyword,
      formats: ['toplist', 'pov'],
      languages: ['en', 'vi'],
      generateVideo: true,
      videoAspectRatio: '9:16'
    });
    
    console.log(`Generated ${result.articles.length} pieces for "${keyword}"`);
  }
});
```

### Pattern 2: Batch Content Generation

```typescript
import { ContentGenerator } from '@/lib/ai/generator';
import Anthropic from '@anthropic-ai/sdk';

async function batchGenerate(topics: string[]) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const generator = new ContentGenerator({
    client: anthropic,
    model: 'claude-3-5-sonnet-20241022'
  });
  
  const results = await Promise.all(
    topics.map(topic => 
      generator.create({
        topic,
        format: 'how-to',
        language: 'en',
        tone: 'professional',
        includeStats: true
      })
    )
  );
  
  return results;
}

// Usage
const articles = await batchGenerate([
  'How to Automate Social Media',
  'How to Use AI for Content Creation',
  'How to Build Marketing Funnels'
]);
```

### Pattern 3: Custom Format Template

```typescript
import { ContentGenerator } from '@/lib/ai/generator';

// Extend with custom format
class CustomContentGenerator extends ContentGenerator {
  async generateProductReview(product: string, sources: any[]) {
    return this.create({
      topic: `${product} Review`,
      format: 'case-study',
      language: 'en',
      tone: 'professional',
      sources,
      customPrompt: `
        Create a comprehensive product review that includes:
        - Key features and benefits
        - Pricing comparison
        - User testimonials
        - Pros and cons analysis
        - Final recommendation
      `
    });
  }
}
```

## Troubleshooting

### Issue: API Rate Limiting

```typescript
// lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(5); // Max 5 concurrent requests

async function fetchWithRateLimit(urls: string[]) {
  return Promise.all(
    urls.map(url => 
      limit(() => fetch(url).then(r => r.json()))
    )
  );
}
```

### Issue: Video Rendering Timeout

```typescript
// Increase timeout for long videos
const output = await renderer.render({
  config: videoConfig,
  outputPath: './output/video.mp4',
  quality: 'high',
  timeoutInMilliseconds: 300000 // 5 minutes
});
```

### Issue: Claude API Errors

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
  maxRetries: 3, // Retry on failure
  timeout: 60000 // 60 second timeout
});

// Handle rate limits gracefully
try {
  const response = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{ role: 'user', content: prompt }]
  });
} catch (error) {
  if (error.status === 429) {
    console.log('Rate limited, waiting...');
    await new Promise(resolve => setTimeout(resolve, 60000));
    // Retry logic here
  }
  throw error;
}
```

### Issue: Memory Errors During Video Rendering

```typescript
// Use streaming for large videos
import { renderMedia, selectComposition } from '@remotion/renderer';

const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: 'ContentVideo'
});

await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: './output/video.mp4',
  chromiumOptions: {
    headless: true,
    args: ['--disable-dev-shm-usage', '--no-sandbox']
  }
});
```

## Testing

```typescript
// __tests__/pipeline.test.ts
import { ContentPipeline } from '@/lib/pipeline';

describe('ContentPipeline', () => {
  it('should generate content from keyword', async () => {
    const pipeline = new ContentPipeline({
      anthropicKey: process.env.ANTHROPIC_API_KEY,
      openaiKey: process.env.OPENAI_API_KEY,
      rapidApiKey: process.env.RAPIDAPI_KEY
    });
    
    const result = await pipeline.execute({
      keyword: 'AI automation',
      formats: ['toplist'],
      languages: ['en'],
      generateVideo: false
    });
    
    expect(result.articles).toHaveLength(1);
    expect(result.articles[0].title).toBeTruthy();
    expect(result.articles[0].body).toBeTruthy();
  });
});
```

Run tests:
```bash
npm run test
# or
yarn test
```

## Additional Resources

- Refer to `HUONG_DAN_CAI_DAT.md` in the project root for detailed setup instructions
- Check `/examples` folder for more usage patterns
- See `/docs` for API reference documentation
