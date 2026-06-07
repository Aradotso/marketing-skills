---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - "how do I generate automated content with AI"
  - "set up the ultimate content pipeline"
  - "automate content research and video generation"
  - "create AI-powered content workflow"
  - "use remotion for automated video rendering"
  - "crawl news and generate content automatically"
  - "build an AI content automation system"
  - "generate multilingual content with Claude"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete TypeScript-based content automation system that researches trending topics, generates multilingual content, and renders videos automatically using Claude/OpenAI and Remotion.

## Overview

This project provides an end-to-end content pipeline that:
- **Auto-crawls** news sources (TechCrunch, a16z, Twitter, LinkedIn) for fresh data
- **Generates content** in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
- **Renders videos** automatically using Remotion for Reels/TikTok/Shorts
- **Supports multilingual** output (English & Vietnamese by default)
- **Customizes tone** (expert, friendly, humorous) based on target audience

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
cp .env.example .env
```

## Configuration

Create a `.env` file in the project root with the following variables:

```bash
# AI Provider (choose one or both)
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPID_API_KEY=your_rapidapi_key_here

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

## Key Components

### 1. Research Module (Auto-Scan)

The research module crawls news sources and extracts insights:

```typescript
import { researchTopic } from './lib/research/scanner';

// Crawl news from the last 24 hours
const insights = await researchTopic({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  maxResults: 10
});

console.log(insights);
// Returns: { articles: [...], keyInsights: [...], trends: [...] }
```

### 2. Content Generation with AI

Generate content using Claude or OpenAI:

```typescript
import { generateContent } from './lib/ai/content-generator';

const content = await generateContent({
  keyword: 'Marketing Automation',
  format: 'toplist', // 'toplist' | 'pov' | 'case-study' | 'how-to'
  language: 'vi', // 'en' | 'vi'
  tone: 'expert', // 'expert' | 'friendly' | 'humorous'
  provider: 'claude', // 'claude' | 'openai'
  researchData: insights // From research module
});

console.log(content.title);
console.log(content.body);
console.log(content.metadata);
```

### 3. Using Claude API

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return message.content[0].text;
}
```

### 4. Using OpenAI API

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
        content: 'You are an expert content writer.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });

  return completion.choices[0].message.content;
}
```

### 5. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: any) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      duration: 30 // seconds
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.slug}.mp4`,
    inputProps: composition.props,
  });

  console.log('Video rendered successfully!');
}
```

### 6. Complete Pipeline Example

```typescript
import { runContentPipeline } from './lib/pipeline';

async function createContent() {
  const result = await runContentPipeline({
    keyword: 'AI in Marketing 2024',
    formats: ['toplist', 'case-study'],
    languages: ['en', 'vi'],
    includeVideo: true,
    videoFormat: 'reel', // 'reel' | 'tiktok' | 'shorts'
    autoPublish: false,
  });

  console.log('Generated:', result.articles.length, 'articles');
  console.log('Videos:', result.videos.length);
  
  return result;
}

createContent();
```

## Common Patterns

### Multi-format Content Generation

```typescript
const formats = ['toplist', 'pov', 'case-study', 'how-to'];

const allContent = await Promise.all(
  formats.map(format => 
    generateContent({
      keyword: 'Digital Marketing Trends',
      format,
      language: 'en',
      tone: 'expert',
      provider: 'claude'
    })
  )
);
```

### Bilingual Content Pipeline

```typescript
async function generateBilingual(keyword: string) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({ keyword, language: 'en', provider: 'openai' }),
    generateContent({ keyword, language: 'vi', provider: 'claude' })
  ]);

  return { en: englishContent, vi: vietnameseContent };
}
```

### Scheduled Content Research

```typescript
import cron from 'node-cron';

// Run research every day at 6 AM
cron.schedule('0 6 * * *', async () => {
  const topics = ['AI', 'Marketing', 'SaaS'];
  
  for (const topic of topics) {
    const insights = await researchTopic({
      keyword: topic,
      sources: ['techcrunch', 'a16z'],
      timeRange: '24h'
    });
    
    // Store or process insights
    await storeInsights(topic, insights);
  }
});
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: Request) {
  try {
    const body = await request.json();
    const { keyword, format, language } = body;

    const content = await generateContent({
      keyword,
      format,
      language,
      tone: 'expert',
      provider: 'claude'
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
import { NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/scanner';

export async function POST(request: Request) {
  const { keyword, sources } = await request.json();

  const insights = await researchTopic({
    keyword,
    sources,
    timeRange: '24h',
    maxResults: 10
  });

  return NextResponse.json(insights);
}
```

## Development

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run tests
npm test

# Lint code
npm run lint
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function retryWithBackoff(fn: Function, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
      } else {
        throw error;
      }
    }
  }
}
```

### Memory Issues with Video Rendering

```typescript
// Render videos sequentially instead of parallel
async function renderVideosSequentially(contents: any[]) {
  const results = [];
  
  for (const content of contents) {
    const video = await renderContentVideo(content);
    results.push(video);
    
    // Clear memory
    if (global.gc) {
      global.gc();
    }
  }
  
  return results;
}
```

### Claude API Timeout

```typescript
// Increase timeout for long-form content
const message = await anthropic.messages.create({
  model: 'claude-3-sonnet-20240229',
  max_tokens: 4096,
  messages: [...],
  timeout: 60000 // 60 seconds
});
```

### Missing Environment Variables

```typescript
// Validate environment on startup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPID_API_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

validateEnv();
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** to avoid hitting API quotas
3. **Cache research results** to reduce redundant API calls
4. **Queue video rendering** for better resource management
5. **Validate inputs** before sending to AI providers
6. **Monitor token usage** to control costs
7. **Store generated content** in a database for reuse
