---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - automate content creation with AI
  - set up AI content pipeline
  - generate videos from text content
  - crawl and research content automatically
  - create multilingual marketing content
  - build automated content workflow
  - generate social media videos with AI
  - scrape trending news for content
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Complete automation system for content creation: from research and scriptwriting to automatic video generation. This TypeScript-based pipeline leverages Claude 3, OpenAI, and Remotion to transform keywords into fully-rendered marketing content across multiple formats and languages.

## What It Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls fresh data from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
2. **AI Writing**: Generates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multilingual**: Outputs both English and Vietnamese versions with customizable tone
4. **Video Rendering**: Automatically creates infographics and short-form videos via Remotion
5. **Multi-Platform**: Exports videos optimized for Reels, TikTok, and Shorts

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

```env
# AI APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Content APIs
TWITTER_API_KEY=your_twitter_api_key_here
LINKEDIN_API_KEY=your_linkedin_api_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Remotion (Video Rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

## Development Server

```bash
# Start Next.js development server
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the application at `http://localhost:3000`

## Key API Patterns

### 1. Content Research Module

```typescript
import { researchTopic } from '@/lib/research';

async function gatherInsights(keyword: string) {
  const insights = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 50
  });
  
  return insights.map(item => ({
    title: item.title,
    url: item.url,
    summary: item.summary,
    publishedAt: item.publishedAt,
    source: item.source
  }));
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { ContentFormat, Language, Tone } from '@/types';

async function createArticle(
  topic: string,
  researchData: any[]
) {
  const content = await generateContent({
    topic,
    format: ContentFormat.TOPLIST, // or POV, CASE_STUDY, HOW_TO
    language: Language.BILINGUAL, // EN, VI, or BILINGUAL
    tone: Tone.EXPERT, // FRIENDLY, HUMOROUS, EXPERT
    researchData,
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  });
  
  return {
    titleEN: content.en.title,
    titleVI: content.vi.title,
    contentEN: content.en.body,
    contentVI: content.vi.body,
    metadata: content.metadata
  };
}
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoTemplate } from '@/types/video';

async function generateVideoFromContent(
  content: string,
  options: {
    template: VideoTemplate;
    platform: 'reels' | 'tiktok' | 'shorts';
  }
) {
  const videoConfig = {
    content,
    template: options.template,
    aspectRatio: options.platform === 'reels' ? '9:16' : '9:16',
    duration: 60, // seconds
    outputFormat: 'mp4'
  };
  
  const result = await renderVideo(videoConfig);
  
  return {
    videoUrl: result.url,
    thumbnailUrl: result.thumbnail,
    duration: result.duration,
    size: result.fileSize
  };
}
```

### 4. Complete Pipeline Workflow

```typescript
import { Pipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new Pipeline({
    aiProvider: 'claude',
    videoEnabled: true,
    autoPublish: false
  });
  
  // Step 1: Research
  const research = await pipeline.research(keyword);
  console.log(`Found ${research.length} sources`);
  
  // Step 2: Generate Content
  const article = await pipeline.generateContent({
    topic: keyword,
    format: 'toplist',
    research
  });
  
  // Step 3: Render Video
  const video = await pipeline.renderVideo({
    content: article.contentEN,
    template: 'infographic',
    platform: 'reels'
  });
  
  // Step 4: Save Results
  await pipeline.save({
    article,
    video,
    metadata: {
      keyword,
      createdAt: new Date(),
      status: 'draft'
    }
  });
  
  return {
    article,
    video
  };
}
```

## API Routes

### POST /api/research

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research';

export async function POST(req: NextRequest) {
  const { keyword, sources, timeframe } = await req.json();
  
  try {
    const results = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeframe: timeframe || '24h'
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

### POST /api/generate

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(req: NextRequest) {
  const { topic, format, language, tone, researchData } = await req.json();
  
  try {
    const content = await generateContent({
      topic,
      format,
      language,
      tone,
      researchData
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

### POST /api/video/render

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  const { content, template, platform } = await req.json();
  
  try {
    const video = await renderVideo({
      content,
      template,
      aspectRatio: platform === 'reels' ? '9:16' : '16:9'
    });
    
    return NextResponse.json({ success: true, video });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Usage Patterns

### Pattern 1: Quick Content Generation

```typescript
import { quickGenerate } from '@/lib/helpers';

// One-liner for simple use cases
const result = await quickGenerate('AI in marketing', {
  format: 'toplist',
  includeVideo: true
});
```

### Pattern 2: Batch Processing

```typescript
import { batchProcess } from '@/lib/batch';

const keywords = ['AI marketing', 'Content automation', 'Video generation'];

const results = await batchProcess(keywords, {
  parallel: 3,
  format: 'how-to',
  language: 'bilingual',
  onProgress: (progress) => console.log(`${progress}% complete`)
});
```

### Pattern 3: Custom AI Prompts

```typescript
import { customPrompt } from '@/lib/ai/custom';

const content = await customPrompt({
  systemPrompt: 'You are a marketing expert specializing in SaaS',
  userPrompt: `Create a case study about ${topic}`,
  model: 'claude-3-opus-20240229',
  temperature: 0.7,
  maxTokens: 4000
});
```

### Pattern 4: Video Templates

```typescript
import { VideoTemplate } from '@/lib/video/templates';

// Use predefined templates
const templates = {
  infographic: new VideoTemplate('infographic'),
  testimonial: new VideoTemplate('testimonial'),
  tutorial: new VideoTemplate('tutorial')
};

const video = await renderVideo({
  content: article.contentEN,
  template: templates.infographic,
  customizations: {
    backgroundColor: '#1a1a1a',
    primaryColor: '#00ff88',
    font: 'Inter'
  }
});
```

## CLI Commands

If the project includes CLI tools:

```bash
# Generate content from keyword
npm run generate -- --keyword "AI marketing" --format toplist

# Research only
npm run research -- --keyword "AI trends" --sources techcrunch,a16z

# Render video from existing content
npm run render-video -- --input ./content/article.json --template infographic

# Full pipeline
npm run pipeline -- --keyword "Marketing automation" --video --publish
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  windowMs: 60000 // 1 minute
});

await limiter.execute(async () => {
  return await generateContent(config);
});
```

### Handling AI Provider Errors

```typescript
import { AIProviderError } from '@/lib/errors';

try {
  const content = await generateContent(config);
} catch (error) {
  if (error instanceof AIProviderError) {
    // Fallback to alternative provider
    const fallbackContent = await generateContent({
      ...config,
      provider: 'openai' // Switch from Claude to OpenAI
    });
    return fallbackContent;
  }
  throw error;
}
```

### Video Rendering Timeout

```typescript
import { renderVideo } from '@/lib/video/renderer';

const video = await renderVideo(config, {
  timeout: 300000, // 5 minutes
  onTimeout: async () => {
    // Queue for background processing
    await queueVideoRender(config);
  }
});
```

### Memory Management for Large Batches

```typescript
import { BatchProcessor } from '@/lib/batch';

const processor = new BatchProcessor({
  concurrency: 2, // Process 2 at a time
  batchSize: 10,
  memoryThreshold: 0.8 // Pause at 80% memory usage
});

await processor.process(keywords, async (keyword) => {
  return await runFullPipeline(keyword);
});
```

## Type Definitions

```typescript
// types/index.ts
export enum ContentFormat {
  TOPLIST = 'toplist',
  POV = 'pov',
  CASE_STUDY = 'case_study',
  HOW_TO = 'how_to'
}

export enum Language {
  EN = 'en',
  VI = 'vi',
  BILINGUAL = 'bilingual'
}

export enum Tone {
  EXPERT = 'expert',
  FRIENDLY = 'friendly',
  HUMOROUS = 'humorous'
}

export interface ContentConfig {
  topic: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData?: any[];
  provider?: 'claude' | 'openai';
  model?: string;
}

export interface VideoConfig {
  content: string;
  template: string;
  aspectRatio: '9:16' | '16:9' | '1:1';
  duration?: number;
  outputFormat?: 'mp4' | 'webm';
}
```

## Best Practices

1. **Always validate research data** before passing to AI to avoid hallucinations
2. **Use bilingual mode sparingly** as it consumes more tokens
3. **Cache research results** for 24h to reduce API calls
4. **Test video templates** before batch rendering
5. **Monitor token usage** to stay within API limits
6. **Implement retry logic** for failed API calls
7. **Use webhooks** for long-running video renders
