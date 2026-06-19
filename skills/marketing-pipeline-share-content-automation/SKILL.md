```markdown
---
name: marketing-pipeline-share-content-automation
description: AI-powered content pipeline for research, scriptwriting, posting, and video generation
triggers:
  - automate content research and video generation
  - generate AI content with automated research
  - create content pipeline with Claude and OpenAI
  - scrape trending news and generate articles
  - build automated marketing content system
  - generate multi-format content with AI
  - create videos from articles automatically
  - set up AI content automation pipeline
---

# Marketing Pipeline Share Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, an end-to-end automated content creation system that handles research, scriptwriting, multi-format article generation, and video rendering using Claude 3, OpenAI, and Remotion.

## What This Project Does

Marketing Pipeline Share is a complete content automation system that:

- **Auto-scrapes trending news** from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
- **Generates multi-format articles** (Toplist, POV, Case Study, How-to) in both English and Vietnamese
- **Creates videos and infographics** automatically using Remotion
- **Optimizes content** for multiple platforms (Reels, TikTok, Shorts)
- **Provides a Next.js interface** for managing the entire content pipeline

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
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key_here

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video rendering
npm run remotion
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # News scraping logic
│   │   └── video/       # Remotion video generation
│   └── utils/           # Helper functions
├── public/              # Static assets
├── remotion/            # Video templates
└── .env.local          # Environment variables
```

## Core API Patterns

### 1. Research & Scraping

```typescript
import { researchTrending } from '@/lib/scraper/trending';

// Scrape trending news by keyword
async function getTrendingContent(keyword: string) {
  const results = await researchTrending({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    limit: 10
  });
  
  return results;
}

// Example usage
const aiNews = await getTrendingContent('artificial intelligence');
console.log(aiNews); // Array of trending articles with metadata
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

// Generate article using Claude or OpenAI
async function createArticle(topic: string, format: string) {
  const article = await generateContent({
    topic,
    format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language: ['en', 'vi'], // Generate both languages
    tone: 'professional', // 'professional' | 'friendly' | 'humorous'
    researchData: await getTrendingContent(topic),
    aiProvider: 'claude' // or 'openai'
  });
  
  return article;
}

// Example: Generate bilingual toplist
const article = await createArticle(
  'Top AI Tools for Marketing 2024',
  'toplist'
);
```

### 3. Multi-Language Generation

```typescript
import { Anthropic } from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateBilingualContent(prompt: string, research: any[]) {
  const systemPrompt = `You are an expert content creator. 
Generate content in both English and Vietnamese.
Use the following research data: ${JSON.stringify(research)}`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `${systemPrompt}\n\n${prompt}`
      }
    ]
  });

  return message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

// Render video from article content
async function generateVideo(articleContent: any) {
  const compositionId = 'ArticleVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: articleContent.title,
      content: articleContent.content,
      language: articleContent.language,
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${articleContent.slug}.mp4`,
    inputProps: {
      title: articleContent.title,
      content: articleContent.content,
    },
  });

  return `out/${articleContent.slug}.mp4`;
}
```

## Common Workflows

### Complete Content Pipeline

```typescript
import { researchTrending } from '@/lib/scraper/trending';
import { generateContent } from '@/lib/ai/content-generator';
import { renderVideo } from '@/lib/video/renderer';
import { publishToPage } from '@/lib/publisher';

async function runContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Researching trending content...');
  const research = await researchTrending({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h'
  });

  // Step 2: Generate article (bilingual)
  console.log('✍️ Generating article...');
  const article = await generateContent({
    topic: keyword,
    format: 'case-study',
    language: ['en', 'vi'],
    tone: 'professional',
    researchData: research,
    aiProvider: 'claude'
  });

  // Step 3: Create video
  console.log('🎬 Rendering video...');
  const videoPath = await renderVideo({
    content: article,
    platform: 'reels', // 'reels' | 'tiktok' | 'shorts'
    aspectRatio: '9:16'
  });

  // Step 4: Auto-publish (optional)
  console.log('📤 Publishing...');
  await publishToPage({
    article,
    video: videoPath,
    platforms: ['facebook', 'instagram']
  });

  return { article, videoPath };
}

// Execute pipeline
runContentPipeline('AI Marketing Automation').then(result => {
  console.log('✅ Pipeline complete!', result);
});
```

### Custom Format Templates

```typescript
// Create custom content format
const formatTemplates = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    itemCount: 10,
    includeRankings: true
  },
  pov: {
    structure: ['hook', 'perspective', 'evidence', 'conclusion'],
    tone: 'opinionated'
  },
  caseStudy: {
    structure: ['problem', 'solution', 'results', 'takeaways'],
    includeMetrics: true
  },
  howTo: {
    structure: ['overview', 'steps', 'tips', 'conclusion'],
    stepByStep: true
  }
};

async function generateWithTemplate(topic: string, template: string) {
  const config = formatTemplates[template];
  
  return await generateContent({
    topic,
    format: template,
    structure: config.structure,
    ...config
  });
}
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTrending } from '@/lib/scraper/trending';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeRange } = await request.json();

  try {
    const results = await researchTrending({
      keyword,
      sources,
      timeRange: timeRange || '24h'
    });

    return NextResponse.json({ success: true, data: results });
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
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  const { topic, format, language, tone, research } = await request.json();

  try {
    const article = await generateContent({
      topic,
      format,
      language,
      tone,
      researchData: research,
      aiProvider: process.env.AI_PROVIDER || 'claude'
    });

    return NextResponse.json({ success: true, article });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  const { content, platform, aspectRatio } = await request.json();

  try {
    const videoPath = await renderVideo({
      content,
      platform,
      aspectRatio
    });

    return NextResponse.json({ 
      success: true, 
      videoUrl: `/videos/${videoPath}` 
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Configuration Best Practices

### AI Provider Selection

```typescript
// lib/ai/provider-config.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7,
    bestFor: ['long-form', 'analysis', 'multilingual']
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.7,
    bestFor: ['creative', 'technical', 'concise']
  }
};

// Select provider based on content type
export function selectProvider(contentType: string) {
  const claudeTypes = ['case-study', 'pov', 'analysis'];
  return claudeTypes.includes(contentType) ? 'claude' : 'openai';
}
```

### Rate Limiting

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private requests: Map<string, number[]> = new Map();

  async checkLimit(key: string, maxRequests: number, windowMs: number) {
    const now = Date.now();
    const requests = this.requests.get(key) || [];
    
    const recentRequests = requests.filter(time => now - time < windowMs);
    
    if (recentRequests.length >= maxRequests) {
      throw new Error('Rate limit exceeded');
    }
    
    recentRequests.push(now);
    this.requests.set(key, recentRequests);
  }
}

export const limiter = new RateLimiter();

// Usage
await limiter.checkLimit('research', 10, 60000); // 10 requests per minute
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys on startup
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }
}

validateEnv();
```

### Scraping Failures

```typescript
// Implement retry logic
async function scrapeWithRetry(url: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fetch(url, {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
          'User-Agent': 'Mozilla/5.0'
        }
      });
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}
```

### Video Rendering Memory Issues

```typescript
// Optimize Remotion rendering
export const renderConfig = {
  // Reduce concurrency for low-memory environments
  concurrency: process.env.NODE_ENV === 'production' ? 1 : 4,
  
  // Use lower quality for previews
  quality: process.env.PREVIEW_MODE ? 50 : 90,
  
  // Cleanup after render
  cleanup: true
};
```

### Content Quality Issues

```typescript
// Validate generated content
function validateArticle(article: any) {
  const checks = {
    hasTitle: !!article.title && article.title.length > 10,
    hasContent: !!article.content && article.content.length > 500,
    hasBothLanguages: article.en && article.vi,
    hasMetadata: !!article.metadata
  };

  const failed = Object.entries(checks)
    .filter(([_, passed]) => !passed)
    .map(([check]) => check);

  if (failed.length > 0) {
    throw new Error(`Article validation failed: ${failed.join(', ')}`);
  }

  return true;
}
```

## Performance Optimization

```typescript
// Cache research results
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.REDIS_URL,
  token: process.env.REDIS_TOKEN,
});

async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  
  if (cached) {
    return JSON.parse(cached);
  }

  const fresh = await researchTrending({ keyword });
  await redis.set(`research:${keyword}`, JSON.stringify(fresh), {
    ex: 3600 // 1 hour cache
  });

  return fresh;
}
```

This skill provides comprehensive coverage of the Marketing Pipeline Share system for automated content creation, research, and video generation.
```
