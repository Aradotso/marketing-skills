---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content research and writing
  - generate AI content from trending topics
  - create videos from written content automatically
  - set up an AI content pipeline with Claude
  - automate marketing content creation workflow
  - build automated content research system
  - generate multilingual content with AI
  - create AI-powered content automation pipeline
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use **marketing-pipeline-share**, an end-to-end content automation system that handles research, scriptwriting, and video generation using AI (Claude 3, OpenAI) and Remotion.

## What It Does

The marketing-pipeline-share project is a complete content production pipeline that:

1. **Auto-researches** trending topics from sources like TechCrunch, a16z, Twitter, LinkedIn
2. **Generates content** in multiple formats (listicles, POV, case studies, how-tos) using Claude/OpenAI
3. **Creates multilingual content** (English & Vietnamese) with customizable tone
4. **Renders videos** and infographics automatically using Remotion
5. **Optimizes for platforms** like Reels, TikTok, and YouTube Shorts

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

Create a `.env.local` file in the project root:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the application at `http://localhost:3000`

## Core Architecture

### TypeScript Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research crawlers
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Shared utilities
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key Components & Usage

### 1. Content Research Module

**Crawl trending topics:**

```typescript
import { researchTopics } from '@/lib/research/crawler';

async function getTrendingContent(keyword: string) {
  const research = await researchTopics({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    limit: 10
  });
  
  return research.insights;
}

// Example usage
const insights = await getTrendingContent('AI automation');
console.log(insights);
```

### 2. AI Content Generation with Claude

**Generate content using Anthropic Claude:**

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(topic: string, format: 'toplist' | 'pov' | 'case-study' | 'how-to') {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Create a ${format} article about ${topic}. Include data-backed insights and current trends.`
    }],
  });

  return message.content[0].text;
}

// Generate toplist article
const article = await generateContent('marketing automation tools', 'toplist');
```

### 3. Multilingual Content Generation

**Create content in multiple languages:**

```typescript
interface ContentConfig {
  topic: string;
  format: string;
  languages: string[];
  tone: 'expert' | 'friendly' | 'humorous';
}

async function generateMultilingualContent(config: ContentConfig) {
  const { topic, format, languages, tone } = config;
  
  const results = await Promise.all(
    languages.map(async (lang) => {
      const prompt = `Write a ${format} about ${topic} in ${lang} with a ${tone} tone.`;
      
      const message = await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }],
      });
      
      return {
        language: lang,
        content: message.content[0].text
      };
    })
  );
  
  return results;
}

// Usage
const multiLang = await generateMultilingualContent({
  topic: 'AI content creation',
  format: 'how-to guide',
  languages: ['english', 'vietnamese'],
  tone: 'expert'
});
```

### 4. OpenAI Integration (Alternative)

**Using OpenAI instead of Claude:**

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer creating data-driven articles.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 2000,
  });

  return completion.choices[0].message.content;
}
```

### 5. Video Generation with Remotion

**Create videos from content:**

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

async function generateVideo(config: VideoConfig) {
  const { content, title, format } = config;
  
  // Define dimensions based on format
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      content,
      ...dimensions[format]
    },
  });

  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${format}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });

  return outputLocation;
}

// Usage
const videoPath = await generateVideo({
  content: 'Your generated article content...',
  title: 'Top 10 AI Tools for 2024',
  format: 'reels'
});
```

### 6. Complete Content Pipeline

**End-to-end workflow:**

```typescript
import { researchTopics } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/render';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('Researching topics...');
    const research = await researchTopics({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeRange: '24h',
      limit: 5
    });

    // Step 2: Generate Content
    console.log('Generating content...');
    const content = await generateContent(
      research.insights.join('\n'),
      'toplist'
    );

    // Step 3: Create multilingual versions
    console.log('Creating translations...');
    const translations = await generateMultilingualContent({
      topic: keyword,
      format: 'toplist',
      languages: ['english', 'vietnamese'],
      tone: 'expert'
    });

    // Step 4: Generate Video
    console.log('Rendering video...');
    const video = await generateVideo({
      content: translations[0].content,
      title: `Top Insights: ${keyword}`,
      format: 'reels'
    });

    return {
      research,
      content: translations,
      video
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const result = await runContentPipeline('marketing automation');
```

## API Routes (Next.js)

### Create Content API Endpoint

```typescript
// src/app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/claude';

export async function POST(request: NextRequest) {
  try {
    const { topic, format, language } = await request.json();

    if (!topic || !format) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    const content = await generateContent(topic, format);

    return NextResponse.json({
      success: true,
      content,
      generatedAt: new Date().toISOString()
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

### Video Generation API

```typescript
// src/app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/render';

export async function POST(request: NextRequest) {
  try {
    const { content, title, format } = await request.json();

    const videoPath = await generateVideo({
      content,
      title,
      format: format || 'reels'
    });

    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${path.basename(videoPath)}`
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Video generation failed' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Rate Limiting AI Requests

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function batchGenerateContent(topics: string[]) {
  const promises = topics.map(topic =>
    limit(() => generateContent(topic, 'toplist'))
  );
  
  return Promise.all(promises);
}
```

### Caching Research Results

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN,
});

async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  
  if (cached) {
    return JSON.parse(cached as string);
  }
  
  const fresh = await researchTopics({ keyword });
  await redis.set(`research:${keyword}`, JSON.stringify(fresh), {
    ex: 3600 // 1 hour cache
  });
  
  return fresh;
}
```

### Error Handling with Retry

```typescript
async function retryableGenerate(
  topic: string,
  maxRetries = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(topic, 'toplist');
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}
```

## Configuration

### Customize Content Templates

```typescript
// src/lib/ai/templates.ts
export const contentTemplates = {
  toplist: `Create a numbered list article about {topic}.
Include:
- Clear headline
- Introduction paragraph
- 5-10 items with descriptions
- Data-backed insights
- Conclusion with CTA`,

  pov: `Write an opinion piece about {topic} from an expert perspective.
Include:
- Strong opening statement
- Personal insights
- Industry trends
- Counterarguments
- Actionable takeaways`,

  caseStudy: `Develop a case study on {topic}.
Structure:
- Background/Challenge
- Solution approach
- Implementation
- Results with metrics
- Lessons learned`,

  howTo: `Create a step-by-step guide for {topic}.
Include:
- Clear objective
- Prerequisites
- Numbered steps with details
- Tips and best practices
- Common pitfalls to avoid`
};
```

### Video Template Configuration

```typescript
// remotion/config.ts
export const videoTemplates = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 900, // 30 seconds
    backgroundColor: '#000000'
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 600, // 20 seconds
    backgroundColor: '#FFFFFF'
  },
  shorts: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 1800, // 60 seconds
    backgroundColor: '#000000'
  }
};
```

## Troubleshooting

### API Rate Limits

**Problem:** Getting rate limit errors from AI APIs

**Solution:**
```typescript
import { RateLimiter } from 'limiter';

const limiter = new RateLimiter({
  tokensPerInterval: 50,
  interval: 'minute'
});

async function rateLimitedGenerate(topic: string) {
  await limiter.removeTokens(1);
  return generateContent(topic, 'toplist');
}
```

### Video Rendering Timeouts

**Problem:** Video generation timing out

**Solution:**
```typescript
const renderWithTimeout = async (config: VideoConfig, timeout = 300000) => {
  return Promise.race([
    generateVideo(config),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Render timeout')), timeout)
    )
  ]);
};
```

### Memory Issues with Large Content

**Problem:** Out of memory when processing large batches

**Solution:**
```typescript
async function* processInChunks<T>(
  items: T[],
  chunkSize: number,
  processor: (item: T) => Promise<any>
) {
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    yield await Promise.all(chunk.map(processor));
  }
}

// Usage
for await (const results of processInChunks(topics, 5, generateContent)) {
  console.log('Processed batch:', results);
}
```

### Invalid API Keys

**Problem:** Authentication failures

**Solution:**
```typescript
async function validateAPIKeys() {
  const checks = {
    anthropic: !!process.env.ANTHROPIC_API_KEY,
    openai: !!process.env.OPENAI_API_KEY,
    rapidapi: !!process.env.RAPIDAPI_KEY
  };

  const missing = Object.entries(checks)
    .filter(([_, valid]) => !valid)
    .map(([key]) => key);

  if (missing.length > 0) {
    throw new Error(`Missing API keys: ${missing.join(', ')}`);
  }
}

// Call before running pipeline
await validateAPIKeys();
```

## Build & Deploy

```bash
# Build for production
npm run build

# Start production server
npm run start

# Export static site (if applicable)
npm run export
```

### Docker Deployment

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

This skill enables AI agents to effectively guide developers through setting up and using the marketing-pipeline-share content automation system with TypeScript, Next.js, AI APIs (Claude/OpenAI), and Remotion video generation.
