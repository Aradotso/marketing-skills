---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation with Claude/OpenAI
triggers:
  - automate content creation with AI research
  - generate marketing content from trending topics
  - create video content from text automatically
  - build AI content pipeline with Claude
  - scrape news and generate social posts
  - automate marketing content workflow
  - use remotion for marketing video generation
  - setup AI content automation system
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to use the **Ultimate AI Content Pipeline** - a comprehensive TypeScript-based system that automates content creation from research to video generation. The pipeline crawls trending news from sources like TechCrunch and Twitter, generates multi-format content using Claude/OpenAI, and renders videos automatically using Remotion.

## What It Does

The Marketing Pipeline Share system provides:

- **Auto-Research**: Crawls news from TechCrunch, a16z, X (Twitter), LinkedIn for trending topics
- **AI Content Generation**: Creates multiple formats (Toplist, POV, Case Study, How-to) in English & Vietnamese
- **Video Rendering**: Automatically generates infographics and short-form videos using Remotion
- **Multi-Platform**: Optimized output for Reels, TikTok, Shorts
- **API Integration**: Works with OpenAI, Anthropic Claude, and RapidAPI

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

# Setup environment variables
cp .env.example .env
```

## Environment Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_API_KEY=your_twitter_api_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Run Remotion video rendering
npm run remotion
```

## Core API Usage Patterns

### 1. Research & Data Crawling

```typescript
import { crawlTrendingNews } from '@/lib/research/crawler';
import { analyzeContent } from '@/lib/research/analyzer';

// Crawl trending topics from multiple sources
async function gatherResearch(keyword: string) {
  const sources = await crawlTrendingNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    limit: 10
  });

  // Analyze and extract insights
  const insights = await analyzeContent(sources, {
    extractStats: true,
    findTrends: true,
    generateSummary: true
  });

  return insights;
}

// Usage
const research = await gatherResearch('AI automation');
console.log(research.topTrends);
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
  language: 'en' | 'vi'
) {
  const prompts = {
    toplist: `Create a top 10 list about ${topic} with data-backed insights`,
    pov: `Write a POV perspective article about ${topic} from an expert viewpoint`,
    'case-study': `Develop a detailed case study analyzing ${topic}`,
    'how-to': `Write a comprehensive how-to guide for ${topic}`
  };

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `${prompts[format]}. Language: ${language}. Include statistics, quotes, and actionable insights.`
    }]
  });

  return message.content[0].text;
}

// Generate bilingual content
const englishContent = await generateContent('AI marketing tools', 'toplist', 'en');
const vietnameseContent = await generateContent('AI marketing tools', 'toplist', 'vi');
```

### 3. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function enhanceContentWithGPT(rawContent: string, tone: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content optimizer. Enhance the following content with a ${tone} tone while maintaining factual accuracy.`
      },
      {
        role: 'user',
        content: rawContent
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}

// Usage with different tones
const professionalVersion = await enhanceContentWithGPT(content, 'professional and authoritative');
const friendlyVersion = await enhanceContentWithGPT(content, 'friendly and conversational');
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion.config';
import path from 'path';

async function generateMarketingVideo(
  contentData: {
    title: string;
    points: string[];
    stats: Array<{ label: string; value: string }>;
  },
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'MarketingVideo',
    inputProps: contentData,
  });

  const outputLocation = `./public/videos/${platform}-${Date.now()}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: contentData,
    ...dimensions[platform],
  });

  return outputLocation;
}

// Generate video for multiple platforms
const videoPath = await generateMarketingVideo({
  title: 'Top 5 AI Tools for 2024',
  points: ['Tool 1 Description', 'Tool 2 Description', 'Tool 3 Description'],
  stats: [
    { label: 'Users', value: '1M+' },
    { label: 'Growth', value: '300%' }
  ]
}, 'reels');
```

### 5. Complete Pipeline Orchestration

```typescript
import { crawlTrendingNews, analyzeContent } from '@/lib/research';
import { generateContent } from '@/lib/ai/content-generator';
import { generateMarketingVideo } from '@/lib/video/renderer';
import { schedulePost } from '@/lib/social/scheduler';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Gathering research...');
    const rawData = await crawlTrendingNews({ 
      keyword, 
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h' 
    });
    
    const insights = await analyzeContent(rawData);

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent(
      keyword,
      'toplist',
      'en',
      { insights, tone: 'professional' }
    );

    // Step 3: Create Visual Assets
    console.log('🎬 Rendering video...');
    const videoData = {
      title: content.headline,
      points: content.mainPoints,
      stats: insights.keyStats
    };
    
    const videoPath = await generateMarketingVideo(videoData, 'reels');

    // Step 4: Schedule Publishing
    console.log('📅 Scheduling post...');
    await schedulePost({
      content: content.text,
      media: [videoPath],
      platforms: ['facebook', 'instagram', 'tiktok'],
      scheduledTime: new Date(Date.now() + 3600000) // 1 hour from now
    });

    return {
      success: true,
      content,
      videoPath,
      scheduled: true
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute full pipeline
const result = await runContentPipeline('AI marketing automation');
```

## Next.js API Routes

### Content Generation Endpoint

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateContent } from '@/lib/ai/content-generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, language } = req.body;

  if (!keyword) {
    return res.status(400).json({ error: 'Keyword is required' });
  }

  try {
    const content = await generateContent(
      keyword,
      format || 'toplist',
      language || 'en'
    );

    res.status(200).json({ success: true, content });
  } catch (error) {
    res.status(500).json({ 
      error: 'Content generation failed',
      message: error.message 
    });
  }
}
```

### Video Rendering Endpoint

```typescript
// pages/api/render-video.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateMarketingVideo } from '@/lib/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { contentData, platform } = req.body;

  try {
    const videoPath = await generateMarketingVideo(
      contentData,
      platform || 'reels'
    );

    res.status(200).json({ 
      success: true, 
      videoUrl: videoPath.replace('./public', '') 
    });
  } catch (error) {
    res.status(500).json({ 
      error: 'Video rendering failed',
      message: error.message 
    });
  }
}
```

## Common Patterns

### Error Handling with Retries

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage with API calls
const content = await withRetry(() => 
  generateContent('AI tools', 'toplist', 'en')
);
```

### Caching Research Results

```typescript
import { createClient } from 'redis';

const redis = createClient({
  url: process.env.REDIS_URL
});

async function getCachedResearch(keyword: string, ttl: number = 3600) {
  const cacheKey = `research:${keyword}`;
  
  // Check cache
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);
  
  // Fetch fresh data
  const research = await crawlTrendingNews({ keyword });
  
  // Cache result
  await redis.setEx(cacheKey, ttl, JSON.stringify(research));
  
  return research;
}
```

### Batch Content Generation

```typescript
async function generateBatchContent(
  keywords: string[],
  format: string,
  language: string
) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      generateContent(keyword, format, language)
    )
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    success: result.status === 'fulfilled',
    content: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}

// Generate content for multiple topics
const batch = await generateBatchContent(
  ['AI automation', 'Marketing tools', 'Social media tips'],
  'toplist',
  'en'
);
```

## Troubleshooting

### API Rate Limits

```typescript
import pLimit from 'p-limit';

// Limit concurrent API calls
const limit = pLimit(3);

const contents = await Promise.all(
  keywords.map(keyword => 
    limit(() => generateContent(keyword, 'toplist', 'en'))
  )
);
```

### Remotion Rendering Issues

```bash
# Ensure ffmpeg is installed
brew install ffmpeg  # macOS
sudo apt-get install ffmpeg  # Ubuntu

# Check Remotion configuration
npx remotion versions

# Clear bundle cache
rm -rf node_modules/.cache
```

### Memory Management for Large Videos

```typescript
// Render with lower quality for preview
const previewVideo = await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  scale: 0.5,  // Reduce size
  quality: 60  // Lower quality
});
```

### Claude API Errors

```typescript
async function safeClaudeCall(prompt: string) {
  try {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{ role: 'user', content: prompt }]
    });
    return message.content[0].text;
  } catch (error) {
    if (error.status === 429) {
      // Rate limit - wait and retry
      await new Promise(resolve => setTimeout(resolve, 5000));
      return safeClaudeCall(prompt);
    }
    throw error;
  }
}
```

This skill enables complete automation of marketing content workflows from research through video generation and publishing.
