---
name: ultimate-ai-content-pipeline
description: Vietnamese AI content automation system with research crawling, multi-format content generation, and video rendering using Claude/OpenAI and Remotion
triggers:
  - automate content creation pipeline
  - generate AI content with research
  - create videos from text automatically
  - build marketing content automation
  - scrape news and generate articles
  - auto-publish content to social media
  - setup Vietnamese content pipeline
  - render marketing videos with Remotion
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript-based content automation system automates the entire content creation workflow: from research/crawling news sources, to generating multi-format articles in Vietnamese and English, to rendering videos automatically. Built with Next.js, Claude/OpenAI APIs, and Remotion for video generation.

## What It Does

- **Auto Research**: Crawls news from TechCrunch, a16z, Twitter/X, LinkedIn for fresh insights
- **Multi-format Content**: Generates articles in multiple formats (toplist, POV, case study, how-to)
- **Bilingual Support**: Creates content in both Vietnamese and English
- **Video Generation**: Automatically renders infographics and short-form videos using Remotion
- **Platform Optimization**: Exports videos for Reels, TikTok, Shorts
- **Full Pipeline**: Keyword → Research → Script → Article → Video → Auto-publish

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

# Setup environment variables
cp .env.example .env.local
```

## Configuration

Create `.env.local` with required API keys:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Database (if using)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_RENDER_PATH=/path/to/remotion/project

# Optional: Auto-publish settings
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_token
FACEBOOK_PAGE_ID=your_page_id
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Run video renderer separately
npm run remotion:render
```

## Core API & Usage Patterns

### 1. Research & Crawling Module

```typescript
import { AutoResearcher } from '@/lib/research/auto-researcher';

async function gatherResearch(keyword: string) {
  const researcher = new AutoResearcher({
    apiKey: process.env.RAPIDAPI_KEY,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
  });

  const insights = await researcher.crawl({
    query: keyword,
    timeframe: '24h',
    minRelevance: 0.7
  });

  return {
    articles: insights.articles,
    trends: insights.trends,
    statistics: insights.statistics,
    quotes: insights.expertQuotes
  };
}
```

### 2. Content Generation with Claude/OpenAI

```typescript
import Anthropic from '@anthropic-ai/sdk';
import { OpenAI } from 'openai';

interface ContentGenerationOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'vi' | 'en';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

async function generateContent(options: ContentGenerationOptions) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const systemPrompt = `You are an expert content creator specializing in ${options.format} format.
Language: ${options.language === 'vi' ? 'Vietnamese' : 'English'}
Tone: ${options.tone}
Use the provided research data to create data-backed, trending content.`;

  const userPrompt = `Create a ${options.format} article about: ${options.keyword}

Research Data:
${JSON.stringify(options.researchData, null, 2)}

Requirements:
- Include specific statistics and quotes from research
- Optimize for SEO and engagement
- Add compelling headlines and CTAs
- Format for social media sharing`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: userPrompt
    }],
    system: systemPrompt
  });

  return {
    content: message.content[0].text,
    metadata: {
      format: options.format,
      language: options.language,
      generatedAt: new Date()
    }
  };
}
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRenderOptions {
  articleContent: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

async function renderContentVideo(options: VideoRenderOptions) {
  const compositionId = 'MarketingVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition settings
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
  });

  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const outputPath = `./output/${Date.now()}-${options.platform}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: options.title,
      content: options.articleContent,
      ...dimensions[options.platform]
    },
  });

  return { videoPath: outputPath };
}
```

### 4. Complete Pipeline Example

```typescript
import { AutoResearcher } from '@/lib/research/auto-researcher';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';
import { publishToFacebook } from '@/lib/social/publisher';

async function fullContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const researcher = new AutoResearcher({
      apiKey: process.env.RAPIDAPI_KEY
    });
    const research = await researcher.crawl({
      query: keyword,
      timeframe: '24h'
    });

    // Step 2: Generate Vietnamese content
    console.log('✍️ Generating Vietnamese content...');
    const viContent = await generateContent({
      keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'expert',
      researchData: research
    });

    // Step 3: Generate English content
    console.log('✍️ Generating English content...');
    const enContent = await generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: research
    });

    // Step 4: Render video
    console.log('🎬 Rendering video...');
    const video = await renderContentVideo({
      articleContent: viContent.content,
      title: keyword,
      platform: 'reels'
    });

    // Step 5: Auto-publish
    console.log('📤 Publishing to Facebook...');
    await publishToFacebook({
      message: viContent.content.substring(0, 500),
      videoPath: video.videoPath,
      pageAccessToken: process.env.FACEBOOK_PAGE_ACCESS_TOKEN,
      pageId: process.env.FACEBOOK_PAGE_ID
    });

    return {
      success: true,
      outputs: {
        vietnamese: viContent,
        english: enContent,
        video: video.videoPath
      }
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
fullContentPipeline('AI Marketing Trends 2026')
  .then(result => console.log('✅ Pipeline complete:', result))
  .catch(err => console.error('❌ Pipeline failed:', err));
```

### 5. Next.js API Routes Pattern

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { fullContentPipeline } from '@/lib/pipeline';

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
    const result = await fullContentPipeline(keyword);
    res.status(200).json(result);
  } catch (error) {
    console.error('API Error:', error);
    res.status(500).json({ 
      error: 'Content generation failed',
      details: error.message 
    });
  }
}
```

## Common Patterns

### Batch Processing Multiple Keywords

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    try {
      const result = await fullContentPipeline(keyword);
      results.push({ keyword, status: 'success', ...result });
      
      // Rate limiting
      await new Promise(resolve => setTimeout(resolve, 2000));
    } catch (error) {
      results.push({ keyword, status: 'failed', error: error.message });
    }
  }
  
  return results;
}
```

### Custom Content Templates

```typescript
const contentTemplates = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    prompt: 'Create a numbered list with detailed explanations'
  },
  pov: {
    structure: ['hook', 'argument', 'evidence', 'counter', 'conclusion'],
    prompt: 'Write from a unique perspective with strong opinions'
  },
  caseStudy: {
    structure: ['background', 'challenge', 'solution', 'results', 'lessons'],
    prompt: 'Analyze a specific example with data and outcomes'
  }
};

async function generateFromTemplate(templateType: keyof typeof contentTemplates, data: any) {
  const template = contentTemplates[templateType];
  // Use template structure in AI prompt
}
```

### Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('🤖 Starting daily content generation...');
  
  const trendingKeywords = await fetchTrendingKeywords();
  await batchGenerateContent(trendingKeywords);
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function retryWithBackoff(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
}
```

### Video Rendering Timeouts

```typescript
// Increase timeout for long renders
const renderWithTimeout = async (options: VideoRenderOptions, timeoutMs = 300000) => {
  return Promise.race([
    renderContentVideo(options),
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Render timeout')), timeoutMs)
    )
  ]);
};
```

### Memory Issues with Large Batches

```typescript
// Process in chunks
async function processInChunks<T>(items: T[], chunkSize: number, processor: (item: T) => Promise<any>) {
  const chunks = [];
  for (let i = 0; i < items.length; i += chunkSize) {
    chunks.push(items.slice(i, i + chunkSize));
  }
  
  for (const chunk of chunks) {
    await Promise.all(chunk.map(processor));
    // Clear memory between chunks
    if (global.gc) global.gc();
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
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

// Call at startup
validateEnv();
```

## Key Files Structure

```
marketing-pineline-share/
├── lib/
│   ├── research/
│   │   └── auto-researcher.ts    # News crawling logic
│   ├── ai/
│   │   └── content-generator.ts  # AI content generation
│   ├── video/
│   │   └── renderer.ts          # Remotion video rendering
│   └── social/
│       └── publisher.ts         # Auto-publish to platforms
├── remotion/
│   ├── index.ts                 # Remotion compositions
│   └── components/              # Video templates
├── pages/
│   └── api/                     # Next.js API routes
└── HUONG_DAN_CAI_DAT.md        # Detailed Vietnamese setup guide
```

Refer to `HUONG_DAN_CAI_DAT.md` for complete Vietnamese installation instructions.
