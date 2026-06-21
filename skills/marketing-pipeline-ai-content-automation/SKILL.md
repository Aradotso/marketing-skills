---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content automation pipeline
  - create automated content with research and video generation
  - generate blog posts and videos automatically with AI
  - set up marketing content pipeline with Claude and OpenAI
  - automate content creation from keyword to video
  - use the marketing pipeline for automated posting
  - configure AI content research and generation workflow
  - build automated content factory with video rendering
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation system that transforms keywords into complete content pieces including research, scripting, and video generation. It crawls real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses Claude 3 or OpenAI to generate multi-format content (articles, videos, infographics) in multiple languages.

## What It Does

- **Auto Research**: Crawls latest news and insights from major tech/business sources
- **AI Content Generation**: Creates content in various formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Multi-language Support**: Generates content in both English and Vietnamese
- **Video Rendering**: Automatically creates videos and infographics using Remotion
- **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

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
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database (if using persistence)
DATABASE_URL=your_database_url_here

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key_here
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Content crawling logic
│   │   ├── generator/   # Content generation
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
// lib/crawler/research.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface ResearchResult {
  title: string;
  url: string;
  content: string;
  publishedAt: Date;
  source: string;
}

export async function crawlLatestContent(
  keyword: string,
  sources: string[] = ['techcrunch', 'twitter', 'linkedin']
): Promise<ResearchResult[]> {
  const client = new RapidAPIClient({
    apiKey: process.env.RAPIDAPI_KEY!,
  });

  const results: ResearchResult[] = [];

  for (const source of sources) {
    const data = await client.search({
      query: keyword,
      source,
      timeframe: '24h',
    });

    results.push(...data.articles);
  }

  return results;
}

// Usage example
const research = await crawlLatestContent('AI automation marketing');
console.log(`Found ${research.length} articles`);
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: ResearchResult[];
}

export async function generateContentWithClaude(
  config: ContentConfig
): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = buildPrompt(config);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(config: ContentConfig): string {
  const researchSummary = config.researchData
    .map(r => `- ${r.title}: ${r.content.substring(0, 200)}`)
    .join('\n');

  return `
You are an expert content writer. Create a ${config.format} article about "${config.keyword}".

Research Data (last 24h):
${researchSummary}

Requirements:
- Language: ${config.language}
- Tone: ${config.tone}
- Format: ${config.format}
- Include data-backed insights
- Use latest trends and news

Generate the complete article:
`;
}
```

### 3. OpenAI Alternative

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

export async function generateContentWithOpenAI(
  config: ContentConfig
): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const prompt = buildPrompt(config);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert marketing content creator.',
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    temperature: 0.7,
    max_tokens: 4000,
  });

  return completion.choices[0]?.message?.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      aspectRatio: getAspectRatio(config.format),
    },
  });

  const outputLocation = `public/videos/${Date.now()}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });

  return outputLocation;
}

function getAspectRatio(format: string): number {
  const ratios = {
    reels: 9 / 16,
    tiktok: 9 / 16,
    shorts: 9 / 16,
  };
  return ratios[format] || 9 / 16;
}
```

### 5. Complete Pipeline Integration

```typescript
// lib/pipeline/content-factory.ts
export class ContentFactory {
  async createContent(keyword: string, options?: Partial<ContentConfig>) {
    // Step 1: Research
    console.log('🔍 Researching latest content...');
    const research = await crawlLatestContent(keyword);

    // Step 2: Generate Content
    console.log('✍️ Generating article...');
    const config: ContentConfig = {
      keyword,
      format: options?.format || 'toplist',
      language: options?.language || 'en',
      tone: options?.tone || 'expert',
      researchData: research,
    };

    const content = await generateContentWithClaude(config);

    // Step 3: Render Video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo({
      title: keyword,
      content,
      format: 'reels',
    });

    return {
      article: content,
      video: videoPath,
      research,
      metadata: {
        keyword,
        format: config.format,
        language: config.language,
        createdAt: new Date(),
      },
    };
  }
}

// Usage
const factory = new ContentFactory();
const result = await factory.createContent('AI Marketing Automation 2024', {
  format: 'how-to',
  language: 'vi',
  tone: 'friendly',
});
```

## Next.js API Routes

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentFactory } from '@/lib/pipeline/content-factory';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const factory = new ContentFactory();
    const result = await factory.createContent(keyword, {
      format,
      language,
      tone,
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlLatestContent } from '@/lib/crawler/research';

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const keyword = searchParams.get('keyword');
  const sources = searchParams.get('sources')?.split(',');

  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword is required' },
      { status: 400 }
    );
  }

  const results = await crawlLatestContent(keyword, sources);

  return NextResponse.json({
    success: true,
    count: results.length,
    data: results,
  });
}
```

## Frontend Component Example

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'en',
          tone: 'expert',
        }),
      });

      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="p-6">
      <h1 className="text-2xl font-bold mb-4">AI Content Generator</h1>
      
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded w-full mb-4"
      />

      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="bg-blue-500 text-white px-6 py-2 rounded disabled:opacity-50"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-6">
          <h2 className="text-xl font-semibold mb-2">Results</h2>
          <div className="prose max-w-none">
            {result.article}
          </div>
          {result.video && (
            <video src={result.video} controls className="mt-4 w-full max-w-md" />
          )}
        </div>
      )}
    </div>
  );
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion studio (for video template editing)
npm run remotion
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const factory = new ContentFactory();
  const results = [];

  for (const keyword of keywords) {
    const result = await factory.createContent(keyword);
    results.push(result);
    
    // Respect API rate limits
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

### Multi-language Generation

```typescript
async function generateMultiLanguage(keyword: string) {
  const factory = new ContentFactory();
  const languages = ['en', 'vi'];
  
  const contents = await Promise.all(
    languages.map(lang =>
      factory.createContent(keyword, { language: lang })
    )
  );

  return {
    en: contents[0],
    vi: contents[1],
  };
}
```

### Scheduled Content Pipeline

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';

export function scheduleContentGeneration(keywords: string[]) {
  // Run daily at 9 AM
  cron.schedule('0 9 * * *', async () => {
    console.log('Starting scheduled content generation...');
    const factory = new ContentFactory();
    
    for (const keyword of keywords) {
      await factory.createContent(keyword);
    }
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delay = 1000; // ms between requests

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
    await new Promise(resolve => setTimeout(resolve, this.delay));
    this.processing = false;
    this.process();
  }
}

export const limiter = new RateLimiter();
```

### Error Handling

```typescript
async function safeContentGeneration(keyword: string) {
  try {
    const factory = new ContentFactory();
    return await factory.createContent(keyword);
  } catch (error) {
    if (error.message.includes('rate limit')) {
      console.log('Rate limit hit, retrying in 60s...');
      await new Promise(resolve => setTimeout(resolve, 60000));
      return safeContentGeneration(keyword);
    }
    
    console.error('Content generation failed:', error);
    throw error;
  }
}
```

### Video Rendering Issues

If video rendering fails:

1. Check Remotion license key in `.env.local`
2. Ensure FFmpeg is installed: `ffmpeg -version`
3. Verify composition ID matches in both config and template
4. Check memory limits for large renders

```typescript
// Increase memory for large renders
export async function renderWithHighMemory(config: VideoConfig) {
  process.env.NODE_OPTIONS = '--max-old-space-size=4096';
  return renderContentVideo(config);
}
```

## Advanced Configuration

### Custom AI Prompts

```typescript
// lib/prompts/templates.ts
export const PROMPT_TEMPLATES = {
  toplist: (keyword: string, research: string) => `
Create a top 10 list about ${keyword}.
Use this research: ${research}
Make it engaging and data-driven.
`,
  casestudy: (keyword: string, research: string) => `
Write a detailed case study about ${keyword}.
Base it on: ${research}
Include metrics and outcomes.
`,
};
```

### Custom Video Templates

```typescript
// remotion/compositions/CustomTemplate.tsx
import { AbsoluteFill } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  content: string;
}> = ({ title, content }) => {
  return (
    <AbsoluteFill className="bg-gradient-to-br from-blue-500 to-purple-600">
      <div className="flex flex-col items-center justify-center h-full p-8">
        <h1 className="text-4xl font-bold text-white mb-4">{title}</h1>
        <p className="text-xl text-white">{content}</p>
      </div>
    </AbsoluteFill>
  );
};
```
