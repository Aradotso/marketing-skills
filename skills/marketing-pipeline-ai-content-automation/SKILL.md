---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - generate video from text using remotion
  - crawl news and create content automatically
  - use marketing pipeline for content automation
  - setup AI content generation workflow
  - create multilingual content with Claude and OpenAI
  - automate social media video creation
  - research and generate articles with AI
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the Ultimate AI Content Pipeline - an automated content creation system that handles research, scriptwriting, multilingual article generation, and video rendering using Claude, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline is an all-in-one TypeScript/Next.js system that:

- **Auto-crawls** recent news from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
- **Generates content** in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
- **Produces bilingual** articles (English & Vietnamese) with customizable tone
- **Renders videos** and infographics automatically using Remotion
- **Exports** optimized video for Reels, TikTok, Shorts

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

# Setup environment variables
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Data Sources
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database
DATABASE_URL=postgresql://user:password@localhost:5432/content_db

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # News crawlers
│   │   ├── generators/  # Content generators
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key APIs and Usage Patterns

### 1. Content Research & Crawling

```typescript
import { NewsCrawler } from '@/lib/crawlers/news-crawler';

interface CrawlConfig {
  sources: string[];
  timeframe: number; // hours
  keywords?: string[];
}

async function researchTopic(keyword: string) {
  const crawler = new NewsCrawler({
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: 24,
    keywords: [keyword],
  });

  const articles = await crawler.crawl();
  
  // Extract insights
  const insights = await crawler.extractInsights(articles);
  
  return {
    articles,
    insights,
    sources: articles.map(a => a.source),
  };
}
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/generators/content-generator';
import { Anthropic } from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  languages: ('en' | 'vi')[];
}

async function generateContent(
  topic: string,
  research: any[],
  config: ContentConfig
) {
  const generator = new ContentGenerator({
    ai: anthropic,
    model: 'claude-3-5-sonnet-20241022',
  });

  const content = await generator.create({
    topic,
    research,
    format: config.format,
    tone: config.tone,
    languages: config.languages,
  });

  return content;
}

// Example usage
const content = await generateContent(
  'AI Marketing Trends 2024',
  researchData,
  {
    format: 'toplist',
    tone: 'expert',
    languages: ['en', 'vi'],
  }
);
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer creating engaging articles.',
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    temperature: 0.7,
    max_tokens: 2000,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  duration: number;
  format: 'reels' | 'tiktok' | 'shorts';
}

async function generateVideo(content: any, config: VideoConfig) {
  const bundled = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints,
      theme: 'modern',
    },
  });

  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints,
    },
  });

  return outputPath;
}

// Example usage
const videoPath = await generateVideo(article, {
  title: 'Top 5 AI Marketing Tools',
  keyPoints: article.sections.map(s => s.summary),
  duration: 60,
  format: 'reels',
});
```

### 5. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude', // or 'openai'
    videoEnabled: true,
    languages: ['en', 'vi'],
  });

  try {
    // Step 1: Research
    const research = await pipeline.research(keyword);
    console.log(`Found ${research.articles.length} articles`);

    // Step 2: Generate content
    const content = await pipeline.generate({
      topic: keyword,
      research: research.insights,
      format: 'toplist',
      tone: 'expert',
    });

    // Step 3: Create video
    const video = await pipeline.renderVideo({
      content: content.en, // Use English version
      duration: 60,
      format: 'reels',
    });

    // Step 4: Save and return
    const result = await pipeline.save({
      content,
      video,
      metadata: {
        keyword,
        sources: research.sources,
        createdAt: new Date(),
      },
    });

    return result;
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

## Next.js API Routes

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { NewsCrawler } from '@/lib/crawlers/news-crawler';

export async function POST(request: NextRequest) {
  try {
    const { keyword, timeframe = 24 } = await request.json();

    const crawler = new NewsCrawler({
      sources: ['techcrunch', 'a16z'],
      timeframe,
      keywords: [keyword],
    });

    const results = await crawler.crawl();

    return NextResponse.json({
      success: true,
      data: results,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentGenerator } from '@/lib/generators/content-generator';
import { Anthropic } from '@anthropic-ai/sdk';

export async function POST(request: NextRequest) {
  try {
    const { topic, research, format, tone, languages } = await request.json();

    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });

    const generator = new ContentGenerator({
      ai: anthropic,
      model: 'claude-3-5-sonnet-20241022',
    });

    const content = await generator.create({
      topic,
      research,
      format,
      tone,
      languages,
    });

    return NextResponse.json({
      success: true,
      data: content,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Configuration

### AI Provider Configuration

```typescript
// src/config/ai.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7,
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4000,
    temperature: 0.7,
  },
};

// Usage
import { aiConfig } from '@/config/ai';

const response = await anthropic.messages.create({
  model: aiConfig.claude.model,
  max_tokens: aiConfig.claude.maxTokens,
  temperature: aiConfig.claude.temperature,
  messages: [...],
});
```

### Video Templates Configuration

```typescript
// remotion/config.ts
export const videoFormats = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
  },
  shorts: {
    width: 1080,
    height: 1920,
    fps: 30,
  },
  landscape: {
    width: 1920,
    height: 1080,
    fps: 30,
  },
};
```

## Common Patterns

### Error Handling

```typescript
import { PipelineError } from '@/lib/errors';

async function safeGenerate(topic: string) {
  try {
    return await generateContent(topic);
  } catch (error) {
    if (error instanceof PipelineError) {
      console.error(`Pipeline error at ${error.stage}:`, error.message);
      // Retry logic or fallback
      return await fallbackGenerate(topic);
    }
    throw error;
  }
}
```

### Rate Limiting

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  windowMs: 60000, // 1 minute
});

async function generateWithLimit(topic: string) {
  await limiter.acquire();
  
  try {
    return await generateContent(topic);
  } finally {
    limiter.release();
  }
}
```

### Caching Research Results

```typescript
import { redis } from '@/lib/redis';

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  
  // Check cache
  const cached = await redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached);
  }

  // Fetch new data
  const research = await researchTopic(keyword);
  
  // Cache for 1 hour
  await redis.setex(cacheKey, 3600, JSON.stringify(research));
  
  return research;
}
```

## Running the Application

```bash
# Development
npm run dev

# Build
npm run build

# Production
npm run start

# Remotion Studio (for video editing)
npm run remotion:studio
```

## Troubleshooting

### API Key Issues

**Problem:** `Error: API key not found`

**Solution:** Ensure environment variables are set correctly:

```bash
# Check if variables are loaded
echo $ANTHROPIC_API_KEY
echo $OPENAI_API_KEY

# Restart dev server after changing .env.local
npm run dev
```

### Video Rendering Fails

**Problem:** Remotion rendering timeout or errors

**Solution:**

```typescript
// Increase timeout
await renderMedia({
  composition,
  serveUrl: bundled,
  timeoutInMilliseconds: 120000, // 2 minutes
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox'],
  },
});
```

### Rate Limiting from APIs

**Problem:** Too many requests errors

**Solution:** Implement exponential backoff:

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const delay = Math.pow(2, i) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries reached');
}
```

### Multilingual Content Issues

**Problem:** Poor translation quality

**Solution:** Use native prompts per language:

```typescript
const prompts = {
  en: 'Write a professional marketing article about {topic}...',
  vi: 'Viết một bài viết marketing chuyên nghiệp về {topic}...',
};

// Generate separately
const englishContent = await generate(prompts.en.replace('{topic}', topic));
const vietnameseContent = await generate(prompts.vi.replace('{topic}', topic));
```

## Best Practices

1. **Always validate crawled data** before passing to AI
2. **Use TypeScript types** for all content structures
3. **Cache expensive operations** (research, AI generation)
4. **Implement proper error boundaries** in React components
5. **Monitor API usage** to avoid rate limits
6. **Version your video templates** for consistency
7. **Test prompts extensively** before production use
