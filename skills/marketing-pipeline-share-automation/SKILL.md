---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation pipeline
  - generate video from research articles
  - create multi-language marketing content
  - scrape trending tech news for content
  - build AI content automation workflow
  - schedule automated social media posts
  - render videos from blog posts automatically
  - crawl news sources for content ideas
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to use Marketing Pipeline Share, an end-to-end content automation system that researches trending topics, generates multi-format content in multiple languages, and automatically renders videos for social media platforms.

## What It Does

Marketing Pipeline Share is a TypeScript-based content production pipeline that:

- **Auto-scans research sources**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for trending content within 24h
- **Generates AI content**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 and OpenAI
- **Multi-language support**: Simultaneous English and Vietnamese content generation
- **Video rendering**: Automatically converts written content to infographics and short-form videos using Remotion
- **Platform optimization**: Exports videos in aspect ratios for Reels, TikTok, and Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# Server runs on http://localhost:3000
```

## Key Architecture

```typescript
// Typical project structure
/src
  /app              # Next.js app router
  /components       # React components
  /lib
    /ai             # Claude & OpenAI integrations
    /crawler        # Web scraping modules
    /video          # Remotion video rendering
  /types            # TypeScript definitions
```

## Core Workflows

### 1. Research & Content Crawling

```typescript
// lib/crawler/scraper.ts
import axios from 'axios';

interface NewsSource {
  name: string;
  url: string;
  selector: string;
}

export async function crawlTechNews(keyword: string, hours: number = 24) {
  const sources: NewsSource[] = [
    { name: 'TechCrunch', url: 'https://techcrunch.com', selector: '.post-block' },
    { name: 'a16z', url: 'https://a16z.com/posts', selector: '.article' }
  ];

  const articles = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(source.url, {
        params: { q: keyword },
        headers: {
          'User-Agent': 'Mozilla/5.0',
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY
        }
      });
      
      // Parse and filter by timestamp
      const parsed = parseArticles(response.data, source.selector);
      const recent = filterByTime(parsed, hours);
      articles.push(...recent);
    } catch (error) {
      console.error(`Failed to crawl ${source.name}:`, error);
    }
  }

  return articles;
}

function filterByTime(articles: any[], hours: number) {
  const cutoff = Date.now() - (hours * 60 * 60 * 1000);
  return articles.filter(a => new Date(a.publishedAt).getTime() > cutoff);
}
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type ContentTone = 'expert' | 'friendly' | 'humorous';

interface GenerateContentParams {
  keyword: string;
  research: string[];
  format: ContentFormat;
  tone: ContentTone;
  language: 'en' | 'vi';
}

export async function generateContent(params: GenerateContentParams) {
  const { keyword, research, format, tone, language } = params;

  const prompt = buildPrompt(keyword, research, format, tone, language);

  // Use Claude for long-form content
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  const content = message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';

  return {
    content,
    metadata: {
      format,
      language,
      wordCount: content.split(/\s+/).length,
      generatedAt: new Date().toISOString()
    }
  };
}

function buildPrompt(
  keyword: string,
  research: string[],
  format: ContentFormat,
  tone: ContentTone,
  language: 'en' | 'vi'
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with detailed explanations',
    'pov': 'Write from a unique perspective with strong opinions',
    'case-study': 'Analyze with data, examples, and actionable insights',
    'how-to': 'Provide step-by-step instructions with clear outcomes'
  };

  const toneInstructions = {
    'expert': 'professional, authoritative, data-driven',
    'friendly': 'conversational, approachable, relatable',
    'humorous': 'witty, entertaining, with clever analogies'
  };

  const langInstruction = language === 'vi' 
    ? 'Write in Vietnamese.' 
    : 'Write in English.';

  return `
You are a content marketing expert. Create a ${format} article about "${keyword}".

Research data:
${research.join('\n\n')}

Requirements:
- Format: ${formatInstructions[format]}
- Tone: ${toneInstructions[tone]}
- ${langInstruction}
- Include specific data points and examples from the research
- Make it actionable and engaging
- Length: 1000-1500 words

Generate the article now:
  `.trim();
}
```

### 3. Dual Language Generation

```typescript
// lib/ai/multi-language.ts
export async function generateBilingualContent(params: Omit<GenerateContentParams, 'language'>) {
  const [english, vietnamese] = await Promise.all([
    generateContent({ ...params, language: 'en' }),
    generateContent({ ...params, language: 'vi' })
  ]);

  return {
    en: english,
    vi: vietnamese
  };
}
```

### 4. Video Rendering with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  contentId: string;
  title: string;
  keyPoints: string[];
  platform: 'reels' | 'tiktok' | 'shorts';
}

const platformSpecs = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 }
};

export async function renderContentVideo(config: VideoConfig) {
  const { contentId, title, keyPoints, platform } = config;
  const specs = platformSpecs[platform];

  // Bundle Remotion composition
  const bundled = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title,
      keyPoints,
      theme: 'modern'
    }
  });

  // Render video
  const outputPath = path.join(
    process.cwd(), 
    'public/videos', 
    `${contentId}-${platform}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      keyPoints
    }
  });

  return {
    path: outputPath,
    url: `/videos/${contentId}-${platform}.mp4`,
    specs
  };
}
```

### 5. Remotion Video Component

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  theme: 'modern' | 'minimal';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, keyPoints, theme }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const titleDuration = fps * 3; // 3 seconds
  const pointDuration = fps * 4; // 4 seconds per point

  return (
    <AbsoluteFill style={{ backgroundColor: theme === 'modern' ? '#1a1a2e' : '#ffffff' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={titleDuration}>
        <AbsoluteFill style={{ justifyContent: 'center', alignItems: 'center' }}>
          <h1 style={{
            fontSize: 80,
            color: theme === 'modern' ? '#ffffff' : '#000000',
            textAlign: 'center',
            padding: '0 100px',
            opacity: Math.min(1, frame / 30)
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Key Points Sequences */}
      {keyPoints.map((point, index) => (
        <Sequence
          key={index}
          from={titleDuration + (index * pointDuration)}
          durationInFrames={pointDuration}
        >
          <AbsoluteFill style={{ justifyContent: 'center', alignItems: 'center', padding: '0 80px' }}>
            <div style={{ fontSize: 60, color: theme === 'modern' ? '#00d4ff' : '#0066cc', marginBottom: 40 }}>
              {index + 1}
            </div>
            <p style={{
              fontSize: 48,
              color: theme === 'modern' ? '#ffffff' : '#333333',
              textAlign: 'center',
              lineHeight: 1.5
            }}>
              {point}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## API Routes

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlTechNews } from '@/lib/crawler/scraper';
import { generateBilingualContent } from '@/lib/ai/multi-language';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, tone, platforms } = await req.json();

    // Step 1: Research
    const research = await crawlTechNews(keyword, 24);
    const researchTexts = research.map(a => `${a.title}\n${a.summary}`);

    // Step 2: Generate content
    const content = await generateBilingualContent({
      keyword,
      research: researchTexts,
      format,
      tone
    });

    // Step 3: Extract key points for video
    const keyPoints = extractKeyPoints(content.en.content);

    // Step 4: Render videos for each platform
    const videos = await Promise.all(
      platforms.map((platform: 'reels' | 'tiktok' | 'shorts') =>
        renderContentVideo({
          contentId: generateId(),
          title: keyword,
          keyPoints,
          platform
        })
      )
    );

    return NextResponse.json({
      success: true,
      content,
      videos,
      research: research.length
    });

  } catch (error) {
    console.error('Generation failed:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - can be enhanced with AI
  const lines = content.split('\n');
  const points = lines
    .filter(line => line.match(/^(\d+\.|•|-)/))
    .map(line => line.replace(/^(\d+\.|•|-)\s*/, ''))
    .slice(0, 5);
  
  return points.length > 0 ? points : [content.slice(0, 200)];
}

function generateId(): string {
  return `content_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
}
```

## Common Patterns

### Full Pipeline Execution

```typescript
// lib/pipeline/executor.ts
export async function executeFullPipeline(keyword: string) {
  console.log('🔍 Starting research...');
  const research = await crawlTechNews(keyword, 24);

  console.log('✍️ Generating content...');
  const content = await generateBilingualContent({
    keyword,
    research: research.map(a => a.summary),
    format: 'toplist',
    tone: 'expert'
  });

  console.log('🎬 Rendering videos...');
  const keyPoints = extractKeyPoints(content.en.content);
  const videos = await Promise.all([
    renderContentVideo({ 
      contentId: generateId(), 
      title: keyword, 
      keyPoints, 
      platform: 'reels' 
    }),
    renderContentVideo({ 
      contentId: generateId(), 
      title: keyword, 
      keyPoints, 
      platform: 'tiktok' 
    })
  ]);

  console.log('✅ Pipeline complete!');
  return { content, videos, research };
}
```

### Scheduling Content

```typescript
// lib/scheduler/queue.ts
import { Queue } from 'bullmq';

const contentQueue = new Queue('content-generation', {
  connection: {
    host: 'localhost',
    port: 6379
  }
});

export async function scheduleContentGeneration(
  keyword: string,
  scheduledFor: Date
) {
  await contentQueue.add(
    'generate',
    { keyword, format: 'toplist', tone: 'expert' },
    { delay: scheduledFor.getTime() - Date.now() }
  );
}
```

## Configuration

### Content Formats Configuration

```typescript
// config/content-formats.ts
export const contentFormats = {
  toplist: {
    minPoints: 5,
    maxPoints: 10,
    includeIntro: true,
    includeConclusion: true
  },
  pov: {
    minWordCount: 800,
    maxWordCount: 1200,
    requireThesis: true
  },
  'case-study': {
    sections: ['background', 'challenge', 'solution', 'results'],
    requireData: true
  },
  'how-to': {
    minSteps: 3,
    maxSteps: 12,
    includePrerequisites: true
  }
};
```

### Video Platform Configuration

```typescript
// config/video-platforms.ts
export const videoPlatformConfig = {
  reels: {
    aspectRatio: '9:16',
    maxDuration: 90,
    recommendedDuration: 30,
    fps: 30
  },
  tiktok: {
    aspectRatio: '9:16',
    maxDuration: 180,
    recommendedDuration: 60,
    fps: 30
  },
  shorts: {
    aspectRatio: '9:16',
    maxDuration: 60,
    recommendedDuration: 45,
    fps: 30
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import { RateLimiter } from 'limiter';

const claudeLimiter = new RateLimiter({
  tokensPerInterval: 50,
  interval: 'minute'
});

export async function callClaudeWithLimit<T>(
  fn: () => Promise<T>
): Promise<T> {
  await claudeLimiter.removeTokens(1);
  return fn();
}
```

### Video Rendering Errors

If video rendering fails:

```typescript
// Add error handling and retries
export async function renderWithRetry(config: VideoConfig, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await renderContentVideo(config);
    } catch (error) {
      console.error(`Render attempt ${i + 1} failed:`, error);
      if (i === maxRetries - 1) throw error;
      await new Promise(r => setTimeout(r, 2000 * (i + 1)));
    }
  }
}
```

### Memory Issues with Large Crawls

```typescript
// Use streaming for large data
import { Readable } from 'stream';

export async function* crawlTechNewsStream(keyword: string) {
  const sources = getSources();
  
  for (const source of sources) {
    const articles = await crawlSource(source, keyword);
    for (const article of articles) {
      yield article;
    }
  }
}
```

### Missing Environment Variables

```typescript
// lib/utils/env-validator.ts
const requiredEnvVars = [
  'ANTHROPIC_API_KEY',
  'OPENAI_API_KEY',
  'RAPIDAPI_KEY'
];

export function validateEnv() {
  const missing = requiredEnvVars.filter(v => !process.env[v]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}\n` +
      'Please check your .env.local file.'
    );
  }
}
```

## Running Tests

```bash
# Unit tests
npm test

# E2E pipeline test
npm run test:pipeline

# Video rendering test
npm run test:video
```

## Production Deployment

```bash
# Build for production
npm run build

# Start production server
npm start

# Or deploy to Vercel
vercel --prod
```

This skill enables AI agents to implement sophisticated content automation workflows using Marketing Pipeline Share's research, generation, and video rendering capabilities.
