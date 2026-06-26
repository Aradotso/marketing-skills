---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I set up the AI content pipeline for automated posting
  - generate video content from text using this marketing automation
  - automate content research and script generation with AI
  - create social media videos automatically from articles
  - set up automated content workflow with Claude and OpenAI
  - build content pipeline from research to video rendering
  - configure AI content generation with multiple formats
  - scrape news and generate marketing content automatically
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to automatic video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-scans research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
- **AI content generation**: Creates multi-format content (Toplists, POV, Case Studies, How-tos) using Claude/OpenAI
- **Multi-language support**: Generates parallel English and Vietnamese content with customizable tone
- **Video rendering**: Automatically converts text content into infographics and short videos via Remotion
- **Platform optimization**: Exports videos in formats optimized for Reels, TikTok, and Shorts

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
ANTHROPIC_API_KEY=your_anthropic_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

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
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # Web scraping modules
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── package.json
```

## Key Components & Usage

### 1. Content Research Module

```typescript
import { researchTopics } from '@/lib/crawlers/research';

async function gatherResearch(keyword: string) {
  const research = await researchTopics({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: 'en'
  });
  
  return {
    articles: research.articles,
    insights: research.insights,
    trending: research.trending
  };
}

// Usage
const data = await gatherResearch('AI automation');
console.log(data.insights);
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/generators/content';
import { ClaudeClient } from '@/lib/ai/claude';
import { OpenAIClient } from '@/lib/ai/openai';

// Using Claude
async function createArticleWithClaude(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const claude = new ClaudeClient(process.env.ANTHROPIC_API_KEY!);
  
  const article = await generateContent({
    client: claude,
    research,
    format,
    language: 'en',
    tone: 'professional',
    model: 'claude-3-opus-20240229'
  });
  
  return article;
}

// Using OpenAI
async function createArticleWithOpenAI(
  research: any,
  format: string
) {
  const openai = new OpenAIClient(process.env.OPENAI_API_KEY!);
  
  const article = await generateContent({
    client: openai,
    research,
    format,
    language: 'vi',
    tone: 'friendly',
    model: 'gpt-4-turbo-preview'
  });
  
  return article;
}
```

### 3. Multi-Language Content Generation

```typescript
import { generateMultiLanguage } from '@/lib/generators/multi-language';

async function createBilingualContent(keyword: string) {
  const research = await researchTopics({ keyword });
  
  const content = await generateMultiLanguage({
    research,
    languages: ['en', 'vi'],
    format: 'toplist',
    tone: 'expert',
    includeMetadata: true
  });
  
  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: content.metadata
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/render';
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

async function generateVideoFromContent(
  article: any,
  format: 'reels' | 'tiktok' | 'shorts'
) {
  const videoConfig = {
    title: article.title,
    content: article.content,
    format,
    dimensions: format === 'reels' ? { width: 1080, height: 1920 } : undefined,
    duration: 30 // seconds
  };
  
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: videoConfig,
  });
  
  const outputPath = `./output/video-${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
  });
  
  return outputPath;
}
```

### 5. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude', // or 'openai'
    language: ['en', 'vi'],
    formats: ['toplist', 'how-to'],
    enableVideo: true,
    videoFormats: ['reels', 'shorts']
  });
  
  // Execute full pipeline
  const result = await pipeline.execute({
    keyword,
    autoPublish: false // set true to auto-post to platforms
  });
  
  return {
    articles: result.articles,
    videos: result.videos,
    insights: result.insights,
    publishedUrls: result.publishedUrls
  };
}

// Usage
const output = await runFullPipeline('AI marketing automation');
console.log('Generated:', output.articles.length, 'articles');
console.log('Generated:', output.videos.length, 'videos');
```

## API Routes (Next.js)

### Generate Content API

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/generators/content';

export async function POST(request: NextRequest) {
  const { keyword, format, language } = await request.json();
  
  try {
    const research = await researchTopics({ keyword });
    const content = await generateContent({
      research,
      format,
      language,
      tone: 'professional'
    });
    
    return NextResponse.json({ 
      success: true, 
      content 
    });
  } catch (error) {
    return NextResponse.json({ 
      success: false, 
      error: error.message 
    }, { status: 500 });
  }
}
```

### Video Rendering API

```typescript
// src/app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/render';

export async function POST(request: NextRequest) {
  const { articleId, format } = await request.json();
  
  try {
    const article = await getArticle(articleId);
    const videoPath = await generateVideoFromContent(article, format);
    
    return NextResponse.json({ 
      success: true, 
      videoUrl: `/videos/${videoPath}` 
    });
  } catch (error) {
    return NextResponse.json({ 
      success: false, 
      error: error.message 
    }, { status: 500 });
  }
}
```

## Configuration

### Content Generation Config

```typescript
// src/config/content.ts
export const contentConfig = {
  formats: {
    toplist: {
      itemCount: 5,
      includeIntro: true,
      includeConclusion: true
    },
    pov: {
      perspective: 'first-person',
      includeCounterarguments: true
    },
    caseStudy: {
      includeMetrics: true,
      includeTimeline: true
    },
    howTo: {
      stepCount: 5,
      includeImages: true
    }
  },
  
  languages: {
    en: {
      tone: 'professional',
      lengthMin: 800,
      lengthMax: 1500
    },
    vi: {
      tone: 'friendly',
      lengthMin: 800,
      lengthMax: 1500
    }
  },
  
  aiModels: {
    claude: 'claude-3-opus-20240229',
    openai: 'gpt-4-turbo-preview'
  }
};
```

### Video Rendering Config

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);

export const videoPresets = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 900 // 30 seconds
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 900
  },
  shorts: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 1800 // 60 seconds
  }
};
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run content generation
npm run generate -- --keyword "AI automation" --format toplist

# Render videos
npm run render -- --article-id 123 --format reels

# Run full pipeline
npm run pipeline -- --keyword "marketing trends" --lang en,vi
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
import cron from 'node-cron';
import { ContentPipeline } from '@/lib/pipeline';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI marketing', 'automation tools', 'content strategy'];
  
  for (const keyword of keywords) {
    const pipeline = new ContentPipeline({ 
      aiProvider: 'claude',
      language: ['en']
    });
    
    await pipeline.execute({ 
      keyword,
      autoPublish: true 
    });
  }
});
```

### Pattern 2: Batch Video Generation

```typescript
async function batchGenerateVideos(articleIds: string[]) {
  const results = await Promise.allSettled(
    articleIds.map(async (id) => {
      const article = await getArticle(id);
      return generateVideoFromContent(article, 'reels');
    })
  );
  
  const successful = results.filter(r => r.status === 'fulfilled');
  return successful.map(r => r.value);
}
```

### Pattern 3: Custom Tone Configuration

```typescript
import { generateContent } from '@/lib/generators/content';

const customTones = {
  humorous: {
    prompt: 'Write in a light, humorous tone with jokes and puns',
    temperature: 0.9
  },
  authoritative: {
    prompt: 'Write in an authoritative, data-driven tone with citations',
    temperature: 0.5
  }
};

async function generateWithCustomTone(research: any, tone: keyof typeof customTones) {
  return generateContent({
    research,
    format: 'toplist',
    toneConfig: customTones[tone],
    language: 'en'
  });
}
```

## Troubleshooting

### Issue: API Rate Limiting

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  windowMs: 60000 // 1 minute
});

async function generateWithRateLimit(keyword: string) {
  await limiter.acquire();
  try {
    return await generateContent({ keyword });
  } finally {
    limiter.release();
  }
}
```

### Issue: Video Rendering Failures

```typescript
import { renderVideo } from '@/lib/video/render';

async function safeRenderVideo(article: any, format: string) {
  try {
    return await renderVideo(article, format);
  } catch (error) {
    console.error('Video rendering failed:', error);
    
    // Retry with lower quality
    return await renderVideo(article, format, {
      quality: 'low',
      scale: 0.5
    });
  }
}
```

### Issue: Research Data Quality

```typescript
async function validateResearch(research: any) {
  if (!research.articles || research.articles.length < 3) {
    throw new Error('Insufficient research data');
  }
  
  if (!research.insights || research.insights.length === 0) {
    console.warn('No insights found, regenerating...');
    return await researchTopics({ 
      keyword: research.keyword,
      minArticles: 5 
    });
  }
  
  return research;
}
```

### Issue: Memory Management for Large Batches

```typescript
async function processBatchInChunks(items: any[], chunkSize = 5) {
  const results = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(item => processItem(item))
    );
    results.push(...chunkResults);
    
    // Allow garbage collection
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  
  return results;
}
```

## Testing

```typescript
// Example test setup
import { describe, it, expect } from 'vitest';
import { generateContent } from '@/lib/generators/content';

describe('Content Generation', () => {
  it('should generate toplist content', async () => {
    const research = { 
      keyword: 'test',
      articles: [...],
      insights: [...]
    };
    
    const content = await generateContent({
      research,
      format: 'toplist',
      language: 'en'
    });
    
    expect(content.title).toBeDefined();
    expect(content.items).toHaveLength(5);
  });
});
```

This skill enables comprehensive automation of content marketing workflows using modern AI and video generation technologies.
