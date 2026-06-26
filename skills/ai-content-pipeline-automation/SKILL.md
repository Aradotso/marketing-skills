---
name: ai-content-pipeline-automation
description: Complete AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation pipeline
  - generate content from research to video
  - set up AI content automation system
  - create videos from articles automatically
  - crawl news and generate content
  - build automated marketing pipeline
  - remotion video generation from content
  - claude openai content pipeline
---

# AI Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with an automated content creation pipeline that handles research, scriptwriting, and video generation. The system crawls news sources, generates multi-format content using Claude/OpenAI, and renders videos automatically using Remotion.

## What This Project Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for fresh data (last 24h)
2. **Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 and OpenAI
3. **Multi-language**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Automatically converts articles to videos/infographics using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment variables
cp .env.example .env
```

## Environment Configuration

Create a `.env.local` file with the following variables:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key
RAPIDAPI_KEY=your_rapidapi_key

# Database
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping modules
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Content Research Crawler

```typescript
import { crawlNewsSource } from '@/lib/crawler/news-crawler';

// Crawl TechCrunch for AI-related articles
const articles = await crawlNewsSource({
  source: 'techcrunch',
  keyword: 'artificial intelligence',
  timeRange: '24h',
  limit: 10
});

// Result structure
interface Article {
  title: string;
  url: string;
  content: string;
  publishedAt: Date;
  source: string;
  insights: string[];
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { Anthropic } from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// Generate content using Claude
const content = await generateContent({
  client: anthropic,
  topic: 'AI Marketing Trends 2024',
  format: 'toplist', // 'toplist' | 'pov' | 'case-study' | 'how-to'
  language: 'en', // 'en' | 'vi'
  tone: 'expert', // 'expert' | 'friendly' | 'humorous'
  researchData: articles,
});

// Content structure
interface GeneratedContent {
  title: string;
  introduction: string;
  mainContent: string;
  conclusion: string;
  keyTakeaways: string[];
  metadata: {
    wordCount: number;
    readingTime: number;
    seoKeywords: string[];
  };
}
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';
import { generateWithOpenAI } from '@/lib/ai/openai-generator';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

const content = await generateWithOpenAI({
  client: openai,
  model: 'gpt-4-turbo-preview',
  topic: 'Content Marketing Automation',
  format: 'how-to',
  context: researchedArticles,
});
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/remotion/VideoComposition';

// Generate video from article content
async function generateVideo(content: GeneratedContent) {
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyTakeaways,
      duration: 30, // seconds
      platform: 'reels', // 'reels' | 'tiktok' | 'shorts'
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.title}.mp4`,
  });
}
```

## Full Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Complete automation workflow
async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    openaiKey: process.env.OPENAI_API_KEY!,
    anthropicKey: process.env.ANTHROPIC_API_KEY!,
    rapidApiKey: process.env.RAPIDAPI_KEY!,
  });

  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    depth: 'detailed',
  });

  // Step 2: Generate Content (multiple formats)
  const contents = await pipeline.generateMultiFormat({
    research,
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi'],
  });

  // Step 3: Create Videos
  const videos = await pipeline.renderVideos({
    contents,
    platforms: ['reels', 'tiktok', 'shorts'],
  });

  return {
    articles: contents,
    videos,
    metadata: {
      keyword,
      generatedAt: new Date(),
      totalArticles: contents.length,
      totalVideos: videos.length,
    },
  };
}

// Usage
const result = await runContentPipeline('AI marketing automation');
console.log(`Generated ${result.totalArticles} articles and ${result.totalVideos} videos`);
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  const { keyword, format, language } = await request.json();

  try {
    // Research phase
    const research = await crawlNewsSource({
      keyword,
      timeRange: '24h',
    });

    // Generation phase
    const content = await generateContent({
      topic: keyword,
      format,
      language,
      researchData: research,
    });

    return NextResponse.json({
      success: true,
      content,
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// src/app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  const { contentId, platform } = await request.json();

  try {
    const videoUrl = await renderVideo({
      contentId,
      platform,
      quality: 'high',
    });

    return NextResponse.json({
      success: true,
      videoUrl,
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Video rendering failed' },
      { status: 500 }
    );
  }
}
```

## Custom Content Formats

```typescript
// Define custom content template
interface ContentTemplate {
  format: string;
  structure: string[];
  prompts: {
    introduction: string;
    body: string;
    conclusion: string;
  };
}

const customTemplate: ContentTemplate = {
  format: 'expert-analysis',
  structure: ['hook', 'context', 'analysis', 'predictions', 'actionables'],
  prompts: {
    introduction: 'Create a compelling hook based on recent data',
    body: 'Provide deep analysis with data-backed insights',
    conclusion: 'Offer actionable predictions for the next 6 months',
  },
};

// Use custom template
const content = await generateContent({
  template: customTemplate,
  research: articles,
});
```

## Remotion Video Templates

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  duration: number;
}> = ({ title, points, duration }) => {
  const frame = useCurrentFrame();
  const fps = 30;

  const titleOpacity = interpolate(
    frame,
    [0, fps * 0.5],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ opacity: titleOpacity, padding: 60 }}>
        <h1 style={{ color: 'white', fontSize: 72 }}>{title}</h1>
      </div>
      {points.map((point, index) => (
        <Point key={index} text={point} index={index} frame={frame} fps={fps} />
      ))}
    </AbsoluteFill>
  );
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await crawlNewsSource({ keyword });
      const content = await generateContent({
        topic: keyword,
        researchData: research,
      });
      return { keyword, content };
    })
  );

  return results;
}
```

### Content Scheduling

```typescript
import { scheduleContent } from '@/lib/scheduler';

await scheduleContent({
  content: generatedArticle,
  platforms: ['facebook', 'linkedin', 'twitter'],
  publishAt: new Date('2024-12-20T10:00:00Z'),
  autoVideo: true, // Also generate and post video
});
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000, // 1 minute
});

await limiter.execute(async () => {
  return await generateContent(params);
});
```

### Video Rendering Timeout

```typescript
// Increase timeout for long videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  timeoutInMilliseconds: 300000, // 5 minutes
  chromiumOptions: {
    gl: 'angle',
  },
});
```

### Memory Issues with Large Research Data

```typescript
// Stream and process research data in chunks
import { streamCrawl } from '@/lib/crawler/stream-crawler';

for await (const chunk of streamCrawl({ keyword })) {
  await processChunk(chunk);
}
```

### Claude API Errors

```typescript
import { retry } from '@/lib/utils/retry';

const content = await retry(
  async () => await generateContent(params),
  {
    maxAttempts: 3,
    delayMs: 1000,
    backoff: 'exponential',
  }
);
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run Remotion studio (preview videos)
npm run remotion:studio

# Render a specific video composition
npm run remotion:render -- ContentVideo out/video.mp4

# Type checking
npm run type-check

# Linting
npm run lint
```
