---
name: marketing-pipeline-automation
description: Automated content pipeline for research, scriptwriting, and video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an automated marketing content pipeline
  - generate videos from blog posts automatically
  - crawl news and create content with Claude
  - build an AI-powered content research system
  - automate social media content with OpenAI
  - create video content pipeline with Remotion
  - research and generate content automatically
---

# Marketing Pipeline Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to use the **Ultimate AI Content Pipeline** - a complete automated content creation system that handles research, scriptwriting, content generation, and video rendering using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline Automation system is a Next.js-based TypeScript application that:

- **Auto-crawls** news and trends from sources like TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates content** in multiple formats (listicles, POV, case studies, how-tos) using Claude/OpenAI
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos** automatically from written content using Remotion
- **Optimizes for platforms** like Reels, TikTok, Shorts with proper aspect ratios

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
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000

# Database (if applicable)
DATABASE_URL=your_database_connection_string
```

## Key Architecture

The pipeline consists of several core modules:

1. **Research Module** - Crawls and aggregates content
2. **Content Generation Module** - Uses AI to create scripts
3. **Video Rendering Module** - Remotion-based video creation
4. **Scheduling Module** - Auto-posts to social platforms

## Core API Endpoints

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const { keyword, sources, timeRange } = await req.json();
  
  // Crawl data from specified sources
  const researchData = await crawlSources({
    keyword,
    sources: sources || ['techcrunch', 'a16z', 'twitter'],
    hours: timeRange || 24
  });
  
  return NextResponse.json({
    success: true,
    data: researchData
  });
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export async function POST(req: NextRequest) {
  const { researchData, format, language, tone } = await req.json();
  
  const prompt = buildPrompt({
    data: researchData,
    format: format || 'listicle',
    language: language || 'en',
    tone: tone || 'professional'
  });
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return NextResponse.json({
    content: message.content[0].text,
    metadata: {
      format,
      language,
      wordCount: message.content[0].text.split(' ').length
    }
  });
}
```

## Research Module Usage

### Crawling News Sources

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface CrawlOptions {
  keyword: string;
  sources: string[];
  hours: number;
}

export async function crawlSources(options: CrawlOptions) {
  const { keyword, sources, hours } = options;
  const results = [];
  
  for (const source of sources) {
    const data = await crawlSource(source, keyword, hours);
    results.push(...data);
  }
  
  return {
    totalArticles: results.length,
    articles: results,
    insights: extractInsights(results),
    trends: analyzeTrends(results)
  };
}

async function crawlSource(source: string, keyword: string, hours: number) {
  const config = {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': `${source}-api.rapidapi.com`
    }
  };
  
  const response = await axios.get(
    `https://${source}-api.rapidapi.com/search`,
    {
      ...config,
      params: { q: keyword, hours }
    }
  );
  
  return response.data.articles;
}
```

## Content Generation Patterns

### Multi-Format Content Generation

```typescript
// lib/content/generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentRequest {
  researchData: any;
  format: 'listicle' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
}

export async function generateContent(request: ContentRequest) {
  const systemPrompt = buildSystemPrompt(request.format, request.tone);
  const userPrompt = buildUserPrompt(request.researchData, request.language);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userPrompt }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });
  
  return {
    content: completion.choices[0].message.content,
    format: request.format,
    language: request.language
  };
}

function buildSystemPrompt(format: string, tone: string): string {
  const formatInstructions = {
    'listicle': 'Create a numbered list article with clear points',
    'pov': 'Write from a unique perspective with strong opinions',
    'case-study': 'Analyze with data, examples, and conclusions',
    'how-to': 'Provide step-by-step actionable instructions'
  };
  
  const toneStyles = {
    'professional': 'Use formal, expert language',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include wit and light humor where appropriate'
  };
  
  return `You are an expert content writer. ${formatInstructions[format]}. ${toneStyles[tone]}.`;
}
```

### Bilingual Content Generation

```typescript
// lib/content/bilingual.ts
export async function generateBilingualContent(researchData: any, format: string) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      researchData,
      format,
      language: 'en',
      tone: 'professional'
    }),
    generateContent({
      researchData,
      format,
      language: 'vi',
      tone: 'professional'
    })
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent,
    publishReady: true
  };
}
```

## Video Rendering with Remotion

### Setting Up Remotion Compositions

```typescript
// remotion/Root.tsx
import { Composition } from 'remotion';
import { ContentVideo } from './compositions/ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920} // 9:16 for Reels/TikTok
        defaultProps={{
          title: '',
          points: [],
          style: 'modern'
        }}
      />
    </>
  );
};
```

### Content Video Component

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate, Sequence } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  style: 'modern' | 'minimal' | 'bold';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, points, style }) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ opacity: titleOpacity, padding: 60 }}>
        <h1 style={{ color: '#fff', fontSize: 72 }}>{title}</h1>
      </div>
      
      {points.map((point, index) => (
        <Sequence from={60 + index * 60} durationInFrames={60} key={index}>
          <PointSlide point={point} index={index} style={style} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const PointSlide: React.FC<{ point: string; index: number; style: string }> = 
  ({ point, index, style }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 15], [0, 1]);
  
  return (
    <AbsoluteFill style={{ opacity, padding: 60 }}>
      <h2 style={{ color: '#FFD700', fontSize: 48 }}>
        {index + 1}. {point}
      </h2>
    </AbsoluteFill>
  );
};
```

### Rendering Videos

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(content: any, outputPath: string) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(path.join(process.cwd(), 'remotion/index.ts'));
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.title,
      points: content.points,
      style: 'modern'
    }
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: content.title,
      points: content.points,
      style: 'modern'
    }
  });
  
  return outputPath;
}
```

## Complete Pipeline Workflow

```typescript
// lib/pipeline/workflow.ts
export async function runContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('Starting research phase...');
  const researchData = await crawlSources({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    hours: 24
  });
  
  // Step 2: Generate content
  console.log('Generating content...');
  const content = await generateBilingualContent(researchData, 'listicle');
  
  // Step 3: Extract key points for video
  const videoPoints = extractKeyPoints(content.en.content, 5);
  
  // Step 4: Render video
  console.log('Rendering video...');
  const videoPath = await renderContentVideo({
    title: keyword,
    points: videoPoints
  }, `./output/${keyword}-${Date.now()}.mp4`);
  
  // Step 5: Return complete package
  return {
    research: researchData,
    content: {
      english: content.en,
      vietnamese: content.vi
    },
    video: {
      path: videoPath,
      aspectRatio: '9:16',
      duration: 300
    }
  };
}
```

## Running the Development Server

```bash
# Start Next.js dev server
npm run dev

# Access the application
# http://localhost:3000
```

## Configuration Files

### TypeScript Config

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rateLimit.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      this.process();
    });
  }
  
  private async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const fn = this.queue.shift()!;
    
    await fn();
    await new Promise(resolve => setTimeout(resolve, 1000)); // 1 second delay
    
    this.processing = false;
    this.process();
  }
}

export const limiter = new RateLimiter();
```

### Error Handling

```typescript
// lib/utils/errorHandler.ts
export class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'rendering',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

export async function withErrorHandling<T>(
  fn: () => Promise<T>,
  stage: string
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    console.error(`Error in ${stage}:`, error);
    throw new PipelineError(
      `Failed at ${stage}`,
      stage as any,
      error as Error
    );
  }
}
```

### Video Rendering Issues

If Remotion rendering fails:

1. Ensure FFmpeg is installed: `npm install @remotion/renderer`
2. Check memory limits for large videos
3. Use lower resolution for testing:

```typescript
// Reduce resolution for faster testing
const testComposition = {
  ...composition,
  width: 540,  // Half resolution
  height: 960
};
```

## Common Patterns

### Scheduled Pipeline Execution

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';

export function scheduleContentGeneration(keywords: string[]) {
  // Run every day at 9 AM
  cron.schedule('0 9 * * *', async () => {
    for (const keyword of keywords) {
      try {
        await runContentPipeline(keyword);
        console.log(`Pipeline completed for: ${keyword}`);
      } catch (error) {
        console.error(`Failed for ${keyword}:`, error);
      }
    }
  });
}
```

### Content Caching

```typescript
// lib/cache/content.ts
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!,
});

export async function getCachedContent(keyword: string) {
  const cached = await redis.get(`content:${keyword}`);
  return cached;
}

export async function cacheContent(keyword: string, content: any, ttl = 3600) {
  await redis.setex(`content:${keyword}`, ttl, JSON.stringify(content));
}
```

This skill provides comprehensive coverage of the Marketing Pipeline Automation system, enabling AI agents to effectively assist developers in implementing automated content creation workflows.
