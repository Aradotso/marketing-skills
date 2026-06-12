---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation
triggers:
  - automate content creation with AI research
  - generate marketing content from keywords
  - create videos from text automatically
  - set up content pipeline with Claude and OpenAI
  - crawl news and generate social media content
  - build automated marketing workflow
  - render videos with Remotion from articles
  - automate content research and writing
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates content creation from research through video generation. The pipeline crawls news sources, generates multilingual content using Claude/OpenAI, and renders videos with Remotion.

## What This Project Does

Marketing Pipeline Share is an end-to-end content automation system that:

- **Auto-Research**: Crawls news from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multilingual Support**: Generates content in both English and Vietnamese
- **Video Rendering**: Automatically creates infographics and short videos using Remotion
- **Platform Optimization**: Exports videos for Reels, TikTok, Shorts

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

### Environment Setup

Create a `.env.local` file in the project root:

```env
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling API
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Navigate to `http://localhost:3000` to access the pipeline interface.

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── crawlers/    # News crawling modules
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research Module - Crawling News

```typescript
import { crawlLatestNews } from '@/lib/crawlers/newsCrawler';
import { analyzeInsights } from '@/lib/ai/insightAnalyzer';

async function researchTopic(keyword: string) {
  // Crawl news from multiple sources
  const newsData = await crawlLatestNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    limit: 20
  });

  // Extract insights using AI
  const insights = await analyzeInsights(newsData, {
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  return {
    rawData: newsData,
    insights
  };
}
```

### 2. Content Generation with AI

```typescript
import { generateContent } from '@/lib/generators/contentGenerator';

async function createArticle(topic: string, research: any) {
  const content = await generateContent({
    topic,
    research,
    format: 'toplist', // 'pov' | 'case-study' | 'how-to'
    languages: ['en', 'vi'],
    tone: 'expert', // 'friendly' | 'humorous'
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY,
    model: 'claude-3-sonnet-20240229'
  });

  return content;
  // Returns: { en: "...", vi: "...", metadata: {...} }
}
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/videoRenderer';
import { createInfographic } from '@/lib/video/templates/infographic';

async function generateVideoFromContent(content: any) {
  // Create video composition
  const composition = createInfographic({
    title: content.title,
    keyPoints: content.keyPoints,
    stats: content.stats,
    duration: 30 // seconds
  });

  // Render video
  const video = await renderVideo({
    composition,
    format: 'reels', // 'tiktok' | 'shorts'
    resolution: '1080x1920',
    fps: 30,
    outputPath: './output/video.mp4'
  });

  return video;
}
```

## Complete Pipeline Example

```typescript
import { runContentPipeline } from '@/lib/pipeline';

async function automateContentCreation() {
  const result = await runContentPipeline({
    keyword: 'AI automation tools 2026',
    
    // Research configuration
    research: {
      sources: ['techcrunch', 'a16z', 'twitter'],
      depth: 'comprehensive' // 'quick' | 'comprehensive'
    },
    
    // Content generation
    content: {
      formats: ['toplist', 'case-study'],
      languages: ['en', 'vi'],
      tone: 'expert',
      aiProvider: 'claude',
      model: 'claude-3-opus-20240229'
    },
    
    // Video rendering
    video: {
      enabled: true,
      platforms: ['reels', 'tiktok', 'shorts'],
      style: 'minimal' // 'dynamic' | 'minimal'
    },
    
    // Auto-publishing (optional)
    publish: {
      enabled: false,
      platforms: ['facebook', 'linkedin'],
      schedule: '2026-06-15T10:00:00Z'
    }
  });

  return result;
  // Returns: {
  //   research: {...},
  //   articles: [{lang: 'en', content: '...'}, ...],
  //   videos: [{platform: 'reels', url: '...'}, ...],
  //   publishStatus: {...}
  // }
}
```

## API Routes (Next.js)

### Trigger Research

```typescript
// src/app/api/research/route.ts
import { NextResponse } from 'next/server';
import { crawlLatestNews } from '@/lib/crawlers/newsCrawler';

export async function POST(request: Request) {
  const { keyword, sources } = await request.json();
  
  try {
    const results = await crawlLatestNews({
      keyword,
      sources,
      timeRange: '24h'
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

### Generate Content

```typescript
// src/app/api/generate/route.ts
import { NextResponse } from 'next/server';
import { generateContent } from '@/lib/generators/contentGenerator';

export async function POST(request: Request) {
  const { topic, research, format, languages } = await request.json();
  
  const content = await generateContent({
    topic,
    research,
    format,
    languages,
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  return NextResponse.json(content);
}
```

## Configuration Patterns

### Custom AI Provider Configuration

```typescript
// lib/config/aiProviders.ts
export const aiProviderConfig = {
  claude: {
    baseURL: 'https://api.anthropic.com/v1',
    models: {
      fast: 'claude-3-haiku-20240307',
      balanced: 'claude-3-sonnet-20240229',
      powerful: 'claude-3-opus-20240229'
    },
    maxTokens: 4096
  },
  openai: {
    baseURL: 'https://api.openai.com/v1',
    models: {
      fast: 'gpt-3.5-turbo',
      balanced: 'gpt-4-turbo-preview',
      powerful: 'gpt-4'
    },
    maxTokens: 4096
  }
};
```

### Custom Remotion Template

```typescript
// remotion/compositions/CustomInfographic.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const CustomInfographic: React.FC<{
  title: string;
  stats: Array<{ label: string; value: string }>;
}> = ({ title, stats }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000', opacity }}>
      <h1 style={{ color: '#fff', fontSize: 48 }}>{title}</h1>
      {stats.map((stat, i) => (
        <div key={i}>
          <span>{stat.label}:</span>
          <span>{stat.value}</span>
        </div>
      ))}
    </AbsoluteFill>
  );
};
```

## Common Workflows

### Workflow 1: Daily News Digest

```typescript
import { scheduleDaily } from '@/lib/scheduler';
import { runContentPipeline } from '@/lib/pipeline';

// Run every day at 8 AM
scheduleDaily('0 8 * * *', async () => {
  await runContentPipeline({
    keyword: 'tech news',
    research: { sources: ['techcrunch', 'a16z'] },
    content: { formats: ['toplist'], languages: ['en'] },
    video: { enabled: true, platforms: ['reels'] },
    publish: { enabled: true, platforms: ['facebook'] }
  });
});
```

### Workflow 2: Keyword-Based Campaign

```typescript
async function createCampaign(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => 
      runContentPipeline({
        keyword,
        content: { formats: ['how-to', 'case-study'] },
        video: { enabled: true }
      })
    )
  );
  
  return results;
}

createCampaign([
  'AI marketing automation',
  'Content creation tools',
  'Social media scheduling'
]);
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rateLimiter.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function batchGenerate(topics: string[]) {
  const promises = topics.map(topic =>
    limit(() => generateContent({ topic, /* ... */ }))
  );
  
  return Promise.all(promises);
}
```

### Claude API Errors

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const message = await anthropic.messages.create({
        model: 'claude-3-opus-20240229',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }]
      });
      
      return message.content[0].text;
    } catch (error) {
      if (error.status === 429) {
        // Rate limit - wait and retry
        await new Promise(r => setTimeout(r, 2000 * (i + 1)));
        continue;
      }
      throw error;
    }
  }
}
```

### Remotion Rendering Issues

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

async function safeRender(compositionId: string) {
  try {
    const bundleLocation = await bundle({
      entryPoint: './remotion/index.ts',
      webpackOverride: (config) => config
    });
    
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: compositionId
    });
    
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: `out/${compositionId}.mp4`
    });
  } catch (error) {
    console.error('Render failed:', error);
    // Fallback to static image export
    // ... fallback logic
  }
}
```

### Memory Management for Large Batches

```typescript
async function processLargeBatch(items: string[]) {
  const batchSize = 10;
  const results = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(item => runContentPipeline({ keyword: item }))
    );
    
    results.push(...batchResults);
    
    // Clear memory between batches
    if (global.gc) global.gc();
  }
  
  return results;
}
```

## Testing

```typescript
// __tests__/pipeline.test.ts
import { runContentPipeline } from '@/lib/pipeline';

describe('Content Pipeline', () => {
  it('should generate content from keyword', async () => {
    const result = await runContentPipeline({
      keyword: 'test keyword',
      research: { sources: ['techcrunch'] },
      content: { formats: ['toplist'], languages: ['en'] },
      video: { enabled: false }
    });
    
    expect(result.articles).toHaveLength(1);
    expect(result.articles[0].lang).toBe('en');
  });
});
```

This skill provides comprehensive coverage of the Marketing Pipeline Share project's automation capabilities, from research crawling through AI-powered content generation to video rendering with Remotion.
