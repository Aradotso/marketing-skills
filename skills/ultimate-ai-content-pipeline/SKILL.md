---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - generate articles and videos from keywords automatically
  - set up an AI content pipeline with research
  - create content workflow with Claude and OpenAI
  - automate social media content generation
  - build an AI-powered content factory
  - scrape news and generate marketing content
  - turn text into videos with Remotion
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a complete content automation system that takes a keyword input and produces research-backed articles in multiple formats and languages, plus auto-generated videos. It combines web scraping, AI content generation (Claude/OpenAI), and video rendering (Remotion) into a single pipeline.

## What It Does

The pipeline automates:
1. **Research**: Crawls recent content from TechCrunch, a16z, Twitter/X, LinkedIn
2. **Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Localization**: Generates both English and Vietnamese versions with adjustable tone
4. **Video Creation**: Renders infographics and short-form videos using Remotion for Reels/TikTok/Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPID_API_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/            # React components
├── lib/                   # Core utilities
│   ├── ai/               # AI integrations (Claude, OpenAI)
│   ├── scraper/          # Web scraping modules
│   └── video/            # Remotion video generation
├── remotion/             # Remotion video templates
└── public/               # Static assets
```

## Key API Usage Patterns

### 1. Research & Scraping

```typescript
import { scrapeRecentNews } from '@/lib/scraper/news-scraper';
import { analyzeContent } from '@/lib/ai/content-analyzer';

async function gatherResearch(keyword: string) {
  // Scrape from multiple sources
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  const rawData = await scrapeRecentNews({
    keyword,
    sources,
    timeframe: '24h'
  });

  // Analyze with AI
  const insights = await analyzeContent({
    data: rawData,
    provider: 'claude', // or 'openai'
    extractInsights: true,
    extractStats: true
  });

  return insights;
}
```

### 2. Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateArticle(
  keyword: string,
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const prompt = `
Based on this research about ${keyword}:
${JSON.stringify(research)}

Create a ${format} article with:
- Engaging headline
- Data-backed insights
- Clear structure
- SEO optimization
`;

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
```

### 3. Multi-Language Generation

```typescript
interface ContentConfig {
  keyword: string;
  format: string;
  languages: string[];
  tone: 'professional' | 'friendly' | 'humorous';
}

async function generateMultiLangContent(config: ContentConfig) {
  const { keyword, format, languages, tone } = config;
  
  const results = await Promise.all(
    languages.map(async (lang) => {
      const prompt = `
Write a ${format} article about ${keyword} in ${lang}.
Tone: ${tone}
Include recent data and trends.
`;

      const content = await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }]
      });

      return {
        language: lang,
        content: content.content[0].text
      };
    })
  );

  return results;
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  stats: { label: string; value: string }[];
  aspectRatio: '9:16' | '16:9' | '1:1'; // Reels/TikTok/Shorts
}

async function generateVideo(config: VideoConfig) {
  const compositionId = 'ContentVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: config,
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: config,
  });

  return outputLocation;
}
```

### 5. Complete Pipeline Integration

```typescript
interface PipelineInput {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: string[];
  generateVideo: boolean;
}

async function runContentPipeline(input: PipelineInput) {
  // Step 1: Research
  console.log('🔍 Gathering research...');
  const research = await gatherResearch(input.keyword);

  // Step 2: Generate articles
  console.log('✍️ Generating content...');
  const articles = await generateMultiLangContent({
    keyword: input.keyword,
    format: input.format,
    languages: input.languages,
    tone: 'professional'
  });

  // Step 3: Generate video (optional)
  let videoPath;
  if (input.generateVideo) {
    console.log('🎬 Rendering video...');
    const keyPoints = extractKeyPoints(articles[0].content);
    const stats = extractStats(research);
    
    videoPath = await generateVideo({
      title: input.keyword,
      keyPoints,
      stats,
      aspectRatio: '9:16'
    });
  }

  return {
    research,
    articles,
    video: videoPath
  };
}

// Helper functions
function extractKeyPoints(content: string): string[] {
  // Parse content and extract main points
  const lines = content.split('\n').filter(line => 
    line.trim().startsWith('-') || line.trim().startsWith('•')
  );
  return lines.slice(0, 5).map(line => line.replace(/^[-•]\s*/, ''));
}

function extractStats(research: any): Array<{label: string; value: string}> {
  // Extract statistical data from research
  return research.stats || [];
}
```

## CLI Commands

If the project includes CLI tools:

```bash
# Generate content from keyword
npm run generate -- --keyword "AI marketing" --format toplist

# Run full pipeline
npm run pipeline -- --keyword "ChatGPT trends" --video

# Scrape only
npm run scrape -- --sources techcrunch,a16z --keyword "startup funding"

# Render video from JSON
npm run render-video -- --input ./data/video-config.json
```

## Next.js API Routes

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { keyword, format, languages } = await request.json();

  try {
    const result = await runContentPipeline({
      keyword,
      format,
      languages,
      generateVideo: true
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

## Common Patterns

### Rate Limiting for AI APIs

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword =>
      limit(() => generateArticle(keyword, {}, 'toplist'))
    )
  );
  return results;
}
```

### Caching Research Results

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.REDIS_URL!,
  token: process.env.REDIS_TOKEN!,
});

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = await redis.get(cacheKey);
  
  if (cached) return cached;
  
  const fresh = await gatherResearch(keyword);
  await redis.setex(cacheKey, 3600, JSON.stringify(fresh)); // 1 hour cache
  
  return fresh;
}
```

### Error Handling & Retries

```typescript
async function generateWithRetry(
  fn: () => Promise<any>,
  maxRetries = 3
) {
  let lastError;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      console.log(`Attempt ${i + 1} failed, retrying...`);
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
  
  throw lastError;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
class RateLimitHandler {
  async callWithBackoff(fn: () => Promise<any>) {
    let delay = 1000;
    while (true) {
      try {
        return await fn();
      } catch (error) {
        if (error.status === 429) {
          await new Promise(resolve => setTimeout(resolve, delay));
          delay *= 2;
        } else {
          throw error;
        }
      }
    }
  }
}
```

### Remotion Rendering Issues

```bash
# Ensure Chrome is installed for headless rendering
npx remotion browser ensure

# Check Remotion config
npx remotion compositions
```

### Missing Environment Variables

```typescript
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPID_API_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing env vars: ${missing.join(', ')}`);
  }
}
```

## Best Practices

1. **Always cache research data** to avoid redundant API calls
2. **Use streaming responses** for real-time content generation feedback
3. **Implement proper error boundaries** in Next.js components
4. **Queue video rendering jobs** for heavy workloads (use Bull, BullMQ)
5. **Version your prompts** to track content quality improvements
6. **Monitor API costs** with logging and alerting

This skill enables AI coding agents to help developers build automated content creation workflows that scale from keyword input to published articles and videos.
