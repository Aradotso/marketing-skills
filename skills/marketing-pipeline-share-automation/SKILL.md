---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video
  - set up marketing pipeline for automated posts
  - generate content from keyword to video
  - create AI-driven content workflow
  - build automated social media content system
  - use marketing pipeline share for content automation
  - implement AI content research and generation
  - set up remotion video rendering pipeline
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive system that automates content creation from research through video generation. The pipeline uses Claude 3/OpenAI for content generation, crawls real-time data from sources like TechCrunch and Twitter, and renders videos using Remotion.

## What This Project Does

Marketing Pipeline Share is an all-in-one content automation system that:

- **Auto-scans research**: Crawls fresh data from news sources, Twitter/X, LinkedIn within 24 hours
- **Generates multi-format content**: Creates blog posts, POV pieces, case studies, how-tos in multiple languages
- **Renders videos automatically**: Converts text content to videos and infographics using Remotion
- **Optimizes for platforms**: Exports content formatted for Reels, TikTok, Shorts

The system transforms a single keyword into a complete content package including research, written content, and video assets.

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

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Research crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { crawlRecentNews } from '@/lib/crawler/news-scraper';
import { analyzeResearchData } from '@/lib/ai/research-analyzer';

async function performResearch(keyword: string) {
  // Crawl fresh data from multiple sources
  const newsData = await crawlRecentNews({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Analyze with AI to extract insights
  const insights = await analyzeResearchData({
    data: newsData,
    model: 'claude-3-opus-20240229',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  return {
    rawData: newsData,
    insights: insights
  };
}
```

### 2. Content Generation with AI

```typescript
import { generateContent } from '@/lib/content/generator';
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function createContentFromResearch(
  keyword: string,
  researchData: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const prompt = `
Based on this research data about "${keyword}":
${JSON.stringify(researchData, null, 2)}

Create a ${format} article in both English and Vietnamese.
Tone: Professional yet friendly.
Include data-backed insights and real examples.
  `;

  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content;
}
```

### 3. Multi-Language Content Generation

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateBilingualContent(
  topic: string,
  tone: 'expert' | 'casual' | 'humorous'
) {
  const systemPrompt = `You are a bilingual content creator. 
Generate content in both English and Vietnamese with ${tone} tone.`;

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: `Create content about: ${topic}` }
    ],
    temperature: 0.7
  });

  return {
    english: extractEnglishVersion(completion.choices[0].message.content),
    vietnamese: extractVietnameseVersion(completion.choices[0].message.content)
  };
}

function extractEnglishVersion(content: string): string {
  // Parse and extract English section
  const match = content.match(/### English\n([\s\S]*?)(?=### Vietnamese|$)/);
  return match ? match[1].trim() : '';
}

function extractVietnameseVersion(content: string): string {
  // Parse and extract Vietnamese section
  const match = content.match(/### Vietnamese\n([\s\S]*?)$/);
  return match ? match[1].trim() : '';
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(
  contentData: {
    title: string;
    keyPoints: string[];
    stats: Array<{ label: string; value: string }>;
  },
  platform: 'reels' | 'tiktok' | 'youtube-shorts'
) {
  // Define video dimensions based on platform
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    'youtube-shorts': { width: 1080, height: 1920 }
  };

  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: contentData
  });

  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${platform}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: contentData,
    ...dimensions[platform]
  });

  return outputPath;
}
```

### 5. Complete Pipeline Workflow

```typescript
import { performResearch } from '@/lib/crawler';
import { generateContent } from '@/lib/ai/generator';
import { renderContentVideo } from '@/lib/video/renderer';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research phase...');
    const research = await performResearch(keyword);

    // Step 2: Generate written content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      keyword,
      researchData: research.insights,
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'professional'
    });

    // Step 3: Extract key points for video
    const videoData = {
      title: content.title,
      keyPoints: content.highlights,
      stats: content.statistics
    };

    // Step 4: Render videos for multiple platforms
    console.log('🎬 Rendering videos...');
    const videos = await Promise.all([
      renderContentVideo(videoData, 'reels'),
      renderContentVideo(videoData, 'tiktok'),
      renderContentVideo(videoData, 'youtube-shorts')
    ]);

    return {
      content,
      videos,
      status: 'success'
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
runContentPipeline('AI marketing automation 2024')
  .then(result => {
    console.log('✅ Pipeline completed:', result);
  })
  .catch(err => {
    console.error('❌ Pipeline failed:', err);
  });
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlRecentNews } from '@/lib/crawler/news-scraper';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources, timeframe } = await request.json();

    const data = await crawlRecentNews({
      keyword,
      sources: sources || ['techcrunch', 'twitter'],
      timeframe: timeframe || '24h'
    });

    return NextResponse.json({
      success: true,
      data
    });
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
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

export async function POST(request: NextRequest) {
  try {
    const { prompt, format, language } = await request.json();

    const message = await anthropic.messages.create({
      model: 'claude-3-opus-20240229',
      max_tokens: 4000,
      messages: [
        {
          role: 'user',
          content: `Generate ${format} content in ${language}: ${prompt}`
        }
      ]
    });

    return NextResponse.json({
      success: true,
      content: message.content
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// src/app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { contentData, platform } = await request.json();

    const videoPath = await renderContentVideo(contentData, platform);

    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${path.basename(videoPath)}`
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Content Format Templates

```typescript
type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';

const formatTemplates: Record<ContentFormat, string> = {
  'toplist': `
# Top [X] [Topic]

## Introduction
[Brief overview]

## 1. [First Item]
[Details with data]

## 2. [Second Item]
[Details with data]

## Conclusion
[Wrap up with insights]
  `,
  'pov': `
# My Take on [Topic]

## The Current Situation
[Context and background]

## Why This Matters
[Personal perspective]

## What I Think Will Happen
[Predictions and insights]
  `,
  'case-study': `
# Case Study: [Company/Product]

## Background
[Context]

## Challenge
[Problem statement]

## Solution
[Approach taken]

## Results
[Data-backed outcomes]
  `,
  'how-to': `
# How to [Achieve Goal]

## Prerequisites
[What you need]

## Step 1: [First Step]
[Detailed instructions]

## Step 2: [Second Step]
[Detailed instructions]

## Conclusion
[Summary and next steps]
  `
};

function getContentTemplate(format: ContentFormat): string {
  return formatTemplates[format];
}
```

### Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run pipeline daily at 6 AM
cron.schedule('0 6 * * *', async () => {
  const keywords = [
    'AI marketing trends',
    'content automation tools',
    'social media strategy'
  ];

  for (const keyword of keywords) {
    try {
      await runContentPipeline(keyword);
      console.log(`✅ Completed pipeline for: ${keyword}`);
    } catch (error) {
      console.error(`❌ Failed for ${keyword}:`, error);
    }
  }
});
```

### Batch Processing

```typescript
async function processBatchKeywords(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runContentPipeline(keyword))
  );

  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');

  return {
    total: keywords.length,
    successful: successful.length,
    failed: failed.length,
    results
  };
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

# Test Remotion video rendering
npm run remotion:preview

# Render video from command line
npm run remotion:render
```

## Remotion Configuration

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');

export default Config;
```

## Troubleshooting

### API Rate Limits

```typescript
import pRetry from 'p-retry';

async function callAIWithRetry(prompt: string) {
  return pRetry(
    async () => {
      const message = await anthropic.messages.create({
        model: 'claude-3-opus-20240229',
        max_tokens: 4000,
        messages: [{ role: 'user', content: prompt }]
      });
      return message;
    },
    {
      retries: 3,
      onFailedAttempt: error => {
        console.log(
          `Attempt ${error.attemptNumber} failed. ${error.retriesLeft} retries left.`
        );
      }
    }
  );
}
```

### Memory Issues with Video Rendering

```typescript
// Increase Node.js memory limit
// In package.json scripts:
{
  "scripts": {
    "remotion:render": "NODE_OPTIONS='--max-old-space-size=4096' remotion render"
  }
}
```

### Missing Environment Variables

```typescript
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}\n` +
      `Please check your .env.local file.`
    );
  }
}

// Call at startup
validateEnv();
```

### Crawler Blocking Issues

```typescript
import { chromium } from 'playwright';

async function crawlWithBrowser(url: string) {
  const browser = await chromium.launch({
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  });

  const context = await browser.newContext({
    userAgent: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
  });

  const page = await context.newPage();
  
  try {
    await page.goto(url, { waitUntil: 'networkidle' });
    const content = await page.content();
    return content;
  } finally {
    await browser.close();
  }
}
```

## Performance Optimization

```typescript
// Cache research results
import NodeCache from 'node-cache';

const researchCache = new NodeCache({ stdTTL: 3600 }); // 1 hour

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = researchCache.get(cacheKey);

  if (cached) {
    console.log('📦 Using cached research data');
    return cached;
  }

  const fresh = await performResearch(keyword);
  researchCache.set(cacheKey, fresh);
  return fresh;
}
```

This skill provides comprehensive guidance for AI agents to implement and extend the Marketing Pipeline Share automation system, covering research automation, AI content generation, and video rendering workflows.
