---
name: marketing-pipeline-share-automation
description: TypeScript-based AI content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - use AI to generate marketing content pipeline
  - set up automated content workflow with Claude and OpenAI
  - create video content automatically from text
  - build content automation system with Remotion
  - research and generate social media content with AI
  - automate multilingual content creation
  - generate videos from blog posts automatically
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from researching trending topics, generating multilingual scripts (English/Vietnamese), to automatically rendering videos and graphics. It integrates Claude 3, OpenAI, and Remotion to create a fully automated content production pipeline.

**Key capabilities:**
- Auto-crawl recent news from TechCrunch, a16z, Twitter, LinkedIn
- Generate content in multiple formats (toplist, POV, case study, how-to)
- Bilingual content creation (English & Vietnamese)
- Automatic video rendering via Remotion
- Next.js-based dashboard for managing content pipeline

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
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for News Crawling
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Custom endpoints
NEXT_PUBLIC_API_URL=http://localhost:3000/api

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
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript definitions
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Content Crawling

```typescript
import { crawlNews } from '@/lib/crawler';

// Crawl recent news by topic
async function researchTopic(keyword: string) {
  const articles = await crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h',
    limit: 10
  });
  
  return articles.map(article => ({
    title: article.title,
    url: article.url,
    publishedAt: article.publishedAt,
    summary: article.summary
  }));
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

// Generate content using Claude or OpenAI
async function createContent(topic: string, format: 'toplist' | 'pov' | 'case-study' | 'how-to') {
  const content = await generateContent({
    topic,
    format,
    provider: 'claude', // or 'openai'
    languages: ['en', 'vi'],
    tone: 'professional', // or 'friendly', 'humorous'
    includeData: true
  });
  
  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: content.meta
  };
}
```

### 3. Claude Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithClaude(prompt: string, context: string[]) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Context: ${context.join('\n\n')}
        
Task: ${prompt}

Generate comprehensive content based on the latest research data provided.`
      }
    ]
  });
  
  return message.content[0].text;
}
```

### 4. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(systemPrompt: string, userPrompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userPrompt }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content;
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

async function generateVideo(content: any, template: string) {
  // Bundle Remotion project
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: template, // e.g., 'ContentVideo', 'Infographic'
    inputProps: {
      title: content.title,
      body: content.body,
      style: 'modern'
    }
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${content.id}.mp4`,
    inputProps: composition.inputProps
  });
  
  return `out/${content.id}.mp4`;
}
```

## Common Workflow Patterns

### Full Content Pipeline

```typescript
import { crawlNews } from '@/lib/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderVideo } from '@/lib/video/renderer';

async function runContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z'],
    timeRange: '24h'
  });
  
  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const content = await generateContent({
    topic: keyword,
    format: 'toplist',
    provider: 'claude',
    languages: ['en', 'vi'],
    researchData: research
  });
  
  // Step 3: Create video
  console.log('🎬 Rendering video...');
  const videoPath = await renderVideo({
    content: content.en,
    template: 'ContentVideo',
    aspectRatio: '9:16' // For Reels/TikTok
  });
  
  return {
    content,
    video: videoPath,
    research
  };
}
```

### Batch Content Generation

```typescript
async function generateBatchContent(topics: string[]) {
  const results = await Promise.all(
    topics.map(async (topic) => {
      try {
        const pipeline = await runContentPipeline(topic);
        return { topic, success: true, data: pipeline };
      } catch (error) {
        return { topic, success: false, error: error.message };
      }
    })
  );
  
  const successful = results.filter(r => r.success);
  console.log(`✅ Generated ${successful.length}/${topics.length} content pieces`);
  
  return results;
}
```

### Custom AI Prompt Templates

```typescript
const CONTENT_TEMPLATES = {
  toplist: `Create a top 10 list about {topic}.
Include:
- Catchy introduction
- Each item with explanation
- Data-backed insights from: {research}
- Compelling conclusion
Language: {language}
Tone: {tone}`,

  caseStudy: `Write a case study analyzing {topic}.
Structure:
- Background & context
- Challenge/Problem
- Solution approach
- Results with metrics
- Key takeaways
Data sources: {research}
Language: {language}`,

  howTo: `Create a comprehensive how-to guide for {topic}.
Include:
- Overview
- Prerequisites
- Step-by-step instructions
- Tips and best practices
- Common pitfalls
Reference: {research}
Language: {language}`
};

function formatPrompt(template: string, vars: Record<string, any>) {
  return template.replace(/\{(\w+)\}/g, (_, key) => vars[key] || '');
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

# Type checking
npm run type-check

# Lint code
npm run lint

# Render Remotion video (development)
npm run remotion:preview

# Render Remotion video (production)
npm run remotion:render
```

## API Routes

The Next.js app provides several API endpoints:

```typescript
// POST /api/research
// Body: { keyword: string, sources?: string[] }
fetch('/api/research', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ keyword: 'AI automation' })
});

// POST /api/generate
// Body: { topic: string, format: string, language: string }
fetch('/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    topic: 'Marketing automation trends',
    format: 'toplist',
    language: 'en'
  })
});

// POST /api/render-video
// Body: { contentId: string, template: string }
fetch('/api/render-video', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    contentId: 'abc123',
    template: 'ContentVideo'
  })
});
```

## Configuration Options

### AI Provider Settings

```typescript
// lib/config/ai.ts
export const AI_CONFIG = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 3000,
    temperature: 0.7
  },
  defaultProvider: 'claude' as 'claude' | 'openai'
};
```

### Crawler Settings

```typescript
// lib/config/crawler.ts
export const CRAWLER_CONFIG = {
  sources: {
    techcrunch: 'https://techcrunch.com/feed/',
    a16z: 'https://a16z.com/feed/',
    twitter: true, // Use RapidAPI
    linkedin: true  // Use RapidAPI
  },
  defaultTimeRange: '24h',
  maxArticles: 20,
  cacheExpiry: 3600 // 1 hour
};
```

### Video Rendering Settings

```typescript
// remotion/config.ts
export const VIDEO_CONFIG = {
  fps: 30,
  durationInFrames: 900, // 30 seconds at 30fps
  width: 1080,
  height: 1920, // 9:16 for mobile
  aspectRatios: {
    reels: { width: 1080, height: 1920 },
    youtube: { width: 1920, height: 1080 },
    square: { width: 1080, height: 1080 }
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function apiCallWithRetry(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited, retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
}
```

### Memory Issues with Large Videos

```typescript
// Use streaming for large video renders
import { renderMedia } from '@remotion/renderer';

await renderMedia({
  composition,
  serveUrl: bundled,
  codec: 'h264',
  outputLocation: output,
  chromiumOptions: {
    gl: 'angle',
    headless: true
  },
  // Optimize memory usage
  concurrency: 2,
  frameRange: [0, 900]
});
```

### Missing Environment Variables

```typescript
// Validate env vars at startup
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

// Call in app initialization
validateEnv();
```

### Content Generation Errors

```typescript
// Implement fallback provider
async function generateContentWithFallback(prompt: string) {
  try {
    return await generateWithClaude(prompt);
  } catch (error) {
    console.warn('Claude failed, falling back to OpenAI:', error);
    return await generateWithOpenAI(prompt);
  }
}
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Use queue systems** (Bull, BullMQ) for video rendering jobs
3. **Implement proper error handling** with retries for AI APIs
4. **Store rendered videos** in cloud storage (S3, Cloudflare R2)
5. **Monitor API usage** to stay within rate limits
6. **Version control prompts** for reproducible content generation
7. **A/B test different content formats** to optimize engagement
