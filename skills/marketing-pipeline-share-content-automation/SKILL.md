---
name: marketing-pipeline-share-content-automation
description: Automated content pipeline for research, scriptwriting, and video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - generate blog posts and videos automatically
  - set up AI content pipeline with Claude and OpenAI
  - create automated marketing content workflow
  - research and write articles using AI crawlers
  - build content automation system with Remotion
  - generate multilingual content with AI research
  - automate video creation from blog posts
---

# Marketing Pipeline Share - Content Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## What This Project Does

Marketing Pipeline Share is an all-in-one AI-powered content automation system that handles the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 and OpenAI
3. **Multi-language Support**: Generates content in both English and Vietnamese with customizable tone
4. **Video/Image Rendering**: Automatically converts written content into videos and infographics using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

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

Create a `.env.local` file in the project root:

```env
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Key Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping & research
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/           # Remotion video templates
```

## Core Usage Patterns

### 1. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

// Generate article with AI
async function createArticle(topic: string) {
  const content = await generateContent({
    topic,
    format: 'toplist', // 'pov' | 'case-study' | 'how-to'
    language: 'vi', // 'en' | 'vi'
    tone: 'expert', // 'friendly' | 'humorous'
    provider: 'claude', // 'openai' | 'claude'
  });
  
  return content;
}

// Example usage
const article = await createArticle('AI Marketing Trends 2024');
console.log(article.title, article.content, article.metadata);
```

### 2. Auto-Research & Crawling

```typescript
import { crawlSources } from '@/lib/crawler/research';

// Fetch latest news and insights
async function researchTopic(keyword: string) {
  const sources = await crawlSources({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 20,
  });
  
  // Extract insights
  const insights = sources.map(source => ({
    title: source.title,
    summary: source.summary,
    url: source.url,
    publishedAt: source.publishedAt,
  }));
  
  return insights;
}

// Example usage
const research = await researchTopic('generative AI marketing');
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';

// Convert article to video
async function generateVideoFromArticle(article: Article) {
  const videoConfig = {
    composition: 'ContentVideo',
    inputProps: {
      title: article.title,
      content: article.content,
      images: article.images,
      duration: 60, // seconds
    },
    outputFormat: 'mp4',
    aspectRatio: '9:16', // For Reels/TikTok
  };
  
  const videoPath = await renderVideo(videoConfig);
  return videoPath;
}
```

### 4. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Full automation pipeline
async function runContentPipeline(topic: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    language: 'vi',
    generateVideo: true,
  });
  
  // Execute full pipeline
  const result = await pipeline.execute({
    topic,
    steps: [
      'research',      // Crawl data
      'analyze',       // Extract insights
      'write',         // Generate article
      'translate',     // Create multi-language versions
      'render',        // Generate video/images
    ],
  });
  
  return {
    article: result.content,
    video: result.videoPath,
    images: result.images,
    research: result.researchData,
  };
}

// Example
const output = await runContentPipeline('ChatGPT for Marketing');
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(req: NextRequest) {
  try {
    const { topic, format, language } = await req.json();
    
    const content = await generateContent({
      topic,
      format,
      language,
      provider: process.env.AI_PROVIDER || 'claude',
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

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlSources } from '@/lib/crawler/research';

export async function POST(req: NextRequest) {
  try {
    const { keyword, sources, timeRange } = await req.json();
    
    const results = await crawlSources({
      keyword,
      sources,
      timeRange,
      apiKey: process.env.RAPIDAPI_KEY,
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

## Common Component Patterns

### Content Generator Component

```typescript
'use client';

import { useState } from 'react';

export function ContentGeneratorForm() {
  const [topic, setTopic] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  async function handleGenerate() {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          topic,
          format: 'toplist',
          language: 'vi',
        }),
      });
      
      const data = await response.json();
      setResult(data.content);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  }
  
  return (
    <div>
      <input
        type="text"
        value={topic}
        onChange={(e) => setTopic(e.target.value)}
        placeholder="Enter topic..."
      />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      {result && <div>{result.content}</div>}
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
npm run start

# Run Remotion studio (for video editing)
npm run remotion:studio
```

## Video Rendering Commands

```bash
# Render a specific composition
npx remotion render ContentVideo output.mp4 --props='{"title":"My Title"}'

# Render for TikTok/Reels (9:16)
npx remotion render ContentVideo output.mp4 --width=1080 --height=1920

# Render for YouTube (16:9)
npx remotion render ContentVideo output.mp4 --width=1920 --height=1080
```

## TypeScript Type Definitions

```typescript
// types/content.ts
export interface ContentConfig {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone?: 'expert' | 'friendly' | 'humorous';
  provider?: 'openai' | 'claude';
}

export interface Article {
  title: string;
  content: string;
  summary: string;
  metadata: {
    keywords: string[];
    category: string;
    readingTime: number;
  };
  images?: string[];
}

export interface ResearchSource {
  title: string;
  summary: string;
  url: string;
  publishedAt: Date;
  source: string;
}

export interface VideoConfig {
  composition: string;
  inputProps: Record<string, any>;
  outputFormat: 'mp4' | 'gif';
  aspectRatio: '16:9' | '9:16' | '1:1';
}
```

## Troubleshooting

### API Key Issues

```typescript
// lib/utils/validate-env.ts
export function validateEnvVars() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

### Rate Limiting Handler

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private lastCall = 0;
  private minInterval = 1000; // 1 second between calls
  
  async throttle<T>(fn: () => Promise<T>): Promise<T> {
    const now = Date.now();
    const timeSinceLastCall = now - this.lastCall;
    
    if (timeSinceLastCall < this.minInterval) {
      await new Promise(resolve => 
        setTimeout(resolve, this.minInterval - timeSinceLastCall)
      );
    }
    
    this.lastCall = Date.now();
    return fn();
  }
}
```

### Error Handling Pattern

```typescript
// lib/utils/error-handler.ts
export class ContentGenerationError extends Error {
  constructor(
    message: string,
    public provider?: string,
    public retryable = true
  ) {
    super(message);
    this.name = 'ContentGenerationError';
  }
}

export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}
```

## Best Practices

1. **Always validate environment variables** before running the pipeline
2. **Use rate limiting** when calling external APIs to avoid quota issues
3. **Cache research results** to minimize redundant API calls
4. **Implement retry logic** for AI generation failures
5. **Store generated content** before rendering videos (videos take time)
6. **Use webhooks or queues** for long-running video rendering tasks
7. **Monitor API costs** especially for Claude and OpenAI calls
