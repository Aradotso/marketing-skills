---
name: marketing-pipeline-ai-content-automation
description: Automate end-to-end content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an AI-powered marketing content pipeline
  - generate blog posts and videos automatically
  - create content workflow with Claude and OpenAI
  - automate research to video content generation
  - build AI content automation system
  - use remotion for automated video content
  - create multi-format content from one keyword
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a complete AI-powered content automation system that transforms a single keyword into multi-format content including blog posts, social media content, and videos. It automatically crawls real-time data from news sources (TechCrunch, a16z, Twitter, LinkedIn), generates content using Claude 3 or OpenAI, and renders videos using Remotion.

**Key Capabilities:**
- Auto-research: Crawl and analyze recent news/trends (24h data)
- Multi-format generation: Toplist, POV, Case Study, How-to articles
- Bilingual support: English and Vietnamese
- Video rendering: Auto-generate infographics and short videos
- Platform optimization: Export for Reels, TikTok, Shorts

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
```

## Configuration

Create a `.env.local` file in the root directory:

```env
# AI Model APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video rendering
npm run remotion
```

Access the application at `http://localhost:3000`

## Core Architecture

The pipeline consists of several stages:

```typescript
// Typical content generation flow
type ContentPipeline = {
  research: ResearchPhase;    // Crawl and analyze data
  generation: GenerationPhase; // AI content creation
  formatting: FormatPhase;     // Multi-format conversion
  rendering: RenderPhase;      // Video/image generation
};
```

## Key Components

### 1. Research & Data Crawling

```typescript
// Example: Fetch trending topics from news sources
interface ResearchConfig {
  keyword: string;
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeframe: '24h' | '7d' | '30d';
  language: 'en' | 'vi';
}

async function conductResearch(config: ResearchConfig) {
  const response = await fetch('/api/research', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(config),
  });
  
  const data = await response.json();
  return {
    trends: data.trends,
    insights: data.insights,
    sources: data.sources,
  };
}

// Usage
const research = await conductResearch({
  keyword: 'AI automation',
  sources: ['techcrunch', 'twitter'],
  timeframe: '24h',
  language: 'en',
});
```

### 2. AI Content Generation

```typescript
// Generate content using Claude or OpenAI
interface ContentGenerationRequest {
  research: ResearchData;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  provider: 'claude' | 'openai';
}

async function generateContent(request: ContentGenerationRequest) {
  const response = await fetch('/api/generate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(request),
  });
  
  return await response.json();
}

// Example: Generate a toplist article
const content = await generateContent({
  research: researchData,
  format: 'toplist',
  tone: 'expert',
  language: 'en',
  provider: 'claude',
});
```

### 3. Multi-Format Content Creation

```typescript
// Transform content into different formats
interface ContentTransformation {
  baseContent: string;
  targetFormats: ContentFormat[];
}

type ContentFormat = 
  | { type: 'blog'; wordCount: number }
  | { type: 'social'; platform: 'twitter' | 'linkedin' | 'facebook' }
  | { type: 'video-script'; duration: number };

async function transformContent(transformation: ContentTransformation) {
  const formatted = await Promise.all(
    transformation.targetFormats.map(async (format) => {
      const response = await fetch('/api/transform', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          content: transformation.baseContent,
          format,
        }),
      });
      return response.json();
    })
  );
  
  return formatted;
}
```

### 4. Video Generation with Remotion

```typescript
// Remotion video composition
import { Composition } from 'remotion';
import { ContentVideo } from './components/ContentVideo';

interface VideoConfig {
  title: string;
  points: string[];
  duration: number;
  fps: number;
}

// Register composition
export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'AI Content Pipeline',
          points: ['Research', 'Generate', 'Publish'],
        }}
      />
    </>
  );
};

// Render video programmatically
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(config: VideoConfig) {
  const bundled = await bundle(path.join(__dirname, './remotion/index.ts'));
  
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: config,
  });
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${config.title}.mp4`,
  });
}
```

### 5. End-to-End Pipeline Example

```typescript
// Complete pipeline from keyword to published content
interface PipelineConfig {
  keyword: string;
  language: 'en' | 'vi';
  formats: ContentFormat[];
  includeVideo: boolean;
}

async function runContentPipeline(config: PipelineConfig) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research phase...');
    const research = await conductResearch({
      keyword: config.keyword,
      sources: ['techcrunch', 'twitter', 'linkedin'],
      timeframe: '24h',
      language: config.language,
    });
    
    // Step 2: Generate base content
    console.log('✍️ Generating content...');
    const baseContent = await generateContent({
      research,
      format: 'how-to',
      tone: 'expert',
      language: config.language,
      provider: 'claude',
    });
    
    // Step 3: Transform into multiple formats
    console.log('🔄 Transforming content...');
    const formatted = await transformContent({
      baseContent: baseContent.text,
      targetFormats: config.formats,
    });
    
    // Step 4: Generate video (optional)
    if (config.includeVideo) {
      console.log('🎬 Rendering video...');
      await renderContentVideo({
        title: config.keyword,
        points: baseContent.keyPoints,
        duration: 30,
        fps: 30,
      });
    }
    
    return {
      research,
      content: baseContent,
      formatted,
      status: 'success',
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const result = await runContentPipeline({
  keyword: 'AI Marketing Automation',
  language: 'en',
  formats: [
    { type: 'blog', wordCount: 1500 },
    { type: 'social', platform: 'linkedin' },
    { type: 'video-script', duration: 60 },
  ],
  includeVideo: true,
});
```

## API Routes

### Research API

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();
  
  // Implement crawling logic with RapidAPI
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  const results = await Promise.all(
    sources.map(source => fetchSourceData(source, keyword, rapidApiKey))
  );
  
  return NextResponse.json({
    trends: aggregateTrends(results),
    insights: extractInsights(results),
    sources: results,
  });
}
```

### Content Generation API

```typescript
// app/api/generate/route.ts
import { Anthropic } from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export async function POST(request: NextRequest) {
  const { research, format, tone, language, provider } = await request.json();
  
  const prompt = buildPrompt(research, format, tone, language);
  
  let content;
  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{ role: 'user', content: prompt }],
    });
    
    content = message.content[0].text;
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{ role: 'user', content: prompt }],
    });
    
    content = completion.choices[0].message.content;
  }
  
  return NextResponse.json({
    text: content,
    keyPoints: extractKeyPoints(content),
    metadata: { format, tone, language },
  });
}
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple articles in parallel
async function batchGenerate(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        language: 'en',
        formats: [{ type: 'blog', wordCount: 1000 }],
        includeVideo: false,
      })
    )
  );
  
  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
  }));
}
```

### Content Scheduling

```typescript
// Schedule content publication
interface ScheduledContent {
  content: GeneratedContent;
  publishAt: Date;
  platforms: ('blog' | 'facebook' | 'twitter' | 'linkedin')[];
}

async function scheduleContent(scheduled: ScheduledContent) {
  // Store in database with publication time
  await db.scheduledPosts.create({
    data: {
      content: scheduled.content,
      publishAt: scheduled.publishAt,
      platforms: scheduled.platforms,
      status: 'pending',
    },
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff for rate limiting
async function fetchWithRetry(url: string, options: RequestInit, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      const response = await fetch(url, options);
      if (response.status === 429) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      return response;
    } catch (error) {
      if (i === retries - 1) throw error;
    }
  }
}
```

### Video Rendering Errors

```typescript
// Handle Remotion rendering failures gracefully
async function safeRenderVideo(config: VideoConfig) {
  try {
    await renderContentVideo(config);
    return { success: true };
  } catch (error) {
    console.error('Video rendering failed:', error);
    
    // Fallback: Generate static image instead
    return {
      success: false,
      fallback: await generateStaticImage(config),
    };
  }
}
```

### Memory Management for Large Batches

```typescript
// Process content in chunks to avoid memory issues
async function processInChunks<T>(
  items: T[],
  processor: (item: T) => Promise<any>,
  chunkSize = 5
) {
  const results = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(chunk.map(processor));
    results.push(...chunkResults);
    
    // Allow garbage collection between chunks
    await new Promise(resolve => setTimeout(resolve, 100));
  }
  
  return results;
}
```

## Best Practices

1. **API Key Security**: Always use environment variables, never commit keys
2. **Error Handling**: Wrap all API calls in try-catch with proper logging
3. **Rate Limiting**: Implement delays between requests to external APIs
4. **Caching**: Cache research results to avoid redundant API calls
5. **Monitoring**: Log pipeline stages for debugging and optimization
6. **Testing**: Test with small batches before scaling up production

## Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [Remotion Documentation](https://remotion.dev)
- [Anthropic Claude API](https://docs.anthropic.com)
- [OpenAI API Reference](https://platform.openai.com/docs)
