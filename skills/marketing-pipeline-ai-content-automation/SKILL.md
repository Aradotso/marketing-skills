---
name: marketing-pipeline-ai-content-automation
description: Automated content pipeline from research to video generation using Claude/OpenAI
triggers:
  - how do I set up the AI content pipeline
  - automate content creation with AI research
  - generate videos from blog posts automatically
  - create multilingual content with Claude
  - crawl news sources for content ideas
  - render videos with Remotion integration
  - build automated marketing content workflow
  - use the ultimate AI content pipeline
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the **Ultimate AI Content Pipeline**, a comprehensive TypeScript-based system that automates the entire content creation workflow: from research and crawling news sources, to generating multilingual blog posts, to rendering videos with Remotion.

## What This Project Does

The marketing-pipeline-share project is an all-in-one content automation system that:

- **Auto-crawls** trending news from TechCrunch, a16z, X (Twitter), LinkedIn within the last 24 hours
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Creates multilingual posts** (English & Vietnamese) with customizable tone of voice
- **Renders videos & infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)

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

Create a `.env.local` file in the root directory:

```env
# AI Model APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_API_KEY=your_twitter_key
LINKEDIN_API_TOKEN=your_linkedin_token

# Database (if needed)
DATABASE_URL=your_database_connection

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
│   │   ├── crawlers/    # News crawling modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core Features & Usage

### 1. Research & Crawling

The system crawls multiple sources for fresh content ideas:

```typescript
// src/lib/crawlers/news-aggregator.ts
import { TechCrunchCrawler } from './techcrunch';
import { TwitterCrawler } from './twitter';
import { LinkedInCrawler } from './linkedin';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: Date;
  summary: string;
  source: string;
}

export async function aggregateNews(
  keyword: string,
  timeframe: number = 24
): Promise<NewsArticle[]> {
  const crawlers = [
    new TechCrunchCrawler(),
    new TwitterCrawler(),
    new LinkedInCrawler()
  ];

  const results = await Promise.all(
    crawlers.map(crawler => crawler.search(keyword, timeframe))
  );

  return results.flat().sort((a, b) => 
    b.publishedAt.getTime() - a.publishedAt.getTime()
  );
}
```

Usage example:

```typescript
import { aggregateNews } from '@/lib/crawlers/news-aggregator';

const articles = await aggregateNews('artificial intelligence', 24);
console.log(`Found ${articles.length} recent articles`);
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: NewsArticle[];
}

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
  }

  async generateWithClaude(config: ContentConfig): Promise<string> {
    const systemPrompt = this.buildSystemPrompt(config);
    const userPrompt = this.buildUserPrompt(config);

    const message = await this.anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      temperature: 0.7,
      system: systemPrompt,
      messages: [
        {
          role: 'user',
          content: userPrompt,
        },
      ],
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  async generateWithOpenAI(config: ContentConfig): Promise<string> {
    const systemPrompt = this.buildSystemPrompt(config);
    const userPrompt = this.buildUserPrompt(config);

    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: userPrompt },
      ],
      temperature: 0.7,
      max_tokens: 4096,
    });

    return completion.choices[0]?.message?.content || '';
  }

  private buildSystemPrompt(config: ContentConfig): string {
    const toneMap = {
      expert: 'professional and authoritative',
      friendly: 'conversational and approachable',
      humorous: 'witty and entertaining',
    };

    return `You are an expert content writer specializing in ${config.format} format.
Write in ${config.language === 'en' ? 'English' : 'Vietnamese'} with a ${toneMap[config.tone]} tone.
Use the provided research data to create data-backed, insightful content.`;
  }

  private buildUserPrompt(config: ContentConfig): string {
    const researchSummary = config.research
      .map(article => `- ${article.title}: ${article.summary}`)
      .join('\n');

    return `Create a ${config.format} article about "${config.keyword}".

Recent research data:
${researchSummary}

Requirements:
- Include specific data points and statistics
- Cite sources when relevant
- Make it engaging and actionable
- Optimize for SEO`;
  }
}
```

### 3. Video Generation with Remotion

Automatically render videos from generated content:

```typescript
// src/lib/video/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  keyPoints: string[];
  duration: number;
  aspectRatio: '9:16' | '16:9' | '1:1';
}

export class VideoRenderer {
  async renderContentVideo(config: VideoConfig): Promise<string> {
    const compositionId = this.getCompositionId(config.aspectRatio);
    const bundleLocation = await bundle(
      path.join(process.cwd(), 'remotion/index.ts')
    );

    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: compositionId,
    });

    const outputPath = path.join(
      process.cwd(),
      'public',
      'videos',
      `${Date.now()}.mp4`
    );

    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: {
        title: config.title,
        content: config.content,
        keyPoints: config.keyPoints,
      },
    });

    return outputPath;
  }

  private getCompositionId(aspectRatio: string): string {
    const compositionMap = {
      '9:16': 'ContentVideoVertical',
      '16:9': 'ContentVideoHorizontal',
      '1:1': 'ContentVideoSquare',
    };
    return compositionMap[aspectRatio] || 'ContentVideoVertical';
  }
}
```

### 4. Complete Pipeline Orchestration

Tie everything together in a single workflow:

```typescript
// src/lib/pipeline/content-pipeline.ts
import { aggregateNews } from '@/lib/crawlers/news-aggregator';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { VideoRenderer } from '@/lib/video/video-renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
  videoAspectRatio?: '9:16' | '16:9' | '1:1';
}

export class ContentPipeline {
  private generator: ContentGenerator;
  private videoRenderer: VideoRenderer;

  constructor() {
    this.generator = new ContentGenerator();
    this.videoRenderer = new VideoRenderer();
  }

  async execute(config: PipelineConfig) {
    // Step 1: Research
    console.log('🔍 Crawling news sources...');
    const research = await aggregateNews(config.keyword, 24);
    console.log(`✅ Found ${research.length} articles`);

    // Step 2: Generate content for each language
    const content: Record<string, string> = {};
    
    for (const lang of config.languages) {
      console.log(`✍️ Generating ${lang.toUpperCase()} content...`);
      content[lang] = await this.generator.generateWithClaude({
        keyword: config.keyword,
        format: config.format,
        language: lang,
        tone: config.tone,
        research,
      });
      console.log(`✅ ${lang.toUpperCase()} content generated`);
    }

    // Step 3: Generate video (optional)
    let videoPath: string | null = null;
    if (config.generateVideo) {
      console.log('🎬 Rendering video...');
      const primaryContent = content[config.languages[0]];
      const keyPoints = this.extractKeyPoints(primaryContent);
      
      videoPath = await this.videoRenderer.renderContentVideo({
        title: config.keyword,
        content: primaryContent,
        keyPoints,
        duration: 30,
        aspectRatio: config.videoAspectRatio || '9:16',
      });
      console.log(`✅ Video rendered: ${videoPath}`);
    }

    return {
      content,
      research,
      videoPath,
    };
  }

  private extractKeyPoints(content: string): string[] {
    // Simple extraction - split by headings or bullet points
    const lines = content.split('\n');
    return lines
      .filter(line => line.match(/^[-*•]|^#+/))
      .slice(0, 5)
      .map(line => line.replace(/^[-*•#\s]+/, ''));
  }
}
```

## API Routes (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const pipeline = new ContentPipeline();
    const result = await pipeline.execute({
      keyword: body.keyword,
      format: body.format || 'toplist',
      languages: body.languages || ['en', 'vi'],
      tone: body.tone || 'friendly',
      generateVideo: body.generateVideo || false,
      videoAspectRatio: body.videoAspectRatio || '9:16',
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Frontend Usage

```typescript
// src/app/page.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          languages: ['en', 'vi'],
          tone: 'friendly',
          generateVideo: true,
          videoAspectRatio: '9:16',
        }),
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-8">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="mb-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-3 border rounded"
        />
      </div>

      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="bg-blue-600 text-white px-6 py-3 rounded disabled:opacity-50"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-8">
          <h2 className="text-2xl font-bold mb-4">Results</h2>
          <div className="prose max-w-none">
            <h3>English Content</h3>
            <div className="bg-gray-50 p-4 rounded">
              {result.content.en}
            </div>
            
            <h3 className="mt-6">Vietnamese Content</h3>
            <div className="bg-gray-50 p-4 rounded">
              {result.content.vi}
            </div>

            {result.videoPath && (
              <div className="mt-6">
                <h3>Generated Video</h3>
                <video src={result.videoPath} controls className="w-full" />
              </div>
            )}
          </div>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
// src/lib/scheduler/content-scheduler.ts
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';
import cron from 'node-cron';

export function scheduleContentGeneration(keywords: string[]) {
  // Run daily at 9 AM
  cron.schedule('0 9 * * *', async () => {
    const pipeline = new ContentPipeline();
    
    for (const keyword of keywords) {
      await pipeline.execute({
        keyword,
        format: 'toplist',
        languages: ['en', 'vi'],
        tone: 'expert',
        generateVideo: true,
      });
    }
  });
}
```

### Pattern 2: Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const pipeline = new ContentPipeline();
  const results = [];

  for (const keyword of keywords) {
    const result = await pipeline.execute({
      keyword,
      format: 'how-to',
      languages: ['en'],
      tone: 'friendly',
      generateVideo: false,
    });
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

## Troubleshooting

### Issue: API rate limits exceeded

**Solution**: Implement rate limiting and retry logic:

```typescript
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error?.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Issue: Video rendering fails

**Solution**: Check Remotion configuration and AWS credentials:

```typescript
// Verify environment variables
if (!process.env.REMOTION_AWS_ACCESS_KEY_ID) {
  throw new Error('REMOTION_AWS_ACCESS_KEY_ID not set');
}

// Use local rendering for development
const useLocalRendering = process.env.NODE_ENV === 'development';
```

### Issue: Memory issues with large content

**Solution**: Stream responses and use pagination:

```typescript
async function* streamContent(config: ContentConfig) {
  const stream = await this.anthropic.messages.stream({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [/* ... */],
  });

  for await (const chunk of stream) {
    if (chunk.type === 'content_block_delta' && 
        chunk.delta.type === 'text_delta') {
      yield chunk.delta.text;
    }
  }
}
```

## Running in Development

```bash
# Start Next.js dev server
npm run dev

# Run Remotion studio
npm run remotion

# Build for production
npm run build

# Start production server
npm start
```

## Key TypeScript Types

```typescript
// src/types/index.ts
export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';
export type AspectRatio = '9:16' | '16:9' | '1:1';

export interface GeneratedContent {
  content: Record<Language, string>;
  research: NewsArticle[];
  videoPath?: string;
  metadata: {
    generatedAt: Date;
    keyword: string;
    format: ContentFormat;
  };
}
```
