---
name: ultimate-ai-content-pipeline
description: Automated AI content pipeline for research, scriptwriting, video generation, and multi-platform publishing using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation from research to video
  - set up AI content pipeline with Claude and OpenAI
  - generate videos automatically from written content
  - crawl news and create social media content automatically
  - build automated marketing content workflow
  - create multilingual content with AI research
  - render videos from blog posts using Remotion
  - automate content publishing pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a complete automated content creation pipeline that handles research, scriptwriting, content generation, and video rendering. It crawls news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, uses AI (Claude 3, OpenAI) to generate content in multiple formats and languages, and automatically renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from major news sources (24h updates)
- **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
- **Multi-language Support**: Generates parallel English & Vietnamese versions
- **Video Rendering**: Automatically converts written content to videos/infographics using Remotion
- **Multi-platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Data Sources (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Optional: Database (if using)
DATABASE_URL=your_database_url
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion rendering (separate)
npm run remotion
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
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core APIs and Functions

### 1. Content Research & Crawling

```typescript
// lib/crawler/newsCrawler.ts
import { crawlNews } from '@/lib/crawler/newsCrawler';

interface CrawlOptions {
  sources: string[];
  timeRange: '24h' | '7d' | '30d';
  keywords?: string[];
}

async function fetchLatestNews(options: CrawlOptions) {
  const news = await crawlNews({
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    keywords: ['AI', 'startup', 'marketing']
  });
  
  return news;
}
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claudeGenerator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentOptions {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  research: any[];
}

export async function generateContent(options: ContentOptions) {
  const prompt = buildPrompt(options);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].text;
}

function buildPrompt(options: ContentOptions): string {
  const researchSummary = options.research
    .map(r => `- ${r.title}: ${r.summary}`)
    .join('\n');
    
  return `
You are an expert content creator. Create a ${options.format} article about ${options.topic}.

Tone: ${options.tone}
Language: ${options.language}

Research Data:
${researchSummary}

Requirements:
- Use real data and insights from the research
- Include specific examples and statistics
- Make it actionable and engaging
- Format for social media readability
`;
}
```

### 3. OpenAI Integration Alternative

```typescript
// lib/ai/openaiGenerator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithGPT(options: ContentOptions) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a professional content creator specializing in ${options.format} content.`
      },
      {
        role: 'user',
        content: buildPrompt(options)
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Multi-language Content Generation

```typescript
// lib/content/multilingualGenerator.ts
export async function generateMultilingualContent(
  topic: string,
  format: string,
  research: any[]
) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      topic,
      format: format as any,
      tone: 'expert',
      language: 'en',
      research
    }),
    generateContent({
      topic,
      format: format as any,
      tone: 'expert',
      language: 'vi',
      research
    })
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 5. Video Rendering with Remotion

```typescript
// lib/video/renderVideo.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoOptions {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export async function renderContentVideo(options: VideoOptions) {
  // Get video dimensions based on format
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  }[options.format];
  
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: options.content,
      title: options.title,
      ...dimensions
    },
  });
  
  // Render video
  const outputLocation = `public/videos/${Date.now()}-${options.format}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });
  
  return outputLocation;
}
```

### 6. Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, Sequence } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  width: number;
  height: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  width,
  height
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  const scale = 0.8 + Math.min(0.2, frame / 60);
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <Sequence from={0} durationInFrames={90}>
        <div
          style={{
            display: 'flex',
            flexDirection: 'column',
            justifyContent: 'center',
            alignItems: 'center',
            padding: 60,
            opacity,
            transform: `scale(${scale})`
          }}
        >
          <h1 style={{ 
            color: '#fff', 
            fontSize: 72, 
            fontWeight: 'bold',
            textAlign: 'center',
            marginBottom: 40
          }}>
            {title}
          </h1>
        </div>
      </Sequence>
      
      <Sequence from={90}>
        <div style={{ 
          padding: 60, 
          color: '#fff',
          fontSize: 32,
          lineHeight: 1.6
        }}>
          {content.split('\n').map((line, i) => (
            <p key={i} style={{ marginBottom: 20 }}>{line}</p>
          ))}
        </div>
      </Sequence>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Workflow

```typescript
// lib/pipeline/contentPipeline.ts
export async function runContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Crawling news sources...');
  const research = await crawlNews({
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    keywords: [keyword]
  });
  
  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateMultilingualContent(
    keyword,
    'toplist',
    research
  );
  
  // Step 3: Render Video
  console.log('🎬 Rendering video...');
  const videoPath = await renderContentVideo({
    content: content.en,
    title: keyword,
    format: 'reels'
  });
  
  return {
    research,
    content,
    videoPath
  };
}
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/contentPipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages } = await request.json();
    
    const result = await runContentPipeline(keyword);
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Usage Patterns

### Pattern 1: Scheduled Content Generation

```typescript
// lib/scheduler/contentScheduler.ts
import cron from 'node-cron';

export function scheduleContentGeneration(keywords: string[]) {
  // Run every day at 9 AM
  cron.schedule('0 9 * * *', async () => {
    for (const keyword of keywords) {
      await runContentPipeline(keyword);
    }
  });
}
```

### Pattern 2: Batch Video Rendering

```typescript
export async function batchRenderVideos(contents: Array<{title: string, content: string}>) {
  const formats = ['reels', 'tiktok', 'shorts'] as const;
  
  const renderPromises = contents.flatMap(content =>
    formats.map(format => 
      renderContentVideo({
        ...content,
        format
      })
    )
  );
  
  return await Promise.all(renderPromises);
}
```

### Pattern 3: Content Quality Check

```typescript
export async function validateContent(content: string): Promise<boolean> {
  const checks = {
    hasHeadline: content.split('\n')[0].length > 10,
    hasMinLength: content.length > 500,
    hasStructure: content.includes('\n\n'),
    hasCallToAction: /đăng ký|subscribe|follow|share/i.test(content)
  };
  
  return Object.values(checks).every(check => check);
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rateLimiter.ts
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
```

### Video Rendering Memory Issues

If Remotion fails with memory errors:

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run remotion
```

### Crawler Blocked by Anti-bot

```typescript
// lib/crawler/proxyConfig.ts
export const crawlerConfig = {
  headers: {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
  },
  timeout: 10000,
  retries: 3
};
```

## Best Practices

1. **Always validate research data** before passing to AI generation
2. **Cache AI responses** to reduce API costs
3. **Use streaming** for long content generation
4. **Batch video renders** during off-peak hours
5. **Monitor API quotas** and implement fallbacks
6. **Version control your prompts** for reproducibility
