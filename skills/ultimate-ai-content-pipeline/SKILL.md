---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up AI content pipeline with auto-research
  - generate videos from AI-written content
  - crawl news sources and create content automatically
  - build automated marketing content workflow
  - create social media content with AI and Remotion
  - automate research to video content pipeline
  - set up multi-language content generation system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript-based content automation system handles the complete pipeline from research (crawling TechCrunch, Twitter, LinkedIn) to AI content generation (Claude/OpenAI) to video rendering (Remotion). It automates content creation with real-time data, multi-language support, and automatic video generation for social media.

## What It Does

- **Auto-Research**: Crawls news sources and social media for trending topics
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multi-Language**: Generates content in English and Vietnamese simultaneously
- **Video Rendering**: Automatically converts written content into videos using Remotion
- **Platform Optimization**: Exports videos for Reels, TikTok, Shorts with proper aspect ratios

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

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

## Core Architecture

The pipeline consists of three main stages:

1. **Research Module**: Data collection and analysis
2. **Content Generation Module**: AI-powered writing
3. **Video Rendering Module**: Remotion-based video creation

## Usage Patterns

### 1. Running the Research Pipeline

```typescript
import { researchTopic } from './lib/research';

async function startResearch() {
  const topic = "AI automation tools";
  
  const research = await researchTopic({
    keyword: topic,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: 'en'
  });
  
  console.log('Research insights:', research.insights);
  console.log('Data points:', research.dataPoints);
  
  return research;
}
```

### 2. Generating Content with AI

```typescript
import { generateContent } from './lib/content-generator';

async function createArticle(researchData: any) {
  const content = await generateContent({
    research: researchData,
    format: 'toplist', // 'pov' | 'case-study' | 'how-to'
    tone: 'expert', // 'friendly' | 'humorous'
    languages: ['en', 'vi'],
    aiProvider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  });
  
  return content;
}
```

### 3. Claude Integration Example

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithClaude(prompt: string, research: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4000,
    messages: [
      {
        role: 'user',
        content: `Based on this research data: ${research}\n\n${prompt}`
      }
    ],
    system: 'You are an expert content writer creating data-backed articles.'
  });
  
  return message.content[0].text;
}
```

### 4. OpenAI Integration Example

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(prompt: string, research: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content writer creating data-backed articles.'
      },
      {
        role: 'user',
        content: `Based on this research: ${research}\n\n${prompt}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content;
}
```

### 5. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion.config';

async function renderContentVideo(content: any) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      subtitle: content.subtitle,
      keyPoints: content.keyPoints,
      duration: 30 // seconds
    }
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.id}.mp4`,
    inputProps: composition.props
  });
  
  console.log('Video rendered successfully!');
}
```

### 6. Complete Pipeline Example

```typescript
import { researchTopic } from './lib/research';
import { generateContent } from './lib/content-generator';
import { renderVideo } from './lib/video-renderer';

async function runCompletePipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('📡 Starting research...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter', 'linkedin'],
      timeframe: '24h'
    });
    
    // Step 2: Generate Content
    console.log('🧠 Generating content...');
    const content = await generateContent({
      research,
      format: 'toplist',
      tone: 'expert',
      languages: ['en', 'vi'],
      aiProvider: 'claude'
    });
    
    // Step 3: Render Video
    console.log('🎬 Rendering video...');
    const video = await renderVideo({
      content,
      platform: 'reels', // 'tiktok' | 'shorts'
      aspectRatio: '9:16'
    });
    
    console.log('✅ Pipeline complete!');
    
    return {
      research,
      content,
      video
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Run the pipeline
runCompletePipeline('AI content automation')
  .then(result => console.log('Success:', result))
  .catch(err => console.error('Failed:', err));
```

### 7. API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runCompletePipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages } = await request.json();
    
    const result = await runCompletePipeline(keyword);
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### 8. Content Format Types

```typescript
interface ContentOptions {
  research: ResearchData;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  languages: ('en' | 'vi')[];
  aiProvider: 'claude' | 'openai';
  model?: string;
}

interface GeneratedContent {
  id: string;
  title: string;
  subtitle: string;
  body: string;
  keyPoints: string[];
  language: string;
  format: string;
  createdAt: Date;
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video
npm run render

# Type checking
npm run type-check

# Linting
npm run lint
```

## Common Workflows

### Daily Content Automation

```typescript
import { CronJob } from 'cron';

// Schedule daily content generation
const dailyJob = new CronJob('0 9 * * *', async () => {
  const topics = ['AI trends', 'Marketing automation', 'Content creation'];
  
  for (const topic of topics) {
    await runCompletePipeline(topic);
  }
});

dailyJob.start();
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => 
      runCompletePipeline(keyword)
        .catch(err => ({ error: err.message, keyword }))
    )
  );
  
  const successful = results.filter(r => !r.error);
  const failed = results.filter(r => r.error);
  
  console.log(`✅ Success: ${successful.length}`);
  console.log(`❌ Failed: ${failed.length}`);
  
  return { successful, failed };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(3); // 3 concurrent requests max

async function generateMultiple(topics: string[]) {
  return Promise.all(
    topics.map(topic => 
      limit(() => runCompletePipeline(topic))
    )
  );
}
```

### Claude API Errors

- **Error 429**: Rate limit exceeded - implement exponential backoff
- **Error 400**: Invalid request - check message format and token limits
- **Error 500**: Server error - retry with exponential backoff

```typescript
async function retryWithBackoff(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
    }
  }
}
```

### Remotion Rendering Issues

- **Memory errors**: Reduce video quality or duration
- **Slow rendering**: Use `--concurrency` flag to parallelize
- **Missing fonts**: Install required fonts in Docker container

```bash
# Render with custom settings
npx remotion render src/index.ts ContentVideo out/video.mp4 --concurrency=4 --quality=80
```

### Environment Variable Errors

Always validate environment variables at startup:

```typescript
const requiredEnvVars = [
  'ANTHROPIC_API_KEY',
  'OPENAI_API_KEY',
  'RAPIDAPI_KEY'
];

requiredEnvVars.forEach(varName => {
  if (!process.env[varName]) {
    throw new Error(`Missing required environment variable: ${varName}`);
  }
});
```

## Performance Optimization

### Caching Research Data

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN
});

async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  if (cached) return cached;
  
  const research = await researchTopic({ keyword });
  await redis.set(`research:${keyword}`, research, { ex: 86400 }); // 24h cache
  
  return research;
}
```

### Parallel Processing

```typescript
async function optimizedPipeline(keyword: string) {
  const research = await researchTopic({ keyword });
  
  // Generate content in both languages simultaneously
  const [enContent, viContent] = await Promise.all([
    generateContent({ research, language: 'en' }),
    generateContent({ research, language: 'vi' })
  ]);
  
  // Render videos in parallel
  const [enVideo, viVideo] = await Promise.all([
    renderVideo({ content: enContent }),
    renderVideo({ content: viContent })
  ]);
  
  return { enContent, viContent, enVideo, viVideo };
}
```
