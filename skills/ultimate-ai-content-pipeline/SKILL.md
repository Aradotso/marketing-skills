---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an AI content pipeline for marketing
  - generate blog posts and videos automatically
  - use Claude and OpenAI for content research
  - create marketing content with Remotion
  - build an automated content workflow
  - scrape news and generate articles with AI
  - turn blog posts into videos automatically
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete TypeScript-based automation system that creates content from research to video generation. The pipeline crawls fresh news from sources like TechCrunch and Twitter, generates articles in multiple formats using Claude/OpenAI, and automatically renders videos with Remotion.

## What It Does

- **Auto-Research**: Crawls real-time data from news sources, social media, and blogs (last 24h)
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multi-language Support**: Generates content in English and Vietnamese with customizable tone
- **Video Rendering**: Automatically converts articles to videos and infographics using Remotion
- **Platform Optimization**: Exports videos optimized for Reels, TikTok, and YouTube Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── research/    # Web scraping & data collection
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research & Data Collection

```typescript
import { scrapeNews } from '@/lib/research/scraper';
import { analyzeContent } from '@/lib/ai/analyzer';

// Scrape fresh content from news sources
const newsData = await scrapeNews({
  sources: ['techcrunch', 'twitter', 'linkedin'],
  keywords: ['AI', 'marketing automation'],
  timeRange: '24h',
  maxResults: 50
});

// Analyze and extract insights
const insights = await analyzeContent(newsData, {
  extractKeyPoints: true,
  identifyTrends: true,
  includeStatistics: true
});
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateArticle(topic: string, format: 'toplist' | 'pov' | 'case-study' | 'how-to') {
  const prompt = `Create a ${format} article about ${topic} based on these insights: ${JSON.stringify(insights)}`;
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}

// Generate content in multiple formats
const article = await generateArticle('AI Marketing Trends', 'toplist');
```

### 3. Multi-language Content Generation

```typescript
import { OpenAI } from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateBilingual(content: string, tone: 'professional' | 'friendly' | 'humorous') {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a bilingual content creator. Generate both English and Vietnamese versions with a ${tone} tone.`
      },
      {
        role: 'user',
        content: `Create bilingual versions of this content: ${content}`
      }
    ]
  });

  const result = completion.choices[0].message.content;
  return parseLanguages(result); // Returns { en: string, vi: string }
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(articleData: ArticleData) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: articleData.title,
      keyPoints: articleData.keyPoints,
      statistics: articleData.statistics,
      style: 'modern'
    }
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `output/${articleData.slug}.mp4`,
    inputProps: composition.inputProps
  });
}
```

### 5. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  aiProvider: 'claude', // or 'openai'
  languages: ['en', 'vi'],
  formats: ['article', 'video', 'infographic']
});

// Run full automation
async function runPipeline(keyword: string) {
  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'twitter', 'a16z'],
    depth: 'detailed'
  });

  // Step 2: Generate content
  const content = await pipeline.generateContent({
    research,
    format: 'toplist',
    tone: 'professional',
    length: 'medium'
  });

  // Step 3: Create visuals
  const media = await pipeline.generateMedia({
    content,
    types: ['featured-image', 'infographic', 'video'],
    platforms: ['instagram', 'tiktok', 'youtube']
  });

  // Step 4: Export
  return pipeline.export({
    content,
    media,
    outputDir: './output',
    formats: ['markdown', 'html', 'json']
  });
}

// Execute
const result = await runPipeline('AI content automation');
console.log('Generated:', result.files);
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/content/generator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    const content = await generateContent({
      keyword,
      format,
      language,
      aiProvider: process.env.AI_PROVIDER || 'claude'
    });

    return NextResponse.json({ success: true, content });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Render Video Endpoint

```typescript
// app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { contentId, platform } = await request.json();

    const videoUrl = await renderVideo({
      contentId,
      platform,
      aspectRatio: platform === 'youtube' ? '16:9' : '9:16'
    });

    return NextResponse.json({ success: true, videoUrl });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video
npm run render -- --props='{"title":"My Video"}'

# Type checking
npm run type-check

# Lint code
npm run lint
```

## Common Patterns

### Custom Content Templates

```typescript
interface ContentTemplate {
  name: string;
  structure: string[];
  tone: string;
  length: number;
}

const templates: Record<string, ContentTemplate> = {
  toplist: {
    name: 'Top List',
    structure: ['intro', 'items', 'conclusion'],
    tone: 'informative',
    length: 1500
  },
  pov: {
    name: 'Point of View',
    structure: ['hook', 'opinion', 'evidence', 'counterpoint', 'conclusion'],
    tone: 'persuasive',
    length: 1200
  },
  caseStudy: {
    name: 'Case Study',
    structure: ['background', 'challenge', 'solution', 'results', 'takeaways'],
    tone: 'analytical',
    length: 2000
  }
};

function applyTemplate(data: any, templateName: string) {
  const template = templates[templateName];
  return generateFromTemplate(data, template);
}
```

### Rate Limiting & Caching

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.REDIS_URL,
  token: process.env.REDIS_TOKEN
});

async function cachedGeneration(key: string, generator: () => Promise<any>) {
  // Check cache
  const cached = await redis.get(key);
  if (cached) return cached;

  // Generate new
  const result = await generator();

  // Cache for 24 hours
  await redis.setex(key, 86400, JSON.stringify(result));

  return result;
}
```

## Troubleshooting

### AI API Errors

**Issue**: `429 Too Many Requests` from Claude/OpenAI

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
}
```

### Remotion Rendering Issues

**Issue**: Video rendering fails or times out

```typescript
// Increase timeout and add error handling
const renderConfig = {
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 120000, // 2 minutes
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  },
  onProgress: ({ progress }) => {
    console.log(`Rendering progress: ${(progress * 100).toFixed(2)}%`);
  }
};
```

### Memory Issues with Large Datasets

**Issue**: Out of memory when processing many articles

```typescript
// Process in batches
async function processBatch<T>(items: T[], batchSize: number, processor: (item: T) => Promise<any>) {
  const results = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(batch.map(processor));
    results.push(...batchResults);
    
    // Allow garbage collection
    if (global.gc) global.gc();
  }
  
  return results;
}
```

### Web Scraping Blocks

**Issue**: Getting blocked by websites during research

```typescript
// Add user agent rotation and delays
import { randomUserAgent } from '@/lib/utils/user-agents';

async function scrapeWithRetry(url: string) {
  const headers = {
    'User-Agent': randomUserAgent(),
    'Accept': 'text/html,application/xhtml+xml',
    'Accept-Language': 'en-US,en;q=0.9'
  };

  // Random delay between requests
  await new Promise(resolve => setTimeout(resolve, Math.random() * 2000 + 1000));

  const response = await fetch(url, { headers });
  return response.text();
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Cache research results** to avoid redundant API calls
3. **Batch process** when generating multiple pieces of content
4. **Monitor token usage** for AI providers to control costs
5. **Use TypeScript types** for all content structures
6. **Test video templates** before bulk rendering
7. **Implement error boundaries** in UI components
8. **Log all pipeline steps** for debugging
