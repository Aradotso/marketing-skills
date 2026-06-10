---
name: marketing-pipeline-share-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, social media posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up AI content pipeline with Claude and OpenAI
  - generate automated marketing content from keywords
  - create AI-powered content workflow with auto-posting
  - build content automation system with video rendering
  - use AI to research trends and generate social media content
  - implement automated content pipeline with Remotion videos
  - configure AI content generator with multi-language support
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use **marketing-pipeline-share**, a comprehensive AI-powered content automation system that handles everything from research and scriptwriting to automatic social media posting and video generation. Built with Next.js and TypeScript, it integrates Claude 3, OpenAI, and Remotion to create a complete content production pipeline.

## What It Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes recent data from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates bilingual content (English & Vietnamese) with customizable tone
4. **Auto Video Rendering**: Transforms written content into videos and infographics using Remotion
5. **Platform Optimization**: Outputs content optimized for Reels, TikTok, Shorts, and social media

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

## Environment Configuration

Create `.env.local` with the following required API keys:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database (if using persistence)
DATABASE_URL=postgresql://user:password@localhost:5432/content_db

# Optional: Social Media APIs (for auto-posting)
FACEBOOK_ACCESS_TOKEN=your_facebook_token
TWITTER_API_KEY=your_twitter_key

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Run video rendering separately
npm run render
```

Access the application at `http://localhost:3000`

## Core API Usage

### 1. Research & Data Collection

```typescript
import { researchContent } from '@/lib/research';
import { ScrapedData } from '@/types/research';

async function collectResearch(keyword: string): Promise<ScrapedData[]> {
  const sources = [
    'techcrunch',
    'a16z',
    'twitter',
    'linkedin'
  ];
  
  const results = await researchContent({
    keyword,
    sources,
    timeframe: '24h',
    maxResults: 50
  });
  
  return results;
}

// Usage
const data = await collectResearch('AI marketing automation');
console.log(`Found ${data.length} relevant articles`);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';
import { ContentFormat, ContentRequest } from '@/types/content';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(request: ContentRequest): Promise<string> {
  const { keyword, format, language, tone, researchData } = request;
  
  const systemPrompt = `You are an expert content creator. Generate a ${format} article about ${keyword} in ${language} with a ${tone} tone. Use the following research data to ensure accuracy and timeliness.`;
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    temperature: 0.7,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: `Research data: ${JSON.stringify(researchData)}\n\nCreate comprehensive content following the ${format} format.`
      }
    ]
  });
  
  return message.content[0].type === 'text' ? message.content[0].text : '';
}

// Usage
const content = await generateContent({
  keyword: 'AI marketing trends 2026',
  format: 'toplist',
  language: 'en',
  tone: 'professional',
  researchData: data
});
```

### 3. Multi-Language Content Generation

```typescript
import { translateContent } from '@/lib/translation';

interface BilingualContent {
  english: string;
  vietnamese: string;
}

async function generateBilingualContent(
  keyword: string,
  format: ContentFormat,
  researchData: ScrapedData[]
): Promise<BilingualContent> {
  // Generate English version
  const englishContent = await generateContent({
    keyword,
    format,
    language: 'en',
    tone: 'professional',
    researchData
  });
  
  // Generate Vietnamese version
  const vietnameseContent = await generateContent({
    keyword,
    format,
    language: 'vi',
    tone: 'friendly',
    researchData
  });
  
  return {
    english: englishContent,
    vietnamese: vietnameseContent
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoConfig } from '@/types/video';

async function renderContentVideo(
  content: string,
  config: VideoConfig
): Promise<string> {
  const bundleLocation = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: config.compositionId || 'ContentVideo',
    inputProps: {
      content,
      theme: config.theme,
      duration: config.duration || 60,
    },
  });
  
  const outputPath = `./out/${Date.now()}-video.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content,
      aspectRatio: config.aspectRatio || '9:16', // Vertical for Reels/TikTok
    },
  });
  
  return outputPath;
}

// Usage
const videoPath = await renderContentVideo(content, {
  compositionId: 'SocialMediaReel',
  aspectRatio: '9:16',
  duration: 30,
  theme: 'modern'
});
```

### 5. Complete Content Pipeline

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    anthropicApiKey: process.env.ANTHROPIC_API_KEY!,
    openaiApiKey: process.env.OPENAI_API_KEY!,
    rapidApiKey: process.env.RAPIDAPI_KEY!,
  });
  
  try {
    // Step 1: Research
    console.log('🔍 Researching...');
    const research = await pipeline.research(keyword);
    
    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await pipeline.generateContent({
      keyword,
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'professional',
      researchData: research,
    });
    
    // Step 3: Create video
    console.log('🎬 Rendering video...');
    const video = await pipeline.renderVideo(content.english, {
      aspectRatio: '9:16',
      duration: 30,
    });
    
    // Step 4: Auto-post (optional)
    if (process.env.FACEBOOK_ACCESS_TOKEN) {
      console.log('📤 Publishing to social media...');
      await pipeline.publishToSocial({
        platforms: ['facebook', 'twitter'],
        content: content.english,
        videoPath: video,
      });
    }
    
    return {
      research,
      content,
      video,
      status: 'success'
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const result = await runFullPipeline('AI content automation');
```

## Content Format Types

```typescript
type ContentFormat = 
  | 'toplist'      // Top 10, Top 5 lists
  | 'pov'          // Point of view / opinion pieces
  | 'case-study'   // Detailed case studies
  | 'how-to'       // Step-by-step guides
  | 'news'         // News articles
  | 'comparison';  // Product/service comparisons

type ToneType = 
  | 'professional' // Formal, business-like
  | 'friendly'     // Casual, approachable
  | 'humorous'     // Light, entertaining
  | 'expert'       // Technical, authoritative
  | 'storytelling'; // Narrative-driven
```

## API Endpoints

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { researchContent } from '@/lib/research';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, sources, timeframe } = req.body;
  
  try {
    const results = await researchContent({
      keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeframe: timeframe || '24h',
      maxResults: 50
    });
    
    res.status(200).json({ results, count: results.length });
  } catch (error) {
    res.status(500).json({ error: 'Research failed' });
  }
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateContent } from '@/lib/ai-generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, format, language, tone, researchData } = req.body;
  
  try {
    const content = await generateContent({
      keyword,
      format,
      language: language || 'en',
      tone: tone || 'professional',
      researchData
    });
    
    res.status(200).json({ content });
  } catch (error) {
    res.status(500).json({ error: 'Content generation failed' });
  }
}
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
import cron from 'node-cron';
import { runFullPipeline } from '@/lib/pipeline';

// Run pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = [
    'AI marketing trends',
    'content automation',
    'social media strategy'
  ];
  
  for (const keyword of keywords) {
    try {
      await runFullPipeline(keyword);
      console.log(`✅ Completed pipeline for: ${keyword}`);
    } catch (error) {
      console.error(`❌ Failed for: ${keyword}`, error);
    }
  }
});
```

### Pattern 2: Batch Processing

```typescript
import { batchGenerateContent } from '@/lib/batch-processor';

async function processBatch(keywords: string[]) {
  const results = await batchGenerateContent({
    keywords,
    format: 'toplist',
    languages: ['en', 'vi'],
    concurrency: 3, // Process 3 at a time
    onProgress: (completed, total) => {
      console.log(`Progress: ${completed}/${total}`);
    }
  });
  
  return results;
}
```

### Pattern 3: Custom Video Templates

```typescript
// src/remotion/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame } from 'remotion';

export const CustomTemplate: React.FC<{
  content: string;
  theme: 'dark' | 'light';
}> = ({ content, theme }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: theme === 'dark' ? '#000' : '#fff',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <h1 style={{ 
        fontSize: 80,
        opacity: Math.min(1, frame / 30)
      }}>
        {content}
      </h1>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
import pLimit from 'p-limit';

const limit = pLimit(2); // Limit concurrent API calls

async function safeApiCall<T>(
  fn: () => Promise<T>,
  retries = 3
): Promise<T> {
  return limit(async () => {
    for (let i = 0; i < retries; i++) {
      try {
        return await fn();
      } catch (error: any) {
        if (error.status === 429 && i < retries - 1) {
          const delay = Math.pow(2, i) * 1000;
          console.log(`Rate limited, waiting ${delay}ms...`);
          await new Promise(resolve => setTimeout(resolve, delay));
          continue;
        }
        throw error;
      }
    }
    throw new Error('Max retries exceeded');
  });
}
```

### Issue: Video Rendering Timeout

```typescript
import { renderMedia } from '@remotion/renderer';

async function renderWithTimeout(
  composition: any,
  serveUrl: string,
  outputPath: string,
  timeoutMs = 300000 // 5 minutes
) {
  return Promise.race([
    renderMedia({
      composition,
      serveUrl,
      outputLocation: outputPath,
      codec: 'h264',
    }),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Render timeout')), timeoutMs)
    ),
  ]);
}
```

### Issue: Memory Issues with Large Content

```typescript
import {WorkerPool } from '@/lib/worker-pool';

const workerPool = new WorkerPool({
  maxWorkers: 4,
  memoryLimit: '2GB'
});

async function processLargeContent(content: string[]) {
  const chunks = chunkArray(content, 10);
  
  for (const chunk of chunks) {
    await workerPool.run(async () => {
      return await generateContent(chunk);
    });
  }
  
  await workerPool.terminate();
}

function chunkArray<T>(array: T[], size: number): T[][] {
  return Array.from(
    { length: Math.ceil(array.length / size) },
    (_, i) => array.slice(i * size, i * size + size)
  );
}
```

### Issue: Research Data Quality

```typescript
import { validateResearchData } from '@/lib/validators';

async function getQualityResearch(keyword: string) {
  const rawData = await researchContent({ keyword });
  
  // Filter and validate
  const qualityData = rawData.filter(item => {
    return (
      validateResearchData(item) &&
      item.relevanceScore > 0.7 &&
      item.publishedAt > new Date(Date.now() - 48 * 60 * 60 * 1000) // Last 48h
    );
  });
  
  if (qualityData.length < 5) {
    console.warn('Insufficient quality data, expanding search...');
    return await researchContent({
      keyword,
      timeframe: '7d',
      sources: ['all']
    });
  }
  
  return qualityData;
}
```

## Performance Optimization

```typescript
import { cache } from '@/lib/cache';

async function getCachedContent(keyword: string, format: ContentFormat) {
  const cacheKey = `content:${keyword}:${format}`;
  
  // Check cache first
  const cached = await cache.get(cacheKey);
  if (cached) {
    console.log('Cache hit!');
    return cached;
  }
  
  // Generate new content
  const content = await generateContent({ keyword, format });
  
  // Cache for 24 hours
  await cache.set(cacheKey, content, 86400);
  
  return content;
}
```

This skill provides comprehensive coverage of the marketing-pipeline-share system for AI content automation, including research, generation, video rendering, and deployment workflows.
