---
name: marketing-pipeline-auto-content
description: Automated AI content pipeline for research, scriptwriting, video generation, and multi-platform publishing using Claude, OpenAI, and Remotion
triggers:
  - set up automated content generation pipeline
  - create AI-powered marketing content workflow
  - build automated video content from research
  - generate social media content with AI automation
  - configure content pipeline with Claude and OpenAI
  - automate research to video content creation
  - set up multi-language content automation system
  - implement AI content pipeline for marketing
---

# Marketing Pipeline Auto Content

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

**marketing-pipeline-share** is a complete AI-powered content automation pipeline that transforms keywords into published content. It crawls news sources (TechCrunch, a16z, Twitter, LinkedIn), generates articles in multiple formats and languages using Claude/OpenAI, and renders videos/graphics using Remotion. The system handles the entire workflow: research → scriptwriting → video generation → publishing.

Built with TypeScript, Next.js, and integrates with Anthropic Claude, OpenAI, RapidAPI, and Remotion for video rendering.

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
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Data Sources
RAPIDAPI_KEY=your_rapidapi_key

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
DATABASE_URL=your_database_connection_string

# Optional: Social Media APIs for auto-posting
FACEBOOK_PAGE_TOKEN=your_fb_token
TWITTER_API_KEY=your_twitter_key
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the application at `http://localhost:3000`

## Core Architecture

The pipeline consists of four main stages:

1. **Research Module** - Crawls and aggregates content from news sources
2. **Content Generation** - Uses AI to create articles in multiple formats
3. **Media Generation** - Renders videos and graphics via Remotion
4. **Publishing** - Auto-posts to social platforms

## Key Components & Usage

### 1. Research & Content Crawling

```typescript
// lib/research/crawler.ts
import { crawlNewsSources } from '@/lib/research/crawler';

interface ResearchResult {
  articles: Array<{
    title: string;
    url: string;
    content: string;
    publishedAt: string;
    source: string;
  }>;
  insights: string[];
  trends: string[];
}

async function performResearch(keyword: string): Promise<ResearchResult> {
  const result = await crawlNewsSources({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });
  
  return result;
}

// Usage example
const research = await performResearch('AI marketing tools');
console.log(`Found ${research.articles.length} articles`);
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentOptions {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  targetAudience: string;
}

async function generateContent(
  research: ResearchResult,
  options: ContentOptions
): Promise<string> {
  const prompt = buildPrompt(research, options);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(research: ResearchResult, options: ContentOptions): string {
  return `
You are a content marketing expert. Based on the following research data, 
create a ${options.format} article in ${options.language} language 
with a ${options.tone} tone for ${options.targetAudience}.

Research Data:
${JSON.stringify(research, null, 2)}

Format Requirements:
- ${options.format === 'toplist' ? 'Create a numbered list with detailed explanations' : ''}
- ${options.format === 'case-study' ? 'Include real examples and data points' : ''}
- Include relevant statistics from the research
- Add actionable insights
- Optimize for social media sharing
`;
}

// Usage
const content = await generateContent(research, {
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  targetAudience: 'Marketing professionals'
});
```

### 3. OpenAI Alternative Implementation

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(
  research: ResearchResult,
  options: ContentOptions
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content marketing expert specializing in ${options.format} content with a ${options.tone} tone.`
      },
      {
        role: 'user',
        content: buildPrompt(research, options)
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'youtube-shorts';
  duration: number;
}

async function generateVideo(config: VideoConfig): Promise<string> {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: config.content,
      format: config.format,
    },
  });

  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: config.content,
      format: config.format,
    },
  });

  return outputLocation;
}

// Usage
const videoPath = await generateVideo({
  content: generatedArticle,
  format: 'reels',
  duration: 30
});
```

### 5. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { performResearch } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { generateVideo } from '@/lib/video/renderer';
import { publishToSocial } from '@/lib/publishing/social';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: Array<'en' | 'vi'>;
  generateVideo: boolean;
  autoPublish: boolean;
  platforms: Array<'facebook' | 'twitter' | 'linkedin'>;
}

async function runContentPipeline(config: PipelineConfig) {
  try {
    // Step 1: Research
    console.log('Starting research phase...');
    const research = await performResearch(config.keyword);
    
    // Step 2: Generate content for each language
    console.log('Generating content...');
    const contents = await Promise.all(
      config.languages.map(lang =>
        generateContent(research, {
          format: config.contentFormat,
          language: lang,
          tone: 'expert',
          targetAudience: 'Marketing professionals'
        })
      )
    );
    
    // Step 3: Generate video if requested
    let videoPath: string | null = null;
    if (config.generateVideo) {
      console.log('Rendering video...');
      videoPath = await generateVideo({
        content: contents[0],
        format: 'reels',
        duration: 30
      });
    }
    
    // Step 4: Publish if auto-publish enabled
    if (config.autoPublish) {
      console.log('Publishing to social media...');
      await Promise.all(
        config.platforms.map(platform =>
          publishToSocial({
            platform,
            content: contents[0],
            videoPath,
          })
        )
      );
    }
    
    return {
      success: true,
      contents,
      videoPath,
      research
    };
  } catch (error) {
    console.error('Pipeline failed:', error);
    throw error;
  }
}

// Usage example
const result = await runContentPipeline({
  keyword: 'AI content marketing 2024',
  contentFormat: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  autoPublish: true,
  platforms: ['facebook', 'twitter']
});
```

### 6. API Routes (Next.js)

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      contentFormat: body.format || 'toplist',
      languages: body.languages || ['en'],
      generateVideo: body.generateVideo || false,
      autoPublish: body.autoPublish || false,
      platforms: body.platforms || []
    });
    
    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

### 7. Frontend Integration

```typescript
// app/components/PipelineForm.tsx
'use client';

import { useState } from 'react';

export default function PipelineForm() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setLoading(true);

    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          languages: ['en', 'vi'],
          generateVideo: true,
          autoPublish: false,
          platforms: ['facebook']
        })
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Failed:', error);
    } finally {
      setLoading(false);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        required
      />
      <button type="submit" disabled={loading}>
        {loading ? 'Processing...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h3>Result:</h3>
          <pre>{JSON.stringify(result, null, 2)}</pre>
        </div>
      )}
    </form>
  );
}
```

## Configuration Patterns

### Custom Research Sources

```typescript
// config/research-sources.ts
export const researchSources = {
  techcrunch: {
    url: 'https://techcrunch.com',
    apiEndpoint: '/api/articles',
    selector: '.article-content',
    rateLimit: 10 // requests per minute
  },
  twitter: {
    endpoint: 'https://api.twitter.com/2/tweets/search/recent',
    headers: {
      'Authorization': `Bearer ${process.env.TWITTER_API_KEY}`
    }
  },
  // Add custom sources
  customBlog: {
    url: 'https://yourblog.com/rss',
    type: 'rss',
    parser: 'xml'
  }
};
```

### Content Templates

```typescript
// config/content-templates.ts
export const contentTemplates = {
  toplist: {
    structure: [
      'introduction',
      'numbered-items',
      'conclusion',
      'cta'
    ],
    minItems: 5,
    maxItems: 10
  },
  caseStudy: {
    structure: [
      'problem',
      'solution',
      'implementation',
      'results',
      'lessons'
    ],
    includeData: true,
    includeQuotes: true
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import pQueue from 'p-queue';

const queue = new pQueue({
  concurrency: 1,
  interval: 1000,
  intervalCap: 5
});

export async function rateLimitedRequest<T>(
  fn: () => Promise<T>
): Promise<T> {
  return queue.add(fn);
}

// Usage
const result = await rateLimitedRequest(() =>
  anthropic.messages.create({...})
);
```

### Video Rendering Memory Issues

```typescript
// Optimize Remotion rendering for large videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  // Reduce memory usage
  chromiumOptions: {
    gl: 'angle',
    headless: true
  },
  concurrency: 2, // Lower for limited memory
  enforceAudioTrack: false
});
```

### Error Handling

```typescript
// lib/utils/error-handler.ts
export class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'rendering' | 'publishing',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

// Usage in pipeline
try {
  await performResearch(keyword);
} catch (error) {
  throw new PipelineError(
    'Research failed',
    'research',
    error as Error
  );
}
```

## Best Practices

1. **Always validate API keys** before starting pipeline
2. **Implement caching** for research results to avoid redundant API calls
3. **Use queue systems** for video rendering to prevent memory issues
4. **Store generated content** in database before publishing
5. **Implement retry logic** for failed API calls
6. **Monitor token usage** for AI providers to control costs
7. **Version control prompts** separately for easy updates

## Common Workflows

### Daily Automated Content

```typescript
// scripts/daily-automation.ts
import cron from 'node-cron';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI trends', 'marketing automation', 'social media'];
  
  for (const keyword of keywords) {
    await runContentPipeline({
      keyword,
      contentFormat: 'toplist',
      languages: ['en', 'vi'],
      generateVideo: true,
      autoPublish: true,
      platforms: ['facebook', 'twitter']
    });
  }
});
```

This skill enables AI agents to help developers set up and use the complete content automation pipeline effectively.
