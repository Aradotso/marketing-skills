---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, video generation and social media posting with Claude/OpenAI integration
triggers:
  - automate content creation from research to video
  - set up AI content pipeline with Claude
  - create automated marketing content workflow
  - generate videos from blog posts automatically
  - build content automation with Remotion
  - scrape news and generate social posts
  - automate content research and writing
  - create AI-driven content pipeline
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a complete AI-powered content automation system that handles the entire content lifecycle: from research (crawling news sources), to script generation (using Claude/OpenAI), to video rendering (with Remotion), and social media posting. It's designed for content creators, marketers, and businesses to automate up to 90% of their content workflow.

**Key capabilities:**
- Auto-scan research from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
- Generate multi-format content (Toplist, POV, Case Study, How-to)
- Bilingual support (English & Vietnamese)
- Automatic video/infographic rendering via Remotion
- Social media optimization for Reels, TikTok, Shorts

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

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Social Media APIs (optional)
FACEBOOK_ACCESS_TOKEN=your_fb_token
TWITTER_API_KEY=your_twitter_key

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── api/             # API routes
│   └── utils/           # Utilities
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research & Content Crawling

```typescript
import { crawlNews } from '@/lib/crawler/news-scraper';
import { analyzeInsights } from '@/lib/ai/insight-analyzer';

async function gatherResearch(keyword: string) {
  // Crawl news from multiple sources
  const newsData = await crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });

  // Extract insights using AI
  const insights = await analyzeInsights(newsData, {
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  });

  return {
    rawData: newsData,
    insights,
    topTrends: insights.slice(0, 5)
  };
}

// Usage
const research = await gatherResearch('AI automation');
console.log(research.topTrends);
```

### 2. Content Generation with AI

```typescript
import { generateContent } from '@/lib/content/generator';
import { ContentFormat, Language, Tone } from '@/lib/types';

async function createContent(topic: string) {
  const content = await generateContent({
    topic,
    format: ContentFormat.TOPLIST, // TOPLIST, POV, CASE_STUDY, HOW_TO
    language: Language.BILINGUAL, // ENGLISH, VIETNAMESE, BILINGUAL
    tone: Tone.EXPERT, // EXPERT, FRIENDLY, HUMOROUS
    aiProvider: 'claude',
    includeDataBacking: true,
    researchData: research // from previous step
  });

  return content;
}

// Example output structure
interface GeneratedContent {
  title: {
    en: string;
    vi: string;
  };
  body: {
    en: string;
    vi: string;
  };
  metadata: {
    format: string;
    keywords: string[];
    readingTime: number;
  };
  mediaAssets: {
    images: string[];
    videoScript?: string;
  };
}
```

### 3. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';
import { VideoTemplate } from '@/remotion/templates';

async function generateVideo(content: GeneratedContent) {
  const videoConfig = {
    template: VideoTemplate.INFOGRAPHIC, // INFOGRAPHIC, TALKING_HEAD, SLIDESHOW
    content: {
      title: content.title.en,
      points: content.body.en.split('\n').slice(0, 5),
      branding: {
        logo: '/path/to/logo.png',
        colors: {
          primary: '#FF6B6B',
          secondary: '#4ECDC4'
        }
      }
    },
    format: {
      width: 1080,
      height: 1920, // Vertical for Reels/TikTok
      fps: 30,
      durationInFrames: 900 // 30 seconds at 30fps
    }
  };

  const videoUrl = await renderVideo(videoConfig);
  return videoUrl;
}
```

### 4. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    depth: 'deep' // 'shallow', 'medium', 'deep'
  });

  // Step 2: Generate Content
  const content = await pipeline.generate({
    research,
    formats: [ContentFormat.TOPLIST, ContentFormat.HOW_TO],
    languages: [Language.ENGLISH, Language.VIETNAMESE]
  });

  // Step 3: Create Videos
  const videos = await pipeline.renderVideos({
    content,
    platforms: ['tiktok', 'reels', 'youtube-shorts']
  });

  // Step 4: Schedule Posts (optional)
  await pipeline.schedule({
    content,
    videos,
    platforms: {
      facebook: { pageId: process.env.FB_PAGE_ID },
      twitter: { accountId: process.env.TWITTER_ID }
    },
    publishAt: new Date('2024-12-01T10:00:00Z')
  });

  return {
    research,
    content,
    videos
  };
}

// Execute pipeline
const result = await runFullPipeline('AI marketing tools 2024');
```

## API Routes

### POST /api/research

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNews } from '@/lib/crawler/news-scraper';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeRange } = await request.json();

  const data = await crawlNews({ keyword, sources, timeRange });

  return NextResponse.json({ success: true, data });
}
```

**Usage:**
```typescript
const response = await fetch('/api/research', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI automation',
    sources: ['techcrunch', 'a16z'],
    timeRange: '24h'
  })
});

const { data } = await response.json();
```

### POST /api/generate

```typescript
// app/api/generate/route.ts
export async function POST(request: NextRequest) {
  const { topic, format, language, tone } = await request.json();

  const content = await generateContent({
    topic,
    format,
    language,
    tone,
    aiProvider: 'claude'
  });

  return NextResponse.json({ success: true, content });
}
```

### POST /api/render-video

```typescript
// app/api/render-video/route.ts
export async function POST(request: NextRequest) {
  const { contentId, template, format } = await request.json();

  const videoUrl = await renderVideo({
    contentId,
    template,
    format
  });

  return NextResponse.json({ success: true, videoUrl });
}
```

## CLI Commands (if available)

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run content pipeline from CLI
npm run pipeline -- --keyword "AI tools" --format toplist

# Render videos in batch
npm run render-videos -- --input ./content/*.json

# Deploy to production
npm run deploy
```

## Advanced Configuration

### Custom AI Prompts

```typescript
// lib/config/prompts.ts
export const customPrompts = {
  toplist: {
    system: `You are an expert content creator specializing in creating engaging top lists...`,
    user: (topic: string) => `Create a top 10 list about ${topic}...`
  },
  caseStudy: {
    system: `You are a business analyst creating detailed case studies...`,
    user: (topic: string) => `Analyze and create a case study about ${topic}...`
  }
};

// Usage in content generator
import { customPrompts } from '@/lib/config/prompts';

const content = await generateContent({
  topic: 'SaaS growth',
  customPrompt: customPrompts.caseStudy
});
```

### Remotion Template Customization

```typescript
// remotion/templates/InfographicTemplate.tsx
import { AbsoluteFill, useCurrentFrame } from 'remotion';

export const InfographicTemplate: React.FC<{
  title: string;
  points: string[];
  branding: BrandingConfig;
}> = ({ title, points, branding }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: branding.colors.primary }}>
      <h1 style={{ opacity: Math.min(1, frame / 30) }}>
        {title}
      </h1>
      {points.map((point, i) => (
        <div
          key={i}
          style={{
            opacity: Math.max(0, Math.min(1, (frame - i * 30) / 30))
          }}
        >
          {point}
        </div>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000 // 1 minute
});

async function safeApiCall() {
  await limiter.wait();
  return await makeApiCall();
}
```

### Handling AI Errors

```typescript
import { retryWithBackoff } from '@/lib/utils/retry';

async function generateWithRetry(prompt: string) {
  try {
    return await retryWithBackoff(
      () => generateContent({ topic: prompt }),
      { maxAttempts: 3, baseDelay: 1000 }
    );
  } catch (error) {
    console.error('Failed after retries:', error);
    // Fallback to alternative provider
    return await generateContent({
      topic: prompt,
      aiProvider: 'openai' // Switch from Claude to OpenAI
    });
  }
}
```

### Video Rendering Timeouts

```typescript
// Increase timeout for long videos
const videoUrl = await renderVideo(config, {
  timeout: 300000, // 5 minutes
  onProgress: (progress) => {
    console.log(`Rendering: ${progress}%`);
  }
});
```

### Memory Issues with Large Datasets

```typescript
// Process research data in batches
async function processLargeResearch(keyword: string) {
  const batchSize = 10;
  const allSources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];

  const results = [];
  for (let i = 0; i < allSources.length; i += batchSize) {
    const batch = allSources.slice(i, i + batchSize);
    const data = await crawlNews({ keyword, sources: batch });
    results.push(data);
  }

  return results.flat();
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement caching** for research data to avoid redundant API calls
3. **Queue video rendering** jobs for better resource management
4. **Validate content** before publishing to social media
5. **Monitor API usage** to stay within rate limits
6. **Store generated content** in a database for versioning and reuse
