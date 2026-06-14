---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation with Claude/OpenAI
triggers:
  - automate content creation with AI research and video generation
  - set up marketing content pipeline with Claude and OpenAI
  - create automated content workflow from research to social media
  - generate videos automatically from written content
  - build AI content automation system with Remotion
  - scrape news and generate marketing content automatically
  - automate social media content with AI pipeline
  - create multi-format content with AI research and video rendering
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive AI-powered content automation system that handles the entire content creation lifecycle: from researching trending topics across news sources (TechCrunch, a16z, Twitter, LinkedIn), to generating multi-format written content in multiple languages, to rendering videos automatically with Remotion. Built with TypeScript and Next.js, it integrates Claude 3, OpenAI, and various content APIs to create a hands-free content production pipeline.

**Key capabilities:**
- Auto-crawl and analyze news from multiple sources (last 24h)
- Generate content in multiple formats (listicles, POV, case studies, how-to)
- Bilingual support (English/Vietnamese) with customizable tone
- Automatic video/infographic rendering via Remotion
- Multi-platform optimization (Reels, TikTok, Shorts)

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
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Database (if applicable)
DATABASE_URL=your_database_connection

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
NODE_ENV=development
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── research/    # Content research modules
│   │   ├── generators/  # Content generators
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript definitions
│   └── utils/           # Utility functions
├── remotion/            # Remotion video compositions
├── public/              # Static assets
└── package.json
```

## Core API Usage

### 1. Research & Content Scraping

```typescript
import { ResearchService } from '@/lib/research/ResearchService';

// Initialize research service
const researchService = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY,
  twitterToken: process.env.TWITTER_BEARER_TOKEN,
});

// Scan for trending topics
const trendingTopics = await researchService.scanTrends({
  keyword: 'AI marketing',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h',
  limit: 10,
});

// Extract insights from research
const insights = await researchService.extractInsights(trendingTopics);

console.log(insights);
// {
//   topStories: [...],
//   keyInsights: [...],
//   dataPoints: [...],
//   trends: [...]
// }
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/generators/ContentGenerator';
import { OpenAI } from 'openai';
import Anthropic from '@anthropic-ai/sdk';

// Initialize AI providers
const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

const claude = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const generator = new ContentGenerator({
  openai,
  claude,
});

// Generate multi-format content
const content = await generator.generate({
  topic: 'AI-powered marketing automation trends 2026',
  format: 'toplist', // 'toplist' | 'pov' | 'case-study' | 'how-to'
  language: 'en', // 'en' | 'vi' | 'both'
  tone: 'professional', // 'professional' | 'friendly' | 'humorous'
  research: insights,
  provider: 'claude', // 'claude' | 'openai'
});

console.log(content);
// {
//   title: "Top 10 AI Marketing Automation Trends in 2026",
//   body: "...",
//   metadata: { wordCount: 1500, readTime: 6 },
//   language: "en"
// }
```

### 3. Bilingual Content Generation

```typescript
// Generate content in both languages
const bilingualContent = await generator.generate({
  topic: 'AI marketing automation',
  format: 'how-to',
  language: 'both',
  tone: 'friendly',
  research: insights,
});

console.log(bilingualContent);
// {
//   en: { title: "...", body: "...", ... },
//   vi: { title: "...", body: "...", ... }
// }
```

### 4. Video Generation with Remotion

```typescript
import { VideoRenderer } from '@/lib/video/VideoRenderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

const videoRenderer = new VideoRenderer();

// Render video from content
const videoConfig = {
  content: content,
  platform: 'reels', // 'reels' | 'tiktok' | 'shorts'
  duration: 60, // seconds
  style: 'minimal', // 'minimal' | 'dynamic' | 'infographic'
};

const video = await videoRenderer.render(videoConfig);

console.log(video);
// {
//   url: "https://...",
//   duration: 60,
//   format: "mp4",
//   resolution: "1080x1920"
// }
```

### 5. Complete Pipeline Execution

```typescript
import { ContentPipeline } from '@/lib/ContentPipeline';

const pipeline = new ContentPipeline({
  openaiKey: process.env.OPENAI_API_KEY,
  claudeKey: process.env.ANTHROPIC_API_KEY,
  rapidApiKey: process.env.RAPIDAPI_KEY,
});

// Run full pipeline
const result = await pipeline.execute({
  keyword: 'AI content marketing',
  contentFormat: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  platforms: ['reels', 'tiktok'],
});

console.log(result);
// {
//   research: { ... },
//   content: { en: {...}, vi: {...} },
//   videos: [
//     { platform: 'reels', url: '...', ... },
//     { platform: 'tiktok', url: '...', ... }
//   ],
//   metadata: { totalTime: 45000, ... }
// }
```

## Next.js API Routes

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchService } from '@/lib/research/ResearchService';

export async function POST(req: NextRequest) {
  const { keyword, sources, timeframe } = await req.json();
  
  const researchService = new ResearchService({
    rapidApiKey: process.env.RAPIDAPI_KEY!,
    twitterToken: process.env.TWITTER_BEARER_TOKEN!,
  });
  
  try {
    const trends = await researchService.scanTrends({
      keyword,
      sources,
      timeframe,
      limit: 10,
    });
    
    return NextResponse.json({ success: true, data: trends });
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
import { ContentGenerator } from '@/lib/generators/ContentGenerator';
import { OpenAI } from 'openai';
import Anthropic from '@anthropic-ai/sdk';

export async function POST(req: NextRequest) {
  const { topic, format, language, tone, research, provider } = await req.json();
  
  const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY! });
  const claude = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY! });
  
  const generator = new ContentGenerator({ openai, claude });
  
  try {
    const content = await generator.generate({
      topic,
      format,
      language,
      tone,
      research,
      provider,
    });
    
    return NextResponse.json({ success: true, data: content });
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
// src/app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { VideoRenderer } from '@/lib/video/VideoRenderer';

export async function POST(req: NextRequest) {
  const { content, platform, duration, style } = await req.json();
  
  const videoRenderer = new VideoRenderer();
  
  try {
    const video = await videoRenderer.render({
      content,
      platform,
      duration,
      style,
    });
    
    return NextResponse.json({ success: true, data: video });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Client-Side Usage

```typescript
// Example React component
'use client';

import { useState } from 'react';

export default function ContentGeneratorForm() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async (keyword: string) => {
    setLoading(true);
    
    try {
      // Step 1: Research
      const researchRes = await fetch('/api/research', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          sources: ['techcrunch', 'a16z', 'twitter'],
          timeframe: '24h',
        }),
      });
      const { data: research } = await researchRes.json();
      
      // Step 2: Generate content
      const contentRes = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          topic: keyword,
          format: 'toplist',
          language: 'both',
          tone: 'professional',
          research,
          provider: 'claude',
        }),
      });
      const { data: content } = await contentRes.json();
      
      // Step 3: Render video
      const videoRes = await fetch('/api/render-video', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          content: content.en,
          platform: 'reels',
          duration: 60,
          style: 'minimal',
        }),
      });
      const { data: video } = await videoRes.json();
      
      setResult({ research, content, video });
    } catch (error) {
      console.error('Pipeline error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <button onClick={() => handleGenerate('AI marketing')}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      {result && (
        <div>
          <h3>Content Generated!</h3>
          <pre>{JSON.stringify(result, null, 2)}</pre>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Custom Content Format

```typescript
import { ContentGenerator } from '@/lib/generators/ContentGenerator';

// Define custom format template
const customFormat = {
  name: 'product-review',
  structure: [
    'introduction',
    'features',
    'pros-and-cons',
    'pricing',
    'conclusion',
  ],
  minWordCount: 1200,
};

const generator = new ContentGenerator({ openai, claude });

const content = await generator.generateWithTemplate({
  topic: 'New AI marketing tool review',
  template: customFormat,
  language: 'en',
  tone: 'professional',
  research: insights,
});
```

### Batch Content Generation

```typescript
const keywords = [
  'AI content marketing',
  'Social media automation',
  'Video marketing trends',
];

const batchResults = await Promise.all(
  keywords.map(async (keyword) => {
    const research = await researchService.scanTrends({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h',
    });
    
    const content = await generator.generate({
      topic: keyword,
      format: 'toplist',
      language: 'both',
      research,
    });
    
    return { keyword, content };
  })
);
```

### Schedule Automated Posts

```typescript
import { ContentPipeline } from '@/lib/ContentPipeline';
import { SocialMediaScheduler } from '@/lib/schedulers/SocialMediaScheduler';

const pipeline = new ContentPipeline({ /* ... */ });
const scheduler = new SocialMediaScheduler();

// Generate and schedule
const result = await pipeline.execute({
  keyword: 'AI trends 2026',
  contentFormat: 'toplist',
  languages: ['en'],
  generateVideo: true,
});

await scheduler.schedule({
  content: result.content.en,
  video: result.videos[0],
  platform: 'facebook',
  scheduledTime: new Date('2026-06-20T10:00:00Z'),
});
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run type checking
npm run type-check

# Lint code
npm run lint

# Render Remotion video locally
npm run remotion:preview

# Render video to file
npm run remotion:render
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
import { retry } from '@/utils/retry';

const contentWithRetry = await retry(
  () => generator.generate({ /* ... */ }),
  {
    maxAttempts: 3,
    delayMs: 1000,
    backoff: 'exponential',
  }
);
```

### Video Rendering Timeout

```typescript
// Increase timeout for long videos
const videoRenderer = new VideoRenderer({
  timeout: 300000, // 5 minutes
  quality: 'medium', // Use 'medium' instead of 'high' for faster rendering
});
```

### Memory Issues with Large Batches

```typescript
// Process in smaller chunks
const chunkSize = 5;
for (let i = 0; i < keywords.length; i += chunkSize) {
  const chunk = keywords.slice(i, i + chunkSize);
  const results = await Promise.all(
    chunk.map(keyword => pipeline.execute({ keyword }))
  );
  // Process results before moving to next chunk
  await saveResults(results);
}
```

### API Key Authentication Errors

```typescript
// Validate environment variables on startup
import { validateEnv } from '@/utils/validateEnv';

validateEnv([
  'OPENAI_API_KEY',
  'ANTHROPIC_API_KEY',
  'RAPIDAPI_KEY',
]);
```

### Content Quality Issues

```typescript
// Add validation and regeneration logic
const content = await generator.generate({ /* ... */ });

if (content.metadata.wordCount < 800) {
  // Regenerate with explicit length requirement
  const betterContent = await generator.generate({
    /* ... */,
    minWords: 1200,
    includeExamples: true,
  });
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** to avoid API quotas
3. **Cache research results** to reduce API calls
4. **Validate generated content** before publishing
5. **Monitor API costs** across providers
6. **Use webhooks** for long-running video renders
7. **Store generated content** in a database for reuse
8. **Test content quality** with sample audiences before automation
