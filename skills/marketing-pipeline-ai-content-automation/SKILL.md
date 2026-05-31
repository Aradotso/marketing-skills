---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, Facebook posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do i automate content creation with AI
  - set up automated marketing content pipeline
  - generate videos from text automatically
  - crawl news and create content with AI
  - use claude and openai for content automation
  - create facebook posts automatically with AI
  - build ai content workflow with remotion
  - research and generate marketing content pipeline
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the **Ultimate AI Content Pipeline** - a comprehensive TypeScript-based system that automates the entire content creation workflow from research to video generation. The pipeline crawls news sources, generates multi-format content using Claude/OpenAI, posts to Facebook automatically, and renders videos using Remotion.

## What This Project Does

The Marketing Pipeline automates:
- **Research**: Auto-crawls TechCrunch, a16z, Twitter, LinkedIn for trending topics
- **Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 and OpenAI
- **Multi-language Support**: Generates content in English and Vietnamese
- **Auto-posting**: Publishes directly to Facebook pages
- **Video Generation**: Converts content to video using Remotion for Reels/TikTok/Shorts

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

Create a `.env.local` file in the project root:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Facebook Integration
FACEBOOK_PAGE_ACCESS_TOKEN=your_facebook_token
FACEBOOK_PAGE_ID=your_page_id

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   ├── facebook/    # Facebook API integration
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key Components & Usage

### 1. Content Research & Crawling

```typescript
import { crawlNewsources } from '@/lib/crawler/news-crawler';

interface CrawlResult {
  articles: Array<{
    title: string;
    url: string;
    content: string;
    publishedAt: string;
    source: string;
  }>;
}

// Crawl news from last 24 hours
async function researchTopic(keyword: string): Promise<CrawlResult> {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const results = await crawlNewsources({
    keyword,
    sources,
    timeRange: '24h',
    maxResults: 20
  });
  
  return results;
}

// Usage
const research = await researchTopic('AI marketing automation');
console.log(`Found ${research.articles.length} articles`);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string;
}

async function generateContent(request: ContentRequest): Promise<string> {
  const prompt = `You are a professional content writer. 
  Based on this research data:
  ${request.researchData}
  
  Create a ${request.format} article in ${request.language} with a ${request.tone} tone about: ${request.keyword}
  
  Requirements:
  - Use data-backed insights from the research
  - Include statistics and quotes
  - Optimize for engagement
  - Length: 800-1200 words`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Usage
const content = await generateContent({
  keyword: 'AI content automation',
  format: 'how-to',
  language: 'en',
  tone: 'expert',
  researchData: JSON.stringify(research.articles)
});
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(request: ContentRequest): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a professional content writer specializing in ${request.format} format.`
      },
      {
        role: 'user',
        content: `Create content about: ${request.keyword}\n\nResearch data: ${request.researchData}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Facebook Auto-Posting

```typescript
import axios from 'axios';

interface FacebookPost {
  message: string;
  link?: string;
  image?: string;
}

async function postToFacebook(post: FacebookPost): Promise<string> {
  const pageId = process.env.FACEBOOK_PAGE_ID;
  const accessToken = process.env.FACEBOOK_PAGE_ACCESS_TOKEN;
  
  const endpoint = `https://graph.facebook.com/v18.0/${pageId}/feed`;
  
  const response = await axios.post(endpoint, {
    message: post.message,
    link: post.link,
    access_token: accessToken
  });
  
  return response.data.id; // Returns post ID
}

// Schedule and post
async function schedulePost(content: string, publishTime?: Date) {
  const postId = await postToFacebook({
    message: content,
    link: 'https://yourblog.com/article'
  });
  
  console.log(`Posted to Facebook: ${postId}`);
  return postId;
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  duration: number;
  format: 'reels' | 'tiktok' | 'shorts';
}

async function generateVideo(config: VideoConfig): Promise<string> {
  // Aspect ratios for different platforms
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
      title: config.title,
      slides: config.content,
      duration: config.duration
    },
  });

  const outputLocation = path.join(
    process.cwd(), 
    'public/videos',
    `${Date.now()}-${config.format}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    ...dimensions[config.format]
  });

  return outputLocation;
}

// Usage
const videoPath = await generateVideo({
  title: 'AI Marketing Automation Tips',
  content: [
    'Tip 1: Automate research',
    'Tip 2: Use AI for content',
    'Tip 3: Schedule posts'
  ],
  duration: 30,
  format: 'reels'
});
```

## Complete Pipeline Workflow

```typescript
import { crawlNewsources } from '@/lib/crawler/news-crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { postToFacebook } from '@/lib/facebook/publisher';
import { generateVideo } from '@/lib/video/renderer';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('📡 Researching topic...');
    const research = await crawlNewsources({
      keyword,
      sources: ['techcrunch', 'a16z'],
      timeRange: '24h',
      maxResults: 15
    });

    // Step 2: Generate Content
    console.log('🧠 Generating content...');
    const article = await generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: JSON.stringify(research.articles)
    });

    // Step 3: Post to Facebook
    console.log('📱 Posting to Facebook...');
    const postId = await postToFacebook({
      message: article.substring(0, 500) + '...',
      link: 'https://yourblog.com/article'
    });

    // Step 4: Generate Video
    console.log('🎬 Creating video...');
    const bulletPoints = article
      .split('\n')
      .filter(line => line.startsWith('-'))
      .slice(0, 5);
      
    const videoPath = await generateVideo({
      title: keyword,
      content: bulletPoints,
      duration: 30,
      format: 'reels'
    });

    console.log('✅ Pipeline complete!');
    return {
      article,
      postId,
      videoPath
    };

  } catch (error) {
    console.error('❌ Pipeline failed:', error);
    throw error;
  }
}

// Execute
runContentPipeline('AI content marketing trends 2026');
```

## API Routes (Next.js)

### Create Content API

```typescript
// src/app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone } = body;

    const content = await generateContent({
      keyword,
      format,
      language,
      tone,
      researchData: body.researchData
    });

    return NextResponse.json({ 
      success: true, 
      content 
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Trigger Pipeline API

```typescript
// src/app/api/pipeline/run/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  const { keyword } = await request.json();
  
  const result = await runContentPipeline(keyword);
  
  return NextResponse.json(result);
}
```

## Common Patterns

### Multi-language Content Generation

```typescript
async function generateBilingualContent(keyword: string) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: researchData
    }),
    generateContent({
      keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      researchData: researchData
    })
  ]);

  return { englishContent, vietnameseContent };
}
```

### Batch Content Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline(keyword);
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Error Handling & Retry Logic

```typescript
async function generateWithRetry(
  request: ContentRequest, 
  maxRetries = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(request);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Access the UI
# http://localhost:3000
```

## Troubleshooting

### API Key Issues
- Verify all API keys are set in `.env.local`
- Check API key permissions (especially Facebook tokens)
- Ensure Anthropic/OpenAI accounts have sufficient credits

### Crawling Errors
- RapidAPI rate limits may apply
- Some news sources may block requests - use proxies if needed
- Check network connectivity

### Video Rendering Failures
- Ensure Remotion license is valid
- Check disk space for video output
- Verify FFmpeg is installed: `npm run remotion doctor`

### Facebook Posting Issues
- Page access token must have `pages_manage_posts` permission
- Token expiration - regenerate from Facebook Developer Console
- Page ID must match the token's associated page

### Memory Issues with Large Content
```typescript
// Process in chunks for large datasets
async function processInChunks<T>(
  items: T[], 
  chunkSize: number,
  processor: (chunk: T[]) => Promise<void>
) {
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    await processor(chunk);
  }
}
```

## Performance Optimization

### Caching Research Results

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN,
});

async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  if (cached) return cached;
  
  const fresh = await crawlNewsources({ keyword });
  await redis.setex(`research:${keyword}`, 3600, JSON.stringify(fresh));
  
  return fresh;
}
```

This skill provides comprehensive coverage of the Marketing Pipeline AI Content Automation system, enabling AI coding agents to effectively assist developers in implementing automated content workflows.
