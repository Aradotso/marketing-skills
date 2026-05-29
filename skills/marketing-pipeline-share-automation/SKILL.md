---
name: marketing-pipeline-share-automation
description: Automate content creation from research to video generation using AI-powered pipelines with Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with marketing pipeline
  - set up AI content generation from research to video
  - create automated marketing content with Claude and OpenAI
  - generate videos from text using Remotion in marketing pipeline
  - build content automation pipeline with AI
  - configure marketing pipeline share for content generation
  - crawl news and generate content automatically
  - create multilingual content with AI automation
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive TypeScript-based content automation system that handles the entire content creation workflow: from research and data crawling, to AI-powered content generation, to automated video rendering. It integrates Claude 3, OpenAI, and Remotion to create a complete content production pipeline.

**Key capabilities:**
- Auto-crawl news from TechCrunch, a16z, Twitter/X, LinkedIn
- Generate content in multiple formats (Top lists, POV, Case Studies, How-to)
- Multilingual support (English & Vietnamese)
- Automatic video and infographic generation with Remotion
- Multiple tone/voice customization

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

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database configuration
DATABASE_URL=your_database_url_here

# Next.js configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── generator/   # Content generation
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Content Research & Crawling

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';

async function researchTopic(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const newsData = await crawlNews({
    keyword,
    sources,
    timeRange: '24h',
    limit: 50
  });
  
  return newsData.articles;
}

// Usage
const articles = await researchTopic('AI automation');
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const prompt = `Create a ${format} article about ${topic} in ${language} with a ${tone} tone.`;
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].text;
}

// Usage
const article = await generateContent(
  'Marketing Automation Tools 2026',
  'toplist',
  'en',
  'expert'
);
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(researchData: any[], format: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content writer specializing in marketing.'
      },
      {
        role: 'user',
        content: `Based on this research: ${JSON.stringify(researchData)}, create a ${format} article.`
      }
    ],
    temperature: 0.7,
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Complete Pipeline Example

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderVideo } from '@/lib/video/remotion-renderer';

async function contentPipeline(keyword: string) {
  // Step 1: Research
  const research = await crawlNews({
    keyword,
    sources: ['techcrunch', 'twitter'],
    timeRange: '24h'
  });
  
  // Step 2: Generate content in both languages
  const contentEN = await generateContent(
    keyword,
    'toplist',
    'en',
    'expert',
    research
  );
  
  const contentVI = await generateContent(
    keyword,
    'toplist',
    'vi',
    'friendly',
    research
  );
  
  // Step 3: Generate video
  const video = await renderVideo({
    content: contentEN,
    format: 'reels', // or 'tiktok', 'shorts'
    template: 'infographic'
  });
  
  return {
    english: contentEN,
    vietnamese: contentVI,
    video: video.url
  };
}

// Execute pipeline
const result = await contentPipeline('AI content automation');
```

### 5. Remotion Video Generation

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <h1 style={{ color: 'white', fontSize: 60 }}>{title}</h1>
      {points.map((point, i) => (
        <p key={i} style={{ 
          opacity: frame > (i + 1) * fps ? 1 : 0,
          transition: 'opacity 0.5s'
        }}>
          {point}
        </p>
      ))}
    </AbsoluteFill>
  );
};

// Render the video
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

async function renderVideo(content: any) {
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content,
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.title}.mp4`,
  });
}
```

## API Routes Examples

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNews } from '@/lib/crawler/news-crawler';

export async function POST(req: NextRequest) {
  const { keyword, sources } = await req.json();
  
  try {
    const data = await crawlNews({ keyword, sources });
    return NextResponse.json({ success: true, data });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/claude-generator';

export async function POST(req: NextRequest) {
  const { topic, format, language, tone, research } = await req.json();
  
  try {
    const content = await generateContent(
      topic,
      format,
      language,
      tone,
      research
    );
    
    return NextResponse.json({ success: true, content });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion video
npm run remotion render

# Preview Remotion compositions
npm run remotion preview

# Type checking
npm run type-check

# Linting
npm run lint
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await crawlNews({ keyword });
      const content = await generateContent(
        keyword,
        'toplist',
        'en',
        'expert',
        research
      );
      return { keyword, content };
    })
  );
  
  return results;
}
```

### Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await getTrendingTopics();
  
  for (const topic of trendingTopics) {
    await contentPipeline(topic);
  }
});
```

### Error Handling Pattern

```typescript
async function safeGenerate(topic: string) {
  try {
    const content = await generateContent(topic, 'toplist', 'en', 'expert');
    return { success: true, content };
  } catch (error) {
    if (error.status === 429) {
      // Rate limit - retry after delay
      await new Promise(resolve => setTimeout(resolve, 60000));
      return safeGenerate(topic);
    }
    
    console.error('Generation failed:', error);
    return { success: false, error: error.message };
  }
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits:

```typescript
// Implement exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Memory Issues with Video Rendering

```typescript
// Render videos in sequence instead of parallel
async function renderVideosSequentially(contents: any[]) {
  const results = [];
  for (const content of contents) {
    const video = await renderVideo(content);
    results.push(video);
    // Allow garbage collection
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  return results;
}
```

### Missing Environment Variables

```typescript
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app startup
validateEnv();
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Implement queue systems** for batch processing
3. **Use streaming responses** for real-time content generation
4. **Monitor API usage** to stay within rate limits
5. **Version control your prompts** for consistent outputs
6. **Store generated content** before video rendering in case of failures
