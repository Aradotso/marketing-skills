---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, script generation, and video creation using Claude/OpenAI and Remotion
triggers:
  - create automated content pipeline
  - generate content with AI research
  - build marketing automation workflow
  - scrape news and generate articles
  - create video content from text
  - automate social media content creation
  - use marketing pipeline share
  - setup AI content generation system
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Marketing Pipeline Share project - a complete AI-powered content automation system that handles research, scriptwriting, article generation, and video rendering. The pipeline integrates Claude 3, OpenAI, web scraping, and Remotion for end-to-end content production.

## What It Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Video/Image Rendering**: Automatically creates videos and infographics using Remotion
5. **Platform Optimization**: Exports content ready for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install

# Copy environment template
cp .env.example .env.local
```

## Configuration

### Required Environment Variables

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_key

# Optional
NODE_ENV=development
```

### Next.js Configuration

The project uses Next.js 13+ with App Router. Typical `next.config.js`:

```typescript
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  experimental: {
    serverActions: true,
  },
  env: {
    OPENAI_API_KEY: process.env.OPENAI_API_KEY,
    ANTHROPIC_API_KEY: process.env.ANTHROPIC_API_KEY,
  },
}

module.exports = nextConfig
```

## Core Architecture

### 1. Research Module (Web Scraping)

```typescript
// lib/research/scraper.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  source: string;
  publishedAt: Date;
  content: string;
}

export async function scrapeNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsArticle[]> {
  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(`https://api.rapidapi.com/news`, {
        params: { q: keyword, source },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
          'X-RapidAPI-Host': 'news-api.rapidapi.com'
        }
      });
      
      articles.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to scrape ${source}:`, error);
    }
  }
  
  return articles;
}

// Extract insights from articles
export function extractInsights(articles: NewsArticle[]): string {
  const combined = articles
    .map(a => `${a.title}\n${a.content}`)
    .join('\n\n');
  
  return combined;
}
```

### 2. AI Content Generation

```typescript
// lib/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: string;
}

export async function generateContentClaude(
  config: ContentConfig
): Promise<string> {
  const prompt = buildPrompt(config);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

export async function generateContentOpenAI(
  config: ContentConfig
): Promise<string> {
  const prompt = buildPrompt(config);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'system',
      content: 'You are an expert content writer specializing in marketing content.'
    }, {
      role: 'user',
      content: prompt
    }],
    max_tokens: 4000,
  });
  
  return completion.choices[0].message.content || '';
}

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings',
    'pov': 'Write from a specific point of view with strong opinions',
    'case-study': 'Analyze a real case with data and outcomes',
    'how-to': 'Provide step-by-step instructions'
  };
  
  const toneInstructions = {
    'expert': 'professional and authoritative',
    'friendly': 'conversational and approachable',
    'humorous': 'witty and entertaining'
  };
  
  return `
Write a ${config.format} article about "${config.keyword}" in ${config.language === 'en' ? 'English' : 'Vietnamese'}.

Format: ${formatInstructions[config.format]}
Tone: ${toneInstructions[config.tone]}

Research Data:
${config.research}

Requirements:
- Use the latest data from the research
- Include specific statistics and examples
- Make it SEO-friendly
- Add a compelling headline
- Length: 1500-2000 words
`;
}
```

### 3. Video Rendering with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  duration: number;
  format: 'reels' | 'tiktok' | 'shorts';
}

const ASPECT_RATIOS = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 },
};

export async function renderVideo(config: VideoConfig): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      slides: config.content,
    },
  });
  
  const { width, height } = ASPECT_RATIOS[config.format];
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      slides: config.content,
    },
    overrideWidth: width,
    overrideHeight: height,
  });
  
  return outputLocation;
}
```

### 4. Complete Pipeline Integration

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeNews, extractInsights } from '@/lib/research/scraper';
import { generateContentClaude } from '@/lib/ai/contentGenerator';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone, includeVideo } = await request.json();
    
    // Step 1: Research
    const articles = await scrapeNews(keyword);
    const research = extractInsights(articles);
    
    // Step 2: Generate Content
    const content = await generateContentClaude({
      keyword,
      format,
      language,
      tone,
      research,
    });
    
    // Step 3: Render Video (optional)
    let videoUrl = null;
    if (includeVideo) {
      const contentSlides = content
        .split('\n\n')
        .filter(p => p.length > 50)
        .slice(0, 5);
      
      const videoPath = await renderVideo({
        title: keyword,
        content: contentSlides,
        duration: 30,
        format: 'reels',
      });
      
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }
    
    return NextResponse.json({
      success: true,
      content,
      videoUrl,
      sources: articles.map(a => a.url),
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Frontend Usage Example

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);

  async function handleGenerate() {
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
          includeVideo: true,
        }),
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  }

  return (
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-3 border rounded"
        />
        
        <button
          onClick={handleGenerate}
          disabled={loading}
          className="bg-blue-600 text-white px-6 py-3 rounded"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
        
        {result && (
          <div className="mt-8 space-y-4">
            <div className="prose max-w-none">
              {result.content}
            </div>
            
            {result.videoUrl && (
              <video controls className="w-full max-w-md">
                <source src={result.videoUrl} type="video/mp4" />
              </video>
            )}
          </div>
        )}
      </div>
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
// lib/batch/processor.ts
export async function batchGenerate(
  keywords: string[],
  config: Omit<ContentConfig, 'keyword' | 'research'>
): Promise<Map<string, string>> {
  const results = new Map<string, string>();
  
  for (const keyword of keywords) {
    const articles = await scrapeNews(keyword);
    const research = extractInsights(articles);
    
    const content = await generateContentClaude({
      ...config,
      keyword,
      research,
    });
    
    results.set(keyword, content);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Content Scheduling

```typescript
// lib/scheduling/scheduler.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export async function scheduleContent(
  content: string,
  publishAt: Date,
  platform: 'facebook' | 'twitter' | 'linkedin'
) {
  return await prisma.scheduledPost.create({
    data: {
      content,
      publishAt,
      platform,
      status: 'pending',
    },
  });
}
```

## Running the Project

```bash
# Development
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (if separate command)
npm run render
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rateLimiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  constructor(private delayMs: number = 1000) {}
  
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
    if (this.processing) return;
    this.processing = true;
    
    while (this.queue.length > 0) {
      const fn = this.queue.shift()!;
      await fn();
      await new Promise(resolve => setTimeout(resolve, this.delayMs));
    }
    
    this.processing = false;
  }
}
```

### Video Rendering Issues

- Ensure Remotion dependencies are installed: `npm install @remotion/bundler @remotion/renderer`
- Check disk space for video output
- Verify REMOTION_LICENSE_KEY for commercial use
- Use `--disable-headless` flag for debugging

### AI Response Parsing

```typescript
export function parseAIResponse(response: string): {
  title: string;
  content: string;
  metadata: Record<string, any>;
} {
  // Extract structured data from AI response
  const lines = response.split('\n');
  const title = lines[0].replace(/^#\s*/, '');
  const content = lines.slice(1).join('\n').trim();
  
  return {
    title,
    content,
    metadata: {
      wordCount: content.split(/\s+/).length,
      generatedAt: new Date().toISOString(),
    },
  };
}
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Implement retry logic** for API failures
3. **Use streaming responses** for real-time content generation
4. **Store generated content** in a database for versioning
5. **Monitor API costs** with usage tracking middleware
6. **Optimize video rendering** by queueing jobs in background workers
